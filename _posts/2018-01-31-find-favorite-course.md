---
layout: post
title: 查找最爱学的课程
category: 数据库
tag:
---

本文记录解决[查找最爱学的课程](https://www.shiyanlou.com/challenges/2651)问题的过程。

### 背景描述

[前面]({{"/2018/01/30/build-simple-database.html" | prepend: site.baseurl}})我们已经搭了个简单的课程数据库，现在要根据表中数据找出每个用户最爱学的课程。

### 目标要求

1. MySQL服务处理运行状态
2. 查询并将查询结果创建一个新表favorite，包含四列：id, user_name, course_name, study_time。
3. favorite表中存储的是所有在usercourse表中有学习记录的用户学习时间最长的课程，如果有多门课程学习时间相同，则都存入该表。

### 实现过程

首先启动MySQL服务。

```
$ sudo mysqld_safe &>/dev/null &
```

将查询结果存入新表可以使用`create table tbname as select`语句，而id列可以最后补上，主要难点在于如何查找分组中的最大值。

按正常思路直接查询SQL可能不太好写，不妨反过来想，最大其实等价于没有比它更大的，这样可以转化为计数子查询来解决。

```
create table favorite as 
select u.name user_name, c.name course_name, uc.study_time
  from usercourse uc
  join user u on u.id=uc.user_id
  join course c on c.id=uc.course_id
  where 1 > (
    select count(distinct(uc1.user_id)) from usercourse uc1
	  where uc1.user_id=uc.user_id and uc1.study_time>uc.study_time
  );
```

最后需要补充id列。

```
alter table favorite add id int not null primary key auto_increment first;
```
