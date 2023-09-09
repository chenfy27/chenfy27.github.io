---
layout: post
title: 线程同步之互斥量
category: 平台
keywords: 
---

互斥量可理解成一把锁，不允许多个线程同时访问公共资源，主要用于解决线程同步访问的问题。

### Linux下的互斥量

相关编程接口声明如下：

```c
#include <pthread.h>
int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *attr);
int pthread_mutex_destroy(pthread_mutex_t *mutex);
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
int pthread_mutex_trylock(pthread_mutex_t *mutex);
```

也可以用宏（结构体常量）来初始化互斥锁：pthread_mutex_t mutex = PTHREAD_MUEXT_INITIALIZER。

以下是个例子。

```c
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>
#define N 5
int tot = 0;
pthread_mutex_t mutex;
void* add(void *arg) {
    int i;
    for (i = 0; i < 10000; i++) {
        pthread_mutex_lock(&mutex);
        tot += 1;
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}
int main() {
    int idx;
    pthread_t tid[N];
    pthread_mutex_init(&mutex, NULL);
    for (idx = 0; idx < N; idx++)
        pthread_create(&tid[idx], NULL, add, NULL);
    for (idx = 0; idx < N; idx++)
        pthread_join(tid[idx], NULL);
    pthread_mutex_destroy(&mutex);
    return 0;
}
```

### Windows下的互斥量

相关编程接口声明如下：

```c
#include <windows.h>
HANDLE CreateMutex(LPSECURITY_ATTRIBUTES lpAttr, BOOL bInitialOwner, LPCSTR lpName);
DWORD WaitForSingleObejct(HANDLE handle, DWORD dwMilliseconds);
DWORD WaitForMultipleObjects(DWORD nCount, const HANDLE *lpHandles, BOOL bWaitAll, DWORD dwMilliseconds);
BOOL ReleaseMutex(HANDLE hMutex);
BOOL CloseHandle(HANDLE hObject);
```

以下是Windows版本的示例。

```c
#include <stdio.h>
#include <assert.h>
#include <windows.h>
#include <process.h>
#define N 5
#define MAX 9000000
int tot = 0;
HANDLE hMutex;
unsigned WINAPI Add(void *arg) {
    int *cnt = (int*)arg;
    for (;;) {
        WaitForSingleObject(hMutex, INFINITE);
        if (tot >= MAX) {
            ReleaseMutex(hMutex);
            break;
        }
        *cnt += 1;
        tot += 1;
        ReleaseMutex(hMutex);
    }
    return 0;
}
int main() {
    HANDLE hThread[N];
    int idx, cnt[N] = {0}, sum = 0;
    hMutex = CreateMutex(NULL, FALSE, NULL);
    for (idx = 0; idx < N; idx++)
        hThread[idx] = (HANDLE)_beginthreadex(NULL, 0, Add, &cnt[idx], 0, NULL);
    WaitForMultipleObjects(N, hThread, TRUE, INFINITE);
    CloseHandle(hMutex);
    for (idx = 0; idx < N; idx++) {
        printf("thread %d adds %d\n", idx, cnt[idx]);
        sum += cnt[idx];
    }
    assert(sum == MAX);
    return 0;
}
```
