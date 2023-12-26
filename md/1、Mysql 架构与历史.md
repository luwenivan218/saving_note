> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7304984457526231078)

[](https://link.juejin.cn?target=)**Mysql 逻辑架构**

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1fd191b5178e4656a18699fc12b49774~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=530&h=816&s=30915&e=png&b=ffffff)

[](https://link.juejin.cn?target=)最上层是服务并不是 Mysql 所独有的，大多数基于网络的客户端 / 服务器的工具或者服务都有类似的架构，比如连接处理，授权认证，安全等。

[](https://link.juejin.cn?target=)第二层是 Mysql 比较有意思的部分。大多数 Mysql 的核心服务都在这一层，包括查询解析，分析，优化，缓存以及所有的内置函数，所有跨存储引擎的功能都在这一层实现：存储过程，视图，触发器等。

[](https://link.juejin.cn?target=)第三层包括了存储引擎，存储引擎负责 Mysql 中数据的存储和提取。服务器通过 API 与存储引擎进行通信，这些接口屏蔽了不同存储引擎之间的差异。存储引擎不会去解析 sql，不同存储引擎之间也不会相互通信，只是简单地响应上层服务器的请求。

[](https://link.juejin.cn?target=)**连接管理与安全性**

[](https://link.juejin.cn?target=)每个客户端连接都会在服务器进程中拥有一个线程，这个连接的查询只会在这个单独的线程中执行，该线程只会轮流在某个 CPU 核心或 CPU 中运行。服务器只会负责缓存线程，因此不需要为每一个新建的连接创建或者销毁线程。

[](https://link.juejin.cn?target=)当客户端连接到 Mysql 服务器时，服务器需要对其进行认证，如果使用了安全套接字 SSL 的方式连接，还可以使用 X.509 证书认证。一旦客户端连接成功，服务器会继续验证该客户端 1 是否具有执行特定查询的权限。

[](https://link.juejin.cn?target=)**优化与执行**

[](https://link.juejin.cn?target=)Mysql 会解析查询，并创建内部数据结构，然后对其进行各种优化，包括重写查询，决定表的读取顺序，以及选择合适的索引等。用户可以通过关键字提示优化器，也可以查询优化器过程（explain）。

[](https://link.juejin.cn?target=)优化器并不关心表使用的什么存储引擎，但存储引擎对于优化查询是有影响的。优化器会请求存储引擎提供容量或某个具体操作的开销信息，以及表数据的统计信息等。

[](https://link.juejin.cn?target=)**并发控制**

[](https://link.juejin.cn?target=)无论何时，只要有多个查询需要再同一时刻修改数据，都会产生并发控制的问题，Mysql 会在两个层面进行并发控制：服务器层与存储引擎层。

[](https://link.juejin.cn?target=)**读写锁**

[](https://link.juejin.cn?target=)读锁是共享的，或者说是相互不阻塞的，多个客户在同一时刻可以同时读取同一个资源，而互不干扰。写锁则是排他的，也就是说一个写锁会阻塞其他的写锁和读锁，这是处于安全策略的考虑，只有这样，才能确保在给定的时间内，只有一个用户能执行写入，并防止其他用户读取正在写入的同一资源。

[](https://link.juejin.cn?target=)**锁粒度**

[](https://link.juejin.cn?target=)一种提高共享资源并发性的方式就是让锁对象更有选择性。尽量只锁定需要修改的部分数据，而不是所有的资源。更理想的方式是只对会修改的数据片进行精确的锁定。任何时候，在给定的资源上，锁定的数据量越少，则系统的并发程度越高，只要相互之间不发生冲突即可。

[](https://link.juejin.cn?target=)问题是加锁也需要消耗资源，锁的各种操作，包括获得锁，检查锁是否已经解除，释放锁等，都会增加系统的开销。如果系统花费大量的时候来管理锁，而不是存储数据，那么系统的性能可能会受到影响。

[](https://link.juejin.cn?target=)所谓锁策略就是在锁的开销和数据的安全性之间需求平衡，这种平衡当然也会影响到性能。一般都是在表上加行级锁，并以各种复杂的方式来实现，以便在锁较多的情况下尽可能低提供更好的性能。

[](https://link.juejin.cn?target=)Mysql 提供了多种选择，每种 Mysql 存储引擎都可以实现自己的锁策略和锁粒度。下面介绍两种最重要的锁策略

[](https://link.juejin.cn?target=)**表锁**

[](https://link.juejin.cn?target=)表锁是 Mysql 中最基本的锁策略，并且是开销最小的策略。他会锁整张表，一个用户对表进行写操作前，需要先获取写锁，这会阻塞其他用户对该表的所有读写操作。只有没有写锁时，其他读取用户才能获取读锁，读锁之间是不相互阻塞的。

[](https://link.juejin.cn?target=)在特定场景中，表锁也可能有良好的性能，例如 READ LOCAL 表锁支持某些类型的并发写操作。另外写锁比读锁有更高的优先级，因此一个写锁请求可能会被插入到读锁队列的前面。

[](https://link.juejin.cn?target=)**行级锁**

[](https://link.juejin.cn?target=)行级锁可以最大程度地支持并发处理，同时也带来了最大的锁开销。Mysql 在 InnoDB 中实现了行级锁。行级锁只在存储引擎层实现，而 Mysql 服务器层没有实现，服务器层完全不了解存储引擎中的锁实现。

[](https://link.juejin.cn?target=)**事务**

[](https://link.juejin.cn?target=)事务就是一组原子性的 sql 查询，或者说一个独立的工作单元。如果数据库引擎能够成功地对数据库应用该组查询的全部语句，那么就执行该组查询。如果其中有任何一个语句因为崩溃或其他原因无法执行，那么所有的语句都不会执行。也就是说事务内的语句，要么全部执行成功，要么全部执行失败。

[](https://link.juejin.cn?target=)我们可以先了解一下事务的 ACID 概念。ACID 表示原子性，一致性，隔离性和持久性。

[](https://link.juejin.cn?target=)原子性：一个事务必须被视为一个不可分割的最小工作单元，整个事务中所有操作要么全部提交成功，要么全部失败回滚，对于一个事务来说，不可能只执行其中的一部分操作，这就是原子性操作。

[](https://link.juejin.cn?target=)一致性：数据库总是从一个一致性的状态转换到另外一个一致性的状态。

[](https://link.juejin.cn?target=)隔离性：通常来说，一个事务所做的修改在最终提交之前，对其他事务是不可见的。

[](https://link.juejin.cn?target=)持久性：一旦事务提交，则其所做的修改就会永久保存到数据库中

[](https://link.juejin.cn?target=)**隔离级别**

[](https://link.juejin.cn?target=)在 SQL 标准中定义了四种隔离级别，每一种级别都规定了一个事务中所做的修改，哪些在事务内和事务间是可见的，哪些是不可见的。较低级别的隔离通常可以执行更高的并发，系统的开销也更低。

[](https://link.juejin.cn?target=)READ UNCOMMITTED(未提交读)：事务中的修改，即使没有提交，对其他事务也是可见的。事务可以读取未提交的数据，这也被称为脏读。

[](https://link.juejin.cn?target=)READ COMMITTED(已提交读)：大多数数据库系统的默认隔离级别都是 READ COMMITTED。该级别满足隔离级别的简单定义，一个事务开始时，只能看见已经提交的事务所做的修改。换句话说就是一个事务从开始到提交之前，所做的任何修改对其他事务都不可见。也叫不可重复读。

[](https://link.juejin.cn?target=)REPEATABLE READ(可重复读)：,MYSQL 默认隔离级别。该级别解决了脏读的问题。保证了在同一事务中多次读取同样记录的结果是一致的。但是理论上可重复读无法解决幻读的问题。所谓幻读就是说当某个事务在读取某个范围内的记录时，另一个事务又在该范围内插入了新的记录，当之前的事务再次读取该范围内的记录时，会产生幻行。InnoDB 等存储引擎通过多版本并发控制解决了幻读的问题。

[](https://link.juejin.cn?target=)SERIALIZABLE(可串行化)：最高隔离级别，通过强制事务串行执行，避免了幻读的问题。简单来说就是给每行数据加上锁，所以可能会导致大量的超时和锁争用的问题。实际应用中很少用这个隔离级别。

<table><thead><tr><th>隔离级别</th><th>脏读可能性</th><th>不可重复读可能性</th><th>幻读可能性</th><th>加锁读</th></tr></thead><tbody><tr><td>READ UNCOMMITTED</td><td>√</td><td>√</td><td>√</td><td>×</td></tr><tr><td>READ COMMITTED</td><td>×</td><td>√</td><td>√</td><td>×</td></tr><tr><td>REPEATABLE READ</td><td>×</td><td>×</td><td>√</td><td>×</td></tr><tr><td>SERIALIZABLE</td><td>×</td><td>×</td><td>×</td><td>√</td></tr></tbody></table>

[](https://link.juejin.cn?target=)**死锁**

[](https://link.juejin.cn?target=)死锁是指两个或多个事务在同一资源上相互占用，并请求锁定对方占用的资源，从而导致恶性循环的现象。当多个事务视图以不同的顺序锁定资源时，就可能会产生死锁。多个事务同时锁定同一个资源时，也会产生死锁。

[](https://link.juejin.cn?target=)InnoDB 目前解决死锁的方法是将持有最少行级排他锁的事务进行回滚。

[](https://link.juejin.cn?target=)**事务日志**

[](https://link.juejin.cn?target=)事务日志可以帮助提高事务的效率，使用事务日志，存储引擎在修改表的数据时只需要修改其内存拷贝，再把该修改行为记录到持久化的事务日志中，而不用每次都将修改的数据本身持久化到磁盘。事务日志使用追加的方式，因此写日志的操作时磁盘上一小块区域内的顺序 IO，而不像随机 IO 需要再磁盘的多个地方移动磁头。事务日志持久化之后，内存中被修改的数据在后台可以慢慢的回刷到磁盘。

[](https://link.juejin.cn?target=)如果数据的修改已经记录到事务日志并持久化，但数据本身还没有写回磁盘，此时系统崩溃，存储引擎在重启时能够自动修复这部分修改的数据。

[](https://link.juejin.cn?target=)**Mysql 中的事务**

[](https://link.juejin.cn?target=)**自动提交**

[](https://link.juejin.cn?target=)Mysql 默认采用自动提交模式，也就是说如果不是显式地开始一个事务，则每个查询都被当做一个事务执行提交操作。

[](https://link.juejin.cn?target=)**在事务中混合使用存储引擎**

[](https://link.juejin.cn?target=)Mysql 服务器层不管理事务，事务是由下层的存储引擎实现的。所以在同一个事务中，使用多个存储引擎是不可靠的。如果在事务中混合使用了事务型和非事务型的表，再正常提交的情况下不会有什么问题，但是如果需要回滚的话，非事务型的表上的修改就无法撤销从而导致数据不一致。

[](https://link.juejin.cn?target=)**隐式和显式锁定**

[](https://link.juejin.cn?target=)InnoDB 采用的是两阶段锁定协议。在事务执行过程中，随时都可以执行锁定，锁只有在执行 COMMIT 或者 ROLLBACK 的时候才会释放，并且所有的锁是在同一时刻释放。InnoDB 会根据隔离级别在需要的时候自动加锁。

[](https://link.juejin.cn?target=)Mysql 也可以通过特地的语句进行显式的锁定。

```
SELECT ... LOCK IN SHARE MODE;
SELECT ... FOR UPDATE;


```

[](https://link.juejin.cn?target=)**多版本并发控制**

[](https://link.juejin.cn?target=)Mysql 的大多数事务型存储引擎实现的都不是简单的行级锁，基于提升并发性能的考虑，一般都同时实现了多版本并发控制。MVCC 是行级锁的一个变种，他在很多情况下避免了加锁操作，因此开销更低。虽然实现机制有所不同，但大都实现了非阻塞的读操作，写操作也只锁定必要的行。

[](https://link.juejin.cn?target=)MVCC 的实现是通过保存数据在某个时间点的快照来实现的。也就是说不管需要执行多长时间，每个事务看到的数据都是一致的。根据事务的开始时间不同，每个事务对同一张表，同一时刻看到的数据可能是不一样的。

[](https://link.juejin.cn?target=)不同的存储引擎 MVCC 的实现是不同的，典型有乐观并发控制和悲观并发控制。

[](https://link.juejin.cn?target=)InnoDB 的 MVCC 是通过在每行记录后面保存两个隐藏的列来实现的。这两列，一个保存了行的创建时间，一个保存行的过期时间。当然存储的并不是实际的时间值而是系统版本号。每开始一个新的事务，系统版本号都会自动递增。事务开始时刻的系统版本号作为事务的版本号，用来和查询到的每行记录的版本号进行比较。下面我们来看看在 REAPEATABLE READ 隔离级别下。MVCC 是如何操作的。

[](https://link.juejin.cn?target=)SELECT：InnoDB 会根据以下两个条件检查每行记录：

[](https://link.juejin.cn?target=)InnoDB 值查找版本早于当前事务版本的数据行，这样就可以保证事务读取的行要么是事务开始前已经存在得，要么是事务自身修改过的。

[](https://link.juejin.cn?target=)行的删除版本要么未定义，要么大于当前事务版本号。确保事务读取到的行在事务开始之前未被删除。

[](https://link.juejin.cn?target=)只有符合上面两个条件，才能返回查询结果。

[](https://link.juejin.cn?target=)INSERT：InnoDB 为新插入的行保存当前系统版本号作为行版本号

[](https://link.juejin.cn?target=)DELETE：InnoDB 为删除的每一行保存当前系统版本号作为行删除标识

[](https://link.juejin.cn?target=)UPDATE：InnoDB 为新插入的行保存当前系统版本号作为行版本号，同时保存当前系统版本号到原来的行作为删除标识

[](https://link.juejin.cn?target=)保存这两个系统版本号，使得大部分读操作都可以不用加锁，这样设计使得读取操作很简单，性能很好，并且也能保证只会读取到符合标准的行。不足之处就是每行记录都需要额外的存储空间。

[](https://link.juejin.cn?target=)MVCC 只在 REPEATABLE READ 和 READ COMMITTED 两个隔离级别下工作。因为 READ UNCOMMITTED 总是读取最新的数据行而不是符合当前事务版本的数据行；而 SERIALIZABLE 则会对所有读取的行都加锁。

[](https://link.juejin.cn?target=)**Mysql 的存储引擎**

[](https://link.juejin.cn?target=)在文件系统中，Mysql 将每个数据库保存为数据目录下的一个子目录。创建表时，Mysql 会在数据库子目录下创建一个和表同名的. frm 文件保存表的定义。可以使用 SHOW TABLE STATUS 命令显式表的相关信息。

```
mysql> show table status like 'user' \G
*************************** 1. row ***************************
Name: user
Engine: InnoDB 
Version: 10 
Row_format: Dynamic 
Rows: 9 
Avg_row_length: 1820 
Data_length: 16384 
Max_data_length: 0 
Index_length: 0 
Data_free: 0 
Auto_increment: 14 
Create_time: 2023-09-24 17:34:24 
Update_time: NULL 
Check_time: NULL 
Collation: utf8mb4_general_ci 
Checksum: NULL 
Create_options: 
Comment: 
1 row in set (0.00 sec)


```

[](https://link.juejin.cn?target=)下面简单介绍一下每一行的含义。

[](https://link.juejin.cn?target=)Name：表名

[](https://link.juejin.cn?target=)Engine：存储引擎，再老版本中这列叫 Type

[](https://link.juejin.cn?target=)Row_format：行的格式，对于 InnoDB 的表，常见的行格式类型有 Compact、Redundant、Dynamic 和 Compressed。

[](https://link.juejin.cn?target=)Compact 行格式：

[](https://link.juejin.cn?target=)Compact 行格式是 MySQL5.0 中引入的，其目标是为了更高效的存储数据记录。其存储结构示意图如下：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d48cb1ae0a6e4b11afcd1798c6739eb2~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1135&h=415&s=21719&e=png&b=ffffff)

[](https://link.juejin.cn?target=)从图中我们可以看出来，一条完整的记录其实可以被分为记录的额外信息和记录的真实数据两部分。

[](https://link.juejin.cn?target=)记录的额外信息

[](https://link.juejin.cn?target=)这部分信息是为了描述这条记录而不是额外添加的一些信息，这些额外信息分为三部分，分别是变长字段长度列表、NULL 值列表和记录头信息；

[](https://link.juejin.cn?target=)变长字段的长度列表

[](https://link.juejin.cn?target=)MySQL 支持一些变长的数据类型，比如 VARCHAR(M)、 VARBINARY(M)、 TEXT 类型、BLOB 类型，这些数据类型修饰列称为变长字段。变长字段中存储多少字节的数据不是固定的，所以我们在存储真实数据的时候需要顺便把这些数据占用的字节数也存起来。也就是变长字段时占用了两部分空间来存储的，（1）真实的数据内容，（2）占用的字节数。

[](https://link.juejin.cn?target=)在 Compact 行格式中，把所有变长字段的真实数据占用的字节长度都存放在记录的开头部位，从而形成一个变长字段长度列表。

[](https://link.juejin.cn?target=)需要注意的是，这里面存储的变长长度和字段顺序是相反的。比如两个 varchar 字段在表结构的顺序是 a(10), b(15)，那么在变长字段长度列表中存储的长度顺序就是 15,10，是反过来的。变长字段的长度列表不是一定存在的，如果表中没有变长类型的字段，或者该记录中所有的变长字段值均为 NULL。

[](https://link.juejin.cn?target=)NULL 值列

[](https://link.juejin.cn?target=)表中的某些列可能存储 NULL 值，如果把这些 NULL 值都放到记录的真实数据中存储则显的比较浪费空间，所以 Compact 行格式把这些值为 NULL 的列统一管理起来，存储到 NULL 值列表中。

[](https://link.juejin.cn?target=)如果表中有字段允许为 NULL，InnoDB 就会开辟一块空间来标识每个字段实际存储的数据是不是 NULL，如果表中的字段都不允许为 NULL，则 NULL 值列表也就不存在了。

[](https://link.juejin.cn?target=)每个允许存储 NULL 的列对应一个二进制位，二进制位按照列的逆序排列，二进制位表示的意义如下：

[](https://link.juejin.cn?target=)二进制位的值为 1 时，代表该列的值为 NULL。

[](https://link.juejin.cn?target=)二进制位的值为 0 时，代表该列的值不为 NULL。

[](https://link.juejin.cn?target=)记录头信息

[](https://link.juejin.cn?target=)记录头信息用于描述该记录的，它是由固定的 5 个字节组成，即 40 个二进制位，不同的位代表不同的意思；

[](https://link.juejin.cn?target=)这些记录头信息中各个属性如下：

<table><thead><tr><th>名称</th><th>大小 (bit)</th><th>描述</th></tr></thead><tbody><tr><td>预留位 1</td><td>1</td><td>没有使用</td></tr><tr><td>预留位 2</td><td>1</td><td>没有使用</td></tr><tr><td>delete_mask</td><td>1</td><td>标记该记录是否被删除。0 - 没有被删除，1 - 记录被删除那么被删除的记录为什么还在页中存储呢？实际上记录并没有从磁盘消失。这些被删除的记录之所以不立即从磁盘上移除是因为移除它们之后其他的记录在磁盘上需要重新排列，导致性能消耗。所以只是打一个标记而已，所有被删除掉的记录都会组成一个所谓的垃圾链表，在这个链表中的记录占用的空间称之为可重用空间，之后如果有新纪录插入到表中的话，可能把这些被删除的记录占用的存储空间覆盖掉。</td></tr><tr><td>min_rec_mask</td><td>1</td><td>B + 树的每层非叶子节点中的最小记录都会添加该标记，值为 1。 0 - 表示不是 B + 树的非叶子节点中的最小记录</td></tr><tr><td>n_owned</td><td>4</td><td>表示当前记录拥有的记录数 。页目录中每个组最后一条记录的头信息中会存储该组一共有多少条记录，作为 n_owned 字段</td></tr><tr><td>heap_no</td><td>13</td><td>表示当前记录在本页中的位置信息，heap_no 是没有值为 0 和 1 的。这是因为 MySQL 会自动给每个页里加两个记录，这两个记录并不是我们自己插入的，所以有时候也称为伪记录或者虚拟记录。这两个伪记录一个代表最小记录，一个代表最大记录。最小记录和最大记录的 heap_no 值分别是 0 和 1，也就是说它们的位置最靠前。</td></tr><tr><td>record_type</td><td>3</td><td>表示当前记录的类型：0 - 普通记录， 1- B + 树非叶子节点记录，2 - 最小记录，3 - 最大记录</td></tr><tr><td>next_record</td><td>16</td><td>表示下一条记录的相对位置，也就是从当前记录的真实数据到下一条记录的真实数据的地址偏移量。</td></tr></tbody></table>

[](https://link.juejin.cn?target=)记录的数据内容

[](https://link.juejin.cn?target=)记录的真实数据除了自定义的列的数据以外，MySQL 还会为每条记录默认的添加一些列（也称为隐藏列），具体的列如下：

<table><thead><tr><th>列名</th><th>是否必须</th><th>占用空间</th><th>描述</th></tr></thead><tbody><tr><td>DB_ROW_ID</td><td>否</td><td>6 字节</td><td>行 ID，唯一标识一条记录</td></tr><tr><td>DB_TRX_ID</td><td>是</td><td>6 字节</td><td>事务 ID</td></tr><tr><td>DB_ROLL_PTR</td><td>是</td><td>7 字节</td><td>回滚指针</td></tr></tbody></table>

[](https://link.juejin.cn?target=)当用户未指定数据表的主键时，MySQL 会选择非 NULL 的 Unique 列作为主键，而如果非 NULL 的 Unique 列也没有，这个时候 MySQL 就会向数据表添加 DB_ROW_ID 字段用来作为主键。记录的数据内容不包括字段值为 NULL 的数据内容。

[](https://link.juejin.cn?target=)Redundant 行格式

[](https://link.juejin.cn?target=)redundant 行格式是 MySQL5.0 之前的一种旧的格式，其结构与 compact 行格式大体还是比较相似的。

[](https://link.juejin.cn?target=)Dynamic 和 Compressed 行格式

[](https://link.juejin.cn?target=)在 MySQL8.0 中，默认行格式就是 Dynamic。Dynamic、Compressed 行格式和 Compact 行格式挺像，只不过在处理行溢出数据时有分歧：

[](https://link.juejin.cn?target=)Compressed 和 Dynamic 两种记录格式对于存放在 BLOB 中的数据采用了完全的行溢出的方式。在数据页中只存放 20 个字节的指针（溢出页的地址），实际的数据都存放在 Off Page（溢出页）中。

[](https://link.juejin.cn?target=)Compact 和 Redundant 两种格式会在记录的真实数据处存储一部分数据（存放 768 个前缀字节）。

[](https://link.juejin.cn?target=)Compressed 行记录格式的另一个功能就是，存储在其中的行数据会以 zlib 的算法进行压缩，因此对于 blob、text、varchar 这类大长度类型的数据能够进行非常有效的存储。

[](https://link.juejin.cn?target=)Rows：表中的行数，InnoDB 引擎该值是估计值，MyISAM 引擎是精确值

[](https://link.juejin.cn?target=)Avg_row_length：平均每行包含的字节数

[](https://link.juejin.cn?target=)Data_length：表数据的大小

[](https://link.juejin.cn?target=)Max_data_length：表数据的最大容量，该值和存储引擎有关

[](https://link.juejin.cn?target=)Index_length：索引的大小

[](https://link.juejin.cn?target=)Data_free：对于 MyISAM 表，表示已分配但目前没有使用的空间。这部分空间包括了之前删除的行，以及后续可以被 INSERT 利用的空间

[](https://link.juejin.cn?target=)Auto_increment：下一个 AUTO_INCREMENT 的值

[](https://link.juejin.cn?target=)Create_time：表的创建时间

[](https://link.juejin.cn?target=)Update_time：表数据的最后修改时间

[](https://link.juejin.cn?target=)Check_time：使用 CHECK TABLE 命令或者 myisamchk 工具最后一次检查表的时间

[](https://link.juejin.cn?target=)Collation：表的默认字符集和字符列排序规则

[](https://link.juejin.cn?target=)Checksum：如果启用，保存的是整个表的实时校验和

[](https://link.juejin.cn?target=)Create_options：创建表时指定的其他选项

[](https://link.juejin.cn?target=)Comment：表注释

[](https://link.juejin.cn?target=)**InnoDB 存储引擎**

[](https://link.juejin.cn?target=)InnoDB 采用 MVCC 来支持高并发，并且实现了四个标准的隔离级别。其默认级别是 REAPEATABLE READ，并且通过间隙锁策略防止幻读的出现。间隙锁使得 InnoDB 不仅锁定查询涉及的行，还会对索引中的间隙进行锁定，以防止幻行的插入。

[](https://link.juejin.cn?target=)InnoDB 表是基于聚簇索引建立的。InnoDB 内部做了很多优化，包括从磁盘读取数据时采用的可预测性预读，能够自动在内存中创建 hash 索引以加速读操作的自适应哈希索引。

[](https://link.juejin.cn?target=)**MyISAM 存储引擎**

[](https://link.juejin.cn?target=)在 Mysql5.1 及之前的版本，MyISAM 是默认的存储引擎。MyISAM 提供了大量的特性，包括全文索引、压缩、空间函数等，但 MyISAM 不支持事务和行级锁，而且崩溃后无法完全恢复。

[](https://link.juejin.cn?target=)**存储**

[](https://link.juejin.cn?target=)MyISAM 会将表存储在两个文件中：数据文件和索引文件，分别以 .myd 和 .myi 为扩展名。Mysql 会根据表的定义来决定采用哪种行格式。

[](https://link.juejin.cn?target=)**特性**

[](https://link.juejin.cn?target=)**加锁与并发**

[](https://link.juejin.cn?target=)MyISAM 对整张表加锁，而不是针对行。读取时会对需要读到的所有表加共享锁，写入时则对表加排他锁。但是在表有读取操作的时候，也可以进行插入操作。

[](https://link.juejin.cn?target=)**修复**

[](https://link.juejin.cn?target=)对于 MyISAM 表，Mysql 可以手工或者自动执行检查和修复操作。执行表的修复可能会导致一些数据的丢失，而且修复操作非常慢。

[](https://link.juejin.cn?target=)**索引特性**

[](https://link.juejin.cn?target=)对于 MyISAM 表即使是 BLOB 和 TEXT 等字段，也可以基于前 500 个字符创建索引。MyISAM 也支持全文索引

[](https://link.juejin.cn?target=)**延迟更新索引键**

[](https://link.juejin.cn?target=)创建 MyISAM 表的时候，如果指定了 DELAY_KEY_WRITE 选项，在每次修改执行完成时，不会立刻将修改的索引数据写入磁盘。而是会写到内存中的键缓冲区，只有在清理键缓冲区或者关闭表时才会将对应的索引快写入磁盘。这种方式提高了写入性能但是在服务器崩溃的时候会造成索引损坏需要执行修复操作。

[](https://link.juejin.cn?target=)**压缩表**

[](https://link.juejin.cn?target=)如果表在创建并导入数据以后不会再进行修改操作，那么这样的表适合采用 MyISAM 压缩表。

[](https://link.juejin.cn?target=)可以使用 myisampack 对 MyISAM 表进行压缩。压缩表是不能进行修改的，极大的减少磁盘空间占用，因此也减少磁盘 IO，从而提升查询性能。