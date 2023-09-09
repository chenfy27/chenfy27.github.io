---
layout: post
title: dll的编写与调用
category: 平台
keywords:
---

Windows环境中的库分为静态库(.lib)和动态库(.dll)。

对于静态库，在编译链接成可执行文件时，链接器从库中复制函数和数据，并把它们和应用程序的其他模块组合起来形成最终的可执行文件(.exe)，发布产品时，只需要发布可执行文件。

对于动态库，一般提供两个文件：一个引入库(.lib)和一个DLL(.dll)。注意虽然后缀名相同，但动态库的引入库文件与静态库有本质区别，对于一个DLL来说，其.lib文件包含该DLL导出的函数和变量的符号名，而.dll文件包含该DLL实际的函数和数据。使用动态库时，在编译链接阶段，只需要链接该DLL的引入库文件，该DLL中的函数代码和数据并不复制到可执行文件中，直到可执行程序运行时，才去加载所需的DLL，将DLL映射到进程的地址空间中，然后访问DLL中导出的函数。在发布时，除可执行程序外，同时还要发布该程序将要调用的动态链接库。

DLL的加载分为隐式加载和显示加载两种方式。

### 编写DLL

为简单起见，下面以两数相减为例进行说明。先建立DLL工程，编写接口函数。

```c
__declspec(dllexport) int sub(int a, int b)
{
    return a - b;
}
```

这样编译就可以得到lib文件和dll文件了。为方便调用者使用，建议增加头文件，对导出函数进行声时，供调用方包含。

```c
#ifndef TEST_DLL_H
#define TEST_DLL_H

__declspec(dllimport) int sub(int a, int b);

#endif
```

其中，标识符\_declspec(dllimport)表示函数是从动态库引入的，可以换成extern或者省略，但建议加上，因为存在该标识时编译器会对代码做优化。

如果采用C++编写，并且不希望做名字改编，可声明为：`extern "C" _declspec(dllimport)`。虽然extern C可以解决C++和C之间相互调用时函数命令问题，但这种做法不能用于导出一个类的成员函数，只能用于导出全局函数。

### 隐式加载DLL

下面以隐式加载的方式调用接口。

```c
#include <stdio.h>
#include "testdll.h"
#pragma comment(lib, "testdll.lib")
int main()
{
    int x, y;
    while (~scanf("%d%d", &x, &y))
        printf("%d\n", sub(x, y));
    return 0;
}
```

将testdll.h和testdll.lib文件拷到工程目录下编译即可。注意，如果没有提供testdll.h头文件，在调用处直接声明也是可以的。

编译通过后生成可执行文件，将testdll.dll放到同一目录下即可正常运行。使用DLL的一个好处在于如果接口内部做了修改，调用者无需重新编译，例如将接口实现重新定义为：

```c
int sub(int a, int b)
{
    return b - a;
}
```

此时，只要重新生成dll文件替换过去即可生效。另外，如果在原DLL基础上扩展了新的接口，对调用方也没有任何影响。例如，扩展了加法接口。

```c
__declspec(dllexport) int add(int a, int b)
{
    return a + b;
}
```

调用方只用原接口的话无需做任何修改，便可照常运行。

### 显式加载DLL

用显式加载方式时要用到LoadLibrary函数和GetProcAddress函数，原型如下：

```c
HINSTANCE LoadLibray(LPCTSTR lpLibFileName);
FARPROC GetProcAddress(HMODULE hModule, LPCWSTR lpProcName);
```

调用接口的示例代码如下：

```c
#include <stdio.h>
#include <windows.h>
int main()
{
    int x, y;
    typedef int (*Func)(int, int);
    HINSTANCE h = LoadLibrary("dlltest.dll");
    Func sub = (Func)GetProcAddress(h, "sub");
    while (~scanf("%d%d", &x, &y))
        printf("%d\n", sub(x, y))
    FreeLibrary(h);
    return 0;
}
```

可以看到，采用显式加载方式时，调用方编译链接过程完全不需要被调用DLL的参与。

### 遗留问题

- stdcall
- def文件
