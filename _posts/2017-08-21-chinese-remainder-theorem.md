---
layout: post
title: 中国剩余定理
category: 算法
keywords:
---

假设有模线性方程组：

```
x % a[1] = b[1]
x % a[2] = b[2]
...
x % a[n] = b[n]
```

其中a[i]两两互质，求最小的正整数x满足以上所有条件。中国剩余定理就是用于解决上述问题，结论如下：

```
x = sigma(b[i] * p[i] * v[i]) % prod
```

其中，prod为a[i]之积，p[i]为除a[i]外其余a之积，即p[i] = prod / a[i]，v[i]是p[i]模a[i]的乘法逆元。

记最小正整数解为x[0]，那么方程组的第k个解可表示为：x[k] = x[0] + (k - 1) * prod。

```c
#include <iostream>
using namespace std;
typedef long long LL;
LL n, a[10], b[10], p[10], v[10], prod = 1, ans;
LL xgcd(LL aa, LL bb, LL &x, LL &y) {
    if (bb == 0) {
        x = 1;
        y = 0;
        return aa;
    }
    LL d = xgcd(bb, aa % bb, y, x);
    y -= aa / bb * x;
    return d;
}
LL inv(LL aa, LL bb) {
    LL x, y;
    xgcd(aa, bb, x, y);
    if (x < 0) x += bb;
    return x;
}
int main() {
    cin >> n;
    for (int i = 0; i < n; i++) {
        cin >> a[i] >> b[i];
        prod *= a[i];
    }
    for (int i = 0; i < n; i++) {
        p[i] = prod / a[i];
        v[i] = inv(p[i], a[i]);
        ans = (ans + b[i] * p[i] * v[i]) % prod;
    }
    cout << ans << endl;
    return 0;
}
```
