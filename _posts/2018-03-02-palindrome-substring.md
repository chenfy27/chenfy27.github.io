---
layout: post
title: 回文子串问题
category: 算法
tag:
---

本文整理一些与字符串回文相关的问题。

### 判断子串回文

给定一个字符串s，然后有多组询问，每次给出区间[l,r]，问子串s[a..b]是否为回文串。

由于询问有多次，可以考虑预处理出所有子串是否为回文，然后依次回答询问即可。有两种做法：区间dp和哈希，下面分别介绍。

#### 区间dp做法

区间dp一般是按区间长度由短往长递推，由于回文分奇回文和偶回文，对应边界情况分别是长度为1和2的子串，预处理的时间复杂度为O(n^2)。

```cpp
#include <bits/stdc++.h>
using namespace std;
char s[10005];
int dp[10005][10005], l, q, a, b;
int main() {
    cin >> s; l = strlen(s);
    for (int i = 0; i < l; i++)
        dp[i][i] = 1;
    for (int i = 1; i < l; i++)
        dp[i-1][i] = s[i-1] == s[i];
    for (int k = 2; k < l; k++) {
        for (int i = 0; i < l; i++) {
            int j = i+k;
            if (j >= l) break;
            if (s[i] != s[j]) dp[i][j] = 0;
            else dp[i][j] = dp[i+1][j-1];
        }
    }
    cin >> q;
    for (int i = 0; i < q; i++) {
        cin >> a >> b;
        cout << (dp[a-1][b-1] ? "Yes" : "No") << endl;
    }
    return 0;
}
```

#### 哈希做法

分别采用左乘和右乘的方式计算哈希，如果是回文串，哈希值必定相同，非回文串的哈希值一般不相等。由于右乘方式要计算子区间哈希值时要用到除法，而取余操作不能直接除，故而先打表求幂时，顺带把它关于模的乘法逆元也算出来，预处理的时间复杂度为O(nlogn)。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
char s[10005];
int p = 131, q, a, b, l, M = 1e9+7;
LL L[10005], R[10005], H[10005], V[10005];
LL LH(int x, int y) {
    LL h = (L[y] - L[x-1] * H[y-x+1]) % M;
    if (h < 0) h += M;
    return h;
}
LL RH(int x, int y) {
    LL h = (R[y] - R[x-1]) * V[x] % M;
    if (h < 0) h += M;
    return h;
}
void exgcd(LL u, LL &x, LL v, LL &y) {
    if (v == 0) {
        x = 1; y = 0;
    } else {
        exgcd(v, y, u%v, x);
        y -= u/v*x;
    }
}
LL inv(LL x, LL y) {
    LL xx, yy;
    exgcd(x, xx, y, yy);
    if (xx < 0) xx += y;
    return xx;
}
void init() {
    H[0] = 1;
    for (int i = 1; i <= l; i++) {
        H[i] = H[i-1] * p % M;
        V[i] = inv(H[i], M);
    }
}
int main() {
    cin >> (s+1); l = strlen(s+1);
    init();
    for (int i = 1; i <= l; i++) {
        L[i] = (L[i-1] * p + s[i]) % M;
        R[i] = (R[i-1] + H[i] * s[i]) % M;
    }
    cin >> q;
    for (int i = 0; i < q; i++) {
        cin >> a >> b;
        cout << (LH(a,b) == RH(a,b) ? "Yes" : "No") << endl;
    }
    return 0;
}
```

### [51nod1092](http://www.51nod.com/onlineJudge/questionCode.html#!problemId=1092) 回文字符串

题面：给出一个长度不超过1000的字符串s，问最少要插入多少个字符可使其变成回文串？

分析：记原串为s，其逆序为s1，求出s与s1的最长公共子序列lcs(s,s1)的长度len(lcs)，那么答案就是len(s)-len(lcs)。

### [51nod1154](http://www.51nod.com/onlineJudge/questionCode.html#!problemId=1154) 回文串划分

题面：给出一个长度不超过5000的字符串s，现要对其进行划分，使划分出的每个子串都是回文串，问最少可划分为几段？

分析：设dp[i]表示前i个字符可划分的最少段数，在求dp[i+1]时，依次枚举以s[i+1]结尾的子串，长度从1递增，检查子串是否为回文，并更新dp[i+1]。

```cpp
#include <bits/stdc++.h>
using namespace std;
char s[5005]; int dp[5005];
int ok(int x, int y) {
    while (x < y) {
        if (s[x] != s[y]) return 0;
        x++, y--;
    }
    return 1;
}
int main() {
    cin >> (s+1); int l = strlen(s+1);
    for (int i = 1; i <= l; i++) {
        dp[i] = 1 + dp[i-1];
        for (int j = i-1; j >= 1; j--)
            if (ok(j,i)) dp[i] = min(dp[i], 1+dp[j-1]);
    }
    cout << dp[l] << endl;
    return 0;
}
```
