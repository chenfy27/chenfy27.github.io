---
layout: post
title: 输入不回显
category: 平台
keywords:
---

有些时候，我们希望在终端输入的内容不要(明文)显示，比如键入的密码口令，下面针对不同的要求整理了几种方法。

### Shell服本

shell中的read命令提供了-s选项，简单方便。

### Windows下C语言

可借助conio.h文件提供的接口实现。

```
#include <conio.h>
int sread(char *prompt, char *buf) {
    int i = 0;
    char ch;
    printf("%s", prompt);
    while ((ch = getch()) != '\r') {
        if (ch == '\b') {
            printf("\b \b");
            if (i) i -= 1;
        } else {
            buf[i++] = ch;
            putchar('*');
        }
    }
    buf[i] = 0;
    return i;
}
```

### Linux下C语言

Linux下有现成的函数getpass实现此功能，但manual上说该函数已废弃，可以通过termios.h提供的接口来实现。

```
#include <termios.h>
void toggle_echo() {
    struct termios tc;
    tcgetattr(STDIN_FILENO, &tc);
    tc.c_lflag ^= ECHO;
    tcsetattr(STDIN_FILENO, TCSAFLUSH, &tc);
}
int sread(char *prompt, char *buf) {
    int i = 0;
    char ch;
    printf("%s", prompt);
    toggle_echo();
    while ((ch = getchar()) != '\n') {
        if ('\b' == ch) {
            if (i) i -= 1;
        } else buf[i++] = ch;
    }
    toggle_echo();
    buf[i] = 0;
    return i;
}
```

