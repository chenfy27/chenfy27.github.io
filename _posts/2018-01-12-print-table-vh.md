---
layout: post
title: 二维表按行列输出
keywords:
category: 平台
---

在C语言中，经常用一维数组来表示二维表，假定表的大小为row\*col，那么第i行第j列元素对应的下标为i*col+j，因而可以简单地将其按二维表输出，如果互换i与j，并用row代替col，那么输出的便是原二维表的转置。

```cpp
#include <bits/stdc++.h>
using namespace std;
int main() {
    int n[40], r = 5, c = 8;
    for (int i = 0; i < 40; i++)
        n[i] = i + 1;
    puts("==== HHHH ====");
    for (int i = 0; i < r; i++) {
        for (int j = 0; j < c; j++)
            printf("%-5d", n[i*c+j]);
        puts("");
    }
    puts("==== VVVV ====");
    for (int i = 0; i < r; i++) {
        for (int j = 0; j < c; j++)
            printf("%-5d", n[j*r+i]);
        puts("");
    }
    return 0;
}
```

运行结果：

```
==== HHHH ====
1    2    3    4    5    6    7    8    
9    10   11   12   13   14   15   16   
17   18   19   20   21   22   23   24   
25   26   27   28   29   30   31   32   
33   34   35   36   37   38   39   40   
==== VVVV ====
1    6    11   16   21   26   31   36   
2    7    12   17   22   27   32   37   
3    8    13   18   23   28   33   38   
4    9    14   19   24   29   34   39   
5    10   15   20   25   30   35   40
```
