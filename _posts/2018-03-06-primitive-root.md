---
layout: post
title: 原根
category: 算法
tag:
---

原根是数论中的一个概念，设m是正整数，a是整数，若a模m的阶等于phi(m)，则称a为模m的一个原根。

### 存在条件

自然数n有原根的充要条件是：

```
n = 2, 4, p^s, 2*p^s  (p>=3, s>=1)
```

### 原根个数

若模n有原根，则n一定恰好有phi(phi(n))个不同余的原根(mod n)。

设g为一个原根，则1, g, g^2, ..., g^(phi(n)-1)恰好组成模n的一个简化剩余系。

### 原根求法

假设g是n的一个原根，那么将phi(n)分解质因数，得到不同的质因子p[i]，必有g^(phi(n)/p[i]) mod n不等于1。

特别的，如果n是质数，那么phi(n)=n-1，对应的有g^((n-1)/p[i]) mod n恒不等于1。

求原根没什么好的办法，由于原根一般不大，直接暴力枚举验证上述条件是否成立即可。

例题：[51nod1135](http://www.51nod.com/onlineJudge/questionCode.html#!problemId=1135) 给出质数p(3<=p<=1e9)，求p最小的原根。

```cpp
#include <bits/stdc++.h>
using namespace std;
int n, p[10], np;
void fact(int x) {
    for (int i = 2; i * i <= x; i++) {
        if (x % i == 0) {
            p[np++] = i;
            while (x % i == 0)
                x /= i;
        }
    }
    if (x > 1) p[np++] = x;
}
int powmod(int a, int b, int c) {
    long long r = 1, t = a;
    while (b) {
        if (b & 1) r = r * t % c;
        t = t * t % c;
        b /= 2;
    }
    return r;
}
bool chk(int g) {
    for (int i = 0; i < np; i++)
        if (powmod(g, (n-1)/p[i], n) == 1)
            return false;
    return true;
}
int main() {
    cin >> n;
    fact(n-1);
    for (int g = 2; g < n; g++) {
        if (chk(g)) {
            cout << g << endl;
            return 0;
        }
    }
    return 0;
}
```
