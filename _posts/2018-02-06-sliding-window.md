---
layout: post
title: 滑动窗口最值
category: 算法
tag:
---

[问题](https://cn.vjudge.net/problem/POJ-2823)：给出一个整数序列a[n]，现在有一个宽度为k的窗口从最左端往右移动，求各个位置处窗口内部元素的最小值和最大值。

最简单的方法是O(nk)暴力，会超时。

可以看成普通的区间最值问题，用线段树和ST表来做，时间复杂度为O(nlogn)，比前一种方法要快，但仍会超时，下面是用线段树来做。

```cpp
#include <iostream>
#include <cstdio>
#include <climits>
using namespace std;
const int N = 1000005;
int n, k, a[N], mx[4*N], mn[4*N];
void pushup(int x) {
    mx[x] = max(mx[2*x], mx[2*x+1]);
    mn[x] = min(mn[2*x], mn[2*x+1]);
}
void build(int x, int l, int r) {
    if (l == r) {
        mx[x] = mn[x] = a[l];
        return;
    }
    int mid = (l + r) / 2;
    build(2*x, l, mid);
    build(2*x+1, mid+1, r);
    pushup(x);
}
int qmn(int x, int l, int r, int L, int R) {
    if (R < l || r < L) return INT_MAX;
    if (L <= l && r <= R) return mn[x];
    int mid = (l + r) / 2;
    return min(qmn(2*x, l, mid, L, R), qmn(2*x+1, mid+1, r, L, R));
}
int qmx(int x, int l, int r, int L, int R) {
    if (R < l || r < L) return INT_MIN;
    if (L <= l && r <= R) return mx[x];
    int mid = (l + r) / 2;
    return max(qmx(2*x, l, mid, L, R), qmx(2*x+1, mid+1, r, L, R));
}
int main() {
    scanf("%d%d", &n, &k);
    k -= 1;
    for (int i = 1; i <= n; i++)
        scanf("%d", &a[i]);
    build(1, 1, n);
    for (int i = 1; i + k <= n; i++)
        printf("%d%s", qmn(1, 1, n, i, i+k), i+k == n ? "\n" : " ");
    for (int i = 1; i + k <= n; i++)
        printf("%d%s", qmx(1, 1, n, i, i+k), i+k == n ? "\n" : " ");
    return 0;
}
```

另外一种做法是用map来维护窗口中的元素，取最大和最小都是O(logk)，总时间复杂度为O(nlogk)，编码简单些，但也会超时。

```cpp
#include <map>
#include <vector>
#include <cstdio>
using namespace std;
int a[1000005];
int main() {
    int n, k;
    scanf("%d%d", &n, &k);
    for (int i = 1; i <= n; i++)
        scanf("%d", &a[i]);
    map<int,int> mp;
    for (int i = 1; i < k; i++)
        mp[a[i]] += 1;
    vector<int> mx, mn;
    for (int i = k; i <= n; i++) {
        mp[a[i]] += 1;
        mn.push_back(mp.begin()->first);
        mx.push_back(mp.rbegin()->first);
        int key = a[i-k+1];
        if (--mp[key] == 0)
            mp.erase(key);
    }
    for (int i = 0; i < mn.size(); i++)
        printf("%d%s", mn[i], i + 1 == mn.size() ? "\n" : " ");
    for (int i = 0; i < mx.size(); i++)
        printf("%d%s", mx[i], i + 1 == mx.size() ? "\n" : " ");
    return 0;
}
```

最后是用单调队列来做，时间复杂度为O(n)，由于POJ没开O2优化，用STL可能会超时，可以用数组来模拟队列，并加上快速IO。

```cpp
#include <cstdio>
#include <deque>
#include <vector>
using namespace std;
int n, k, a[1000005];
deque<int> qmx, qmn;
vector<int> amx, amn;
void add(int x) {
    while (!qmx.empty() && qmx.back() < x)
        qmx.pop_back();
    qmx.push_back(x);
    while (!qmn.empty() && qmn.back() > x)
        qmn.pop_back();
    qmn.push_back(x);
}
void del(int x) {
    if (qmx.front() == x)
        qmx.pop_front();
    if (qmn.front() == x)
        qmn.pop_front();
}
int main() {
    scanf("%d%d", &n, &k);
    for (int i = 1; i <= n; i++)
        scanf("%d", &a[i]);
    for (int i = 1; i < k; i++)
        add(a[i]);
    for (int i = k; i <= n; i++) {
        add(a[i]);
        amx.push_back(qmx.front());
        amn.push_back(qmn.front());
        del(a[i-k+1]);
    }
    for (int i = 0; i < amn.size(); i++)
        printf("%d%s", amn[i], i + 1 == amn.size() ? "\n" : " ");
    for (int i = 0; i < amx.size(); i++)
        printf("%d%s", amx[i], i + 1 == amx.size() ? "\n" : " ");
    return 0;
}
```
