---
layout: post
title: 二元一次方程问题
category: 算法
tag:
---

### [51nod1548](http://www.51nod.com/onlineJudge/questionCode.html#!problemId=1548) 欧姆诺和糖果

题面：有A和B两种物品，每件A花费Wa，收益Ha，每件B花费Wb，收益Hb，要求总花费不超过C(<=1e9)，求最大收益。

分析：不妨设物品A的性价比更高（如不满足交换二者即可），那么Ha/Wa>=Hb/Wb，即Ha\*Wb>=Hb\*Wa。考虑Wb件A物品与Wa件B物品，因为Wb\*Wa=Wa\*Wb，故而它们的花费是一样的，但收益却有高低，因为Ha\*Wb>=Hb\*Wa，因此将Wa件B物品换成Wb件A物品是更优的选择，至少不会更差。根据这条性质，用反证法容易证明，取到最优值时，性价比低的物品数肯定不会超过sqrt(C)，从而得到了sqrt(n)时间复杂度的枚举法。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
LL c, hr, hb, wr, wb, ans;
int main() {
    cin >> c >> hr >> hb >> wr >> wb;
    for (LL i = 0; i * i <= c; i++) {
        if (i * wr <= c)
            ans = max(ans, hr*i + (c-wr*i)/wb*hb);
        if (i * wb <= c)
            ans = max(ans, hb*i + (c-wb*i)/wr*hr);
    }
    cout << ans << endl;
    return 0;
}
```

### [51nod1352](http://www.51nod.com/onlineJudge/questionCode.html#!problemId=1352) 集合计数

题面：给定int32范围内的正整数A,B,C，问方程Ax+By=C有多少组正整数解(x,y)？

分析：根据扩展欧几里德算法判断方程是否有解，如有则求出一组解(x0,y0)，那么通解可表示为(x0+kB,y0-kA)，统计正整数解即可。需要注意的是用扩展欧几里德算法求出解的特征，\|x0\|+\|y0\|最小，且系数绝对值大的那项绝对值小。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
LL gcd(LL a, LL b) {
    return b ? gcd(b, a%b) : a;
}
void exgcd(LL a, LL &x, LL b, LL &y) {
    if (b == 0) {
        x = 1; y = 0;
    } else {
        exgcd(b, y, a%b, x);
        y -= a/b*x;
    }
}
LL T, n, a, b, d, k, x, y, z, ans;
int main() {
    ios::sync_with_stdio(0);
    cin >> T;
    while (T--) {
        cin >> n >> a >> b;
        d = gcd(a, b);
        if ((n+1) % d) {
            ans = 0;
        } else {
            a /= d; b /= d; k = (n+1) / d;
            exgcd(a, x, b, y);   // ax + by = 1
            x *= k; y *= k;      // ax + by = k
            ans = x/b + y/a;
            if (x > 0 && x % b == 0) ans -= 1;
            if (y > 0 && y % a == 0) ans -= 1;
        }
        cout << ans << endl;
    }
    return 0;
}
```

其中，`ans -= 1`那两行最多只有一行满足条件。
