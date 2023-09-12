---
layout: post
title: 排序函数用法
category: cpp
keywords:
---

### C排序库函数

c语言提供的排序库函数为qsort，需要头文件stdlib.h，底层用快速排序实现，不稳定，其函数原型为：

```
void qsort(void *base, int n, int size, int (*cmp)());
```

其中比较函数cmp的原型为：

```
int cmp(const void *x, const void *y);
```

manual上对其返回值做了详细说明：如果x小于y，则返回值小于0；相等则返回0；x大于y则返回值大于0。

下面举例说明其用法。

```
typedef struct ST { int a, b;}ST;
ST f[1000];
// 按b升序排，b相同则按a降序排
int cmp(const void *x, const void *y) {
    const ST *xx = (const ST*)x;
    const ST *yy = (const ST*)y;
    if (xx->b != yy->b)
        return xx->b < yy->b ? -1 : 1;
    if (xx->a != yy->a)
        return xx->a > yy->a ? -1 : 1;
    return 0;
}
qsort(f, 1000, sizeof(f[0]), cmp);
```

### C++排序库函数

c++中常用的排序函数为sort和stable_sort，分别为不稳定和稳定版本，需要头文件algorithm.h，这两函数的用法完全一样，以sort为例，原型为：

```
void sort(RandomAccessIterator first, RandomAccessIterator last);
void sort(RandomAccessIterator first, RandomAccessIterator last, Compare cmp);
```

使用例子：

```
struct ST {
    int a, b;
    bool operator<(const ST &o) const {
        if (b != o.b) return b < o.b;
        return a > o.a;
    }
}f[1000];
sort(f, f+1000, cmp);
```

如果不带cmp参数，则为对整数按从小到大排序。

### 说明

- qsort中比较函数返回类型为int，一般根据大小关系返回-1/0/1；而sort的比较函数返回类型为bool，因此直接返回比较关系就可以了。
- qsort比较函数不建议写成`return a - b;`的形式，因为有溢出的可能。

