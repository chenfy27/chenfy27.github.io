---
layout: post
title: linux静态库和动态库
category: 平台
keywords:
---

### 基本概念

库从本质上来说是一种可执行代码的二进制格式，可以被载入到内存中执行。库分为静态库和动态库两种。

在Linux下，静态库的名字一般为libxxx.a，其中xxx为库的名字，使用静态库时，整个库的所有数据都会被整合到目标代码中，因而文件比较大，执行时不再需要外部库的支持，如果库函数有变化，程序必须重新编译。

动态库的名字一般是libxxx.M.N.so，其中xxx为库名，M是主版本号，N是副版本号，版本号是可选的，但名字必须要有。动态库在编译时并没有被编进目标代码中，程序执行到相关函数时才调用库中的相应函数，因此可执行文件较小，程序运行环境必须提供相应的动态库。动态库的修改不影响程序，因而升级比较方便。

对于静态库，链接器会找出程序需要的函数，将它们拷到执行文件，由于这种拷贝是完整的，一旦链接成功，静态库也就不再需要了。对于动态库，则会在程序内打个标记指明程序运行时，要先载入这个库。由于动态库节省空间，在链接时会优先去链动态库，即如果同时存在静态库和动态库，不特别指定的话，将与动态库链接。

### 静态库的制作与使用

为方便说明，建立add.c和sub.c两个文件，分别编写加法和减法接口如下：

```
int add(int a, int b)
{
    return a + b;
}
```

```
int sub(int a, int b)
{
    return a - b;
}
```

编译生成静态库文件：

```
# gcc -c add.c sub.c
# ar -rsv libtest.a add.o sub.o
```

这样，静态库是做好了，用`ar -t libtest.a`可查看该库包含哪些obj文件。接下来写程序调用静态库中的接口。

```
#include <stdio.h>
int main()
{
    int x, y;
    while (~scanf("%d%d", &x, &y))
        printf("%d %d\n", add(x, y), sub(x, y));
    return 0;
}
```

在编译时，指定链接静态库，得到的可执行程序可独立运行。

```
# gcc -o call call.c -L. -ltest
```

### 动态库的制作和使用

当程序因动态库问题无法运行时，可用ldd命令查看该程序到底依赖了哪些so文件。在Linux系统中，动态链接库的配置文件一般在/etc/ld.so.conf文件内，该文件会包含/etc/ld.so.conf.d目录。当修改了ld.so.conf或者/ld.so.conf.d目录下的文件，或者往该目录下拷贝了新的动态库时，要执行ldconfig命令，它负责搜索/lib, /usr/lib以及/etc/ld.so.conf中所列的目录下可用的动态链接库文件到缓存/etc/ld.so.cache中。

执行ldconfig时，如果带了目录，则表示在缓存/etc/ld.so.cache中追加指定目录下的共享库；如果是单独运行，则它只会搜索/lib, /usr/lib和/etc/ld.so.conf列出的目录，重建/etc/ld.so.cache。

生成动态库可先生成obj文件，再先成so文件：

```
# gcc -c -fPIC add.c sub.c
# gcc -o libtest.so -fPIC -shared add.o sub.o
```

也可以一步到位：

```
# gcc -o libtest.so -fPIC -shared add.c sub.c
```

其中，-shared选项指定生成动态链接库，不用该标识外部程序将无法连接；-fPIC表示编译为位置独立的代码，不用此选项的话生成的代码是位置相关的，不能达到真正共享代码段的目的。

动态库的使用有两种方式：动态链接和动态加载。下面分别示例说明。

#### 动态链接

编译时指定链接动态库：

```
# gcc -o call call.c -L. -ltest
```

要让生成的程序能够运行，可以执行ldconfig \`pwd\`命令，也可将动态库加到搜索范围内并执行ldconfig命令。

#### 动态加载

Linux提供了一组API用于动态加载so文件。

```
#include <dlfcn.h>
void* dlopen(const char *filename, int flag);
char* dlerror(void);
void* dlsym(void *handle, const char *symbol);
int dlclose(void *handle);
```

说明：

- filename如果不以/开头，则为相对路径，将按照(1)环境变量LD_LIBRARY；(2)/etc/ld.so.cache；(3)/lib, /usr/lib的顺序查找该文件。
- flag表示在什么时候解决未定义的符号调用，可取RTLD_LAZY和RTLD_NOW。

```
#include <stdio.h>
#include <dlfcn.h>
typedef int (*Func)(int, int);
int main()
{
    void *handle = dlopen("libtest.so", RTLD_LAZY);
    if (dlerror()) perror("dlopen");
    Func fnadd = dlsym(handle, "add");
    if (dlerror()) perror("load add");
    Func fnsub = dlsym(handle, "sub");
    if (dlerror()) perror("load sub");

    int x, y;
    while (~scanf("%d%d", &x, &y))
        printf("%d %d\n", fnadd(x, y), fnsub(x, y));
    
    dlclose(handle);
    return 0;
}
```

如果采用动态加载方式，编译时须指定`-rdynamic -ldl`选项。

```
# gcc -o call call.c -rdynamic -ldl
```
