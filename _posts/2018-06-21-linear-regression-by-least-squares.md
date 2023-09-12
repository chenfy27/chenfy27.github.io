---
layout: post
title: 线性回归之最小二乘法
category: 机器
tag:
---

### 问题描述

假设自变量$$(X_{1}, X_{2},\cdots,X_{n})$$与因变量$$Y$$之间满足某种线性关系：

$$Y = \theta_{0} + \theta_{1}X_{1} + \theta_{2}X_{2} + \cdots + \theta_{n}X_{n}$$

为了表示方便，令$$X_{0}=1$$，那么上式可用矩阵的方式简写为：

$$Y = X^{T}\theta$$

其中$$X$$和$$\theta$$为列向量。

$$
X = 
\begin{bmatrix}
    X_{0} \\
    X_{1} \\
    \cdots \\
    X_{n} \\
\end{bmatrix}
,
\theta =
\begin{bmatrix}
    \theta_{0} \\
    \theta_{1} \\
    \cdots \\
    \theta_{n} \\
\end{bmatrix}
$$

现给出$$m$$组样本数据，求参数$$\theta$$，使上述等式关系尽可能得符合各个样本。这是一个典型的线性回归问题，常见的解法是最小二乘法和梯度下降法，这里介绍最小二乘法。

### 最小二乘法

根据关系式$$Y = X^{T}\theta$$，构造残差函数$$L(\theta)=\frac{1}{2}(X^{T}\theta-Y)^{T}(X^{T}\theta-Y)$$，为了让$$L(\theta)$$取得最小值，令$$L(\theta)$$对$$\theta$$的一阶导数为零，解出$$\theta$$：

$$ \theta = (X^{T}X)^{-1} X^{T} Y$$

可见，通过最小二乘法求解$$\theta$$主要是进行矩阵乘法运算，由于矩阵乘法的时间复杂度是$$O(n^{3})$$级别的，因此只适用于数据量较小的场景。

### 求解实例

假设有线性关系$$Y = 9 + 4X_{1} + X_{2} + 2X_{3}$$，下面随机生成一些数据作为样本，用于求解系数。

```
import random

def func(x1, x2, x3):
    return 9 + 4*x1 + x2 + 2*x3

for i in range(5):
    x1 = random.randint(1, 10)
    x2 = random.randint(1, 10)
    x3 = random.randint(1, 10)
    y = func(x1, x2, x3)
    print(y, x1, x2, x3)
```

某次运行结果如下：

```
71 10 8 7
72 10 5 9
42 2 5 10
39 1 8 9
56 6 7 8
```

下面根据最小二乘法的结论来求解参数。

```
import numpy as np
from numpy import dot
from numpy.linalg import inv

X = np.array([[1,10,8,7],[1,10,5,9],[1,2,5,10],[1,1,8,9],[1,6,7,8]])
Y = np.array([[71],[72],[42],[39],[56]])

theta = dot(dot(inv(dot(X.T, X)), X.T), Y)
print(theta)
```

运行结果如下：

```
[[9.]
[4.]
[1.]
[2.]]
```

即通过回归分析得到的关系式为：$$Y = 9 + 4X_{1} + X_{2} + 2X_{3}$$，与预期相符。

再如，假定国内生产总值GDP与消费Consumption、货币供应量MO、进出口总额EM以及投资Investment这4个因素存在某种线性关系，[这里]({{"/data/yeardata.csv" | prepend: site.baseurl}})提供了一些样本数据，试着量化该线性关系。

```
import pandas as pd
import numpy as np
from numpy import dot
from numpy.linalg import inv

data = pd.read_csv('yeardata.csv')
X = data.iloc[:, 1:5]
X['X0'] = 1
X = X.iloc[:, [4,0,1,2,3]]
#print(X)
Y = data.iloc[:, 0]
#print(Y)

theta = dot(dot(inv(dot(X.T, X)), X.T), Y)
print(theta)
```

计算结果如下：

```
[-94.51398652   1.32825128   2.06138007   0.38959829   0.11848647]
```

即$$GDP = -94.514 + 1.328Consumption + 2.061MO + 0.390EM + 0.118Investment$$。
