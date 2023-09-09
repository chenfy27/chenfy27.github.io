---
layout: post
title: fcntl记录锁
category: 平台
keywords:
---

flock()可以对文件进行加锁，但有一些缺陷，比如只能对整个文件加锁，只能放置劝告式锁，很多NFS实现不识别flock()放置的锁等，而fcntl()记录锁弥补了所有这些不足。

使用fcntl()可以在一个文件的任意部分上放置一把锁，这个文件部分可以是一个字节，也可以是整个文件。一般地，fcntl()用来锁住文件中与应用程序定义的记录边界对应的字节范围。

fcntl()接口定义如下：

```c
#include <fcntl.h>
int fcntl(int fd, int cmd, ... /* struct flock *arg */);
struct flock {
    short l_type;       /* F_RDLCK, F_WRLCK, F_UNLCK */
    short l_whence;     /* SEEK_SET, SEEK_CUR, SEEK_END */
    off_t l_start;      /* relative starting offset in bytes */
    off_t l_len;        /* # bytes; 0 means until end-of-file */
    pid_t l_pid;        /* PID returned by F_GETLK */
};
```

用于记录上锁的cmd参数有3个值：F_SETLK, F_SETLKW, F_GETLK，这3个命令要求第三个参数arg是指向flock结构的指针。

下面是常见用法：

```c
int reg_lock(int fd, int cmd, int type, off_t offset, int whence, off_t len) {
    struct flock lock;
    lock.l_type = type;
    lock.l_start = offset;
    lock.l_whence = whence;
    lock.l_len = len;
    return fcntl(fd, cmd, &lock);
}
pid_t test_lock(int fd, int type, off_t offset, int whence, off_t len) {
    struct flock lock;
    lock.l_type = type;
    lock.l_start = offset;
    lock.l_whence = whence;
    lock.l_len = len;
    if (-1 == fcntl(fd, F_GETLK, &lock))
        return -1;
    return F_UNLCK == lock.l_type ? 0 : lock.l_pid;
}
#define read_lock(fd, offset, whence, len)       reg_lock(fd, F_SETLK, F_RDLCK, offset, whence, len)
#define readw_lock(fd, offset, whence, len)      reg_lock(fd, F_SETLKW, F_RDLCK, offset, whence, len)
#define write_lock(fd, offset, whence, len)      reg_lock(fd, F_SETLK, F_WRLCK, offset, whence, len)
#define writew_lock(fd, offset, whence, len)     reg_lock(fd, F_SETLKW, F_WRLCK, offset, whence, len)
#define unlock(fd, offset, whence, len)          reg_lock(fd, F_SETLK, F_UNLCK, offset, whence, len)
#define is_read_locked(fd, offset, whence, len)  test_lock(fd, F_RDLCK, offset, whence, len)
#define is_write_locked(fd, offset, whence, len) test_lock(fd, F_WRLCK, offset, whence, len)
```

关于记录锁的一些注意事项：

- fcntl记录锁既可用于读也可用于写，对于一个文件的任意字节，最多只能存在一种类型的锁（读或写），并且一个给定的字节可以有多个读锁，但只能有一个写锁。
- 当进程关闭记录锁文件的所有描述符，或者进程本身终止时，与该文件关联的所有锁都会被删除。
- 记录锁不能通过fork由子进程继承。
- 记录锁不应与标准IO库函数一起使用，因为库函数会执行内部缓冲，如果需要读写，使用read/write函数。

下面是一个完整的例子。

```c
#include <fcntl.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdarg.h>
#include <sys/wait.h>

#define N 5

int reg_lock(int fd, int cmd, int type, off_t offset, int whence, off_t len)
{
    struct flock lock;
    lock.l_type = type;
    lock.l_start = offset;
    lock.l_whence = whence;
    lock.l_len = len;
    return fcntl(fd, cmd, &lock);
}

#define writew_lock(fd, offset, whence, len) reg_lock(fd, F_SETLKW, F_WRLCK, offset, whence, len)
#define unlock(fd, offset, whence, len) reg_lock(fd, F_SETLK, F_UNLCK, offset, whence, len)

void ulog(const char *fmt, ...)
{
    va_list ap;
    fprintf(stderr, "[%d] ", getpid() % 10000);
    va_start(ap, fmt);
    vfprintf(stderr, fmt, ap);
    va_end(ap);
    fprintf(stderr, "\n");
}

void child_process()
{
    ulog("this is a new child");
    int i, fd, ret;
    fd = open("test.lock", O_WRONLY | O_CREAT, 0644);
    ret = writew_lock(fd, 1, 1, 1);
    if (ret < 0) perror("fcntl");
    for (i = 0; i < 10; i++)
    {
        ulog("i=%d", i);
        usleep(1000);
    }
    unlock(fd, 1, 1, 1);
    close(fd);
    exit(0);
}

int main()
{
    int i;
    pid_t pid[N];
    for (i = 0; i < N; i++)
    {
        pid[i] = fork();
        if (pid[i] == 0)
            child_process();
    }
    for (i = 0; i < N; i++)
    {
        wait(NULL);
        ulog("collected a child");
    }
    return 0;
}
```

