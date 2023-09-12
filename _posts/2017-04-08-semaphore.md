---
layout: post
title: 线程同步之信号量
category: 平台
keywords:
---

信号量是一个由内核维护的计数器，其值永远不会小于0，通常用于进程之间或同一进程不同线程之间做同步。

信号量初始值代表共享资源的个数，进程或线程减小一个信号量可预约对共享资源的独占访问，在使用完之后增加一个信号量来释放共享资源，便于让其他进程或线程访问。

二值信号量是最简单的信号量，它只有0和1两种取值。

posix信号量有命名和匿名两种，匿名信号量编程接口如下：

```
#include <semaphore.h>
int sem_init(sem_t *sem, int pshared, unsigned int value);
int sem_wait(sem_t *sem);
int sem_post(sem_t *sem);
int sem_destroy(sem_t *sem);
```

命名信号量的创建与销毁方式不同，编程接口如下：

```
#include <fcntl.h>
#include <sys/stat.h>
#include <semaphore.h>
sem_t* sem_open(const char *name, int oflag, mode_t mode, unsigned int value);
int sem_wait(sem_t *sem);
int sem_post(sem_t *sem);
int sem_close(sem_t *sem);
int sem_unlink(const char *name);
```

注意：MAC OS系统不支持匿名信号量，只能用命名信号量。

下面的例子用信号量控制线程对共享资源的独占访问，功能与互斥锁类似。

```
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>
#define N 5
int tot = 0;
sem_t sem;
void* add(void *arg) {
    int i;
    for (i = 0; i < 10000; i++) {
        sem_wait(&sem);
        tot += 1;
        sem_post(&sem);
    }
    return NULL;
}
int main() {
    int idx;
    pthread_t tid[N];
    sem_init(&sem, 0, 1);
    for (idx = 0; idx < N; idx++)
        pthread_create(&tid[idx], NULL, add, NULL);
    for (idx = 0; idx < N; idx++)
        pthread_join(tid[idx], NULL);
    sem_destroy(&sem);
    return 0;
}
```
