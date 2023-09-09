---
layout: post
title: odbc连接数据库
category: 数据库
keywords:
---

本文介绍在Uinx/Linux环境下通过ODBC连接几种主流数据库的配置方法。

### 准备工作

安装ODBC基本组件：

```
# yum install -y unixODBC unixODBC-devel
```

安装完成后可用`odbcinst -j`命令查看安装配置文件所在的位置，有两个比较常用的配置，一个是ODBC驱动配置，默认在/etc/odbcinst.ini，另一个是系统数据源配置，默认在/etc/odbc.ini。

### 连接MySQL

#### 1. 安装MySQL连接驱动

```
# yum install -y mysql-connector-odbc
```

安装好驱动后，驱动信息会自动追加到驱动配置odbcinst.ini中，像这样：

```
[MySQL]
Description=ODBC for MySQL
Driver=/usr/lib/libmyodbc5.so
Setup=/usr/lib/libodbcmyS.so
Driver64=/usr/lib64/libmyodbc5w.so
Setup64=/usr/lib64/libodbcmyS.so
FileUsage=1
```

#### 2. 配置MySQL数据源(DSN)

数据源可定义在系统DSN中，也可以定义在用户DSN中，视需要而定。以下是一个配置例子。

```
[mysql223]
driver = MySQL
server = 192.1.1.223
port = 3306
user = root
password = 62960909
```

其中：

- driver是MySQL驱动库的名称，要与odbcinst.ini中配的名字一致，另外也可以直接写.so文件的位置，但不推荐这么做。
- 如果配置不指定user和password，那么在连接时必须给定，命令行和API都有相应的选项或参数。

#### 3. 连接测试

isql命令格式：isql DSN [user [password]] [options]

如果配置指定了用户名和密码，连接时指定DSN即可：`isql mysql223`。

在连接时指定用户名和密码：`isql mysql223 root 62960909`。

### 连接SQLServer

#### 1. 安装SQLServer连接驱动

```
# yum install -y freetds freetds-devel
```

安装好后可用`tsql -C`查看编译时的选项信息。

如果驱动安装完后没有将驱动信息更新到odbcinst.ini中，则需要手工配置。

```
[SQLServer]
Description = ODBC for SQLServer
Driver = /usr/lib/libtdsodbc.so
Setup = /usr/lib/libtdsS.so
FileUsage = 1
```

其中.so文件的路径要与实际安装位置相符。

#### 2. 配置数据源

数据源配置文件为freetds.conf，一般在/etc目录下。以下是个示例：

```
[sqlsever224]
host = 192.1.1.224
port = 1433
tds version = 7.0
```

#### 3. 连接测试

```
# tsql -S sqlserver224 -U test -P test123
```

### 连接Oracle

#### 1. 安装驱动

不同版本的Oracle对应驱动版本也不太一样，可到[官网](http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html)下载与数据库版本对应的驱动，至少要安装basic,develop,odbc包，建议安装sqlplus包，便于排障。

```
# rpm -ivh *.rpm
```

#### 2. 配置环境变量

```
export ORACLE_HOME=/usr/lib/oracle/10.2.0.4/client
export LD_LIBRARY_PATH=$ORACLE_HOME/lib
```

注意路径要与实际相符。

#### 3. 测试连通性

```
# sqlplus test/test123@192.1.1.225:1521/orcl
```

如果连不通，检查客户端与Oracle服务器网络是否连通，另外检查服务器上tnsnames.ora文件中使用的主机名还是IP地址，建议用IP，确保客户端能正常访问。连接成功后再往下配置ODBC。

#### 4. 配置ODBC

在odbcinst.ini中添加Oracle驱动。

```
[Oracle]
Description = ODBC for Oracle
Driver = /usr/lib/oracle/10.2.0.4/client/lib/libsqora.so.10.1
Setup = 
```

在odbc.ini中配置数据源。

```
[oracle225]
driver = Oracle
server = 192.1.1.225
port = 1521
servername = orcl255
```

其中servername是tnsnames.ora中配置的SID，而tnsnames.ora的位置由TNS_NAMES环境变量指定。

```
# export TNS_NAMES=/etc
# cat /etc/tnsnames.ora
orcl255 =
    (DESCRIPTION =
        (ADDRESS = (PROTOCOL = TCP) (HOST = 192.1.1.225) (PORT = 1521))
        (CONNECT_DATA = (SERVICE_NAME = ORCL))
    )
```

配置好后可用isql命令测试连接。

```
# isql oracle225 system keyouhac
```

### 连接达梦数据库

达梦数据库是款国产数据库，应用还有待推广。

在Linux环境下通过-i选项进行交互式安装，这里选择自定义安装方式，安装组件选择客户端和ODBC相关驱动，安装完成后即可用客户端工具连接数据库测试。

```
$ sudo ./DMInstall.bin -i
$ /opt/dmdbms/tool/disql
SQL> conn SYSDBA/SYSDBA@192.1.1.190
SQL> select * from dmhr.job;
```

也可以通过unixODBC提供的isql工具连接，但需要配置驱动和数据源。

```
$ cat /etc/odbcinst.ini
[DM7]
Description = ODBC DRIVER FOR DM7
Driver = /opt/dmdbms/bin/libodbc.so

$ cat /etc/odbc.ini
[dm190]
Description = dmdb
Driver = DM7
SERVER = 192.1.1.190
UID = SYSDBA
PWD = SYSDBA
TCP_PORT = 5236

$ isql dm190
$ isql dm190 SYSDBA SYSDBA
```

### ODBC编程

以下是通过ODBC连接mysql数据库并执行sql语句的示例，其他类型数据库类似。

```c
#include <sql.h>
#include <sqltypes.h>
#include <sqlext.h>
#include <stdio.h>
HENV henv;
HDBC hdbc;
HSTMT hsmt;
SQLRETURN rc;
int Success(SQLRETURN ret) {
    return ret == SQL_SUCCESS || ret == SQL_SUCCESS_WITH_INFO;
}
int Failed(SQLRETURN ret) {
    return !Success(ret);
}
void ODBCError(SQLSMALLINT handleType, SQLHANDLE handle) {
    BYTE buf[256], sqlstate[256];
    SQLGetDiagRec(handleType, handle, 1, sqlstate, NULL, buf, sizeof(buf), NULL);
    printf("[%s]%s", sqlstate, buf);
}
int main() {
    if (Failed(SQLAllocHandle(SQL_HANDLE_ENV, NULL, &henv))) {
        ODBCError(SQL_HANDLE_ENV, henv);
        return -1;
    }
    SQLSetEnvAttr(henv, SQL_ATTR_ODBC_VERSION, (SQLPOINTER)SQL_OV_ODBC3, SQL_IS_INTEGER);
    if (Failed(SQLAllocHandle(SQL_HANDLE_DBC, henv, &hdbc))) {
        ODBCError(SQL_HANDLE_DBC, hdbc);
        return -1;
    }
    if (Failed(SQLConnect(hdbc, (SQLCHAR*)"mysql", SQL_NTS, (SQLCHAR*)"root", SQL_NTS,
            (SQLCHAR*)"123456", SQL_NTS))) {
        ODBCError(SQL_HANDLE_DBC, hdbc);
        return -1;
    }

    {
        char user[128], host[128], pass[128];
        long cbuser = 0, cbhost = 0, cbpass = 0;
        if (Failed(SQLAllocStmt(hdbc, &hsmt))) {
            ODBCError(SQL_HANDLE_STMT, hsmt);
            return -1;
        }
        SQLExecDirect(hsmt, (SQLCHAR*)"select user, host, password from mysql.user", SQL_NTS);
        SQLBindCol(hsmt, 1, SQL_C_CHAR, user, sizeof(user), &cbuser);
        SQLBindCol(hsmt, 2, SQL_C_CHAR, host, sizeof(host), &cbhost);
        SQLBindCol(hsmt, 3, SQL_C_CHAR, pass, sizeof(pass), &cbpass);
        while (Success(SQLFetchScroll(hsmt, SQL_FETCH_NEXT, 0)))
            printf("user=[%s], host=[%s], pass=[%s]\n", user, host, pass);
    }

    SQLFreeHandle(SQL_HANDLE_STMT, hsmt);
    SQLDisconnect(hdbc);
    SQLFreeHandle(SQL_HANDLE_DBC, hdbc);
    SQLFreeHandle(SQL_HANDLE_ENV, henv);

    return 0;
}
```
