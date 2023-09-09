---
layout: post
title: 洗牌算法
category: 算法
keywords: random,shuffle,select,algo
---

假设有数组A[n]，现要打乱元素的顺序，使各个元素等概率地出现在各个位置，称为洗牌操作。

下面是一种简单高效的做法，其思路在从左往右扫描，每次从右侧元素中随机选择一个与当前元素交换位置：

```
function Shuffle(A, n)
    for i = 0 to n-1
        j = i + rand() % (n - i)
        swap(A[i], A[j])
```

由于只扫了一遍，时间复杂度为O(n)。
