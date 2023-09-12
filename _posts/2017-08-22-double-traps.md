---
layout: post
title: 浮点数陷阱
category: cpp
keywords:
---

由于计算机表示精度有限，使用浮点数计算时需注意些问题。

### 大小比较

浮点数之间大于、小于关系可以直接比较，但判断是否相等则看两数之差的绝对值是否在指定误差范围内。

```
#define EPS 1e-6
double x, y;
while (~scanf("%lf%lf", &x, &y)) {
    if (x < y)
        printf("%f is smaller than %f\n", x, y);
    else if (x > y)
        printf("%f is bigger than %f\n", x, y);
    else if (fabs(x - y) < EPS)
        printf("%f is equal to %f\n", x, y);
}
```

### 格式化输入输出

- float类型输入和输出都用%f
- double类型输入用%lf，输出用%f

即浮点数输出时都用%f，标准里没有规定%lf的用法。

### 四舍五入与截断

- 浮点数截断小数部分取整：`printf("%d\n", int(x));`
- 浮点数四舍五入方式取整：`printf("%d\n", int(x + 0.5));`
- 以四舍五入的方式保留两位小数：`printf("%.2f\n", x);`

注意，用%.0f来四舍五入取整是不行的，可用如下程序来验证。

```
#include <stdio.h>
int main() {
    int i; double x;
    for (i = 0; i < 1000000; i++) {
        x = i * 1.0 / 100000;
        printf("%f %.0f %d\n", x, x, (int)(x + 0.5));
    }
    return 0;
}
```

运行代码并用awk来寻找不一致的输出数据，结果如下：

```
$ ./a.out | awk '$2 != $3 {print NR, $0}'
50001 0.500000 0 1
250001 2.500000 2 3
450001 4.500000 4 5
650001 6.500000 6 7
850001 8.500000 8 9
```
