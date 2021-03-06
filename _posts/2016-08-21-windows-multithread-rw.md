---
layout: post
title: windows下多线程读写文件
category: 排障
keywords: multithread
---

月初，我收到现场实施的技术人员反馈，说应用发布在运维时偶尔会出现IE报AppCrash错误异常退出的现象，错误详细信息中的模块项直指自主开发的SPIHook.dll文件。SPI模块早在我入职前就有，且一直以来功能稳定，除后来发现的解引用空指针问题外，基本没什么变更，突然出现的问题让我有些摸不着头脑。

我打算根据现场的环境与版本信息搭个测试环境，尝试复现这个问题。令人沮丧的是折腾了一个上午，一次都没有复现，而技术人员说同时多开几个浏览器有很大几率崩溃。在没有想到好办法之前，只好远程调试。首先我需要做的是定位程序究竟是执行到哪个地方崩掉的，这个库的源文件只有一个，有30多个函数，我用最土的方法，在每个函数的入口以及所有出口加上唯一的打印信息，并在比较长的函数中间添加多处打印，确保出现问题时需要检查的代码不超过20行。打印内容大致如下：

- 函数入口：Enter function
- 函数每个return前：function exitX
- 长函数中间：function breakpointX

其中function为对应函数的名称，X为序号。

考虑到输出内容可能会比较多，为了减少调试对现场的影响，同时便于筛选调试信息，我注释掉了原来写日志文件的操作，并封装新打印函数，将内容通过OutputDebugString输出，用DebugView工具来查看。技术人员现场替换文件很快复现了问题，我满怀期待地打开日志，却发现了一些奇怪的现象：在相邻的两条打印内容中居然还有其他地方的内容被打印，而且来自同一个进程！缓过神来后，我意识到这是个多线程工作的进程。

同时启动多个IE导致的多进程交叉打印输出已经够头疼了，每个进程还同时开多个线程无疑让排查变得更为复杂。我简单描述了下当前的问题，向高人请教，他提示我打印时把线程号一并输出。经过一番思考，我觉得这是个好办法。在添加线程号后，每条日志内容格式为：序号 时间 进程ID 过滤标识 线程ID 日志内容，截取部分内容如下：

```
[root@dev spihook]# head hac5330.log | cat -A
00000000^I17:52:58.063^I[2740] <HAC> [5516] in DllMain()^I$
00000002^I17:52:58.063^I[2740] <HAC> [5516] DllMain exit^I$
00000004^I17:52:58.063^I[2740] <HAC> [3592] in DllMain()^I$
00000006^I17:52:58.063^I[2740] <HAC> [3592] DllMain exit^I$
00000008^I17:52:58.063^I[2740] <HAC> [2608] in DllMain()^I$
00000010^I17:52:58.063^I[2740] <HAC> [2608] DllMain exit^I$
00000012^I17:52:58.656^I[1888] <HAC> [4832] in DllMain()^I$
00000014^I17:52:58.656^I[1888] <HAC> [4832] DllMain exit^I$
00000016^I17:53:00.184^I[1888] <HAC> [5548] in WSPSocket()^I$
00000018^I17:53:00.184^I[1888] <HAC> [5548] WSPSocket exit^I$
```

反馈的日志文件内容比较多，每个文件中只有一个进程异常退出，我想将日志按进程ID、线程ID及时间依次排序，然后找哪个调用没有相应的返回就应该能看出些端倪，然而用sort命令处理完后，由于日志内容过多，一时间找不到问题点。通过分析，我发现一个重要的事实：自己关心的其实是每个线程最后打印的那条信息。意识到这点非常关键，我写了个AWK程序用于提取关注的内容，由于最后输出带exit的线程出现异常的可能性较低，暂时把它们滤掉。

```
==================== handle.sh ====================
#!/bin/bash
while [ $# -ne 0 ]; do
    echo "********** ["$1"] **********"
    sed 's#\[\(.*\)\] <HAC> \[\(.*\)\] #\1\t\2\t#' $1 |
        sort -t$'\t' -k3,3 -k4,4 -k1n,1 |
        ./find.awk
    shift 1
done
==================== find.awk ====================
#!/bin/awk -f
BEGIN {FS="\t"; tag=""; line=""; msg=""} 
{   id = $3"#"$4 
    if (id != tag) {
        if (tag != "" && msg !~ /exit/) 
            print NR-1, ":", line
        tag = id
    }
    line = $0; msg=$5
}
```

处理结果令人振奋，我看到了想要的信息。

```
[root@dev spihook]# ./handle.sh *.log
********** [1030.log] **********
22105 : 00049447        10:00:29.545    4380    2232    now read spihook.ini
24267 : 00049453        10:00:29.545    4380    6488    now read spihook.ini
25483 : 00049452        10:00:29.545    4380    6936    in GetHacAddr()
********** [5930.log] **********
3789 : 00205614         9:59:34.305     3232    3284    now read spihook.ini
8699 : 00205648         9:59:34.305     3232    4976    now read spihook.ini
********** [hac5330.log] **********
750 : 00055477          17:53:35.128    1888    2632    now read spihook.ini
3823 : 00055476         17:53:35.128    1888    5360    now read spihook.ini
4801 : 00067131         17:53:42.991    1888    5548    in GetProcessNameFromPid()
********** [hac5500.log] **********
8778 : 00059240         17:55:04.704    2876    2920    now read spihook.ini
9720 : 00059246         17:55:04.704    2876    5572    in GetVdhParamFromFile()
11898 : 00059242        17:55:04.704    2876    5724    in GetVdhParamFromFile()
********** [hac5540.log] **********
2128 : 00004037         15:55:40.154    5548    1688    in GetVdhParamFromFile()
2259 : 00004044         15:55:40.154    5548    2408    in GetHacAddr()
2387 : 00004036         15:55:40.154    5548    3156    in GetVdhParamFromFile()
2465 : 00007492         15:55:49.732    5548    6020    in GetProcessNameFromPid()
********** [hac5610.log] **********
10157 : 00080808        17:56:19.506    2404    4700    return from PrintInfo function
11126 : 00072467        17:56:14.732    2404    5472    in GetHacAddr()
12260 : 00072461        17:56:14.732    2404    6072    now read spihook.ini
********** [hac5720.log] **********
5932 : 00090899         17:57:25.322    2308    3884    now read spihook.ini
7236 : 00090895         17:57:25.322    2308    4208    now read spihook.ini
8063 : 00090897         17:57:25.322    2308    5496    in GetHacAddr()
********** [hac5724.log] **********
19693 : 00118277        15:57:23.317    2180    3220    in GetHacAddr()
21521 : 00126924        15:57:28.527    2180    3576    in ProcIdToSessionId()
24377 : 00126925        15:57:28.527    2180    4876    in ProcIdToSessionId()
25970 : 00118276        15:57:23.317    2180    5872    in GetHacAddr()
```

根据时间点和各异常线程的最后一条打印，以下几处有重大嫌疑：

- 读spihook.ini文件；
- GetVdhParamFromFile函数，该函数是从某个文件中读取二进制数据；
- GetHacAddr函数，该函数是从文件中读取一个IP地址；

容易发现，上述三条都有个共同特征，就是读文件，在我印象中以只读方式操作文件是线程安全的，为了验证这个想法，我写了个简单的测试程序，结果令我大跌眼镜，程序竟然崩了，确认没犯错误后我尝试将文件操作这几行代码放入临界区中，再运行多次，一切正常。我赶紧再写了个Linux版程序做同样测试，不需要做任何互斥就能得到正确结果。可见以只读方式操作文件在Linux下是线程安全的，而在Windows下则不是，包括用API中的GetPrivateProfile*函数读INI文件(后续跟进发现上述代码在VC6.0环境下有问题，而在VS2005环境下就正常了)。

问题已经定位，我把三处读文件操作的代码都置入临界区，重新编了个库给现场测试，技术人员反馈说测试了多次，确实没再出过问题了，我总算松了口气。考虑到是给生产环境用，为安全起见，不应该有太多调试信息，我从代码库取了对应版本的代码，并把加锁操作代码合入其中并编译动态库。本以为这事总算完结，可现场测试后反馈说问题又出了，而且很频繁。我仔细比较变更代码，确认锁没加错后，发现唯一的不同就是之前调试版没有输出日志到文件，记得之前做写日志接口时这块我还专门测过，以追加方式写文件在Linux下是线程安全的，难道到了windows下又变成非线程安全了？我把读文件程序改成了写文件版本，测试发现确实如此。

#### 收获

- windows下读写文件是否为非线程安全的跟环境有关，在早期产品VC6.0中不安全；
- 多进程多线程程序调试方法；
- 掌握几门脚本语言会事半工倍；

#### 思考

- 如果某个函数调用了其他函数，在其他函数返回后的代码有不当操作，用上述做法还能查出来吗？
- 检查hac5610.log的处理结果，程序为什么会崩？windows下多线程读不同的文件也不安全吗？

