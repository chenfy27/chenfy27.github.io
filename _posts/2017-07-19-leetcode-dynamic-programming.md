---
layout: post
title: leetcode动态规划专练
category: 算法
keywords:
---

### [House Robber](https://leetcode.com/problems/house-robber/#/description)

题面：给定非负整数数组A[n]，从中选出若干个元素，要求所选元素不能相邻，求所选元素之和的最大值。

思路：假设已经知道对于A[i]的答案，为f[i]，要推出A[i+1]的答案，对于第i+1个元素，如果选，那么第i个元素就不能选，f[i+1] = a[i+1] + f[i-1]；如果不选，那么有f[i+1] = f[i]，从而f[i+1] = max(f[i], a[i+1] + f[i-1])，因递推式中每一项只与前两项有关，可将空间优化到O(1)。

```python
def rob(self, nums):
    last, now = 0, 0
    for i in nums:
        last, now = now, max(now, last + i)
    return now
```

### [House Robber II](https://leetcode.com/problems/house-robber-ii/#/description)

题面：非负整数数组A[n]构成一个环，从中选若干个元素，要求所选元素不能相邻，求所选元素之和的最大值。

思路：根据是否选择a[n]分成两种情况，转化为不带环的情况。

```python
def rob1(self, nums):
    last, now = 0, 0
    for i in nums:
        last, now = now, max(now, last + i)
    return now
def rob(self, nums):
    if len(nums) == 0: return 0
    return max(self.rob1(nums[:-1]), nums[-1] + self.rob1(nums[1:-2]))
```

### [House Robber III](https://leetcode.com/problems/house-robber-iii/#/description)

题面：在二叉树上的每个节点上有非负整数值，现要从树上选出若干个节点，要求所选节点不能是相邻的父子节点，求所选节点值之和的最大值。

思路：自上而下直接递归可得到正确结果，但因为做了大量重复计算，效率太低，宜改成自下而上的递推做法。对于某个节点，先计算其左右子节点的结果，包括选择和不选择两种情况分别可得到的最大结果，然后考虑根节点本身，也分选择和不选择两种情况。

```python
def rob1(self, root):
    if not root: return (0, 0)
    left = self.rob1(root.left)
    right = self.rob1(root.right)
    return (root.val + left[1] + right[1], max(left) + max(right))
def rob(self, root):
    return max(self.rob1(root))
```
