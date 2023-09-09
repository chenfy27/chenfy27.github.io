---
layout: post
title: python读写excel
category: python
keywords:
---

python操作excel需要安装openpyxl库，执行`sudo pip install openpyxl`即可。

### 基本概念

- 一个excel电子表格文档(.xls或.xlsx文件)称为一个工作薄(workbook)。
- 每个工作薄可以包含一个或多个工作表(sheet)。
- 用户当前查看的表，或关闭excel前最后查看的表，称为活动表(active sheet)。
- 每个表包含一些行和列，处于特定行与列的格子称为单元格(cell)。

### 常用方法

- 通过load_workbook()函数加载excel文件，传入文件名，返回一个workbook对象。
- 通过workbook对象的get_sheet_names()方法可取得工作薄中所有表名的列表。
- 通过workbook对象的get_active_sheet()方法可获得活动表对象。
- 通过workbook对象的get_sheet_by_name()方法可获得指定名称的表对象。
- sheet常用属性：title为表名；max_row为行数；max_column为列数。
- 通过sheet.cell(row=x, column=y)可方问坐标为(x,y)的cell对象。
- cell对象常用属性：value为内容，row为行号，column为列号，coordinate为坐标。
- 通过workbook对象的save()方法保存内容到文件。

### 示例

假设有以下数据表格要读取：

<table border="1" cellspacing="0" cellpadding="5">
<tr><th>id</th><th>name</th><th>device</th><th>ip</th></tr>
<tr><td>101</td><td>chenfy1</td><td>linux133</td><td>192.168.1.133</td></tr>
<tr><td>102</td><td>chenfy2</td><td>linux134</td><td>192.168.1.134</td></tr>
</table>
<p></p>

```python
import openpyxl

wb = openpyxl.load_workbook('test.xlsx')
sh = wb.get_sheet_names()
print sh
cur = wb.get_active_sheet()
print cur.title

for i in xrange(1, cur.max_row + 1):
    for j in xrange(1, cur.max_column + 1):
        print cur.cell(row=i, column=j).value,
    print
```

运行结果如下：

```
chenfy@study python $ ./b.py
[u'Sheet1', u'Sheet2', u'Sheet3']
Sheet1
id name device ip
101 chenfy1 linux133 192.168.1.133
102 chenfy2 linux134 192.168.1.134
```
