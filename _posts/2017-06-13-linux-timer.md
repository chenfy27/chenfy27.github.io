---
layout: post
title: linux下的定时器
category: 平台
keywords:
---

在开发过程中，有时需要定时，对于简单的需求，可使用sleep或usleep让进程休眠，前者时间单位为秒，后者为微秒。有时不希望进程被阻塞，而是正常执行，在到了指定的时间后再去执行某些操作，这时可考虑使用定时器，Linux下常用的定时器有alarm和setitimer。

### alarm函数

alarm也称为闹钟函数，可在进程中设置一个定时器，当时间到达时便给进程发送SIGALRM信号，函数原型如下：

```
unsigned int alarm(unsigned int seconds);
```

- 参数：seconds为新定时器倒计时的秒数，传0表示取消之前的定时器。
- 返回值：失败返回-1，成功时如果之前已设置过定时器，则返回剩余秒数，否则返回0。

下面是个简单的例子。

```
#include <stdio.h>
#include <unistd.h>
#include <signal.h>
void func(int signo) {
    puts("receive SIGALRM");
}
int main() {
    signal(SIGALRM, func);
    alarm(3);
    pause();
    return 0;
}
```

### setitimer函数

alarm只能精确到秒，精度也许不能满足要求，这时可以考虑setitimer函数，它有两个功能，一是指定一段时间后发出信号，二是每隔一段时间发出信号。函数原型如下：

```
struct timeval {
    time_t tv_sec;
    suseconds_t tv_usec;
};
struct itimerval {
    struct timeval it_interval;
    struct timeval it_value;
};
int setitimer(int which, const struct itimerval *newval, struct itimerval *oldval);
```

其中it_value表示离下次执行剩余时间，定时到达后将it_interval赋给it_value，自动开始下一次定时。

Linux为每个任务安排了3个内部定时器：

- ITIMER_REAL: 实时定时器，无论进程在何种模式下运行（甚至被挂起），它总在计时，时间到了便向进程发SIGALRM信号。
- ITIMER_VIRTUAL: 只当进程在用户模式时才计时，时间到了向进程发送SIGVTALRM信号。
- ITIMER_PROF: 进程在用户模式和内核模式均计时，时间到了向进程发送SIGPROF信号。

以下是个示例，每秒输出一行文字：

```
#include <stdio.h>
#include <signal.h>
#include <unistd.h>
#include <sys/time.h>
void func(int signo) {
    puts("timing");
}
int main() {
    struct itimerval val;
    val.it_value.tv_sec = 1;
    val.it_value.tv_usec = 0;
    val.it_interval = val.it_value;
    signal(SIGPROF, func);
    setitimer(ITIMER_PROF, &val, NULL);
    for (;;);
    return 0;
}
```
