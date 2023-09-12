---
layout: post
title: 算法练习1802
category: 算法
tag:
---

### [hiho1185](http://hihocoder.com/problemset/problem/1185) 连通性三之强连通分量

题面：有n个草场，每个草场有w[i]的牧草，草场之间有m条单向路径，最初羊群在1号草场，当吃完一个草场的草后，可继续前往其他草场，问最多能吃掉多少草？

分析：由于图可能存在环，不能直接dfs，考虑先求原图的强连通分量，将各连通分量缩成点，再用这些点构成新图，这样新图不存在环，可以用dfs求。

```
#include <bits/stdc++.h>
using namespace std;
const int N = 20000;
int n, m, w[N+5], vis[N+5], nscc, scc[N+5], id[N+5];
vector<int> g[N+5], G[N+5], g1[N+5], seq;
void dfs1(int x) {
    vis[x] = 1;
    for (auto i : g[x])
        if (!vis[i]) dfs1(i);
    seq.push_back(x);
}
void dfs2(int x) {
    id[x] = nscc;
    scc[nscc] += w[x];
    for (auto i : G[x])
        if (!id[i]) dfs2(i);
}
void find_scc() {
    for (int i = 1; i <= n; i++)
        if (!vis[i]) dfs1(i);
    for (int i = n-1; i >= 0; i--) {
        if (!id[seq[i]]) {
            nscc += 1;
            dfs2(seq[i]);
        }
    }
}
set<pair<int,int>> st;
void add_edge(int a, int b) {
    int u = id[a], v = id[b];
    if (u == v || st.count(make_pair(u, v)))
        return;
    g1[u].push_back(v);
}
void new_graph() {
    for (int i = 1; i <= n; i++)
        for (auto j : g[i])
            add_edge(i, j);
}
int ans = 0;
void dfs(int x, int v) {
    v += scc[x];
    ans = max(ans, v);
    for (auto i : g1[x])
        dfs(i, v);
}
int main() {
    cin >> n >> m;
    for (int i = 1; i <= n; i++)
        cin >> w[i];
    for (int i = 0; i < m; i++) {
        int a, b;
        cin >> a >> b;
        g[a].push_back(b);
        G[b].push_back(a);
    }
    find_scc();
    new_graph();
    dfs(id[1], 0);
    cout << ans << endl;
    return 0;
}
```

### [Word Ordering](https://csacademy.com/contest/beta-round-1/task/word_ordering/)

题面：重新定义字母a-z的大小关系，并且大写字母比所有小写字母大，给出一些字符串，要求按新的字典序输出。

分析：根据新的字典序定义比较函数，传给sort函数即可。

```
#include <bits/stdc++.h>
using namespace std;
map<int,int> mp;
bool cmp(string a, string b) {
    for (auto i = 0; i < a.length() && i < b.length(); i++)
        if (a[i] != b[i]) return mp[a[i]] < mp[b[i]];
    return a.length() < b.length();
}
int main()
{
    int n; string s;
    cin >> s >> n;
    for (int i = 0; i < 26; i++) {
        mp[s[i]] = i;
        mp[s[i]-32] = i+26;
    }
    vector<string> v;
    for (int i = 0; i < n; i++) {
        cin >> s;
        v.push_back(s);
    }
    sort(v.begin(), v.end(), cmp);
    for (auto i : v) cout << i << endl;
    return 0;
}
```

### [Sorting Partition](https://csacademy.com/contest/beta-round-1/task/sorting_partition/)

题面：给出一个整数数组，要求将其划分为多个子数组，使得将各个子数组排序后整个数组为有序，求最大子数组数。

分析：从左往右扫，记到当前元素为止的最大值amax，如果后续存在比amax小的元素，则不可在此划分，应继续，否则截成一个子数组，因而需要预处理前缀最大值和后缀最小值。

```
#include <bits/stdc++.h>
using namespace std;
int n, a[100005], pre[100005], suf[100005];
int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++)
        scanf("%d", &a[i]);
    pre[1] = a[1];
    for (int i = 2; i <= n; i++)
        pre[i] = max(a[i], pre[i-1]);
    suf[n] = a[n];
    for (int i = n-1; i > 0; i--)
        suf[i] = min(a[i], suf[i+1]);
    int ans = 0;
    for (int i = 1; i <= n; i++) {
        ans += 1;
        while (i < n && pre[i] > suf[i+1])
            i += 1;
    }
    printf("%d\n", ans);
    return 0;
}
```

### [CF920G](http://codeforces.com/contest/920/problem/G) List Of Integers

题面：记L(x,p)表示序列{y}，满足y>x且gcd(y,p)=1，现有多组询问，每个询问给出x, p, k，求L(x,p)的第k个元素。

分析：该问题可拆分成几个小问题。

- 1~n中能被a整除的数有多少个？

答案是n/a。

- 1~n中与a互质的数有多少个呢？

可以对a做因式分解，得到各个不同的质因子，再根据容斥原理进行统计。

- 如何求第k个元素？

设A(p,n)表示1~n中与p互质的数的个数，显然n越大，A(n,p)也就越大，因此具有单调性。对于L(x,p)，如果不考虑大于x的条件，那么这是一个从1开始的序列，加上大于x的限制相当于选择了新的起点，设第k个元素为ans，那么必有A(ans,p)=A(x,p)+k，且ans最小，可用二分搜索求解。

```
#include <bits/stdc++.h>
using namespace std;
int nf, f[10];
void fact(int n) {
    nf = 0;
    for (int i = 2; i * i <= n; i++) {
        if (n % i == 0) {
            f[nf++] = i;
            while (n % i == 0)
                n /= i;
        }
    }
    if (n > 1) f[nf++] = n;
}
void dfs(int x, int pos, int times, int &cnt) {
    if (pos >= nf) {
        if (times & 1)
            cnt -= x;
        else
            cnt += x;
        return;
    }
    dfs(x, pos+1, times, cnt);
    dfs(x/f[pos], pos+1, times+1, cnt);
}
int getcnt(int x) {
    int cnt = 0;
    dfs(x, 0, 0, cnt);
    return cnt;
}
int go(int x, int p, int k) {
    fact(p);
    int target = k + getcnt(x);
    int lo = 0, hi = 1e9, mid;
    while (lo < hi) {
        mid = lo + (hi - lo) / 2;
        int cnt = getcnt(mid);
        if (cnt < target)
            lo = mid + 1;
        else
            hi = mid;
    }
    return lo;
}
int main() {
    int t, x, p, k;
    scanf("%d", &t);
    while (t--) {
        scanf("%d%d%d", &x, &p, &k);
        printf("%d\n", go(x, p, k));
    }
    return 0;
}
```
