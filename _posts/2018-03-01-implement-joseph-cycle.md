---
layout: post
title: 模拟约瑟夫环
category: 算法
tag:
---

### 约瑟夫环问题

有编号为1~n的n个人围成一个圈，从第1个人开始报数，数到m的人出列，后面的人重新从1开始报数，按出列顺序输出每个人的编号。

这是个简单的模拟问题，可以用双向链表来做，实现时有两处可做些优化。

1. 如果m比当前环的长度l还要大，那么肯定绕圈了，结果跟只走m%l步一样。
2. 由于是环，往左与往右都能到目标位置，尽量按步数少的方向走。

```cpp
#include <bits/stdc++.h>
using namespace std;
int n, m, l[10005], r[10005];
void init() {
    l[1] = n;
    for (int i = 2; i <= n; i++)
        l[i] = i-1;
    r[n] = 1;
    for (int i = 1; i < n; i++)
        r[i] = i+1;
}
void del(int x) {
    r[l[x]] = r[x];
    l[r[x]] = l[x];
}
int move(int x, int a[], int t) {
    for (int i = 0; i < t; i++)
        x = a[x];
    return x;
}
void go() {
    int x = 1, d = n;
    while (d) {
        int R = (m-1) % d;
        int L = d - R;
        if (R <= L)
            x = move(x, r, R);
        else
            x = move(x, l, L);
        del(x); d -= 1;
        cout << x << " ";
        x = r[x];
    }
    cout << endl;
}
int main() {
    cin >> n >> m;
    init(); go();
    return 0;
}
```

### 问题扩展

假设有2n个人围成一个环，编号分别为1~2n，求最小的数m，使用编号为n+1~2n的n个人先出列。

原题可参考[51nod1875 丢手绢](http://www.51nod.com/onlineJudge/questionCode.html#!problemId=1875)。

这个问题没想到好的解法，只能从小到大暴力枚举m去尝试，普通模拟可能耗时会更长，而上述代码由于两处优化，在n<14的情况下跑得还算快。

