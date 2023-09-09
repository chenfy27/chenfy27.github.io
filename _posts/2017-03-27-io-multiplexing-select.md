---
layout: post
title: io复用之select模型
category: 网络
keywords:
---

通常，网络程序服务端要能同时接收来自多个客户端的连接，处理请求并做出响应，可以针对每个连接创建子进程专门处理，这种做法的缺点是相耗资源较多，另外如果要牵涉到进程间通信，编程难度会大大增加。

IO复用技术在一定程度上可避免此类问题，无论连接的客户端有多少个，提供服务的进程只有一个。常用的IO复用模型有select/poll/epoll，这里介绍select模型的用法。

select模型中最重要的要属select函数和相应的宏操作了，声明如下：

```c
#include <sys/select.h>
#include <sys/time.h>
struct timeval {
    long tv_sec;
    long tv_usec;
};
int select(int maxfd1, fd_set *rset, fd_set *wset, fd_set *eset, struct timeval *timeout);
FD_ZERO(fd_set *fds);
FD_SET(int fd, fd_set *fds);
FD_CLR(int fd, fd_set *fds);
FD_ISSET(int fd, fd_set *fds);
```

其中：

- maxfd1为最大文件描述符的值+1。
- 返回-1表示出错，0表示超时，其他值为已就绪的文件描述符的个数。
- timeout为超时时长，有三种情况：
    * timeout为NULL，表示无限等待下去，直到有文件描述符就绪或者被信号中断为止。等待可被信号中断，此时select将返回-1，并设置errno为EINTR。
    * timeout->tv_sec = timeout->tv_usec = 0，此时不等待，直接返回，加入集合的描述符都会被测试，返回已就绪的描述符个数。
    * timeout->tv_sec != 0 \|\| timeout->tv_usec != 0，此时设置了最长等待时间，当有描述符就绪或者到了超时时间，函数返回。

以下是基于select的回射服务器例子。

```c
#include <sys/select.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/time.h>
#include <sys/socket.h>
#include <string.h>
#include <arpa/inet.h>

int ListenAt(short port) {
    struct sockaddr_in ser;
    bzero(&ser, sizeof(ser));
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
    int i, serfd, clifd, maxfd, ret;
    fd_set fds, fds1;
    struct timeval timeout;
    char buf[128];

    maxfd = serfd = ListenAt(atoi(argv[1]));
    FD_ZERO(&fds);
    FD_SET(serfd, &fds);
    for (;;) {
        timeout.tv_sec = 3; timeout.tv_usec = 0;
        fds1 = fds;
        ret = select(maxfd + 1, &fds1, NULL, NULL, &timeout);
        if (ret == -1) break;
        if (ret == 0) continue;
        for (i = 0; i <= maxfd; i++) {
            if (FD_ISSET(i, &fds1)) {
                if (i == serfd) {
                    clifd = Accept(serfd);
                    FD_SET(clifd, &fds);
                    if (maxfd < clifd) maxfd = clifd;
                } else {
                    int msglen = read(i, buf, sizeof(buf));
                    if (msglen == 0) {
                        FD_CLR(i, &fds);
                        close(i);
                    } else {
                        write(i, buf, msglen);
                    }
                }
            }
        }
    }
    close(serfd);
    return 0;
}
```

上述示例是在Linux环境下的，Windows下也有相应的接口，但稍有区别。

```c
#include <winsock2.h>
int select(int nfds, fd_set *rset, fd_set *wset, fd_set *eset, const struct timeval *timeout);
```

由于Linux的文件描述符从0开始递增，因此可以找出当前文件描述符数量和最后生成的文件描述符之间的关系，fd_set结构用一个数组就够。而Windows套接字的句柄并非从0开始，且句柄之间无规律可循，因此fd_set除数组外还用了个记录句柄数的变量，从而select函数中第一个参数无意义，只为保证兼容性。

处理fd_set的四个宏的名称、功能和用法与Linux完全一样。 下面是Windows版本示例代码。

```cpp
#include <winsock.h>
#include <windows.h>
#include <stdio.h>
#pragma comment(lib, "ws2_32.lib")
SOCKET ListenAt(short port) {
    WSADATA wsaData;
    SOCKADDR_IN ser;

    WSAStartup(MAKEWORD(2, 2), &wsaData);
    SOCKET sock = socket(AF_INET, SOCK_STREAM, 0);
    memset(&ser, 0, sizeof(ser));
    ser.sin_family = AF_INET;
    ser.sin_addr.s_addr = htonl(INADDR_ANY);
    ser.sin_port = htons(port);
    bind(sock, (SOCKADDR*)&ser, sizeof(ser));
    listen(sock, 5);
    return sock;
}
SOCKET Accept(SOCKET listenSock) {
    SOCKADDR_IN cli;
    int size = sizeof(cli);
    SOCKET cliSock = accept(listenSock, (SOCKADDR*)&cli, &size);
    return cliSock;
}
int main(int argc, char *argv[]) {
    fd_set fds, fds1;
    struct timeval timeout;

    SOCKET sock = ListenAt(atoi(argv[1]));
    FD_ZERO(&fds);
    FD_SET(sock, &fds);
    for (;;) {
        timeout.tv_sec = 3; timeout.tv_usec = 0;
        fds1 = fds;
        int ret = select(0, &fds1, NULL, NULL, &timeout);
        if (ret == SOCKET_ERROR) break;
        if (ret == 0) continue;
        for (size_t i = 0; i < fds.fd_count; i++) {
            if (FD_ISSET(fds.fd_array[i], &fds1)) {
                if (fds.fd_array[i] == sock) {
                    SOCKET clisock = Accept(sock);
                    FD_SET(clisock, &fds);
                } else {
                    char buf[128];
                    int len = recv(fds.fd_array[i], buf, sizeof(buf), 0);
                    if (len == 0) {
                        FD_CLR(fds.fd_array[i], &fds);
                        closesocket(fds.fd_array[i]);
                    } else {
                        send(fds.fd_array[i], buf, len, 0);
                    }
                }
            }
        }
    }
    closesocket(sock);
    return 0;
}
```

其他说明：

- 由于select可能会修改timeout中字段的值，因此每次调select前都要重新赋值。
- 假设检测到某个套接字有内容可读，且内容长度超出了缓存大小，没有关系，本次只读取了前面部分，该套接字仍处理有数据要读状态，下次select调用时，仍会检测到。换句话说，甚至可以每次只读取一个字节，只不过效率会比较低。

