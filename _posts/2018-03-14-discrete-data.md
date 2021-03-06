---
layout: post
title: 数据离散化
category: 算法
tag:
---

有的时候，数据量本身不大，但其数据范围可能很大，在对这类数据应用树状数组、线段树等数据结构时，直接套用将占用大量的内存空间，一般先对数据做离散化处理。

### 原理

举个例子，假设原始数据用数组a表示，取值如下。

```
a[] = {83765, 3982671, -9087236, 3982671, 4692814}
```

对a排序并做去重处理得到数组b。

```
b[] = {-9087236, 83765, 3982671, 4692814}
```

那么根据数组b以及各元素的下标，原数组a可映射成数组c。

```
c[] = {1, 2, 0, 2, 3}
```

此时，对c套用各种数据结构就不会超内存了。

### 实现

设原数组为a[n]，排序去重后的数组记为b[m]，a[n]做离散化后对应的数组为c[n]。

```
#include <bits/stdc++.h>
using namespace std;
int n, m, a[105], b[105], c[105];
int main() {
    cin >> n;
    for (int i = 0; i < n; i++) {
        cin << a[i];
        b[i] = a[i];
    }
    sort(b, b+n);
    m = unique(b, b+n) - b;
    for (int i = 0; i < n; i++)
        c[i] = lower_bound(b, b+m, a[i]) - b;
    return 0;
}
```

### 对应关系

虽然可在c[]上应用数据结构，但结果可能还是要用原始数据a[]来表示，这就需要知道三个数组之间的对应关系。

- 已知原数组下标idx，求它对应离散化后的值。

答案是c[idx]。

- 已知原值为val，求它对应离散化后的值。

答案是\*lower_bound(b, b+m, val)。

- 已知离散化后的值为val，求其对应的原值。

答案是b[val]。

