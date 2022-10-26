官方文档：https://docs.cloudera.com/index.html

总结文档：https://mp.weixin.qq.com/s/QrQN6F54P7dUeWIJqS9QSQ



# 安装

http://www.amoscloud.com/?p=2638
http://www.amoscloud.com/?p=2839



# 简介

CDH是由cloudera进行开发的大数据一站式平台管理解决方案,基于Hadoop生态的第三方发行版本。

使用CDH部署集群不能代替对各个组件进行单独的学习了解的工作,非常推荐大家从单个组件安装部署开始最后在统一使用CDH部署。



组件兼容：CDH每个版本都会有兼容认证都是经过严格的测试之后公布的,理论上来说只要统一CDH版本就不会出现兼容问题。

稳定安全：版本更新快。通常情况，比如CDH每个季度会有一个update，每一年会有一个release。

安装配置管理：统一的网页进行安装配置,非常详细的文档以及配置的分类注解以及推荐配置。

资源监控管理运维：运维简单。提供了管理、监控、诊断、配置修改的工具，管理配置方便，定位问题快速、准确，使运维工作简单，有效。



# 集群规划

1.管理节点（Master Hosts）：主要用于运行Hadoop的管理进程，比如HDFS的NameNode，YARN的ResourceManager。

2.工具节点（Utility Hosts）:主要用于运行非管理进程的其他进程，比如Cloudera Manager和Hive Metastore。

3.边缘节点（Edge Hosts）：用于集群中启动作业的客户端机器，边缘节点的数量取决于工作负载的类型和数量。

4.工作节点（Worker Hosts）：主要用于运行DataNode以及其他分布式进程，比如ImpalaD。



小于10台：一个管理节点主要用于安装NameNode和ResourceManager，工具节点和边缘节点复用一个，用于安装Cloudera Manager等，剩余3-7台工作节点。

10-20台：我们会用2个管理节点用于安装2个NameNode，一个工具节点用于安装Cloudera Manager等，如果机器充足或者Hue/HiveServer2/Flume的负载特别高，可以考虑独立出边缘节点用于部署这些角色，否则也可以跟Cloudera Manager复用。最后还剩下7-17个工作节点。



# 组件端口

**Hadoop：**

> 50070、9870：HDFS WEB UI端口
>
> 8088 ： Yarn 的WEB UI 接口
>
> 10020、  19888：historyserver端口



**CDH：**

> 7180： Cloudera Manager WebUI端口



**Spark：**

> 7077 ： spark 的master与worker进行通讯的端口 standalone集群提交Application的端口
> 8080 ： master的WEB UI端口 资源调度
> 8081 ： worker的WEB UI 端口 资源调度
> 4040 ： Driver的WEB UI 端口 任务调度
> 18080、18088：Spark History Server的WEB UI 端口



**Hbase:**

> 60010：Hbase的master的WEB UI端口
> 60030：Hbase的regionServer的WEB UI 管理端口



**HUE：**

> 8888： Hue WebUI 端口



**Hive:**

> 10000：hiveserver2， Hive 的JDBC端口



**Zookeeper:**

> 2181 ： 客户端连接zookeeper的端口
> 2888 ： zookeeper集群内通讯使用，Leader监听此端口



**Redis：**

> 6379： Redis服务端口



**Kafka：**

> 9092： Kafka集群节点之间通信的RPC端口



# 集群内存分配

Cloudera Manager 会针对每台主机计算 CDH & Cloudera Management Services 使用的内存总量的最大内存提交量。

```
total_system_memory ：一台机器的总内存，合理分配给下面的地方。

available_memory_for_hadoop = (total_system_memory * 0.8)，其他内存分配给系统。
total_hadoop_java_heap = sum ( hadoop_java_heaps ) * 1.3，hadoop_java_heaps 为该节点所有角色java_heap之和，详见下图
```

![图片](picture/640)



# 集成 spark-sql

https://mp.weixin.qq.com/s/1CAJxfhudvoWqCeMBwYfhA



CDH 阉割了spark 的 spark-thrift（对应hive的hiveserver2）的服务和 Spark SQL CLI（对应hive 的Beeline）。

CDH 推荐使用 impala 代替 spark-sql。



CDH 版本的 spark-shell 仍然可以使用 sparksession，sparksession 可以生成DataFrame，使用spark-sql的一切功能。

只是编程的时候需要连接hive，可以使用 SparkSession + Hive Metastore 的方式访问 Hive。无需集成



只有外部客户端想通过spark-thrift使用spark-sql的时候才需要集成。

集成spark-thrift：https://xujiahua.github.io/posts/20200410-spark-thrift-server-cdh/







































































































































































