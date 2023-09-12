---
layout: post
title: 二分图
category: 算法
keywords:
---

二分图是图论中的一种特殊模型，设G=(V,E)是一个无向图，如果顶点V可以分割为两个不相交的子集(A,B)，并且图中的每条边所关联的两个顶点分别属于这两个不同的顶点集，则称图G为一个二分图。

### 1. 判定二分图

判定一个图是否为二分图可以用染色法，由二分图定义可知，任意一条边的两个顶点的颜色互不相同，因此问题可以转化为是否存在一个合理的染色方案，使得该无向图满足每一条边两端的顶点颜色都不相同。可以任选一点作为起点，通过BFS或者DFS来实现，时间复杂度为O(n)。

以下是DFS实现的核心代码：

```
int color[N] = {0};
int dfs(int x, int c) {
    color[x] = c;
    for (int i = 0; i < g[x].size(); i++) {
        int j =g[x][i];
        if (c == color[j])
            return 1;
        if (!color[j] && dfs(j, -c)
            return 1;
    }
    return 0;
}
```

### 2. 二分图最大匹配

给定一个二分图G，在G的一个子图M中，如果M的边集中的任意两条边都不依附于同一个顶点，则称M是一个**匹配**。  
边数最多的匹配称为**最大匹配**。  
如果一个匹配中，每个顶点都和某条边相关联，则称该匹配为**完全匹配**，或**完备匹配**。

求二分图最大匹配一般用匈牙利算法，思路大致是：

1. 依次枚举每一个顶点i；
2. 若顶点i尚未匹配，则以该点为起点找增广路，如果存在，则匹配数加1；

参考代码如下：

```
#include <stdio.h>
#include <string.h>
#include <vector>
std::vector<int> g[N];
int match[N], visit[N];
int dfs(int x) {
    visit[x] = 1;
    for (auto i : g[x]) {
        int j = match[i];
        if (!j || (!visit[j] && dfs(j))) {
            match[i] = x;
            match[x] = i;
            return 1;
        }
    }
    return 0;
}
int main() {
    int n, m, u, v, i, ans;
    scanf("%d%d", &n, &m);
    for (i = 0; i < m; i++) {
        scanf("%d%d", &u, &v);
        g[u].push_back(v);
        g[v].push_back(u);
    }
    for (ans = 0, i = 1; i <= n; i++) {
        if (match[i]) continue;
        memset(visit, 0, sizeof(visit));
        ans += dfs(i);
    }
    printf("%d\n", ans);
    return 0;
}
```

### 3. 最小顶点覆盖

问题：在图G中选取尽可能少的点，使得图中每一条边至少有一个端点被选中。这个问题在二分图中被称为最小顶点覆盖，即用最少的点去覆盖所有的边。

可以证明：最小顶点覆盖数 = 二分图最大匹配

### 4. 最大独立集

问题：在图G中选取尽可能多的点，使得任意两个点之间没有连边。这个问题在二分图中被称为最大独立集问题。

可以证明：最大独立集的点数 = 总点数 - 二分图最大匹配

