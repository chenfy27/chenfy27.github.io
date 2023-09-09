---
layout: post
title: 线程同步之临界区
category: 平台
keywords: 
---

Windows下有多种线程同步方法，出于安全考虑，程序运行方式划分为用户模式和内核模式，用户模式同步方法有临界区，内核模式同步方法有互斥量、信号量、事件等，用户模式同步速度快，而内核模式同步能提供更多的功能，并且可以指定超时时间，防止产生死锁。

下面介绍临界区的用法，相关API如下：

```c
#include <windows.h>
void InitializeCriticalSection(LPCRITICAL_SECTION lpCriticalSection);
void EnterCriticalSection(LPCRITICAL_SECTION lpCriticalSection);
void LeaveCriticalSection(LPCRITICAL_SECTION lpCriticalSection);
void DeleteCriticalSection(LPCRITICAL_SECTION lpCriticalSection);
```

以下是个简单的例子，经测试，对于同样的数据范围，临界区比互斥量快很多。

```c
#include <windows.h>
#include <process.h>
#include <stdio.h>
#define N 5
CRITICAL_SECTION gCriticalSection;
int tot = 0;
unsigned WINAPI Add(void *arg) {
    int i;
    for (i = 0; i < 10000; i++) {
        EnterCriticalSection(&gCriticalSection);
        tot += 1;
        LeaveCriticalSection(&gCriticalSection);
    }
    return 0;
}
int main() {
    int idx;
    HANDLE hThread[N];
    InitializeCriticalSection(&gCriticalSection);
    for (idx = 0; idx < N; idx++)
        hThread[idx] = (HANDLE)_beginthreadex(NULL, 0, Add, NULL, 0, NULL);
    WaitForMultipleObjects(N, hThread, TRUE, INFINITE);
    DeleteCriticalSection(&gCriticalSection);
    return 0;
}
```
