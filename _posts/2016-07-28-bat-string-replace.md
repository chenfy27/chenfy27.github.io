---
layout: post
title: 批处理做文本替换
category: 平台
keywords: bat,string
---
Linux下因grep,sed,awk等工具的存在，实现文本查找与替换很轻松。  
标配的Windows批处理命令非常有限，但用基本的命令也可以手动实现类似sed做法的文本替换功能。  
举个例子，要将data.txt文件中所有的\"abcd\"替换成\"ABCD\"，将结果写入data1.txt中。

```
@echo off
setlocal enabledelayedexpansion

cd .>data1.txt
for /f "delims=" %%a in (data.txt) do (
	set line=%%a
	set line=!line:abcd=ABCD!
	echo !line!>>data1.txt
)
```

