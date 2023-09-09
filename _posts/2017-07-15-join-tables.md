---
layout: post
title: 常用的表连接
category: 数据库
keywords:
---

在使用关系型数据库时，多表联查是很常见的事情，在关联两个表时按大类可分为内连接和外连接，而外连接又有左外连接，右外连接和全外连接之分。在实际工作中使用最多的要属左外连接和内连接，下面对这两种方式做简单介绍。

内连接用inner join表示，也可简写成join；左连接用left join表示。

假设语句为A join B on condition，内连接执行过程可用如下伪代码表示：

```
for a in table A:
    for b in table B:
        if condition(a, b) holds:
            AddToResult(a, b)
```

即对表A中的每一条记录a，遍历表B中的各条记录，检查连接条件是否成立，如果成立，则将(a,b)构成一条大的记录加到结果集中。

同样地，假设语句为A left join B on condition，左连接执行过程可表示为：

```
for a in table A:
    found = false
    for b in table B:
        if condition(a, b) holds:
            AddToResult(a, b)
            found = true
    if not found:
        AddToResult(a, NULL)
```

易见，左连接与内连接的区别在于：对于主表中的某条记录，如果副表中不存在作何与之满足连接条件的记录，那么将会在结果集中插入一条仅含主表信息的记录，对应的副表信息为NULL。

下面通过一个例子加以说明：

```
mysql> select * from t1;
+------+--------+
| id   | name   |
+------+--------+
|    1 | chenfy |
|    2 | maq    |
|    2 | zhumy  |
|    5 | zhaol  |
|    4 | zhongy |
+------+--------+

mysql> select * from t2;
+------+--------+
| id   | name   |
+------+--------+
|    1 | chenfy |
|    2 | mac    |
|    3 | dengc  |
|    5 | zhaozb |
|    5 | zhangl |
+------+--------+

mysql> select t1.id, t1.name, t2.id, t2.name from t1 join t2 on t1.id=t2.id;
+------+--------+------+--------+
| id   | name   | id   | name   |
+------+--------+------+--------+
|    1 | chenfy |    1 | chenfy |
|    2 | maq    |    2 | mac    |
|    2 | zhumy  |    2 | mac    |
|    5 | zhaol  |    5 | zhaozb |
|    5 | zhaol  |    5 | zhangl |
+------+--------+------+--------+

mysql> select t1.id, t1.name, t2.id, t2.name from t1 left join t2 on t1.id=t2.id;
+------+--------+------+--------+
| id   | name   | id   | name   |
+------+--------+------+--------+
|    1 | chenfy |    1 | chenfy |
|    2 | maq    |    2 | mac    |
|    2 | zhumy  |    2 | mac    |
|    5 | zhaol  |    5 | zhaozb |
|    5 | zhaol  |    5 | zhangl |
|    4 | zhongy | NULL | NULL   |
+------+--------+------+--------+
```
