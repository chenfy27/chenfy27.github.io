---
layout: post
title: 哈希求字符串匹配
category: 算法
keywords:
---

介绍一种等比级数哈希字符串的方法，这种方法的一个特性是通过O(n)预处理出前缀哈希值后可O(1)获得任意子串的哈希值，这点在有些时候非常有用。

假设有长度为n的字符串s[n]，下标从1开始，记h[i]为前i个字符的哈希结果，计算方法为：

```
h[0] = 0
h[i] = h[i-1] * p + s[i]
```

说明：

- p为质数，一般取131或233。
- 哈希值用unsigned long long存储。
- 计算过程中无需取模，让其自然溢出即可。

计算完h[1~n]后，子串[l,r]的哈希值可由h[r]与h[l-1]推出来：h[l~r] = h[r] - h[l-1] * p ^ (r-l+1)，为快速得出结果，可先将p的幂打表。

下面来看道[KMP算法的模板题](http://hihocoder.com/problemset/problem/1015)：给出模式串和原串，问模式串在原串中出现过多少次？

咱们用哈希来做，先打表p的幂，对原串预处理出前缀的哈希值h[i]，然后计算出模式串的哈希值，最后扫一遍原串长度为模式串长度的子串，如果哈希值与模式串相等则认为匹配，时间复杂度为O(n+m)，与KMP算法相当。

```cpp
#include <stdio.h>
#include <string.h>
int i, n, cnt, tl, sl;
char s[1000005], t[10005];
unsigned long long p[1000005], h[1000005], th, tt;
int main() {
    for (p[0] = i = 1; i <= 1000000; i++)
        p[i] = p[i-1] * 131;
    scanf("%d", &n);
    while (n--) {
        scanf("%s%s", t+1, s+1);
        for (h[0] = 0, i = 1; s[i]; i++)
            h[i] = h[i-1] * 131 + s[i];
        for (th = 0, i = 1; t[i]; i++)
            th = th * 131 + t[i];
        tl = strlen(t+1); sl = strlen(s+1);
        for (cnt = 0, i = tl; i <= sl; i++) {
            tt = h[i] - h[i-tl] * p[tl];
            if (tt == th) cnt += 1;
        }
        printf("%d\n", cnt);
    }
    return 0;
}
```
