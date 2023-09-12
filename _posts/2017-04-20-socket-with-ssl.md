---
layout: post
title: ssl安全通信
category: 网络
keywords:
---

通过openssl可以自行签发证书，有了证书和私钥后就可以进行SSL安全通信了。

调用SSL提供的接口流程如下图所示：

![]({{"/pic/ssl_socket.png" | prepend: site.baseurl}})

### 服务端

```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <openssl/ssl.h>
#include <openssl/err.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>

#define CERT_FILE  "./cert/server.crt"
#define KEY_FILE   "./cert/server.key"

int main(int argc, char *argv[]) {
    SSL_CTX *ctx;
    int listenfd, newfd;
    struct sockaddr_in ser, cli;

    SSL_library_init();
    OpenSSL_add_all_algorithms();
    SSL_load_error_strings();
    if (NULL == (ctx = SSL_CTX_new(SSLv23_server_method()))) {
        ERR_print_errors_fp(stderr);
        exit(-1);
    }
    if (SSL_CTX_use_certificate_file(ctx, CERT_FILE, SSL_FILETYPE_PEM) != 1) {
        ERR_print_errors_fp(stderr);
        exit(-1);
    }
    if (SSL_CTX_use_PrivateKey_file(ctx, KEY_FILE, SSL_FILETYPE_PEM) != 1) {
        ERR_print_errors_fp(stderr);
        exit(-1);
    }
    if (1 != SSL_CTX_check_private_key(ctx)) {
        ERR_print_errors_fp(stderr);
        exit(-1);
    }

    listenfd = socket(AF_INET, SOCK_STREAM, 0);
    memset(&ser, 0, sizeof(ser));
    ser.sin_family = AF_INET;
    ser.sin_addr.s_addr = htonl(INADDR_ANY);
    ser.sin_port = htons(atoi(argv[1]));

    if (-1 == bind(listenfd, (struct sockaddr*)&ser, sizeof(struct sockaddr))) {
        perror("bind");
        exit(-1);
    }
    if (-1 == listen(listenfd, 5)) {
        perror("listen");
        exit(-1);
    }

    for (;;) {
        socklen_t size = sizeof(struct sockaddr);
        if (-1 == (newfd = accept(listenfd, (struct sockaddr*)&cli, &size))) {
            perror("accept");
            break;
        }
        SSL *ssl = SSL_new(ctx);
        SSL_set_fd(ssl, newfd);
        if (-1 == SSL_accept(ssl)) {
            ERR_print_errors_fp(stderr);
            close(newfd);
            break;
        }
        {
            char buf[256]; int len;
            while ((len = SSL_read(ssl, buf, sizeof(buf))) > 0) {
                buf[len] = 0;
                printf("%s", buf);
                SSL_write(ssl, buf, len);
            }
            SSL_shutdown(ssl);
            SSL_free(ssl);
            close(newfd);
        }
    }
    SSL_CTX_free(ctx);
    close(listenfd);
    return 0;
}
```

### 客户端

```
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <openssl/ssl.h>
#include <openssl/err.h>
int main(int argc, char *argv[]) {
    SSL_CTX *ctx;
    int sockfd;
    struct sockaddr_in ser;

    SSL_library_init();
    OpenSSL_add_all_algorithms();
    SSL_load_error_strings();
    if (NULL == (ctx = SSL_CTX_new(SSLv23_client_method()))) {
        ERR_print_errors_fp(stderr);
        exit(-1);
    }

    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    memset(&ser, 0, sizeof(ser));
    ser.sin_family = AF_INET;
    ser.sin_addr.s_addr = inet_addr(argv[1]);
    ser.sin_port = htons(atoi(argv[2]));
    if (-1 == connect(sockfd, (struct sockaddr*)&ser, sizeof(ser))) {
        perror("connect");
        exit(-1);
    }
    SSL *ssl = SSL_new(ctx);
    SSL_set_fd(ssl, sockfd);
    if (-1 == SSL_connect(ssl)) {
        ERR_print_errors_fp(stderr);
        exit(-1);
    }
    {
        char buf[256]; int len;
        freopen("/etc/passwd", "r", stdin);
        while (fgets(buf, sizeof(buf), stdin)) {
            len = strlen(buf);
            if (SSL_write(ssl, buf, len) > 0) {
                buf[0] = 0;
                len = SSL_read(ssl, buf, sizeof(buf));
                if (len > 0) {
                    buf[len] = 0;
                    printf("%s", buf);
                }
            }
        }
        SSL_shutdown(ssl);
        SSL_free(ssl);
    }
    close(sockfd);
    SSL_CTX_free(ctx);
    return 0;
}
```
