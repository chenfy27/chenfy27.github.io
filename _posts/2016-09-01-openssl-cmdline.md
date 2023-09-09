---
layout: post
title: openssl常用命令行
category: 运维
keywords:
---

#### Base64编解码

```
[root@dev ~]# echo -n "chenfy" | openssl base64
Y2hlbmZ5
[root@dev ~]# echo "Y2hlbmZ5" | openssl base64 -d
chenfy[root@dev ~]# 
```

#### 文件一致性校验

```
[root@dev ~]# openssl md5 /etc/passwd
MD5(/etc/passwd)= d66feb4f3f36a9e2f9cd8aaa6d8c1d30

[root@dev ~]# openssl sha1 /etc/passwd
SHA1(/etc/passwd)= 2201ea5ac6e340439ace6376e26e71bc3d75759e
```

#### 对称算法加解密

```
[root@dev ~]# openssl enc -e -aes-128-cbc -k 1234 </etc/passwd >cipher
[root@dev ~]# openssl enc -d -aes-128-cbc -k 1234 <cipher | diff /etc/passwd -
[root@dev ~]#

[root@dev ~]# openssl enc -e -des-ede3-cbc -a -k 1234 </etc/passwd >cipher
[root@dev ~]# openssl enc -d -des-ede3-cbc -a -k 1234 <cipher | diff /etc/passwd -
[root@dev ~]#

[root@dev ~]# key=31065e2b664a297a18402d073f51685c4c36625d1c7b0e28
[root@dev ~]# iv=15231e086d7e5f39
[root@dev ~]# echo "chenfy" | openssl enc -e -des-ede3-cbc -a -K $key -iv $iv
dSlEi++Uf7E=
[root@dev ~]# echo "dSlEi++Uf7E=" | openssl enc -d -des-ede3-cbc -a -K $key -iv $iv
chenfy

[root@dev ~]# echo "chenfy" | openssl enc -e -aes-128-cbc -k "123456" -a
U2FsdGVkX18C2NhH+QBPWnaNesQ0/K1VLcYpyNIkm0g=
[root@dev ~]# echo "U2FsdGVkX18C2NhH+QBPWnaNesQ0/K1VLcYpyNIkm0g=" | openssl enc -d -aes-128-cbc -k "123456" -a
chenfy
```

