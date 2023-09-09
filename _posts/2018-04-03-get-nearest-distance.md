---
layout: post
title: 点到线段的最短距离
category: 算法
tag:
---

### 问题描述

在二维平面上，给出三个点的坐标分别为A(ax,ay), B(bx,by), C(cx,cy)，求点A到线段BC的最短距离。

### 问题分析

从点A向直线BC作垂线，垂足可能在线段BC的左侧，在线段BC上，或者在线段BC的右侧三种情况，下面分别讨论。

- 垂足在线段BC左侧

此时角ABC为钝角，满足：

```
AC * AC > AB * AB + BC * BC
```

最短距离显然是AB。

- 垂足在线段BC右侧

与左侧正好对称，有：

```
AB * AB > AC * AC + BC * BC
```

最短距离为AC。

- 垂直在线段BC上

最短距离是三角形ABC以边BC的高，可通过海伦公式先求出面积，再求出高得到答案。

### 代码实现

```cpp
struct Point {
    double x, y;
};
const double eps = 1e-6;
double GetDistance(Point A, Point B) {
    return sqrt((A.x-B.x) * (A.x-B.x) + (A.y-B.y) * (A.y-B.y));
}
double GetNearest(Point A, Point B, Point C) {
    double a = GetDistance(A, B);
    double b = GetDistance(A, C);
    double c = GetDistance(B, C);
    if (a*a > b*b + c*c)
        return b;
    if (b*b > a*a + c*c)
        return a;
    double l = (a+b+c) / 2;
    double s = sqrt(l*(l-a)*(l-b)*(l-c));
    return 2*s/c;
}
```
