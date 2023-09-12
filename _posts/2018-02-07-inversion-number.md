---
layout: post
title: 逆序数
category: 算法
keywords:
---

[问题](http://www.51nod.com/onlineJudge/questionCode.html#!problemId=1019)：在一个排列中，如果前面的数大于后面的数，则称为一个逆序，任意两数构成的逆序之和称为这个排列的逆序数。现给出数字序列，求逆序数。

方法1：暴力求解，时间复杂度为O(n^2)。

方法2：是在归并排序的过程中进行统计，时间复杂度为O(nlogn)，是最快的方法。

```
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
int n, a[50005], t[50005], ans;
void merge_sort(int l, int r) {
    if (l + 1 >= r) return;
    int m = (l + r) / 2;
    merge_sort(l, m);
    merge_sort(m, r);
    int i = l, j = m, k = l;
    while (i < m && j < r) {
        if (a[i] <= a[j]) {
            t[k++] = a[i++];
        } else if (a[i] > a[j]) {
            t[k++] = a[j++];
            ans += m - i;
        }
    }
    while (i < m) t[k++] = a[i++];
    while (j < r) t[k++] = a[j++];
    for (i = l; i < r; i++) a[i] = t[i];
}
int main() {
    scanf("%d", &n);
    for (int i = 0; i < n; i++)
        scanf("%d", &a[i]);
    merge_sort(0, n);
    printf("%d\n", ans);
    return 0;
}
```

方法3：利用bit可快速插点问线的性质，将插点操作时增加的值设为1，即可实现计数的功能。为方便统计，在插入时要按一定的顺序进行。每个元素有两个属性：下标和值，因而有两种做法。

第1种做法是按下标顺序挨个加入，bit的下标表示值，在加入某个元素时，bit中元素的下标都比它要小，所以要统计值比它大的元素个数，由于本题中元素值的取值范围太大，数组开不下，需要先离散化。

```
#include <bits/stdc++.h>
using namespace std;
int n, m, a[50005], b[50005], c[50005], f[50005];
int lowbit(int x) {
    return x & -x;
}
void add(int x, int v) {
    while (x <= n) {
        f[x] += v;
        x += lowbit(x);
    }
}
int sum(int x) {
    int ret = 0;
    while (x > 0) {
        ret += f[x];
        x -= lowbit(x);
    }
    return ret;
}
int main() {
    scanf("%d", &n);
    for (int i = 0; i < n; i++) {
        scanf("%d", &a[i]);
        b[i] = a[i];
    }
    sort(b, b+n);
    m = unique(b, b+n) - b;
    for (int i = 0; i < n; i++)
        c[i] = lower_bound(b, b + m, a[i]) - b;
    int ans = 0;
    for (int i = 0; i < n; i++) {
        ans += sum(n) - sum(c[i]+1);
        add(c[i]+1, 1);
    }
    printf("%d\n", ans);
    return 0;
}
```

第2种做法是按元素值从小到大的顺序依次插入，bit的下标代表元素下标，在加入某个元素时，bit中元素的值均不超过它，需要统计bit中下标比它大的元素个数。

```
#include <bits/stdc++.h>
using namespace std;
struct st {
    int x, v;
}a[50005];
int n, f[50005];
int lowbit(int x) {
    return x & -x;
}
void add(int x, int v) {
    while (x <= n) {
        f[x] += v;
        x += lowbit(x);
    }
}
int sum(int x) {
    int ret = 0;
    while (x > 0) {
        ret += f[x];
        x -= lowbit(x);
    }
    return ret;
}
bool cmp(st p, st q) {
    if (p.v != q.v) return p.v < q.v;
    return p.x < q.x;
}
int main() {
    scanf("%d", &n);
    for (int i = 0; i < n; i++) {
        scanf("%d", &a[i].v);
        a[i].x = i;
    }
    sort(a, a+n, cmp);
    int ans = 0;
    for (int i = 0; i < n; i++) {
        ans += sum(n) - sum(a[i].x+1);
        add(a[i].x+1, 1);
    }
    printf("%d\n", ans);
    return 0;
}
```

这两种做法的时间复杂度都是O(nlogn)，但常数比归并排序要大一些。

