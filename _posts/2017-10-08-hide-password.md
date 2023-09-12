---
layout: post
title: 隐藏敏感信息
keywords:
category: 平台
---

传参调用其他工具来完成特定的任务是很常见的事情，当参数包含密码等敏感数据时不安全，最好能处理一下，否则在后台用ps命令一查就全知道了。最容易想到的是加密传密文，对端收到后解密得到明文。本文介绍除加密外的常用方法。

### 类C语言程序

可以在程序开始处获取命令行参数，然后将argv的内容修改掉。

```
#include <stdio.h>
#include <string.h>
#include <unistd.h>
int main(int argc, char *argv[]) {
    for (i = 1; i < argc; i++)
        puts(argv[i]);
    for (int i = 1; i < argc; i++)
        memset(argv[i], 'X', strlen(argv[i]));
    for (;;) sleep(3);
    return 0;
}
```

此时，如果运行程序`./test 62960909`，然后在后台用`ps -fC test`查看，发现参数变成了一串X。

```
[chenfy@localhost cpp]$ ./test 62960909
62960909
^Z
[1]+  Stopped                 ./test 62960909
[chenfy@localhost cpp]$ ps -fC test
UID        PID  PPID  C STIME TTY          TIME CMD
chenfy   15534 15321  0 17:05 pts/1    00:00:00 ./test XXXXXXXX
```

该方法只对类Unix系统有效，在Windows系统下，通过wmic命令仍能看到完整的参数信息。

### 类Shell脚本

假设要执行的命令为：command {password}，将其修改为：echo \"{password}\" \| command 形式即可实现隐藏。

如果除了密码外，还有其他参数，可考虑用xargs来组装。例如，以下是个输出命令参数的脚本：

```
#!/bin/bash
for i in "$@"; do echo $i; done
while true; do sleep 1; done
```

执行：echo \"abc\" \"123\" \| ./test 时，用ps查看test进程是不带参数的，另外搜echo进程也找不到对应的记录。
