---
layout: post
title: posix信号量
category: 平台
tag:
---

信号量是一种用于提供不同进程间或者一个给定进程的不同线程间同步手段的原语。

### 基本操作

posix信号量是计数信号量，它提供以下三种基本操作：

- 创建一个信号量，并设置初始值
- 等待一个信号量的值变为大于0，然后将它的值减1
- 给一个信号量的值加1，并唤醒等待该信号量的任意线程

### 有名与无名

posix信号量可以是有名的，也可以是基于内存的，区别如下：

- 有名信号量总是能够在不同进程间共享，而基于内存的信号量则必须在创建时指定是否在进程间共享。
- 有名信号量至少有随内核的持续性，基于内存的信号量则具有随进程的持续性。

在编码上，两者的函数调用也有些区别。

```
        有名信号量                        基于内存信号量
        sem_open                          sem_init
                        sem_wait
                        sem_trywait
                        sem_post
                        sem_get_value
        sem_close                         sem_destroy
        sem_unlink
```

### 对比互斥锁和条件变量

计数值只能取0和1的信号量也称为二值信号量，二值信号量可用于互斥，就像互斥锁一样。

```
初始化信号量为1
sem_wait(&sem);
临界区
sem_post(&sem);
```

对应互斥锁的代码如下：

```
初始化互斥锁
pthread_mutex_lock(&mutex);
临界区
pthread_mutex_unlock(&mutex);
```

互斥锁必须总是由锁住它的线程解锁，而信号量的挂出却不必由执行过它的等待操作的同一线程执行。以生产者-消费者问题为例，生产者伪代码如下：

```
将信号量get初始化为0
将信号量put初始化为1

for (;;) {
    sem_wait(&put);
    把数据放入缓冲区
    sem_post(&get);
}
```

消费者伪代码如下：

```
for (;;) {
    sem_wait(&get);
    处理缓冲区中的数据
    sem_post(&put);
}
```

由于信号量是计数，挂出操作总是能被记住，而当向一个条件变量发送信号时，如果没有线程等待在该条件变量上，那么该信号将丢失。

### 编程接口

- sem_open创建一个新的信号量或者打开一个已存在的信号量。有名信号量总是既可用于线程间同步，又可用于进程间同步。
- sem_close关闭打开着的信号量，但并不会从系统中删除。当进程终止时，它所有打开着的信号量都会自动关闭。
- sem_unlink从系统中删除有名信号量。
- sem_wait测试信号量的值，如大于0，那么将其减1并立即返回；否则调用线程进入睡眠，直到该值变为大于0。sem_trywait在信号量值为0时不进入睡眠，而是返回EAGAIN错误。如果被某个信号中断，sem_wait可能过早地返回，返回的错误为EINTR。
- sem_post将信号量值加1，然后唤醒等待在该信号量的任意线程。
- sem_getvalue获取信号量当前的值。
- 基于内存的信号量由sem_init初始化，若shared为0，那么信号量为同一进程的多个线程共享；否则是在进程间共享的，该信号量必须放到某种类型的共享内存中。

```
#include <semaphore.h>

sem_t* sem_open(const char *name, int oflag);
sem_t* sem_open(const char *name, int oflag, mode_t mode, unsinged int value);
int sem_close(sem_t *sem);
int sem_unlink(const char *name);

int sem_init(sem_t *sem, int shared, unsigned int value);
int sem_destroy(sem_t *sem);

int sem_wait(sem_t *sem);
int sem_trywait(sem_t *sem);
int sem_post(sem_t *sem);
int sem_getvalue(sem_t *sem, int *val);
```
