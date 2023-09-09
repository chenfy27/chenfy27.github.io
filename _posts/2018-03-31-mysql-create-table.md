---
layout: post
title: 搭建简易成绩数据库
category: 数据库
tag:
---

本文记录解决挑战[搭建一个简易的成绩管理系统的数据库](https://www.shiyanlou.com/courses/9/labs/2769/document)的过程。

### 目标要求

- MySQL服务处于运行状态
- 新建数据库的名称为gradesystem
- gradesystem包含三张表
    - student: sid(主键), sname, gender
    - course: cid(主键), cname
    - mark: mid(主键), sid, cid, score

### 解决过程

首先启服mysqld服务，并登录。

```
$ sudo service mysql start
$ mysql -uroot
```

建立gradesystem数据库。

```
mysql> create database gradesystem;
mysql> use gradesystem;
```

依次建立student, course, mark表。

```
mysql> create table student(
    -> sid int primary key auto_increment,
    -> sname varchar(32) not null,
    -> gender varchar(16) not null);

mysql> show create table student;
CREATE TABLE `student` (
  `sid` int(11) NOT NULL AUTO_INCREMENT,
  `sname` varchar(32) NOT NULL,
  `gender` varchar(16) NOT NULL,
  PRIMARY KEY (`sid`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
```

```
mysql> create table course(
    -> cid int primary key auto_increment,
    -> cname varchar(32) not null);

mysql> show create table course;
CREATE TABLE `course` (
  `cid` int(11) NOT NULL AUTO_INCREMENT,
  `cname` varchar(32) NOT NULL,
  PRIMARY KEY (`cid`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 
```

```
mysql> create table mark(
    -> mid int primary key auto_increment,
    -> sid int,
    -> cid int,
    -> score int not null,
    -> foreign key(sid) references student(sid),
    -> foreign key(cid) references course(cid));

mysql> show create table mark;
CREATE TABLE `mark` (
  `mid` int(11) NOT NULL AUTO_INCREMENT,
  `sid` int(11) DEFAULT NULL,
  `cid` int(11) DEFAULT NULL,
  `score` int(11) NOT NULL,
  PRIMARY KEY (`mid`),
  KEY `sid` (`sid`),
  KEY `cid` (`cid`),
  CONSTRAINT `mark_ibfk_1` FOREIGN KEY (`sid`) REFERENCES `student` (`sid`),
  CONSTRAINT `mark_ibfk_2` FOREIGN KEY (`cid`) REFERENCES `course` (`cid`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
```

最后是将数据插入各个表中。

```
mysql> insert into student(sname, gender) values
    -> ('Tom', 'male'),
    -> ('Jack', 'male'),
    -> ('Rose', 'female');
mysql> select * from student;
+-----+-------+--------+
| sid | sname | gender |
+-----+-------+--------+
|   1 | Tom   | male   |
|   2 | Jack  | male   |
|   3 | Rose  | female |
+-----+-------+--------+
```

```
mysql> insert into course(cname) values('math'), ('physics'), ('chemistry');
mysql> select * from course;
+-----+-----------+
| cid | cname     |
+-----+-----------+
|   1 | math      |
|   2 | physics   |
|   3 | chemistry |
+-----+-----------+
```

```
mysql> insert into mark(sid,cid,score) values
    -> (1,1,80), (2,1,85), (3,1,90),
    -> (1,2,60), (2,2,90), (3,2,75),
    -> (1,3,95), (2,3,75), (3,3,85);
mysql> select * from mark;
+-----+------+------+-------+
| mid | sid  | cid  | score |
+-----+------+------+-------+
|   1 |    1 |    1 |    80 |
|   2 |    2 |    1 |    85 |
|   3 |    3 |    1 |    90 |
|   4 |    1 |    2 |    60 |
|   5 |    2 |    2 |    90 |
|   6 |    3 |    2 |    75 |
|   7 |    1 |    3 |    95 |
|   8 |    2 |    3 |    75 |
|   9 |    3 |    3 |    85 |
+-----+------+------+-------+
```
