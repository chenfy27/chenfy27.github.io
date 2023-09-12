---
layout: post
title: 二分搜索
category: 算法
keywords:
---

二分搜索也称折半搜索，即每次都能将搜索范围缩小一半，直到找出解为止，优点是比较次数少，查找速度快，缺点是要求待查表有序，并支持随机访问。虽然二分搜索的原理简单，但要写出没有bug的代码却也不是件易事，下面通过几个例子加以说明。

#### 例1. STL之lower_bound

给定一非降序整数数组A和整数k，求最小的i使得A[i]大于等于k，如果不存在则返回-1。

```
int LowerBound(int a[], int n, int k) {
    int lo = 0, hi = n - 1, mid;
    while (lo < hi) {
        mid = lo + (hi - lo) / 2;
        if (k < a[mid])
            lo = mid + 1;
        else
            hi = mid;
    }
    return (lo < n && a[lo] >= k) ? lo : -1;
}
```

#### 例2. STL之upper_bound

给定一非降序整数数组A和整数k，求最小的i使得A[i]大于k，如果不存在则返回-1。

```
int UpperBound(int a[], int n, int k) {
    int lo = 0, hi = n - 1, mid;
    while (lo < hi) {
        mid = lo + (hi - lo) / 2;
        if (k <= a[mid])
            lo = mid + 1;
        else
            hi = mid;
    }
    return (lo < n && a[lo] > k) ? lo : -1;
}
```

#### 例3. 最后一个等于k的数

给定一非降序整数数组A和整数k，求最大的i使得A[i]等于k，如不存在则返回-1。

```
int Find(int a[], int n, int k) {
    int lo = 0, hi = n - 1, mid;
    while (lo < hi) {
        mid = lo + (hi - lo + 1) / 2;
        if (k < a[mid])
            lo = mid + 1;
        else if (k > a[mid])
            hi = mid - 1;
        else
            lo = mid;
    }
    return (lo < n && a[lo] == k) ? lo : -1;
}
```

#### 例4. 开平方

给定一个整数x（0 < x < 10^100），求x的算术平方根的整数部分。

这个问题更快的解法是牛顿迭代，这里只为演示二分搜索的使用。先通过倍增法估算出解的范围，然后进行二分搜索即可。

```
x = int(raw_input())
lo = hi = 1
while hi < x:
    lo, hi = hi, hi * 2
while lo < hi:
    mid = lo + (hi - lo + 1) / 2
    if mid * mid > x:
        hi = mid - 1
    else:
        lo = mid
print lo
```

#### 总结

- 搜索范围通常用闭区间表示，循环条件则为：lo < hi。
- 中间值mid的计算要么为：lo + (hi - lo) / 2；要么为：lo + (hi - lo + 1) / 2；这是避免死循环的关键点，其实只要考虑`lo + 1 == hi`的情景即可，选取的mid必须能使搜索范围变小，而不是原地踏步。
- 退出循出时必有：lo == hi，对它进行校验是有必要的，因为有可能原始搜索空间为空，循环根本就没进去过，或者满足条件的解不存在。但如果能保证最初的搜索范围内一定有解，则可以不做校验，如例4。
 
