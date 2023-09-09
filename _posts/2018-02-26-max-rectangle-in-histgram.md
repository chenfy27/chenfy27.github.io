---
layout: post
title: 直方图中最大矩形面积
category: 算法
tag:
---

### [51nod1102](http://www.51nod.com/onlineJudge/questionCode.html#!problemId=1102) 面积最大的矩形

题面：给定一个直方图，在图中找出一个矩形，使其面积最大，求最大的面积。

分析：这是道比较有趣的面试题，朴素做法是枚举每个点，以其高度作为矩形的高度，并尽可能地往左右两端延伸，得到最大面积，时间复杂度为O(n^2)，效率不够。网上有通过单调栈的O(n)做法，但不太容易理解。下面这种做法是在朴素做法的基础上优化，先预处理出每个高度能到达的最左最右边界，然后扫一遍得到最优解，总时间复杂度也为O(n)，而单调栈的做法其实是对该做法的再次优化，减少遍历次数。

记h[x]表示坐标为x处的高度，l[x]表示x能往左延伸的最小值，但不包含在内，即h[l[x]] < h[x]，同理，r[x]表示x能往右延伸的最大值，那么在预处理时可用dp的思想跳跃式进行。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
int n, h[50005], l[50005], r[50005];
int main() {
    cin >> n;
    for (int i = 1; i <= n; i++)
        cin >> h[i];
    l[1] = 0;
    for (int i = 2; i <= n; i++) {
        int j = i-1;
        while (h[j] >= h[i])
            j = l[j];
        l[i] = j;
    }
    r[n] = n+1;
    for (int i = n-1; i >= 1; i--) {
        int j = i+1;
        while (h[j] >= h[i])
            j = r[j];
        r[i] = j;
    }
    LL ans = 0;
    for (int i = 1; i <= n; i++)
        ans = max(ans, (LL)h[i] * (r[i]-l[i]-1));
    cout << ans << endl;
    return 0;
}
```
