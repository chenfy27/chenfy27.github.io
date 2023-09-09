---
layout: post
title: 批量创建和删除用户
category: 运维
tag:
---

本文记录解决[批量创建删除用户和组](https://www.shiyanlou.com/challenges/3882)问题过程。

### 背景说明

小楼是一个系统管理员，需要为一个教室中的服务器添加一个老师和若干学生用户，手动添加太麻烦，请为他编写一个bash脚本userctr.sh，实现批量添加和删除用户。老师用户名，学生用户名和学生数量使用参数进行控制。

userctr.sh脚本执行时包括四个参数：

```
bash userctr.sh add|del 教师名 学生名前缀 学生数量
```

### 具体要求

1. 学生数量参数，范围1~10，若超过10或不为正整数，则报错打印parameter error
2. 学生名前缀为字符串，只允许包含小写字母，否则报错打印parameter error
3. 每个用户默认使用zsh，教师用户名默认具备sudo权限
4. 每个用户设置一个随机6位数字密码，在添加命令执行后将用户名和对应的密码输出
5. 如果某个用户名已经存在，则默认不需要创建该用户，输出时密码显示为6个星号

以下是个执行示例：

```
# 添加一个 teacher 用户和 stu1 到 stu6 6个学生用户
$ bash userctr.sh add teacher stu 6
teacher:901231
stu1:271828
stu2:928172
stu3:******
stu4:384712
stu5:098273
stu6:921098

# 删除一个 teacher 用户和 stu1 到 stu6 6个学生用户
$ bash test.sh del teacher stu 6
```

### 分析与解答

首先是输入参数检查，包括以下几部分：

- 参数为4个
- 参数1只能是add或者del
- 参数3只能包含小写字母
- 参数4只能包含数字，且范围是1~10

接下来的问题是如何生成6位随机数字做密码？可以有多种做法，比如：

- 算用户名的md5，取前6位数字：`echo $name | md5sum | tr -dc [0-9] | head -c 6`
- 用$RANDOM生成数字串
- 通过/dev/random等生成随机数

这里可以采用简单的做法，取纳秒中的6位数字：`date +%N | tail -c 6`

以下是完整的代码实现。

```
#!/bin/bash

function ParamError() {
    echo "parameter error"
    exit 1
}

function AddUser() {
    if grep -qw $1 /etc/passwd; then
        echo "$1:******"
    else
        useradd $1 -m -s /usr/bin/zsh
        password=`date +%N | tail -c 6`
        echo $password | passwd $1 --stdin &>/dev/null
        echo "$1:$password"
    fi
}

function DelUser() {
    userdel -r $1 &>/dev/null
}

function UserHandle() {
    if [ $1 = "add" ]; then
        AddUser $2
    else
        DelUser $2
    fi
}

[ $# -ne 4 ] && ParamError
[ $1 != "add" -a $1 != "del" ] && ParamError
if echo $3 | grep -vq [a-z]; then
    ParamError
fi
if echo $4 | grep -vq [0-9]; then
    ParamError
elif [ $4 -lt 1 -o $4 -gt 10 ]; then
    ParamError
fi

UserHandle $1 $2
echo "$2 ALL=(ALL) ALL" >/etc/sudoers.d/$2

for ((i=1; i<=$4; i++)); do
    UserHandle $1 "$3$i"
done
```
