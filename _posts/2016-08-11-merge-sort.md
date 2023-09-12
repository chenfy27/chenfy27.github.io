---
layout: post
title: 归并排序
category: 算法
keywords: sort
---

归并排序是一种采用分治策略的简单高效排序算法，最好和最坏时间复杂度均为O(nlogn)，优势在于它稳定、适用于外排序，缺点在于需要额外的O(n)辅助空间。

### 排序思路

1. 将待排序区间分成左右两块大小相同的区间
2. 分别对左右两子区间排序
3. 归并左右子区间使整体区间有序

上述做法用递归很容易实现，递归出口是区间只有1个元素，无需再分。

以下是对整型数组按从小到大排列的示例代码。

```
void Merge(int *a, int lbeg, int rbeg, int rend, int *t) {
    int i = lbeg, j = rbeg, k = lbeg;
    while (i < rbeg && j < rend)
        t[k++] = a[i] <= a[j] ? a[i++] : a[j++];
    while (i < rbeg) t[k++] = a[i++];
    while (j < rend) t[k++] = a[j++];
    for (i = lbeg; i < rend; i++)
        a[i] = t[i];
}
void MSort(int *a, int lbeg, int rend, int *t) {
    if (lbeg + 1 < rend) {
        int rbeg = (lbeg + rend) / 2;
        MSort(a, lbeg, rbeg, t);
        MSort(a, rbeg, rend, t);
        Merge(a, lbeg, rbeg, rend, t);
    }
}
void MergeSort(int *a, int size) {
    int *t = malloc(sizeof(int) * size);
    MSort(a, 0, size, t);
    free(t);
}
```

自下而上观察归并排序的过程，可以写出对应的非递归做法。

```
void MergeSort(int *a, int size) {
    int i, j, *t = malloc(sizeof(int) * size);
    for (i = 1; i < size; i *= 2)
        for (j = 0; j + i < size; j += 2 * i)
            Merge(a, j, j + i, min(size, j + 2 * i), t);
    free(t);
}
```

