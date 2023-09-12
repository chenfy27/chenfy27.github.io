---
layout: post
title: 算法练习1707
category: 算法
keywords:
---

### [51nod2006](http://www.51nod.com/onlineJudge/questionCode.html#!problemId=2006) 飞行员配对

题面：有a个A类飞行员和b个B类飞行员，每架飞机上必须有能相互配合的A类飞行员和B类飞行员各1个，给出各飞行员之间的配合关系，求最佳飞行员配对方案，使一次派出的飞行数最多。

思路：裸的二分图最大匹配。

### [51nod1873](http://www.51nod.com/onlineJudge/questionCode.html#!problemId=1873) 初中算术

题面：给出a(0.0<a<99.999)和n(0<n<26)，求a^n的精确值。

思路1：python的整数运算是精确的，可以先将a转为整数，求幂后再确定小数点的位置输出。

思路2：用python的decimal模块直接计算，但输出时如果有前导0，默认是以科学计数法表示的，需要处理成小数表示，不知是否有现成的方法可以转。

### [cf817c](http://codeforces.com/problemset/problem/817/C) Really Big Numbers

题面：给定一个数D，对于整数X，如果X与它的各位数之和的差不小于D，则称X为大数。现给定N和D，求1~N中大数的个数。

思路：显然，如果X是大数，那么X+1也是大数，因此对于给定的D，需要找到一个数A，满足A为大数，但A-1不是大数，由于单调性，可用二分搜索。

### [hiho1335](http://hihocoder.com/problemset/problem/1335) Email Merge

题面：有一列名单，每一条名单中包含用户名和对应的邮箱(可能有多个)，如果有两条记录使用了同一个邮箱，则这两个用户名指的是同一个人。要求按输入顺序输出所有人，同一个人的用户名在一行输出，按输入顺序显示。

思路：考虑到字符串不能作下标，用个map建立用户名与下标的对应关系，这样方便处理。如果两个用户名共用了一个邮箱，则在这两点间连一条边。最后按下标顺序做DFS，找出所有相连的点，按下标排序输出即可。

### [hiho1330](http://hihocoder.com/problemset/problem/1330) 重排数组

题面：给定1~N的一个排列P[N]和初始数组A[1]=1, A[2]=2,...A[N]=N，每次将第i个元素放到P[i]位置上，问至少要经过多少次操作可得到原始数组？

思路：分别计算每个元素回到原处需要的步数d[i]，取所有d[i]的最小公倍数即为答案。

### [hiho1523](http://hihocoder.com/problemset/problem/1523) 重排数组2

题面：给定1~N的一个排列，每次可任选一个数放到最左边，问至少需要多少次操作才可以使整个排列为升序？

思路：找规律，从大到小依次判断每个数是否需要操作，如果x需要放到最左边，那么1~x-1必然也需要放到最左边，因此只需找出第一个要被操作的数即可。

### [hiho1469](http://hihocoder.com/problemset/problem/1469) 福字

题面：给出n*n非数整数矩阵，找出最大的方阵，使得方阵中除第1行和第1列外其余元素都满足a[x][y]=a[x-1][y]+1且a[x][y]=a[x][y-1]+1。

思路：设f[i][j]表示以a[i][j]为右下角的最大方阵的大小，当a[i][j]=a[i-1][j]+1且a[i][j]=a[i][j-1]+1时有f[i][j]=min(f[i-1][j],f[i][j-1])+1，否则f[i][j]=1。

### [hiho1482](http://hihocoder.com/problemset/problem/1482) 出勤记录2

题面：用OAL三个字符组成长度为N的字符串，要求最多出现一次A，并且不能连续出现三次L，问有多少种方法，结果对1e9+7取余。

思路：设f[i][j][k]表示字符串长度为i，已出现过j次A，末尾有k个连续L的方案数，需要计算f[i][0][0], f[i][0][1], f[i][0][2], f[i][1][0], f[i][1][1], f[i][1][2]，初始情况为i=1，针对这6个值，分别考虑第i个字符为OAL时对应的下一状态，得到递推关系，即可求解。

### [hiho1051](http://hihocoder.com/problemset/problem/1051) 补提交卡

题面：有100天签到记录，其中有n天没签，现有m张补签卡，每张补签卡可将一次未签到改成已签到，求可得到的最长连续签到天数。

思路：由于总长才100，最简单的方法是枚举所有区间。考虑到最长连续签到天数必是连续的一串，故而未签到的记录也是连续的，因此可枚举长度为m+1的未签到记录，以此作为连续签到的两端(不包含)，取最大值即可。

```
int t, n, m, ans, a[105];
int main() {
    scanf("%d", &t);
    while (t--) {
        scanf("%d%d", &n, &m);
        for (int i = 1; i <= n; i++)
            scanf("%d", &a[i]);
        ans = n <= m ? 100 : 0;
        for (int i = 0; i + m < n; i++)
            ans = max(ans, a[i+m+1] - a[i] - 1);
        printf("%d\n", ans);
    }
    return 0;
}
```

### [hiho1049](http://hihocoder.com/problemset/problem/1049) 后序遍历

题面：根据前序遍历和中序遍历序列，求后序遍历序列。

思路：两个简单的递归即可。

```
char pre[32], mid[32];
void dfs(int ll, int lr, int rl, int rr) {
    int cnt = 0;
    if (ll > lr || rl > rr) return;
    for (int i = rl; mid[i] != pre[ll]; i++, cnt++);
    dfs(ll + 1, ll + cnt, rl, rl + cnt - 1);
    dfs(ll + cnt + 1, lr, rl + cnt + 1, rr);
    printf("%c", pre[ll]);
}
int main() {
    scanf("%s%s", pre, mid);
    dfs(0, strlen(pre) - 1, 0, strlen(mid) - 1);
    return 0;
}
```

### [hiho1050](http://hihocoder.com/problemset/problem/1050) 树中的最长路

题面：求树中任意两点间距离的最大值。

思路：更一般的问题是求连通图的直径，做法是任选一点开始做bfs，然后以最后访问的点为起点再做一次bfs，第二次bfs的起终点即为端点，其距离为最大。

```
int n, x, y, d, vis[100005];
vector<int> g[100005];
int bfs(int u, int &v) {
    int a, b;
    queue<int> p, q;
    memset(vis, 0, sizeof(vis));
    p.push(u); q.push(0); vis[u] = 1;
    while (!p.empty()) {
        a = p.front(); p.pop();
        b = q.front(); q.pop();
        for (auto i : g[a]) {
            if (vis[i]) continue;
            p.push(i); q.push(b + 1); vis[i] = 1;
        }
    }
    v = a;
    return b;
}
int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) {
        scanf("%d%d", &x, &y);
        g[x].push_back(y);
        g[y].push_back(x);
    }
    bfs(1, x);
    printf("%d\n", bfs(x, y));
    return 0;
}
```

### [hiho1052](http://hihocoder.com/problemset/problem/1052) 基因工程

题面：给定一个由ACGT组成的字符串S和整数K(1<=K<=len(S))，记由S的前K个字符组成字符串为S1，后K个字符组成的字符串为S2，要使S1与S2相同，问最少要在S中修改多少个字符？

思路：S1第x个字符对应S2第x+len(S)-K的字符，如果len(S)-k的位置仍然属于S1，那么对应S2的字符位置为x+2(len(S)-k)...，一直递推下去，这些位置的字符都必须相同，可找出这些位置字符出现次数最多的，然后将出现少的改成出现多的即可。需要注意K等于len(S)的特例。

```
int t, k, l, vis[1005];
char s[1005];
void addchar(int x, int cnt[], int &most) {
    if (x >= l || vis[x]) return;
    vis[x] = 1;
    switch (s[x]) {
        case 'A': most = max(most, ++cnt[0]); break;
        case 'C': most = max(most, ++cnt[1]); break;
        case 'G': most = max(most, ++cnt[2]); break;
        case 'T': most = max(most, ++cnt[3]); break;
        default: break;
    }
}
int handle() {
    int ans = 0;
    if (l == k) return 0;
    for (int i = 0; i < k; i++) {
        if (vis[i]) continue;
        int cnt[4] = {0}, most = 0, tag;
        for (int u = i; u < k; u += l - k) {
            addchar(u, cnt, most);
            addchar(u + l - k, cnt, most);
        }
        for (tag = 0; tag < 4 && cnt[tag] != most; tag++);
        for (int z = 0; z < 4; z++)
            if (z != tag)
                ans += cnt[z];
    }
    return ans;
}
int main() {
    scanf("%d", &t);
    while (t--) {
        scanf("%s%d", s, &k);
        l = strlen(s);
        memset(vis, 0, sizeof(vis));
        printf("%d\n", handle());
    }
    return 0;
}
```

### [hiho1053](http://hihocoder.com/problemset/problem/1053) 居民迁移

题面：在X轴上有N个点，每个点上住有若干居民，现要进行迁移，要求迁移距离不能超过R，问迁移后所有据点居民数最大值最小可以是多少？

思路：二分答案，难点在于给定最大值X，如何判定该方案是否可行。可换个角度看，把点看成区间，把所有居民按X轴从左往右分配到各个据点中去。

```
struct st {
    int x, y;
    bool operator<(const st &o) const {
        return x < o.x;
    }
}p[100005], q[100005], pp[100005];
int t, n, r;
int chk(int v) {
    for (int i = 0; i < n; i++) {
        p[i] = pp[i];
        q[i].x = p[i].x;
        q[i].y = 0;
    }
    for (int i = 0, j = 0; i < n; i++) {
        if (j > n || p[i].x < q[j].x - r) return 0;
        if (p[i].x > q[j].x + r) {i--, j++; continue;}
        if (p[i].y > v - q[j].y) {
            p[i].y -= v - q[j].y;
            q[j].y = v;
            i--, j++;
        } else {
            q[j].y += p[i].y;
            p[i].y = 0;
        }
    }
    return 1;
}
int handle() {
    int lo = 0, hi = 1e9, mid;
    while (lo < hi) {
        mid = lo + (hi - lo) / 2;
        if (chk(mid))
            hi = mid;
        else
            lo = mid + 1;
    }
    return lo;
}
int main() {
    scanf("%d", &t);
    while (t--) {
        scanf("%d%d", &n, &r);
        for (int i = 0; i < n; i++)
            scanf("%d%d", &pp[i].x, &pp[i].y);
        sort(pp, pp + n);
        printf("%d\n", handle());
    }
    return 0;
}
```

### [cf616d](http://codeforces.com/contest/616/problem/D) Longest k-good segments

题面：给定数组a[n]和正整数K，求最长的子区间[l,r]，使得该子区间所包含不同数的个数不超过K。

思路：双指针扫一遍即可。


