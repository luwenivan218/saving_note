> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [lxw1234.com](http://lxw1234.com/archives/2015/10/538.htm)

> 关键字：Kafka 分区、Partition Kafka 中可以将 Topic 从物理上划分成一个或多个分区（Partition），每个分区在物理上对应一个文件夹，以”topicName_partitionIndex”的命名方式命名，该文件夹下存储这个分区的所有消息 (.log) 和索引文件(.index)，这使得 Kafka 的吞吐率可以水平扩展。

关键字：Kafka 分区、Partition

Kafka 中可以将 Topic 从物理上划分成一个或多个分区（Partition），每个分区在物理上对应一个文件夹，以”topicName_partitionIndex”的命名方式命名，该文件夹下存储这个分区的所有消息 (.log) 和索引文件(.index)，这使得 Kafka 的吞吐率可以水平扩展。

生产者在生产数据的时候，可以为每条消息指定 Key，这样消息被发送到 broker 时，会根据分区规则选择被存储到哪一个分区中，如果分区规则设置的合理，那么所有的消息将会被均匀的分布到不同的分区中，这样就实现了负载均衡和水平扩展。另外，在消费者端，同一个消费组可以多线程并发的从多个分区中同时消费数据（后续将介绍这块）。

上面所说的分区规则，是实现了 kafka.producer.Partitioner 接口的类，可以自定义。比如，下面的代码 SimplePartitioner 中，将消息的 key 做了 hashcode，然后和分区数（numPartitions）做模运算，使得每一个 key 都可以分布到一个分区中：

```
package com.lxw1234.kafka;
 
import kafka.producer.Partitioner;
import kafka.utils.VerifiableProperties;
 
public class SimplePartitioner implements Partitioner {
	
	public SimplePartitioner (VerifiableProperties props) {
	}
	
	@Override
	public int partition(Object key, int numPartitions) {
		int partition = 0;
		String k = (String)key;
		partition = Math.abs(k.hashCode()) % numPartitions;
		return partition;
	}
	
}

```

在创建 Topic 时候可以使用–partitions <numPartitions> 指定分区数。也可以在 server.properties 配置文件中配置参数 num.partitions 来指定默认的分区数。

但有一点需要注意，为 Topic 创建分区时，分区数最好是 broker 数量的整数倍，这样才能是一个 Topic 的分区均匀的分布在整个 Kafka 集群中，假设我的 Kafka 集群由 4 个 broker 组成，以下图为例：

![](http://img.lxw1234.com/1030-1.jpg)

创建带分区的 Topic
------------

现在创建一个 topic “lxw1234”，为该 topic 指定 4 个分区，那么这 4 个分区将会在每个 broker 上各分布一个：

```
./kafka-topics.sh 
--create 
--zookeeper zk1:2181,zk2:2181,zk3:2181 
--replication-factor 1
--partitions 4 
--topic lxw1234

```

![](http://img.lxw1234.com/1030-2.jpg)

这样所有的分区就均匀分布在集群中，如果创建 topic 时候指定了 3 个分区，那么就有一个 broker 上没有该 topic 的分区。

带分区规则的生产者
---------

现在用一个生产者示例（PartitionerProducer），向 Topic lxw1234 中发送消息。该生产者使用的分区规则，就是上面的 SimplePartitioner。从 0-10 一共 11 条消息，每条消息的 key 为”key”+index，消息内容为”key”+index+”–value”+index。比如：key0–value0、key1–value1、、、key10–value10。

```
package com.lxw1234.kafka;
 
import java.util.Properties;
 
import kafka.javaapi.producer.Producer;
import kafka.producer.KeyedMessage;
import kafka.producer.ProducerConfig;
 
public class PartitionerProducer {
	public static void main(String[] args) {
		Properties props = new Properties();
		props.put("serializer.class", "kafka.serializer.StringEncoder");
		props.put("metadata.broker.list", "127.0.0.17:9091,127.0.0.17:9092,127.0.0.102:9091,127.0.0.102:9092");
		props.put("partitioner.class", "com.lxw1234.kafka.SimplePartitioner");
		Producer<String, String> producer = new Producer<String, String>(new ProducerConfig(props));
	    String topic = "lxw1234";
	    for(int i=0; i<=10; i++) {
	    	String k = "key" + i;
	    	String v = k + "--value" + i;
	    	producer.send(new KeyedMessage<String, String>(topic,k,v));
	    }
	    producer.close();
	}
}
 

```

理论上来说，生产者在发送消息的时候，会按照 SimplePartitioner 的规则，将 key0 做 hashcode，然后和分区数（4）做模运算，得到分区索引：

hashcode(”key0”) % 4 = 1

hashcode(”key1”) % 4 = 2

hashcode(”key2”) % 4 = 3

hashcode(”key3”) % 4 = 0

         ……

对应的消息将会被发送至相应的分区中。

统计各分区消息的消费者
-----------

下面的消费者代码用来验证，在消费数据时，打印出消息所在的分区及消息内容：

```
package com.lxw1234.kafka;
 
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Properties;
 
import kafka.consumer.Consumer;
import kafka.consumer.ConsumerConfig;
import kafka.consumer.ConsumerIterator;
import kafka.consumer.KafkaStream;
import kafka.javaapi.consumer.ConsumerConnector;
import kafka.message.MessageAndMetadata;
 
public class MyConsumer {
	public static void main(String[] args) {
		String topic = "lxw1234";
		ConsumerConnector consumer = Consumer.createJavaConsumerConnector(createConsumerConfig()); 
		Map<String, Integer> topicCountMap = new HashMap<String, Integer>();
		topicCountMap.put(topic, new Integer(1));
		Map<String, List<KafkaStream<byte[], byte[]>>> consumerMap = consumer.createMessageStreams(topicCountMap);
		KafkaStream<byte[], byte[]> stream =  consumerMap.get(topic).get(0);
		ConsumerIterator<byte[], byte[]> it = stream.iterator();
	    while(it.hasNext()) {
	    	MessageAndMetadata<byte[], byte[]> mam = it.next();
	    	System.out.println("consume: Partition [" + mam.partition() + "] Message: [" + new String(mam.message()) + "] ..");
	    }
	      
	}
	
	private static ConsumerConfig createConsumerConfig() {
	    Properties props = new Properties();
	    props.put("group.id","group1");
	    props.put("zookeeper.connect","127.0.0.132:2181,127.0.0.133:2182,127.0.0.134:2183");
	    props.put("zookeeper.session.timeout.ms", "400");
	    props.put("zookeeper.sync.time.ms", "200");
	    props.put("auto.commit.interval.ms", "1000");
	    props.put("auto.offset.reset", "smallest");
	    return new ConsumerConfig(props);
	  }
}
 
 

```

运行程序验证结果
--------

先启动消费者，再运行生产者。

之后在消费者的控制台可以看到如下输出：

![](http://img.lxw1234.com/1030-3.jpg)

结果和正常预期一致。

相关阅读：

[Kafka 架构和原理深度剖析](http://lxw1234.com/archives/2015/09/504.htm)  
[Kafka 安装配置测试](http://lxw1234.com/archives/2015/09/510.htm)

接下来将学习使用 Kafka 的底层 API（low-level API），指定分区和 offset 来消费数据。

##### 您可以关注 [lxw 的大数据田地](http://lxw1234.com/) ，或者 [加入邮件列表](http://163.fm/YHfRFnF) ，随时接收博客更新的通知邮件。

如果觉得本博客对您有帮助，请 [赞助作者](http://lxw1234.com/pay-blog) 。

转载请注明：[lxw 的大数据田地](http://lxw1234.com/) » [Kafka 分区机制介绍与示例](http://lxw1234.com/archives/2015/10/538.htm)