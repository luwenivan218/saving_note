> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1497822078)

> 一、什么是主从复制：主从复制，是用来建立一个和主数据库完全一样的数据库环境，称为从数据库；主数据库一般是准实时的业务数据库。二、主从复制的作用（好处，或者说为什么要做主从）重点

转

 2023-03-28  阅读 (160)  点赞 (0)

原文地址：https://cloud.tencent.com/developer/article/1185462

主从复制，是用来建立一个和主数据库完全一样的数据库环境，称为从数据库；主数据库一般是准实时的业务数据库。

1、做数据的热备，作为后备数据库，主数据库服务器故障后，可切换到从数据库继续工作，避免数据丢失。

2、架构的扩展。业务量越来越大，I/O 访问频率过高，单机无法满足，此时做多库的存储，降低磁盘 I/O 访问的频率，提高单个机器的 I/O 性能。

3、读写分离，使数据库能支撑更大的并发。在报表中尤其重要。由于部分报表 sql 语句非常的慢，导致锁表，影响前台服务。如果前台使用 master，报表使用 slave，那么报表 sql 将不会造成前台锁，保证了前台速度。

1. 数据库有个 bin-log 二进制文件，记录了所有 sql 语句。

2. 我们的目标就是把主数据库的 bin-log 文件的 sql 语句复制过来。

3. 让其在从数据的 relay-log 重做日志文件中再执行一次这些 sql 语句即可。

**4. 下面的主从配置就是围绕这个原理配置**

5. 具体需要三个线程来操作：

binlog 输出线程。每当有从库连接到主库的时候，主库都会创建一个线程然后发送 binlog 内容到从库。

在从库里，当复制开始的时候，从库就会创建两个线程进行处理：

从库 I/O 线程。当 START SLAVE 语句在从库开始执行之后，从库创建一个 I/O 线程，该线程连接到主库并请求主库发送 binlog 里面的更新记录到从库上。从库 I/O 线程读取主库的 binlog 输出线程发送的更新并拷贝这些更新到本地文件，其中包括 relay log 文件。

从库的 SQL 线程。从库创建一个 SQL 线程，这个线程读取从库 I/O 线程写到 relay log 的更新事件并执行。

可以知道，对于每一个主从复制的连接，都有三个线程。拥有多个从库的主库为每一个连接到主库的从库创建一个 binlog 输出线程，每一个从库都有它自己的 I/O 线程和 SQL 线程。

主从复制如图：

![](http://image.skjava.com/article/series/mysql/202303282258578371.png)

原理图

还不懂？没关系，这图也一样：

![](http://image.skjava.com/article/series/mysql/202303282258585282.png)

*   步骤一：主库 db 的更新事件 (update、insert、delete) 被写到 binlog
*   步骤二：从库发起连接，连接到主库
*   步骤三：此时主库创建一个 binlog dump thread，把 binlog 的内容发送到从库
*   步骤四：从库启动之后，创建一个 I/O 线程，读取主库传过来的 binlog 内容并写入到 relay log
*   步骤五：还会创建一个 SQL 线程，从 relay log 里面读取内容，从 Exec_Master_Log_Pos 位置开始执行读取到的更新事件，将更新内容写入到 slave 的 db

一、Master 主服务器上的配置（103.251.237.42）

1. 编辑 my.cnf （命令查找文件位置：find / -name my.cnf）

![](http://image.skjava.com/article/series/mysql/202303282258597963.png)

在 [mysqld] 中注释掉 bind-address = 127.0.0.1 不然 mysql 无法远程

![](http://image.skjava.com/article/series/mysql/202303282259012074.png)

![](http://image.skjava.com/article/series/mysql/202303282259023615.png)

server-id = 1 中 1 是可以自己定义的，但是需要保持它的唯一性，是服务器的唯一标识

`1.log_bin 启动MySQL二进制日志`

`2.binlog_do_db 指定记录二进制日志的数据库`

`3.binlog_ignore_db 指定不记录二进制日志的数据库。`

![](http://image.skjava.com/article/series/mysql/202303282259035076.png)

注释掉 binlog_do_db 和 binlog_ignore_db ，则表示备份全部数据库

做完这些后，重启下数据库

2. 登陆主服务器 mysql 创建从服务器用到的账户和权限;

![](http://image.skjava.com/article/series/mysql/202303282259048467.png)

@之后 IP 可访问主服务器，这里值定从服务器 IP

新建密码为 masterbackup 的 masterbackup 用户，并赋予 replication slave 权限

![](http://image.skjava.com/article/series/mysql/202303282259061378.png)

可以看到用户 masterbackup 已经添加

3. 查看主数据库的状态

![](http://image.skjava.com/article/series/mysql/202303282259073419.png)

记录 mysql-bin.000007 以及 276，编写以下命令待用；

change master to master_host='103.251.237.42',master_port=3306,master_user='masterbackup',master_password='masterbackup',master_log_file='mysql-bin.000007',master_log_pos=276;

二、Slave 从服务器配置上的配置（103.251.237.45）

1. 编辑 my.cnf（命令查找文件位置：find / -name my.cnf）

![](http://image.skjava.com/article/series/mysql/2023032822590855610.png)

在 [mysqld] 中

![](http://image.skjava.com/article/series/mysql/2023032822590990511.png)

relay-log = slave-relay-bin

relay-log-index = slave-relay-bin.index

暂时不清楚这是做什么的。加入这两条。

重启 mysql 服务

![](http://image.skjava.com/article/series/mysql/2023032822591109412.png)

登陆 mysql，停止同步命令

![](http://image.skjava.com/article/series/mysql/2023032822591257413.png)

执行用上面准备的命令； 登录 Slave 从服务器，连接 Master 主服务器：

![](http://image.skjava.com/article/series/mysql/2023032822591390414.png)

重新启动数据同步；

![](http://image.skjava.com/article/series/mysql/2023032822591518915.png)

查看 Slave 信息；如图两句都为 yes，则状态正常

三、从主从服务器测试结果

![](http://image.skjava.com/article/series/mysql/2023032822591649316.png)

在主服务器创建一个数据库

![](http://image.skjava.com/article/series/mysql/2023032822591736817.png)

在从服务器上查看刚才创建的数据库

可以查到，主从服务器配置完成。（技术文）当然，还有主主复制，如果有感兴趣的朋友可以留言。

**其实主从复制也存在一些问题：**

1.  负载均衡，由于复制的时间差，不能保证同步读，而且写仍然单点，没法多点写，我对这个理解就是半吊子的读写均衡。
    
2.  容灾，基本都是有损容灾，因为数据不同步，谁用谁知道，半吊子的容灾。
    

> 可能只是提供一种成本较低的数据备份方案加不完美的容灾和负载均衡吧，这种方案注定是一种过渡方案，个人认为必须更新了。当然，在不是体量巨大的情况下，还是不失为一个优化的解决办法。

**1、主从的好处是？**

**2、主从的原理是？**

**3、从数据库的读的延迟问题了解吗？如何解决？**

**4、做主从后主服务器挂了怎么办？**

大家可以思考后留言哈。积极思考，东西才是你的。这才是最后的干货。