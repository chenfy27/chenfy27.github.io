---
layout: post
title: 最小生成树
category: 算法
keywords:
---

一个图的最小生成树是原图的极小连通子图，包含原图中的所有顶点，并且有保持图连通的最少的边。最小生成树一般用Kruskal算法或Prim算法求。

### Kruskal算法

思路是先将图中所有边按权值从小到大排序，然后依次遍历各条边，如果该边的两个顶点不连通，则选用该边构成最小生成树，否则丢弃。由于边接n个顶点只需要n-1条边，循环可以提前终止。点之间是否连通可以用并查集来维护。

例题：[求最小生成树](http://hihocoder.com/problemset/problem/1098)

```cpp
int f[100005];
struct Edge {
    int s, t, c;
    bool operator<(const Edge &o) const {
        return c < o.c;
    }
}e[1000005];
int root(int x) {
    return x == f[x] ? x : f[x] = root(f[x]);
}
int connected(int x, int y) {
    return root(x) == root(y);
}
void merge(int x, int y) {
    f[root(x)] = root(y);
}
int main() {
    int n, m, i, j, ans;
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) f[i] = i;
    for (i = 0; i < m; i++)
        scanf("%d%d%d", &e[i].s, &e[i].t, &e[i].c);
    sort(e, e + m);
    for (ans = i = j = 0; i < m && j < n - 1; i++) {
        if (!connected(e[i].s, e[i].t)) {
            j += 1;
            ans += e[i].c;
            merge(e[i].s, e[i].t);
        }
    }
    printf("%d\n", ans);
    return 0;
}
```

### Prim算法

Prim算法的策略以及代码实现与Dijkstra算法几乎是相同的，唯一的区别在于Dijkstra算法的点在入堆时用的是源点到该点的距离，而Prim算法中用的是边权。

```cpp
int n, m, a, b, c, v[10005], ans;
vector<pair<int,int> > g[10005];
struct ST {
    int idx, dst;
    bool operator<(const ST &o) const {
        return dst > o.dst;
    }
};
priority_queue<ST> pq;
int main() {
    scanf("%d%d", &n, &m);
    for (int i = 0; i < m; i++) {
        scanf("%d%d%d", &a, &b, &c);
        g[a].push_back(make_pair(b, c));
        g[b].push_back(make_pair(a, c));
    }
    pq.push((ST){a, 0});
    while (!pq.empty()) {
        ST t = pq.top(); pq.pop();
        if (v[t.idx]) continue;
        v[t.idx] = 1; ans += t.dst;
        for (size_t i = 0; i < g[t.idx].size(); i++) {
            pair<int,int> j = g[t.idx][i];
            if (!v[j.first])
                pq.push((ST){j.first, j.second});
        }
    }
    printf("%d\n", ans);
    return 0;
}
```

### 说明

- 一般来讲，稠密图用Prim算法更快，而稀疏图用Kruskal更快。

