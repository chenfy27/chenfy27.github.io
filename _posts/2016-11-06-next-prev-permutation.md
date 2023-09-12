---
layout: post
title: 字典序排列
category: 算法
keywords:
---

问题：给定由一组互不相同的整数构成的排列，如何求它的字典序上/下一个排列？

STL提供了现成接口next_permutation/prev_permutation，这里讨论该算法的底层实现。假设数字都存在数组a[n]中，按如下方法可得到下一排列。

### 求next_permutation

1. 从右往左扫描，找到第一个位置i，满足a[i] < a[i+1]。
2. 如果步骤1中没找到，说明当前排列已是最大，直接退出。
3. 从右往左扫描，找到第一个位置j，满足a[j] > a[i]，由于步骤2没退出，j一定存在。
4. 交换a[i]与a[j]。
5. 将a[i]后的所有元素逆序。

### 求prev_permutation

方法与求next_permutation类似，区别在于步骤1要找的i满足的条件是a[i] > a[i+1]，相应的步骤3中要找的j应满足a[j] < a[i]。

### 参考代码

```
#include <stdio.h>
void swap(int *x, int *y) {
    int t = *x; *x = *y; *y = t;
}
void print(int a[], int n) {
    int i;
    for (i = 0; i < n; ++i)
        printf("%d", a[i]);
    puts("");
}
void reverse(int a[], int lo, int hi) {
    int i, j;
    for (i = lo, j = hi; i < j; ++i, --j)
        swap(&a[i], &a[j]);
}
int next_permutation(int a[], int n) {
    int i, j;
    for (i = n - 2; i >= 0 && a[i] > a[i+1]; --i);
    if (i < 0) return 0; // no more next
    for (j = n - 1; j >= 0 && a[j] < a[i]; --j);
    swap(&a[i], &a[j]);
    reverse(a, i + 1, n - 1); 
    return 1;
}
int prev_permutation(int a[], int n) {
    int i, j;
    for (i = n - 2; i >= 0 && a[i] < a[i+1]; --i);
    if (i < 0) return 0; // no more prev
    for (j = n - 1; j >= 0 && a[j] > a[i]; --j);
    swap(&a[i], &a[j]);
    reverse(a, i + 1, n - 1); 
    return 1;
}
int main() {
    int i, n, a[10];
    scanf("%d", &n);
    for (i = 0; i < n; ++i)
        a[i] = i + 1;
    for (print(a, n); next_permutation(a, n); print(a, n));
    for (print(a, n); prev_permutation(a, n); print(a, n));
    return 0;
}
```

### 扩展

- 如果元素允许重复，又该怎么求？（仍可采用以上方法）

