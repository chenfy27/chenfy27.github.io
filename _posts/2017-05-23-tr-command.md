---
layout: post
title: shell命令之tr
category: 运维
keywords:
---

tr从标准输入读取字符，写到标准输出，在拷贝的过程中，可以做些替换和删除的操作。

### 常用选项

- -C: 取补集
- -c: 取补集
- -s: 压缩相同连续字符
- -d: 删除字符
- -u: 输出不带缓冲

### 常见用法

- tr [-cCsu] string1 string2

将string1中的字符转换成string2中的字符。如果string1长度小于string2，那么string2中多出的部分会被忽略；如果string1长度大于string2，那么string2将重复最后一个字符，直到与string1等长。

- tr [-cCu] -d string1

将存在于string1中的字符都删掉。

- tr [-cCu] -s string1

将存在于string1中的字符进行压缩，即连续出现多次同一字符时只输出一次。

- tr [-cCu] -ds string1 string2

为上面两个的组合，删除string1中的字符，并压缩string2中的字符。

### POSIX字符类

关于字符集合，tr支持POSIX字符类，见下：

- alnum: 数字+字母
- alpha: 字母
- cntrl: 控制字符
- digit: 数字
- graph: 图形字符
- lower: 小写字母
- print: 可打印字符
- punct: 标点符号
- space: 空白字符
- upper: 大写字母
- xdigit: 16进制字符

### 例子

```
echo "chenfy27" | tr "a-z" "A-Z"
echo "chenfy27" | tr -d "0-9"
echo "aaabbccccddd" | tr -s "a-d"
tr -d "\r" <dosfile >unixfile
tr -cd [[:alnum:]] <file
tr -d ": " </etc/passwd
seq 100 | tr "\n" "+" | sed 's/+$/\n/' | bc  (seq -s"+" 100 | bc)
tr -c [[:alpha:]] "\n" </etc/passwd | grep -v "^$" | sort | uniq -c | awk '{print $2, $1}'
```
