> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [lxw1234.com](http://lxw1234.com/archives/2015/09/504.htm)

> 关键字：Kafka、Kafka 架构、Kafka 原理 背景介绍 Kafka 简介 Kafka 是一种分布式的，基于发布 / 订阅的消息系统。

关键字：Kafka、Kafka 架构、Kafka 原理

Kafka 简介
--------

Kafka 是一种分布式的，基于发布 / 订阅的消息系统。主要设计目标如下：

*   以时间复杂度为 O(1) 的方式提供消息持久化能力，并保证即使对 TB 级以上数据也能保证常数时间的访问性能
*   高吞吐率。即使在非常廉价的商用机器上也能做到单机支持每秒 100K 条消息的传输
*   支持 Kafka Server 间的消息分区，及分布式消息消费，同时保证每个 partition 内的消息顺序传输
*   同时支持离线数据处理和实时数据处理

为什么要用 Message Queue
-------------------

*   **解耦**  
    在项目启动之初来预测将来项目会碰到什么需求，是极其困难的。消息队列在处理过程中间插入了一个隐含的、基于数据的接口层，两边的处理过程都要实现这一接口。这允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束
*   **冗余**  
    有时在处理数据的时候处理过程会失败。除非数据被持久化，否则将永远丢失。消息队列把数据进行持久化直到它们已经被完全处理，通过这一方式规避了数据丢失 风险。在被许多消息队列所采用的” 插入 - 获取 - 删除” 范式中，在把一个消息从队列中删除之前，需要你的处理过程明确的指出该消息已经被处理完毕，确保你的 数据被安全的保存直到你使用完毕。
*   **扩展性**  
    因为消息队列解耦了你的处理过程，所以增大消息入队和处理的频率是很容易的；只要另外增加处理过程即可。不需要改变代码、不需要调节参数。扩展就像调大电力按钮一样简单。
*   **灵活性 & 峰值处理能力**  
    在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见；如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列能够使关键组件顶住增长的访问压力，而不是因为超出负荷的请求而完全崩溃。
*   **可恢复性**  
    当体系的一部分组件失效，不会影响到整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。而这种允许重试或者延后处理请求的能力通常是造就一个略感不便的用户和一个沮丧透顶的用户之间的区别。
*   **送达保证**  
    消息队列提供的冗余机制保证了消息能被实际的处理，只要一个进程读取了该队列即可。在此基础上，IronMQ 提供了一个” 只送达一次” 保证。无论有多少进 程在从队列中领取数据，每一个消息只能被处理一次。这之所以成为可能，是因为获取一个消息只是” 预定” 了这个消息，暂时把它移出了队列。除非客户端明确的 表示已经处理完了这个消息，否则这个消息会被放回队列中去，在一段可配置的时间之后可再次被处理。

*   **顺序保证**  
    在许多情况下，数据处理的顺序都很重要。消息队列本来就是排序的，并且能保证数据会按照特定的顺序来处理。IronMO 保证消息浆糊通过 FIFO（先进先出）的顺序来处理，因此消息在队列中的位置就是从队列中检索他们的位置。
*   **缓冲**  
    在任何重要的系统中，都会有需要不同的处理时间的元素。例如, 加载一张图片比应用过滤器花费更少的时间。消息队列通过一个缓冲层来帮助任务最高效率的执行—写入队列的处理会尽可能的快速，而不受从队列读的预备处理的约束。该缓冲有助于控制和优化数据流经过系统的速度。
*   **理解数据流**  
    在一个分布式系统里，要得到一个关于用户操作会用多长时间及其原因的总体印象，是个巨大的挑战。消息系列通过消息被处理的频率，来方便的辅助确定那些表现不佳的处理过程或领域，这些地方的数据流都不够优化。
*   **异步通信**  
    很多时候，你不想也不需要立即处理消息。消息队列提供了异步处理机制，允许你把一个消息放入队列，但并不立即处理它。你想向队列中放入多少消息就放多少，然后在你乐意的时候再去处理它们。

常用 Message Queue 对比
-------------------

*   **RabbitMQ**  
    RabbitMQ 是使用 Erlang 编写的一个开源的消息队列，本身支持很多的协议：AMQP，XMPP, SMTP, STOMP，也正因如此，它非常重量级，更适合于企业级的开发。同时实现了 Broker 构架，这意味着消息在发送给客户端时先在中心队列排队。对路由，负 载均衡或者数据持久化都有很好的支持。
*   **Redis**  
    Redis 是一个基于 Key-Value 对的 NoSQL 数据库，开发维护很活跃。虽然它是一个 Key-Value 数据库存储系统，但它本身支持 MQ 功能， 所以完全可以当做一个轻量级的队列服务来使用。对于 RabbitMQ 和 Redis 的入队和出队操作，各执行 100 万次，每 10 万次记录一次执行时间。测试 数据分为 128Bytes、512Bytes、1K 和 10K 四个不同大小的数据。实验表明：入队时，当数据比较小时 Redis 的性能要高于 RabbitMQ，而如果数据大小超过了 10K，Redis 则慢的无法忍受；出队时，无论数据大小，Redis 都表现出非常好的性能，而 RabbitMQ 的出队性能则远低于 Redis。
*   **ZeroMQ**  
    ZeroMQ 号称最快的消息队列系统，尤其针对大吞吐量的需求场景。ZMQ 能够实现 RabbitMQ 不擅长的高级 / 复杂的队列，但是开发人员需要自己组合 多种技术框架，技术上的复杂度是对这 MQ 能够应用成功的挑战。ZeroMQ 具有一个独特的非中间件的模式，你不需要安装和运行一个消息服务器或中间件，因 为你的应用程序将扮演了这个服务角色。你只需要简单的引用 ZeroMQ 程序库，可以使用 NuGet 安装，然后你就可以愉快的在应用程序之间发送消息了。但 是 ZeroMQ 仅提供非持久性的队列，也就是说如果 down 机，数据将会丢失。其中，Twitter 的 Storm 中默认使用 ZeroMQ 作为数据流的传 输。
*   **ActiveMQ**  
    ActiveMQ 是 Apache 下的一个子项目。 类似于 ZeroMQ，它能够以代理人和点对点的技术实现队列。同时类似于 RabbitMQ，它少量代码就可以高效地实现高级应用场景。
*   **Kafka/Jafka**  
    Kafka 是 Apache 下的一个子项目，是一个高性能跨语言分布式 Publish/Subscribe 消息队列系统，而 Jafka 是在 Kafka 之上孵 化而来的，即 Kafka 的一个升级版。具有以下特性：快速持久化，可以在 O(1) 的系统开销下进行消息持久化；高吞吐，在一台普通的服务器上既可以达到 10W/s 的吞吐速率；完全的分布式系统，Broker、Producer、Consumer 都原生自动支持分布式，自动实现复杂均衡；支持 Hadoop 数据并行加载，对于像 Hadoop 的一样的日志数据和离线分析系统，但又要求实时处理的限制，这是一个可行的解决方案。Kafka 通过 Hadoop 的并行 加载机制来统一了在线和离线的消息处理，这一点也是本课题所研究系统所看重的。Apache Kafka 相对于 ActiveMQ 是一个非常轻量级的消息系统，除了性能非常好之外，还是一个工作良好的分布式系统。

Terminology
-----------

*   **Broker**  
    Kafka 集群包含一个或多个服务器，这种服务器被称为 broker
*   **Topic**  
    每条发布到 Kafka 集群的消息都有一个类别，这个类别被称为 topic。（物理上不同 topic 的消息分开存储，逻辑上一个 topic 的消息虽然保存于一个或多个 broker 上但用户只需指定消息的 topic 即可生产或消费数据而不必关心数据存于何处）
*   **Partition**  
    parition 是物理上的概念，每个 topic 包含一个或多个 partition，创建 topic 时可指定 parition 数量。每个 partition 对应于一个文件夹，该文件夹下存储该 partition 的数据和索引文件
*   **Producer**  
    负责发布消息到 Kafka broker
*   **Consumer**  
    消费消息。每个 consumer 属于一个特定的 consuer group（可为每个 consumer 指定 group name，若不指定 group name 则属于默认的 group）。使用 consumer high level API 时，同一 topic 的一条消息只能被同一个 consumer group 内的一个 consumer 消费，但多个 consumer group 可同时消费这一消息。

Kafka 架构
--------

![](http://img.lxw1234.com/0924-1.jpg)

如上图所示，一个典型的 kafka 集群中包含若干 producer（可以是 web 前端产生的 page view，或者是服务器日志，系统 CPU、memory 等），若干 broker（Kafka 支持水平扩展，一般 broker 数量越多，集群吞吐率越高）， 若干 consumer group，以及一个 [Zookeeper](http://zookeeper.apache.org/) 集 群。Kafka 通过 Zookeeper 管理集群配置，选举 leader，以及在 consumer group 发生变化时进行 rebalance。producer 使用 push 模式将消息发布到 broker，consumer 使用 pull 模式从 broker 订阅并消费消息。

### Push vs. Pull

作为一个 messaging system，Kafka 遵循了传统的方式，选择由 producer 向 broker push 消息并由 consumer 从 broker pull 消息。一些 logging-centric system，比如 Facebook 的 [Scribe](https://github.com/facebookarchive/scribe) 和 Cloudera 的 [Flume](http://flume.apache.org/), 采用非常不同的 push 模式。事实上，push 模式和 pull 模式各有优劣。  
push 模式很难适应消费速率不同的消费者，因为消息发送速率是由 broker 决定的。push 模式的目标是尽可能以最快速度传递消息，但是这样很容易造 成 consumer 来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。而 pull 模式则可以根据 consumer 的消费能力以适当的速率消费消息。

### Topic & Partition

Topic 在逻辑上可以被认为是一个在的 queue，每条消费都必须指定它的 topic，可以简单理解为必须指明把这条消息放进哪个 queue 里。 为了使得 Kafka 的吞吐率可以水平扩展，物理上把 topic 分成一个或多个 partition，每个 partition 在物理上对应一个文件夹，该文件 夹下存储这个 partition 的所有消息和索引文件。

![](http://img.lxw1234.com/0924-2.jpg)

每个日志文件都是 “log entries” 序列，每一个`log entry`包含一个 4 字节整型数（值为 N），其后跟 N 个字节的消息体。每条消息都有一个当前 partition 下唯一的 64 字节的 offset，它指明了这条消息的起始位置。磁盘上存储的消费格式如下：  
message length ： 4 bytes (value: 1+4+n)  
“magic” value ： 1 byte  
crc ： 4 bytes  
payload ： n bytes  
这个 “log entries” 并非由一个文件构成，而是分成多个 segment，每个 segment 名为该 segment 第一条消息的 offset 和 “.kafka” 组成。另外会有一个索引文件，它标明了每个 segment 下包含的`log entry`的 offset 范围，如下图所示。

![](http://img.lxw1234.com/0924-3.jpg)

因为每条消息都被 append 到该 partition 中，是顺序写磁盘，因此效率非常高（经验证，顺序写磁盘效率比随机写内存还要高，这是 Kafka 高吞吐率的一个很重要的保证）。

![](http://img.lxw1234.com/0924-4.jpg)

每一条消息被发送到 broker 时，会根据 paritition 规则选择被存储到哪一个 partition。如果 partition 规则设置的合理，所有 消息可以均匀分布到不同的 partition 里，这样就实现了水平扩展。（如果一个 topic 对应一个文件，那这个文件所在的机器 I/O 将会成为这个 topic 的性能瓶颈，而 partition 解决了这个问题）。在创建 topic 时可以在`$KAFKA_HOME/config/server.properties`中指定这个 partition 的数量 (如下所示)，当然也可以在 topic 创建之后去修改 parition 数量。

```
# The default number of log partitions per topic. More partitions allow greater
# parallelism for consumption, but this will also result in more files across
# the brokers.
num.partitions=3

```

在发送一条消息时，可以指定这条消息的 key，producer 根据这个 key 和 partition 机制来判断将这条消息发送到哪个 parition。 paritition 机制可以通过指定 producer 的 paritition. class 这一参数来指定，该 class 必须实现`kafka.producer.Partitioner`接口。本例中如果 key 可以被解析为整数则将对应的整数与 partition 总数取余，该消息会被发送到该数对应的 partition。（每个 parition 都会有个序号）

```
import kafka.producer.Partitioner;
import kafka.utils.VerifiableProperties;
 
public class JasonPartitioner implements Partitioner {
    public JasonPartitioner(VerifiableProperties verifiableProperties) {}
    @Override
    public int partition(Object key, int numPartitions) {
        try {
            int partitionNum = Integer.parseInt((String) key);
            return Math.abs(Integer.parseInt((String) key) % numPartitions);
        } catch (Exception e) {
            return Math.abs(key.hashCode() % numPartitions);
        }
    }
}

```

如果将上例中的 class 作为 partition.class，并通过如下代码发送 20 条消息（key 分别为 0，1，2，3）至 topic2（包含 4 个 partition）。

```
public void sendMessage() throws InterruptedException{
　　for(int i = 1; i <= 5; i++){
　　      List messageList = new ArrayList<KeyedMessage<String, String>>();
　　      for(int j = 0; j < 4; j++）{
　　          messageList.add(new KeyedMessage<String, String>("topic2", j+"", "The " + i + " message for key " + j));
　　      }
　　      producer.send(messageList);
    }
　　producer.close();
}

```

则 key 相同的消息会被发送并存储到同一个 partition 里，而且 key 的序号正好和 partition 序号相同。（partition 序号从 0 开始，本例中的 key 也正好从 0 开始）。如下图所示。

![](http://img.lxw1234.com/0924-5.jpg)

对于传统的 message queue 而言，一般会删除已经被消费的消息，而 Kafka 集群会保留所有的消息，无论其被消费与否。当然，因为磁盘限制，不可能永久保留所有数据（实际 上也没必要），因此 Kafka 提供两种策略去删除旧数据。一是基于时间，二是基于 partition 文件大小。例如可以通过配置`$KAFKA_HOME/config/server.properties`，让 Kafka 删除一周前的数据，也可通过配置让 Kafka 在 partition 文件超过 1GB 时删除旧数据，如下所示。

```
　　############################# Log Retention Policy #############################
 
# The following configurations control the disposal of log segments. The policy can
# be set to delete segments after a period of time, or after a given size has accumulated.
# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
# from the end of the log.
 
# The minimum age of a log file to be eligible for deletion
log.retention.hours=168
 
# A size-based retention policy for logs. Segments are pruned from the log as long as the remaining
# segments don't drop below log.retention.bytes.
#log.retention.bytes=1073741824
 
# The maximum size of a log segment file. When this size is reached a new log segment will be created.
log.segment.bytes=1073741824
 
# The interval at which log segments are checked to see if they can be deleted according
# to the retention policies
log.retention.check.interval.ms=300000
 
# By default the log cleaner is disabled and the log retention policy will default to 
#just delete segments after their retention expires.
# If log.cleaner.enable=true is set the cleaner will be enabled and individual logs 
#can then be marked for log compaction.
log.cleaner.enable=false

```

这里要注意，因为 Kafka 读取特定消息的时间复杂度为 O(1)，即与文件大小无关，所以这里删除文件与 Kafka 性能无关，选择怎样的删除策略只 与磁盘以及具体的需求有关。另外，Kafka 会为每一个 consumer group 保留一些 metadata 信息—当前消费的消息的 position，也即 offset。这个 offset 由 consumer 控制。正常情况下 consumer 会在消费完一条消息后线性增加这个 offset。当然，consumer 也可将 offset 设成一个较小的值，重新消费一些消息。因为 offet 由 consumer 控制，所以 Kafka broker 是无状态的，它不需要标记哪些消息被哪些 consumer 过，不需要通过 broker 去保证同一个 consumer group 只有一个 consumer 能消费某一条消息，因此也就不需要锁机制，这也为 Kafka 的高吞吐率提供了有力保障。

### Replication & Leader election

Kafka 从 0.8 开始提供 partition 级别的 replication，replication 的数量可在`$KAFKA_HOME/config/server.properties`中配置。

```
default.replication.factor = 1

```

该 Replication 与 leader election 配合提供了自动的 failover 机制。replication 对 Kafka 的吞吐率是有一定影响的，但极大的增强了可用性。默认情况 下，Kafka 的 replication 数量为 1。　　每个 partition 都有一个唯一的 leader，所有的读写操作都在 leader 上完 成，leader 批量从 leader 上 pull 数据。一般情况下 partition 的数量大于等于 broker 的数量，并且所有 partition 的 leader 均匀分布在 broker 上。follower 上的日志和其 leader 上的完全一样。  
和大部分分布式系统一样，Kakfa 处理失败需要明确定义一个 broker 是否 alive。对于 Kafka 而言，Kafka 存活包含两个条件，一是它必须 维护与 Zookeeper 的 session(这个通过 Zookeeper 的 heartbeat 机制来实现)。二是 follower 必须能够及时将 leader 的 writing 复制过来，不能 “落后太多”。  
leader 会 track“in sync”的 node list。如果一个 follower 宕机，或者落后太多，leader 将把它从”in sync” list 中移除。这里所描述的 “落后太多” 指 follower 复制的消息落后于 leader 后的条数超过预定值，该值可在`$KAFKA_HOME/config/server.properties`中配置

```
#If a replica falls more than this many messages behind the leader, the leader will remove the follower from ISR and treat it as dead
replica.lag.max.messages=4000
 
#If a follower hasn't sent any fetch requests for this window of time, the leader will remove the follower from ISR (in-sync replicas) and treat it as dead
replica.lag.time.max.ms=10000  

```

需要说明的是，Kafka 只解决”fail/recover”，不处理 “Byzantine”（“拜占庭”）问题。  
一条消息只有被 “in sync” list 里的所有 follower 都从 leader 复制过去才会被认为已提交。这样就避免了部分数据被写进了 leader，还没来得及被任何 follower 复制就宕机了，而造成数据丢失（consumer 无法消费这些数据）。而对于 producer 而言，它可以选择是否等待消息 commit，这可以通过`request.required.acks`来设置。这种机制确保了只要 “in sync” list 有一个或以上的 flollower，一条被 commit 的消息就不会丢失。  
这里的复制机制即不是同步复制，也不是单纯的异步复制。事实上，同步复制要求 “活着的”follower 都复制完，这条消息才会被认为 commit，这种 复制方式极大的影响了吞吐率（高吞吐率是 Kafka 非常重要的一个特性）。而异步复制方式下，follower 异步的从 leader 复制数据，数据只要被 leader 写入 log 就被认为已经 commit，这种情况下如果 follwer 都落后于 leader，而 leader 突然宕机，则会丢失数据。而 Kafka 的这种使用 “in sync” list 的方式则很好的均衡了确保数据不丢失以及吞吐率。follower 可以批量的从 leader 复制数据，这样极大的提高复制性能（批量写磁盘），极 大减少了 follower 与 leader 的差距（前文有说到，只要 follower 落后 leader 不太远，则被认为在 “in sync” list 里）。

上文说明了 Kafka 是如何做 replication 的，另外一个很重要的问题是当 leader 宕机了，怎样在 follower 中选举出新的 leader。因为 follower 可能落后许多或者 crash 了，所以必须确保选择 “最新” 的 follower 作为新的 leader。一个基本的原则就 是，如果 leader 不在了，新的 leader 必须拥有原来的 leader commit 的所有消息。这就需要作一个折衷，如果 leader 在标明一条消息被 commit 前等待更多的 follower 确认，那在它 die 之后就有更 多的 follower 可以作为新的 leader，但这也会造成吞吐率的下降。  
一种非常常用的选举 leader 的方式是 “majority 灵秀”（“少数服从多数”），但 Kafka 并未采用这种方式。这种模式下，如果我们有 2f+1 个 replica（包含 leader 和 follower）， 那在 commit 之前必须保证有 f+1 个 replica 复制完消息，为了保证正确选出新的 leader，fail 的 replica 不能超过 f 个。因为在剩 下的任意 f+1 个 replica 里，至少有一个 replica 包含有最新的所有消息。这种方式有个很大的优势，系统的 latency 只取决于最快的几台 server，也就是说，如果 replication factor 是 3，那 latency 就取决于最快的那个 follower 而非最慢那个。majority vote 也有一些劣势，为了保证 leader election 的正常进行，它所能容忍的 fail 的 follower 个数比较少。如果要容忍 1 个 follower 挂掉，必须要有 3 个以上的 replica，如果要容忍 2 个 follower 挂掉，必须要有 5 个以上的 replica。也就是说，在生产环境下为了保证较高的容错程度，必须要有大量 的 replica，而大量的 replica 又会在大数据量下导致性能的急剧下降。这就是这种算法更多用在 [Zookeeper](http://zookeeper.apache.org/) 这种共享集群配置的系统中而很少在需要存储大量数据的系统中使用的原因。例如 HDFS 的 HA feature 是基于 [majority-vote-based journal](http://blog.cloudera.com/blog/2012/10/quorum-based-journaling-in-cdh4-1)，但是它的数据存储并没有使用这种 expensive 的方式。  
实际上，leader election 算法非常多，比如 Zookeper 的 [Zab](http://web.stanford.edu/class/cs347/reading/zab.pdf), [Raft](https://ramcloud.stanford.edu/wiki/download/attachments/11370504/raft.pdf) 和 [Viewstamped Replication](http://pmg.csail.mit.edu/papers/vr-revisited.pdf)。而 Kafka 所使用的 leader election 算法更像微软的 [PacificA](http://research.microsoft.com/apps/pubs/default.aspx?id=66814) 算法。  
Kafka 在 Zookeeper 中动态维护了一个 ISR（in-sync replicas） set，这个 set 里的所有 replica 都跟上了 leader，只有 ISR 里的成员才有被选为 leader 的可能。在这种模式下，对于 f+1 个 replica，一个 Kafka topic 能在保证不丢失已经 ommit 的消息的前提下容忍 f 个 replica 的失败。在大多数使用场景中，这种模式是非常有利的。事实上，为了容忍 f 个 replica 的失败，majority vote 和 ISR 在 commit 前需要等待的 replica 数量是一样的，但是 ISR 需要的总的 replica 的个数几乎是 majority vote 的一半。  
虽然 majority vote 与 ISR 相比有不需等待最慢的 server 这一优势，但是 Kafka 作者认为 Kafka 可以通过 producer 选择是否被 commit 阻塞来改善这一问题，并且节省下来的 replica 和磁盘使得 ISR 模式仍然值得。

上文提到，在 ISR 中至少有一个 follower 时，Kafka 可以确保已经 commit 的数据不丢失，但如果某一个 partition 的所有 replica 都挂了，就无法保证数据不丢失了。这种情况下有两种可行的方案：

*   等待 ISR 中的任一个 replica“活” 过来，并且选它作为 leader
*   选择第一个 “活” 过来的 replica（不一定是 ISR 中的）作为 leader

这就需要在可用性和一致性当中作出一个简单的平衡。如果一定要等待 ISR 中的 replica“活”过来，那不可用的时间就可能会相对较长。而且如果 ISR 中的所有 replica 都无法 “活” 过来了，或者数据都丢失了，这个 partition 将永远不可用。选择第一个 “活” 过来的 replica 作为 leader，而这个 replica 不是 ISR 中的 replica，那即使它并不保证已经包含了所有已 commit 的消息，它也会成为 leader 而作为 consumer 的数据源（前文有说明，所有读写都由 leader 完成）。Kafka0.8.* 使用了第二种方式。根据 Kafka 的文档，在以后的版本 中，Kafka 支持用户通过配置选择这两种方式中的一种，从而根据不同的使用场景选择高可用性还是强一致性。

上文说明了一个 parition 的 replication 过程，然尔 Kafka 集群需要管理成百上千个 partition，Kafka 通过 round-robin 的方式来平衡 partition 从而避免大量 partition 集中在了少数几个节点上。同时 Kafka 也需要平衡 leader 的 分布，尽可能的让所有 partition 的 leader 均匀分布在不同 broker 上。另一方面，优化 leadership election 的过程也是很重要的，毕竟这段时间相应的 partition 处于不可用状态。一种简单的实现是暂停宕机的 broker 上的所有 partition，并为之选举 leader。实际上，Kafka 选举一个 broker 作为 controller，这个 controller 通过 watch Zookeeper 检测所有的 broker failure，并负责为所有受影响的 parition 选举 leader，再将相应的 leader 调整命令发送至受影响的 broker，过程如下图所示。

![](http://img.lxw1234.com/0924-6.jpg)

这样做的好处是，可以批量的通知 leadership 的变化，从而使得选举过程成本更低，尤其对大量的 partition 而言。如果 controller 失败了，幸存的所有 broker 都会尝试在 Zookeeper 中创建 / controller->{this broker id}，如果创建成功（只可能有一个创建成功），则该 broker 会成为 controller，若创建不成功，则该 broker 会等待新 controller 的命令。

![](http://img.lxw1234.com/0924-7.jpg)

### Consumer group

（本节所有描述都是基于 consumer hight level API 而非 low level API）。  
每一个 consumer 实例都属于一个 consumer group，每一条消息只会被同一个 consumer group 里的一个 consumer 实例消费。（不同 consumer group 可以同时消费同一条消息）

![](http://img.lxw1234.com/0924-8.jpg)

很多传统的 message queue 都会在消息被消费完后将消息删除，一方面避免重复消费，另一方面可以保证 queue 的长度比较少，提高效率。而如上文所将，Kafka 并不删除 已消费的消息，为了实现传统 message queue 消息只被消费一次的语义，Kafka 保证保证同一个 consumer group 里只有一个 consumer 会消费一条消息。与传统 message queue 不同的是，Kafka 还允许不同 consumer group 同时消费同一条消息，这一特性可以为消息的多元化处理提供了支持。实际上，Kafka 的设计理念之一就是同时提供离线处理和实时处理。根据这一 特性，可以使用 Storm 这种实时流处理系统对消息进行实时在线处理，同时使用 Hadoop 这种批处理系统进行离线处理，还可以同时将数据实时备份到另一 个数据中心，只需要保证这三个操作所使用的 consumer 在不同的 consumer group 即可。下图展示了 Kafka 在 Linkedin 的一种简化部署。

![](http://img.lxw1234.com/0924-9.jpg)

为了更清晰展示 Kafka consumer group 的特性，笔者作了一项测试。创建一个 topic (名为 topic1)，创建一个属于 group1 的 consumer 实例，并创建三个属于 group2 的 consumer 实例，然后通过 producer 向 topic1 发送 key 分别为 1，2，3r 的消息。结果发现属于 group1 的 consumer 收到了所有的这三条消息，同时 group2 中的 3 个 consumer 分别收到了 key 为 1，2，3 的消息。如下图所示。

![](http://img.lxw1234.com/0924-10.jpg)

### Consumer Rebalance

（本节所讲述内容均基于 Kafka consumer high level API）  
Kafka 保证同一 consumer group 中只有一个 consumer 会消息某条消息，实际上，Kafka 保证的是稳定状态下每一个 consumer 实例只会消费某一个或多个特定 partition 的数据，而某个 partition 的数据只会被某一个特定的 consumer 实例所消费。这样设计的劣势是无法让同一个 consumer group 里的 consumer 均匀消费数据，优势是每个 consumer 不用都跟大量的 broker 通信，减少通信开销，同时也降低了分配难度，实现也 更简单。另外，因为同一个 partition 里的数据是有序的，这种设计可以保证每个 partition 里的数据也是有序被消费。  
如果某 consumer group 中 consumer 数量少于 partition 数量，则至少有一个 consumer 会消费多个 partition 的数据，如果 consumer 的数量与 partition 数量相同，则正好一个 consumer 消费一个 partition 的数据，而如果 consumer 的数量多于 partition 的数量时，会有部分 consumer 无法消费该 topic 下任何一条消息。  
如下例所示，如果 topic1 有 0，1，2 共三个 partition，当 group1 只有一个 consumer(名为 consumer1) 时，该 consumer 可消费这 3 个 partition 的所有数据。

![](http://img.lxw1234.com/0924-11.jpg)

增加一个 consumer(consumer2) 后，其中一个 consumer（consumer1）可消费 2 个 partition 的数据，另外一个 consumer(consumer2) 可消费另外一个 partition 的数据。

![](http://img.lxw1234.com/0924-12.jpg)

再增加一个 consumer(consumer3) 后，每个 consumer 可消费一个 partition 的数据。consumer1 消费 partition0，consumer2 消费 partition1，consumer3 消费 partition2

![](http://img.lxw1234.com/0924-13.jpg)

再增加一个 consumer（consumer4）后，其中 3 个 consumer 可分别消费一个 partition 的数据，另外一个 consumer（consumer4）不能消费 topic1 任何数据。

![](http://img.lxw1234.com/0924-14.jpg)

此时关闭 consumer1，剩下的 consumer 可分别消费一个 partition 的数据。

![](http://img.lxw1234.com/0924-15.jpg)

接着关闭 consumer2，剩下的 consumer3 可消费 2 个 partition，consumer4 可消费 1 个 partition。

![](http://img.lxw1234.com/0924-16.jpg)

再关闭 consumer3，剩下的 consumer4 可同时消费 topic1 的 3 个 partition。

![](http://img.lxw1234.com/0924-17.jpg)

consumer rebalance 算法如下：

*   Sort PT (all partitions in topic T)
*   Sort CG(all consumers in consumer group G)
*   Let i be the index position of Ci in CG and let N=size(PT)/size(CG)
*   Remove current entries owned by Ci from the partition owner registry
*   Assign partitions from i_N to (i+1)_N-1 to consumer Ci
*   Add newly assigned partitions to the partition owner registry

目前 consumer rebalance 的控制策略是由每一个 consumer 通过 Zookeeper 完成的。具体的控制方式如下：

*   Register itself in the consumer id registry under its group.
*   Register a watch on changes under the consumer id registry.
*   Register a watch on changes under the broker id registry.
*   If the consumer creates a message stream using a topic filter, it also registers a watch on changes under the broker topic registry.
*   Force itself to rebalance within in its consumer group. 在这种策略下，每一个 consumer 或者 broker 的增加或者减少都会触发 consumer rebalance。因为每个 consumer 只负责调整自己所消费的 partition，为了保证整个 consumer group 的一致性，所以当一个 consumer 触发了 rebalance 时，该 consumer group 内的其它所有 consumer 也应该同时触发 rebalance。

目前（2015-01-19）最新版（0.8.2）Kafka 采用的是上述方式。但该方式有不利的方面：

*   **Herd effect**  
    任何 broker 或者 consumer 的增减都会触发所有的 consumer 的 rebalance
*   **Split Brain**  
    每个 consumer 分别单独通过 Zookeeper 判断哪些 partition down 了，那么不同 consumer 从 Zookeeper“看” 到的 view 就可能不一样，这就会造成错误的 reblance 尝试。而且有可能所有的 consumer 都认为 rebalance 已经完成了，但实际上可能并非如此。

根据 Kafka 官方文档，Kafka 作者正在考虑在还未发布的 [0.9.x 版本中使用中心协调器 (coordinator)](https://cwiki.apache.org/confluence/display/KAFKA/Kafka+0.9+Consumer+Rewrite+Design)。 大体思想是选举出一个 broker 作为 coordinator，由它 watch Zookeeper，从而判断是否有 partition 或者 consumer 的增减，然后生成 rebalance 命令，并检查是否这些 rebalance 在所有相关的 consumer 中被执行成功，如果不成功则重试，若成功则认为此次 rebalance 成功（这个过程跟 replication controller 非常类似，所以我很奇怪为什么当初设计 replication controller 时没有使用类似方式来解决 consumer rebalance 的问题）。流程如下：

![](http://img.lxw1234.com/0924-18.jpg)

### 消息 Deliver guarantee

通过上文介绍，想必读者已经明天了 producer 和 consumer 是如何工作的，以及 Kafka 是如何做 replication 的，接下来要讨 论的是 Kafka 如何确保消息在 producer 和 consumer 之间传输。有这么几种可能的 delivery guarantee：

*   `At most once` 消息可能会丢，但绝不会重复传输
*   `At least one` 消息绝不会丢，但可能会重复传输
*   `Exactly once` 每条消息肯定会被传输一次且仅传输一次，很多时候这是用户所想要的。Kafka 的 delivery guarantee semantic 非常直接。当 producer 向 broker 发送消息时，一旦这条消息被 commit，因数 replication 的存在，它就不会丢。 但是如果 producer 发送数据给 broker 后，遇到的网络问题而造成通信中断，那 producer 就无法判断该条消息是否已经 commit。这一点 有点像向一个自动生成 primary key 的数据库表中插入数据。虽然 Kafka 无法确定网络故障期间发生了什么，但是 producer 可以生成一种类似于 primary key 的东西，发生故障时幂等性的 retry 多次，这样就做到了`Exactly one`。截止到目前 (Kafka 0.8.2 版本，2015-01-25)，这一 feature 还并未实现，有希望在 Kafka 未来的版本中实现。（所以目前默认情况下一条消息从 producer 和 broker 是确保了`At least once`，但可通过设置 producer 异步发送实现`At most once`）。  
    接下来讨论的是消息从 broker 到 consumer 的 delivery guarantee semantic。（仅针对 Kafka consumer high level API）。consumer 在从 broker 读取消息后，可以选择 commit，该操作会在 Zookeeper 中存下该 consumer 在该 partition 下读取的消息的 offset。该 consumer 下一次再读该 partition 时会从下一条开始读取。如未 commit，下一次读取 的开始位置会跟上一次 commit 之后的开始位置相同。当然可以将 consumer 设置为 autocommit，即 consumer 一旦读到数据立即自动 commit。如果只讨论这一读取消息的过程，那 Kafka 是确保了`Exactly once`。但实际上实际使用中 consumer 并非读取完数据就结束了，而是要进行进一步处理，而数据处理与 commit 的顺序在很大程度上决定了消息从 broker 和 consumer 的 delivery guarantee semantic。
*   读完消息先 commit 再处理消息。这种模式下，如果 consumer 在 commit 后还没来得及处理消息就 crash 了，下次重新开始工作后就无法读到刚刚已提交而未处理的消息，这就对应于`At most once`
*   读完消息先处理再 commit。这种模式下，如果处理完了消息在 commit 之前 consumer crash 了，下次重新开始工作时还会处理刚刚未 commit 的消息，实际上该消息已经被处理过了。这就对应于`At least once`。在很多情况使用场景下，消息都有一个 primary key，所以消息的处理往往具有幂等性，即多次处理这一条消息跟只处理一次是等效的，那就可以认为是`Exactly once`。 （人个感觉这种说法有些牵强，毕竟它不是 Kafka 本身提供的机制，而且 primary key 本身不保证操作的幂等性。而且实际上我们说 delivery guarantee semantic 是讨论被处理多少次，而非处理结果怎样，因为处理方式多种多样，我们的系统不应该把处理过程的特性—如是否幂等性，当成 Kafka 本身的 feature）
*   如果一定要做到`Exactly once`，就需要协调 offset 和实际操作的输出。精典的做法是引入两阶段提交。如 果能让 offset 和操作输入存在同一个地方，会更简洁和通用。这种方式可能更好，因为许多输出系统可能不支持两阶段提交。比如，consumer 拿到数 据后可能把数据放到 HDFS，如果把最新的 offset 和数据本身一起写到 HDFS，那就可以保证数据的输出和 offset 的更新要么都完成，要么都不完 成，间接实现`Exactly once`。（目前就 high level API 而言，offset 是存于 Zookeeper 中的，无法存于 HDFS，而 low level API 的 offset 是由自己去维护的，可以将之存于 HDFS 中）  
    总之，Kafka 默认保证`At least once`，并且允许通过设置 producer 异步提交来实现`At most once`。而`Exactly once`要求与目标存储系统协作，幸运的是 Kafka 提供的 offset 可以使用这种方式非常直接非常容易。

纸上得来终觉浅，绝知些事要躬行。笔者希望能亲自测一下 Kafka 的性能，而非从网上找一些测试数据。所以笔者曾在 0.8 发布前两个月做过详细的 Kafka0.8 性能测试，不过很可惜测试报告不慎丢失。所幸在网上找到了 Kafka 的创始人之一的 [Jay Kreps 的 bechmark](http://engineering.linkedin.com/kafka/benchmarking-apache-kafka-2-million-writes-second-three-cheap-machines)。以下描述皆基于该 benchmark。（该 benchmark 基于 Kafka0.8.1）

测试环境
----

该 benchmark 用到了六台机器，机器配置如下

*   Intel Xeon 2.5 GHz processor with six cores
*   Six 7200 RPM SATA drives
*   32GB of RAM
*   1Gb Ethernet 这 6 台机器其中 3 台用来搭建 Kafka broker 集群，另外 3 台用来安装 Zookeeper 及生成测试数据。6 个 drive 都直接以非 RAID 方式挂载。实际上 kafka 对机器的需求与 Hadoop 的类似。

producer 吞吐率
------------

该项测试只测 producer 的吞吐率，也就是数据只被持久化，没有 consumer 读数据。

### 1 个 producer 线程，无 replication

在这一测试中，创建了一个包含 6 个 partition 且没有 replication 的 topic。然后通过一个线程尽可能快的生成 50 million 条比较短（payload100 字节长）的消息。测试结果是 **_821,557 records/second_**（**_78.3MB/second_**）。  
之所以使用短消息，是因为对于消息系统来说这种使用场景更难。因为如果使用 MB/second 来表征吞吐率，那发送长消息无疑能使得测试结果更好。  
整个测试中，都是用每秒钟 delivery 的消息的数量乘以 payload 的长度来计算 MB/second 的，没有把消息的元信息算在内，所以实际的网络 使用量会比这个大。对于本测试来说，每次还需传输额外的 22 个字节，包括一个可选的 key，消息长度描述，CRC 等。另外，还包含一些请求相关的 overhead，比如 topic，partition，acknowledgement 等。这就导致我们比较难判断是否已经达到网卡极限，但是把这些 overhead 都算在吞吐率里面应该更合理一些。因此，我们已经基本达到了网卡的极限。  
初步观察此结果会认为它比人们所预期的要高很多，尤其当考虑到 Kafka 要把数据持久化到磁盘当中。实际上，如果使用随机访问数据系统，比如 RDBMS， 或者 key-velue store，可预期的最高访问频率大概是 5000 到 50000 个请求每秒，这和一个好的 RPC 层所能接受的远程请求量差不多。而该测试中远超于此的原因有 两个。

*   Kafka 确保写磁盘的过程是线性磁盘 I/O，测试中使用的 6 块廉价磁盘线性 I/O 的最大吞吐量是 822MB/second，这已经远大于 1Gb 网卡所能带来的吞吐量了。许多消息系统把数据持久化到磁盘当成是一个开销很大的事情，这是因为他们对磁盘的操作都不是线性 I/O。
*   在每一个阶段，Kafka 都尽量使用批量处理。如果想了解批处理在 I/O 操作中的重要性，可以参考 David Patterson 的”[Latency Lags Bandwidth](http://www.ll.mit.edu/HPEC/agendas/proc04/invited/patterson_keynote.pdf)“

### 1 个 producer 线程，3 个异步 replication

该项测试与上一测试基本一样，唯一的区别是每个 partition 有 3 个 replica（所以网络传输的和写入磁盘的总的数据量增加了 3 倍）。每一 个 broker 即要写作为 leader 的 partition，也要读（从 leader 读数据）写（将数据写到磁盘）作为 follower 的 partition。测试结果为 **_786,980 records/second_**（**_75.1MB/second_**）。  
该项测试中 replication 是异步的，也就是说 broker 收到数据并写入本地磁盘后就 acknowledge producer，而不必等所有 replica 都完成 replication。也就是说，如果 leader crash 了，可能会丢掉一些最新的还未备份的数据。但这也会让 message acknowledgement 延迟更少，实时性更好。  
这项测试说明，replication 可以很快。整个集群的写能力可能会由于 3 倍的 replication 而只有原来的三分之一，但是对于每一个 producer 来说吞吐率依然足够好。

### 1 个 producer 线程，3 个同步 replication

该项测试与上一测试的唯一区别是 replication 是同步的，每条消息只有在被`in sync`集合里的所有 replica 都复制过去后才会被置为 committed（此时 broker 会向 producer 发送 acknowledgement）。在这种模式下，Kafka 可以保证即使 leader crash 了，也不会有数据丢失。测试结果为 **_421,823 records/second_**（**_40.2MB/second_**）。  
Kafka 同步复制与异步复制并没有本质的不同。leader 会始终 track follower replica 从而监控它们是否还 alive，只有所有`in sync`集合里的 replica 都 acknowledge 的消息才可能被 consumer 所消费。而对 follower 的等待影响了吞吐率。可以通过增大 batch size 来改善这种情况，但为了避免特定的优化而影响测试结果的可比性，本次测试并没有做这种调整。

### 3 个 producer,3 个异步 replication

该测试相当于把上文中的 1 个 producer, 复制到了 3 台不同的机器上（在 1 台机器上跑多个实例对吞吐率的增加不会有太大帮忙，因为网卡已经基本饱和了），这 3 个 producer 同时发送数据。整个集群的吞吐率为 **_2,024,032 records/second_**（**_193,0MB/second_**）。

Producer Throughput Vs. Stored Data
-----------------------------------

消息系统的一个潜在的危险是当数据能都存于内存时性能很好，但当数据量太大无法完全存于内存中时（然后很多消息系统都会删除已经被消费的数据，但当 消费速度比生产速度慢时，仍会造成数据的堆积），数据会被转移到磁盘，从而使得吞吐率下降，这又反过来造成系统无法及时接收数据。这样就非常糟糕，而实际 上很多情景下使用 queue 的目的就是解决数据消费速度和生产速度不一致的问题。  
但 Kafka 不存在这一问题，因为 Kafka 始终以 O（1）的时间复杂度将数据持久化到磁盘，所以其吞吐率不受磁盘上所存储的数据量的影响。为了验证这一特性，做了一个长时间的大数据量的测试，下图是吞吐率与数据量大小的关系图。

![](http://img.lxw1234.com/0924-19.jpg)

上图中有一些 variance 的存在，并可以明显看到，吞吐率并不受磁盘上所存数据量大小的影响。实际上从上图可以看到，当磁盘数据量达到 1TB 时，吞吐率和磁盘数据只有几百 MB 时没有明显区别。  
这个 variance 是由 Linux I/O 管理造成的，它会把数据缓存起来再批量 flush。上图的测试结果是在生产环境中对 Kafka 集群做了些 tuning 后得到的，这些 tuning 方法可参考[这里](http://kafka.apache.org/documentation.html#hwandos)。

consumer 吞吐率
------------

需要注意的是，replication factor 并不会影响 consumer 的吞吐率测试，因为 consumer 只会从每个 partition 的 leader 读数据，而与 replicaiton factor 无关。同样，consumer 吞吐率也与同步复制还是异步复制无关。

### 1 个 consumer

该测试从有 6 个 partition，3 个 replication 的 topic 消费 50 million 的消息。测试结果为 **_940,521 records/second_**（**_89.7MB/second_**）。  
可以看到，Kafkar 的 consumer 是非常高效的。它直接从 broker 的文件系统里读取文件块。Kafka 使用 [sendfile API](http://www.ibm.com/developerworks/library/j-zerocopy/) 来直接通过操作系统直接传输，而不用把数据拷贝到用户空间。该项测试实际上从 log 的起始处开始读数据，所以它做了真实的 I/O。在生产环境下，consumer 可以直接读取 producer 刚刚写下的数据（它可能还在缓存中）。实际上，如果在生产环境下跑 [I/O stat](http://en.wikipedia.org/wiki/Iostat)，你可以看到基本上没有物理 “读”。也就是说生产环境下 consumer 的吞吐率会比该项测试中的要高。

### 3 个 consumer

将上面的 consumer 复制到 3 台不同的机器上，并且并行运行它们（从同一个 topic 上消费数据）。测试结果为 **_2,615,968 records/second_**（**_249.5MB/second_**）。  
正如所预期的那样，consumer 的吞吐率几乎线性增涨。

Producer and Consumer
---------------------

上面的测试只是把 producer 和 consumer 分开测试，而该项测试同时运行 producer 和 consumer，这更接近使用场景。实际上目前的 replication 系统中 follower 就相当于 consumer 在工作。  
该项测试，在具有 6 个 partition 和 3 个 replica 的 topic 上同时使用 1 个 producer 和 1 个 consumer，并且使用异步复制。测试结果为 **_795,064 records/second_**（**_75.8MB/second_**）。  
可以看到，该项测试结果与单独测试 1 个 producer 时的结果几乎一致。所以说 consumer 非常轻量级。

消息长度对吞吐率的影响
-----------

上面的所有测试都基于短消息（payload 100 字节），而正如上文所说，短消息对 Kafka 来说是更难处理的使用方式，可以预期，随着消息长度的增大，records/second 会减小，但 MB/second 会有所提高。下图是 records/second 与消息长度的关系图。

![](http://img.lxw1234.com/0924-20.jpg)

正如我们所预期的那样，随着消息长度的增加，每秒钟所能发送的消息的数量逐渐减小。但是如果看每秒钟发送的消息的总大小，它会随着消息长度的增加而增加，如下图所示。

![](http://img.lxw1234.com/0924-21.jpg)

从上图可以看出，当消息长度为 10 字节时，因为要频繁入队，花了太多时间获取锁，CPU 成了瓶颈，并不能充分利用带宽。但从 100 字节开始，我们可以看到带宽的使用逐渐趋于饱和（虽然 MB/second 还是会随着消息长度的增加而增加，但增加的幅度也越来越小）。

端到端的 Latency
------------

上文中讨论了吞吐率，那消息传输的 latency 如何呢？也就是说消息从 producer 到 consumer 需要多少时间呢？该项测试创建 1 个 producer 和 1 个 consumer 并反复计时。结果是，**_2 ms (median), 3ms (99th percentile, 14ms (99.9th percentile)_**。  
（这里并没有说明 topic 有多少个 partition，也没有说明有多少个 replica，replication 是同步还是异步。实际上这会极大影响 producer 发送的消息被 commit 的 latency，而只有 committed 的消息才能被 consumer 所消费，所以它会最终影响端到端的 latency）

重现该 benchmark
-------------

如果读者想要在自己的机器上重现本次 benchmark 测试，可以参考[本次测试的配置和所使用的命令](https://gist.github.com/jkreps/c7ddb4041ef62a900e6c)。  
实际上 Kafka Distribution 提供了 producer 性能测试工具，可通过`bin/kafka-producer-perf-test.sh`脚本来启动。所使用的命令如下

```
Producer
Setup
bin/kafka-topics.sh --zookeeper esv4-hcl197.grid.linkedin.com:2181 --create --topic test-rep-one --partitions 6 --replication-factor 1
bin/kafka-topics.sh --zookeeper esv4-hcl197.grid.linkedin.com:2181 --create --topic test --partitions 6 --replication-factor 3
 
Single thread, no replication
 
bin/kafka-run-class.sh org.apache.kafka.clients.tools.ProducerPerformance test7 50000000 100 -1 acks=1 bootstrap.servers=esv4-hcl198.grid.linkedin.com:9092 buffer.memory=67108864 batch.size=8196
 
Single-thread, async 3x replication
 
bin/kafktopics.sh --zookeeper esv4-hcl197.grid.linkedin.com:2181 --create --topic test --partitions 6 --replication-factor 3
bin/kafka-run-class.sh org.apache.kafka.clients.tools.ProducerPerformance test6 50000000 100 -1 acks=1 bootstrap.servers=esv4-hcl198.grid.linkedin.com:9092 buffer.memory=67108864 batch.size=8196
 
Single-thread, sync 3x replication
 
bin/kafka-run-class.sh org.apache.kafka.clients.tools.ProducerPerformance test 50000000 100 -1 acks=-1 bootstrap.servers=esv4-hcl198.grid.linkedin.com:9092 buffer.memory=67108864 batch.size=64000
 
Three Producers, 3x async replication
bin/kafka-run-class.sh org.apache.kafka.clients.tools.ProducerPerformance test 50000000 100 -1 acks=1 bootstrap.servers=esv4-hcl198.grid.linkedin.com:9092 buffer.memory=67108864 batch.size=8196
 
Throughput Versus Stored Data
 
bin/kafka-run-class.sh org.apache.kafka.clients.tools.ProducerPerformance test 50000000000 100 -1 acks=1 bootstrap.servers=esv4-hcl198.grid.linkedin.com:9092 buffer.memory=67108864 batch.size=8196
 
Effect of message size
 
for i in 10 100 1000 10000 100000;
do
echo ""
echo $i
bin/kafka-run-class.sh org.apache.kafka.clients.tools.ProducerPerformance test $((1000*1024*1024/$i)) $i -1 acks=1 bootstrap.servers=esv4-hcl198.grid.linkedin.com:9092 buffer.memory=67108864 batch.size=128000
done;
 
Consumer
Consumer throughput
 
bin/kafka-consumer-perf-test.sh --zookeeper esv4-hcl197.grid.linkedin.com:2181 --messages 50000000 --topic test --threads 1
 
3 Consumers
 
On three servers, run:
bin/kafka-consumer-perf-test.sh --zookeeper esv4-hcl197.grid.linkedin.com:2181 --messages 50000000 --topic test --threads 1
 
End-to-end Latency
 
bin/kafka-run-class.sh kafka.tools.TestEndToEndLatency esv4-hcl198.grid.linkedin.com:9092 esv4-hcl197.grid.linkedin.com:2181 test 5000
 
Producer and consumer
 
bin/kafka-run-class.sh org.apache.kafka.clients.tools.ProducerPerformance test 50000000 100 -1 acks=1 bootstrap.servers=esv4-hcl198.grid.linkedin.com:9092 buffer.memory=67108864 batch.size=8196
 
bin/kafka-consumer-perf-test.sh --zookeeper esv4-hcl197.grid.linkedin.com:2181 --messages 50000000 --topic test --threads 1 

```

broker 配置如下:

```
############################# Server Basics #############################
 
# The id of the broker. This must be set to a unique integer for each broker.
broker.id=0
 
############################# Socket Server Settings #############################
 
# The port the socket server listens on
port=9092
 
# Hostname the broker will bind to and advertise to producers and consumers.
# If not set, the server will bind to all interfaces and advertise the value returned from
# from java.net.InetAddress.getCanonicalHostName().
#host.name=localhost
 
# The number of threads handling network requests
num.network.threads=4
 
# The number of threads doing disk I/O
num.io.threads=8
 
# The send buffer (SO_SNDBUF) used by the socket server
socket.send.buffer.bytes=1048576
 
# The receive buffer (SO_RCVBUF) used by the socket server
socket.receive.buffer.bytes=1048576
 
# The maximum size of a request that the socket server will accept (protection against OOM)
socket.request.max.bytes=104857600
 
 
############################# Log Basics #############################
 
# The directory under which to store log files
log.dirs=/grid/a/dfs-data/kafka-logs,/grid/b/dfs-data/kafka-logs,/grid/c/dfs-data/kafka-logs,/grid/d/dfs-data/kafka-logs,/grid/e/dfs-data/kafka-logs,/grid/f/dfs-data/kafka-logs
 
# The number of logical partitions per topic per server. More partitions allow greater parallelism
# for consumption, but also mean more files.
num.partitions=8
 
############################# Log Flush Policy #############################
 
# The following configurations control the flush of data to disk. This is the most
# important performance knob in kafka.
# There are a few important trade-offs here:
#    1. Durability: Unflushed data is at greater risk of loss in the event of a crash.
#    2. Latency: Data is not made available to consumers until it is flushed (which adds latency).
#    3. Throughput: The flush is generally the most expensive operation. 
# The settings below allow one to configure the flush policy to flush data after a period of time or
# every N messages (or both). This can be done globally and overridden on a per-topic basis.
 
# Per-topic overrides for log.flush.interval.ms
#log.flush.intervals.ms.per.topic=topic1:1000, topic2:3000
 
############################# Log Retention Policy #############################
 
# The following configurations control the disposal of log segments. The policy can
# be set to delete segments after a period of time, or after a given size has accumulated.
# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
# from the end of the log.
 
# The minimum age of a log file to be eligible for deletion
log.retention.hours=168
 
# A size-based retention policy for logs. Segments are pruned from the log as long as the remaining
# segments don't drop below log.retention.bytes.
#log.retention.bytes=1073741824
 
# The maximum size of a log segment file. When this size is reached a new log segment will be created.
log.segment.bytes=536870912
 
# The interval at which log segments are checked to see if they can be deleted according 
# to the retention policies
log.cleanup.interval.mins=1
 
############################# Zookeeper #############################
 
# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
zookeeper.connect=esv4-hcl197.grid.linkedin.com:2181
 
# Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=1000000
 
# metrics reporter properties
kafka.metrics.polling.interval.secs=5
kafka.metrics.reporters=kafka.metrics.KafkaCSVMetricsReporter
kafka.csv.metrics.dir=/tmp/kafka_metrics
# Disable csv reporting by default.
kafka.csv.metrics.reporter.enabled=false
 
replica.lag.max.messages=10000000
 

```

*   [使用消息队列的 10 个理由](http://www.oschina.net/translate/top-10-uses-for-message-queue)
*   [Apache Kafka](http://kafka.apache.org/)
*   [Efficient data transfer through zero copy](http://www.ibm.com/developerworks/library/j-zerocopy/)
*   [Benchmarking Apache Kafka: 2 Million Writes Per Second (On Three Cheap Machines)](http://engineering.linkedin.com/kafka/benchmarking-apache-kafka-2-million-writes-second-three-cheap-machines)
*   [Kafka 0.8 Producer Performance](http://liveramp.com/blog/kafka-0-8-producer-performance-2/)

本文转载自：http://www.jasongj.com/2015/01/02/Kafka%E6%B7%B1%E5%BA%A6%E8%A7%A3%E6%9E%90/

如果觉得本博客对您有帮助，请 [赞助作者](http://lxw1234.com/pay-blog) 。

转载请注明：[lxw 的大数据田地](http://lxw1234.com/) » [Kafka 架构和原理深度剖析](http://lxw1234.com/archives/2015/09/504.htm)