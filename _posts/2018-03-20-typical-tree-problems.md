---
layout: post
title: 典型的树形问题
category: 算法
tag:
---

### [51nod1037](http://www.51nod.com/onlineJudge/questionCode.html#!problemId=1307) 绳子与重物

题面：有编号为0~n-1的n条绳子，每条绳子下栓了一个重量为w[i]的重物，绳子的最大负重为c[i]，每条绳子可挂在别的绳子下，也可直接挂在钩子上（编号为-1），如果绳子下所有重物的重量之和大于绳子的最大负重就会断掉，依次给出每条绳子的负重、重物重量以及挂载编号，问最多可以挂多少条绳子且不会出现绳子断掉的情况。

分析：二分答案，对于给定的绳子数，dfs遍历整棵树，求出各个节点的负重，检查是否会断，时间复杂度为O(n)，总时间复杂度为O(nlogn)。

```
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
LL n, c[50005], w[50005], p[50005], h[50005];
vector<int> g[50005];
int dfs(int x) {
    h[x] = w[x];
    for (auto i : g[x]) {
        if (!dfs(i)) return 0;
        h[x] += h[i];
    }
    return h[x] <= c[x];
}
int chk(int x) {
    for (int i = 0; i <= x; i++)
        g[i].clear();
    for (int i = 1; i <= x; i++)
        g[p[i]].push_back(i);
    return dfs(0);
}
int main() {
    cin >> n;
    c[0] = 1e18; w[0] = 0;
    for (int i = 1; i <= n; i++) {
        cin >> c[i] >> w[i] >> p[i];
        p[i] += 1;
    }
    int lo = 1, hi = n, mid;
    while (lo < hi) {
        mid = lo + (hi-lo+1) / 2;
        if (chk(mid))
            lo = mid;
        else
            hi = mid-1;
    }
    cout << lo << endl;
    return 0;
}
```

### [51nod1405](http://www.51nod.com/onlineJudge/questionCode.html#!problemId=1405) 树的距离之和

题面：给定一棵有编号为1~n的n个节点构成的无根树，对于每个节点，求它到其他节点的距离之和。

分析：任选一个（比如编号为1）节点作为根，定义son[i]表示以编号为i的节点为根的子树包含的节点数，dp[i]为所求。第1次dfs统计出son[i]，顺带可计算出dp[1]，然后进行递推。设x是一个非根节点，其父节点为y，并且dp[y]已算出，那么dp[x]可分两部分算出：以x为根的子树中的节点，到x的距离比到y的距离少1，这样的节点有son[x]个；其余节点到x的距离比到y的距离多1，这样的节点有(n-num[x])个。

```
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
int n, son[100005];
vector<int> g[100005];
LL dp[100005];
void dfs1(int x, int p, int d, LL &sum) {
    sum += d;
    son[x] = 1;
    for (auto i : g[x]) if (i != p) {
        dfs1(i, x, d+1, sum);
        son[x] += son[i];
    }
}
void dfs2(int x, int p) {
    if (p > 0)
        dp[x] = dp[p] + n - 2 * son[x];
    for (auto i : g[x]) if (i != p) {
        dfs2(i, x);
    }
}
int main() {
    cin >> n;
    for (int i = 1; i < n; i++) {
        int x, y;
        cin >> x >> y;
        g[x].push_back(y);
        g[y].push_back(x);
    }
    dfs1(1, -1, 0, dp[1]);
    dfs2(1, -1);
    for (int i = 1; i <= n; i++)
        cout << dp[i] << endl;
    return 0;
}
```
