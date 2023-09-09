---
layout: post
title: 捕捉段错误
category: 平台
keywords:
---

段错误是一种很常见的错误，原因很多，比如越界读或写内存，如果没有详细的运行日志供分析，一般不好定位问题出在哪里。

下面介绍通过捕捉段错误信号SIGSEGV，并输出发生段错误时的函数调用堆栈信息，协助查找段错误出现的位置，在不能产生coredump文件的环境下这是一种替换方案。

```c
#include <signal.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <execinfo.h>
void OnSigSEGV(int signo, siginfo_t *info, void *act) {
    void *buffer[100];
    int cnt = backtrace(buffer, sizeof(buffer) / sizeof(buffer[0]));
    backtrace_symbols_fd(buffer, cnt, 1);
    abort();
}
void TrapFault() {
    struct sigaction act;
    sigemptyset(&act.sa_mask);
    act.sa_flags = SA_SIGINFO;
    act.sa_sigaction = OnSigSEGV;
    sigaction(SIGSEGV, &act, NULL);
}
void func2() {
    memset(NULL, 0, 123);
}
void func1() {
    func2();
}
void func() {
    func1();
}
int main() {
    TrapFault();
    func();
    return 0;
}
```

在macbook上的某次执行结果如下：

```
$ ./x
0   x                                   0x000000010d562e7c OnSigSEGV + 44
1   libsystem_platform.dylib            0x00007fffb03c9bba _sigtramp + 26
2   libSystem.B.dylib                   0x00007fffbe377796 __crashreporter_info__ + 90109150
3   libsystem_c.dylib                   0x00007fffb0276f7f __memset_chk + 22
4   x                                   0x000000010d562f01 func2 + 33
5   x                                   0x000000010d562f19 func1 + 9
6   x                                   0x000000010d562f29 func + 9
7   x                                   0x000000010d562f49 main + 25
8   libdyld.dylib                       0x00007fffb01bc255 start + 1
Abort trap: 6
```

从结果很容易看出，问题出在func2函数的memset操作上。

同样的代码在Centos7上执行，某次结果如下：

```
$ ./x
./x[0x4006f3]
/lib64/libc.so.6(+0x35250)[0x7f85ee704250]
/lib64/libc.so.6(+0x89754)[0x7f85ee758754]
./x[0x400781]
./x[0x40079b]
./x[0x4007ab]
./x[0x4007c5]
/lib64/libc.so.6(__libc_start_main+0xf5)[0x7f85ee6f0b35]
./x[0x4005f9]
已放弃
```

此时需要addr2line工具将地址做下转换，方便查看。

```
$ addr2line 0x4007c5 0x4007ab 0x40079b 0x400781 -sCf -e x
main
a.c:32
func
a.c:28
func1
a.c:25
func2
a.c:22
```

可见，在func2中调用了libc.so.6中的某个库函数后出错了。

有时，为了保证服务不挂掉，通过父子进程的方式来工作，子进程真正干活，而父进程则监视子进程状态，子进程异常退出后由父进程再次产生子进程。如果子进程的业务逻辑比较复杂，可能会出现段错误，可以考虑通过信号来捕捉。

以下是个简单的例子。

```c
#include <unistd.h>
#include <sys/wait.h>
#include <stdio.h>
#include <string.h>
#include <signal.h>
#include <stdlib.h>
void CatchSEGV(int signo) {
    fprintf(stderr, "!!! segment fault!!!\n");
    exit(-1);
}
void ChildProcess() {
    signal(SIGSEGV, CatchSEGV);
    char *p = NULL;
    strcpy(p, "hello");
    exit(0);
}
int main() {
    pid_t pid = fork();
    if (pid == 0) {
        ChildProcess();
    } else if (pid > 0) {
        wait(NULL);
    }
    return 0;
}
```
