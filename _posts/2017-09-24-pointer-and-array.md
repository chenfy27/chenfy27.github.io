---
layout: post
title: 理解指针数组与数组指针
category: cpp
keywords:
---

指针数组和数组指针是比较重要、且容易用错的概念，这里用一个例子加以说明。

具体要求：在主函数中定义变量，将其作为出参传入Generate函数，该函数动态分配内存，返回一个字符串数组，然后再将其传入Print函数分别打印，最后传入Free函数释放动态分配的内存。

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
int Generate(char ***p) {
    *p = calloc(2, sizeof(void*));
    (*p)[0] = strdup("hello");
    (*p)[1] = strdup("world");
    return 2;
}
void Print(char **p, int n) {
    for (int i = 0; i < n; i++)
        printf("%s\n", p[i]);
}
void Free(char ***p, int n) {
    for (int i = 0; i < n; i++)
        free((*p)[i]);
    free(*p);
}
int main() {
    char **p = NULL;
    int cnt = Generate(&p);
    Print(p, cnt);
    Free(&p, cnt);
    return 0;
}
```

注意

- 下标运算符[]的优先级比指针解引用\*要高。
- \*p[x]: p是数组，数组中的元素为指针，表示解引用下标为x的指针。
- (\*p)[x]: p是指针，指向一个数组，表示取数组中下标为x的元素。
