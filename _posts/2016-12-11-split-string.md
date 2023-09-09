---
layout: post
title: 分割字符串
category: cpp
keywords:
---

很多时候为了方便，会将多个字段的值用特定的符号分隔拼成一个字符串发送，而接收端则需要拆分解析得到各个字段的值，以下是几种常见的做法。

### 利用strsep函数

函数原型为：`char *strsep(char **stringp, const char *delim);`

```cpp
void split1(char *str, char *delim) {
    char *token;
    for (token = strsep(&str, delim); token; token = strsep(&str, delim))
        printf("[%s]", token);
    puts("");
}
```

例如输入字符串为：

```
chenfy:x:1000:1000::/home/chenfy:/bin/bash
```

分隔符为冒号，运行结果为：

```
[chenfy][x][1000][1000][][/home/chenfy][/bin/bash]
```

### 通过strtok函数

该函数与strsep类似，原型为：`char *strtok(char *str, const char *delim);`

```cpp
void split2(char *str, char *delim) {
    char *token;
    for (token = strtok(str, delim); token; token = strtok(NULL, delim))
        printf("[%s]", token);
    puts("");
}
```

对于同样的输入字符串结果为：

```
[chenfy][x][1000][1000][/home/chenfy][/bin/bash]
```

strtok函数内部使用了静态变量，不可重入，因此只能用在单线程环境中。如果需要在多线程环境中使用，应改用线程安全版本strtok_r。

函数原型为：`char *strtok_r(char *str, const char *delim, char **saveptr);`

```cpp
void split3(char *str, char *delim) {
    char *token, *p;
    for (token = strtok_r(str, delim, &p); token; token = strtok_r(NULL, delim, &p))
        printf("[%s]", token);
    puts("");
}
```

### 自定义分割函数

strsep不是标准的C库函数，可移植性不如strtok，另外strsep同样存在线程不安全的问题，但它有个特点有些时候很有用：连续出现分隔符出来的是空字符串。

下面是自己实现的分割函数。

```cpp
int split(char *str, char delim, char buf[][64]) {
    int p1 = 0, p2 = 0, cnt = 0, len = 1 + strlen(str);
    while (p1 < len) {
        while (str[p2] && str[p2] != delim) p2++;
        str[p2] = 0;
        strcpy(buf[cnt++], str + p1);
        p1 = p2 = p2 + 1;
    }
    return cnt;
}
```

