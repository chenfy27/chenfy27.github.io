---
layout: post
title: 背包模板题
category: 算法
tag:
---

### [hiho1038](http://hihocoder.com/problemset/problem/1038) 01背包

题面：有n种物品和一个容量为v的背包，每种物品只有1件，第i种物品的花费为c[i]，价值为w[i]，求背包所能容纳的最大价值。

```
#include <bits/stdc++.h>
using namespace std;
int n, v, c[505], w[505], f[100005];
void zpack(int ci, int wi) {
    for (int j = v; j >= ci; j--)
        f[j] = max(f[j], f[j-ci] + wi);
}
int main() {
    cin >> n >> v;
    for (int i = 1; i <= n; i++)
        cin >> c[i] >> w[i];
    for (int i = 1; i <= n; i++)
        zpack(c[i], w[i]);
    cout << f[v] << endl;
    return 0;
}
```

### [hiho1043](http://hihocoder.com/problemset/problem/1043) 完全背包

题面：有n种物品和一个容量为v的背包，每种物品都有无数件，第i种物品的花费为c[i]，价值为w[i]，求背包所能容纳的最大价值。

```
#include <bits/stdc++.h>
using namespace std;
int n, v, c[505], w[505], f[100005];
void cpack(int ci, int wi) {
    for (int j = ci; j <= v; j++)
        f[j] = max(f[j], f[j-ci] + wi);
}
int main() {
    cin >> n >> v;
    for (int i = 1; i <= n; i++)
        cin >> c[i] >> w[i];
    for (int i = 1; i <= n; i++)
        cpack(c[i], w[i]);
    cout << f[v] << endl;
    return 0;
}
```

### [51nod1086](http://www.51nod.com/onlineJudge/questionCode.html#!problemId=1086) 多重背包

题面：有n种物品和一个容量为v的背包，每种物品m[i]件，第i种物品的花费为c[i]，价值为w[i]，求背包所能容纳的最大价值。

```
#include <bits/stdc++.h>
using namespace std;
int n, v, c[105], w[105], m[105], f[50005];
void zpack(int ci, int wi) {
    for (int j = v; j >= ci; j--)
        f[j] = max(f[j], f[j-ci] + wi);
}
void cpack(int ci, int wi) {
    for (int j = ci; j <= v; j++)
        f[j] = max(f[j], f[j-ci] + wi);
}
void mpack(int ci, int wi, int mi) {
    if (ci * mi >= v) {
        cpack(ci, wi);
    } else {
        int k = 1;
        while (k < mi) {
            zpack(k * ci, k * wi);
            mi = mi - k;
            k = k * 2;
        }
        zpack(mi * ci, mi * wi);
    }
}
int main() {
    cin >> n >> v;
    for (int i = 1; i <= n; i++)
        cin >> c[i] >> w[i] >> m[i];
    for (int i = 1; i <= n; i++)
        mpack(c[i], w[i], m[i]);
    cout << f[v] << endl;
    return 0;
}
```
