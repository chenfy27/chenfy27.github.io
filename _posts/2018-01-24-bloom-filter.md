---
layout: post
title: bloom filter简介
category: 算法
tag:
---

Bloom Filter是一种时空效率都很高的随机数据结构，通过一个位数组和多哈希函数来解决海量数据的判重问题，适用于不严格要求100%正确的场合，也可以用来对数据进行初步过滤。

Bloom Filter的原理与Bitmap十分类似，区别仅在于使用k个而不是单个哈希函数，以降低因哈希碰撞冲突造成的误判。

Bloom Filter一般需要提供两个接口：

- 将元素加入集合
- 给定元素，判断集合中是否存在

### 初始化

初始集合bits[m]是个全为0的位数组。

### 添加元素

在添加元素时，分别计算出k个哈希值hi，并将bits[hi]置成1。

### 判断存在性

给定元素，分别计算出k个哈希值hi，只要有一个bits[hi]是0，则该元素一定不在集合中；反之，如果全部bits[hi]都是1，那么该元素有很大概率存在于集合中。

### 示例代码

以下代码简单实现了个Bloom Filter。

```python
class BloomFilter(object):
    def __init__(self, size):
        self.size = 2 ** size
        self.bits = [False] * self.size
        self.keys = [131, 151, 233]
    def get_hash(self, word, p):
        h = 0
        for i in word:
            h = (h * p + ord(i)) & (self.size - 1)
        return h
    def insert(self, word):
        for x in self.keys:
            h = self.get_hash(word, x)
            self.bits[h] = True
    def might_contain(self, word):
        for x in self.keys:
            h = self.geth(word, x)
            if not self.bits[h]:
                return False
        return True
```
