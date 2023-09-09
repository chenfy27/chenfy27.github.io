---
layout: post
title: python风格代码荟萃
category: python
keywords:
---

### 遍历一个范围内的数字

```python
for i in xrange(6):
    print i ** 2
```

xrange会返回一个迭代器，用来一次一个值地遍历一个范围，这种方式比range更省内存。在python3中xrange已经改名为range。

### 遍历集合

```python
colors = ['red', 'green', 'blue', 'yellow']
for color in colors:
    print color
```

### 反向遍历集合

```python
for color in reversed(colors):
    print color
```

### 遍历集合及其下标

```python
for i, color in enumerate(colors):
    print i, '-->', color
```

### 遍历两个集合

```python
names = ['raymond', 'rachel', 'mattthew']
colors = ['red', 'green', 'blue', 'yellow']
for name, color in izip(names, colors):
    print name, '-->', color
```

zip在内存中生成一个新的列表，需要更多的内存，izip比zip效率更高。在python3中，izip改名为zip，替换了原来的zip成为内置函数。

### 有序遍历

```python
colors = ['red', 'green', 'blue', 'yellow']
for color in sorted(colors):
    print color
for color in sorted(coloes, reverse = True):
    print color
```

### 自定义排序顺序

```python
colors = ['red', 'green', 'blue', 'yellow']
print sorted(colors, key=len)
```

### 列表解析和生成器

```python
print sum(i ** 2 for i in xrange(10))
```

### 在循环内识别多个退出点

```python
def find(seq, target):
    for i, value in enumerate(seq):
        if value == target:
            break
    else:
        return -1
    return i
```

### 分离临时上下文

```python
with open('help.txt', 'w') as f:
    with redirect_stdout(f):
        help(pow)
```

上述代码用于演示如何临时把标准输出重定向到一个文件，然后再恢复正常。注意redirect_stdout在python3.4加入。

### 打开关闭文件

```python
with open('data.txt') as f:
    data = f.read()
```

### 使用锁

```python
lock = threading.Lock()
with lock:
    print 'critical section 1'
    print 'critical section 2'
```

### 用字典计数

```python
colors = ['red', 'green', 'red', 'blue', 'green', 'red']

d = {}
for color in colors:
    d[color] = d.get(color, 0) + 1

d = defaultdict(int)
for color in colors:
    d[color] += 1
```

