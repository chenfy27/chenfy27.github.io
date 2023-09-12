---
layout: post
title: 高斯消元
category: 算法
keywords:
---

高斯消元法一般用于求解线性方程组，做法大致是对增广矩阵做初等行变换，先化为上三角矩阵，再化为对角矩阵，最后化为单位矩阵，那么最右一列的值即为解。

### 高斯消元一般流程

下面以三元一次方程为例进行说明。

```
x  +  y +  z = 26
x  -  y      = 1
2x -  y +  z = 18
```

对增广矩阵做初等行变换过程如下：

```
1   1   1   26
1  -1   0   1
2  -1   1   18
--------------------
1   1   1   26
0  -2  -1  -25
0  -3  -1  -34
--------------------
1   1   1   26
0  -2  -1  -25
0   0  0.5  3.5
--------------------
1   1   1   26
0  -2   0  -18
0   0  0.5  3.5
--------------------
1   1   1   26
0  -2   0  -18
0   0   1   7
--------------------
1   1   0   19 
0  -2   0  -18
0   0   1   7
--------------------
1   1   0   19 
0   1   0   9
0   0   1   7
--------------------
1   0   0   10
0   1   0   9
0   0   1   7
```

从而原方程组的解为：x=10, y=9, z=7。

### 无解与多解判定

- 如果在求上三角矩阵的过程中出现某行系数全为0，而值不为0，则方程组无解。
- 在有解的前提下，如果增广矩阵的秩小于未知数个数，则方程组有多解。

### [例题](http://hihocoder.com/problemset/problem/1195)与代码

```
#include <stdio.h>
#include <math.h>
#define EPS 1e-6
int n, m;
double x[1005][1005];
void swap(int u, int v) {
    int j; double t;
    for (j = 0; j < sizeof(x[j]) / sizeof(x[j][0]); j++)
        t = x[u][j], x[u][j] = x[v][j], x[v][j] = t;
}
int Gauss() {
    int i, j, k, r;
    int multians = 0;
    double z;
    for (k = 1; k <= n; k++) {
        r = k;
        for (i = k + 1; i <= m; i++) {
            if (fabs(x[i][k]) > fabs(x[r][k]))
                r = i;
        }
        if (r != k) swap(r, k);
        if (fabs(x[k][k]) < EPS) {
            multians = 1;
            continue;
        }
        for (i = k + 1; i <= m; i++) {
            z = x[i][k] / x[k][k];
            for (j = k; j <= n + 1; j++)
                x[i][j] -= x[k][j] * z;
        }
    }
    for (i = 1; i <= m; i++) {
        for (j = 1; j <= n && fabs(x[i][j]) < EPS; j++);
        if (j > n && fabs(x[i][n+1]) > EPS)
            return 0;
    }
    if (multians) return 2;
    for (k -= 1; k >= 1; k--) {
        x[k][n+1] /= x[k][k];
        x[k][k] = 1;
        for (i = k - 1; i >= 1; i--) {
            z = x[i][k];
            for (j = k; j <= n + 1; j++)
                x[i][j] -= x[k][j] * z;
        }
    }
    return 1;
}
int main() {
    int i, j, ret;
    scanf("%d%d", &n, &m);
    for (i = 1; i <= m; i++) {
        for (j = 1; j <= n + 1; j++)
            scanf("%lf", &x[i][j]);
    }
    ret = Gauss();
    if (ret == 0) puts("No solutions");
    else if (ret > 1) puts("Many solutions");
    else for (i = 1; i <= n; i++)
        printf("%d\n", (int)(x[i][n+1] + 0.5));
    return 0;
}
```
