---
layout: post
title: 循环队列
category: 算法
keywords:
---

循环队列在普通队列的基础上，将队头与队尾相连，解决了队列假满的问题。

```
typedef struct {
    int front;
    int rear;
    int count;
    int data[QUEUE_MAX_SIZE];
}Queue;

void InitQueue(Queue *q) {
    memset(q, 0, sizeof(Queue));
}

int IsQueueEmpty(Queue *q) {
    return q->count == 0;
}

int IsQueueFull(Queue *q) {
    return q->count == QUEUE_MAX_SIZE;
}

int EnQueue(Queue *q, int x) {
    if (IsQueueFull(q)) return -1;
    q->data[q->rear] = x;
    q->count += 1;
    q->rear = (q->rear + 1) % QUEUE_MAX_SIZE;
    return 0;
}

int DeQueue(Queue *q, int *x) {
    if (IsQueueEmpty(q)) return -1;
    *x = q->data[q->front];
    q->count -= 1;
    q->front = (q->front + 1) % QUEUE_MAX_SIZE;
    return 0;
}
```

### 常数优化

在计算下标时，用到了效率不高的取余操作，如果将队列大小设为2的幂，那么可以将取余运算转化为位运算，提高效率。例如队列大小取128时，以下语句是等价的：

```
#define QUEUE_MAX_SIZE 128

q->rear = (q->rear + 1) % QUEUE_MAX_SIZE;
q->rear = (q->rear + 1) & (QUEUE_MAX_SIZE - 1);

q->front = (q->front + 1) % QUEUE_MAX_SIZE;
q->front = (q->front + 1) & (QUEUE_MAX_SIZE - 1);
```
