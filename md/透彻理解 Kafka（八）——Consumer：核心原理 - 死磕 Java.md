> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/8338957597)

> 了解了 Kafka 的生产者基本原理后，本章，我将讲解 Consumer 消费者的基本原理，通过本章的内容，读者可以理解一条消息从获取到被消费的整个流程。一、基本使用 Kafka 的消费者

了解了 Kafka 的生产者基本原理后，本章，我将讲解 Consumer 消费者的基本原理，通过本章的内容，读者可以理解一条消息从获取到被消费的整个流程。

一、基本使用
------

Kafka 的消费者是 **非线程安全** 的。我们先来看下 Consumer 的基本使用，一个正常的 Consumer 消费消息的逻辑需要具备以下几个步骤 ：

1.  配置消费者客户端参数，然后创建相应的消费者实例；
2.  订阅主题；
3.  拉取消息并消费；
4.  提交消费位移；
5.  关闭消费者实例。

来看一个示例，更好的体会下：

```
    public class ConsumerDemo {
        public static void main(String[] args) throws Exception {
            String topicName = “test - topic”;
            String groupId = “test - group”;
    
            Properties props = new Properties();
            props.put(“bootstrap.servers”, “localhost:9092”);
            props.put(“group.id”, groupId);
            
            props.put(“enable.auto.commit”, “true”);
            props.put(“auto.commit.ineterval.ms”, “1000”);
            
            props.put(“auto.offset.reset”, “earliest”);
            props.put(“key.deserializer”, “org.apache.kafka.common.serialization.StringDeserializer”);
            props.put(“value.deserializer”, “org.apache.kafka.common.serialization.SttringDeserializer”);
    
            KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(props);
            
            consumer.subscribe(Arrays.asList(topicName));
    
            try {
                while (true) {
                    
                    ConsumerRecords<String, String> records = consumer.poll(1000); 
                    for (ConsumerRecord<String, String> record : records) {
                        System.out.println(record.offset() + “, ” + record.key() + “, ”+record.value());
                    }
                }
            } catch (Exception e) {
            }
        }
    }


```

二、整体架构
------

每个 Consumer 都要属于一个`consumer.group`消费者组，Topic 的一个分区只会分配给一个消费组下的一个 Consumer 来处理，每个 Consumer 可能会分配多个分区，也有可能某个 Consumer 没有分配到任何分区：

![](http://image.skjava.com/article/series/kafka/202307312119597451.png)

注意：消费者组是一个逻辑上的概念，它将旗下的消费者归为一类，每一个消费者只隶属于一个消费者组。

三、消费位移
------

每个 Consumer 的内存里都保存着分区的消费 offset，包括：上一次提交的 offset，当前消费到的 offset。

Consumer 工作线程（也就是我们调用`poll`方法的线程）会定期提交 offset。在老版本 Kafka 中，位移是提交到 Zookeeper，但是高并发场景下这种设计是有问题的，Zookeeper 是做分布式协调的，属于轻量级的元数据存储，不适合做高并发读写，作为数据存储。

所以之后 Kafka 版本中，消费者不再将 offset 提交到 Zookeeper，而是提交到 Broker 的一个内部 Topic—— **__consumer_offsets** ，提交时 Key 是`group.id + topic + 分区号`，Value 就是当前 offset 的值。每隔一段时间，Kafka Broker 内部会对这个 Topic 进行 Compact，也就是只保留最新的数据即可。

> **__consumer_offsets** 这个内部 Topic 默认有 50 个分区，这样如果 Kafka 集群很大，比如有 50 台机器，就可以用 50 台机器来抗 offset 提交的请求压力，性能上要好很多。

### 3.1 自动提交

默认的消费位移提交方式是 **自动提交** ，可以通过 Consumer 的客户端参数`enable.auto.commit` 配置。注意，自动提交是默认每隔 5 秒进行一次提交，不是指每消费一条消息就提交一次，可以通过参数`auto.commit.interval.ms`配置间隔时间。

Consumer 每次从 Broker 拉取消息之前，都会检查下是否可以进行位移提交，如果可以，就会提交上一次轮询的位移：

![](http://image.skjava.com/article/series/kafka/202307312120037142.png)

### 3.2 手动提交

手动提交可以细分为 **同步提交** 和 **异步提交** ，对应于 Kafka Consumer 中的`commitSync()`和`commitAsync()`两种方法。

**同步提交**

`commitSync()`方法会根据 poll 方法拉取的最新位移来进行提交，只能提交当前批次对应的 position 值，只要没有发生不可恢复的错误（ Unrecoverable Error），它就会阻塞消费者线程直至位移提交完成。

比如，下面的代码按分区粒度同步提交消费位移 ：

```
    ConsumerRecords<String, String> records = consumer.poll(1000);
    for (TopicPartition partition : records.partitions()) {
        List<ConsumerRecord<String, String>> partitionRecords = records.records(partition);
        for (ConsumerRecord<String, String> record : partitionRecords) {
            
        }
        long lastConsumedOffset = partitionRecords.get(partitionRecords.size() - 1).offset();
        consumer.commitSync(Collections.singletonMap(partition, new  
              OffsetAndMetadata(lastConsumedOffset + 1)));
        
    }


```

**异步提交**

异步提交的方式—— `commitAsync()`在执行的时候消费者线程不会被阻塞，可能在提交消费位移的结果还未返回之前就开始了新一次的拉取操作异步提交的方式—— `commitAsync()`在执行的时候消费者线程不会被阻塞，可能在提交消费位移的结果还未返回之前就开始了新一次的拉取操作 。

```
    while (isRunning.get()) { 
        ConsumerRecords<String, String> records = consumer.poll(1000);
        for (ConsumerRecord<String, String> record : records) {
            
        }
        consumer.commitAsync(new OffsetCommitCallback() {
            @Override
            public void onComplete(Map<TopicPartition, OffsetAndMetadata> offsets,Exception  
                                   exception) {
                if (exception == null) {
                    System.out.println(offsets) ;
                } else {
                    log.error("fail to commit offsets {}", offsets, exception);
                }
            }  
        });
    }


```

### 3.3 seek

消费者有一个`seek`方法，该方法为我们提供了从特定位置读取消息的能力，我们可以通过这个方法来向前跳过若干消息，也可以通过这个方法来向后回溯若干消息，这样为消息的消费提供了很大的灵活性。

`seek`方法也为我们提供了将消费位移保存在外部存储介质中的能力，还可以配合再均衡监听器来提供更加精准的消费能力。

```
    consumer.subscribe(Arrays.asList(topic));
    
    for (TopicPartition tp: assignment) {
        long offset = getOffsetFromDB(tp); 
        consumer.seek(tp, offset);
    }


```

```
    while (true) {
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
        for (TopicPartition partition : records.partitions()) {
            List<ConsumerRecord<String, String>> partitionRecords = records.records(partition);
            for (ConsumerRecord<String, String> record : partitionRecords) {
                
            )
            long lastConsumedOffset = partitionRecords
                   .get(partitionRecords.size() - 1).offset();
            
            storeOffsetToDB(partition, lastConsumedOffset + 1);
        }
    }


```

四、消费问题
------

由于 Kafka Consumer 的位移提交机制，可能出现 **重复消费** 和 **消息丢失** 的情况。

### 4.1 重复消费

假设 Consumer 刚 poll 到消息，并且都处理完了，此时还没来得及提交 offset，Consumer 就宕机了。Consumer 再次重启会重新消费到这一批消息，再次处理一遍，就发生了消息的重复消费。

比如，Poll 到了一批数据：`offset = 65510~65532`，Consumer 很快处理完了，并且写入了数据库，结果还没来得及提交 offset 就宕机了，上一次提交的`offset = 65509`，Consumer 重启后，它会再次拉取`offset = 65510~65532`的消息，然后重复处理一遍。

### 4.2 消息丢失

假设 Consumer 刚 poll 到消息，然后还没来得及处理，刚好到了触发自动提交的时间点，此时如果 Consumer 宕机然后再次重启，消息就丢失了。

比如，Poll 到了一批数据：`offset = 65510~65532`，然后触发自动提交 offset，此时`offset = 65532`已经提交给了 Kafka Broker，接着当 Consumer 准备对这批数据进行处理时就直接宕机了，下次重启的时候，会从`offset = 65533`这个位置开始消费，之前的一批数据就丢失了。

五、核心参数
------

**fetch.min.bytes**

该参数用来配置 Consumer 在一次拉取请求（调用 poll 方法）中能从 Kafka 中拉取的最小数据量，默认值为 1 个字节。Kafka 在收到 Consumer 的拉取请求时，如果返回给 Consumer 的数据量小于这个参数所配置的值，那么它就需要进行等待，直到数据量满足这个参数的配置大小。

> 可以适当调大这个参数的值以提高一定的吞吐量，不过也会造成额外的延迟，对于延迟敏感的应用可能就不可取了。

**fetch.max.wait.ms**

这个参数也和`fetch.min.bytes`参数有关，如果 Kafka 仅仅参考`fetch.min.bytes`参数的要求，那么有可能会一直阻塞等待而无法发送响应给 Consumer，显然这是不合理的。`fetch.max.wait.ms`参数用于指定 Kafka 的等待时间，默认值为 500 ( ms ）。如果 Kafka 中没有足够多的消息而满足不了`fetch.min.bytes`参数的要求，那么最终会等待 500ms 。

**max.poll.records**

这个参数用来配置 Consumer 在一次拉取请求中拉取的最大消息数，默认值为 500 条。

**max.poll.interval.ms**

如果 Consumer 的两次 Poll 操作间隔超过了这个时间，那么就会认为这个 Consume 处理能力太弱了，会被踢出消费组，将它原来的分区分配给其它消费者。

**request.timeout.ms**

这个参数用来配置 Consumer 等待请求响应的最长时间，默认值为 30000 ( ms ）。

**auto.offset.reset**

这个参数的意思是：如果 Consumer 重启，发现要消费的 offset 不在分区的最新分段日志里，那么从哪里开始消费。有三种策略可以选择：earliest、latest、none，配置为其余值会报出异常。

**enable.auto.commit**

是否开启自动提交消费位移的功能，默认开启。

**auto.commit.interval.ms**

当`enbale.auto.commit`参数设置为 true 时才生效 ，表示开启自动提交消费位移功能时自动提交消  
费位移的时间间隔。

**heartbeat.interval.ms**

Consumer 心跳时间，Broker 必须和 Consumer 保持心跳才能知道 Consumer 是否故障了，如果发生故障就会让其它的 Consumer 进行 Rebalance 操作。

**session.timeout.ms**

Kafka 多长时间感知不到一个 Consumer 的心跳就认为它故障了，默认是 10 秒。

**connection.max.idle.ms**

Consumer 跟 Broker 的 Socket 连接如果空闲时间超过了该参数值，就会自动回收连接，当下次再消费时才重新建立 Socket 连接。生产建议设置为 -1，不要去回收。