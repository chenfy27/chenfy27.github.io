---
layout: post
title: 三分搜索
category: 算法
tag:
---

二分搜索一般用于寻找正好满足某种条件的临界值，要求搜索区间满足单调性，而三分搜索一般是在凸函数或者凹函数上寻找极值。

### 原理

假设函数图像在区间[lo,hi]是开口向上的曲线，那么区间内必有极大值，取三分点m1和m2，若f(m1)>f(m2)，那么极大值位于[lo,m2]内，排除区间[m2,hi]；反之极大值区于[m1,hi]内，可排除[lo,m1]，时间复杂度为O(logn)。

### [51nod1629](http://www.51nod.com/onlineJudge/questionCode.html#!problemId=1629) B君的圆锥

题面：已知圆锥的表面积为S，求它的体积V的最大值。

分析：设圆锥半径为r，由于表面积S固定，直观上看，当r很小时圆锥虽然高，但太瘦，导致体积小；当r很大时圆锥胖，但太矮，体积也不大；应取合适的r值，使宽度和高度平衡，方能取到最大体积。

```cpp
#include <bits/stdc++.h>
using namespace std;
const double pi = acos(-1);
int s;
double getl(double r) {
    return (s - pi * r * r) / pi / r;
}
double geth(double r) {
    double l = getl(r);
    return sqrt(l * l - r * r);
}
double vol(double r) {
    double h = geth(r);
    return pi * r * r * h / 3;
}
int main() {
    cin >> s;
    double lo = 0, hi = sqrt(s/2/pi);
    while (hi - lo > 1e-6) {
        double m1 = lo + (hi - lo) / 3;
        double m2 = lo + (hi - lo) / 3 * 2;
        double v1 = vol(m1), v2 = vol(m2);
        if (v1 > v2)
            hi = m2;
        else
            lo = m1;
    }
    printf("%.6f\n", vol(lo));
    return 0;
}
```

### [奶牛的聚会](https://nanti.jisuanke.com/t/10829)

题面：在x轴上有n个点，给定第i个点的坐标x[i]和权值w[i]，需要在x轴上选一点d，使得sum(w[i]\*\|d-x[i]\|^3)最小，求该最小值。

分析：d过于偏左或者偏右都会导致结果太大，应该在中间某处取，使得左右两边的带权距离平衡，以获得最小值。

```cpp
#include <bits/stdc++.h>
using namespace std;
int n, T;
double x[50005], w[50005];
double f(double d) {
    double ret = 0;
    for (int i = 1; i <= n; i++) {
        double u = fabs(d - x[i]);
        ret += u * u * u * w[i];
    }
    return ret;
}
int main() {
    cin >> T;
    for (int z = 1; z <= T; z++) {
        double lo = 1e6, hi = -1e6, ans = 9e18;
        cin >> n;
        for (int i = 1; i <= n; i++) {
            cin >> x[i] >> w[i];
            lo = min(lo, x[i]);
            hi = max(hi, x[i]);
        }
        if (n == 1) {
            ans = 0;
        } else {
            while (hi - lo > 1e-6) {
                double m1 = lo + (hi - lo) / 3;
                double m2 = lo + (hi - lo) / 3 * 2;
                double v1 = f(m1), v2 = f(m2);
                if (v1 < v2)
                    hi = m2, ans = v1;
                else
                    lo = m1, ans = v2;
            }
        }
        printf("Case #%d: %d\n", z, int(ans + 0.5));
    }
    return 0;
}
```

### 在整数内搜索

上面两个例子都是在实数范围内做三分搜索，当搜索区间长度小于指定精度时则停止。

如果要求取值必须为整数，三分搜索同样可以工作，区别在于循环终止时的处理：当lo+2等于hi时停止搜索，此时m1=m2，且lo,m1,hi是三个相邻的整数，分别计算函数在这三个点的取值，取三者的极值即可。
