# Spark Streaming 概述

来源去向：

Kafka、 Flume 和 TCP 套接字 		-->	Spark Streaming 		--> 	HDFS，数据库



Spark Streaming 的核心抽象是 DStream ，流数据按照时间切分成批数据（RDD)。

![image-20220128102105527](picture/image-20220128102105527.png)

![image-20220128102117617](picture/image-20220128102117617.png)



# wordcount

```scala
object StreamWordCount {
    def main(args: Array[String]): Unit = {
        val sparkConf = new SparkConf().setMaster("local[*]").setAppName("StreamWordCount")
        val ssc = new StreamingContext(sparkConf, Seconds(3))
        
        //通过监控端口创建 DStream，读进来的数据为一行行
        val lineStreams = ssc.socketTextStream("linux1", 9999)
        
        val wordStreams = lineStreams.flatMap(_.split(" "))
        val wordAndOneStreams = wordStreams.map((_, 1))
        val wordAndCountStreams = wordAndOneStreams.reduceByKey(_+_)
        
        //启动
        ssc.start() 
        ssc.awaitTermination()
    }
}
```



# 自定义Receiver

```scala
class CustomerReceiver(host: String, port: Int) extends Receiver[String](StorageLevel.MEMORY_ONLY) {
    
    override def onStart(): Unit = {
        new Thread("Socket Receiver") { 
            override def run() {
                receive()}
        }.start()
    }
    
    //读数据并将数据发送给 Spark 
    def receive(): Unit = {
        var socket: Socket = new Socket(host, port)
        val reader = new BufferedReader(new InputStreamReader(socket.getInputStream, StandardCharsets.UTF_8))

		//当 receiver 没有关闭并且输入数据不为空，则循环发送数据给 Spark 
        input = reader.readLine()
        var input: String = null
        while (!isStopped() && input != null) {
            store(input)
            input = reader.readLine()
        }
        
        reader.close() 
        socket.close()
        
        restart("restart")
    }
    
    override def onStop(): Unit = {}
}
```

```scala
object FileStream {
    def main(args: Array[String]): Unit = {
        val sparkConf = new SparkConf().setMaster("local[*]").setAppName("StreamWordCount")
        val ssc = new StreamingContext(sparkConf, Seconds(5))
        
		val lineStream = ssc.receiverStream(new CustomerReceiver("hadoop102", 9999))
        
        val wordStream = lineStream.flatMap(_.split("\t"))
        val wordAndOneStream = wordStream.map((_, 1))
        val wordAndCountStream = wordAndOneStream.reduceByKey(_ + _)
        
        SparkStreamingContext ssc.start() ssc.awaitTermination()
    }
}
```



# kafka 数据源

spark-streaming-kafka 0.8 废弃

spark-streaming-kafka 1.0 使用 Direct 模式，不是receiver 模式。即每个executor自己从kafka拉取数据，而不是通过receiver executor拉取数据。

```scala
import org.apache.kafka.clients.consumer.{ConsumerConfig, ConsumerRecord} import org.apache.spark.SparkConf
import org.apache.spark.streaming.dstream.{DStream, InputDStream}
import org.apache.spark.streaming.kafka010.{ConsumerStrategies, KafkaUtils, LocationStrategies}
import org.apache.spark.streaming.{Seconds, StreamingContext}

object DirectAPI {
    def main(args: Array[String]): Unit = {
        val sparkConf: SparkConf = new SparkConf().setAppName("ReceiverWordCount").setMaster("local[*]")
        val ssc = new StreamingContext(sparkConf, Seconds(3))
        
        //定义 Kafka 参数
        val kafkaPara: Map[String, Object] = Map[String, Object](
            ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG ->"linux1:9092,linux2:9092,linux3:9092", 
            ConsumerConfig.GROUP_ID_CONFIG -> "atguigu", 
            "key.deserializer" ->"org.apache.kafka.common.serialization.StringDeserializer", 
            "value.deserializer" ->"org.apache.kafka.common.serialization.StringDeserializer")
        
        //读取 Kafka 数据创建 DStream
        val kafkaDStream: InputDStream[ConsumerRecord[String, String]] = KafkaUtils.createDirectStream[String, String](
            ssc,LocationStrategies.PreferConsistent, ConsumerStrategies.Subscribe[String, String](Set("atguigu"), kafkaPara))
        //处理数据
        val valueDStream: DStream[String] = kafkaDStream.map(record => record.value()).map((_, 1)).reduceByKey(_ + _).print()

        ssc.start() 
        ssc.awaitTermination()
    }
}
```



# 无状态转化操作 

无状态转化算子：就是算子应用到 DStream 的每个RDD上面去，不会影响其他RDD

![image-20220128172421965](picture/image-20220128172421965.png)

transform：转化RDD操作，通过任意函数把RDD转化成RDD

```scala
import StreamingContext._
val wordAndCountDStream: DStream[(String, Int)] = lineDStream.transform(rdd =>{
    val words: RDD[String] = rdd.flatMap(_.split(" "))  
    val wordAndOne: RDD[(String, Int)] = words.map((_, 1))
    val value: RDD[(String, Int)] = wordAndOne.reduceByKey(_ + _)
    value})
```

join

```scala
object JoinTest {
    def main(args: Array[String]): Unit = {
        //3.从端口获取数据创建流
        val lineDStream1: ReceiverInputDStream[String] = ssc.socketTextStream("linux1", 9999)
        val lineDStream2: ReceiverInputDStream[String] = ssc.socketTextStream("linux2", 8888)
        
        //4.将两个流转换为 KV 类型
        val wordToOneDStream: DStream[(String, Int)] = lineDStream1.flatMap(_.split(" ")).map((_, 1))
        val wordToADStream: DStream[(String, String)] = lineDStream2.flatMap(_.split(" ")).map((_, "a"))
        
        //5.流的 JOIN
        val joinDStream: DStream[(String, (Int, String))] = wordToOneDStream.join(wordToADStream)
    }
}
```



# 有状态转化操作

在 DStream 中跨批次维护状态，要借助一个状态变量。



UpdateStateByKey

```scala
object WorldCount {
    def main(args: Array[String]) {
        
        // 定义更新状态方法，参数 values 为当前批次单词频度，state 为以往批次单词频度
        val updateFunc = (values: Seq[Int], state: Option[Int]) => { 
            val currentCount = values.foldLeft(0)(_ + _)
            val previousCount = state.getOrElse(0) 
            Some(currentCount + previousCount)
        }
        
        val conf = new SparkConf().setMaster("local[*]").setAppName("NetworkWordCount")
        val ssc = new StreamingContext(conf, Seconds(3)) ssc.checkpoint("./ck")
        val lines = ssc.socketTextStream("linux1", 9999)
        
        val words = lines.flatMap(_.split(" "))
        val pairs = words.map(word => (word, 1))
        
        // 使用 updateStateByKey 来更新状态，统计从运行开始以来单词总的次数
        val stateDstream = pairs.updateStateByKey[Int](updateFunc) 
        stateDstream.print()
        
        ssc.start()
        ssc.awaitTermination()
    }
}
```



Window Operations 

```
window ：返回一个新的DStream 
countByWindow：返回一个滑动窗口计数流中的元素个数
reduceByWindow：通过使用自定义函数整合滑动区间流元素来创建一个新的单元素流
reduceByKeyAndWindow：
```

```scala
object WorldCount {
    def main(args: Array[String]) { 
        val conf = newSparkConf().setMaster("local[2]").setAppName("NetworkWordCount") 
        val ssc = new StreamingContext(conf, Seconds(3)) 
        ssc.checkpoint("./ck")
        
        val lines = ssc.socketTextStream("linux1", 9999)
        val words = lines.flatMap(_.split(" "))
        val pairs = words.map(word => (word, 1))
        
        //批次3秒，窗口（计算范围）12秒，滑动时间（触发间隔）6秒，窗口与滑动时间是批次时间的整数倍
        val wordCounts = pairs.reduceByKeyAndWindow((a:Int,b:Int) => (a + b),Seconds(12), Seconds(6))
        wordCounts.print()
        
        ssc.start()
        ssc.awaitTermination()
    }
}
```



# DStream输出

print()：在运行流程序的 driver 节点上打印 DStream 中每一批次数据的最开始 10 个元素。



saveAsTextFiles(prefix, [suffix])：以 text 文件形式存储这个 DStream 的内容。每一批次的存储文件名基于参数中的 prefix 和 suffix。

saveAsObjectFiles(prefix, [suffix])：以 Java 对象序列化的方式将 Stream 中的数据保存为SequenceFiles 。

saveAsHadoopFiles(prefix, [suffix])：将 Stream 中的数据保存为 Hadoop files. 每一批次的存储文件名基于参数中的为"prefix-TIME_IN_MS[.suffix]"。



foreachRDD(func)：将函数 func 用于产生于 stream 的每一个RDD。参数传入的函数 func 实现将每一个RDD 中数据推送到外部系统，如将RDD 存入文件。

`1) 连接不能写在 driver 层面（序列化）。`

`2) 如果写在 foreach 则每个 RDD 中的每一条数据都创建，得不偿失。`

`3) 增加 foreachPartition，在分区创建（获取）。`



# 优雅关闭

MonitorStop

```scala
import java.net.URI
import org.apache.hadoop.conf.Configuration import org.apache.hadoop.fs.{FileSystem, Path}
import org.apache.spark.streaming.{StreamingContext, StreamingContextState} 

class MonitorStop(ssc: StreamingContext) extends Runnable {
    override def run(): Unit = {
        val fs: FileSystem = FileSystem.get(new URI("hdfs://linux1:9000"), new Configuration(), "atguigu")
        while (true) { 
            try
            	Thread.sleep(5000) 
            catch {
                case e: InterruptedException => e.printStackTrace()
            }
            val state: StreamingContextState = ssc.getState
            val bool: Boolean = fs.exists(new Path("hdfs://linux1:9000/stopSpark")) 
            if (bool) {
                if (state == StreamingContextState.ACTIVE) { 
                    ssc.stop(stopSparkContext = true, stopGracefully = true) 
                    System.exit(0)
                }
            }
        }
    }
}
```

SparkTest

```scala
import org.apache.spark.SparkConf
import org.apache.spark.streaming.dstream.{DStream, ReceiverInputDStream} 
import org.apache.spark.streaming.{Seconds, StreamingContext}

object SparkTest {
    def createSSC(): _root_.org.apache.spark.streaming.StreamingContext = {
        val update: (Seq[Int], Option[Int]) => Some[Int] = (values: Seq[Int], status: Option[Int]) => {
            //当前批次内容的计算
            val sum: Int = values.sum
            //取出状态信息中上一次状态
            val lastStatu: Int = status.getOrElse(0)
            Some(sum + lastStatu)
        }
        val sparkConf: SparkConf = new SparkConf().setMaster("local[4]").setAppName("SparkTest")
        //设置优雅的关闭
        sparkConf.set("spark.streaming.stopGracefullyOnShutdown", "true")
        val ssc = new StreamingContext(sparkConf, Seconds(5)) 
        ssc.checkpoint("./ck")
        val line: ReceiverInputDStream[String] = ssc.socketTextStream("linux1", 9999) 
        val word: DStream[String] = line.flatMap(_.split(" "))
        val wordAndOne: DStream[(String, Int)] = word.map((_, 1))
        val wordAndCount: DStream[(String, Int)] = wordAndOne.updateStateByKey(update) 
        wordAndCount.print()
        ssc
    }
    def main(args: Array[String]): Unit = {
        val ssc: StreamingContext = StreamingContext.getActiveOrCreate("./ck", () => createSSC())
        new Thread(new MonitorStop(ssc)).start() 
        ssc.start()
        ssc.awaitTermination()
    }
}
```



案例实操：picture/案例实操.doc