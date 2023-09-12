---
layout: post
title: 输入输出错误
category: 排障
tag:
---

我们知道，数据无论是在内存还是磁盘中，底层都是01串，与编码格式无关，但你是否碰到过因为文件名编码格式不对而创建或打开文件失败的情况？这是我最近遇到的一件怪事。

在这个需求中，我们使用了阿里云的OSS文件系统，将其挂载到本地作为大容量存储使用。问题的表现很简单，将终端和LANG变量设成UTF-8时，用touch命令创建一个名称含中文的文件，一切正常，但如果采用GBK编码，则报输入输出错误(Input/output error)。

由于touch命令无法获取更详细的错误信息，我决定通过C代码来做同样的操作，因为函数会返回错误码，由此可从返回值知道进一步信息。

```
#include <stdio.h>
#include <errno.h>
int main() {
    char filename[] = {0xd6, 0xd0, 0xce, 0xc4, 0x00};
    FILE *fp = fopen(filename, "w");
    if (fp == NULL)
        printf("open failed: ret=%d, msg=[%s]\n", errno, strerror(errno));
    else
        fclose(fp);
    return 0;
}
```

然而，运行后得到的结果仍然是输入输出错误，我把调用由fopen换成open，结果是一样的，errno为5，但它居然与manual中所列出的任何一个错误码都没匹配上！同样的程序放在本地磁盘上运行，一切正常，只有在OSS挂载的区执行才会报错。这一度让我觉得是挂载硬件的问题，它不支持GBK编码吗？

我对硬件一窍不通，甚至连用什么命令查看硬件信息都不知道，根据报错内容在网上搜了一上午几乎一无所获，而在阿里云OSS存储官方介绍中也没查到编码限制。

如果真是发生了很底层的错误，哪里会有相关记录呢？我想到了syslog，于是我开启监视syslog，并再次运行测试程序，果然有输出。

```
[root@localhost ~]# tail -f /var/log/messages
Jan 19 15:41:34 localhost s3fs[11686]: s3fs.cpp:list_bucket(2452): ListBucketRequest returns with error.
Jan 19 15:41:34 localhost s3fs[11686]: s3fs.cpp:s3fs_readdir(2382): list_bucket returns error(-5).
```

内容显得很友好，给出了源文件名和代码行。由于ossfs是开源的，我很快在github上找到项目主页并下载源码进行查看，发现它竟然还会组xml格式的消息往外发，而得到的错误正是对方返回的。由于OSS挂载是其他同事调研和实现的，我全程没参与，对其中的原理并不清楚，令我费解的是本地创建文件为何要远端来确认？

检测端校验逻辑无法得知，返回的-5也不知道是什么意思，但从源码中发现有往外做HTTP交互的过程，我查了下本地连接情况，还真发现有连接存在。

```
[root@localhost log]# netstat -naopt | grep ossfs
tcp        0      0 10.10.3.3:46618         106.14.230.37:80        ESTABLISHED 11686/ossfs          off (0.00/0/0)
tcp        1      0 10.10.3.3:57036         106.14.230.37:80        CLOSE_WAIT  11686/ossfs          off (0.00/0/0)
tcp        1      0 10.10.3.3:59340         106.14.230.37:80        CLOSE_WAIT  11686/ossfs          off (0.00/0/0)
tcp        0      0 10.10.3.3:46616         106.14.230.37:80        ESTABLISHED 11686/ossfs          off (0.00/0/0)
```

这个106.14.230.37是个公网地址，看来是个服务器，我打开浏览器，输入http://106.14.230.37:80想看下这是个什么网站，却发现根本打不开。

好在HTTP是明文传输的，可以尝试抓包，查看具体的报文内容。

```
[root@localhost ~]# tcpdump host 106.14.230.37 and port 80 -s0 -w oss.cap
```

准备妥当后，再次运行测试程序，查看抓包结果，这次终于找到了错误码对应的说明：编码必须是UTF-8格式。

![]({{"/pic/osscap.jpg" | prepend: site.baseurl}})


