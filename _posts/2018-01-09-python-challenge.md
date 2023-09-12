---
layout: post
title: python challenge
keywords:
category: python
---

### [第0关](http://www.pythonchallenge.com/pc/def/0.html)

计算2的38次方。

```
>>> 2**38
274877906944
>>> pow(2,38)
274877906944
```

### [第1关](http://www.pythonchallenge.com/pc/def/map.html)

实现字母移位编解码。

```
import string, sys
a = string.ascii_lowercase
b = a[2:] + a[:2]
s = sys.stdin.read()
print s.translate(string.maketrans(a, b))
```

### [第2关](http://www.pythonchallenge.com/pc/def/ocr.html)

从一大段字符串中寻找只出现过一次的字符，并按原顺序输出。

```
import urllib, re, sys
def get_file(fname):
    url = 'http://www.pythonchallenge.com/pc/def/' + fname
    return urllib.urlopen(url).read()
dat = get_file('ocr.html')
txt = re.findall(r'<!--(.*?)-->', dat, re.S)[-1]
d = {}
for i in txt:
    d[i] = d.get(i, 0) + 1
for i in txt:
    if d[i] == 1:
        sys.stdout.write(i)
```

### [第3关](http://www.pythonchallenge.com/pc/def/equality.html)

从一大段字符中找出小写字母，要求它的左右分别恰好有三个大写字母，按原顺序输出。

```
import urllib, re, sys
def get_file(fname):
    url = 'http://www.pythonchallenge.com/pc/def/' + fname
    return urllib.urlopen(url).read()
dat = get_file('equality.html')
txt = re.findall(r'<!--(.*?)-->', dat, re.S)[-1]
ans = re.findall(r'[^A-Z][A-Z]{3}([a-z])[A-Z]{3}[^A-Z]', txt, re.S)
print ''.join(ans)
```

### [第4关](http://www.pythonchallenge.com/pc/def/linkedlist.php)

有一组类似链表结构的网页，每个页面给出了下一个页面的地址参数，遍历整个链找到最后的结点。

```
import urllib, re
def get_file(fname):
    url = 'http://www.pythonchallenge.com/pc/def/' + fname
    return urllib.urlopen(url).read()
pre = 'linkedlist.php?nothing='
idx = '12345'
while True:
    dat = get_file(pre + idx)
    print dat
    ans = re.search(r'\d+', dat)
    if ans is None: break
    idx = ans.group(0)
```

### [第5关](http://www.pythonchallenge.com/pc/def/peak.html)

使用pickle模块加载序列化后的内容，然后打印出来。

```
import urllib, pickle
def get_file(fname):
    url = 'http://www.pythonchallenge.com/pc/def/' + fname
    return urllib.urlopen(url).read()
dat = get_file('banner.p')
ans = pickle.loads(dat)
print '\n'.join([''.join([j[0] * j[1] for j in i]) for i in ans])
```

### [第6关](http://www.pythonchallenge.com/pc/def/channel.html)

与第4关类似，但这次结点信息存在于一个zip文件中，需要从头到尾拼接各个文件的注释内容。

```
import urllib, StringIO, zipfile, re
def get_zip(fname):
    url = 'http://www.pythonchallenge.com/pc/def/' + fname
    return zipfile.ZipFile(StringIO.StringIO(urllib.urlopen(url).read()))
z = get_zip('channel.zip')
idx = 'readme'
cmt = ''
while True:
    cmt += z.getinfo(idx + '.txt').comment
    txt = z.read(idx + '.txt')
    ret = re.search(r'\d{2,}', txt)
    if ret is None: break
    idx = ret.group(0)
print cmt
```

### [第7关](http://www.pythonchallenge.com/pc/def/oxygen.html)

在一张图片的中间取一部分像素点，将其灰度值转换成对应的Ascii字符。

```
>>> import urllib, StringIO
>>> from PIL import Image
>>> src = urllib.urlopen('http://www.pythonchallenge.com/pc/def/oxygen.png').read()
>>> img = Image.open(StringIO.StringIO(src))
>>> w, h = img.size
>>> ''.join([chr(img.getpixel((i, h/2))[0]) for i in xrange(0, w, 7)])
'smart guy, you made it. the next level is [105, 110, 116, 101, 103, 114, 105, 116, 121]pe_'
>>> ''.join(map(chr, [105, 110, 116, 101, 103, 114, 105, 116, 121]))
'integrity'
```

### [第8关](http://www.pythonchallenge.com/pc/def/integrity.html)

给出经bzip压缩过的用户名和密码内容，将其还原。

```
>>> import bz2
>>> uid='BZh91AY&SYA\xaf\x82\r\x00\x00\x01\x01\x80\x02\xc0\x02\x00 \x00!\x9ah3M\x07<]\xc9\x14\xe1BA\x06\xbe\x084'
>>> pwd='BZh91AY&SY\x94$|\x0e\x00\x00\x00\x81\x00\x03$ \x00!\x9ah3M\x13<]\xc9\x14\xe1BBP\x91\xf08'
>>> bz2.BZ2Decompressor().decompress(uid)
'huge'
>>> bz2.BZ2Decompressor().decompress(pwd)
'file'
```

### [第9关](http://www.pythonchallenge.com/pc/return/good.html)

给出一个整数列表代表一条线，其中第i点的x,y坐标下标分别为2i, 2i+1，根据坐标点画出该图。

```
>>> from PIL import Image, ImageDraw
>>> l1 = [146,399, ...]
>>> l2 = [156,141, ...]
>>> img = Image.new('1', (500,500), 1)
>>> draw = ImageDraw.Draw(img)
>>> draw.line(zip(l1[0::2], l1[1::2]))
>>> draw.line(zip(l2[0::2], l2[1::2]))
>>> img.save('out.png')
```

### [第10关](http://www.pythonchallenge.com/pc/return/bull.html)

求[look-and-say sequence](https://en.wikipedia.org/wiki/Look-and-say_sequence)中第31项的长度。

```
a = ['1']
for i in xrange(30):
    t = ''
    n = 0
    x = ''
    for i in a[-1]:
        if n == 0:
            n = 1
            x = i
        elif i == x:
            n += 1
        else:
            t += str(n) + x
            n = 1
            x = i
    else:
        t += str(n) + x
    a.append(t)
print len(a[30])
```

### [第11关](http://www.pythonchallenge.com/pc/return/5808.html)

给出一张图像，要求去掉x,y坐标和为奇数的像素点。

```
import urllib
import StringIO
from PIL import Image, ImageDraw
url = 'http://www.pythonchallenge.com/pc/return/cave.jpg'
dat = urllib.urlopen(url).read()
img = Image.open(StringIO.StringIO(dat))
w, h = img.size
for i in xrange(w):
    for j in xrange(h):
        if (i + j) % 2:
            img.putpixel((i,j), 0)
img.save('out.png')
```
