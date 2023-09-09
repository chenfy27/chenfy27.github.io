---
layout: post
title: 优化visit数组的重置操作
category: 算法
tag:
---

有时候，需要以数据中的每一项为起点进行dfs或者bfs，比如寻找二分图的最大匹配问题，此时一般要借助visit数组来标记某一项是否被访问过了，避免死循环。在数据量较大的情况下，visit数组开得也比较大，多次memset整个visit数组要花很长时间。

以下代码展示了一般的做法：

```cpp
#include <bits/stdc++.h>
using namespace std;
int n, mx, a[100005], vis[100005];
void bfs(int x) {
    queue<int> q;
    q.push(x); vis[x] = 1;
    while (!q.empty()) {
        int t = q.front(); q.pop();
        cout << t << " ";
        if (2*t <= mx && !vis[2*t])
            q.push(2*t), vis[2*t] = 1;
        if (t/2 >= 1  && !vis[t/2])
            q.push(t/2), vis[t/2] = 1;
    }
    cout << endl;
}
int main() {
    cin >> n;
    for (int i = 0; i < n; i++) {
        cin >> a[i];
        mx = max(mx, a[i]);
    }
    for (int i = 0; i < n; i++) {
        memset(vis, 0, sizeof(vis));
        bfs(a[i]);
    }
    return 0;
}
```

其实没有必要每次都调用memset重置vis数组，考虑用id作为搜索时是否访问过的标记，代码如下：

```cpp
nclude <bits/stdc++.h>
using namespace std;
int n, mx, a[100005], vis[100005];
void bfs(int c, int x) {
    queue<int> q;
    q.push(x); vis[x] = c;
    while (!q.empty()) {
        int t = q.front(); q.pop();
        cout << t << " ";
        if (2*t <= mx && vis[2*t] != c)
            q.push(2*t), vis[2*t] = c;
        if (t/2 >= 1  && vis[t/2] != c)
            q.push(t/2), vis[t/2] = c;
    }
    cout << endl;
}
int main() {
    cin >> n;
    for (int i = 0; i < n; i++) {
        cin >> a[i];
        mx = max(mx, a[i]);
    }
    for (int i = 0; i < n; i++)
        bfs(i+1, a[i]);
    return 0;
}
```

