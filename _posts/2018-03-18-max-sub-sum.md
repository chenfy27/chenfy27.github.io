---
layout: post
title: 最大子段和问题
category: 算法
tag:
---

### [51nod1049](http://www.51nod.com/onlineJudge/questionCode.html#!problemId=1049) 最大子段和

题面：给定整数数组a[n]，求它的连续子段和的最大值，当所有整数均为负数为时和为0。

分析：设dp[i]表示以数字a[i]结尾的子段的和的最大值，那么dp[i] = max(a[i], a[i]+dp[i-1])。

```
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
int a[50005], n;
LL ans, dp[50005];
int main() {
    cin >> n;
    for (int i = 1; i <= n; i++)
        cin >> a[i];
    for (int i = 1; i <= n; i++) {
        dp[i] = max(dp[i-1] + a[i], (LL)a[i]);
        ans = max(ans, dp[i]);
    }
    cout << ans << endl;
    return 0;
}
```

### [51nod1050](http://www.51nod.com/onlineJudge/questionCode.html#!problemId=1050) 循环数组最大子段和

题面：给定整数构成的循环数组a[n]，求它的连续子段和的最大值，当所有整数均为负数时和为0。

分析：如果取到最优值时没有跨过首尾元素，那么可当成普通数组的最大子段和来做；反之，可按普通数组求最小子段和，再用总和减去最小子段和即可。

```
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
LL n, a[50005], dp1[50005], dp2[50005];
LL ans1, ans2, sum;
int main() {
    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
        sum += a[i];
    }
    for (int i = 1; i <= n; i++) {
        dp1[i] = max(0LL, max(a[i], a[i]+dp1[i-1]));
        ans1 = max(ans1, dp1[i]);
        dp2[i] = min(0LL, min(a[i], a[i]+dp2[i-1]));
        ans2 = min(ans2, dp2[i]);
    }
    cout << max(ans1, sum-ans2) << endl;
    return 0;
}
```

### [51nod1065](http://www.51nod.com/onlineJudge/questionCode.html#!problemId=1065) 最小正子段和

题面：给定整数数组a[n]，从中选出一个子序列，要求该序列的和大于0，且最小。

分析：预处理出前缀和，二分查找。

```
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
LL n, a[50005], p[50005], ans = LLONG_MAX;
map<LL,LL> mp;
int main() {
    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
        p[i] = a[i] + p[i-1];
        mp[p[i]] = i;
    }
    for (int i = 1; i <= n; i++) {
        auto it = mp.upper_bound(p[i-1]);
        if (it != mp.end() && it->second >= i) {
            ans = min(ans, it->first - p[i-1]);
        }
    }
    cout << ans << endl;
    return 0;
}
```

### [51nod1051](http://www.51nod.com/onlineJudge/questionCode.html#!problemId=1051) 最大子矩阵和

题面：给出m\*n的矩阵，找一个子矩阵，要求该子矩阵的元素之和最大。

分析：枚举子矩阵的行上下边界，固定行后求出各列元素之和，转化为一维的最大子段和问题可得到最优解的列范围。可预处理出各列的前缀和，便于快速算出指定行范围内各列元素之和。

```
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
LL m, n, a[505][505], d[505][505], x[505], dp[505], ans;
int main() {
    cin >> m >> n;
    for (int i = 1; i <= n; i++)
    for (int j = 1; j <= m; j++)
        cin >> a[i][j];
    for (int j = 1; j <= m; j++)
    for (int i = 1; i <= n; i++)
            d[i][j] = a[i][j] + d[i-1][j];
    for (int i = 1; i <= n; i++)
    for (int j = i; j <= n; j++) {
        for (int k = 1; k <= m; k++)
            x[k] = d[j][k] - d[i-1][k];
        for (int k = 1; k <= m; k++) {
            dp[k] = max(x[k], x[k] + dp[k-1]);
            ans = max(ans, dp[k]);
        }
    }
    cout << ans << endl;
    return 0;
}
```
