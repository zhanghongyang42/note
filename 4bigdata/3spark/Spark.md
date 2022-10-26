Spark B站尚硅谷视频

Spark 动物书最新版

Spark 官网：https://spark.apache.org/docs/latest/

中文文档：https://spark-reference-doc-cn.readthedocs.io/zh_CN/latest

深入理解：https://spark-internals.books.yourtion.com/index.html



# 概述

spark 比 MR 优点：数据存储内存中(尤其是多个作业间的数据交互通过内存而不是磁盘)，使得迭代算法和交互式分析非常快。

spark 既有计算功能，又有调度功能。我们一般只使用计算功能配合YARN的调度功能。

![img](picture/20190324122107378.png)

Spark Core: 是其他组件的基础，RDD是其主要抽象。

Spark SQL：可以使用SQL或者HQL查询数据。

Spark Streaming：对实时数据进行流失计算的组件。

MLlib：机器学习库。

GraphX：用于图计算。



# 安装

详见 picture/环境搭建.docx

详见 picture/install.docx



# 配置

### 配置生效顺序

sparkconf > spark-submit --选项 > spark-defaults.conf配置 > spark-env.sh配置 > 默认值 。

`cluster模式不会读取 sparkconf 配置，直接读取 spark-submit 配置。client模式都会读取，sparkconf 优先级最高。`



以上是提交到集群中的spark代码配置生效顺序，如果使用 idea 的话。配置生效顺序是 sparkconf > resource 下配置文件。

` resource 下配置文件在提交到集群的时候失效。但是 spark-submit有参数支持提交配置文件`



### 查看配置

显示完整的参数列表。

```
./bin/spark-submit --hp
```



web UI （http://:4040）"Environment"标签页，显示你已经配置的属性。



### 配置属性

##### 应用属性

| 属性名                   | 缺省值                                      | 意义                                                         |
| :----------------------- | :------------------------------------------ | :----------------------------------------------------------- |
| `spark.app.name`         | (none)                                      | The name of your application. This will appear in the UI and in log data. |
| `spark.master`           | (none)                                      | The cluster manager to connect to. See the list of[ allowed master URL's](https://colobu.com/2014/12/10/spark-configuration/scala-programming-guide.html#master-urls). |
| `spark.executor.memory`  | 512m                                        | Amount of memory to use per executor process, in the same format as JVM memory strings (e.g.`512m`,`2g`). |
| `spark.serializer`       | org.apache.spark.serializer. JavaSerializer | Class to use for serializing objects that will be sent over the network or need to be cached in serialized form. The default of Java serialization works with any Serializable Java object but is quite slow, so we recommend[using`org.apache.spark.serializer.KryoSerializer` and configuring Kryo serialization](https://colobu.com/2014/12/10/spark-configuration/tuning.html) when speed is necessary. Can be any subclass of[`org.apache.spark.Serializer`](https://colobu.com/2014/12/10/spark-configuration/api/scala/index.html#org.apache.spark.serializer.Serializer). |
| `spark.kryo.registrator` | (none)                                      | If you use Kryo serialization, set this class to register your custom classes with Kryo. It should be set to a class that extends[`KryoRegistrator`](https://colobu.com/2014/12/10/spark-configuration/api/scala/index.html#org.apache.spark.serializer.KryoRegistrator). See the[tuning guide](https://colobu.com/2014/12/10/spark-configuration/tuning.html#data-serialization) for more details. |
| `spark.local.dir`        | /tmp                                        | Directory to use for "scratch" space in Spark, including map output files and RDDs that get stored on disk. This should be on a fast, local disk in your system. It can also be a comma-separated list of multiple directories on different disks. NOTE: In Spark 1.0 and later this will be overriden by SPARK_LOCAL_DIRS (Standalone, Mesos) or LOCAL_DIRS (YARN) environment variables set by the cluster manager. |
| `spark.logConf`          | false                                       | Logs the effective SparkConf as INFO when a SparkContext is started. |



##### Runtime Environment

| 属性名                                        | 缺省值                                      | 意义                                                         |
| :-------------------------------------------- | :------------------------------------------ | :----------------------------------------------------------- |
| `spark.executor.extraJavaOptions`             | (none)                                      | A string of extra JVM options to pass to executors. For instance, GC settings or other logging. Note that it is illegal to set Spark properties or heap size settings with this option. Spark properties should be set using a SparkConf object or the spark-defaults.conf file used with the spark-submit script. Heap size settings can be set with spark.executor.memory. |
| `spark.executor.extraClassPath`               | (none)                                      | Extra classpath entries to append to the classpath of executors. This exists primarily for backwards-compatibility with older versions of Spark. Users typically should not need to set this option. |
| `spark.executor.extraLibraryPath`             | (none)                                      | Set a special library path to use when launching executor JVM's. |
| `spark.files.userClassPathFirst`              | false                                       | (Experimental) Whether to give user-added jars precedence over Spark's own jars when loading classes in Executors. This feature can be used to mitigate conflicts between Spark's dependencies and user dependencies. It is currently an experimental feature. |
| `spark.python.worker.memory`                  | 512m                                        | Amount of memory to use per python worker process during aggregation, in the same format as JVM memory strings (e.g.`512m`,`2g`). If the memory used during aggregation goes above this amount, it will spill the data into disks. |
| `spark.executorEnv.[EnvironmentVariableName]` | (none)                                      | Add the environment variable specified by`EnvironmentVariableName` to the Executor process. The user can specify multiple of these and to set multiple environment variables. |
| `spark.mesos.executor.home`                   | driver side`SPARK_HOME`                     | Set the directory in which Spark is installed on the executors in Mesos. By default, the executors will simply use the driver's Spark home directory, which may not be visible to them. Note that this is only relevant if a Spark binary package is not specified through`spark.executor.uri`. |
| `spark.mesos.executor.memoryOverhead`         | executor memory * 0.07, with minimum of 384 | This value is an additive for`spark.executor.memory`, specified in MiB, which is used to calculate the total Mesos task memory. A value of`384` implies a 384MiB overhead. Additionally, there is a hard-coded 7% minimum overhead. The final overhead will be the larger of either `spark.mesos.executor.memoryOverhead` or 7% of `spark.executor.memory`. |



##### Shuffle Behavior

| 属性名                                    | 缺省值 | 意义                                                         |
| :---------------------------------------- | :----- | :----------------------------------------------------------- |
| `spark.shuffle.consolidateFiles`          | false  | If set to "true", consolidates intermediate files created during a shuffle. Creating fewer files can improve filesystem performance for shuffles with large numbers of reduce tasks. It is recommended to set this to "true" when using ext4 or xfs filesystems. On ext3, this option might degrade performance on machines with many (>8) cores due to filesystem limitations. |
| `spark.shuffle.spill`                     | true   | If set to "true", limits the amount of memory used during reduces by spilling data out to disk. This spilling threshold is specified by`spark.shuffle.memoryFraction`. |
| `spark.shuffle.spill.compress`            | true   | Whether to compress data spilled during shuffles. Compression will use`spark.io.compression.codec`. |
| `spark.shuffle.memoryFraction`            | 0.2    | Fraction of Java heap to use for aggregation and cogroups during shuffles, if`spark.shuffle.spill` is true. At any given time, the collective size of all in-memory maps used for shuffles is bounded by this limit, beyond which the contents will begin to spill to disk. If spills are often, consider increasing this value at the expense of`spark.storage.memoryFraction`. |
| `spark.shuffle.compress`                  | true   | Whether to compress map output files. Generally a good idea. Compression will use`spark.io.compression.codec`. |
| `spark.shuffle.file.buffer.kb`            | 32     | Size of the in-memory buffer for each shuffle file output stream, in kilobytes. These buffers reduce the number of disk seeks and system calls made in creating intermediate shuffle files. |
| `spark.reducer.maxMbInFlight`             | 48     | Maximum size (in megabytes) of map outputs to fetch simultaneously from each reduce task. Since each output requires us to create a buffer to receive it, this represents a fixed memory overhead per reduce task, so keep it small unless you have a large amount of memory. |
| `spark.shuffle.manager`                   | HASH   | Implementation to use for shuffling data. A hash-based shuffle manager is the default, but starting in Spark 1.1 there is an experimental sort-based shuffle manager that is more memory-efficient in environments with small executors, such as YARN. To use that, change this value to`SORT`. |
| `spark.shuffle.sort.bypassMergeThreshold` | 200    | (Advanced) In the sort-based shuffle manager, avoid merge-sorting data if there is no map-side aggregation and there are at most this many reduce partitions. |



##### Spark UI

| 属性名                    | 缺省值                   | 意义                                                         |
| :------------------------ | :----------------------- | :----------------------------------------------------------- |
| `spark.ui.port`           | 4040                     | Port for your application's dashboard, which shows memory and workload data. |
| `spark.ui.retainedStages` | 1000                     | How many stages the Spark UI remembers before garbage collecting. |
| `spark.ui.killEnabled`    | true                     | Allows stages and corresponding jobs to be killed from the web ui. |
| `spark.eventLog.enabled`  | false                    | Whether to log Spark events, useful for reconstructing the Web UI after the application has finished. |
| `spark.eventLog.compress` | false                    | Whether to compress logged events, if`spark.eventLog.enabled` is true. |
| `spark.eventLog.dir`      | file:///tmp/spark-events | Base directory in which Spark events are logged, if`spark.eventLog.enabled` is true. Within this base directory, Spark creates a sub-directory for each application, and logs the events specific to the application in this directory. Users may want to set this to a unified location like an HDFS directory so history files can be read by the history server. |



##### Compression and Serialization

| 属性名                                   | 缺省值                                      | 意义                                                         |
| :--------------------------------------- | :------------------------------------------ | :----------------------------------------------------------- |
| `spark.broadcast.compress`               | true                                        | Whether to compress broadcast variables before sending them. Generally a good idea. |
| `spark.rdd.compress`                     | false                                       | Whether to compress serialized RDD partitions (e.g. for`StorageLevel.MEMORY_ONLY_SER`). Can save substantial space at the cost of some extra CPU time. |
| `spark.io.compression.codec`             | snappy                                      | The codec used to compress internal data such as RDD partitions and shuffle outputs. By default, Spark provides three codecs:`lz4`,`lzf`, and`snappy`. You can also use fully qualified class names to specify the codec, e.g.`org.apache.spark.io.LZ4CompressionCodec`,`org.apache.spark.io.LZFCompressionCodec`, and`org.apache.spark.io.SnappyCompressionCodec`. |
| `spark.io.compression.snappy.block.size` | 32768                                       | Block size (in bytes) used in Snappy compression, in the case when Snappy compression codec is used. Lowering this block size will also lower shuffle memory usage when Snappy is used. |
| `spark.io.compression.lz4.block.size`    | 32768                                       | Block size (in bytes) used in LZ4 compression, in the case when LZ4 compression codec is used. Lowering this block size will also lower shuffle memory usage when LZ4 is used. |
| `spark.closure.serializer`               | org.apache.spark.serializer. JavaSerializer | Serializer class to use for closures. Currently only the Java serializer is supported. |
| `spark.serializer.objectStreamReset`     | 100                                         | When serializing using org.apache.spark.serializer.JavaSerializer, the serializer caches objects to prevent writing redundant data, however that stops garbage collection of those objects. By calling 'reset' you flush that info from the serializer, and allow old objects to be collected. To turn off this periodic reset set it to -1. By default it will reset the serializer every 100 objects. |
| `spark.kryo.referenceTracking`           | true                                        | Whether to track references to the same object when serializing data with Kryo, which is necessary if your object graphs have loops and useful for efficiency if they contain multiple copies of the same object. Can be disabled to improve performance if you know this is not the case. |
| `spark.kryo.registrationRequired`        | false                                       | Whether to require registration with Kryo. If set to 'true', Kryo will throw an exception if an unregistered class is serialized. If set to false (the default), Kryo will write unregistered class names along with each object. Writing class names can cause significant performance overhead, so enabling this option can enforce strictly that a user has not omitted classes from registration. |
| `spark.kryoserializer.buffer.mb`         | 0.064                                       | Initial size of Kryo's serialization buffer, in megabytes. Note that there will be one buffer*per core* on each worker. This buffer will grow up to`spark.kryoserializer.buffer.max.mb` if needed. |
| `spark.kryoserializer.buffer.max.mb`     | 64                                          | Maximum allowable size of Kryo serialization buffer, in megabytes. This must be larger than any object you attempt to serialize. Increase this if you get a "buffer limit exceeded" exception inside Kryo. |



##### Execution Behavior

| 属性名                             | 缺省值                                                       | 意义                                                         |
| :--------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `spark.default.parallelism`        | Local mode: number of cores on the local machineMesos fine grained mode: 8Others: total number of cores on all executor nodes or 2, whichever is larger | Default number of tasks to use across the cluster for distributed shuffle operations (`groupByKey`,`reduceByKey`, etc) when not set by user. |
| `spark.broadcast.factory`          | org.apache.spark.broadcast. TorrentBroadcastFactory          | Which broadcast implementation to use.                       |
| `spark.broadcast.blockSize`        | 4096                                                         | Size of each piece of a block in kilobytes for`TorrentBroadcastFactory`. Too large a value decreases parallelism during broadcast (makes it slower); however, if it is too small,`BlockManager` might take a performance hit. |
| `spark.files.overwrite`            | false                                                        | Whether to overwrite files added through SparkContext.addFile() when the target file exists and its contents do not match those of the source. |
| `spark.files.fetchTimeout`         | false                                                        | Communication timeout to use when fetching files added through SparkContext.addFile() from the driver. |
| `spark.storage.memoryFraction`     | 0.6                                                          | Fraction of Java heap to use for Spark's memory cache. This should not be larger than the "old" generation of objects in the JVM, which by default is given 0.6 of the heap, but you can increase it if you configure your own old generation size. |
| `spark.storage.unrollFraction`     | 0.2                                                          | Fraction of`spark.storage.memoryFraction` to use for unrolling blocks in memory. This is dynamically allocated by dropping existing blocks when there is not enough free storage space to unroll the new block in its entirety. |
| `spark.tachyonStore.baseDir`       | System.getProperty("java.io.tmpdir")                         | Directories of the Tachyon File System that store RDDs. The Tachyon file system's URL is set by`spark.tachyonStore.url`. It can also be a comma-separated list of multiple directories on Tachyon file system. |
| `spark.storage.memoryMapThreshold` | 8192                                                         | Size of a block, in bytes, above which Spark memory maps when reading a block from disk. This prevents Spark from memory mapping very small blocks. In general, memory mapping has high overhead for blocks close to or below the page size of the operating system. |
| `spark.tachyonStore.url`           | tachyon://localhost:19998                                    | The URL of the underlying Tachyon file system in the TachyonStore. |
| `spark.cleaner.ttl`                | (infinite)                                                   | Duration (seconds) of how long Spark will remember any metadata (stages generated, tasks generated, etc.). Periodic cleanups will ensure that metadata older than this duration will be forgotten. This is useful for running Spark for many hours / days (for example, running 24/7 in case of Spark Streaming applications). Note that any RDD that persists in memory for more than this duration will be cleared as well. |
| `spark.hadoop.validateOutputSpecs` | true                                                         | If set to true, validates the output specification (e.g. checking if the output directory already exists) used in saveAsHadoopFile and other variants. This can be disabled to silence exceptions due to pre-existing output directories. We recommend that users do not disable this except if trying to achieve compatibility with previous versions of Spark. Simply use Hadoop's FileSystem API to delete output directories by hand. |
| `spark.hadoop.cloneConf`           | false                                                        | If set to true, clones a new Hadoop`Configuration` object for each task. This option should be enabled to work around`Configuration` thread-safety issues (see[SPARK-2546](https://issues.apache.org/jira/browse/SPARK-2546) for more details). This is disabled by default in order to avoid unexpected performance regressions for jobs that are not affected by these issues. |
| `spark.executor.heartbeatInterval` | 10000                                                        | Interval (milliseconds) between each executor's heartbeats to the driver. Heartbeats let the driver know that the executor is still alive and update it with metrics for in-progress tasks. |



##### Networking

| 属性名                                  | 缺省值           | 意义                                                         |
| :-------------------------------------- | :--------------- | :----------------------------------------------------------- |
| `spark.driver.host`                     | (local hostname) | Hostname or IP address for the driver to listen on. This is used for communicating with the executors and the standalone Master. |
| `spark.driver.port`                     | (random)         | Port for the driver to listen on. This is used for communicating with the executors and the standalone Master. |
| `spark.fileserver.port`                 | (random)         | Port for the driver's HTTP file server to listen on.         |
| `spark.broadcast.port`                  | (random)         | Port for the driver's HTTP broadcast server to listen on. This is not relevant for torrent broadcast. |
| `spark.replClassServer.port`            | (random)         | Port for the driver's HTTP class server to listen on. This is only relevant for the Spark shell. |
| `spark.blockManager.port`               | (random)         | Port for all block managers to listen on. These exist on both the driver and the executors. |
| `spark.executor.port`                   | (random)         | Port for the executor to listen on. This is used for communicating with the driver. |
| `spark.port.maxRetries`                 | 16               | Default maximum number of retries when binding to a port before giving up. |
| `spark.akka.frameSize`                  | 10               | Maximum message size to allow in "control plane" communication (for serialized tasks and task results), in MB. Increase this if your tasks need to send back large results to the driver (e.g. using`collect()` on a large dataset). |
| `spark.akka.threads`                    | 4                | Number of actor threads to use for communication. Can be useful to increase on large clusters when the driver has a lot of CPU cores. |
| `spark.akka.timeout`                    | 100              | Communication timeout between Spark nodes, in seconds.       |
| `spark.akka.heartbeat.pauses`           | 600              | This is set to a larger value to disable failure detector that comes inbuilt akka. It can be enabled again, if you plan to use this feature (Not recommended). Acceptable heart beat pause in seconds for akka. This can be used to control sensitivity to gc pauses. Tune this in combination of `spark.akka.heartbeat.interval` and `spark.akka.failure-detector.threshold` if you need to. |
| `spark.akka.failure-detector.threshold` | 300.0            | This is set to a larger value to disable failure detector that comes inbuilt akka. It can be enabled again, if you plan to use this feature (Not recommended). This maps to akka's `akka.remote.transport-failure-detector.threshold`. Tune this in combination of `spark.akka.heartbeat.pauses` and `spark.akka.heartbeat.interval` if you need to. |
| `spark.akka.heartbeat.interval`         | 1000             | This is set to a larger value to disable failure detector that comes inbuilt akka. It can be enabled again, if you plan to use this feature (Not recommended). A larger interval value in seconds reduces network overhead and a smaller value ( ~ 1 s) might be more informative for akka's failure detector. Tune this in combination of `spark.akka.heartbeat.pauses` and `spark.akka.failure-detector.threshold` if you need to. Only positive use case for using failure detector can be, a sensistive failure detector can help evict rogue executors really quick. However this is usually not the case as gc pauses and network lags are expected in a real Spark cluster. Apart from that enabling this leads to a lot of exchanges of heart beats between nodes leading to flooding the network with those. |



##### Scheduling

| 属性名                                              | 缺省值              | 意义                                                         |
| :-------------------------------------------------- | :------------------ | :----------------------------------------------------------- |
| `spark.task.cpus`                                   | 1                   | Number of cores to allocate for each task.                   |
| `spark.task.maxFailures`                            | 4                   | Number of individual task failures before giving up on the job. Should be greater than or equal to 1. Number of allowed retries = this value - 1. |
| `spark.scheduler.mode`                              | FIFO                | The[scheduling mode](https://colobu.com/2014/12/10/spark-configuration/job-scheduling.html#scheduling-within-an-application) between jobs submitted to the same SparkContext. Can be set to`FAIR` to use fair sharing instead of queueing jobs one after another. Useful for multi-user services. |
| `spark.cores.max`                                   | (not set)           | When running on a[standalone deploy cluster](https://colobu.com/2014/12/10/spark-configuration/spark-standalone.html) or a[Mesos cluster in "coarse-grained" sharing mode](https://colobu.com/2014/12/10/spark-configuration/running-on-mesos.html#mesos-run-modes), the maximum amount of CPU cores to request for the application from across the cluster (not from each machine). If not set, the default will be`spark.deploy.defaultCores` on Spark's standalone cluster manager, or infinite (all available cores) on Mesos. |
| `spark.mesos.coarse`                                | false               | If set to "true", runs over Mesos clusters in["coarse-grained" sharing mode](https://colobu.com/2014/12/10/spark-configuration/running-on-mesos.html#mesos-run-modes), where Spark acquires one long-lived Mesos task on each machine instead of one Mesos task per Spark task. This gives lower-latency scheduling for short queries, but leaves resources in use for the whole duration of the Spark job. |
| `spark.speculation`                                 | false               | If set to "true", performs speculative execution of tasks. This means if one or more tasks are running slowly in a stage, they will be re-launched. |
| `spark.speculation.interval`                        | 100                 | How often Spark will check for tasks to speculate, in milliseconds. |
| `spark.speculation.quantile`                        | 0.75                | Percentage of tasks which must be complete before speculation is enabled for a particular stage. |
| `spark.speculation.multiplier`                      | 1.5                 | How many times slower a task is than the median to be considered for speculation. |
| `spark.locality.wait`                               | 3000                | Number of milliseconds to wait to launch a data-local task before giving up and launching it on a less-local node. The same wait will be used to step through multiple locality levels (process-local, node-local, rack-local and then any). It is also possible to customize the waiting time for each level by setting`spark.locality.wait.node`, etc. You should increase this setting if your tasks are long and see poor locality, but the default usually works well. |
| `spark.locality.wait.process`                       | spark.locality.wait | Customize the locality wait for process locality. This affects tasks that attempt to access cached data in a particular executor process. |
| `spark.locality.wait.node`                          | spark.locality.wait | Customize the locality wait for node locality. For example, you can set this to 0 to skip node locality and search immediately for rack locality (if your cluster has rack information). |
| `spark.locality.wait.rack`                          | spark.locality.wait | Customize the locality wait for rack locality.               |
| `spark.scheduler.revive.interval`                   | 1000                | The interval length for the scheduler to revive the worker resource offers to run tasks (in milliseconds). |
| `spark.scheduler.minRegisteredResourcesRatio`       | 0                   | The minimum ratio of registered resources (registered resources / total expected resources) (resources are executors in yarn mode, CPU cores in standalone mode) to wait for before scheduling begins. Specified as a double between 0 and 1. Regardless of whether the minimum ratio of resources has been reached, the maximum amount of time it will wait before scheduling begins is controlled by config`spark.scheduler.maxRegisteredResourcesWaitingTime`. |
| `spark.scheduler.maxRegisteredResourcesWaitingTime` | 30000               | Maximum amount of time to wait for resources to register before scheduling begins (in milliseconds). |
| `spark.localExecution.enabled`                      | false               | Enables Spark to run certain jobs, such as first() or take() on the driver, without sending tasks to the cluster. This can make certain jobs execute very quickly, but may require shipping a whole partition of data to the driver. |



##### Security

| 属性名                                    | 缺省值 | 意义                                                         |
| :---------------------------------------- | :----- | :----------------------------------------------------------- |
| `spark.authenticate`                      | false  | Whether Spark authenticates its internal connections. See`spark.authenticate.secret` if not running on YARN. |
| `spark.authenticate.secret`               | None   | Set the secret key used for Spark to authenticate between components. This needs to be set if not running on YARN and authentication is enabled. |
| `spark.core.connection.auth.wait.timeout` | 30     | Number of seconds for the connection to wait for authentication to occur before timing out and giving up. |
| `spark.core.connection.ack.wait.timeout`  | 60     | Number of seconds for the connection to wait for ack to occur before timing out and giving up. To avoid unwilling timeout caused by long pause like GC, you can set larger value. |
| `spark.ui.filters`                        | None   | Comma separated list of filter class names to apply to the Spark web UI. The filter should be a standard[ javax servlet Filter](http://docs.oracle.com/javaee/6/api/javax/servlet/Filter.html). Parameters to each filter can also be specified by setting a java system property of: `spark.<class name of filter>.params='param1=value1,param2=value2'` For example: `-Dspark.ui.filters=com.test.filter1` `-Dspark.com.test.filter1.params='param1=foo,param2=testing'` |
| `spark.acls.enable`                       | false  | Whether Spark acls should are enabled. If enabled, this checks to see if the user has access permissions to view or modify the job. Note this requires the user to be known, so if the user comes across as null no checks are done. Filters can be used with the UI to authenticate and set the user. |
| `spark.ui.view.acls`                      | Empty  | Comma separated list of users that have view access to the Spark web ui. By default only the user that started the Spark job has view access. |
| `spark.modify.acls`                       | Empty  | Comma separated list of users that have modify access to the Spark job. By default only the user that started the Spark job has access to modify it (kill it for example). |
| `spark.admin.acls`                        | Empty  | Comma separated list of users/administrators that have view and modify access to all Spark jobs. This can be used if you run on a shared cluster and have a set of administrators or devs who help debug when things work. |



##### Spark Streaming

| 属性名                                         | 缺省值   | 意义                                                         |
| :--------------------------------------------- | :------- | :----------------------------------------------------------- |
| `spark.streaming.blockInterval`                | 200      | Interval (milliseconds) at which data received by Spark Streaming receivers is coalesced into blocks of data before storing them in Spark. |
| `spark.streaming.receiver.maxRate`             | infinite | Maximum rate (per second) at which each receiver will push data into blocks. Effectively, each stream will consume at most this number of records per second. Setting this configuration to 0 or a negative number will put no limit on the rate. |
| `spark.streaming.unpersist`                    | true     | Force RDDs generated and persisted by Spark Streaming to be automatically unpersisted from Spark's memory. The raw input data received by Spark Streaming is also automatically cleared. Setting this to false will allow the raw data and persisted RDDs to be accessible outside the streaming application as they will not be cleared automatically. But it comes at the cost of higher memory usage in Spark. |
| `spark.executor.logs.rolling.strategy`         | (none)   | Set the strategy of rolling of executor logs. By default it is disabled. It can be set to "time" (time-based rolling) or "size" (size-based rolling). For "time", use`spark.executor.logs.rolling.time.interval` to set the rolling interval. For "size", use`spark.executor.logs.rolling.size.maxBytes` to set the maximum file size for rolling. |
| `spark.executor.logs.rolling.time.interval`    | daily    | Set the time interval by which the executor logs will be rolled over. Rolling is disabled by default. Valid values are `daily`, `hourly`, `minutely` or any interval in seconds. See`spark.executor.logs.rolling.maxRetainedFiles` for automatic cleaning of old logs. |
| `spark.executor.logs.rolling.size.maxBytes`    | (none)   | Set the max size of the file by which the executor logs will be rolled over. Rolling is disabled by default. Value is set in terms of bytes. See`spark.executor.logs.rolling.maxRetainedFiles` for automatic cleaning of old logs. |
| `spark.executor.logs.rolling.maxRetainedFiles` | (none)   | Sets the number of latest rolling log files that are going to be retained by the system. Older log files will be deleted. Disabled by default. |



### 集群管理器

每种集群管理都有自己额外的配置参数. 可以在下面的页面中找到每种管理器相应的配置:

- [YARN](https://spark.apache.org/docs/latest/running-on-yarn.html#configuration)
- [Mesos](https://spark.apache.org/docs/latest/running-on-mesos.html)
- [Standalone Mode](https://spark.apache.org/docs/latest/spark-standalone.html#cluster-launch-scripts)



### 环境变量

有些Spark设置可以通过环境变量来设置, YARN中该配置不生效。

conf/spark-env.sh不存在默认值。可以从conf/spark-env.sh.template复制，来获取默认值。

| 环境变量         | 意义                                                         |
| :--------------- | :----------------------------------------------------------- |
| JAVA_HOME        | Location where Java is installed (if it's not on your default `PATH`). |
| PYSPARK_PYTHON   | Python binary executable to use for PySpark.                 |
| SPARK_LOCAL_IP   | IP address of the machine to bind to.                        |
| SPARK_PUBLIC_DNS | Hostname your Spark program will advertise to other machines. |



### 配置日志

Spark使用log4j记录日志. 你可以在conf增加一个log4j.properties文件. 其文件夹下有一个log4j的模版.



# 原理架构

[Spark内核.docx](D:\all\4bigdata\3spark\picture\Spark内核.docx)



Spark 相比 MR ，一个是全程在内存中运行，一个是多了有向无环图，一个是使用rdd这种弹性分布式数据集。



### 运行方式

local 模式

伪集群模式：一台机器，多个进程。

Standalone 模式：不同于hadoop的Standalone是单机模式，spark的Standalone是spark 自己分布式调度。

yarn client 模式：driver在client运行。交互式程序，spark-shell 或者 pyspark 都必须使用client模式。

yarn cluster 模式 ：driver在application master进程中运行。

mesos 模式



### 资源角色

![aaa](picture/aaa.png)

spark存在多种模式运行，每种模式涉及到的角色不尽相同，这里就不展开详细介绍其他角色了。

master 和 worker 是 spark 独立部署时负责资源调度的两个角色，这里不介绍。

核心角色是driver 和 executor，ApplicationMaster 是 yarn cluster 中的角色。



driver：spark的管理角色。创建SparkContext，DAGScheduler，TaskScheduler，生成job，调度task。

executor：spark的执行角色。存储RDD，执行一个job，并行或者顺序执行多个task。



ApplicationMaster：YARN AppMaster的一个实现，在yarn cluster模式，帮助 Driver 和 Executor 申请资源，监控执行，处理失败等。

`spark 在 yarn cluster上运行，所有容器都是一开始就开辟好，在整个运行周期始终存在的。因为spark不落地到磁盘，所以不能像MR一样一个容器一个容器的开辟。`



### 资源调度

![1574654673702](picture/1574654673702.png)

Driver（SparkContext） 启动后会和ResourceManager 通讯申请启动ApplicationMaster（SparkExecutorLauncher）。

ResourceManager 分配 container，在合适的NodeManager 上启动ApplicationMaster，负责向ResourceManager 申请 Executor 内存。

Executor 进程启动后会向Driver（SparkContext） 反向注册，Executor 全部注册完成后Driver 开始执行main 函数。进入运行计算过程



### 计算角色

Application：一次提交，一个Application，一个工作流。一个Application有多个job。

job ：MR的一个map reduce是一个job。spark的一个有向无环图就是一个job，一个有向无环图可以完成多个map reduce的功能，直到由action 算子触发。

stage：每个有向无环图，遇到一个 shuffle 算子，便划分一个stage（相当于MR 的一个map reduce）。

task：一个shuffle算子前的顺序执行的多个算子是一个task，一个stage RDD分区数就是task数量，一个线程一个task，同一个stage中，整个集群并行执行task的数量称之为并行度。并行度就是分区数。



一个job就是一个容器，stage，task都是发生在容器内部的计算，

job结束，要返回结果。stage结束，容器间要交换数据，一个task是一个RDD算子计算的这个容器的部分数据，是最小的一个任务。



### 运行计算

##### 提交过程

![img](picture/2019011311575574.png)

Application 向 RM 申请资源，开启任务。

driver 启动SparkContext，DAGScheduler，DAGScheduler。

SparkContext 启动调度程序（DAGScheduler 和 TaskScheduler） 

DAGScheduler：把作业分解为若干Stage，并构建DAG。

DAGScheduler：把每个阶段的任务提交到集群，进行任务调度。

executor执行任务。



##### DAG的构建

DAGScheduler

![image-20211123194232205](picture/image-20211123194232205.png)

spark任务包括两种任务类型：

shuffle map 任务：在一个RDD分区中进行计算，根据分区函数，将输出写到新的分区中。

result 任务：运行在最终阶段，将每个分区的结果返回给driver，driver汇总最终结果。



每次遇到一个shuffle算子，便会划分一个阶段。

含有shuffle 算子的阶段叫做shuffle map task，最后一个阶段叫result task。



##### 任务调度

TaskScheduler



1.读取executor列表，构建任务到executor的映射。

2.将任务分配给内核中的executor。

`分配任务，就近原则：同一进程，同一节点，同一机架，同一集群`



![图片1](picture\图片1.png)

每一列是一个job，job划分了stage，每个stage中的每个RDD（小黑方框）就是task，每个红框的分区数就是并行度。

每一列就是一个容器，每个RDD/task 就是一个线程。



##### 任务执行

1.executor确保JAR包和依赖都是最新的，需要更新的话会重新下载。

2.反序列化任务代码。

3.执行任务代码。

4.序列化执行结果，返回driver。



# Spark使用

### spark shell

```shell
#本地启动
bin/spark-shell --master local[2]

#集群启动
bin/spark-shell --master spark://node01:7077 --executor-memory 1g --total-executor-cores 2  

#yarn启动
spark-shell --master yarn-client --executor-memory 1G --total-executor-cores 2  
```

```shell
#示例
sc.textFile("file:///sparkdatas/wordcount.txt").flatMap(_.split(" ")).map((_,1)).reduceByKey(_ + _).collect()
```



### Scala 开发

开发的几种方式：

1、直接使用开发环境，local模式，在idea里面写完直接运行，全部使用windows环境。

2、开发环境使用idea，local模式开发，打成 jar 包提交到集群上运行。



在windows 本地开发，就需要一套和生产环境相同的开发环境：

1、语言：JDK、Scala SDK

2、pom文件依赖：spark、hadoop、hive 等。

3、配置文件：resource 下配置文件，可以从生产环境复制,复制后需要修改ip，host等。yarn-site.xml，core-site.xml，hdfs-site.xml，hive-site.xml



开发环境配置点：https://blog.csdn.net/Code_LT/article/details/124864685

1、安装JDK，安装maven，安装idea。

2、安装scala：安装Scala包，安装 idea的Scala插件，

3、创建maven项目，项目或者模块右键，要添加 scala 框架支持，

4、导入spark在内的pom依赖。

5.spark部分代码可能需要python2环境变量。

6.搭建hadoop环境支持，因为hadoop windos 有特殊文件，所以需要一套windos 版的hadoop。没有应该也没事。

```
https://blog.csdn.net/READLEAF/article/details/107946678

解压相应 Hadoop 版本到指定文件夹

需确保 bin 下有如下两个文件：hadoop.dll、winutils.exe, github上下载 winutils.exe

配置环境变量：编辑”Path” 变量，追加字段”%HADOOP_HOME%\bin;” 
```

7.其他问题

https://stackoverflow.com/questions/33161629/spark-yarn-client-on-windows-7-issue 问题中参数注释掉即可

https://mp.weixin.qq.com/s/Rwz5uAI-TfnTBpppsMTfBg



pom文件,可能需要处理依赖冲突。

```xml
<properties>
        <slf4j.version>1.7.22</slf4j.version>
        <log4j.version>1.2.17</log4j.version>
        <hadoop.version>3.0.0-cdh6.3.2</hadoop.version>
        <hive.version>2.1.1-cdh6.3.2</hive.version>
        <scala.version>2.11.12</scala.version>
        <spark.version>2.4.0-cdh6.3.2</spark.version>
</properties>
```

代码示例

```scala
object SparkFirst {

  def main(args: Array[String]): Unit = {

    val sparkConf:SparkConf = new SparkConf().setMaster("local[2]").setAppName("sparkLocalCount")
    val sparkContext = new SparkContext(sparkConf)
    sparkContext.setLogLevel("WARN")
      
    val file: RDD[String] = sparkContext.textFile("file:///F:\\wordcount\\wordcount.txt")
    val flatMap: RDD[String] = file.flatMap( x => x.split(" "))
    val map: RDD[(String, Int)] = flatMap.map(x => (x,1))
    val key: RDD[(String, Int)] = map.reduceByKey((x,y) => x)
    val by: RDD[(String, Int)] = key.sortBy(x => x._2,false)
    key.saveAsTextFile("file:///F:\\wordcount\\output2\\result.txt")

    sparkContext.stop()
  }
}
```



idea 中使用maven 打好 jar包，即可提交spark程序运行

```shell
./bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master yarn \
--deploy-mode cluster \
--driver-memory 1g \
--executor-memory 1g \
--executor-cores 1 \
--queue queueOne \
examples/target/scala-2.11/jars/spark-examples*.jar 10
```



# 核心抽象

Spark Core 的核心抽象是 **RDD**，主要切入点是 SparkContext，

Spark SQL 的核心抽象是 **DataFrame（SchemaRDD）**，主要切入点是 SQLContext，HiveContext （扩展了SQLContext，支持Hive特定的功能）。



在Spark2.0之后，**SparkSession** 封装了SparkContext、SQLContext、HiveContext。

在Spark2.0之后，**DataSet** 统一了RDD和DataFrame，https://www.cnblogs.com/mr-bigdata/p/14426049.html



# Spark Core

spark 实现了3种数据结构，分别是RDD（弹性分布式数据集），累加器（分布式共享只写变量），和广播变量（分布式共享只读变量）。



### RDD简介

RDD（Resilient Distributed Dataset）弹性分布式数据集 ，它是弹性的， 不可变，可分区，元素可并行计算 的分布式集合。

```
弹性
	存储的弹性：内存与磁盘存储的自动切换；
	计算的弹性：计算出错重试机制，调度引擎自动处理Stage的失败以及Task的失败；
	容错的弹性：基于血统的高效容错机制，Checkpoint和Persist可主动或被动触发，数据丢失可以自动恢复；
	分区的弹性：可根据需要重新分区，动态调整。

可分区
并行计算
分布式

不可变：RDD生成就不可以改变，但是可以生成新的RDD。
```



![图片1](picture\图片1.png)



RDD具有5大属性：

**A list of dependencies on other RDDs**：每个rdd都有一系列的父rdd，RDD之间的依赖关系。

**A list of partitions**：每个RDD都有一系列的分区。

**A function for computing each split**：每个切片的计算函数。

**A Partitioner for key-value RDDs**：分区函数，决定RDD如何分区。数据为KV数据时生效，如HashPartitioner。非KV时，由并行度决定。

**a list of preferred locations to compute each split**：一个列表，存储每个Partition的优先位置（数据所在位置），task会被优先分配到此位置。

记忆：竖着看一系列依赖，横着看一系列分区，每个切片的运算，重新分区，切片被分配到哪台机器上。



### RDD创建

```scala
//从集合（内存）创建
val rdd1 = sc.parallelize(Array(1,2,3,4,5,6,7,8))
var rdd1 = sc.makeRDD(1 to 100)

//从外部数据创建
val rdd2 = sc.textFile("/words.txt")

//从其他RDD转换
val rdd3=rdd2.flatMap(_.split(" "))
```



### 读取与保存

```python
#读取文本文件，每一行为一个元素
inputRdd = sc.textFile("/words.txt")
#保存文本文件
reslut.saveASTextFile(outputFile)

#读取多文本文件，键是文件名，值是文件内容。
inputRdd = sc.wholeTextFile("aaa/word-*.txt") 

#读取json数据，按文本读取，然后解析。
import json
data = inputRdd.map(lambda x: json.loads(x))
#保存json数据
data.map(lambda x: json.dump(x)).saveASTextFile(outputFile)
```

```scala
//读取MySQL数据
val rdd = new org.apache.spark.rdd.JdbcRDD (sc,
                                            () => {
                                                Class.forName ("com.mysql.jdbc.Driver").newInstance()
                                                java.sql.DriverManager.getConnection ("jdbc:mysql://lh:3306/rdd", "root", "123")
                                            },
                                            "select * from rddtable where id >= ? and id <= ?;",
                                            1,10,1,
                                            r => (r.getInt(1), r.getString(2)))

//写入MySQL数据
def insertData(iterator: Iterator[String]): Unit = {
    Class.forName ("com.mysql.jdbc.Driver").newInstance()
    val conn = java.sql.DriverManager.getConnection("jdbc:mysql://localhost:3306/rdd", "root", "admin")
    iterator.foreach(data => {
        val ps = conn.prepareStatement("insert into rddtable(name) values (?)")
        ps.setString(1, data) 
        ps.executeUpdate()
    })
}

val data = sc.parallelize(List("Female", "Male","Female"))
data.foreachPartition(insertData)
```



### 算子简介

transfermation：把现有RDD转换成新的RDD，返回值为RDD。有些 transfermation 算子会进行shuffle 操作。

Aciton：触发对RDD的计算，结果返回用户或者保存到外部存储器中。



`惰性计算/懒执行：在执行到Action算子之前， 所有的transfermation 算子都不会执行。因为要构建有向无环图后，真正执行`



### RDD transfermation

```python
#map，对rdd中的每一个对象/每一条数据 进行操作
maprdd = source.map(lambda x：x+x)

# Filter,为false就过滤
filterRDD = sourceFilter.filter(lambda x: "err" in x)

# Flatmap,先map再flat
flatMap = sourceFlat.flatMap(lambda x: x.split(" "))

# distinct()去重重复元素，开销很大，需要混洗、
filterRDD = sourceFilter.distinct()

# sample 采样，是否替换。
sourceFilter.sample(false,0.5)

# sortBy，排序,参数代表根据kv的值降序排序，
rdd1.sortBy(x=>x._2,false)
```



### 多RDD transfermation

```python
#union 合并两个RDD，不会去重
uniRDD = filterRDD.union(aaRDD)

#intersection 交集，去重，慢
bbbRDD = filterRDD.intersection(aaRDD)

#substract 第一个RDD的补集，需要shuffle
bbbRDD = filterRDD.substract(aaRDD)

#cartesian 求笛卡尔积
bbbRDD = filterRDD.cartesian(aaRDD)

#zip，将两个RDD变成key-valuRDD
pairdd = rdd1.zip(rdd2)
```



### RDD Action

```python
#take，返回driver10个元素
uniRDD.take(10)
uniRDD.top(10)
uniRDD.takeSample(10)

#collect，返回driver所有数据，会撑爆driver
uniRDD.collect()

#count 元素个数
uniRDD.count()

#reduce，可实现累加操作
sun = rDD.reduce(lambda x，y：x+y)

#foreach ，对每个元素执行函数操作
uniRDD.foreach(print)

#aggregate

```



### RDD statistics Action

```python
data.count()
data.mean()
data.sum()
data.max()
data.min()
data.first()
data.variance() #方差
data.stdev()	#标准差
```



### PairRDD transfermation

PairRDD：key-value的RDD

```python
# 创建
pairdd = source.map(lambda x：(x.split(" ")[0],x))

#keys(),获取键值对的所有键
pairdd.keys()

#values()，获取键值对的所有值
pairdd.values()

#mapValues(),对键值对的值进行map操作，键不变。
pairdd.mapValues(lambda x：x+x)

#flatMapValues,对value进行map，flat后，分别与自己的key组合
x = sc.parallelize([("a", ["x", "y", "z"]), ("b", ["p", "r"])])
x.flatMapValues(lambda x:x).collect()
[('a', 'x'), ('a', 'y'), ('a', 'z'), ('b', 'p'), ('b', 'r')]

#groupByKey()，对key进行分组
pairdd.groupByKey()

#sortByKey(),根据键排序
pairdd.sortByKey()
```



### 多PairRDD transferma

```python
#subtractByKey,删掉pairdd中与otherpair键相同的元素
pairdd.subtractByKey(otherpair)

#join，内连接
pairdd.join(otherpair)

#leftOuterJoin，左连接
pairdd.leftOuterJoin(otherpair)

#cogroup，相同的键值分组到一起。
pairdd.cogroup(otherpair)
```



### PairRDD Action

```python
#lookup,查看某键的所有值
pairdd.lookup(key)

#colletAsMap,
pairdd.colletAsMap()

#countByKey，每个键的值分别计数
pairdd.countByKey()

#reduceByKey(),对key分组，对每组value函数操作
pairdd.reduceByKey(lambda x,y：x+y)
```



### 算子优化

mapPartitions

```scala
// map操作的每条数据，mapPartitions 直接获取一个分区数据的迭代器进行操作，简化了打开数据库连接或创建随机数生成器等公共操作

val counts = words.map(word => (word, 1)).reduceByKey { (x, y) => x + y }
val counts1 = words.mapPartitions(it => it.map(word => (word, 1))).reduceByKey { (x, y) => x + y }
```



foreachPartition 

```scala
val list = new ArrayBuffer()

myRdd.foreach(record => {list += record})
rdd.foreachPartition(it => {it.foreach(r => {list += r})})
```



reduceByKey

```
reduceByKey 和 groupByKey 都可以使用的时候。使用 reduceByKey，会在map端提前进行combine操作
```



filter + coalesce

```
filter过滤会造成各个分区数据量差异变大。
coalesce 重新分区（当分区前后分区数量相差不大时，coalesce可以不进行shuffle）。
```



### RDD分区

分区就是并行度，可以在构建RDD时指定。

```scala
val dataRDD: RDD[Int] = sparkContext.makeRDD(List(1,2,3,4), 4)
val fileRDD: RDD[String] = sparkContext.textFile("input", 2)
```



在计算过程中，可以对KV RDD重新进行分区，repartition重新分区要shuffle。

```python
#repartition， 重新分区，使用随机数来重新分区
rdd.repartition()

#partitionBy，可以使用HashPartitioner和RangePartitioner或者自定义，使用自己的key来重新分区
rdd1.partitionBy(new org.apache.spark.HashPartitioner(2))
```



### RDD持久化

对多次使用的 RDD 进行持久化，否则每次使用RDD，都会重新计算。



Persist：不会改变血缘关系。懒执行。

```scala
//持久化与消除持久化，有多种级别：持久化内存，持久化磁盘，序列化内存等。
data.persist(StorageLevel.MEMORY_ONLY)
data.unpersist(blocking=true)
```



checkpoint ：将数据写入磁盘，血缘关系会从checkpoint重新开始。懒执行。

checkpoint 会在job action触发后，再启动一个job进行 checkpoint，所以可以 rdd.cache，避免调用两次。

```scala
// 设置检查点路径
sc.setCheckpointDir("./checkpoint1")

// 增加缓存,避免再重新跑一个 job 做 checkpoint 
wordToOneRdd.cache()
wordToOneRdd.checkpoint()

wordToOneRdd.collect().foreach(println)
```



### 广播变量

广播变量：分布式共享只读变量。

Driver 给所有Executor的每个Task分发一个只读的变量。

如果不使用广播变量的话，会给每个task都发这个外部变量。使用了广播变量，同一个executor的所有task会使用同一个外部变量，大大节约了内存。

```python
#定义广播变量
words_new = sc.broadcast(["scala", "java", "hadoop", "spark", "akka"])

#获取使用广播变量
data = words_new.value
source.map(lambda x：x in data)

#更新广播变量
words_new.unpersist()
words_new = sc.broadcast(["scala", "hadoop", "spark", "akka"])

#销毁广播变量，销毁之后可查询，不可使用
words_new.destroy()
```



### 累加器

累加器：分布式共享只写变量。使用 RDD分区数据更新 driver端的变量。

Driver 给所有Executor的每个Task分发一个需要修改的变量。修改完后，传回Driver端进行 merge。



系统累加器

```scala
val rdd = sc.makeRDD(List(1,2,3,4,5))

// 声明累加器
var sum = sc.longAccumulator("sum"); 

// 使用累加器，将累加器变量发给executor，task进行累加后，发回driver合并。
rdd.foreach(num => {sum += num})
```



自定义累加器

```scala
class WordCountAccumulator extends AccumulatorV2[String, mutable.Map[String, Long]]{
    var map : mutable.Map[String, Long] = mutable.Map()
    override def isZero: Boolean = { map.isEmpty}// 累加器是否为初始状态
    override def copy(): AccumulatorV2[String, mutable.Map[String, Long]] = new WordCountAccumulator // 复制累加器
    override def reset(): Unit = map.clear() // 重置累加器
    override def add(word: String): Unit = {map(word) = map.getOrElse(word, 0L) + 1L}
	override def merge(other: AccumulatorV2[String, mutable.Map[String, Long]]): Unit = { // 合并累加器
        val map1 = map
        val map2 = other.value
        map = map1.foldLeft(map2)
        ((innerMap, kv ) => {innerMap(kv._1) = innerMap.getOrElse(kv._1, 0L) + kv._2 innerMap})
    }
    override def value: mutable.Map[String, Long] = map  // 返回累加器的结果 （Out）
}
```



### 序列化

算子以外的代码都是在Driver 端执行, 算子里面的代码都是在 Executor 端执行。

算子内如果会用到算子外的数据，要保证数据（对象）是可以序列化的。否则报错。

序列化可以原生实现，通过Java的Serializable实现（序列化一切类），spark提供了Kryo的轻量化实现。



下列代码使用Kryo 代替spark 序列化的原生实现。

```scala
object serializable_Kryo {
    def main(args: Array[String]): Unit = { 
        // 替换默认的序列化机制，注册需要使用 kryo 序列化的自定义类
        val conf: SparkConf = new SparkConf().setAppName("SerDemo").setMaster("local[*]")
			.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer").registerKryoClasses(Array(classOf[Searcher])) 
        val sc = new SparkContext(conf)
        
        val rdd: RDD[String] = sc.makeRDD(Array("hello world", "hello atguigu", "atguigu", "hahah"), 2)
        val searcher = new Searcher("hello")
        val result: RDD[String] = searcher.getMatchedRDD1(rdd)
        result.collect.foreach(println)
    }
}
```



https://blog.51cto.com/u_12902538/3727315



### 血缘关系

http://www.xumenger.com/spark-core-5-rdd-dependency-20201125/

https://zhuanlan.zhihu.com/p/150990856

https://juejin.cn/post/6844904022088876040



# Spark SQL

### Spark SQL 概述

Spark SQL 起源于 **Shark**，Shark 修改了Hive中的内存管理、物理计划和执行三个模块，使得SQL语句直接运行在Spark上。

Shark 之后发展了**Hive on Spark** 和 **Spark SQL**，Hive on Spark 作为过渡，允许 Hive 的使用 Spark 作为执行引擎，这里不在赘述。



Spark SQL 与 Hive 完全解耦，和 Spark 紧密度更高。

数据兼容方面 不但兼容hive，还可以从RDD、parquet文件、JSON文件中获取数据。

性能优化方面 可以使用 Spark 更加个性的优化方式。

组件扩展方面 无论是SQL的语法解析器、分析器还是优化器都可以重新定义，进行扩展。



Spark SQL的切入口从SQLContext，HiveContext 从spark2.0发展成 **SparkSession**。核心抽象DataFrame 从spark1.6发展成 **DataSet**。



### Spark SQL 架构

https://zhuanlan.zhihu.com/p/107904954

![bc772f56dd5d51198166d0f4bea44915](picture/bc772f56dd5d51198166d0f4bea44915.jpg)



### DataFrame 简介

DataFrame是一种以RDD为基础的分布式数据集，类似于传统数据库中的二维表格。

DataFrame所表示的二维表数据集的每一列都带有名称和类型。而RDD无从得知所存数据元素的具体内部结构。

![image-20220127141129932](picture/image-20220127141129932.png)

使用dataset 进行编程，程序能否运行可以在编译期即可得知。而DataFrame需要在运行期才可以得知。

![img](picture/6796519-07884947dc9abedb.png)



创建DF

```

```

创建DS

```Scala
case class Person(name: String, age: Int)
val ds = List(Person("liudd", 18)).toDS()

val ds = Seq(1,2,3,4,5).toDS
```



### RDD&DF&DS互转

##### RDD 转换为 DF

通过 case class 创建 DataFrames（反射）

```Scala
//定义case class，相当于表结构
case class People(var name:String,var age:Int)

object TestDataFrame1 {
    def main(args: Array[String]): Unit = {
        val conf = new SparkConf().setAppName("RDDToDataFrame").setMaster("local")
        val sc = new SparkContext(conf)
        val context = new SQLContext(sc)
    	
        // 将本地的数据读入 RDD， 并将 RDD 与 case class 关联
        val peopleRDD = sc.textFile("E:\\666\\people.txt")
        					.map(line => People(line.split(",")(0), line.split(",")(1).trim.toInt))
    	
    	// 将RDD 转换成 DataFrames
        import context.implicits._
    	val df = peopleRDD.toDF
    
        //将DataFrames创建成一个临时的视图
        df.createOrReplaceTempView("people")
        
    	//使用SQL语句进行查询
    	context.sql("select * from people").show()
    }
}
```

过 structType 创建 DataFrames（编程接口）

```Scala
object TestDataFrame2 {
    def main(args: Array[String]): Unit = {
        val conf = new SparkConf().setAppName("TestDataFrame2").setMaster("local")
    	val sc = new SparkContext(conf)
    	val sqlContext = new SQLContext(sc)
        
    	val fileRDD = sc.textFile("E:\\666\\people.txt")
    	
        // 将 RDD 数据映射成 Row，需要 import org.apache.spark.sql.Row
    	val rowRDD: RDD[Row] = fileRDD.map(line => {
            val fields = line.split(",")
            Row(fields(0), fields(1).trim.toInt)
        })
        
    	// 创建 StructType 来定义结构
    	val structType: StructType = StructType(
            StructField("name", StringType, true) ::
            StructField("age", IntegerType, true) :: Nil
        )
        
    val df: DataFrame = sqlContext.createDataFrame(rowRDD,structType)
    }
}
```



#####  DF 转换为 RDD

```Scala
aaaRDD = df.rdd
```



#####  DF 转换为 DS

```Scala
case class Person(name: String, age: Int)
val ds = df.as[Person]
```



##### DS 转换为 DF

```Scala
val df2 = ds.toDF
```



### 读取保存

##### file

json，parquet，jdbc，orc，libsvm，csv，text

```Scala
val df = spark.read.json("test.json")
val df2 = spark.read.parquet("E:\\666\\users.parquet")

df1.write.json("E:\\111")
df1.write.format("parquet").mode(SaveMode.Append).save("E:\\444")
```



##### mysql

```Scala
// 从 MySQ 中加载数据
object SparkMysql {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName("TestMysql").setMaster("local")
    val sc = new SparkContext(conf)
    val sqlContext = new SQLContext(sc)

    val url = "jdbc:mysql://192.168.123.102:3306/hivedb"
    val table = "dbs"
    val properties = new Properties()
    properties.setProperty("user","root")
    properties.setProperty("password","root")
      
    val df = sqlContext.read.jdbc(url,table,properties)

    df.createOrReplaceTempView("dbs")
    sqlContext.sql("select * from dbs").show()

  }
}

// 往 MySQ 中写入数据
object Spark2Mysql {
    def main(args: Array[String]): Unit = { 
        val url ="jdbc:mysql://localhost:3306/userdb"
    	val tableName = "person"
    	val properties = new Properties()
    	properties.setProperty("user","root")
   	 	properties.setProperty("password","admin")

    	val jdbc: Unit = df.write.mode(SaveMode.Append).jdbc(url,tableName,properties)
    }
}
```



### SQL 语法编程

```scala
// 创建一个临时表
df.createOrReplaceTempView("people") //临时表，在spark session 范围内生效
df.createGlobalTempView("people") //全局临时表 application范围内生效

// 进行sql查询
spark.sql("SELECT * FROM people").show()
spark.sql("SELECT * FROM global_temp.people").show()
```



### DSL 语法编程

```python
#展示
df.show()
df.printSchema()

#选择列
df.select(col("name"), col("age").plus(1)).show()

#过滤
df.filter(col("age").gt(21)).show()

#分组
df.groupBy("age").count().show()

#计数
df.filter(col("age")>30).count()
```



### UDF & UDAF

```scala
//注册UDF函数
spark.udf.register("addName",(x:String)=> "Name:"+x) 

//将UDF函数用于SQL
spark.sql("Select addName(name),age from people").show()
```



```scala
//定义UDAF函数
case class User01(username:String,age:Long)
case class AgeBuffer(var sum:Long,var count:Long)

class MyAveragUDAF1 extends Aggregator[User01,AgeBuffer,Double]{ 
    
    override def zero: AgeBuffer = AgeBuffer(0L,0L)
    override def reduce(b: AgeBuffer, a: User01): AgeBuffer = {
        b.sum = b.sum + a.age 
        b.count = b.count + 1 
        b
    }
    override def merge(b1: AgeBuffer, b2: AgeBuffer): AgeBuffer = { 
        b1.sum = b1.sum + b2.sum
        b1.count = b1.count + b2.count
        b1
    }
    override def finish(buff: AgeBuffer): Double = buff.sum.toDouble/buff.count
    
    //DataSet 默认额编解码器，用于序列化，固定写法
    override def bufferEncoder: Encoder[AgeBuffer] = Encoders.product
    override def outputEncoder: Encoder[Double] = Encoders.scalaDouble
}

var myAgeUdaf1 = new MyAveragUDAF1

spark.udf.register("avgAge", functions.udaf(myAgeUdaf1))
spark.sql("select avgAge(age) from user").show
```



### Spark with Hive

https://juejin.cn/post/7109853101003440158



##### SparkSession + Hive Metastore

```scala
// 这种集成方式中，Spark 仅仅是“白嫖”了 Hive 的 Metastore
// 除了txt格式的表，其他格式暂不支持
import org.apache.spark.sql.SparkSession
import  org.apache.spark.sql.DataFrame
 
val spark = SparkSession.builder()
                   .config("hive.metastore.uris", s"thrift://hiveHost:9083")
                   .enableHiveSupport()
                   .getOrCreate()

val hiveDataBase: String = "app"
spark.sql("use " + hiveDataBase)

val df: DataFrame = spark.sql(“select * from salaries”)
```



##### Beeline + Spark Thrift Server

使用客户端 Spark SQL CLI 或者 Beeline，连接 Spark Thrift Server，访问Hive元数据（需配置）使用。



# Spark MLlib

Spark ml是基于DataFrame的API，Spark mllib是基于RDD的API。

Spark mllib现在处于维护模式，但算子仍然比 Spark ml 丰富。



MLlib库主要包括以下5个部分：

(1)   Utilities：线性代数、统计学、数据处理等工具。

(2)   特征处理：特征抽取、特征转换、特征降维、特征选择。

(3)   ML 算法：分类、回归、聚类、降维、协同过滤。

(4)   Pipeline：tools for constructing, evaluating, and tuning ML Pipelines。

(5)   持久化：模型与pipeline的保存、读取操作。



### 数据类型

##### Local Vector

```scala
import org.apache.spark.mllib.linalg.{Vector, Vectors}
object testVector {
    def main(args: Array[String]) {
        
        val vd: Vector = Vectors.dense(2, 0, 6) //建立密集向量
        
        val vs: Vector = Vectors.sparse(4, Array(0, 1, 2, 3), Array(9, 5, 2, 7)) //建立稀疏向量
    }
}
```



##### Local matrix

```scala
import org.apache.spark.mllib.linalg.{Matrix, Matrices}
object LocalMatrix {
    def main(args: Array[String]) {
        
        // Create a dense matrix ((1.0, 2.0), (3.0, 4.0), (5.0, 6.0))
        val dm: Matrix = Matrices.dense(3, 2, Array(1.0, 3.0, 5.0, 2.0, 4.0, 6.0))
    
        // Create a sparse matrix ((9.0, 0.0), (0.0, 8.0), (0.0, 6.0))
        val sm: Matrix = Matrices.sparse(3, 2, Array(0, 1, 3), Array(0, 2, 1), Array(9, 6, 8))
       
        println(dm(2,1))
    }
}
```



##### Labled Point

```scala
import org.apache.spark.mllib.linalg.{Vector, Vectors}
import org.apache.spark.mllib.regression.LabeledPoint
object testLabeledPoint {
    def main(args: Array[String]) {
        
        val vd: Vector =  Vectors.dense(2, 0, 6) //建立密集向量
        val pos = LabeledPoint(1, vd) //对密集向量建立标记点
        
        println(pos.features)                              
        println(pos.label)                                  
    }
}
```

```scala
import org.apache.spark._
import org.apache.spark.mllib.util.MLUtils
import org.apache.spark.mllib.regression.LabeledPoint
import org.apache.spark.rdd.RDD
object LabeledPoint2Test {
    def main(args: Array[String]) {
        val conf = new SparkConf().setMaster("local").setAppName("testLabeledPoint2")
        val sc = new SparkContext(conf) 
        
        val mu: RDD[LabeledPoint] = MLUtils.loadLibSVMFile(sc, "D://a.txt")
        mu.foreach(println) 
    }
}
```



Distributed matrix：行矩阵（RowMatrix）、行索引矩阵（IndexedRowMatrix）、三元组矩阵（CoordinateMatrix）和 分块矩阵（BlockMatrix）四种。



##### 行矩阵

```scala
import org.apache.spark._
import org.apache.spark.mllib.linalg.{Vector, Vectors}
import org.apache.spark.mllib.linalg.distributed.RowMatrix
object RowMatrixTest {
    def main(args: Array[String]) {
        val conf = new SparkConf().setMaster("local").setAppName("testRowMatrix")    
        val sc = new SparkContext(conf)          
        
        //每一个行就是一个本地向量。
        val rdd = sc.textFile("D://C04//RowMatrix.txt").map(_.split(' ').map(_.toDouble)).map(line => Vectors.dense(line))  
        
        val rm = new RowMatrix(rdd)                  
        println(rm.numRows())                    
        println(rm.numCols())                    
    }
}
```



##### 行索引矩阵

```scala
import org.apache.spark._
import org.apache.spark.mllib.linalg.distributed.{IndexedRow, RowMatrix, IndexedRowMatrix}
import org.apache.spark.mllib.linalg.{Vector, Vectors}
object IndexedRowMatrixTest {
    def main(args: Array[String]) {
        val conf = new SparkConf().setMaster("local").setAppName("testIndexedRowMatrix")         
        val sc = new SparkContext(conf) 
        
        val rdd = sc.textFile( "D://C04//RowMatrix.txt").map(_.split(' ').map(_.toDouble))
        			.map(line => Vectors.dense(line)).map((vd) => new IndexedRow(vd.size,vd))  // 行索引矩阵       
        
        val irm = new IndexedRowMatrix(rdd)                             
        println(irm.getClass)                                          
        println(irm.rows.foreach(println))                                
  }
}
```



##### 三元组矩阵

```scala
import org.apache.spark._
import org.apache.spark.mllib.linalg.{Vector, Vectors}
import org.apache.spark.mllib.linalg.distributed.{CoordinateMatrix, MatrixEntry}
object CoordinateRowMatrixTest {
    def main(args: Array[String]) {
        val conf = new SparkConf().setMaster("local").setAppName("testCoordinateRowMatrix")      
        val sc = new SparkContext(conf)  
        
        val rdd = sc.textFile( "D://C04//RowMatrix.txt").map(_.split(' ').map(_.toDouble))
        			.map(vue => (vue(0).toLong,vue(1).toLong,vue(2)))
        			.map(vue2 => new MatrixEntry(vue2_1,vue2_2,vue2_3))  //MatrixEntry是一个Tuple类型的元素，包含行、列和元素的值
       
        val crm = new CoordinateMatrix(rdd)                        
        println(crm.entries.foreach(println))                      
        println(crm.numRows())
        println(crm.numCols())
    }
}
```



##### 分块矩阵

```scala
// 矩阵分块由((int,int),matrix)元祖所构成，（int,int）是该部分矩阵所处的矩阵的索引位置，Matrix表示该索引位置上的子矩阵
import org.apache.spark._
import org.apache.spark.mllib.linalg.distributed.{CoordinateMatrix, MatrixEntry,BlockMatrix}
import org.apache.spark.storage.StorageLevel
object BlockMatrixTest {
    def main(args: Array[String]) {
        val conf = new SparkConf().setMaster("local").setAppName("testCoordinateRowMatrix")        
        val sc = new SparkContext(conf)                        
        
        val rdd = sc.textFile( "D://C04//RowMatrix.txt").map(_.split(' ').map(_.toDouble))
        			.map(vue => (vue(0).toLong,vue(1).toLong,vue(2))).map(vue2 => new MatrixEntry(vue2 _1,vue2 _2,vue2 _3))
        
        val crm = new CoordinateMatrix(rdd)                     
        val matA: BlockMatrix =crm.toBlockMatrix().cache()
        
        matA.validate() //检验，分块不正确抛出异常
        val ata = matA.transpose.multiply(matA) // Calculate A^T A.
    }
}
```



### 统计实现

##### 均值和方差

```scala
import org.apache.spark._
import org.apache.spark.mllib.linalg.Vectors
import org.apache.spark.mllib.stat.Statistics
import org.apache.spark.{SparkConf, SparkContext}
object SummaryTest {
    def main(args: Array[String]) {
        val conf = new SparkConf() .setMaster("local") .setAppName("testSummary") 
        val sc = new SparkContext(conf) 
    
        val rdd = sc.textFile( "D://BigData//Workspace//IDEAScala//SparkDemo//src//MLLIB//C04//testSummary.txt")
        .map(_.split(' ').map(_.toDouble)).map(line => Vectors.dense(line))  
    
        val summary = Statistics.colStats(rdd) //获取Statistics实例
        println(summary.mean) //计算均值
        println(summary.variance) //计算标准差
        //还可以计算 max，min，mean，variance，非零数，总计数。
    }
}
```



##### 距离计算

```scala
import org.apache.spark._
import org.apache.spark.mllib.linalg.Vectors
import org.apache.spark.mllib.stat.Statistics
import org.apache.spark.{SparkConf, SparkContext}
object SummaryTest {
    def main(args: Array[String]) {
        val conf = new SparkConf() .setMaster("local") .setAppName("testSummary") 
        val sc = new SparkContext(conf) 
    
        val rdd = sc.textFile( "D://BigData//Workspace//IDEAScala//SparkDemo//src//MLLIB//C04//testSummary.txt")
        .map(_.split(' ').map(_.toDouble)).map(line => Vectors.dense(line))  
    
        val summary = Statistics.colStats(rdd) //获取Statistics实例
        println(summary.normL1)     //计算曼哈段距离
        println(summary.normL2)     //计算欧几里得距离
    }
}
```



##### 相关系数计算

```scala
import org.apache.spark.mllib.linalg._
import org.apache.spark.mllib.stat.Statistics
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}
object Correlation_3 {
    def main(args: Array[String]): Unit = {
        val conf = new SparkConf().setMaster("local").setAppName("testSummary") 
        val sc = new SparkContext(conf) 
        
        val seriesX: RDD[Double] = sc.parallelize(Array(1, 2, 3, 3, 5))  
        val seriesY: RDD[Double] = sc.parallelize(Array(11, 22, 33, 33, 555))
        val correlation: Double = Statistics.corr(seriesX, seriesY, "pearson") //皮尔逊相关系数
        val correlation: Double = Statistics.corr(rddX, rddY, "spearman") //斯皮尔曼相关系数

        val data: RDD[Vector] = sc.parallelize(Seq( 
            Vectors.dense(1.0, 10.0, 100.0),
            Vectors.dense(2.0, 20.0, 200.0),
            Vectors.dense(5.0, 33.0, 366.0))) 
        val correlMatrix: Matrix = Statistics.corr(data, "pearson")
    }
}
```











































# 调优

详见picture/Spark调优.pdf



# 面试

### spark快的原因

```
使用了DAG，减少了shuffle次数。
每个map reduce阶段通过内存交换数据。
持久化机制，避免了重复计算。
```



### RDD容错机制

RDD首先使用cache机制读取数据，然后会使用checkpoint机制读取数据，最后会用Lineage找到父RDD重新计算数据。



谱系图/血缘关系

```
窄依赖:每一个父RDD的Partition被子RDD的一个Partition使用。
宽依赖:每一个父RDD的Partition被子RDD的多个Partition使用。

RDD的Lineage会记录RDD单个块上执行的单个操作，当该RDD的部分分区数据丢失时，它可以根据这些信息来重新运算和恢复丢失的数据分区。
```

持久化

```
将结果在内存或者磁盘中进行缓存，避免重复计算。lineage不变。
```

checkpoint

```
在文件系统HDFS上进行持久化，RDD变成checkpointRDD，lineage 改变。

调用sparkContext的setCheckpointDir方法，设置一个容错文件系统目录HDFS。
在RDD所处的job运行结束后，启动一个单独的job 将RDD 持久化到文件系统。
```



### 数据倾斜

定义：在算子shuffle过程中会出现数据倾斜问题，是由于不同的 key数据量不同导致的不同 task 数据量不同。

表现：某一task明显慢于其他task执行速度。某一task反复出现OOM问题。注意是否是数据量大引起的。

定位：逻辑分析shuffle算子是否会倾斜。根据log文件定位到数据倾斜的shuffle算子。

```
数据源聚合，根本上避免shuffle。
提高reduce 端并行度，可以直接尝试。

先用sample(false,0.x)采样key，找出倾斜的key把数据集拆成倾斜的部分和不倾斜的部分。
不倾斜的部分走正常流程。

fileter、聚合、join类算子造成的倾斜分别处理。
其中join类算子分为小表join大表，大表join大表单独倾斜key，大表join大表很多倾斜key。

空值倾斜，可以过滤，也可以当作特殊key按以上方法处理
```



数据源聚合，根本上避免shuffle

```
直接在hive表中 把同一key的value 拼接成字符串，在spark中用map处理。
数值数据可以直接在hive中累积计算。

在原始数据中，增大key的粒度，key数量减少，task数据量增加，是否会改善数据倾斜要看场景。
```



提高reduce 端并行度，比较简单，可以开始的时候尝试。

```
rdd ,直接在shuffle算子中传入并行度。
df , 使用 spark.sql.shuffle.partitions 参数设置。
并不能从根本上改变数据倾斜问题，同一个key数据量大，还是不行，只是一个task处理的key更少了。
```



避免filter导致的数据shuffle，见算子优化。



针对聚合类的shuffle（groupByKey、reduceByKey ），随机key双重聚合。

```
给key加随机前缀，打散聚合，去掉前缀，再次聚合。
join类的shuffle没有效果。
```



小表join大表，将reduce join转化为map join

```
普通join是reducejoin，即先根据key进行shuffle，再在reduce端进行join。
广播小RDD（将RDD collect到driver端，再广播） + map 算子 实现mapjoin。
```



两表join，对单独的倾斜key 进行打散

```
数据量大时，可以先sample 10%。通过countByKey()找到引起倾斜的一两个key。
把这个kye单独做个rdd，单独join。
根据saprk机制，rdd中只有一个key，会默认打散。
```



大表join大表，倾斜key很多，随机数扩容进行join

```
扩容：选择一个 RDD，使用 flatMap 进行扩容N倍，对每条数据的 key 添加数值前缀（1~N 的数值），将一条数据映射为N条数据；
稀释：选择另外一个 RDD，进行map 映射操作，每条数据的 key 都打上一个随机数作为前缀（1~N 的随机数）；
将两个处理后的RDD，进行 join 操作。不会改变操作前的join结果。

稀释RDD，reduce端结果不变，主要是为了匹配扩容RDD的key；扩容RDD，增加了task数量，需要更大的内存，但是也打散了key。join数量不变。

优化：只对倾斜key 做随机数扩容进行join的操作。然后与不倾斜key join结果进行union。
```







































































