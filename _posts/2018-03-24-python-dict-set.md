---
layout: post
title: python字典与集合常用操作
category: python
tag:
---

### 字典操作

- 创建空字典

```python
d = {}
```

- 测试键k是否为字典成员

```python
if k in d:
if d.has_key(k):
```

- 取键k对应的项

```python
# be sure k exists in dict d
d[k]

# not sure k exists in dict d
d.get(k, defval)
```

- 删除键k对应的项

```python
# be sure k exists in dict d
del d[k]
d.pop(k)

# not sure k exists in dict d
d.pop(k, defret)
```

- 遍历字典

```python
for k in d.keys(): print(k)
for v in d.values(): print(v)
for k,v in d.items(): print(k, v)
```

### 集合操作

- 创建空集合

```python
s = set()
```

- 用列表创建集合

```python
s = set([3,4,6,8])
s = set('hello')
```

- 集合运算

```python
s = a | b
s = a & b
s = a - b
s = a ^ b
```

- 添加元素

```python
# insert single element
s.add(x)

# insert elements by list
s.update([3,2,4,9])
```

- 删除元素

```python
# be sure x exists in set
s.remove(x)

# not sure x exists in set
s.discard(x)
```

- 检测x是否在集合中

```python
if x in s:
```

- 遍历集合

```python
for i in s: print(i)
for i in sorted(s): print(i)
```

