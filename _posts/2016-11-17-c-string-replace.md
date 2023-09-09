---
layout: post
title: c字符串替换
category: cpp
keywords:
---

C语言库函数中没有现成的字符串替换函数，需要手写，由于字符串替换是个常用的工具，这里记录一下。

```cpp
int strrep(char *str, char *oldstr, char *newstr, int count) {
    int offset = 0, len = strlen(str), oldlen = strlen(oldstr), newlen = strlen(newstr);
    while (count--) {
        char *p = strstr(str + offset, oldstr);
        if (!p) return len;
        offset = p - str;
        memmove(p + newlen, p + oldlen, len - offset - oldlen + 1);
        memcpy(p, newstr, newlen);
        len += newlen - oldlen;
        offset += newlen;
    }
    return len;
}
```

说明：接口返回最终字符串的长度，参数count代表最多替换次数，如需全部替换，建议填-1。

有些时候，希望在匹配时不区分大小写，若匹配成功则进行替换。以下是一种实现，思路是复制一个小写的样本用于查找需要替换的位置，而替换操作在原字符串上进行。

```cpp
int strirep(char *str, char *oldstr, char *newstr, int count) {
    int i, offset = 0, pos, cnt = 0;
    int len = strlen(str), oldlen = strlen(oldstr), newlen = strlen(newstr);
    char *lowstr = (char*)malloc(len+1), *lowold = (char*)malloc(oldlen+1);
    for (i = 0; i <= len; i++)    lowstr[i] = tolower(str[i]);
    for (i = 0; i <= oldlen; i++) lowold[i] = tolower(oldstr[i]);
    while (count--) {
        char *p = strstr(lowstr + offset, lowold);
        if (!p) return len;
        offset = p - lowstr;
        pos = offset + cnt * (newlen - oldlen);
        memmove(str + pos + newlen, str + pos + oldlen, len - pos - olelen + 1);
        memcpy(str + pos, newstr, newlen);
        len += newlen - oldlen;
        offset += oldlen;
        cnt += 1;
    }
    return len;
}
```

附：由于上述两个接口都是直接在原字符串的基础上操作，因此要求调用者保证缓冲区大小足够。
