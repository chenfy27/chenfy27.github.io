---
layout: post
title: 最小二乘法
category: 机器
tag:
---

最小二乘法也称最小平方法，是一种数学优化技术，通过最小化误差的平方和来寻找数据的最佳函数匹配。

### 原理

假设结果y与m个因素x[i]有关，且为线性关系，即：

```
y = k[0] + k[1] * x[1] + k[2] * x[2] + ... + k[m] * x[m]
```

经过测量，现有n组数据(n >= m)，求合适的k[i]，使得上述表达式与测量结果尽可能的相符。

最小二乘法的做法很简单，让所有残差的平方和最小即可，设p[i]表示第i组测试数据，把k[i]看成未知数，最小化f：

```
f = (p[1] - y[1]) ^ 2 + (p[2] - y[2]) ^ 2 + ... + (p[n] - y[n]) ^ 2
```

要使f取得最值，分别让f对k[i]对偏导，令其等于0，可得到m个线性方程，联立起来即可求出各个k[i]。

### 实例

这里举个简单例子说明最小二乘法的过程。现要用钱去买游戏币，售币处贴出公示如下：

```
 10元 —— 11个游戏币
 20元 —— 23个游戏币
 50元 —— 58个游戏币
100元 —— 123个游戏币
```

显然，付的钱数与得到的游戏币存在某种线性关系，设有：y = ax + b，代入以上数据得：

```
 10a + b = 11
 20a + b = 23
 50a + b = 58
100a + b = 123
```

构造残差函数

```
f = (10a + b - 11) ^ 2 + (20a + b - 23) ^ 2 + (50a + b - 58) ^ 2 + (100a + b - 123) ^ 2
```

分别对a, b求偏导，并令其等于0，才能使f取得最值。

```
20(10a + b - 11) + 40(20a + b - 23) + 100(50a + b - 58) + 200(100a + b - 123) = 0
 2(10a + b - 11) +  2(20a + b - 23) +   2(50a + b - 58) +   2(100a + b - 123) = 0
```

解出结果：a = 1.2439, b = -2.2245，从而关系式为：y = 1.2439x - 2.2245。

有了该表达式，现在可以对未知点进行预测了，例如80块钱，大约可以买97个游戏币，而125元钱大约可以买153个。

### python实现

```python
import numpy as np
from scipy.optimize import leastsq

def func(p, x):
    k, b = p
    return k * x + b

def error(p, x, y):
    return func(p, x) - y

X = np.array([10, 20, 50, 100])
Y = np.array([11, 23, 58, 123])
P = (0, 0)

ret = leastsq(error, P, args=(X, Y))

k, b = ret[0]
print(k, b)
```

解出k=1.24388, b=-2.2245，与上面手算的一致。可以通过pyplot将数据以图的形式直观的展现出来。

```python
import matplotlib.pyplot as plt

plt.figure()
plt.scatter(X, Y, color='red')
x = np.linspace(0, 200, 200)
y = k * x + b
plt.plot(x, y)
plt.show()
```