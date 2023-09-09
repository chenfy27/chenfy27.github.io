---
layout: post
title: 三角形面积
category: 算法
tag:
---

### 问题描述

在二维平面上给定三个点的坐标，分别为A(xa,ya), B(xb,yb), C(xc,yc)，求出ABC组成的三角形的面积。

### 公式法

可以用海伦公式计算三角形的面积，设三边长分别为a, b, c，记半周长l=(a+b+c)/2，那么三角形的面积可表示为：s=sqrt(l\*(l-a)\*(l-b)\*(l-c))。

```cpp
#include <bits/stdc++.h>
using namespace std;
struct Point {
    double x, y;
};
double dist(Point A, Point B) {
    return sqrt((A.x-B.x) * (A.x-B.x) + (A.y-B.y) * (A.y-B.y));
}
double area(Point A, Point B, Point C) {
    double a = dist(A, B);
    double b = dist(A, C);
    double c = dist(B, C);
    double l = (a+b+c) / 2;
    double s = sqrt(l*(l-a)*(l-b)*(l-c));
    return s;
}
int main() {
    Point A, B, C;
    cin >> A.x >> A.y;
    cin >> B.x >> B.y;
    cin >> C.x >> C.y;
    cout << area(A, B, C) << endl;
    return 0;
}
```

### 叉积法

用向量AB叉乘向量AC，忽略方向，其大小的绝对值为三角形面积的2倍。

```cpp
#include <bits/stdc++.h>
using namespace std;
struct Point {
    double x, y;
};
typedef Point Vector;
double area(Point A, Point B, Point C) {
    Vector a{B.x-A.x, B.y-A.y};
    Vector b{C.x-A.x, C.y-A.y};
    double s = fabs(a.x * b.y - a.y * b.x) / 2;
    return s2;
}
int main() {
    Point A, B, C;
    cin >> A.x >> A.y;
    cin >> B.x >> B.y;
    cin >> C.x >> C.y;
    cout << area(A, B, C) << endl;
    return 0;
}
```
