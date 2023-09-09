---
layout: post
title: 流水作业调度问题
category: 算法
tag:
---

### 问题描述

有n个作业，需要在由2台机器M1和M2组成的流水线上进行加工，每个作业的顺序都是先在M1上加工，然后在M2上加工，M1和M2加工作业i所需要的时间分别为a[i]和b[i]。可以合理地安排作业的执行顺序，使得从第1个作业在M1上加工开始，到最后一个作业在M2上加工完成为止，所用的总时间最少，求这个最少时间。

### 解决策略

1. 将作业分为F和G两部分，F中的作业满足a[i] < b[i]，G中的作业满足a[i] >= b[i]。
2. 将F中的作业按a排升序，G中的作业按b排降序。
3. 依次执行F和G中的作业，花费的总时间最少。

### 代码实现

题目可参考：[51nod1205](http://www.51nod.com/onlineJudge/questionCode.html#!problemId=1205) 流水线调度

```cpp
#include <bits/stdc++.h>
using namespace std;
struct st {int a, b;}t;
vector<st> f, g;
int n, ans1, ans2;
bool cmp1(st x, st y) {
    return x.a < y.a;
}
bool cmp2(st x, st y) {
    return x.b > y.b;
}
int main() {
    cin >> n;
    for (int i = 0; i < n; i++) {
        cin >> t.a >> t.b;
        if (t.a < t.b)
            f.push_back(t);
        else
            g.push_back(t);
    }
    sort(f.begin(), f.end(), cmp1);
    sort(g.begin(), g.end(), cmp2);
    for (auto i : f) {
        ans1 += i.a;
        ans2 = max(ans1,ans2) + i.b;
    }
    for (auto i : g) {
        ans1 += i.a;
        ans2 = max(ans1,ans2) + i.b;
    }
    cout << ans2 << endl;
    return 0;
}
```
