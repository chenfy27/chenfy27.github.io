---
layout: post
title: 树与图的直径
category: 算法
tag:
---

在一棵树中，将任意两个节点之间最短距离的最大值定义为该树的直径。类似地，在一个无向图中，将任意两点间最短距离的最大值定义为图的直径。

### 树的直径

求树的直径一般用两次bfs解决，当然也可以用两次dfs，这里以bfs为例进行说明。先在树上任选一点A作为起点进行bfs，记录最后被访问的点B，那么B离A的距离最远，再以B为起点进行第二次bfs，即可得到直径。

```
#include <bits/stdc++.h>
using namespace std;
int n, vis[100005];
vector<int> g[100005];
void bfs(int start, int &last, int &dist) {
    last = dist = 0;
    queue<pair<int,int>> q;
    memset(vis, 0, sizeof(vis));
    q.push(make_pair(start, 0)); vis[start] = 1;
    while (!q.empty()) {
        pair<int,int> pr = q.front(); q.pop();
        last = pr.first;
        dist = pr.second;
        for (auto i : g[pr.first]) {
            if (!vis[i]) {
                q.push(make_pair(i, pr.second+1));
                vis[i] = 1;
            }
        }
    }
}
int main() {
    cin >> n;
    int x, y, last, dist;
    for (int i = 1; i < n; i++) {
        cin >> x >> y;
        g[x].push_back(y);
        g[y].push_back(x);
    }
    bfs(x, last, dist);
    bfs(last, last, dist);
    cout << dist << endl;
    return 0;
}
```

### 无向图的直径

由于图可能不连通，且可能有环，不能使用类似树的方法求解，考虑用更为一般的做法，根据直径定义，先通过floyd算法求出任意两点间的最短距离，再取最大值即可，注意图不连通的情况。

```
#include <bits/stdc++.h>
using namespace std;
int n, m, d[105][105];
const int inf = 0x3f3f3f3f;
int main() {
    memset(d, 0x3f, sizeof(d));
    cin >> n >> m;
    for (int i = 0; i < m; i++) {
        int x, y;
        cin >> x >> y;
        d[x][y] = d[y][x] = 1;
    }
    for (int k = 1; k <= n; k++)
    for (int i = 1; i <= n; i++)
    for (int j = 1; j <= n; j++)
        d[i][j] = min(d[i][j], d[i][k]+d[k][j]);
    int ans = 0;
    for (int i = 1; i <= n; i++)
    for (int j = i+1; j <= n; j++)
        ans = max(ans, d[i][j]);
    cout << (ans == inf ? -1 : ans) << endl;
    return 0;
}
```
