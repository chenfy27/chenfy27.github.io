---
layout: post
title: io复用之epoll
category: 网络
keywords:
---

### select模型的缺陷与优势

调用select函数后，并不是把发生变化的文件描述符单独集中到一起，而是通过观察监视对象fd_set的变化，找出发生变化的文件描述符，因此无法避免针对所有文件描述符的循环语句；而且，作为监视对象的fd_set变量会发生变化，所以调用select函数前要复制并保存原有信息，并在每次调用select函数时传递新的监视对象信息，这是性能上的致命弱点。

然而，大部分操作系统都支持select函数，如果满足以下两个条件，则应优先用select。

- 服务器端接入者少。
- 程序应具有较好的兼容性。

### epoll模型简介

epoll的优点与select的缺点正好相反：

- 无需编写以监视状态变化为目的的针对所有文件描述符的循环语句。
- 调用epoll_wait函数时无需每次都传递监视对象信息。

相关编程接口声明如下：

```c
struct epoll_event {
    __unit32_t events;
    epoll_data_t data;
};

typedef union epoll_data {
    void *ptr;
    int fd;
    __uint32_t u32;
    __uint64_t u64;
}epoll_data_t;

#include <sys/epoll.h>
int epoll_create(int size);
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
```

接口说明：

- epoll_create用于创建epoll例程，size用于指定例程的大小，但该值只是建议，Linux2.6.8之后的内核将忽略该参数值，根据情况调整例程大小。成功返回epoll例程文件描述符，失败返回-1。
- epoll_ctl用于注册/修改/注销监视对象文件描述符到epoll例程。
    - op代表操作类型，可以是EPOLL_CTL_ADD/EPOLL_CTL_DEL/EPOLL_CTL_MOD。
    - epoll_event结构体用于保存发生事件的文件描述符，也可在epoll例程中注册文件描述符时，用于注册关注的事件。
- events中可以保存的常量及所指的事件类型如下，可用按位或传递多个。
    - EPOLLIN：需要读取数据的情况。
    - EPOLLOUT：输出缓冲为空，可以立即发送数据。
    - EPOLLPRI：收到OOB数据的情况。
    - EPOLLRDHUP：断开连接或半关闭的情况，在边缘触发时很有用。
    - EPOLLERR：发生错误的情况。
    - EPOLLET：以边缘触发的方式得到事件通知。
    - EPOLLONESHOT：发生一次事件后，相应文件描述符不再收到事件通知，因此要向epoll_ctl的第二个参数传递EPOLL_CTL_MOD，再次设置事件。

### 基于epoll的回射服务端

```c
#include <sys/epoll.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>

#define EPOLL_SIZE 128

int ListenAt(short port) {
    struct sockaddr_in ser;
    memset(&ser, 0, sizeof(ser));
    ser.sin_family = AF_INET;
    ser.sin_addr.s_addr = htonl(INADDR_ANY);
    ser.sin_port = htons(port);
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    bind(fd, (struct sockaddr*)&ser, sizeof(struct sockaddr));
    listen(fd, 10);
    return fd;
}

int Accept(int serfd) {
    struct sockaddr_in cli;
    socklen_t len = sizeof(cli);
    int clifd = accept(serfd, (struct sockaddr*)&cli, &len);
    return clifd;
}

int main(int argc, char *argv[]) {
    int i, serfd, clifd, epfd, event_cnt;
    struct epoll_event *ep_events, event;
    char buf[128];

    serfd = ListenAt(atoi(argv[1]));

    epfd = epoll_create(EPOLL_SIZE);
    ep_events = malloc(sizeof(struct epoll_event) * EPOLL_SIZE);

    event.events = EPOLLIN;
    event.data.fd = serfd;
    epoll_ctl(epfd, EPOLL_CTL_ADD, serfd, &event);

    for (;;) {
        event_cnt = epoll_wait(epfd, ep_events, EPOLL_SIZE, -1);
        if (-1 == event_cnt) break;
        for (i = 0; i < event_cnt; i++) {
            if (ep_events[i].data.fd == serfd) {
                clifd = Accept(serfd);
                event.events = EPOLLIN;
                event.data.fd = clifd;
                epoll_ctl(epfd, EPOLL_CTL_ADD, clifd, &event);
            } else {
                int msglen = read(ep_events[i].data.fd, buf, sizeof(buf));
                if (0 == msglen) {
                    epoll_ctl(epfd, EPOLL_CTL_DEL, ep_events[i].data.fd, NULL);
                    close(ep_events[i].data.fd);
                } else write(ep_events[i].data.fd, buf, msglen);
            }
        }
    }
    close(serfd);
    close(epfd);
    return 0;
}
```

### 条件触发和边缘触发

条件触发和边缘触发的区别在于发生事件的时间点。对于条件触发方式，只要输入缓冲中还剩有数据，就将以事件方式再次注册；而对于边缘触发，只在输入缓冲收到数据时注册1次事件，即使输入缓冲中还留有数据，也不会再进行注册。select模型是条件触发的，epoll默认也是条件触发的。

对于边缘触发方式，有两点要掌握：

- 通过errno变量验证错误原因。
- 为完成非阻塞IO，更改套接字特性。

这里只需要知道：read函数发现输入缓冲中没有数据可读时返回-1，并且errno置为EAGAIN。

将套接字修改为非阻塞方式可参考如下代码：

```c
#include <fcntl.h>
int flag = fcntl(fd, F_GETFL, 0);
fcntl(fd, F_SETFL, flag | O_NONBLOCK);
```

在边缘触发方式中，由于接收数据时只注册1次事件，因此，一旦发生输入相关事件，就应该读取输入缓冲中的全部数据，因而需要验证输入缓冲是否为空。

以下是采用边缘触发方式的回射服务器代码。

```c
#include <sys/epoll.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <sys/socket.h>
#include <arpa/inet.h>

#define EPOLL_SIZE 128

int ListenAt(short port) {
    struct sockaddr_in ser;
    memset(&ser, 0, sizeof(ser));
    ser.sin_family = AF_INET;
    ser.sin_addr.s_addr = htonl(INADDR_ANY);
    ser.sin_port = htons(port);
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    bind(fd, (struct sockaddr*)&ser, sizeof(struct sockaddr));
    listen(fd, 10);
    return fd;
}

int Accept(int serfd) {
    struct sockaddr_in cli;
    socklen_t len = sizeof(cli);
    int clifd = accept(serfd, (struct sockaddr*)&cli, &len);
    return clifd;
}

void SetNonBlock(int fd) {
    int flag = fcntl(fd, F_GETFL, 0);
    fcntl(fd, F_SETFL, flag | O_NONBLOCK);
}

int main(int argc, char *argv[]) {
    int i, serfd, clifd, epfd, event_cnt;
    struct epoll_event *ep_events, event;
    char buf[128];

    serfd = ListenAt(atoi(argv[1]));
    SetNonBlock(serfd);

    epfd = epoll_create(EPOLL_SIZE);
    ep_events = malloc(sizeof(struct epoll_event) * EPOLL_SIZE);

    event.events = EPOLLIN;
    event.data.fd = serfd;
    epoll_ctl(epfd, EPOLL_CTL_ADD, serfd, &event);

    for (;;) {
        event_cnt = epoll_wait(epfd, ep_events, EPOLL_SIZE, -1);
        if (-1 == event_cnt) break;
        for (i = 0; i < event_cnt; i++) {
            if (ep_events[i].data.fd == serfd) {
                clifd = Accept(serfd);
                SetNonBlock(clifd);
                event.events = EPOLLIN | EPOLLET;
                event.data.fd = clifd;
                epoll_ctl(epfd, EPOLL_CTL_ADD, clifd, &event);
            } else {
                for (;;) {
                    int msglen = read(ep_events[i].data.fd, buf, sizeof(buf));
                    if (0 == msglen) {
                        epoll_ctl(epfd, EPOLL_CTL_DEL, ep_events[i].data.fd, NULL);
                        close(ep_evnets[i].data.fd);
                        break;
                    } else if (msglen < 0) {
                        if (errno == EAGAIN) break;
                    } else write(ep_events[i].data.fd, buf, msglen);
                }
            }
        }
    }
    close(serfd);
    close(epfd);
    return 0;
}
```

与条件触发相比，边缘触发可以分离接收数据和处理数据的时间点。
