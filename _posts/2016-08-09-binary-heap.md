---
layout: post
title: 二叉堆
category: 算法
keywords: 算法, 二叉堆
---

二叉堆是一棵完全二叉树，一般用数组来表示，且下标从1开始。

假设某个非根节点的下标为n(>1)，那么它的：

- 左子节点下标为2n
- 右子节点下标为2n+1
- 父节点下标为n/2

### 堆的性质

根据父子节点的大小关系，二叉堆可分为大根堆和小根堆。  

- 大根堆：h[i] >= h[2i], h[i] >= h[2i+1]
- 小根堆：h[i] <= h[2i], h[i] <= h[2i+1]

二叉堆支持的操作有建堆、插入、弹出最值等，下面以大根堆为例分别说明。

### 建堆操作

建堆过程是调整现有数组的顺序，使之满足堆的性质。做法是从最后一个分支节点（下标为n/2）开始，到根节点为止，依次对每个分支节点做下沉操作，总时间复杂度为O(n)。

```cpp
void down(int idx) {
    int i, j;
    for (i = idx; (j = 2*i) <= size; i = j) {
        if (j+1 <= size && h[j+1] > h[j])
            j += 1;
        if (h[j] <= h[i]) break;
        swap(h[i], h[j]);
    }
}
void heapify(int n) {
    size = n;
    for (int i = n/2; i >= 1; i--)
        down(i);
}
```

### 添加元素操作

先将元素放到最后，再上浮调整以满足堆的性质，时间复杂度为O(logn)。

```cpp
void up(int idx) {
    while (idx > 1 && h[idx] > h[idx/2]; i /= 2)
        swap(h[i], h[i/2]);
}
void push(int val) {
    size += 1;
    h[size] = val;
    up(size);
}
```

### 弹出最值操作

二叉堆的最值在堆顶，即下标为1的位置。  
弹出最值后，把最后的元素补到该处，对它做下沉操作，时间复杂度为O(logn)。

```cpp
int top() {
    return h[1];
}
void pop() {
    h[1] = h[size];
    size -= 1;
    down(1);
}
```


### 判空操作

```cpp
bool empty() {
    return size == 0;
}
```

