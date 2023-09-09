---
layout: post
title: readn与writen
category: 网络
keywords:
---

流式套接字上read/write函数的行为与普通文件IO有所区别，表现在read输入或write输出的字节数可能会比请求数要少，但这并不是出错的状态，其原因在于内核中用于套接字的缓冲区可能已经达到上限，此时需要调用者再次调用read/write函数，以输入或输出剩余字节。

有些版本的Unix实现在往一个管道中写多于4096字节数据时会表现出这样的行为，这个现象在read一个套接字时很常见，但在write一个字节流套接字时只能在该套接字为非阻塞时才会出现。考虑到这些因素，为保险起见，通常封装接口readn/writen来解决这个问题。

```c
ssize_t readn(int fd, void *buf, size_t nbytes)
{
    size_t nread, nleft = nbytes;
    char *ptr = (char*)buf;
    while (nleft > 0) {
        if (-1 == (nread = read(fd, ptr, nleft))) {
            if (EINTR == errno) nread = 0;
            else return -1;
        } else if (0 == nread) // EOF
            break;
        nleft -= nread;
        ptr += nread;
    }
    return nbytes - nleft;
}
ssize_t writen(int fd, const void *buf, size_t nbytes)
{
    size_t nwrite, nleft = nbytes;
    const char *ptr = (const char*)buf;
    while (nleft > 0) {
        if (-1 == (nwrite = write(fd, ptr, nleft))) {
            if (EINTR == errno) nwrite = 0;
            else return -1;
        }
        nleft -= nwrite;
        ptr += nwrite;
    }
    return nbytes;
}
```
