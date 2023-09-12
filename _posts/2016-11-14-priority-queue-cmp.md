---
layout: post
title: 优先队列自定义
category: cpp
keywords:
---

默认情况下，优先队列存的是整数，为大根堆，即声明`priority_queue<int> pq;`等价于`priority_queue<int, vector<int>, less<int> > pq`，如果想改用小根堆，则应定义为：`priority_queue<int, vector<int>, greater<int> > pq;`。

有些时候，上述两种声明仍然无法满足需要，例如存入优先队列中的不是整数，而是结构体，这时应自定义比较函数。举例如下：

```
struct ST {
    int a, b;
    bool operator<(const ST &x) const {
    	return a < x.a; // 以a为键的大根堆；如要以a为键的小根堆，改成return x.a < a;
    }
};
priority_queue<ST> pq;
```

这是另外一种写法：

```
struct ST {
    int a, b;
    friend bool operator<(const ST &x, const ST &y) {
    	return x.a < y.a; // 以a为键的大根堆；如要以a为键的小根堆，改成return y.a < x.a;
    }
}
priority_queue<ST> pq;
```

如果既要用大根堆，又要用小根堆，可以同时重载小于和大于符号。

```
struct ST {
    int a, b;
    bool operator<(const ST &x) const {return a < x.a;}
    bool operator>(const ST &x) const {return a > x.a;}
};
priority_queue<ST> pq1; // 以a为键的大根堆
priority_queue<ST, vector<ST>, greater<ST> > pq2; // 以a为键的小根堆
```

