---
layout: post
title: python中的栈和队列
category: python
tag:
---

栈和队列是较常用的数据结构，本文整理了它们在python下的高效实现方法。

### 栈

python内置了list类型，通过它的append和pop方法就可以实现栈的功能，由于各项操作都是O(1)的，速度还算快。

```python
class Stack():
    def __init__(self):
        self.data = []
    
    def push(self, item):
        self.data.append(item)

    def top(self):
        return self.data[-1]

    def pop(self):
        self.data.pop()
    
    def empty(self):
        return len(self.data) == 0

    def size(self):
        return len(self.data)
```

经测试验证，使用collections模块中的deque来实现栈效率还要高些，deque常用的接口列出如下：

<table border="1" cellspacing="0" style="width:800px; height:100px; text-align:left" cellpadding="6">
<tr><td>append(x)</td><td>add x to the right side of the deque</td></tr>
<tr><td>appendleft(x)</td><td>add x to the left side of the deque</td></tr>
<tr><td>pop()</td><td>remove and return an element from the right side of the deque</td></tr>
<tr><td>popleft()</td><td>remove and return an element from the left side of the deque</td></tr>
<tr><td>clear()</td><td>remove all elements from the deque</td></tr>
<tr><td>count(x)</td><td>count the number of elements equal to x</td></tr>
<tr><td>len(d)</td><td>return number of elements in deque d</td></tr>
<tr><td>if d:</td><td>check if deque d is empty or not</td></tr>
</table><p/>

在python中，对于list, tuple, string, dict等类型的对象x，都可以用if x的方式来判断是否为空。

对于栈，使用append和pop的组合即可实现。

### 队列

可以用list的append/pop(0)或者insert(0)/pop()来实现队列，由于insert(0)和pop(0)是O(n)的，效率不够好，建议直接用标准库collections提供的deque。

对于队列，使用append和popleft的组合即可实现，当然也可以用appendleft和pop来做。

### 优先队列

可以使用list结构自行实现二叉堆，而python库中也有多种实现，经过测试，性能最好的要属heapq模块，建议使用。

需要注意的是，python提供的二叉堆堆顶元素下标为0，并且默认均是小根堆。

<table border="1" cellspacing="0" style="width:800px; height:100px; text-align:left" cellpadding="6">
<tr><td>heappush(heap, item)</td><td>push item onto heap and maintain heap invariant</td></tr>
<tr><td>heap[0]</td><td>access the smallest item without popping it</td></tr>
<tr><td>heappop(heap)</td><td>pop and return the smallest item from heap</td></tr>
<tr><td>heapify(x)</td><td>transform list x into a heap, in-place, in linear time</td></tr>
</table><p/>

### 性能测试

下面编写代码对上述三种数据结构做性能测试。

```python
import logging as log
from collections import deque
import heapq

def TestStack(n):
    log.debug('===== begin test stack =====')
    s = deque()
    for i in range(n):
        s.append(i)
    log.debug('stack push ok')
    i = n
    while s:
        i -= 1
        assert(i == s.pop())
    log.debug('stack pop ok')

def TestQueue(n):
    log.debug('===== begin test queue =====')
    q = deque()
    for i in range(n):
        q.append(i)
    log.debug('queue push ok')
    i = 0
    while q:
        assert(i == q.popleft())
        i += 1
    log.debug('queue pop ok')

def TestPriorityQueue(n):
    log.debug('===== begin test priority queue =====')
    pq = []
    for i in range(n):
        heapq.heappush(pq, i)
    log.debug('priority queue push ok')
    i = 0
    while pq:
        assert(i == pq[0])
        heapq.heappop(pq)
        i += 1
    log.debug('priority queue pop ok')

def main():
    log.basicConfig(level=log.DEBUG, format='%(asctime)s - %(message)s')
    N = 1000000
    TestStack(N)
    TestQueue(N)
    TestPriorityQueue(N)

if __name__ == '__main__':
    main()
```

某次运行结果如下：

```
$ time python3 z.py 
2018-06-23 15:34:45,030 - ===== begin test stack =====
2018-06-23 15:34:45,175 - stack push ok
2018-06-23 15:34:45,384 - stack pop ok
2018-06-23 15:34:45,384 - ===== begin test queue =====
2018-06-23 15:34:45,521 - queue push ok
2018-06-23 15:34:45,740 - queue pop ok
2018-06-23 15:34:45,740 - ===== begin test priority queue =====
2018-06-23 15:34:46,001 - priority queue push ok
2018-06-23 15:34:46,859 - priority queue pop ok

real       0m1.889s
user       0m1.790s
sys        0m0.078s
```

运行速度非常快，接近C/C++的性能。
