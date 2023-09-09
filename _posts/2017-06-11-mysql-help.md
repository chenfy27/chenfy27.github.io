---
layout: post
title: 查看mysql自带帮助
category: 数据库
keywords:
---

在使用mysql的过程中，可能会经常遇到些细节问题，比如：

- 某个语句的语法记不清了，需要快速查找。
- 某个字段类型的取值范围是什么？
- 当前版本是否支持某些函数？
- 当前版本是否支持某些功能？

对于类似以上列出的问题，都可以通过mysql自带的帮助文档找到答案，下面通过些例子加以说明。

查看某个对象说明的语法很简单，格式为\"? object\"。

首先，如果不熟悉帮助能提供什么，用\"? contents\"可以显示主目录，它包含了所有内容。

```
mysql> ? contents
You asked for help about help category: "Contents"
For more information, type 'help <item>', where <item> is one of the following
categories:
   Account Management
   Administration
   Compound Statements
   Data Definition
   Data Manipulation
   Data Types
   Functions
   Functions and Modifiers for Use with GROUP BY
   Geographic Features
   Help Metadata
   Language Structure
   Plugins
   Procedures
   Storage Engines
   Table Maintenance
   Transactions
   User-Defined Functions
   Utility
```

例如，想查一下用TEXT类型存储字符串支持的最大长度是多少，应该在Data Types节里。

```
mysql> ? data types
You asked for help about help category: "Data Types"
For more information, type 'help <item>', where <item> is one of the following
topics:
   AUTO_INCREMENT
   BIGINT
   BINARY
   BIT
   BLOB
   BLOB DATA TYPE
   BOOLEAN
   CHAR
   CHAR BYTE
   DATE
   DATETIME
   DEC
   DECIMAL
   DOUBLE
   DOUBLE PRECISION
   ENUM
   FLOAT
   INT
   INTEGER
   LONGBLOB
   LONGTEXT
   MEDIUMBLOB
   MEDIUMINT
   MEDIUMTEXT
   SET DATA TYPE
   SMALLINT
   TEXT
   TIME
   TIMESTAMP
   TINYBLOB
   TINYINT
   TINYTEXT
   VARBINARY
   VARCHAR
   YEAR DATA TYPE
```

可以看到有TEXT类型，继续\"? text\"即可查到答案。

```
mysql> ? text
Name: 'TEXT'
Description:
TEXT[(M)] [CHARACTER SET charset_name] [COLLATE collation_name]

A TEXT column with a maximum length of 65,535 (216 − 1) characters.
The effective maximum length is less if the value contains multibyte
characters. Each TEXT value is stored using a 2-byte length prefix that
indicates the number of bytes in the value.

An optional length M can be given for this type. If this is done, MySQL
creates the column as the smallest TEXT type large enough to hold
values M characters long.

URL: http://dev.mysql.com/doc/refman/5.6/en/string-type-overview.html
```

又如，想查一下mysql支持哪些位运算符，应该在/Functions/Bit Functions下，输入\"? bit functions\"即可。

```
mysql> ? bit functions
You asked for help about help category: "Bit Functions"
For more information, type 'help <item>', where <item> is one of the following
topics:
   &
   <<
   >>
   BIT_COUNT
   ^
   |
   ~
```
