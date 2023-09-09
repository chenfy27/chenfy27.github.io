---
layout: post
category: 算法
title: 平方分割（莫队算法）
keywords:
---

莫队算法通常用来解决一类多次区间询问问题，使用该算法有两个要求：

- 问询可离线做预处理
- 如已知[l,r]的结果，可以方便地知道[l-1,r],[l+1,r],[l,r-1],[l,r+1]的结果

处理流程大致如下：

- 读入所有问询，分块排序
- 初始化边界与当前结果
- 依次处理每条问询，保存结果
- 输出各条问询的答案

[例题](http://www.spoj.com/problems/DQUERY/)：给定正整数数组a[n]和q组问询(l,r)，对于每次问询，输出区间(l,r)包含多少个不同的数。

以下代码评测会超时，加入快速输入可以通过。

```cpp
#include <bits/stdc++.h>
using namespace std;
int n, a[30005], m, cnt[1000005], bs, L, R, ans[200005], cur;
struct st {
    int l, r, x;
    bool operator<(const st &u) const {
        if (l/bs == u.l/bs) return r < u.r;
        return l < u.l;
    }
}q[200005];
void add(int x) {
    if (++cnt[x] == 1) cur++;
}
void del(int x) {
    if (--cnt[x] == 0) cur--;
}
int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++)
        scanf("%d", a+i);
    scanf("%d", &m);
    bs = sqrt(m);
    for (int i = 1; i <= m; i++) {
        scanf("%d%d", &q[i].l, &q[i].r);
        q[i].x = i;
    }
    sort(q+1, q+1+m);
    for (int i = 1; i <= m; i++) {
        while (L < q[i].l) del(a[L++]);
        while (L > q[i].l) add(a[--L]);
        while (R < q[i].r) add(a[++R]);
        while (R > q[i].r) del(a[R--]);
        ans[q[i].x] = cur;
    }
    for (int i = 1; i <= m; i++)
        printf("%d\n", ans[i]);
    return 0;
} 
```

