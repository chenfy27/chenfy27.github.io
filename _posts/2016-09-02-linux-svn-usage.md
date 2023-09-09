---
layout: post
title: linux下svn基本使用
category: 运维
keywords: linux, svn
---

#### 1. 检出代码

```
svn --username=chenfy co https://192.168.1.100:3003/svn/product/trunk ~/trunk
```

#### 2. 增删文件

```
svn add test.c
svn delete test.c
```

#### 3. 查看变更内容

```
svn st
svn diff
svn diff test.c
```

#### 4. 放弃本地修改

```
svn revert test.c
svn revert *
```

#### 5. 更新本地文件

```
svn up
svn up -r1234
```

#### 6. 提交变更

需要先配置SVN_EDITOR环境变量，可以在.bashrc中增加：`export SVN_EDITOR=vim`。

```
svn ci
svn ci test.c
```

#### 7. 查看变更记录

```
svn log -v
svn log test.c
svn diff -r1233:1234 -x --ignore-eol-style
```

#### 8. 服务器地址变更

```
svn switch --relocate old_url new_url
```

#### 9. 提交记录与变更历史

```
svn log -v | less
svn diff -r$(($1-1)):$1 -x --ignore-eol-style | less
```

#### 10. 更多

```
svn help
```

