---
layout: post
title: shell常用工具
keywords:
category: 运维
---

### 查看集合列表

```
kill -l
iconv -l
man ascii
```

### 查看配置文件说明

```
man sshd_config
man yum.conf
```

### 将内容对齐输出

```
cat /etc/passwd | column -t -s:
```

### 快速生成大文件

```
dd if=/dev/zero of=bigfile bs=1M count=100
```

### 反选删除文件

```
shopt -s extglob
rm -f !(*.pdf)
```

### 设置uid及sticky bit

```
chmod u+s test.txt
chmod u-s test.txt
chmod a+t dir; chmod 1755 dir
chmod a-t dir; chmod 0755 dir
```

### 查看进程信息

```
ps -ft pts/1
ps -fp 1741,1744
ps -fu chenfy,root
ps -fC httpd
pidof httpd
top -p 1447
```

### 系统时间

```
date -s 2018-01-18
date -s 17:34:12
date -s "2018-01-18 17:34:12"
date +%s
date "+%y-%m-%d %H:%M:%S"
date -d +2day
date -d "2013-1-1" +%s
date -d @1516254655
```

### 查找安装包

```
$ type md5sum
md5sum is /usr/bin/md5sum
$ rpm -qf /usr/bin/md5sum
coreutils-8.22-15.el7.x86_64

$ yum provides md5sum
coreutils-8.22-18.el7.x86_64 : A set of basic GNU tools commonly used in shell scripts
Repo        : base
Matched from:
Filename    : /usr/bin/md5sum
```

### sudo无需密码

```
echo "chenfy ALL=(ALL) NOPASSWD:ALL" >>/etc/sudoers
```

### 命令行参数封装

```
for i in "$@"; do
    echo $i
done
/path/to/exe/file $*
```

### unix与dos格式互转

最简单的方法是直接用unix2dos和dos2unix命令，直接在原文件的基础上改。

```
dos2unix file.txt
unix2dos file.txt
```

如果没有安装此工具，可通过sed命令来实现。

```
sed -i 's/\r//' file.txt
sed -i 's/$/\r/' file.txt
```

### rzsz传文件异常

用rz与sz上传下载二进制文件，有时会异常中断，原因在于默认用的文本模式传，解决办法是使用-b参数。

```
alias rz='rz -yb'
alias sz='sz -yb'
```

其中-y参数表示目标存在时覆盖。

### 查看文件的头与尾

head和tail分别用于查看文件的头与尾，一般有两种用法。以head为例，可以查看文件的前N行，也可以查看除去末尾N行外的全部行。

```
seq 5 | head -n 2
seq 5 | head -n -2
```

tail命令有类似的用法。

```
seq 5 | tail -n 2
seq 5 | tail -n +2
```

其中-n是以行为单位为处理的，如果用-c，则按字符为单位处理。

