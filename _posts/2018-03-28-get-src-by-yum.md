---
layout: post
title: yum获取源码
category: 平台
tag:
---

在redhat/centos系列操作系统上，经常使用yum来安装各种软件和工具，有时我们可能对软件的具体代码实现感兴趣，或者标准实现不能满足要求，希望能对源码加以修改达到自己的目的，这些场景下都需要先获取到软件的源码。

其实通过yum是可以取到软件源码的，但需要先安装些组件。

```
yum install -y epel-release
yum install -y yum-utils
```

下面以shellinabox为例，说明下载源码的步骤，其他软件类似。

```
yumdownloader --source --destdir=/tmp shellinabox
rpm -ivh shellinabox-2.20-5.el7.src.rpm
```

在用户主目录会生成rpmbuild目录，shellinabox的源码便保存在该目录下。

