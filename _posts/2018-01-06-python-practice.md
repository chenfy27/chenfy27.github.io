---
layout: post
title: python练习1801
keywords:
category: python
---

### [UVA10815](https://cn.vjudge.net/problem/UVA-10815) Andy\'s First Dictionary

题面：给出一篇文章，找出其中包含的所有不同单词（不区分大小写），转成小写并按字典序输出。

```cpp
#include <bits/stdc++.h>
using namespace std;
int main() {
    string s, word;
    set<string> ans;
    while (cin >> s) {
        for (int i = 0; i < s.length(); i++)
            s[i] = isalpha(s[i]) ? tolower(s[i]) : ' ';
        stringstream ss(s);
        while (ss >> word) ans.insert(word);
    }
    for (auto i : ans) cout << i << endl;
    return 0;
}
```

```python
import sys, re
ans = set()
for line in sys.stdin:
    ret = re.findall(u'[a-zA-Z]+', line)
    for i in ret: ans.add(i.lower())
for i in sorted(ans): print i
```

### [CF911B](http://codeforces.com/contest/911/problem/B) Two Cakes

题面：有a块A类蛋糕、b块B类蛋糕和n个盘子，现要把蛋糕放到盘子里，要求：(1)所有蛋糕都要放进盘子里；(2)每个盘子最少有一块蛋糕；(3)一个盘子只能装同一类蛋糕。问如何放才能最大化装蛋糕块数最少的盘子所放蛋糕数，输出该值。

由于原题数据范围较小，可以暴力模拟。

```python
n, a, b = map(int, raw_input().split())
for i in xrange(1, min(a, b) + 1):
    if a / i + b / i >= n:
        ans = i
print ans
```

也可以从盘子的角度考虑，A类蛋糕负责放前i个盘子，B类蛋糕负责放后面的n-i个盘子。

```python
n, a, b = map(int, raw_input().split())
print max([min(a / i, b / (n - i)) for i in range(1, n)])
```

如果数据量较大，考虑用二分答案。

```python
def ok(n, a, b, x):
    if x > a or x > b:
        return False
    return a / x + b / x >= n

n, a, b = map(int, raw_input().split())
lo, hi = 1, a
while lo < hi:
    mid = lo + (hi - lo + 1) / 2
    if ok(n, a, b, mid):
        lo = mid
    else:
        hi = mid - 1
print lo
```

### [CF798A](http://codeforces.com/contest/798/problem/A) Mike and palindrome

题面：给出一个只含小写字母的字符串，修改其中一个字符，问能否够成回文。

```python
s = raw_input()
d = 0
for i in xrange(len(s) / 2):
    d += s[i] != s[-i-1]
print 'YES' if d == 1 or d == 0 and len(s) % 2 else 'NO'
```

由于python支持负索引，下标i对应回文的位置下标可以写成-i-1。

```python
s = raw_input()
d = 0
for i, j in zip(s, s[::-1]):
    d += i != j
print 'YES' if d == 2 or d == 0 and len(s) % 2 else 'NO'
```

这种方式相当于做了两遍计算。如果数据量较大，在python2中建议用izip代替zip，需要from itertools import izip。

### [CF43A](http://codeforces.com/contest/43/problem/A) Football

题面：两球队比赛，进行了n场，给出每场获胜的队名，输出赢场数最多的队名。

```python
n = input()
print sorted([raw_input() for i in xrange(n)])[n/2]
```

### [CF672B](http://codeforces.com/contest/672/problem/B) Different is Good

题面：给定一个由小写字母组成的长度为n的字符串s，修改其中k个字符，使得s的所有子串各不相同，求k的最小值，如不存在输出-1。

```python
n, s = input(), raw_input()
print n - len(set(s)) if n <= 26 else -1
```

### [CF219A](http://codeforces.com/contest/219/problem/A) k-String

题面：给定数字k和字符串s，能否通过重排s构成新的串，使得该串正好可划分成k个相同的串。

```python
k, s = input(), sorted(raw_input())
a = s[::k] * k
print ''.join(a) if sorted(a) == s else -1
```

### [CF831B](http://codeforces.com/contest/831/problem/B) Keyboard Layouts

题面：给出a-z的两种排列，并给定基于第一种排列的字符串s(可能含大写字母、数字等字符)，输出对应第二种排列，要求保持原大小写。

```python
I = raw_input
a, b, s = I(), I(), I()
c = {}
for i, j in zip(a, b):
    c[i] = j
    c[i.upper()] = j.upper()
print ''.join([c.get(i, i) for i in s])
```

```python
from string import maketrans
I = raw_input
a, b, s = I(), I(), I()
print s.translate(maketrans(a + a.upper(), b + b.upper()))
```

### [CF4C](http://codeforces.com/contest/4/problem/C) Regstration system

题面：给定数字n，随后有n条注册请求，每条请求为一个只含小写字母的字符串，代表用户名，如果用户名在系统中不存在，则插入并返回"OK"，否则返回该用户名，并在后面带上最小的数字，编号以1开头。

```python
n = input()
d = {}
for i in xrange(n):
    w = raw_input()
    d[w] = d.get(w, 0) + 1
    print 'OK' if d[w] == 1 else w + str(d[w] - 1)
```

### [CF711A](http://codeforces.com/contest/711/problem/A) Bus to Udayland

题面：用一个二维字符数组表示客车座位情况，固定有5列，第1,2列和第4,5列为座位，第3列为过道，O表示空位，X表示已有人坐，要求找两个相邻的空座，改用+表示，并输出。

```python
import sys, re
input()
s = sys.stdin.read()
if re.search('OO', s):
    print 'YES'
    print re.sub('OO', '++', s, 1)
else:
    print 'NO'
```

### [CF489C](http://codeforces.com/contest/489/problem/C) Given Length and Sum of Digits

题面：给定正数m和非负整数s，构造数字x，满足x正好有m位，且各位数字之和恰好为s，求x的最小值和最大值。

```python
def fmin(n, s):
    ans = 0
    for i in xrange(0, n):
        x = 0 if i or s == 0 else 1
        y = max(s - 9 * (n - i - 1), x)
        ans = ans * 10 + y
        s -= y
    return ans

def fmax(n, s):
    ans = 0
    for i in xrange(n):
        t = min(s, 9)
        ans = ans * 10 + t
        s -= t
    return ans

def possible(n, s):
    if s == 0: return n == 1
    return n * 9 >= s

m, s = map(int, raw_input().split())
if not possible(m, s):
    print -1, -1
else:
    print fmin(m, s), fmax(m, s)
```
