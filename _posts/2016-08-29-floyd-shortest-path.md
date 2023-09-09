---
layout: post
title: floyd最短路
category: 算法
keywords:
---

Floyd算法用于求加权图中任意两点之间的最短路径，很好地体现了动态规划思想。

### 算法流程

1. 初始化任意两点之间的距离为正无穷；
2. 加入一个点，遍历任意两点的距离，检查两点之间如果经过新加点的话路径是否会缩短，如果会则更新；
3. 重复步骤2直到所有点都加完；

时间复杂度为O(n^3)，经过处理后图上任意两点之间的最短路径都被求出来了。

设dist[i][j]表示从点i到点j的最短路径长度，那么初始化：`dist[i][j] = (i == j) ? 0 : INF`，处理过程如下：

```cpp
for (k = 1; k <= n; k++)
for (i = 1; i <= n; i++)
for (j = 1; j < n; j++)
	dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);
```

### [例题](http://hihocoder.com/problemset/problem/1089)

```cpp
#include <stdio.h>
#include <string.h>
#define INT_MAX 0x3f3f3f3f
int g[105][105];
int min(int x, int y) {
    return x < y ? x : y;
}
int main() {
    int n, m, u, v, l, i, j, k;
    scanf("%d%d", &n, &m);
    for (i = 1; i <= n; i++)
    for (j = 1; j <= n; j++)
        g[i][j] = i == j ? 0 : INT_MAX;
    for (i = 0; i < m; i++) {
        scanf("%d%d%d", &u, &v, &l);
        g[u][v] = g[v][u] = min(g[u][v], l);
    }
    for (k = 1; k <= n; k++)
    for (i = 1; i <= n; i++)
    for (j = 1; j <= n; j++)
        g[i][j] = min(g[i][j], g[i][k] + g[k][j]);
    for (i = 1; i <= n; i++)
    for (j = 1; j <= n; j++)
        printf("%d%c", i == j ? 0 : g[i][j], j == n ? '\n' : ' ');
    return 0;
}
```

