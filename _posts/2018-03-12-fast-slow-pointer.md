---
layout: post
title: floyd判圈
category: 算法
tag:
---

floyd判圈算法也称为快慢指针算法，可在有限状态机、迭代函数或者链表上判断是否存在环，如存在，还能求出环的长度以及环的起点。

### 算法原理

该算法的过程很简单，给定起始点S（不需要在环内），在S处设置两个指针A和B，每次A前进一步，同时B前进两步，如果没有环，B永远在A的前面；否则必存在某一时刻，B与A再次相遇。

- 判环是否存在

按上述步骤模拟，如果A与B未相遇且B已走到头，则不存在环。反之，若A与B再次相遇，则存在环。

- 环的长度

假设环存在，A与B再次相遇的点记为M，让A继续前进，再次回到M所经过的步数即为环的长度。

- 环的起点

让A从S开始，B从M开始，每次两指针均前进一步，首次相遇的点就是环的起点。

### 例题[uva11549](https://cn.vjudge.net/problem/UVA-11549) Calculator Conundrum

题面：有款老式计算器，最多只能显示n(1<=n<=9)位数字，最开始上面显示的数字为k，然后不断平方，如溢出则显示结果的前n位，问显示的最大数字是多少？

```cpp
#include <bits/stdc++.h>
using namespace std;
int Next(int n, int x) {
    int a[32], z = 0;
    long long r = (long long)x * x;
    while (r) {
        a[z++] = r % 10;
        r /= 10;
    }
    int ret = 0;
    for (int i = z, j = 0; i > 0 && j < n; i--, j++)
        ret = ret * 10 + a[i-1];
    return ret;
}
int go(int n, int k) {
    int a = k, b = k, ans = k;
    do {
        a = Next(n, a);
        b = Next(n, b); ans = max(ans, b);
        b = Next(n, b); ans = max(ans, b);
    } while (a != b);
    return ans;
}
int main() {
    int T, n, k;
    cin >> T;
    while (T--) {
        cin >> n >> k;
        cout << go(n, k) << endl;
    }
    return 0;
}
```

