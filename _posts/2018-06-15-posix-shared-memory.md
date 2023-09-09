---
layout: post
title: posix共享内存
category: 平台
tag:
---

共享内存是所有可用IPC形式中速度最快的，因为一旦内存区映射到进程的地址空间，进程间数据的传递就不再涉及内核，但是往共享内存中读写一般需要某种形式的同步，比如互斥锁、条件变量、记录锁、信号量等。

### 编程接口

posix提供了两种在无亲缘关系进程间共享内存区的方法：

- 内存映射文件：由open函数打开，再由mmap函数把得到的描述符映射到当前进程地址空间中的一个文件。
- 共享内存区对象：由shm_open打开一个IPC名字，所返回的描述符由mmap函数映射到当前进程的地址空间。

```c
int shm_open(const char *name, int oflag, mode_t mode);
int shm_unlink(const char *name);
```

说明：

- oflag必须含有O_RDONLY或者O_RDWR，还可以指定O_CREAT, O_EXEC或者O_TRUNC。
- shm_open中的mode参数必须指定，如果没指定O_CREAT标志，置为0即可。

无论哪种方式，都需要用到mmap函数，它的原型如下：

```c
void* mmap(void *addr, size_t len, int prot, int flags, int fd, off_t offset);
```

其中：

- addr指定起始地址，通常设为NULL，由内核自行选择。函数返回值是共享内存的起始地址。
- len为映射到地址空间的字节数，它从被映射文件开头起第offset个字节处开始计算。offset通常设置为0。
- 内存区的保扩由prot指定，常见值有PROT_READ和PROT_WRITE。
- flags一般为MAP_SHARED或者MAP_PRIVATE，表示对其他进程是否可见。

mmap成功返回后，fd可以关闭，该操作对于由mmap建立的映射关系没有影响。

在处理mmap时，普通文件或者共享内存区对象的大小都可以通过ftruncate函数进行修改。

```c
int ftruncate(int fd, off_t length);
```

从进程的地址空间删除映射关系，使用munmap函数。

```c
int munmap(void *addr, size_t len);
```

### 示例

假设有两个进程同时执行将计数器不断加1的操作，计数器放在共享内存中，通过信号量来同步。为简单起见，这里使用父子进程，无亲缘关系的进程同样适用。

以下示例代码用的普通文件做内存映射，信号量用的是有名的，因而不需要放到共享内存中就能用于进程间同步。

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <semaphore.h>
#include <fcntl.h>
#include <sys/mman.h>
#include <sys/wait.h>
#define N 100
int main() {
    unlink("shm1");
    int fd = open("shm1", O_RDWR | O_CREAT, 0644);
    int *cnt = mmap(NULL, sizeof(int), PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    ftruncate(fd, sizeof(int));
    close(fd);

    sem_unlink("sem1");
    sem_t *sem = sem_open("sem1", O_RDWR | O_CREAT, 0644, 1);
    if (sem == SEM_FAILED) {
        perror("sem_open");
    }

    setbuf(stdout, NULL);
    int i;
    if (fork() == 0) {
        for (i = 0; i < N; i++) {
            sem_wait(sem);
            *cnt += 1;
            printf("child: %d\n", *cnt);
            sem_post(sem);
        }
        exit(0);
    }
    for (i = 0; i < N; i++) {
        sem_wait(sem);
        *cnt += 1;
        printf("parent: %d\n", *cnt);
        sem_post(sem);
    }
    wait(NULL);
    return 0;
}
```

同样的功能，下面代码用共享内存区对象来实现，并把信号量放到共享内存中。

```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>
#include <semaphore.h>
#include <sys/wait.h>
#include <sys/mman.h>
#define N 100
struct Node {
    int cnt;
    sem_t sem;
}*p;
int main() {
    shm_unlink("shm2");
    int fd = shm_open("shm2", O_RDWR | O_CREAT, 0644);
    ftruncate(fd, sizeof(struct Node));
    p = mmap(NULL, sizeof(struct Node), PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    close(fd);

    setbuf(stdout, NULL);
    sem_init(&p->sem, 1, 1);

    int i;
    if (fork() == 0) {
        for (i = 0; i < N; i++) {
            sem_wait(&p->sem);
            p->cnt += 1;
            printf("child: %d\n", p->cnt);
            sem_post(&p->sem);
        }
        exit(0);
    }
    for (i = 0; i < N; i++) {
        sem_wait(&p->sem);
        p->cnt += 1;
        printf("parent: %d\n", p->cnt);
        sem_post(&p->sem);
    }
    wait(NULL);
    return 0;
}
```

### 特殊文件

有名的信号量可直接在不同进程之间做同步，而匿名信号量若想用于进程间同步则需要放在共享内存中。从上述编程接口来看，无论是用open还是shm_open来创建共享内存，都需要在文件系统中创建一个文件。在某些实现上，可以简化该操作。

4.4BSD提供匿名内存映射，彻底避免了文件的创建和打开，其办法是将mmap的flags参数设成MAP_SHARED \| MAP_ANON，把fd参数置为-1。这样的内存区自动被初始化为0。

```c
p = mmap(NULL, sizeof(int), PROT_READ | PROT_WRITE, MAP_SHARED | MAP_ANON, -1, 0);
```

而SVR4则可以用/dev/zero进行映射。

```c
int fd = open("/dev/zero", O_RDWR);
int *p = mmap(NULL, sizeof(int), PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
close(fd);
```
