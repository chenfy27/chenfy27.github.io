---
layout: post
title: 循环数组的最小表示
category: 算法
tag:
---

### 问题描述

假设有数组a[n]，可通过旋转得到n种表示，求字典序最小的表示。

最小表示的典型应用是判断同构问题，例如判断两个字符串是否为变形词，只需将各个串转成最小表示，再判断最小表示是否相同即可。

### 具体实现

直观做法是枚举所有表示，取字典序最小的，但时间复杂度较高，这里介绍个线性做法。

以字符串为例，下面代码求的是变形词的最小表示，循环数组的求法完全一样。

```
#include <bits/stdc++.h>
using namespace std;
int MinExp(string s, int len) {
    int i = 0, j = 1, k = 0;
    while (i < len && j < len && k < len) {
        int t = s[(i+k)%len] - s[(j+k)%len];
        if (t == 0) {
            k += 1;
        } else {
            if (t > 0) {
                i += k + 1;
            } else {
                j += k + 1;
            }
            if (i == j) j += 1;
            k = 0;
        }
    }
    return min(i, j);
}
string Print(string s, int len, int start) {
    string ret;
    for (int i = start, j = 0; j < len; j++, i = (i+1) % len)
        ret += s[i];
    return ret;
}
int main() {
    string s;
    while (cin >> s) {
        int start = MinExp(s, s.length());
        cout << Print(s, s.length(), start) << endl;
    }
    return 0;
}
```
