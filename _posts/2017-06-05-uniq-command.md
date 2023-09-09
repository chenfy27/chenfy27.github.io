---
layout: post
title: shell命令之uniq
category: 运维
keywords:
---

uniq命令用于处理文件中的重复行，一般与sort命令并用。

常用选项：

- -c: 以行为单位统计出现次数
- -d: 只打印重复出现的行
- -u: 只打印没重复出现的行
- -i: 比较时忽略大小写
- -f num: 比较时忽略前num个字段
- -s num: 比较时忽略前num个字符
