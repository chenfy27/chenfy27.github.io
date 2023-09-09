---
layout: post
title: 树链剖分
category: 算法
keywords:
---

树链剖分一般用于解决树上路径更新或查询问题，基本思路是将树拆分成链，然后用其他数据结构去维护链，从而将树上的问题转化为区间问题。

例1：基于点权[HDU3966](http://acm.hdu.edu.cn/showproblem.php?pid=3966)

题目大意是有一棵含有n个节点的树，节点编号为1~n，已知树上各节点的初始权值，然后有q组操作，每组操作为以下三种之一：

- I x y z: 将节点x与节点y的路径上所有点的权值增加z；
- D x y z: 将节点x与节点y的路径上所有点的权值减少z；
- Q x: 查询节点x的权值；

先做树链剖分，转为化插线问点问题，可以用线段树或树状数组来实现。

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 50005;
int n, m, p, a[N], f[4*N], lz[4*N];
int dep[N], son[N], top[N], siz[N], fa[N];
int tid[N], pos[N], tot;
vector<int> g[N];
void dfs1(int x, int p, int h) {
    dep[x] = h; siz[x] = 1; fa[x] = p;
    for (auto i : g[x]) {
        if (p != i) {
            dfs1(i, x, h + 1);
            siz[x] += siz[i];
            if (siz[x] > siz[son[x]])
                son[x] = i;
        }
    }
}
void dfs2(int x, int p, int t) {
    top[x] = t; tid[x] = ++tot; pos[tot] = x;
    if (son[x] == 0) return;
    dfs2(son[x], x, t);
    for (auto i : g[x]) {
        if (i != p && i != son[x])
            dfs2(i, x, i);
    }
}
void build(int x, int l, int r) {
    if (l == r) {
        lz[x] = 0;
        f[x] = a[pos[l]];
    } else {
        int mid = (l + r) / 2;
        build(2*x, l, mid);
        build(2*x+1, mid + 1, r);
        lz[x] = 0;
    }
}
void pushdown(int x) {
    lz[2*x] += lz[x];
    lz[2*x+1] += lz[x];
    lz[x] = 0;
}
void add(int x, int l, int r, int L, int R, int v) {
    if (l == L && r == R) {
        lz[x] += v;
        return;
    }
    pushdown(x);
    int mid = (l + r) / 2;
    if (R <= mid) add(2*x, l, mid, L, R, v);
    else if (L > mid) add(2*x+1, mid+1, r, L, R, v);
    else add(2*x, l, mid, L, mid, v), add(2*x+1, mid+1, r, mid+1, R, v);
}
void update(int x, int y, int z) {
    while (top[x] != top[y]) {
        if (dep[top[x]] < dep[top[y]])
            swap(x, y);
        add(1, 1, n, tid[top[x]], tid[x], z);
        x = fa[top[x]];
    }
    if (dep[x] > dep[y]) swap(x, y);
    add(1, 1, n, tid[x], tid[y], z);
}
int query(int x, int l, int r, int u) {
    if (l == r) {
        return f[x] + lz[x];
    }
    pushdown(x);
    int mid = (l + r) / 2;
    if (u <= mid) return query(2*x, l, mid, u);
    return query(2*x+1, mid+1, r, u);
}
int main() {
    while (~scanf("%d%d%d", &n, &m, &p)) {
        for (int i = 1; i <= n; i++)
            scanf("%d", &a[i]);
        tot = 0;
        memset(son, 0, sizeof(son));
        for (int i = 1; i <= n; i++)
            g[i].clear();
        for (int i = 1; i < n; i++) {
            int x, y;
            scanf("%d%d", &x, &y);
            g[x].push_back(y);
            g[y].push_back(x);
        }
        dfs1(1, 0, 1);
        dfs2(1, 0, 1);
        build(1, 1, n);
        for (int i = 0; i < p; i++) {
            char op[4];
            int x, y, z;
            scanf("%s", op);
            if ('I' == op[0]) {
                scanf("%d%d%d", &x, &y, &z);
                update(x, y, z);
            } else if ('D' == op[0]) {
                scanf("%d%d%d", &x, &y, &z);
                update(x, y, -z);
            } else {
                scanf("%d", &x);
                printf("%d\n", query(1, 1, n, tid[x]));
            }
        }
    }
    return 0;
}
```

例2：基于边权[SPOJ375](http://www.spoj.com/problems/QTREE/)

题目大意为给定一棵含有n个节点的树以及初始各个边的权值，现有多次操作，每次操作为以下两种之一：

- CHANGE a b: 将第a条边的权值改为b；
- QUERY a b: 询问节点a到节点b的路径上的最大边权；

将边权记录到子节点上，树剖后转化为区间最值问题，可用线段树解决。需要注意查询时区间不能包含最近公共祖先的权值。

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 10005;
int T, n, a, b, c, seq, f[4*N], from[N], to[N];
int son[N], fa[N], dep[N], siz[N], top[N], tid[N], pos[N], w[N];
vector<pair<int,int> > g[N];
void dfs1(int x, int u, int h) {
    fa[x] = u; dep[x] = h; siz[x] = 1; son[x] = 0;
    for (size_t i = 0; i < g[x].size(); i++) {
        int v = g[x][i].first;
        if (v != u) {
            w[v] = g[x][i].second;
            dfs1(v, x, h + 1);
            siz[x] += siz[v];
            if (siz[v] > siz[son[x]])
                son[x] = v;
        }
    }
}
void dfs2(int x, int u, int t) {
    top[x] = t; tid[x] = ++seq; pos[seq] = x;
    if (son[x] == 0) return;
    dfs2(son[x], x, t);
    for (size_t i = 0; i < g[x].size(); i++) {
        int v = g[x][i].first;
        if (v != u && v != son[x])
            dfs2(v, x, v);
    }
}
void build(int x, int l, int r) {
    if (l == r) {
        f[x] = w[pos[l]];
    } else {
        int mid = (l + r) / 2;
        build(2*x, l, mid);
        build(2*x+1, mid+1, r);
        f[x] = max(f[2*x], f[2*x+1]);
    }
}
void update(int x, int l, int r, int u, int v) {
    if (l == r) {
        f[x] = v;
    } else {
        int mid = (l + r) / 2;
        if (u <= mid) update(2*x, l, mid, u, v);
        else update(2*x+1, mid+1, r, u, v);
        f[x] = max(f[2*x], f[2*x+1]);
    }
}
int query(int x, int l, int r, int u, int v) {
    if (l == u && r == v) return f[x];
    int mid = (l + r) / 2;
    if (v <= mid) return query(2*x, l, mid, u, v);
    if (u > mid) return query(2*x+1, mid+1, r, u, v);
    return max(query(2*x, l, mid, u, mid), query(2*x+1, mid+1, r, mid+1, v));
}
int main() {
    scanf("%d", &T);
    while (T--) {
        scanf("%d", &n);
        seq = 0;
        for (int i = 1; i <= n; i++)
            g[i].clear();
        for (int i = 1; i < n; i++) {
            scanf("%d%d%d", &a, &b, &c);
            g[a].push_back(make_pair(b, c));
            g[b].push_back(make_pair(a, c));
            from[i] = a; to[i] = b;
        }
        dfs1(1, 0, 1);
        dfs2(1, 0, 1);
        build(1, 1, n);
        char op[10];
        while (scanf("%s", op), op[0] != 'D') {
            scanf("%d%d", &a, &b);
            if (op[0] == 'C') {
                int z = fa[from[a]] == to[a] ? from[a] : to[a];
                update(1, 1, n, tid[z], b);
            } else {
                int ans = 0;
                while (top[a] != top[b]) {
                    if (dep[top[a]] < dep[top[b]]) swap(a, b);
                    ans = max(ans, query(1, 1, n, tid[top[a]], tid[a]));
                    a = fa[top[a]];
                }
                if (a != b) {
                    if (dep[a] < dep[b]) swap(a, b);
                    ans = max(ans, query(1, 1, n, tid[b] + 1, tid[a]));
                }
                printf("%d\n", ans);
            }
        }
    }
    return 0;
}
```
