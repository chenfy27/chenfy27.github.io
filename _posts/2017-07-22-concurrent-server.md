---
layout: post
title: 常见的并发服务模型
category: 网络
keywords:
---

常见的并发服务器实现模型有以下三类：

- 多进程服务器：创建子进程处理每个请求
- 多路复用服务器：通过捆绑并统一管理I/O对象提供服务
- 多线程服务器：创建线程处理每个请求

### 基于多进程方式

对于每个连接请求，单独创建一个子进程处理，注意点：

- 父子进程关闭不再需要的描述符。由于套接字属于操作系统所有，进程只不过是拥有代表相应套接字的文件描述符，在调用fork后，将会有2个文件描述符指向同一个套接字。由于当套接字存在多个描述符时，只有全部描述符都关闭后，才能销毁它，因此，在调用fork后，父子进程都应把无关的套接字文件描述符关掉。
- 子进程的回收。

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <signal.h>
#include <arpa/inet.h>
#include <sys/socket.h>
int EchoService(int fd)
{
    int len;
    char buf[512];
    while ((len = read(fd, buf, sizeof(buf))) > 0)
        write(fd, buf, len);
    return 0;
}
int ListenAt(int port)
{
    struct sockaddr_in ser;
    memset(&ser, 0, sizeof(ser));
    ser.sin_family = AF_INET;
    ser.sin_addr.s_addr = htonl(INADDR_ANY);
    ser.sin_port = htons(port);

    int listenfd = socket(AF_INET, SOCK_STREAM, 0);
    if (bind(listenfd, (struct sockaddr*)&ser, sizeof(ser)) < 0)
        perror("bind");
    if (listen(listenfd, 10) < 0)
        perror("listen");
    return listenfd;
}
int main(int argc, char *argv[])
{
    int listenfd, clifd;
    struct sockaddr_in cli;
    socklen_t size;
    pid_t pid;

    signal(SIGCHLD, SIG_IGN);
    listenfd = ListenAt(atoi(argv[1]));
    for (;;) {
        size = sizeof(cli);
        if ((clifd = accept(listenfd, (struct sockaddr*)&cli, &size)) < 0)
            continue;
        pid = fork();
        if (pid == 0) {
            close(listenfd);
            EchoService(clifd);
            close(clifd);
            return 0;
        } else {
            close(clifd);
        }
    }
    close(listenfd);
    return 0;
}
```

### 基于多路复用方式

```c
#include <stdio.h>
#include <sys/socket.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <arpa/inet.h>
int ListenAt(int port)
{
    struct sockaddr_in ser;
    memset(&ser, 0, sizeof(ser));
    ser.sin_family = AF_INET;
    ser.sin_addr.s_addr = htonl(INADDR_ANY);
    ser.sin_port = htons(port);

    int fd = socket(AF_INET, SOCK_STREAM, 0);
    if (bind(fd, (struct sockaddr*)&ser, sizeof(ser)) < 0)
        perror("bind");
    if (listen(fd, 10) < 0)
        perror("listen");
    return fd;
}
int main(int argc, char *argv[])
{
    int listenfd, maxfd, ret, clifd;
    struct sockaddr_in cli;
    socklen_t size;
    fd_set fds, fds1;

    maxfd = listenfd = ListenAt(atoi(argv[1]));
    FD_ZERO(&fds);
    FD_SET(listenfd, &fds);
    for (;;) {
        fds1 = fds;
        ret = select(maxfd + 1, &fds1, NULL, NULL, NULL);
        if (ret == -1) break;
        for (int i = 0; i <= maxfd; i++) {
            if (!FD_ISSET(i, &fds1)) continue;
            if (i == listenfd) {
                size = sizeof(cli);
                clifd = accept(listenfd, (struct sockaddr*)&cli, &size);
                FD_SET(clifd, &fds);
                if (maxfd < clifd) maxfd = clifd;
            } else {
                char buf[512];
                int len = read(i, buf, sizeof(buf));
                if (len == 0) {
                    FD_CLR(i, &fds);
                    close(i);
                } else {
                    write(i, buf, len);
                }
            }
        }
    }
    return 0;
}
```
### 基于多线程方式

对每个请求，单独创建一个线程对处理，注意点：

- socket描述符的传递
- 线程的回收

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <pthread.h>
#include <arpa/inet.h>
#include <sys/socket.h>
void* EchoService(void *arg)
{
    int len, fd = (int)arg;
    char buf[512];
    while ((len = read(fd, buf, sizeof(buf))) > 0)
        write(fd, buf, len);
    return NULL;
}
int ListenAt(int port)
{
    struct sockaddr_in ser;
    memset(&ser, 0, sizeof(ser));
    ser.sin_family = AF_INET;
    ser.sin_addr.s_addr = htonl(INADDR_ANY);
    ser.sin_port = htons(port);

    int listenfd = socket(AF_INET, SOCK_STREAM, 0);
    if (bind(listenfd, (struct sockaddr*)&ser, sizeof(ser)) < 0)
        perror("bind");
    if (listen(listenfd, 10) < 0)
        perror("listen");
    return listenfd;
}
int main(int argc, char *argv[])
{
    int listenfd, clifd;
    struct sockaddr_in cli;
    socklen_t size;
    pthread_t tid;

    listenfd = ListenAt(atoi(argv[1]));
    for (;;) {
        size = sizeof(cli);
        if ((clifd = accept(listenfd, (struct sockaddr*)&cli, &size)) < 0)
            continue;
        pthread_create(&tid, NULL, EchoService, (void*)clifd);
        pthread_detach(tid);
    }
    close(listenfd);
    return 0;
}
```

