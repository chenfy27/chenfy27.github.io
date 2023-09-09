---
layout: post
title: 蓄水池抽样
category: 算法
keywords:
---


给出一个长度为N的流，N未知，对流中的数据只能访问一次，从中随机选出K(<=N)个元素，要求数据流中所有元素被选中的概率相等。

策略：先选中编号为1~K的元素，然后枚举编号为K+1~N的元素，设当前元素编号为X，它被选中的概率为K/X，如果被选中，则以等概率的方式从结果集中选出一个替换掉。

```c
#include <stdlib.h>
#include <stdio.h>
#include <time.h>
#define N 1024
#define K 10
void sample(int n, int a[], int k) {
    int i, r, t;
    for (i = k + 1; i <= n; i++) {
        r = 1 + rand() % i;
        if (r <= k)
            t = a[i], a[i] = a[r], a[r] = t;
    }
}
int main() {
    int i, a[N+1];
    for (i = 1; i <= N; i++) a[i] = i;
    srand(time(NULL));
    sample(N, a, K);
    for (i = 1; i <= K; i++)
        printf("%d%c", a[i], i < K ? ' ' : '\n');
    return 0;
}
```

该做法的正确性可用数学归纳法证明。
