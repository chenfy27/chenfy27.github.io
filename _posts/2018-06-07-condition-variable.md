---
layout: post
title: 线程同步之条件变量
category: 平台
tag:
---

互斥锁用于上锁，以保证任何时刻只有一个线程在临界区内执行。有时当线程获取到互斥锁后，还需要满足特定的条件才能继续执行，那么该线程可以等待在某个条件变量上。条件变量总是有一个互斥锁与之关联。

### 编程接口

条件变量相关的常用编程接口如下：

```c
#include <pthread.h>

int pthread_cond_wait(pthread_cond_t *cptr, pthread_mutex_t *mptr);
int pthread_cond_signal(pthread_cond_t *cptr);
int pthread_cond_broadcast(pthread_cond_t *cptr);

int pthread_mutex_init(pthread_mutex_t *mptr, const pthread_mutexattr_t *attr);
int pthread_mutex_destroy(pthread_mutex_t *mptr);

int pthread_cond_init(pthread_cond_t *cptr, const pthread_condattr_t *attr);
int pthread_cond_destroy(pthread_cond_t *cptr);
```

- 等待线程：在线程调用pthread_cond_wait后，将解锁之前锁住的互斥锁，并进入睡眠。在以后某个时刻其他线程通过操作条件变量将其唤醒，再次锁住该互斥锁。
- 触发线程：在条件满足后，既可以调用pthread_cond_signal只唤醒一个线程，也可以调用pthread_cond_broadcast唤醒所有等待在对应条件变量上的所有线程。

给条件变量发送信号的代码大致如下：

```c
struct {
    pthread_mutex_t mutex;
    pthread_cond_t cond;
    其他变量
} var = {PTHREAD_MUTEX_INITIALIZER, PTHREAD_COND_INITIALIZER, ...};

pthread_mutex_lock(&var.mutex);
设置条件为真
pthread_cond_signal(&var.cond);
pthread_mutex_unlock(&var.mutex);
```

测试条件并进入睡眠以等待该条件为真的代码大致如下：

```c
pthread_mutex_lock(&var.mutex);
while (条件为假)
    pthread_cond_wait(&var.cond, &var.mutex);
pthread_mutex_unlock(&var.mutex);
```

### 生产者消费者问题

为简化问题，假设有一个大小为N的数组，开M个线程当作生产者，把第i个元素的值设为i，另外再开1个线程作为消费者，验证第i个元素的值是否为i。要求生产者和消费者同时运行。

首先，多个生产者同时访问公共资源，生产者之间需要同步，这里用互斥锁就行了。另一方面，只有当元素被生产者设置后消费者才能进行验证，因此消费者和生产者之间也需要同步，这种等待同步与生产者之间的竞争同步有所区别，用条件变量来实现。

```c
#include <stdio.h>
#include <pthread.h>

#define N 1000000
#define M 5

int buf[N], cnt[M];
struct {
    pthread_mutex_t mutex;
    int idx;
} put = {PTHREAD_MUTEX_INITIALIZER};

struct {
    pthread_mutex_t mutex;
    pthread_cond_t cond;
    int nready;
} ready = {PTHREAD_MUTEX_INITIALIZER, PTHREAD_COND_INITIALIZER};

void* produce(void *arg) {
    for (;;) {
        pthread_mutex_lock(&put.mutex);
        if (put.idx >= N) {
            pthread_mutex_unlock(&put.mutex);
            return NULL;
        }
        buf[put.idx] = put.idx;
        put.idx += 1;
        pthread_mutex_unlock(&put.mutex);

        pthread_mutex_lock(&ready.mutex);
        if (ready.nready == 0)
            pthread_cond_signal(&ready.cond);
        ready.nready += 1;
        pthread_mutex_unlock(&ready.mutex);

        *((int*)arg) += 1;
    }
}

void* consume(void *arg) {
    int i;
    for (i = 0; i < N; i++) {
        pthread_mutex_lock(&ready.mutex);
        while (ready.nready == 0)
            pthread_cond_wait(&ready.cond, &ready.mutex);
        ready.nready -= 1;
        pthread_mutex_unlock(&ready.mutex);

        if (buf[i] != i)
            printf("buf[%d]=%d\n", i, buf[i]);
    }
    return NULL;
}

int main() {
    int i;
    pthread_t tid_produce[M], tid_consume;

    pthread_setconcurrency(M+1);
    for (i = 0; i < M; i++)
        pthread_create(&tid_produce[i], NULL, produce, &cnt[i]);
    pthread_create(&tid_consume, NULL, consume, NULL);

    for (i = 0; i < M; i++) {
        pthread_join(tid_produce[i], NULL);
        printf("cnt[%d]=%d\n", i, cnt[i]);
    }
    pthread_join(tid_consume, NULL);

    return 0;
}
```
