---
layout: post
title: 调试打印函数
category: 平台
keywords:
---

有时程序出了原因不明的问题，缺乏必要的运行日志供分析，而环境又没提供调试工具来跟踪，要想知道更多的程序运行时细节，简单粗暴的方法就是将感兴趣的内容都打印出来，该做法的缺点是得重新编译文件做替换。

对于简单的console程序，可以直接将内容输出到终端，对于比较复杂的程序，建议输出到磁盘文件中。

```c
static void ulog(const char *fmt, ...) {
    FILE *fp = fopen("ulog.txt", "a");
    va_list ap;
    va_start(ap, fmt);
    vfprintf(fp, fmt, ap);
    va_end(ap);
    fprintf(fp, "\n");
    fclose(fp);
}
```

上述接口已能满足大部分调试要求，但对于性能调优，需要增加时间项；对于多进程多线程，可考虑加上进程ID或线程ID。

```c
static void ulog(const char *fmt, ...)
{
    struct timeval tv;
    gettimeofday(&tv, NULL);
    struct tm *tm = localtime(&tv.tv_sec);

    FILE *fp = fopen("ulog.txt", "a");
    fprintf(fp, "%02d:%02d:%02d.%03d ",
        tm->tm_hour, tm->tm_min, tm->tm_sec, tv.tv_usec / 1000);
    va_list ap;
    va_start(ap, fmt);
    vfprintf(fp, fmt, ap);
    va_end(ap);
    fprintf(fp, "\n");
    fclose(fp);
}
```

对于简单使用场景，也可以通过宏来实现。

```c
#define ulog(fmt, ...) {\
    FILE *fp = fopen("ulog.txt", "a");\
    fprintf(fp, "[%04d] %s:%d (%s) $ "fmt"\n", getpid() % 10000,\
        __FILE__, __LINE__, __func__, ##__VA_ARGS__);\
    fclose(fp);\
}
```
