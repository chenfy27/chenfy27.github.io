---
layout: post
title: 单实例运行
category: 平台
keywords:
---

有时候为避免冲突，会限制同一时刻最多只有一个进程副本在运行，一般通过记录锁机制来实现。具体做法是每个进程启动时打开一个固定名字的文件，并在该文件的整体上加一把写锁，在此之后创建写锁的尝试都将失败，向后续进程副本指明已经有一个副本在运行。当进程副本终止时，锁将被自动清除。

```c
#include <fcntl.h>
#include <string.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/stat.h>

#define LOCKFILE "/var/run/test.pid"

int IsRunning() {
    int fd;
    char buf[16];
    struct flock lock;

    fd = open(LOCKFILE, O_RDWR | O_CREAT, 0644);
    if (fd < 0) {
        perror("open");
        exit(1);
    }

    lock.l_type = F_WRLCK;
    lock.l_whence = F_WRLCK;
    lock.l_start = 0;
    lock.l_len = 0;
    if (fcntl(fd, F_SETLK, &lock) < 0) {
        if (errno == EACCES || errno == EAGAIN) {
            close(fd);
            return 1;
        }
        perror("fcntl");
        exit(1);
    }

    ftruncate(fd, 0);
    sprintf(buf, "%ld", (long)getpid());
    write(fd, buf, strlen(buf));
    return 0;
}

int main() {
    if (IsRunning()) {
        puts("another copy is running");
        return 0;
    } else for (;;sleep(1));
    return 0;
}
```
