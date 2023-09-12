---
layout: post
title: 邻接表的数组表示
category: 算法
keywords:
---

图一般用邻接矩阵或者邻接表来存储，邻接矩阵简单易懂，但比较浪费空间，特别是对稀疏图，而邻接表一般要链表或者vector容器来实现，手写链表麻烦，用vector要C++的支持，且最好要开O2优化，否则效率不理想。下面介绍种只用数组实现邻接表的方法。

假设图中点数为N，边数为M（无向图边数要乘2），按边来存，开数组fr[M+1], to[M+1], wt[M+1]分别表示第i条边的起点，终端和权值，另外开数组next[M+1]表示fr[i]的下一条边的位置，最后用数组first[N+1]记录每个顶点第一条边的位置，读图时采用类似链表倒插法维护上述五个数组即可。

```
int n, m, fr[1005], to[1005], wt[1005], next[1005], first[105];
for (int i = 1; i <= m; i++) {
    scanf("%d%d%d", &fr[i], &to[i], &wt[i]);
    next[i] = first[fr[i]];
    first[fr[i]] = i;
}
```

根据first和next数组可方便地遍历某个点的边。

```
void dfs(int c) {
    vis[c] = 1;
    for (int e = first[x]; e; e = next[e]) {
        if (!vis[to[e]])
            dfs(to[e]);
    }
}
```

