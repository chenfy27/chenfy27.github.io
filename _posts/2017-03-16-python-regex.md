---
layout: post
title: python正则匹配
category: python
keywords:
---

Python提供了re库用于实现字符串的正则匹配。

### 基本使用

常用编程接口：

- compile(pattern, flags=0)
- match(pattern, string, flags=0)
- search(pattern, string, flags=0)
- findall(pattern, string, flags=0)
- finditer(pattern, string, falgs=0)
- split(pattern, string, maxsplit=0, flags=0)
- sub(pattern, repl, string, count=0, flags=0)
- subn(pattern, repl, string, count=0, flags=0)

match,search函数返回的是match对象，而findall/finditer返回的是match对象列表(迭代器)。

常用的flag取值：

- I(IGNORECASE)
- M(MULTILINE)
- S(DOTALL)

在模式中可用小括号给匹配内容加标记进行分组，通过match对象的group方法进行提取。

match对象：

- group(num=0): 匹配正则表达式的第num个分组，默认0为整个匹配串。
- groups(): 返回一个包含所有分组的字符串元组。

### 应用举例

例1：检查输入是否为有效的变量标识符。

```
import sys, re
pat = re.compile("^[a-zA-Z_][a-zA-Z0-9_]*$");
for line in sys.stdin:
    if re.search(pat, line):
        print('ok')
    else:
        print('nok')
```

例2：[从Markdown文件中提取图片链接](https://www.shiyanlou.com/challenges/2817) 

```
import re, sys
pat = re.compile(r'!\[.*\]\((.*)\)')
with open(sys.argv[1], 'rt') as f:
    for i in re.findall(pat, f.read()):
        print(i)
```

例3：找出文件中以元音字符开头的单词，去重后按字典序输出。

```
import re
pat = re.compile(r'[aeiou][a-z]*', re.I)
words = set()
with open('data') as f:
    for word in re.findall(pat, f.read()):
        words.add(word)
for word in sorted(words):
    print(word)
```

例4：分组提取ip地址

```
import re
pat = re.compile(r'(\d+).(\d+).(\d+).(\d+)')
with open('data') as f:
    txt = f.read().splitlines()
    for ip in txt:
        mat = re.search(pat, ip)
        print("====", ip, "====")
        print("group():", mat.group())
        print("group(1):", mat.group(1))
        print("group(2):", mat.group(2))
        print("groups():", mat.groups())
```

运行结果：

```
==== 118.26.66.138 ====
group(): 118.26.66.138
group(1): 118
group(2): 26
groups(): ('118', '26', '66', '138')
==== 172.16.1.186 ====
group(): 172.16.1.186
group(1): 172
group(2): 16
groups(): ('172', '16', '1', '186')
```

### 常用技巧

- 用括号分组，在调group()时传入整数1或2便可得到文本的不同部分，传0或不传返回整个匹配的文本。而调用groups()则一次返回所有分组。
- 用管道符来匹配多个分组，作为正则表达式的一部分。
- 问号匹配0次或1次，星号匹配0次或多次，加号匹配1次或多次，花括号匹配特定次数。
- python的正则匹配默认是贪心的，尽可能匹配最长的字符串；如果在花括号后跟一个问号，则表示非贪心匹配，即匹配尽可能短的字符串。
- 点匹配除换行符外的所有字符。
- \d, \w, \s分别匹配数字，单词和空格，而\D, \W, \S分别匹配除数字，单词与空格外的所有字符。
- 向re.compile()传入re.I或re.IGNORECASE作为第二个参数，表示不区分大小写匹配。
- 用sub()方法可替换字符串，其原型为：`sub(pattern, repl, string, count=0, flags=0)`。
- 如果只调用一次match/search/sub，那么可以在传regex对象的地方可以直接给模式串；而如果要多次循环调用，建议先调compile得到regex对象再传，提高效率。

