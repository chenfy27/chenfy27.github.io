---
layout: post
title: 线段树
category: 算法
keywords: 线段树,算法
---

线段树是一棵完全二叉树，常用于解决区间统计问题。

### 线段树的结构

- 通常用数组来模拟，开4倍原始大小以避免越界。
- 从下标为1的元素（根节点）开始存数据。
- 根节点的范围是1~n。
- 设当前节点的范围是[x,y]，那么左子节点的范围是[x,(x+y)/2]，右子节点的范围是[(x+y)/2+1,y]。
- 节点范围可在函数调用时通过参数传递，以节省存储空间。

### 例题

- [插点问线](http://codevs.cn/problem/1080/)

```
#include <iostream>
using namespace std;
typedef long long LL;
const int N = 1e5+5;
LL a[N], sum[4*N];
void pushup(int x) {
    sum[x] = sum[2*x] + sum[2*x+1];
}
void build(int x, int l, int r) {
    if (l == r) {
        sum[x] = a[l];
        return;
    }
    int mid = (l + r) / 2;
    build(2*x, l, mid);
    build(2*x+1, mid+1, r);
    pushup(x);
}
void update(int x, int l, int r, int u, int v) {
    if (u < l || u > r) return;
    if (l == r) {
        sum[x] += v;
        return;
    }
    int mid = (l + r) / 2;
    update(2*x, l, mid, u, v);
    update(2*x+1, mid+1, r, u, v);
    pushup(x);
}
LL query(int x, int l, int r, int L, int R) {
    if (R < l || r < L) return 0;
    if (L <= l && r <= R) return sum[x];
    int mid = (l + r) / 2;
    return query(2*x, l, mid, L, R) + query(2*x+1, mid+1, r, L, R);
}
int main() {
    int n, m;
    cin >> n;
    for (int i = 1; i <= n; i++)
        cin >> a[i];
    build(1, 1, n);
    cin >> m;
    for (int i = 0; i < m; i++) {
        int x, y, z;
        cin >> x >> y >> z;
        if (x == 1)
            update(1, 1, n, y, z);
        else
            cout << query(1, 1, n, y, z) << endl;
    }
    return 0;
}
```

- [插线问点](http://codevs.cn/problem/1081/)

```
#include <iostream>
using namespace std;
typedef long long LL;
const int N = 1e5+5;
LL a[N], lz[4*N];
void pushdown(int x) {
    lz[2*x] += lz[x];
    lz[2*x+1] += lz[x];
    lz[x] = 0;
}
void update(int x, int l, int r, int L, int R, int v) {
    if (R < l || r < L) return;
    if (L <= l && r <= R) {
        lz[x] += v;
        return;
    }
    int mid = (l + r) / 2;
    pushdown(x);
    update(2*x, l, mid, L, R, v);
    update(2*x+1, mid+1, r, L, R, v);
}
LL query(int x, int l, int r, int u) {
    if (u < l || u > r) return 0;
    if (l == r) {
        a[l] += lz[x];
        lz[x] = 0;
        return a[l];
    }
    int mid = (l + r) / 2;
    pushdown(x);
    return query(2*x, l, mid, u) + query(2*x+1, mid+1, r, u);
}
int main() {
    int n, m;
    cin >> n;
    for (int i = 1; i <= n; i++)
        cin >> a[i];
    cin >> m;
    for (int i = 0; i < m; i++) {
        int op, x, y, z;
        cin >> op;
        if (op == 1) {
            cin >> x >> y >> z;
            update(1, 1, n, x, y, z);
        } else {
            cin >> x;
            cout << query(1, 1, n, x) << endl;
        }
    }
    return 0;
}
```

- [插线问线](http://codevs.cn/problem/1082/)

```
#include <iostream>
using namespace std;
typedef long long LL;
const int N = 2e5+5;
LL n, m, a[N], sum[4*N], lz[4*N];
void pushup(int x) {
    sum[x] = sum[2*x] + sum[2*x+1];
}
void pushdown(int x, int lcnt, int rcnt) {
    lz[2*x] += lz[x];
    lz[2*x+1] += lz[x];
    sum[2*x] += lz[x] * lcnt;
    sum[2*x+1] += lz[x] * rcnt;
    lz[x] = 0;
}
void build(int x, int l, int r) {
    if (l == r) {
        cin >> a[l];
        sum[x] = a[l];
        return;
    }
    int mid = (l + r) / 2;
    build(2*x, l, mid);
    build(2*x+1, mid+1, r);
    pushup(x);
}
void update(int x, int l, int r, int L, int R, int v) {
    if (R < l || r < L) return;
    if (L <= l && r <= R) {
        sum[x] += (LL)v * (r-l+1);
        lz[x] += v;
        return;
    }
    int mid = (l + r) / 2;
    pushdown(x, mid-l+1, r-mid);
    update(2*x, l, mid, L, R, v);
    update(2*x+1, mid+1, r, L, R, v);
    pushup(x);
}
LL query(int x, int l, int r, int L, int R) {
    if (R < l || r < L) return 0;
    if (L <= l && r <= R) return sum[x];
    int mid = (l + r) / 2;
    pushdown(x, mid-l+1, r-mid);
    return query(2*x, l, mid, L, R) + query(2*x+1, mid+1, r, L, R);
}
int main() {
    ios::sync_with_stdio(0);
    cin >> n;
    build(1, 1, n);
    cin >> m;
    for (int i = 0; i < m; i++) {
        int a, b, c, d;
        cin >> a;
        if (a == 1) {
            cin >> b >> c >> d;
            update(1, 1, n, b, c, d);
        } else {
            cin >> b >> c;
            cout << query(1, 1, n, b, c) << endl;
        }
    }
    return 0;
}
```

