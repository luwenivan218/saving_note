> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1378627079)

> 上一篇我们讲了关于视图应用与优化，本篇我们讲解内存优化。本篇短小精悍，通俗易懂。注意：以下都是在 MySQL 目录下的 my.ini 文件中改写。一、InnoDB 内存优化 InnoDB 用

转

 2023-03-28  阅读 (189)  点赞 (0)

原文地址：https://cloud.tencent.com/developer/article/1185462

> 上一篇我们讲了关于视图应用与优化，本篇我们讲解内存优化。本篇短小精悍，通俗易懂。

![](http://image.skjava.com/article/series/mysql/202303282258482961.png)

注意：以下都是在 MySQL 目录下的 my.ini 文件中改写。

一、InnoDB 内存优化

InnoDB 用一块内存区域做 I/O 缓存池，该缓存池不仅用来缓存 InnoDB 的索引块，而且也用来缓存 InnoDB 的数据块。

1、innodb_log_buffer_size

决定了 InnoDB 重做日志缓存的大小，可以避免 InnoDB 在事务提交前就执行不必要的日志写入磁盘操作。

2、设置 Innodb_buffer_pool_size

改变量决定了 InnoDB 存储引擎表数据和索引数据的最大缓存区大小。

![](http://image.skjava.com/article/series/mysql/202303282258489592.png)

二、MyISAM 内存优化

MyISAM 存储引擎使用 key_buffer 缓存索引模块，加速索引的读写速度。对于 MyISAM 表的数据块，mysql 没有特别的缓存机制，完全依赖于操作系统的 IO 缓存。

1、read_rnd_buffer_size

对于需要做排序的 MyISAM 表查询，如带有 order by 子句的 sql，适当增加 read_rnd_buffer_size 的值，可以改善此类的 sql 性能。但需要注意的是 read_rnd_buffer_size 独占的，如果默认设置值太大，就会造成内存浪费。

2、key_buffer_size 设置

key_buffer_size 决定 MyISAM 索引块缓存分区的大小。直接影响到 MyISAM 表的存取效率。对于一般 MyISAM 数据库，建议 1/4 可用内存分配给 key_buffer_size:

key_buffer_size=2G

3、read_buffer_size

如果需要经常顺序扫描 MyISAM 表，可以通过增大 read_buffer_size 的值来改善性能。但需要注意的是 read_buffer_size 是每个 seesion 独占的，如果默认值设置太大，就会造成内存浪费。

三、调整 MySQL 参数并发相关的参数

1、调整 max_connections

提高并发连接

2、调整 thread_cache_size

加快连接数据库的速度，MySQL 会缓存一定数量的客户服务线程以备重用，通过参数 thread_cache_size 可控制 mysql 缓存客户端线程的数量。

3、innodb_lock_wait_timeout

控制 InnoDB 事务等待行锁的时间，对于快速处理的 SQL 语句，可以将行锁等待超时时间调大，以避免发生大的回滚操作。（技术文）