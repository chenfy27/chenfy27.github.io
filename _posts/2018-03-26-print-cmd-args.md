---
layout: post
title: 打印命令行参数
category: 运维
tag:
---

一般大型系统是由许多的小程序构成，而程序之间的相互调用十分常见，为了增强程序的适应能力，往往会提供命令行参数进行传参。有些时候，被调用的程序没能按预期工作，检查调用时的命令行参数可快速判断问题到底出在调用方还是被调用方。

简单的做法是在调用方把完整的调用命令打印出来，当然，也可以在被调用方将各个参数做打印，但这可能要涉及到修改源码重新编译，能否不改代码将命令行参数都打印出来的？答案是肯定的。

在Linux平台下可通过shell脚本做中转。具体来讲，假设被调用的程序名为test，将被调用程序的名称由test重命名为test.exe，然后在相同位置创建脚本test，内容如下：

```
>/tmp/cmd.log
for i in "$@"; do
    echo -n "[$i]" >>/tmp/cmd.log
done
echo >>/tmp/cmd.log

./test.exe "$@"
```

在Windows系统下，通过bat批处理可实现类似的效果，由于Windows按扩展名来区分文件类型，需要修改调用方的代码。例如，被调用程序名为test.exe，调用命令为`test.exe arg1 arg2 arg3`，现添加call.bat脚本作为中转，调用命仅变为`call.bat test.exe arg1 arg2 arg3`。

以下是call.bat的内容。

```
@echo off
for %%i in (%*) do echo [%%~i]
start %*
```

如果不想修改调用方代码，可考虑写个exe做中转。

