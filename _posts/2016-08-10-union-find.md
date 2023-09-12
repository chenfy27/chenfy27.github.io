---
layout: post
title: 并查集
category: 算法
keywords: 算法, 并查集
---

并查集(UnionFind)是一种简单高效的数据结构，主要用于解决一类连通性判断的问题。以下是经典描述：平面上最初有n个孤立的点，编号分别为1~n，随后进行m次操作，每次从中选择两个点并将它们连接起来。现在任意选出两个点，问它们是否连通？

### 1.初始化

设f[i]表示节点i的父节点。

```
void Init(int n) {
    for (i = 1; i <= n; i++) f[i] = i;
}
```

### 2.合并集合

```
int Root(int x) {
    return x == f[x] ? x : f[x] = Root(f[x]);
}

void Merge(int x, int y) {
    f[Root(x)] = Root(y);
}
```

### 3.判断连通性

```
int IsConnected(int x, int y) {
    return Root(x) == Root(y);
}
```

并查集的一个经典应用是Kruskal算法求最小生成树。
