---
layout: post
title: shell命令之find
category: 运维
keywords:
---

find命令用于查找文件，支持的查找方式很多，功能强大，本文仅列举常用的选项。

### 按路径名/文件名搜索

- -name pattern: 文件基本名匹配shell模式pattern。
- -iname pattern: 与-name类似，但忽略大小写。
- -path pattern: 路径匹配模式。
- -ipath pattern: 与-path类似，但忽略大小写。
- -regex pattern: 匹配文件名或目录。
- -iregex pattern: 同-regex，但忽略大小写。

### 按文件时间搜索

- -amin 分钟: 在指定时间内被读取过的文件或目录
- -cmin 分钟：在指定时间内状态被修改过的文件或目录
- -mmin 分钟：在指定时间内内容被修改过的文件或目录
- -atime 天数：同-amin，单位为天
- -ctime 天数：同-cmin，单位为天
- -mtime 天数：同-mmin，单位为天

### 按文件类型搜索

- -type t: 文件类型为t，支持的文件类型如下：
	- b: 块设备
	- c: 字符设备
	- d: 目录
	- f: 普通文件
	- l: 符号链接
	- p: FIFO
	- s: socket

### 按文件大小搜索

- -size [+-]n[ckMGTP]

说明：

- 2k指1k~2k
- -2k指0k~1k
- +2k指2k到无穷大

### 按文件属主搜索

- -user username: 文件属主为username。
- -uid userid: 文件属主ID为userid。
- -group grpname: 文件所属用户组为grpname。
- -gid grpid: 文件所属用户组ID为grpid。

### 按文件权限搜索

- -perm [+-]mode

### 搜索并执行操作

- -print：默认动作，打印至屏幕
- -ls：显示找到文件的详细属性
- -exec command {} \;
- -ok command {} \;

注意：-ok会提供交互式以便确认，而-exec不会。

### 逻辑组合

- 与：-a
- 或：-o
- 非：-not

如果不指定两个条件之前的逻辑关系，默认按与处理。

### 重要提示

- 使用通配符\*和?指定find要查找的文件模式时，要用单、双引号或\\符号转移通配符，保证通配符能正确地传递给find，否则shell会对通配符进行扩展。
- 使用-exec执行的命令后要用转义后的分号\\;结束，否则find命令会报错说找不到-exec参数。
- 当指定条件逻辑组合时，要对所使用的括号进行转义，添加反斜杠，并且\\(和\\)的两侧都要有空格。
- 逻辑与连接符-a可以省略，但逻辑或连接符-o不能省。

### 例子

```
find . \( -name "tags" -o -name "cscope*" \) -exec rm -f {} \;
find . ! -name "*.h" -a -type f
find /usr/include -iname "*.h" -exec cp -f {} /tmp/linux \;
find . -iregex ".*\(\.txt\|\.pdf\)$"
find . -maxdepth 3 -type f -atime -7 -size -10k
find . -type f -name "*.txt" -perm 777 -delete
find . -type f -name "*.php" ! -perm 644
find . -type f -name "*.txt" -exec cat {} \; >all.txt
find . -type f -mtime +30 -name "*.log" -exec cp {} old \;
find . -type f -name "*.txt" -exec printf "File: %s\n" {} \;
find . -empty
find . -path "include" -prune -o -name "*.txt" -print
```
