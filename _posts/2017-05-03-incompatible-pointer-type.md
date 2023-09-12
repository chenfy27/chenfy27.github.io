---
layout: post
title: 不匹配的指针类型
category: 排障
keywords:
---

往往越是临近版本发布，就越可能测出一些奇怪的问题，下面的案例便是五一版本发布前夕发现的问题，比较有意思，结果出人意料。

### 背景与问题现象

近期准备将产品从centos5迁移到centos7上，由于操作系统版本的差异，迁移时存在一些问题，其中的大多数都能找到原因并予以修复。在即将完工的时候测试又提了个bug，由我来排查，这个问题是关于HacCheck进程的。

HacCheck是一个后台运行的保姆进程，最初它的功能是定期检测核心进程是否存在，如不在则启动之。后来由于各种定制需求，在里面加了诸如磁盘容量告警、工单反馈等定期执行的作务。因它是监控其他核心进程的进程，为避免自己因段错误等原因异常挂掉，采用父子进程方式，即父进程起子进程，由子进程实际做检测等各项工作，父进程则监视子进程运行状态，发现子进程异常退出则再起子进程工作。

现在测试发现的问题是当有核心进程挂掉时，HacCheck既没有发出警告，也没有启动相关进程，根本就没在工作。

### 问题排查

我在后台用`ps -ef | grep HacCheck`命令查了下活动进程，发现HacCheck只有一条记录，按理父子进程方式应该会有两条，说明父与子只有一个在运行。通过`tail -f HacCheck.log`查看输出日志，果然父进程在不断地起子进程，看样子是子进程起来后会因错立马退出了。

为验证猜想，我把父子进程工作方式改成单进程模式，运行发现确实有段错误，接下来便要定位下段错误出现的位置。通过不断地加断点调试信息和运行，最终发现在读取HacCheck.ini文件时程序崩溃了。INI的读写用的是自己封装的C++接口，底层涉及到自己实现的字符串类、哈希表和基于哈希表的Map，比较复杂。我仔细过了下接口源码，没发现什么问题，只有一处不太确定，就是有两个函数返回值类型为CString类，而实际返回的是局部char数组的指针，细想觉得这没有问题，因为返回时会调CString的构造函数将char数组拷贝出来。我试着将char数组声明成static，并在函数开头将char数组清零，运行后问题仍然存在，说明确实不是这里的问题。

难道接口底层实现真有问题？我觉得可能性很小，毕竟这个库已经用了近10年，且被多个模块调用，历史上没出过任何问题。我通过ulimit命令设置使其产生core文件，并用gdb查看，显示程序崩溃时确实是在底层的接口函数中，排查顿时陷入僵局。

```
(gdb) bt 
#0 0x000000000040c77f in CMap<int, int, char const*, char const*>::GetAssocAt(int, unsigned int&) const () 
#1 0x000000000040c522 in CMap<int, int, char const*, char const*>::Lookup(int, char const*&) const () 
#2 0x000000000040b35c in CIniFile::LookupSection(char const*) () 
#3 0x000000000040b163 in CIniFile::LookupKey(char const*, char const*) () 
#4 0x000000000040b908 in CIniFile::GetKeyStringValue(char const*, char const*) () 
#5 0x000000000040bcd3 in CIniFile::GetKeyIntValue(char const*, char const*) () 
#6 0x0000000000405ba1 in HacCheckChild () at HacCheck.cpp:1101 
#7 0x00000000004064b9 in main (argc=1, argv=0x7fffd05daba8) at HacCheck.cpp:1217 
```

一天后，我忽然想起在《C专家编程》里看到过一句话，大意是说程序运行时可能在某个地方有越界读或写，本应立即段错误退出，但实际却没有，而是继续运行，到另一个看上去跟发生错误的点毫无关系的地方崩掉了！这种问题排查起来非常困难。我现在面对的问题会不会是这种情况呢？

我从出现段错误的代码行试着往前找，在大约前200行的位置（位于同一个大函数里）发现居然有完全相同的调用，而此处没有发生段错误。我想，很有可能是执行了中间的代码后引发的问题，如果真是上述这种情况，那么问题就出在这200多行代码里。

我继续用断点打印的思想来缩小范围，做法是每隔几行就调一下读INI的函数，看程序会不会崩。最终定位到一行代码，是个函数调用，做工单反馈的，深入调试发现在连接数据库的地方会有问题，换句话说，在这个函数内什么都不干，光连接下数据库然后关闭连接返回，调用就会段错误。

连接数据库用的是libhac_sqldual.so库，这又是个自己封装的底层库，同样用了很多年。经过上面的调试和分析，此时我已经确定问题出在这个库上，然而一番走查并没有找到问题所在。将我引入正道的是一次意外的编译过程，在make过程中输出了4条告警信息，两处是将指针转换成整数，另两处是不兼容的指针类型转换。下面是有告警的代码：

```
typedef struct {
    char host[64];
    char user[64];
    char passwd[64];
    char dbname[64];
    unsigned int port;
    int actived;
    int errorno;
    int dbtype;
    char errormsg[256];
    MYSQL *connection;
}db_conn;

db_conn conn;
...
int nRet = mysql_real_connect(conn->connection, conn->host, conn->user,
    conn->passwd, conn->dbname, conn->port, GetDBDomainSocket(), 2);
if (!nRet) {
    ErrorHandle();
}
...
mysql_set_character_set(&conn->connection, "gbk");
...
```

我上mysql官网查了下这两个函数的原型，如下所示：

```
MYSQL* mysql_real_connect(MYSQL *mysql, const char *host, const char *user,
    const char *passwd, const char *db, unsigned int port,
    const char *unix_socket, unsigned long client_flag);
/* return MYSQL* connection handler if connection was successful, NULL otherwise */

int mysql_set_character_set(MYSQL *mysql, const char *csname);
/* return 0 for success, nozero if an error occurred */
```

对比发现调用mysql_real_connect时将返回值从MYSQL\*强转成int，但关系不大；而调用mysql_set_character_set时形参要求传MYSQL\*，而实参给的是MYSQL\*\*，这才是致命的。

后续跟进发现，mysql_set_character_set调用是最近加的，主要用于解决迁移到centos7上后会话查询界面工单变更类型显示乱码问题，这也就解释了为什么以前没有此问题。

### 总结

- 编译告警不可忽视。
- 对于最近发现的问题，如果在老版本上测试正常，建议先走查下最近提交的代码。
- 真正导致段错误的代码有可能不在发生段错误的地方，而是在它前面某处。
