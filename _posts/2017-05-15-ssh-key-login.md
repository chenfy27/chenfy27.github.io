---
layout: post
title: ssh密钥登录
category: 运维
keywords:
---

对于经常访问的远程linux主机，每次输密码登录比较麻烦，可以改成ssh密钥方式，省去输密码的过程。

```
$ ssh-keygen -t rsa
$ ssh-copy-id -i ~/.ssh/id_rsa.pub chenfy@10.10.1.76
$ ssh chenfy@10.10.1.76
```

如果没有ssh-copy-id命令，可以通过其他方法实现，只需把id_rsa.pub中的内容追加到远程主机上用户主目录下的.ssh/authorized_keys文件即可。

```
$ cat ~/.ssh/id_rsa.pub | ssh chenfy@10.10.1.76 "cat - >>~/.ssh/authorized_keys"
```
