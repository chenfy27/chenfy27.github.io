---
layout: post
title: 划分树
category: 算法
keywords:
---

划分树与线段树类似，主要用于解决多次询问区间第K小的问题。

### 基本方法

首先，离线建立划分树，这是一个自上而下分层处理的过程，需要用到以下变量：

- s[N]为原序列排序后的数组
- t[dep][i]为划分到第dep层时第i个位置的数
- tol[dep][i]代表第dep层的前i个数中划分到左子树的数的个数

对于区间[l,r]，设排序后的中位数为v，划分时将不大于v的数划分到左子树，大于v的数划分到右子树。如果区间中存在多个与v相等的数，则应均分，使左右子树分到的数的个数之差不超过1。

查询时，则根据k和当前区间tol的关系判断是到左子树还是右子树中查，注意更新区间边界以及k的值。

### [例题](http://poj.org/problem?id=2104)

给定一个数字序列，然后有多次询问Q(l, r, k)，即区间[l,r]内第k小的数是多少。


```
#include <cstdio>
#include <algorithm>
using namespace std;
int n, m, t[20][100005], s[100005], tol[20][100005];
void build(int dep, int l, int r) {
    if (l == r) return;
    int mid = (l + r) / 2;
    int same = mid - l + 1;
    for (int i = l; i <= r; i++)
        same -= t[dep][i] < s[mid];
    int lpos = l, rpos = mid + 1;
    for (int i = l; i <= r; i++) {
        if (t[dep][i] < s[mid]) {
            t[dep+1][lpos++] = t[dep][i];
        } else if (t[dep][i] == s[mid] && same > 0) {
            t[dep+1][lpos++] = t[dep][i];
            same -= 1;
        } else {
            t[dep+1][rpos++] = t[dep][i];
        }
        tol[dep][i] = tol[dep][l-1] + lpos - l;
    }
    build(dep + 1, l, mid);
    build(dep + 1, mid + 1, r);
}
int query(int dep, int l, int r, int ql, int qr, int k) {
    if (l == r) return t[dep][l];
    int mid = (l + r) / 2;
    int cnt = tol[dep][qr] - tol[dep][ql-1];
    if (cnt >= k) {
        int nql = l + tol[dep][ql-1] - tol[dep][l-1];
        int nqr = nql + cnt - 1;
        return query(dep + 1, l, mid, nql, nqr, k);
    } else {
        int nqr = qr + tol[dep][r] - tol[dep][qr];
        int nql = nqr - (qr - ql - cnt);
        return query(dep + 1, mid + 1, r, nql, nqr, k - cnt);
    }
}
int main() {
    int x, y, z;
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) {
        scanf("%d", &t[0][i]);
        s[i] = t[0][i];
    }
    sort(s + 1, s + 1 + n);
    build(0, 1, n);
    for (int i = 0; i < m; i++) {
        scanf("%d%d%d", &x, &y, &z);
        printf("%d\n", query(0, 1, n, x, y, z));
    }
    return 0;
}
```
