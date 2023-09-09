---
layout: post
title: 横表与纵表互转
category: 数据库
keywords: db
---

为简单起见，假设横表th如下：

<table border="1" cellspacing="0" style="width:400px; height:100px; text-align:center" cellpadding="5">
<tr><th>姓名</th><th>语文</th><th>数学</th><th>英语</th></tr>
<tr><td>张三</td><td>83</td><td>91</td><td>75</td></tr>
<tr><td>李四</td><td>87</td><td>74</td><td>93</td></tr>
</table>  

<p></p>
相应的纵表tv为：

<table border="1" cellspacing="0" style="width:300px; height:100px; text-align:center" cellpadding="5">
<tr><th>姓名</th><th>课程</th><th>成绩</th></tr>
<tr><td>张三</td><td>语文</td><td>83</td></tr>
<tr><td>张三</td><td>数学</td><td>91</td></tr>
<tr><td>张三</td><td>英语</td><td>75</td></tr>
<tr><td>李四</td><td>语文</td><td>87</td></tr>
<tr><td>李四</td><td>数学</td><td>74</td></tr>
<tr><td>李四</td><td>英语</td><td>93</td></tr>
</table>

### 横表转纵表

```sql
select 姓名, '语文' as 课程, 语文 from th
union select 姓名, '数学' as 课程, 数学 from th
union select 姓名, '英语' as 课程, 英语 from th;
```

### 纵表转横表

```sql
select t.姓名, 语文, 数学, 英语 from (select distinct 姓名 From tv) t
left join (select 姓名, 成绩 as 语文 from tv where 课程 = '语文') t1 on t.姓名 = t1.姓名
left join (select 姓名, 成绩 as 数学 from tv where 课程 = '数学') t2 on t.姓名 = t2.姓名
left join (select 姓名, 成绩 as 英语 from tv where 课程 = '英语') t3 on t.姓名 = t3.姓名;
```

