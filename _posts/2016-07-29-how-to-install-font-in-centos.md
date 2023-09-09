---
layout: post
title: linux下安装字体
category: 运维
keywords: font
---

Windows下安装字体只需要将字体文件拷贝到Fonts目录下就行了，Linux下相对麻烦些。  
这里以CentOS下安装Consolas字体为例说明，假定字体文件已准备好，在/tmp/Consolas.ttf。

```
[root@dev ~]# mkdir /usr/share/fonts/consolas
[root@dev ~]# cd /usr/share/fonts/consolas
[root@dev ~]# cp /tmp/Consolas.ttf .
[root@dev ~]# mkfontscale
[root@dev ~]# mkfontdir
[root@dev ~]# fc-cache
```

