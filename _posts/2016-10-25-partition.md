---
layout: post
title: partition算法
category: 算法
keywords:
---

给定一个无序数组，要求将该数组划分为左右两个子数组，使得满足某种性质的元素都位于左子数组中，不满足性质的元素都在右子数组中。

partition算法建立在循环不变式上，设无序数组为A[n]，下标小于等于i的元素是满足条件的元素，大于等于j的元素是待处理的元素。流程如下：

- 初始化i为-1，j为0；
- 让j从头到尾遍历A：如果A[j]满足性质，则i加1，交换A[i]与A[j]；
- 如有需要，返回i，即左子数组最后一个元素的下标，便于递归处理；

### 快速排序：

```
int Partition(int *a, int lo, int hi) {
    int i, j;
    for (i = lo - 1, j = lo; j <= hi; j++)
        if (a[j] <= a[hi]) /* pick right-most element as pivot */
            swap(&a[++i], &a[j]);
    return i;
}
```

注：这种做法虽然时间复杂度也是O(n)，但交换次数比较多，常数大，快排建议两端齐扫的写法。

```
void Partition(int *a, int lo, int hi) {
    int i = lo, j = hi, p = a[hi], t;
    while (i <= j) {
        while (i <= j && a[i] < p) i++;
        while (i <= j && a[j] > p) j--;
        if (i <= j) {
            t = a[i], a[i] = a[j], a[j] = t;
            i++, j--;
        }
    }
}
```

### [奇偶分割数组问题](http://www.lintcode.com/zh-cn/problem/partition-array-by-odd-and-even/)：

```
void PartitionArray(vector<int> &nums) {
    for (int i = -1, j = 0; j < nums.size(); j++)
        if (nums[j] & 1)
            swap(a[++i], a[j]);
}
```

高效写法：

```
void PartitionArray(vector<int> &nums) {
    int i = 0, j = nums.size() - 1;
    while (i <= j) {
        while (i <= j && nums[i] % 2 == 1) i++;
        while (i <= j && nums[j] % 2 == 0) j--;
        if (i <= j) {
            swap(nums[i], nums[j]);
            i++, j--;
        }
    }
}
```

### [数组划分问题](http://www.lintcode.com/zh-cn/problem/partition-array/)：

```
int PartitionArray(vector<int> &nums, int k) {
    int i, j;
    for (i = -1, j = 0; j < nums.size(); j++)
        if (nums[j] < k)
            swap(nums[++i], nums[j]);
    return i + 1;
}
```

### 扩展——三分partition

在上述数组划分问题中，如果要求将数组划分为三部分，左边的元素均小于k，中间的元素都等于k，右边的元素都大于k，如何实现？

思路类似，二分partition是遇到满足条件的就往前扔，三分partition则是遇到小的往前扔，遇到大的往后扔，相等的直接跳过。

```
int PartitionArray(vector<int> &nums, int k) {
    int i, j, n;
    for (i = -1, j = 0, n = nums.size(); j < n; j++) {
        if (nums[j] < k)
            swap(nums[j], nums[++i]);
        else if (nums[j] > k)
            swap(nums[j], nums[--n]);
    }
    return i + 1;
}
```

