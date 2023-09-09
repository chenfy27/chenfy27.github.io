---
layout: post
title: mysql远程访问配置
category: 运维
keywords: mysql
---

有时Mysql数据库布署在Linux服务器上，需要在远端直接访问数据库，配置步骤如下：

#### 1. 确认mysqld服务正常

```
# netstat -naopt | grep mysqld
tcp   0  0 0.0.0.0:8004   0.0.0.0:*    LISTEN   2862/mysqld   off (0.00/0/0)
```

可见mysqld服务已启动，在8004端口监听等待连接。  
mysqld服务端口配置对应/etc/my.cnf文件[mysqld]下的port选项，默认为3306。

#### 2. 创建远程访问用户

```
mysql> grant all privileges on dbname.* to chenfy@'%' identified by 'chenfy123';
mysql> flush privileges;
```

#### 3. 远程连接访问

```
# mysql -h {IP} -P {port} -uchenfy -pchenfy123
```
