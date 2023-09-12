---
layout: post
title: udp socket编程基础
keywords:
category: 网络
---

在4层TCP/IP模型中，传输层分为TCP和UDP两种方式，与TCP相比，UDP具有以下特点：

- 提供不可靠的数据传输服务。UDP在结构上更简洁，不会发送类似ACK的应答消息，也不会像SEQ那样给数据包分配序号，因而性能一般比TCP更高。
- 不会进行流控制。
- 无需经过连接过程，UDP中只有创建套接字和数据交换过程。
- 服务端和客户端均只需要1个套接字，即只需1个UDP套接字就能和多台主机通信。
- 存在数据边界，因此传输中调用IO函数的次数非常重要，输入函数的调用次数要和输出函数的调用次数完全一致，才能保证接收全部已发送数据。

### 基于UDP的IO函数

与TCP套接字不同，UDP套接字不会保持连接状态，因此每次传输数据都要添加目标地址信息。

```
#include <sys/socket.h>
ssize_t sendto(int sock, void *buf, size_t nbytes, int flags, struct sockaddr *to, socklen_t addrlen);
ssize_t recvfrom(int sock, void *buf, size_t nbytes, int flags, struct sockaddr *from, socklen_t *addrlen);
```

UDP在发送消息时，需要给出接收端的地址信息；而在接收消息时，可得到本消息的发送端地址信息。由于不存在请求边接和受理过程，UDP在某种意义上无法明确区分服务端和客户端，一般用发送端和接收端加以区分。

### Linux版本基于UDP的回射程序

发送端代码

```
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <sys/socket.h>
#include <arpa/inet.h>
int main(int argc, char *argv[]) {
    char buf[512];
    socklen_t szaddr;
    struct sockaddr_in dest, peer;

    memset(&dest, 0, sizeof(dest));
    dest.sin_family = AF_INET;
    dest.sin_addr.s_addr = inet_addr(argv[1]);
    dest.sin_port = htons(atoi(argv[2]));
    int fd = socket(AF_INET, SOCK_DGRAM, 0);

    while (fgets(buf, sizeof(buf), stdin)) {
        sendto(fd, buf, strlen(buf), 0, (struct sockaddr*)&dest, sizeof(dest));
        szaddr = sizeof(peer);
        int len = recvfrom(fd, buf, sizeof(buf), 0, (struct sockaddr*)&peer, &szaddr);
        fwrite(buf, len, 1, stdout);
    }
    close(fd);
    return 0;
}
```

接收端代码

```
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <sys/socket.h>
#include <arpa/inet.h>
int main(int argc, char *argv[]) {
    socklen_t szaddr;
    int len, fd;
    char buf[512];
    struct sockaddr_in local, peer;

    memset(&local, 0, sizeof(local));
    local.sin_family = AF_INET;
    local.sin_addr.s_addr = htonl(INADDR_ANY);
    local.sin_port = htons(atoi(argv[1]));

    fd = socket(AF_INET, SOCK_DGRAM, 0);
    if (-1 == bind(fd, (struct sockaddr*)&local, sizeof(local)))
        perror("bind");

    for (;;) {
        szaddr = sizeof(peer);
        len = recvfrom(fd, buf, sizeof(buf), 0, (struct sockaddr*)&peer, &szaddr);
        fwrite(buf, len, 1, stdout);
        sendto(fd, buf, len, 0, (struct sockaddr*)&peer, szaddr);
    }
    close(fd);
    return 0;
}
```

程序启动顺序不重要，只要保证在调用sendto函数之前目标主机的程序已经运行即可。

### Windows版本基于UDP的回射程序

发送端代码

```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <winsock2.h>
#pragma comment(lib, "ws2_32.lib")
int main(int argc, char *argv[]) {
    WSADATA wsaData;
    SOCKET sock;
    char buf[512];
    SOCKADDR_IN dest, peer;

    memset(&dest, 0, sizeof(dest));
    dest.sin_family = AF_INET;
    dest.sin_addr.s_addr = inet_addr(argv[1]);
    dest.sin_port = htons(atoi(argv[2]));

    WSAStartup(MAKEWORD(2, 2), &wsaData);
    sock = socket(AF_INET, SOCK_DGRAM, 0);

    while (fgets(buf, sizeof(buf), stdin)) {
        sendto(sock, buf, strlen(buf), 0, (struct sockaddr*)&dest, sizeof(dest));
        int szaddr = sizeof(peer);
        int len = recvfrom(sock, buf, sizeof(buf), 0, (struct sockaddr*)&peer, &szaddr);
        fwrite(buf, len, 1, stdout);
    }
    closesocket(sock);
    WSACleanup();
    return 0;
}
```

接收端代码

```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <winsock2.h>
#pragma comment(lib, "ws2_32.lib")
int main(int argc, char *argv[]) {
    WSADATA wsaData;
    SOCKET sock;
    char buf[512];
    SOCKADDR_IN local, peer;

    memset(&local, 0, sizeof(local));
    local.sin_family = AF_INET;
    local.sin_addr.s_addr = htonl(INADDR_ANY);
    local.sin_port = htons(atoi(argv[1]));

    WSAStartup(MAKEWORD(2, 2), &wsaData);
    sock = socket(AF_INET, SOCK_DGRAM, 0);
    bind(sock, (SOCKADDR*)&local, sizeof(local));

    for (;;) {
        int szaddr = sizeof(peer);
        int len = recvfrom(sock, buf, sizeof(buf), 0, (struct sockaddr*)&peer, &szaddr);
        fwrite(buf, len, 1, stdout);
        sendto(sock, buf, len, 0, (struct sockaddr*)&peer, sizeof(peer));
    }
    closesocket(sock);
    WSACleanup();
    return 0;
}
```

### UDP地址分配

对于TCP套接字，客户端在调用connect函数过程中自动完成IP和端口分配给套接字的过程。在UDP中，调用sendto函数传输数据之前应完成套接字的地址分配，如调用sento时发现尚未分配，则在首次调用sendto时自动分配IP和端口，该地址一直保留到程序结束为止。

换句话说，调用sendto函数时将自动分配IP和端口，因此，UDP发送端通常无需显示做地址分配。

### 已连接UDP套接字

调用sendto函数传输数据过程大致分为以下3个阶段：

- 向UDP套接字注册目标IP和端口；
- 传输数据；
- 删除UDP套接字中注册的目标地址信息；

每次调用sendto都重复上述过程，因此可重复利用同一UDP套接字向不同目标传输数据，这种未注册目标地址信息的套接字称为未连接套接字。反之，调用了connect函数注册目标地址的套接字称为已连接套接字。默认情况下，UDP套接字为未连接套接字。对UDP套接字调用connect函数并不是要与对方UDP套接字连接，而是向UDP套接字注册目标IP和端口信息。

如要与同一主机进行长时间通信，将UDP套接字设成已连接方式将会提高效率。

另外，对于已连接套接字，除sendto/recvfrom外，还可以用read/write函数进行通信。
