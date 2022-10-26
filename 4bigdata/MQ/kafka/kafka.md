# 简介

Apache Kafka 是一个开源的分布式消息队列。大多消息中间件都是基于JMS标准实现的，Apache Kafka 类似于JMS的实现。

Kafka 作用：削峰，来异构、解耦系统。



# 架构

**kafka broker**：kafka集群中包含的服务器

**Kafka Producer**：消息生产者、发布消息到 kafka 集群的终端或服务。

**Kafka consumer**：消息消费者、负责消费数据。

**Kafka Topic**: 主题，一类消息的名称。存储数据时将一类数据存放在某个topci下。



注意：Kafka的元数据都是存放在zookeeper中



# 原理

### 副本

把数据复制保存多份，可以防止数据丢失，但是会占据磁盘。



### 分区与分片

![img](picture/c409470d124981148ee82ed8ee1fe21c.png)

topic 主题：是一个逻辑概念。

partition 分区：是一个物理概念，是文件夹，每个broker都可以有多个分区。但是分区在不同broker上，可以加大并发。

segment 分片：分区下的数据存储细分，为了分区的数据不那么大，方便索引和查询，所以用了分片。



### 数据不丢失机制

生产者：ack机制（生产者需要接收到确认信息），有3个级别。

```
0 : 生产者只是不断的发送数据, 不关心broker是否接收到了数据
1: 生产者向broker发送数据, 发送到了一个topic中, 要求topic中的主节点接收到数据, 认为数据接收到了
-1 :要求topic中分片上所有的副本都接收到了数据,认为数据已经接收到了
```



消费者：通过偏移量维护，kafka自己记录了每次消费的offset数值，下次继续消费的时候，会接着上次的offset进行消费。



# 安装

需要先安装zookeeper



解压

```shell
tar -zxvf kafka_2.11-2.4.1.tgz -C /opt/module/
mv kafka_2.11-2.4.1/ kafka
#kafka下创建logs
mkdir logs
```

修改配置文件

```shell
cd config/
vi server.properties
```

```shell
#broker的全局唯一编号，不能重复
broker.id=0
#删除topic时真正删除数据
delete.topic.enable=true
#kafka运行日志存放的路径
log.dirs=/opt/module/kafka/data
#配置连接Zookeeper集群地址
zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop104:2181/kafka
```

环境变量

```shell
sudo vi /etc/profile.d/my_env.sh
```

```shell
export KAFKA_HOME=/opt/module/kafka
export PATH=$PATH:$KAFKA_HOME/bin
```

```shell
source /etc/profile.d/my_env.sh
```

分发安装包

修改broker.id

启动

```shell
bin/kafka-server-start.sh -daemon 
bin/kafka-server-stop.sh
```



# 配置

### 生产者压力测试

创建一个test topic，设置为3个分区2个副本

```shell
bin/kafka-topics.sh --zookeeper hadoop102:2181,hadoop103:2181,hadoop104:2181/kafka\
--create --replication-factor 2 --partitions 3 --topic test
```

启动压测脚本

```shell
bin/kafka-producer-perf-test.sh --topic test \
--record-size 100 \		#record-size是一条信息有多大，单位是字节。
--num-records 10000000 \	#num-records是总共发送多少条信息。
--throughput -1 \		#throughput 是每秒多少条信息，设成-1，表示不限流
--producer-props bootstrap.servers=hadoop102:9092,hadoop103:9092,hadoop104:9092
```

打印出的参数解析

```shell
699884 records sent, 139976.8 records/sec (13.35 MB/sec), 1345.6 ms avg latency, 2210.0 ms max latency.
#13.35 MB/sec，副本数为2，集群网络总带宽30m/s。
```

创建 topic 时可以设置 batch.size和 linger.ms 。用于改变吞吐量和延时。

Kafka需要考虑高吞吐量与延时的平衡。



### 消费者压力测试

Consumer的测试，如果这四个指标（IO，CPU，内存，网络）都不能改变，考虑增加分区数来提升性能。

```shell
 #开始测试
 bin/kafka-consumer-perf-test.sh --broker-list hadoop102:9092,hadoop103:9092,hadoop104:9092 --topic test 、
 --fetch-size 10000 \	#指定每次fetch的数据的大小
 --messages 10000000 \	#总共要消费的消息个数
 --threads 1
```

结果说明

```shell
start.time, end.time, data.consumed.in.MB, MB.sec,data.consumed.in.nMsg, nMsg.sec
2021-08-03 21:17:21:778, 2021-08-03 21:18:19:775, 514.7169, 8.8749, 5397198, 93059.9514

#开始测试时间，测试结束数据，共消费数据514.7169MB，吞吐量8.8749MB/s
```

吞吐量受网络带宽和fetch-size的影响



### 机器数量计算

Kafka机器数量= 2 *（峰值生产速度 * 副本数 / 100）+ 1

峰值生产速度：峰值生产速度可以压测得到。

副本数：默认1个，企业一般2个。



### Kafka分区数计算

分区数一般设置为：3-10个



创建一个只有1个分区的topic

测试这个topic的producer吞吐量和consumer吞吐量。

假设他们的值分别是Tp和Tc，单位可以是MB/s。

然后假设总的目标吞吐量是Tt，那么分区数 = Tt / min（Tp，Tc）



例如：producer吞吐量 = 20m/s；consumer吞吐量 = 50m/s，期望吞吐量100m/s；分区数 = 100 / 20 = 5分区



### 消费者组消费者数量

生产速度快，消费者跟不上。

可以通过消费者组来解决，多个消费者来同时消费同一个topic的数据。

消费者组中消费者数量不可大于分区数量，否则会闲置。



# 命令

查看Kafka Topic列表

```shell
bin/kafka-topics.sh --zookeeper hadoop102:2181/kafka --list
```

查看Kafka Topic详情

```shell
bin/kafka-topics.sh --zookeeper hadoop102:2181/kafka --describe --topic topic_log
```

创建Kafka Topic

```shell
bin/kafka-topics.sh --zookeeper hadoop102:2181,hadoop103:2181,hadoop104:2181/kafka\
--create --replication-factor 1 --partitions 1 --topic topic_log
```

删除Kafka Topic

```shell
bin/kafka-topics.sh --delete --zookeeper hadoop102:2181,hadoop103:2181,hadoop104:2181/kafka --topic topic_log
```

Kafka生产消息

```shell
bin/kafka-console-producer.sh \
--broker-list hadoop102:9092 --topic topic_log
```

Kafka消费消息

```shell
bin/kafka-console-consumer.sh \
--bootstrap-server hadoop102:9092 --from-beginning --topic topic_log
```



# API

第一步: 添加kafka相关的依赖

```xml
<dependency>     
    <groupId>org.apache.kafka</groupId>     
    <artifactId>kafka-clients</artifactId>     
    <version>0.11.0.1</version>   
</dependency>
```



编写生产者 : 

```java
// 测试kafka的生产者
public class KafkaProducerTest {

    public static void main(String[] args) {
        //1.1 设置生产者相关的参数
        Properties props = new Properties();
        props.put("bootstrap.servers", "node01:9092"); // 连接的服务器地址
        props.put("acks", "all"); // 保证数据不丢失的参数
        props.put("retries", 3); // retries :  数据发送失败, 重试的次数
        props.put("batch.size", 16384); // batch.size : 发送一批的数据大小  单位 : 字节
        props.put("linger.ms", 1); // linger.ms :  描述什么时候发送一次数据  单位毫秒
        props.put("buffer.memory", 33554432); // buffer.memory :  缓冲池大小  实际使用的 服务器资源充足, 设置和内存等量
        // 数据的发送, 是不是需要经过网络的传输: 必然涉及到序列化
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

        //1. 创建一个kafka的生产者对象  :  KafkaProducer
        Producer<String, String> producer = new KafkaProducer<String, String>(props);
        
        for (int i = 0; i < 100; i++) {
            producer.send(new ProducerRecord<String, String>("my-topic", Integer.toString(i)));
           // producer.send(new ProducerRecord<String, String>())
        }
        producer.close();
    }
}
```



编写消费者

```java
public class KafkaConsumerTest {

    public static void main(String[] args) {
        // 1.1 消费者的配置对象
        Properties props = new Properties();
        props.put("bootstrap.servers", "node01:9092"); //服务的地址
        props.put("group.id", "test"); // 组id  设置点对点 和发布订阅
        props.put("enable.auto.commit", "true"); // 自动提交:
        props.put("auto.commit.interval.ms", "1000"); // 提交间隔时间 :  1000毫秒
        // 反序列化 :
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

        //1. 创建一个kafka的消费者: KafkaConsumer
        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(props);
        consumer.subscribe(Arrays.asList("my-topic"));
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(100);  // 取数据
            for (ConsumerRecord<String, String> record : records) {
                System.out.println("发送过来的数据是: " + record.value());
            }
        }
    }
}
```

 说明：可以通过kafka控制台和Java客户端进行相互测试：例如，Java客户端发送消息，控制台创建消费者消费数据。





































