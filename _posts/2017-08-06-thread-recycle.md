---
layout: post
title: 线程回收
category: 平台
keywords:
---

如果进程中的任意线程调用了exit/_exit/_Exit，那么整个进程就会终止。类似的，如果默认动作是终止进程，那么发送到线程的信号就会终止整个进程。

线程可以通过3种方式退出，即在不终止整个进程的情况下，停止它的控制流。

- 从线程函数中返回，返回值就是线程的退出码。
- 被同一进程中的其他线程取消。
- 线程调用pthread_exit退出。

相关接口如下：

```c
#include <pthread.h>
void pthread_exit(void *retval);
int pthread_join(pthread_t tid, void **retval);
```

其中，retval是一个无类型指针，与线程函数参数相似，进程中的其他线程可通过pthread_join函数访问到这个指针；调用pthread_join将使线程阻塞，直到指定的线程返回或者被取消，如果对退出状态不感兴趣，可将retval设为NULL。

与子进程需要回收类似，线程也要回收，否则将一直占用资源，直到进程退出。

回收线程一般有两种方式，一种是某个线程通过调用pthread_join函数等待另一个线程结束，并可获得线程退出状态；另一种是不关心执行结果，设置为自动回收，子线程退出后立即回收掉，其他线程不能调pthread_join去获取执行结果。

### 等待线程

通过调用pthread_join等待指定的线程。

```c
pthread_t tid;
pthread_create(&tid, NULL, ThreadFunc, NULL);
pthread_join(tid);
```

### 分离线程

有3种方法，效果是一致的。

- 在创建线程中调用pthread_detach(tid)。
- 在子线程中调用pthread_detach(pthread_self())。
- 在创建线程前设置线程分离属性。

```c
void* ThreadFunc(void *arg)
{
    return NULL;
}

pthread_t tid;
pthread_create(&tid, NULL, ThreadFunc, NULL);
pthread_detach(tid);
```

```c
void* ThreadFunc(void *arg)
{
    pthread_detach(pthread_self());
    return NULL;
}

pthread_t tid;
pthread_create(&tid, NULL, ThreadFunc, NULL);
```

```c
void* ThreadFunc(void *arg)
{
    return NULL;
}

pthread_t tid;
pthread_attr_t attr;
pthread_attr_init(&attr);
pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);
pthread_create(&tid, &attr, ThreadFunc, NULL);
```
