---
layout: post
title: 多属性记录表示法
category: cpp
tag:
---

### 问题描述

假设有n场活动，第i场活动的开始时间为a[i]，结束时间为b[i]，要求按开始时间从小到大排序，如开始时间相同则按结束时间从小到大排，输出活动编号及相应的起止时间。

### 结构体描述法

常规的做法是用结构体数组来保存活动信息。

```cpp
#include <bits/stdc++.h>
using namespace std;
struct st {
    int a, b, x;
}f[10];
int n;
bool cmp(st u, st v) {
    if (u.a != v.a) reutrn u.a < v.a;
    return u.b < v.b;
}
int main() {
    cin >> n;
    for (int i = 0; i < n; i++) {
        cin >> f[i].a >> f[i].b;
        f[i].x = i+1;
    }
    sort(f, f+n, cmp);
    for (int i = 0; i < n; i++)
        cout << f[i].x << ": (" << f[i].a << "," << f[i].b << ")\n";
    return 0;
}
```

### 一维数组表示法

也可以直接用三个一维数组来表示。

```cpp
#include <bits/stdc++.h>
using namespace std;
int n, a[10], b[10], x[10];
bool cmp(int u, int v) {
    if (a[u] != a[v]) return a[u] < a[v];
    return b[u] < b[v];
}
int main() {
    cin >> n;
    for (int i = 0; i < n; i++) {
        cin >> a[i] >> b[i];
        x[i] = i+1;
    }
    sort(x, x+n);
    for (int i = 0; i < n; i++)
        cout << x[i] << ": (" << a[x[i]] << "," << b[x[i]] << ")\n";
    return 0;
}
```

从表达能力上看，一维数组比结构体表示法要强，因为结构体表示法在做排序后不能快速获取原来编号的记录，而一维数组表示法却可方便地取到。
