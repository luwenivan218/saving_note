> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/4046538266)

> 了解了 Kafka 的整体架构之后，我将站在客户端的角度对 Kafka 的生产者（Producer）和消费者（Consumer）进行讲解。本章，我们先从 Producer 开始，我将讲解 P

转

 2023-07-31  阅读 (170)  点赞 (0)

原文地址：https://www.tpvlog.com/article/279

了解了 Kafka 的整体架构之后，我将站在客户端的角度对 Kafka 的生产者（Producer）和消费者（Consumer）进行讲解。本章，我们先从 Producer 开始，我将讲解 Producer 的基本原理，通过本章的内容，读者可以理解一条消息从生产到发送的整个流程。

一、基本使用
------

我们先来看下 Producer 的基本使用，一个正常的消息发送逻辑需要具备以下几个步骤 ：

1.  配置生产者客户端参数，创建相应的生产者实例；
2.  创建待发送的消息；
3.  发送消息；
4.  关闭生产者实例。

来看一个示例，更好的体会下：

```
    public class ProducerDemo {
    
        public static void main(String[] args) throws Exception {
            Properties props = new Properties();
    
            
            props.put("bootstrap.servers", "ressmix01:9092,ressmix02:9092,ressmix03:9092");  
            
            props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
            
            props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
            
            props.put("acks", "-1");
            props.put("retries", 3);
            
            props.put("batch.size", 323840);
            props.put("linger.ms", 10);
            
            props.put("buffer.memory", 33554432);
            
            props.put("max.block.ms", 3000);
    
            
            KafkaProducer<String, String> producer = new KafkaProducer<String, String>(props);
            
            ProducerRecord<String, String> record = new ProducerRecord<>(
                    "test-topic", "test-key", "test-value");
    
            
            producer.send(record, new Callback() {
                @Override
                public void onCompletion(RecordMetadata metadata, Exception exception) {
                    if(exception == null) {
                        
                        System.out.println("消息发送成功");  
                    } else {
                        
                    }
                }
    
            });
    
            Thread.sleep(10 * 1000); 
    
            
            
    
            
            producer.close();
        }    
    }


```

> 对于 Kafka Producer 而言，它是 **线程安全** 的，我们可以在多线程的环境中复用它。

二、核心原理
------

Producer 的整体架构如下，它由两个线程协调运行： **主线程** 和 **Sender 线程** 。主线程主要负责控制消息的创建和封装，Sender 线程负责实际的消息发送。

![](http://image.skjava.com/article/series/kafka/202307312119436711.png)

### 2.1 主线程

我来讲解下上述消息发送的流程，首先来看主线程的操作：

1.  首先，Producer 发送消息时，必须把消息封装成一个`ProducerRecord`对象，里面包含了要发送的 Topic、发送到哪个分区、分区 Key、消息内容、时间戳等等；
2.  接着，如果有定义拦截器，消息会先被 **拦截器** 处理一下；
3.  然后`ProducerRecord`对象会被交给 **序列化器** ，转化成自定义的协议格式；
4.  接着，数据会被交给 **Partitioner 分区器** ，它会选择一个合适的分区（默认采用轮询策略，如果指定了分区 Key，则根据 Key 进行 hash 路由）；
5.  最后，消息会被发送到 Producer 内部的一块缓冲区 **RecordAccumulator** 里。

RecordAccumulator，也叫做消息累加器，它的内部为每个分区都准备了一个双端队列（ Deque ），消息会被按批次封装成 **ProducerBatch** 对象并入队，ProducerBatch 中可以包含多个 ProducerRecord。

所以，如果 Producer 发送消息的速度太快，就会导致 RecordAccumulator 缓存空间不足，此时主线程就会阻塞，直到超时抛出异常，可以通过参数`max.block.ms` 配置超时时间，默认值 60000，即 60 秒。

> RecordAccumulator 主要用来缓存消息，以便 Sender 线程可以批量发送消息，进而减少网络传输的资源消耗，提升性能 。RecordAccumulator 缓存的大小可以通过 Producer 客户端参数`buffer.memory`配置，默认 32MB。

### 2.2 Sender 线程

Producer 内部还有一个 Sender 线程，负责消息的真正发送：

1.  首先，Sender 线程从 RecordAccumulator 缓冲区获取消息；
2.  然后，会将消息转变成`<Node, List< ProducerBatch>`的形式，其中 Node 表示 Kafka 集群的 Broker 节点 ；
3.  接着，再进行一次转换，封装成`<Node,Request>`的形式；
4.  最后，Sender 线程在将消息发往 Broker 之前，还会将消息保存到 InFlightRequests 中， InFlightRequests 的具体形式为 `Map<Nodeld, Deque<Request>>`，它的主要作用是缓存已经发出去但还没有收到响应的请求。InFlightRequests 中还未确认的请求越多，则说明该 Node 的负载越大 。

> 我们可以限制 Producer 与 Broker 之间连接的最多缓存请求数，通过参数 `max.in.flight.requests.per.connection`设置，默认值为 5，即每个连接最多只能缓存 5 个未响应的请求，超过就不能再向这个连接发送更多的请求了。如果该参数值设置为 1，就表示 Producer 同一时间只能发送一条消息。

三、核心参数
------

**1. ack**

指定分区中必须要有多少个副本收到这条消息，生产者才会认为这条消息写入成功。

*   acks = 1：默认值，即 Leader 成功写入即可；
*   acks = 0：不管写入 Broker 的消息到底成功没有，发出去就认为成功了，适合实时数据流分析的业务和场景；
*   acks = all/-1：等待 ISR 中的所有副本都成功 。

**2. max.request.size**

设置生产者能发送的消息的最大值，默认值为 1048576B，即 1MB。一般情况下，生产环境会把这个值设置得大一些，比如 10MB。

**3. retries**

生产者发送消息失败时的重次数，默认值为 0，即不重试。如果重试达到指定次数还失败，就抛出异常。一般重试主要用于解决网络抖动和 Partition Leader 宕机重新选择的问题。

**4. retry.backoff.ms**

重试间隔时间，默认值 100，即 100ms。

**5. request.timeout.ms**

设置 Producer 等待请求响应的最长时间，默认值为 30000，即 30 秒，超过这个时间收不到响应，就会抛出 TimeoutException 异常。注意，这个参数值需要比 Broker 端的`replica.lag.time.max.ms`参数值大，这样可以减少因 Producer 重试而引起的消息重复的概率。

**6. buffer.memory**

Producer 客户端中用于缓存消息的缓冲区大小，默认值为 33554432 (32MB) 。

**7. max.block.ms**

当生产者的发送缓冲区满了，主线程就会阻塞，默认阻塞 60 秒。

**8. batch.size**

消息会被按批次封装成 **ProducerBatch** 对象，这个参数用来设置 batch 的大小，默认值是 16384，即 16kb。如果 batch 设置太小，会导致频繁网络请求，吞吐量下降；如果 batch 设置太大，会导致消息被缓存很久才能发送出去，导致延时增加。一般如果要在生产环境改变该值，则需要进行压测确定。

**9. linger.ms**

这个参数用来设置消息在缓存区的最大逗留时间，默认值为 0，即消息不过缓冲区立即被发送出去。一般生产上会设置该值为 10-100ms，也就是说，消息首先进入缓冲区并按批次处理，如果 100ms 内 batch 满了（默认 16kb），那么消息会被发送出去，如果 100ms 内 batch 没满，那么消息也会被发送出去。这样就可以防止消息的发送延迟过长，避免给缓冲区内存造成过大的压力。