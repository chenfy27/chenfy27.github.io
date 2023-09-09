---
layout: post
title: 矩阵快速幂
category: 算法
keywords:
---

计算矩阵的幂可以用类似计算数字幂的倍增法，在对数时间内算出结果，这在解决很多线性递推或者二次递推问题时很有用，关键在于根据递推式构造合适的矩阵。

### [51nod1126](http://www.51nod.com/onlineJudge/questionCode.html#!problemId=1126) 求递推序列的第n项

题面：定义序列f(1)=1, f(2)=1, f(n)=(A\*f(n-1)+B\*f(n-2))%7，给出A和B，求f(n)，其中1<=n<=1e9。

分析：由于n较大，直接递推太慢，考虑构造矩阵通过快速幂来实现。需要注意的是，第1项f(1)不能通过递推来算，只能特判，但这里正好与结果相符，因而可以不用单独处理。

```cpp
#include <bits/stdc++.h>
using namespace std;
struct matrix {int d[2][2];};
matrix mul(matrix a, matrix b) {
    matrix c;
    for (int i = 0; i < 2; i++)
    for (int j = 0; j < 2; j++) {
        int z = 0;
        for (int k = 0; k < 2; k++)
            z = (z + a.d[i][k] * b.d[k][j]) % 7;
        c.d[i][j] = z;
    }
    return c;
}
int main() {
    int n, a, b;
    cin >> a >> b >> n;
    matrix t = {a, b, 1, 0};
    matrix u = {1, 0, 0, 1};
    n -= 1;
    while (n) {
        if (n & 1) u = mul(u, t);
        t = mul(t, t);
        n /= 2;
    }
    int ans = (u.d[1][0] + u.d[1][1]) % 7;
    if (ans < 0) ans += 7;
    cout << ans << endl;
    return 0;
}
```

### [51nod1113](http://www.51nod.com/onlineJudge/questionCode.html#!problemId=1113) 矩阵快速幂

题面：给出一个n\*n的方阵，元素均为正整数，求方阵的m次方，结果中各元素对1e9+7取模。

分析：矩阵快速幂模板题。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
struct matrix {LL d[105][105];};
LL n, m, mod = 1e9+7;
matrix mul(matrix a, matrix b) {
    matrix c;
    for (LL i = 1; i <= n; i++) {
        for (LL j = 1; j <= n; j++) {
            LL z = 0;
            for (LL k = 1; k <= n; k++)
                z = (z + a.d[i][k] * b.d[k][j]) % mod;
            c.d[i][j] = z;
        }
    }
    return c;
}
int main() {
    cin >> n >> m;
    matrix r, t;
    for (LL i = 1; i <= n; i++)
    for (LL j = 1; j <= n; j++)
        r.d[i][j] = i == j ? 1 : 0;
    for (LL i = 1; i <= n; i++)
    for (LL j = 1; j <= n; j++)
        cin >> t.d[i][j];
    for (LL b = m; b; b /= 2) {
        if (b & 1) r = mul(r, t);
        t = mul(t, t);
    }
    for (LL i = 1; i <= n; i++) {
        for (LL j = 1; j <= n; j++)
            cout << r.d[i][j] << " ";
        cout << endl;
    }
    return 0;
}
```
