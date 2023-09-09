---
layout: post
title: 记录对齐输出
category: 平台
keywords:
---

在终端用ls列出文件时，结果是按列对齐的；命令行方式执行mysql查询时，结果也显得非常整齐。这个小技巧是如何实现的呢？

最近在调研通过odbc操作数据库，遇到了类似的问题，做了个简单实现版本，做法如下：

- 先按列扫一遍结果集，求出每列的最大宽度；
- 逐条记录输出，每个字段按该列的最大宽度显示，不足补空格，对齐方式任意；
- 中文字符长度要注意编码问题，用strlen求长度时GBK为2字节，UTF-8为3字节，输出时统一占2字节宽度；

以下是关键代码：

```c
int Strlen(const char *s) {
    int i, len;
    for (i = len = 0; s[i]; i++) {
        if (s[i] < 0) i += 2, len += 1;
        len++;
    }
    return len;
}
void ShowResultTable(char **result, int row, int col) {
    char sep[1024] = {0};
    int i, j, k = 0, width[128] = {0};
    for (j = 0; j < col; j++) {
        sep[k++] = '+';
        for (i = 0; i < row; i++)
            width[j] = max(width[j], Strlen(result[i*col+j]));
        for (i = 0; i < width[j] + 2; i++)
            sep[k++] = '-';
    }
    sep[k++] = '+';
    puts(sep);
    for (i = 0; i < row; i++) {
        for (j = 0; j < col; j++) {
            int len = Strlen(result[i*col+j]);
            printf("| %s", result[i*col+j]);
            for (k = 0; k + len <= width[j]; k++)
                putchar(' ');
        }
        puts("|");
    }
    puts(sep);
}
```

上面的xlen函数只处理了UTF-8编码的情况，而且不准确，应该按照UTF-8和GBK编码范围来做。运行结果如下：

```
+--------+---------------------------+-------------+-------+
| 马学铭 | maxueming@dameng.com      | 15312348552 | 30000 |
| 程擎武 | chengqingwu@dameng.com    | 13912366391 | 9000  |
| 郑吉群 | zhengjiqun@dameng.com     | 18512355646 | 15000 |
| 陈仙   | chenxian@dameng.com       | 13012347208 | 12000 |
| 金纬   | jinwei@dameng.com         | 13612374154 | 10000 |
| 李慧军 | lihuijun@dameng.com       | 18712372091 | 10000 |
| 常鹏程 | changpengcheng@dameng.com | 18912366321 | 5000  |
| 谢俊人 | xiejunren@dameng.com      | 14712377545 | 5000  |
| 苏国华 | suguohua@dameng.com       | 15612350864 | 30000 |
| 强洁芳 | qiangjiefang@dameng.com   | 18112349195 | 10000 |
+--------+---------------------------+-------------+-------+
```
