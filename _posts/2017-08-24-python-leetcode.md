---
layout: post
title: python练习1708
keywords:
category: python
---

- 实现pow(x, n)，其中x为浮点数，n为整数。

```python
def myPow(x, n):
    return math.pow(x, n)
```

- 给定正整数a和数组b表示的大整数，求a^b%1337的值。

```python
def superPow(a, b):
    x = 0
    for i in b:
        x = x * 10 + i
    return pow(a, x, 1337)
```

- 给定由花括号、方括号和圆括号构成的字符串，判断序列是否有效。

```python
def isValid(s):
    v = []
    d = {")":"(", "]":"[", "}":"{"}
    for i in s:
        if i in d.values():
            v.append(i)
        elif i in d.keys():
            if len(v) == 0 or d[i] != v.pop():
                return False
        else:
            return False
    return len(v) == 0
```

- 给定n对圆括号，求能构成的所有有效括号序列。

```python
def generateParenthesis(n):
    def gen(s, nl, nr, v = []):
        if nr == 0:  v.append(s)
        if nl > 0:   gen(s + '(', nl - 1, nr, v)
        if nr > nl:  gen(s + ')', nl, nr - 1, v)
        return v
    return gen('', n, n)
```
