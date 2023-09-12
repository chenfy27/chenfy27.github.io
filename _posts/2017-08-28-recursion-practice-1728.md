---
layout: post
title: 递归练习1708
category: 算法
keywords:
---

### 给定10进制正整数n，输出其2进制表示。

```
#include <stdio.h>
void dfs(int n) {
    if (n > 1) dfs(n / 2);
    printf("%d", n & 1);
}
int main() {
    int n;
    while (~scanf("%d", &n)) {
        dfs(n);
        puts("");
    }
    return 0;
}
```

更一般地，将正整数n用b进制输出的代码如下：

```
void print(int n, int b) {
    if (n >= b) print(n / b);
    printf("%d", n % b);
}
```

### [Sinus Dances](http://acm.timus.ru/problem.aspx?space=1&num=1149)

定义：A[n] = sin(1-sin(2+sin(3-sin(4+...sin(n))...), S[n] = (...(A[1]+n)A[2]+n-1)A[3]+...+2)A[n]+1，现给定n，输出S[n]。

```
#include <bits/stdc++.h>
using namespace std;
void dfsA(int n) {
    if (n == 1) {
        printf("sin(1");
    } else {
        dfsA(n - 1);
        printf("%ssin(%d", n & 1 ? "+" : "-", n);
    }
}
void A(int n) {
    dfsA(n);
    for (int i = 0; i < n; i++)
        printf(")");
}
void S(int x, int n) {
    if (x == 1) {
        A(1);
        printf("+%d", n);
    } else {
        printf("(");
        S(x - 1, n);
        printf(")");
        A(x);
        printf("+%d", n + 1 - x);
    }
}
int main() {
    int n;
    while (~scanf("%d", &n)) {
        S(n, n);
        puts("");
    }
    return 0;
}
```
