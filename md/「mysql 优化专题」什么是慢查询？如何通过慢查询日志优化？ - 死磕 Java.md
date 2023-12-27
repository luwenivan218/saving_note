> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1323235924)

> 在小伙伴们开发的项目中，对于 MySQL 排查问题找出性能瓶颈来说，最容易发现并解决的问题就是 MYSQL 的慢查询以及没有用索引的查询。日志就跟人们写的日记一样，记录着过往的事情。但

转

 2023-03-28  阅读 (224)  点赞 (0)

原文地址：https://cloud.tencent.com/developer/article/1185462

> 在小伙伴们开发的项目中，对于 MySQL 排查问题找出性能瓶颈来说，最容易发现并解决的问题就是 MYSQL 的慢查询以及没有用索引的查询。

日志就跟人们写的日记一样，记录着过往的事情。但是人的日记是主观的（记自己想记的内容），而数据库的日志是客观的，根据记录内容分为以下好几种日志：

a、错误日志：记录启动、运行或停止 mysqld 时出现的问题。

b、通用日志：记录建立的客户端连接和执行的语句。

c、更新日志：记录更改数据的语句。该日志在 MySQL 5.1 中已不再使用。

d、二进制日志：记录所有更改数据的语句。还用于复制。

e、慢查询日志：记录所有执行时间超过 long_query_time 秒的所有查询或不使用索引的查询。

f、Innodb 日志：innodb redo log

缺省情况下，所有日志创建于 mysqld 数据目录中。

可以通过刷新日志，来强制 mysqld 来关闭和重新打开日志文件（或者在某些情况下切换到一个新的日志）。

当你执行一个 FLUSH LOGS 语句或执行 mysqladmin flush-logs 或 mysqladmin refresh 时，则日志被老化。

对于存在 MySQL 复制的情形下，从复制服务器将维护更多日志文件，被称为接替日志。

这次我们介绍的就是慢查询日志。何谓慢查询日志？MySQL 会记录下查询超过指定时间的语句，我们将超过指定时间的 SQL 语句查询称为慢查询，都记在慢查询日志里，我们开启后可以查看究竟是哪些语句在慢查询

![](http://image.skjava.com/article/series/mysql/202303282258513471.png)

mysql>show variables like “%slow%”; 查看慢查询配置，没有则在 my.cnf 中添加，如下

![](http://image.skjava.com/article/series/mysql/202303282258522602.png)

分析日志，可用 mysql 提供的 mysqldumpslow，使用很简单，参数可–help 查看

![](http://image.skjava.com/article/series/mysql/202303282258533693.png)

![](http://image.skjava.com/article/series/mysql/202303282258545964.png)

queries total: 总查询次数 unique: 去重后的 sql 数量

sorted by : 输出报表的内容排序

最重大的慢 sql 统计信息, 包括 平均执行时间, 等待锁时间, 结果行的总数, 扫描的行总数.

Count, sql 的执行次数及占总的 slow log 数量的百分比.

Time, 执行时间, 包括总时间, 平均时间, 最小, 最大时间, 时间占到总慢 sql 时间的百分比.

95% of Time, 去除最快和最慢的 sql, 覆盖率占 95% 的 sql 的执行时间.

Lock Time, 等待锁的时间.

95% of Lock , 95% 的慢 sql 等待锁时间.

Rows sent, 结果行统计数量, 包括平均, 最小, 最大数量.

Rows examined, 扫描的行数量.

Database, 属于哪个数据库

Users, 哪个用户, IP, 占到所有用户执行的 sql 百分比

Query abstract, 抽象后的 sql 语句

Query sample, sql 语句