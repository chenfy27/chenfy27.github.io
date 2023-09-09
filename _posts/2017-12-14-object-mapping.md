---
layout: post
title: 对象映射到数字
keywords:
category: 算法
---

很多时候，待处理的数据对象其唯一标识不是简单的数字，比如可能是个字符串，或者是个元组，甚至更复杂的复合结构，直接处理很不方便，且计算效率差，通常会考虑做个映射，将每个不同的唯一标识映射成唯一的数字，并提供标识与数字之间互转的方法，这样可大大简化代码，提升效率。

以下是该做法的示例程序，以字符串唯一标识进行说明，但不限于字符串，可以是任意类型。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef string TYPE;
map<TYPE,int> kcache;
vector<TYPE> vcache;
int ID(TYPE s) {
    if (kcache.count(s))
        return kcache[s];
    vcache.push_back(s);
    return kcache[s] = vcache.size() - 1;
}
TYPE DI(int id) {
    return vcache[id];
}
int main() {
    string x;
    while (cin >> x)
        cout << "[" << x << "] => " << ID(x) << endl;
    for (int i = 0; i < vcache.size(); i++)
        cout << "key: " << i << "; val: [" << DI(i) << "]" << endl;
    return 0;
}
```

对于输入数据：

```
e c e b c e a a a f
```

运行结果如下：

```
[e] => 0
[c] => 1
[e] => 0
[b] => 2
[c] => 1
[e] => 0
[a] => 3
[a] => 3
[a] => 3
[f] => 4
key: 0; val: [e]
key: 1; val: [c]
key: 2; val: [b]
key: 3; val: [a]
key: 4; val: [f]
```

