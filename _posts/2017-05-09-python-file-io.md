---
layout: post
title: python读写文件
category: python
keywords:
---

### 方法1：调用open/close

1. 调用open()函数，返回一个File对象。
2. 调用File对象的read()/write()方法进行读写文件。
3. 调用File对象的close()方法关闭文件。

```
f = open('/etc/passwd', 'rt')
content = f.read()
f.close()
print(content, end='')
```

File对象的read()和readlines()方法都可以读文件，区别是read()返回一个大字符串，包含整个文件内容，而readlines()返回的是一个字符串列表，每个字符串对应文件的某一行。

### 方法2：使用with语句

```
# read entire file as a single string
with open('/tmp/test', 'rt') as f:
    data = f.read()

# iterate over the lines of file
with open('/tmp/test', 'rt') as f:
    for line in f:
        print(line, end='')

# write chunks of data
with open('/tmp/test', 'wt') as f:
    f.write(text1)
    f.write(text2)

# redirect print statement
with open('/tmp/test', 'wt') as f:
    print(line1, file=f)
    print(line2, file=f)
```

使用with语句，在with控制结束时，会自动关闭文件。

关于open的模式说明如下：

- r为读，w为写，a为追加写，x表示不存在时才能写，四选一，默认为读。
- t代表文本模式，b代表二进制模式，二选一，默认为文本。
