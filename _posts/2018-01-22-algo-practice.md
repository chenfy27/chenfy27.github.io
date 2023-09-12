---
layout: post
title: 算法练习1801
category: 算法
tag:
---

### [CF371C](http://codeforces.com/contest/371/problem/C) Hamburgers

题面：已知做汉堡要B,S,C三种原料，单价分别为pb,ps,pc，现给出食谱，并已经有原料数量分别为nb,ns,nc和钱r，问最多可以做多少个汉堡？

分析：汉堡越多，需要的钱不会更少，存在单调性，可用二分搜索，注意确定好上下界，避免溢出。

```
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
LL b, s, c, nb, ns, nc, pb, ps, pc, r;
bool chk(LL x) {
    return r >= pb * max(b * x - nb, 0LL) + ps * max(s * x - ns, 0LL) + pc * max(c * x - nc, 0LL);
}
int main() {
    string str;
    cin >> str;
    for (auto i : str) {
        if (i == 'B') b++;
        if (i == 'S') s++;
        if (i == 'C') c++;
    }
    cin >> nb >> ns >> nc >> pb >> ps >> pc >> r;
    LL lo = 0, hi = 2e12, mid;
    while (lo < hi) {
        mid = lo + (hi - lo + 1) / 2;
        if (chk(mid))
            lo = mid;
        else
            hi = mid - 1;
    }
    cout << lo << endl;
    return 0;
}
```

### [CF915C](http://codeforces.com/contest/915/problem/C) Permute Digits

题面：给定数字a和b，1<=a,b<=1e18，求a的一个排列构成的数X，使得X在满足不超过b的前提下尽可能得大，输入保证解一定存在。

分析：可以从左到右，从小到大依次确定各位数，不断更新答案。

```
#include <bits/stdc++.h>
using namespace std;
string a;
long long b;
int main() {
    cin >> a >> b;
    sort(a.begin(), a.end());
    for (int i = 0; i < a.length(); i++) {
        for (int j = i + 1; j < a.length(); j++) {
            string t = a;
            swap(t[i], t[j]);
            if (stoll(t) > stoll(a) && stoll(t) <= b)
                swap(a[i], a[j]);
        }
    }
    cout << a << endl;
    return 0;
}
```

也可以从左到右，从大到小搜索答案，找到的第一个即为所求。

```
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
LL a, b, cnt[10], n, p[20];
void dfs(int x, LL y) {
    if (x == 0) {
        cout << y << endl;
        exit(0);
    }
    for (int i = 9; i >= 0; i--) {
        if (cnt[i] && y + i * p[x] <= b) {
            cnt[i] -= 1;
            dfs(x - 1, y + i * p[x]);
            cnt[i] += 1;
        }
    }
}
int main() {
    cin >> a >> b;
    while (a) {
        n += 1;
        cnt[a%10] += 1;
        a /= 10;
    }
    p[1] = 1;
    for (int i = 2; i <= n; i++)
        p[i] = p[i-1] * 10;
    dfs(n, 0);
    return 0;
}
```

### [UVA-12333](https://cn.vjudge.net/problem/UVA-12333) Revenge of Fibonacci

题面：给出一个不超过40位的数字a，问fib数列中以a为前缀的项最小序号是多少？如果前10w都不存在，输出-1。

分析：由于询问很多，先预处理出前10w项，考虑到前缀最长只有40位，可以只计算每一项的前几位，为保证准确性，可以多算几位，比如前50位，利用滚动数组优化空间，前缀查询可通过trie高效实现。

```
#include <bits/stdc++.h>
using namespace std;
struct Node {
    int val;
    struct Node *next[10];
}root;
int T, f[2][21000], L, R;
void insert(int idx, string &s) {
    Node *p = &root;
    for (auto i : s) {
        int d = i - '0';
        if (p->next[d] == NULL) {
            p->next[d] = new Node();
            p->next[d]->val = idx;
        }
        p = p->next[d];
    }
}
int query(string &s) {
    if (s == "1") return 0;
    Node *p = &root;
    for (auto i : s) {
        int d = i - '0';
        if (p->next[d] == NULL)
            return -1;
        p = p->next[d];
    }
    return p->val ? p->val : -1;
}
int main() {
    f[0][0] = f[1][0] = 1;
    L = R = 0;
    for (int i = 2, j = 0; i < 100000; i++, j = !j) {
        for (int k = L; k <= R; k++) {
            f[j][k] += f[!j][k];
            if (f[j][k] > 9) {
                f[j][k] -= 10;
                f[j][k+1] += 1;
            }
        }
        if (f[j][R+1]) R += 1;
        if (R - L > 60) L += 1;
        string k;
        for (int u = R, v = 0; u >= L && v < 40; u--, v++)
            k += f[j][u] + '0';
        insert(i, k);
    }
    ios::sync_with_stdio(0);
    cin >> T;
    for (int i = 1; i <= T; i++) {
        string k;
        cin >> k;
        cout << "Case #" << i << ": " << query(k) << endl;
    }
    return 0;
}
```

### [CF919B](http://codeforces.com/contest/919/problem/B) Perfect Number

题面：称各位数之和为10的数为完美数，求第k(1<=k<=10000)小的完美数。

分析：由于k范围较小，可以直接暴力枚举。

```
#include <bits/stdc++.h>
using namespace std;
bool chk(int x) {
    int sum = 0;
    while (x) {
        sum += x % 10;
        x /= 10;
    }
    return sum == 10;
}
int main() {
    int n, k;
    cin >> k;
    for (n = 1; k; n++)
        if (chk(n)) k--;
    cout << n - 1 << endl;
}
```

如果k的范围变大，可考虑dfs构造完美数的方法。


```
#include <bits/stdc++.h>
using namespace std;
vector<int> ans;
void dfs(int r, int n) {
    if (r == 0) {
        ans.push_back(n);
        return;
    }
    for (int i = 0; i < 10; i++) {
        if (i == 0 && n == 0) continue;
        if (i <= r) dfs(r - i, n * 10 + i);
    }
}
int main() {
    dfs(10, 0);
    sort(ans.begin(), ans.end());
    int k;
    cin >> k;
    cout << ans[k-1] << endl;
    return 0;
}
```

### [CF816B](http://codeforces.com/problemset/problem/816/B) Karen and Coffee

题面：咖啡有n种原料，已知煮咖啡时每种原料的温度要求范围l[i]~r[i]，对于某个特定的温度，如果它包含于至少k种原料的温度范围内，则认为该温度是可接受的，给定k，现有多组询问(a,b)，问在温度范围a~b之间有多少个可接受温度？温度都是正整数。

分析：先插线，然后求前缀和得到各个温度的覆盖数，根据该值与k的大小关系判断此温度是否为可接受温度，然后再做前缀和预处理，根据前缀和可在常数时间内回答各个询问。

```
#include <bits/stdc++.h>
using namespace std;
const int N = 200000;
int n, k, q, a, b, l, r, f[N+5];
int main() {
    ios::sync_with_stdio(0);
    cin >> n >> k >> q;
    for (int i = 0; i < n; i++) {
        cin >> l >> r;
        f[l]++, f[r+1]--;
    }
    for (int i = 1; i <= N; i++) f[i] += f[i-1];
    for (int i = 1; i <= N; i++) f[i] = f[i] >= k;
    for (int i = 1; i <= N; i++) f[i] += f[i-1];
    for (int i = 0; i < q; i++) {
        cin >> a >> b;
        cout << f[b] - f[a-1] << endl;
    }
    return 0;
}
```

### [CF920F](http://codeforces.com/contest/920/problem/F) Sum and Replace

题面：记d(x)表示正整数x的约数个数，给定初始数组a[n]，有两种操作：

- REPLACE l r：对于区间[l, r]中的每个元素a[i]，把它替换成d(a[i])
- SUM l r：求区间[l, r]中元素之和

分析：可以先用筛选法预处理出所有d[n]，然后就是个插线问线问题，由于要求使用在线算法，考虑用线段树，但问题关键在于如何快速更新。经过观察发现，对于正整数n，执行很少次的n=d[n]操作便会得到n=1或n=2，后续再执行n=d[n]不会有作何变化，利用该特性可以通过维护区间最大值进行优化。

```
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
const int N = 300005;
int n, m, a[N+5], d[1000005];
LL sum[4*N], mx[4*N];
void init() {
    for (int i = 1; i <= 1e6; i++) {
        for (int j = i; j <= 1e6; j += i)
            d[j] += 1;
    }
}
void pushup(int x) {
    sum[x] = sum[2*x] + sum[2*x+1];
    mx[x] = max(mx[2*x], mx[2*x+1]);
}
void build(int x, int l, int r) {
    if (l == r) {
        sum[x] = mx[x] = a[l];
        return;
    }
    int mid = (l + r) / 2;
    build(2*x, l, mid);
    build(2*x+1, mid+1, r);
    pushup(x);
}
void update(int x, int l, int r, int L, int R) {
    if (R < l || r < L) return;
    if (L <= l && r <= R && mx[x] < 3) return; // core optimization
    if (l == r) {
        a[l] = d[a[l]];
        sum[x] = mx[x] = a[l];
        return;
    }
    int mid = (l + r) / 2;
    update(2*x, l, mid, L, R);
    update(2*x+1, mid+1, r, L, R);
    pushup(x);
}
LL query(int x, int l, int r, int L, int R) {
    if (R < l || r < L) return 0;
    if (L <= l && r <= R) return sum[x];
    int mid = (l + r) / 2;
    return query(2*x, l, mid, L, R) + query(2*x+1, mid+1, r, L, R);
}
int main() {
    init();
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++)
        scanf("%d", &a[i]);
    build(1, 1, n);
    for (int i = 0; i < m; i++) {
        int t, l, r;
        scanf("%d%d%d", &t, &l, &r);
        if (t == 1)
            update(1, 1, n, l, r);
        else
            printf("%lld\n", query(1, 1, n, l, r));
    }
    return 0;
}
```
