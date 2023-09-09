---
layout: post
title: windows下编译openssl
category: 平台
keywords:
---

openssl库提供了多而强的功能，可直接拿来用，Linux环境下一般默认已安装，下面介绍如何在Windows下编译。

### 准备工作

1. 下载并安装ActivePerl，注意将其bin目录添加到系统环境变量PATH中。
2. 下载并安装VS2005或更高版本，安装时勾选SDK组件。
3. 从[官网](https://www.openssl.org/source)下载openssl源码包，解压。

### 参考步骤

从开始菜单找到VS2005的命令行窗口并打开，切换到openssl源码目录，依次执行以下命令。

```
C:\openssl-0.9.8> perl Configure VC-WIN32
C:\openssl-0.9.8> ms\do_ms
```

修改ms\ntdll.mak文件中CFLAG定义，将告警级别/W3改为/W0，然后编译。

```
C:\openssl-0.9.8> nmake -f ms\ntdll.mak
```

如果编译成功，输出内容在out32dll目录下，只需要libeay32.dll, libeay32.lib, ssleay32.dll和ssleay32.lib四个文件，将需要的头文件和库文件都拷贝出来即可，头文件只需拷inc32目录。

