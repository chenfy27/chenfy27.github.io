---
layout: post
title: tcp socket编程基础
category: 网络
keywords: 
---

### 字节序转换函数

```
unsigned short htons(unsigned short);
unsigned short ntohs(unsigned short);
unsigned long htonl(unsigned long);
unsigned long ntohl(unsigned long);
```

### 地址转换函数

```
in_addr_t inet_addr(const char *ip);
int inet_aton(const char *ip, struct in_addr *addr);
char* inet_ntoa(struct in_addr addr);
```

### Linux版最简回射服务器

```
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
int ListenAt(short port) {
    struct sockaddr_in ser;
    memset(&ser, 0, sizeof(ser));
    ser.sin_family = AF_INET;
    ser.sin_addr.s_addr = htonl(INADDR_ANY);
    ser.sin_port = htons(port);
    int sockfd = socket(AF_INET, SOCK_STREAM, 0);
    bind(sockfd, (struct sockaddr*)&ser, sizeof(ser));
    listen(sockfd, 5);
    return sockfd;
}
int Accept(int listenfd) {
    struct sockaddr_in cli;
    socklen_t size = sizeof(cli);
    return accept(listenfd, (struct sockaddr*)&cli, &size);
}
int main(int argc, char *argv[]) {
    int listenfd = ListenAt(atoi(argv[1]));
    for (;;) {
        char buf[128];
        int len;
        int clifd = Accept(listenfd);
        while ((len = read(clifd, buf, sizeof(buf))) > 0)
            write(clifd, buf, len);
        close(clifd);
    }
    close(listenfd);
    return 0;
}
```

### Linux版最简回射客户端

```
#include <unistd.h>
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
int main(int argc, char *argv[]) {
    char buf[128];
    int ret;
    struct sockaddr_in ser;
    memset(&ser, 0, sizeof(ser));
    ser.sin_family = AF_INET;
    ser.sin_addr.s_addr = inet_addr(argv[1]);
    ser.sin_port = htons(atoi(argv[2]));
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    connect(fd, (struct sockaddr*)&ser, sizeof(ser));
    while (-1 != (ret = scanf("%[^\n]%*c", buf))) {
        if (ret == 0) {
            fgets(buf, sizeof(buf), stdin);
            continue;
        }
        write(fd, buf, strlen(buf));
        memset(buf, 0, sizeof(buf));
        read(fd, buf, sizeof(buf));
        puts(buf);
    }
    close(fd);
    return 0;
}
```

### Windows版最简回射服务器

```
#include <winsock.h>
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
SOCKET Accept(SOCKET listensock) {
    SOCKADDR_IN cli;
    int size = sizeof(cli);
    return accept(listensock, (SOCKADDR*)&cli, &size);
}
int main(int argc, char *argv[]) {
    char buf[128];
    int len;
    SOCKET listensock = ListenAt(atoi(argv[1]));
    for (;;) {
        SOCKET clisock = Accept(listensock);
        while ((len = recv(clisock, buf, sizeof(buf), 0)) > 0)
            send(clisock, buf, len, 0);
        closesocket(clisock);
    }
    closesocket(listensock);
    return 0;
}
```

### Windows版最简回射客户端

```
#include <stdio.h>
#include <winsock.h>
#pragma comment(lib, "ws2_32.lib")
int main(int argc, char *argv[]) {
    char buf[128];
    int ret;
    SOCKADDR_IN ser;
    WSADATA wsaData;
    WSAStartup(MAKEWORD(2, 2), &wsaData);
    SOCKET sock = socket(AF_INET, SOCK_STREAM, 0);
    memset(&ser, 0, sizeof(ser));
    ser.sin_family = AF_INET;
    ser.sin_addr.s_addr = inet_addr(argv[1]);
    ser.sin_port = htons(atoi(argv[2]));
    connect(sock, (SOCKADDR*)&ser, sizeof(ser));
    while ((ret = scanf("%[^\n]%*c", buf)) != -1) {
        if (ret == 0) {
            fgets(buf, sizeof(buf), stdin);
            continue;
        }
        send(sock, buf, strlen(buf), 0);
        memset(buf, 0, sizeof(buf));
        recv(sock, buf, sizeof(buf), 0);
        puts(buf);
    }
    closesocket(sock);
    return 0;
}
```


