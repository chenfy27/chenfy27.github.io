---
layout: post
title: 搭建课程数据库环境
category: 数据库
tag:
---

本文记录解决[一个简单的课程数据库](https://www.shiyanlou.com/challenges/2650)问题过程。

### 背景描述

为测试一个新的功能组件，现需要构建一个简单的课程数据库。

请尝试在MySQL中创建一个名为shiyanlou的数据库，建立用户表、课程表和学习记录表，并导入以下csv数据格式的数据。

- [user.csv]({{"/data/shiyanlou_user.csv" | prepend: site.baseurl}})：1000名用户数据，包含两列，分别为用户ID和用户名
- [course.csv]({{"/data/shiyanlou_course.csv" | prepend: site.baseurl}})：10门课程数据，包含两列，分别为课程ID和课程名
- [usercourse.csv]({{"/data/shiyanlou_usercourse.csv" | prepend: site.baseurl}})：100条用户课程学习记录，包含三列，分别是用户ID，课程ID和学习时间

### 具体要求

1. 让MySQL服务处于运行状态（最初没启MySQL服务，用户名root，密码为空）
2. 新的数据库名为shiyanlou，设置的可读写管理用户为shiyanlou/shiyanlou。
3. shiyanlou数据库包含3个表：user, course, usercourse，分别对应csv中的数据，注意usercourse表包含四列，即id, user_id, course_id, study_time。

### 搭建过程

查看csv数据文件，发现course.csv中包括中文字符，对于宽字符，需要考虑编码问题。

先看下数据文件用的什么编码格式：

```
$ head -n1 shiyanlou_course.csv | hexdump -C
00000000  36 33 2c e6 96 b0 e6 89  8b e6 8c 87 e5 8d 97 e4  |63,.............|
00000010  b9 8b e7 8e a9 e8 bd ac  e5 ae 9e e9 aa 8c e6 a5  |................|
00000020  bc 0a                                             |..|
```

显然，中文内容采用的utf8编码，为避免转码操作，这里建库时直接指定为utf8编码。另外需要注意的是行尾结束符是0x0a(\'\n\')，这个在导入时要用到。

```
$ sudo mysqld_safe &>/dev/null &
$ mysql -uroot 
mysql> create database shiyanlou default character set utf8;
mysql> use shiyanlou;
```

接下来建立数据表，并导入数据。

```
mysql> create table user(
    -> id int not null primary key auto_increment,
    -> name varchar(128));
mysql> load data infile "/tmp/user.csv" into table user
    -> character set utf8
    -> fields terminated by ','
    -> lines terminated by '\n';

mysql> create table course(
    -> id int not null primary key auto_increment,
    -> name varchar(128));
mysql> load data infile "/tmp/course.csv" into table course
    -> character set utf8
    -> fields terminated by ','
    -> lines terminated by '\n';
```

为确保数据已正确导入，这里手工检查下，需要先将环境编码调成utf8。

```
mysql> show variables like 'char%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | latin1                     |
| character_set_connection | latin1                     |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | latin1                     |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)

mysql> set names utf8;
```

查看中文部分内容，看能否正常显示，如下结果表示导入数据正常。

```
mysql> select name, hex(name) from course limit 1;
+-------------------+------------------------------------+
| name              | hex(name)                          |
+-------------------+------------------------------------+
| Linux基础入门 | 4C696E7578E59FBAE7A180E585A5E997A8 |
+-------------------+------------------------------------+
1 row in set (0.00 sec)
```

至于关系表usercourse，由于数据文件只有三列，先按三列建表，并导入数据。

```
mysql> create table usercourse(
    -> user_id int not null,
    -> course_id int not null,
    -> study_time int not null default 0,
    -> foreign key fk_uid(user_id) references user(id) on update cascade on delete update,
    -> foreign key fk_cid(course_id) references course(id) on update cascade on delete restrict);

mysql> load data infile "/tmp/usercourse.csv" into table usercourse
    -> character set utf8
    -> fields terminated by ','
    -> lines terminated by '\n';
```

然后再补上自增的id列，加列默认为最后一列，可用after fieldname指定位置，如要放到首列，用first即可。

```
mysql> alter table usercourse add id int not null primary key auto_increment first;
```

最后，还需要添加访问该数据库的帐户。

```
mysql> grant all privileges on shiyanlou.* to shiyanlou identified by "shiyanlou";
mysql> flush privileges;
```

退出root用户，以新建的shiyanlou用户登录，检查能否正常访问库表。

```
$ mysql -ushiyanlou -pshiyanlou shiyanlou
mysql> use database shiyanlou;
```

均可正常访问。到此，一个简单的课程数据库环境就搭好了。

### 知识点

- MySQL服务管理，用户管理
- 将数据文件导入MySQL
- 中文编码问题处理
- 基本的建库建表语句，外键约束
