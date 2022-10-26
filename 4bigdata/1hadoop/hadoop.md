B站尚硅谷视频

书籍： [Hadoop权威指南（第四版）中文版.pdf](picture\Hadoop权威指南（第四版）中文版.pdf) 

官网：http://hadoop.apache.org/



# 安装

Hadoop部署模式有：本地模式/单机模式（一个JVM进程）、伪分布模式（一台机器，多个进程模拟）、完全分布式模式、HA完全分布式模式。

`完全分布式模式需要配置slave及相关文件。以下安装是完全分布式安装`



[apache27安装.docx]: picture/apache27安装.docx

[apache33安装.docx]: picture/apache30安装.docx



网速测试

```shell
python -m SimpleHTTPServer
```

磁盘压测

```shell
#写入测试
hadoop jar /export/servers/hadoop-2.6.0-cdh5.14.0/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.6.0-cdh5.14.0.jar TestDFSIO  -write -nrFiles 10 -fileSize 10MB
hdfs dfs -text /benchmarks/TestDFSIO/io_write/part-00000

#读取测试
hadoop jar /export/servers/hadoop-2.6.0-cdh5.14.0/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.6.0-cdh5.14.0.jar TestDFSIO -read -nrFiles 10 -fileSize 10MB
hdfs dfs -text /benchmarks/TestDFSIO/io_read/part-00000

#清除测试数据
hadoop jar /export/servers/hadoop-2.6.0-cdh5.14.0/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.6.0-cdh5.14.0.jar TestDFSIO -clean
```

常用端口号说明

| 端口名称                  | Hadoop2.x   | Hadoop3.x         |
| ------------------------- | ----------- | ----------------- |
| NameNode内部通信端口      | 8020 / 9000 | 8020 /  9000/9820 |
| NameNode HTTP UI          | 50070       | 9870              |
| MapReduce查看执行任务端口 | 8088        | 8088              |
| 历史服务器通信端口        | 19888       | 19888             |



# 配置

实际使用的配置文件中不能有注释

安装好集群后，通过 http://192.168.75.101:8188/conf 查看。



### hadoop-env.sh

```
export JAVA_HOME=/export/servers/jdk1.8.0_141
```



### works

```shell
cd /export/servers/hadoop-2.6.0-cdh5.14.0/etc/hadoop
vim works

#删除localhost，不能有空行空格
node01
node02
node03
```



### core-site.xml

https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/core-default.xml

```shell
cd /export/servers/hadoop-2.6.0-cdh5.14.0/etc/hadoop
vim core-site.xml
```

```xml
<configuration>

    <!-- 文件系统的端口-->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop102:8020</value>
	</property>
    
    <!-- hadoop数据的临时存储目录-->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/module/hadoop-3.1.3/data</value>
	</property>

</configuration>
```



### hdfs-site.xml

https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml

```shell
cd /export/servers/hadoop-2.6.0-cdh5.14.0/etc/hadoop
vim hdfs-site.xml
```

```xml
<configuration>
    
	<!-- nn web端访问地址-->
	<property>
        <name>dfs.namenode.http-address</name>
        <value>hadoop102:9870</value>
    </property>
    
	<!-- 2nn web端访问地址-->
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>hadoop104:9868</value>
    </property>
    
    <!-- HDFS副本的数量 -->
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
    
    
     <!-- namenode fsimage 本地存储的位置 -->
	<property>
		<name>dfs.namenode.name.dir</name>
		<value>file:///export/servers/hadoop-2.6.0-cdh5.14.0/hadoopDatas/namenodeDatas</value>
	</property>
    
    <!--  namenode edits 本地存储的位置  -->
    <property>
		<name>dfs.namenode.edits.dir</name>
		<value>file:///export/servers/hadoop-2.6.0-cdh5.14.0/hadoopDatas/dfs/nn/edits</value>
	</property>
    
    <!-- namenode checkpoint 本地存储的位置  -->
	<property>
		<name>dfs.namenode.checkpoint.dir</name>
		<value>file:///export/servers/hadoop-2.6.0-cdh5.14.0/hadoopDatas/dfs/snn/name</value>
	</property>
    <property>
		<name>dfs.namenode.checkpoint.edits.dir</name>
		<value>file:///export/servers/hadoop-2.6.0-cdh5.14.0/hadoopDatas/dfs/nn/snn/edits</value>
	</property>    
   
    <!--  dataNode 数据块 本地存储的位置，多块硬盘都要配置上 -->
	<property>
		<name>dfs.datanode.data.dir</name>
		<value>file:///export/servers/hadoop-2.6.0-cdh5.14.0/hadoopDatas/datanodeDatas</value>
	</property>

    
    <!-- nn监控心跳线程数，数量为20*ln台数 -->
    <property>
    	<name>dfs.namenode.handler.count</name>
    	<value>21</value>
	</property>
    
    <!-- 心跳掉线时间 = 2*dfs.namenode.heartbeat.recheck-interval + 10*dfs.heartbeat.interval = 2*5分钟 + 10*3秒 -->
    
    <!-- nn检查心跳时间 -->
    <property>
        <name>dfs.namenode.heartbeat.recheck-interval</name>
        <value>300000</value>
    </property>
    
    <!-- dn心跳间隔 -->
	<property>
    	<name>dfs.heartbeat.interval</name>
    	<value>3</value>
	</property>
    
    
    <!-- 每隔一小时checkpoint一次 -->
	<property>
    	<name>dfs.namenode.checkpoint.period</name>
    	<value>3600</value>
	</property>

	<!-- 每1百万操作checkpoint一次 -->
	<property>
    	<name>dfs.namenode.checkpoint.txns</name>
  		<value>1000000</value>
	</property>
    
    <!-- 1分钟检查一次操作次数 -->
	<property>
  		<name>dfs.namenode.checkpoint.check.period</name>
  		<value>60</value>
	</property >
    
</configuration>
```



### mapred-site.xml

https://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml

```shell
cd /export/servers/hadoop-2.6.0-cdh5.14.0/etc/hadoop
vim mapred-site.xml
```

```xml
<configuration>
     
    <!-- MR程序运行位置 -->
    <property>
        <name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
    
    <!-- 历史服务器web端地址 -->
	<property>
    	<name>mapreduce.jobhistory.webapp.address</name>
    	<value>hadoop102:19888</value>
	</property>
    
    <!-- 设置-->
    
	<!-- 当MapTask完成的比例达到该值后才会为ReduceTask申请资源。默认是0.05 -->
	<property>
  		<name>mapreduce.job.reduce.slowstart.completedmaps</name>
  		<value>0.05</value>
	</property>
    
    <!-- 开启JVM重用，属于同一job的顺序执行的x个task可以共享一个JVM。减少启动jvm的开销 -->
    <property>
  		<name>mapred.job.reuse.jvm.num.tasks</name>
  		<value>1</value>
	</property>
 
    <!-- 运算-->
    
    <!-- 环形缓冲区大小，默认100m，越大溢写次数越少，MR越快 -->
	<property>
  		<name>mapreduce.task.io.sort.mb</name>
  		<value>100</value>
	</property>
    
	<!-- 环形缓冲区溢写阈值，默认0.8，越大溢写次数越少，MR越快 -->
	<property>
  		<name>mapreduce.map.sort.spill.percent</name>
  		<value>0.80</value>
	</property>

	<!-- merge合并次数，默认10个，越大，MR越快 -->
	<property>
 		<name>mapreduce.task.io.sort.factor</name>
  		<value>10</value>
	</property>
    
    <!-- 每个Reduce去Map中拉取数据的并行数。默认值是5 -->
	<property>
  		<name>mapreduce.reduce.shuffle.parallelcopies</name>
  		<value>5</value>
	</property>

	<!-- Buffer中的数据达到多少比例开始写入磁盘，默认值0.66。 -->
	<property>
  		<name>mapreduce.reduce.shuffle.merge.percent</name>
  		<value>0.66</value>
	</property>
	
    <!-- 资源-->
	<!-- map任务内存，运行每个map任务实际用到的内存，一般在执行任务前动态设置-->
	<property>
  		<name>mapreduce.map.memory.mb</name>
  		<value>-1</value>
	</property>
    
	<!-- map任务CPU核数，运行每个map任务实际用到的CPU核数 -->
	<property>
  		<name>mapreduce.map.cpu.vcores</name>
  		<value>1</value>
	</property>
    
    <!-- reduce任务内存，运行每个reduce任务实际用到的内存，一般为map任务的2倍，一般在执行任务前动态设置-->
	<property>
  		<name>mapreduce.reduce.memory.mb</name>
  		<value>-1</value>
	</property>
    
    <!-- map任务CPU核数，运行每个map任务实际用到的CPU核数 -->
	<property>
  		<name>mapreduce.reduce.cpu.vcores</name>
  		<value>2</value>
	</property>

</configuration>
```



### yarn-site.xml

https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-common/yarn-default.xml

```shell
cd /export/servers/hadoop-2.6.0-cdh5.14.0/etc/hadoop
vim yarn-site.xml
```

ResourceManager

```xml
<configuration>
    
     <!-- ResourceManager供客户端访问的地址-->
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>hadoop01:8032</value>
    </property>
    
    <!-- ResourceManager供ApplicationMaster访问的地址 -->
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>hadoop01:8030</value>
    </property>
    
    <!-- ResourceManager供NodeManager访问的地址 -->
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>hadoop01:8031</value>
    </property>
   
     <!-- ResourceManager的对外web ui地址-->
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>hadoop101:8088</value>
    </property>

    <!-- 配置调度器，默认容量调度 -->
    <property>
        <name>yarn.resourcemanager.scheduler.class</name>
        <value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler</value>
    </property>

</configuration>
```

NodeManager

```xml
<configuration>

	<!-- NodeManager使用内存数，NodeManager所在机器分配给NodeManager的内存 -->
	<property>
		<name>yarn.nodemanager.resource.memory-mb</name>
		<value>4096</value>
	</property>
    
    <!-- NodeManager的CPU核数，NodeManager所在机器分配给NodeManager的CPU核数 -->
	<property>
		<name>yarn.nodemanager.resource.cpu-vcores</name>
		<value>4</value>
	</property>
    
    <!-- 容器最小内存，per node 4GB|256MB,4GB-8GB|512MB,8GB-24GB|1024MB,24 GB|2048 MB-->
	<property>
		<name>yarn.scheduler.minimum-allocation-mb</name>
		<value>1024</value>
	</property>

	<!-- 容器最大内存，推荐同 NodeManager内存 -->
	<property>
		<name>yarn.scheduler.maximum-allocation-mb</name>
		<value>2048</value>
	</property>

	<!-- 容器最小CPU核数 -->
	<property>
		<name>yarn.scheduler.minimum-allocation-vcores</name>
		<value>1</value>
	</property>

	<!-- 容器最大CPU核数-->
	<property>
		<name>yarn.scheduler.maximum-allocation-vcores</name>
		<value>2</value>
	</property>

	<!-- 虚拟内存和物理内存设置比例, -->
	<property>
		<name>yarn.nodemanager.vmem-pmem-ratio</name>
		<value>2.1</value>
	</property>

</configuration>
```



# HDFS

Hadoop Distribute File System 是一个分布式文件系统，通过统一的命名空间目录树来定位文件。

Hadoop文件系统除了HDFS外，还有本地文件系统和amazon S3文件系统。

![image-20210304154517017](picture/image-20210304154517017.png)

**设计目标**

单用户写入，不可修改

流式访问：一次写入，多次读取

延迟略高：毫秒级不适合

小文件数量限制：受限于namenode内存，百万到千万个。



**HDFS概念**

数据块：磁盘进行读写的最小单位。512B

文件系统块：数据块整数倍。

HDFS块（block）：128M，但是1M文件只占1M空间。每个文件的块数量一般小于MR的节点数量。

块缓存：一个块可缓存在一个datanode内存中，缓存池用于管理缓存权限和资源使用。



### NN & DN

**client**：客户端提供一个接口操作HDFS



**Namenode**：

管理文件系统的命名空间，维护系统树上所有文件和目录 。存储元数据与文件到数据块的映射。

通过磁盘上 fsimage 和edit 两个文件进行管理。不保存块的位置信息。

- `fsimage 镜像文件： 命名空间目录结构（目录文件属性、存储信息）。`

- `Edits 编辑日志：   存放的是 Hadoop 文件系统的所有更改操作。`

这些信息同时也保存在Namenode的内存中。



**datanode**：

存储数据本身和元数据（数据块的长度，块数据的校验和，以及时间戳）。

周期性（1小时）的向NameNode上报所有的块信息。

心跳机制：3秒一次。超过10分钟无心跳，节点不可用。心跳返回NN命令，如复制或删除。



**second namenode**

定期合并 images 和 edit。防止edits过大，也可作为元数据备份。



### 元数据维护过程

启动

```
Namenode启动
Fsimage->内存    edits操作后->内存
文件块位置 DataNode信息 -> 定时汇报 内存
```

写数据时元数据变化

```
先写进edits文件,再进内存元数据
```

合并元数据：即Secondnarynamenode 定时的把edits文件中的操作转化成Fsimage的目录树

```
原因：为了保证启动时Fsimage和edits进内存时尽可能快，需要edits小，所以需要定时的把edits文件中的操作转化成Fsimage的目录树。

过程：Secondnarynamenode 发出合并请求, 设为一个checkpoint ,之后发生上图过程

时机： 
默认服务启动时 进行checkpoint
默认一小时合并一次			dfs.namenode.checkpoint.period=3600
默认100w条数据合并一次	dfs.namenode.checkpoint.txns=1000000
```

![image-20210225153730804](picture/image-20210225153730804.png)



### HDFS的高可用

hadoop2.x实现了HDFS的高可用。

[高可用.docx]: /picture高可用.docx



架构修改：

1. 活动namenode和备用namenode 都在内存维护一份元数据。

2. 活动namenode和备用namenode需要共享 Edits，只有Active NameNode可以做写 Edits。

   `共享 Edits：QJM通过3个journal节点实现。或者通过NFS实现。`

3. datanode需要发送两份数据块处理报告。

4. client使用特定机制处理namenode失效问题，



故障切换与规避：

使用zokeeper的zkfailover监控确保仅有一个活动的namenode。也可手动切换。

如果因为网络，两个namenode都活动：同一时间QJM仅允许一个namenode写入数据（通过znode锁）。使用ssh规避命令杀死namenode进程。避免脑裂发生



### HDFS Shell

```shell
#获取帮助
hadoop fs -help

#创建文件夹
hadoop fs -mkdir -p /test/input 

#查看文件夹下文件
hdfs dfs -ls /test/input
#查看文件夹下所有文件
hdfs dfs -ls -R /
#显示文件(夹)大小
hdfs dfs -count /
hdfs dfs -count /aa.txt
#显示可用空间
hadoop fs -df -h /
#显示文件内容
hadoop fs -cat /test/input/aaa.txt
#显示文件末尾
hadoop fs -tail /test/input/aaa.txt

#将文件从本地复制到hdfs
hadoop fs -put /root/install.log  /test/input 
#将文件从本地移动到hdfs
hadoop fs -moveFromLocal aaa.txt /test/input 
#将文件从hdfs复制到本地
hadoop fs -get /test/input/aaa.txt /root
#本地文件追加到hdfs文件末尾
hadoop fs -appendToFile /root/install.log /test/input/aaa.txt
#将文件从hdfs复制到hdfs
hadoop fs -cp /test/input/aaa.txt /
#将文件从hdfs移动到hdfs
hadoop fs -mv /user/log.txt /

#删除文件或文件夹
hadoop fs -rm -r /log.txt

#修改文件所属
hadoop fs -chown root:root /test/input/aaa.txt
#修改文件权限
hadoop fs -chmod 777 /test/input/aaa.txt

#文件数量限制
hdfs dfsadmin -setQuota 2 /test/input
#清除数量限制
hdfs dfsadmin -clrQuota /test/input
#空间大小限制
hdfs dfsadmin -setSpaceQuota 4k /test/input
#清除空间限制
hdfs dfsadmin -clrSpaceQuota /test/input
#查看配额限制
hdfs dfs -count -q -h /test/input

#小文件合并
hdfs dfs -getmerge /config/*.xml ./hello.xml
```



### Hadoop java

```java
FileSystem fileSystem;

//初始化
@Before
public void init() throws Exception{
    //读取数据由平台上的协议确定
    URI uri = new URI("hdfs://192.168.*.*:9000");
    Configuration conf = new Configuration();
    fileSystem = FileSystem.get(uri, conf);
}

//查看目录
@Test
public void Catalog() throws Exception{
    Path path = new Path("/poker");
    FileStatus fileStatus = fileSystem.getFileStatus(path);
    System.out.println("*************************************");    
    System.out.println("文件根目录: "+fileStatus.getPath()); 
    System.out.println("这文件目录为：");
    for(FileStatus fs : fileSystem.listStatus(path)){ 
        System.out.println(fs.getPath()); 
    } 
}
  
//浏览文件
@Test
public void look() throws Exception{
    Path path = new Path("/core-site.xml");
    FSDataInputStream fsDataInputStream = fileSystem.open(path);
    System.out.println("*************************************");
    System.out.println("浏览文件：");
    int c;
    while((c = fsDataInputStream.read()) != -1){
        System.out.print((char)c);
    }
    fsDataInputStream.close();
}

//上传文件
@Test
public void upload() throws Exception{
    Path srcPath = new Path("C:/Users/Administrator/Desktop/hadoop/hadoop.txt");  
    Path dstPath = new Path("/");  
    fileSystem.copyFromLocalFile(false, srcPath, dstPath);
    fileSystem.close(); 
}

@Test
public void putFileToHDFS() throws IOException, InterruptedException, URISyntaxException {
	FileInputStream fis = new FileInputStream(new File("e:/banhua.txt"));
	FSDataOutputStream fos = fileSystem.create(new Path("/banhua.txt"));
	IOUtils.copyBytes(fis, fos, configuration);
    
	IOUtils.closeStream(fos);
	IOUtils.closeStream(fis);
    fileSystem.close();
}

//下载文件
@Test
public void testCopyToLocalFile() throws IOException, InterruptedException, URISyntaxException{
    fs.copyToLocalFile(false, new Path("/banzhang.txt"), new Path("e:/banhua.txt"), true);
    fs.close();
}

@Test
public void download() throws Exception{
    InputStream in = fileSystem.open(new Path("/hadoop.txt"));  
    OutputStream out = new FileOutputStream("E://hadoop.txt");  
    IOUtils.copyBytes(in, out, 4096, true);  
}

//删除文件
@Test
public void delete() throws Exception{
    Path path = new Path("hdfs://192.168.*.*:9000/hadoop.txt");
    fileSystem.delete(path,true);
}

//文件重命名
@Test
public void testRename() throws IOException, InterruptedException, URISyntaxException{
    fileSystem.rename(new Path("/banzhang.txt"), new Path("/banhua.txt"));
	fs.close();
}

// 判断是文件还是文件夹
@Test
public void testListStatus() throws IOException, InterruptedException, URISyntaxException{
	FileStatus[] listStatus = fileSystem.listStatus(new Path("/"));
	for (FileStatus fileStatus : listStatus) {
        if (fileStatus.isFile()) {
            System.out.println("f:"+fileStatus.getPath().getName());
        }
    }
    fs.close();
}
```



### HDFS文件读取过程

![image-20210225162519027](picture/image-20210225162519027.png)





2.客户端向namenode请求两次,第一次请求是否有权限,第二次文件每个块的具体位置。

3.就近读取：离client近的 block , 同样距离选最近一次心跳的datanode。

45.并行读取：同时从多个datanode进行读取。

6.读完进行 checksum 验证 ，不通过重新请求namenode，从其他机器再读。



### HDFS文件写入过程

![image-20210225161047435](picture/image-20210225161047435.png)

1.HDFS客户端让 分布式文件系统 创建文件。

2.分布式文件系统让namenode 在命名空间创建文件，执行检查 是否有权限,写入的文件夹是否存在。然后创建数据块。

3.FSData OutputStream 将数据分成数据包，写入数据队列。然后将数据流式传入到 datanonode中。

5.FSData OutputStream 也维护着ACK ，收到确认回执，才会从数据队列删除数据包。



写操作中，遇到datanode故障：

1.将数据包全部放入数据队列最前端。

2.找到新的datanode，并通知namenode 。

3.删除管线中的故障datanode，与新datanode组成新管线。

4.执行写入过程3.5。之后namenode副本不足时也会检查。



### HDFS 安全模式

当hdfs处于安全模式时，只可以读数据，无法增删改及其他任何操作。

nn启动时，进入安全模式，dn汇报block，达到副本数标准比例（配置文件配置）后，退出。

```shell
#手动进入安全模式
hdfs dfsadmin -safemode enter

#手动退出安全模式
hdfs dfsadmin -safemode leave

#查看安全模式状态
hdfs dfsadmin -safemode get
```



### Hadoop Archives

把多个小文件归档成一个文件。归档后无法修改。

不起压缩作用，只是节约了namespace的空间。

har作为MR输入时，不会提高效率。



创建 Archives：一个xxx.har就是一个文件系统

```shell
hadoop archive -archiveName name -p <src> <dest>
hadoop archive -archiveName test.har -p /input /output
```

查看 Archives

```powershell
#查看的是hdfs文件系统，会显示index，part等
hadoop fs -ls /output/test.har

#可以用下列方法查看原文件
hadoop fs har:///output/test.har
hadoop fs har://node01:9000/output/test.har
```

解压 Archives

```shell
hadoop fs -cp har:///output/test.har hdfs:/aaa
hadoop distcp har:///output/test.har hdfs:/aaa
```



### 回收站

java程序删除的文件默认不放入回收站



core-site.xml

```xml
<!-- 启用回收站，回收站保存文件60分钟后删除 -->
<property>
	<name>fs.trash.interval</name>
	<value>60</value>
</property>

<!-- 回收站每10分钟检查删除哪些文件 -->
<property>
    <name>fs.trash.checkpoint.interval</name>
	<value>10</value>
</property>
```



回收站路径：/user/atguigu/.Trash/

```shell
#恢复回收站文件
hadoop fs -mv /user/atguigu/.Trash/Current/input /input

#清空回收站
hadoop fs -expunge
```



### 纠删码

3.0新特性，纠删码，需要至少5台机器组成的集群。

https://www.cnblogs.com/itlz/p/14090193.html



3副本策略，3个相同数据块中两个是冗余的。

5机器纠删码，使用3个不同的数据块和生成的两个校验块，保证任意3个块可用即可恢复。

3副本策略，3个块需要9个存储。5机器纠删码，3个块需要5个存储。



减少了磁盘存储，但是恢复数据的时候增大了计算和IO开销，适用于平时用的少，不容易出故障的冷数据。

```shell
#帮助
hdfs ec

#查看当前支持的纠删码策略
hdfs ec -listPolicies

#开启对RS-3-2-1024k策略的支持
hdfs ec -enablePolicy  -policy RS-3-2-1024k

#对/input设置策略，纠删码策略针对某个目录
hdfs ec -setPolicy -path /input -policy RS-3-2-1024k
```



### 异构存储

对不同用途的数据采用不同的存储策略



存储类型：

​	RAM_DISK：内存

​	SSD：固态

​	DISK：磁盘

​	ARCHIVE：不特指某一介质，存储归档的极冷数据。



存储策略：

​	Lazy_Persist ：数据存储在内存中，其他副本存储在DISK中。

​	One_SSD：数据存储在SSD中，其他副本存储在DISK中。

​	Warm：数据存储在DISK中，其他数据存储方式为ARCHIVE。

​	All_SSD：全部数据都存储在SSD中。

​	Hot：全部数据存储在DISK中

​	Cold：全部数据以ARCHIVE的方式保存



```shell
#查看当前有哪些存储策略可以用
hdfs storagepolicies -listPolicies

#为指定路径（数据存储目录）设置指定的存储策略
hdfs storagepolicies -setStoragePolicy -path xxx -policy xxx

#获取指定路径（数据存储目录或文件）的存储策略
hdfs storagepolicies -getStoragePolicy -path xxx

#取消存储策略，之后策略同上级目录，都没有策略就是hot
hdfs storagepolicies -unsetStoragePolicy -path xxx

#查看文件块的分布
bin/hdfs fsck xxx -files -blocks -locations

#查看集群节点
hadoop dfsadmin -report
```



### 小文件过多优化

小文件过多，是namenode维护的空间过大，速度变慢。



1. 数据采集的时候合成小文件。MR程序读文件的时候使用 combinfileinputformat，将小文件使用一个分片。
2. 使用MR程序合并小文件。
3. 使用Hadoop Archives。



### 磁盘&集群 均衡

增加一块空白硬盘时，可以执行磁盘数据均衡命令

```shell
#生成均衡计划
hdfs diskbalancer -plan hadoop103
#执行均衡计划
hdfs diskbalancer -execute hadoop103.plan.json

#查看当前均衡任务的执行情况
hdfs diskbalancer -query hadoop103
#取消均衡任务
hdfs diskbalancer -cancel hadoop103.plan.json
```

```shell
#集群均衡，存储利用率不超过10%
sbin/start-balancer.sh -threshold 10
```



# YARN

### YARN角色

![image-20220121145222307](picture/image-20220121145222307.png)

resource manager：负责所有资源的监控分配和管理，处理client，监控node manager，监控启动Application Manager。

node manager：负责每个节点启动和监控容器。



Application master：向RM申请一次资源，对每个应用的资源进行管理。对每个应用的监控与容错。

container：用于执行特定应用程序的进程，每个容器都有内存，CPU 等资源限制。



### 运行机制

![img](picture/19588241-20f80b693f04d35e.png)

1.客户端 请求 resource manager，运行一个application master进程。

2.在容器中启动application master。

3.application master 应用决定做什么，再起一个container，或者计算返回结果。



**资源请求**

请求顺序：数据所在节点开辟容器，数据所在节点的同机架节点，数据所在节点的同集群节点。

动态请求：application master 可以在开始开启所有容器（spark），也可在之后再开启容器（MR）。



### 调度机制

FIFO调度器：先进先出。小作业会被阻塞，不适合共享集群。

容量调度器：留下一部分资源给小作业用。会降低大作业效率。

公平调度器：每个队列动态的平分资源，队列内每个作业也平分资源。



CDH默认使用公平调度器。原生YARN默认容量调度器。



延迟调度：

数据所在节点资源不够时，会等待几秒，仍不够，会使用其他节点。

容量调度器和公平调度器都支持延迟调度。



##### 容量调度器

分成几个队列（部分资源）。在一个队列内，使用FIFO调度。

弹性队列：队列资源不够用，但仍有其他资源时，会调用其他资源。

队列容量：设置合适值，防止侵占其他队列容量。



通过capacity-scheduler.xml进行队列资源配置。

用哪个队列要应用指定，默认default队列。



##### 公平调度器

队列在用户提交第一个应用时动态创建。

公平并不是每个队列完全50%，也可指定权重。

队列内默认公平调度，但也可以FIFO。



队列放置：

有指明队列时，应用优先放入指明队列中，其次放入用户名队列中，最后放入默认队列中。

没有指明队列时，会以用户名创建队列。



主导资源公平性：

当应用使用多种资源时，其占比最大资源（内存占总内存）作为他的主导资源。

主导资源占比的比例，决定公平调度的分配。



抢占：

资源被用尽时，可设置抢占，强制释放占用超额资源的作业，然后重新启动所有。



### 资源配置

yarn内存是实际内存，不得大于所在节点的内存。

yarn核数属于虚拟核，可以用来监控集群是否有机器挂掉，也可以监控是否有任务抢占了过多的计算资源。



```
资源计算：设置符合下列等式，资源利用率会比较高
yarn.nodemanager.resource.cpu-vcores / yarn.scheduler.maximum-allocation-vcores  = 按照核数 最少启动的容器数
yarn.nodemanager.resource.memory-mb / 按照核数 最少启动的容器数  = 每个容器的最大内存
```



集群使用率低

Yarn 显示资源使用率达到 100%，而集群节点内存，CPU 等资源却使用不是很高。
目前集群可能存在的问题是，每个 Container 分配的资源过高，实际任务并不需要这么多资源，从而出现了资源被分配完，但是使用率低的情况。

解决思路是增加容器数量：减少每个容器使用的内存，增加整个yarn集群的虚拟核数。

```
1.增大 yarn.nodemanager.resource.cpu-vcores： 48
2.调小 Map 任务内存 mapreduce.map.memory： 1.5G
3.调小 Reduce 任务内存 mapreduce.reduce.memory.mb 1.5G
4.调小 Map 任务最大堆栈 mapreduce.map.java.opts.max.heap 1.2G
5.调小 Reduce 任务最大堆栈 mapreduce.reduce.java.opts.max.heap 1.2G 

不过注意，对于某些对于内存需求较高的任务，需要单独设定，保证不出现outofmemory 的情况。
```



### 常用命令

```shell
#列出所有Application
yarn application -list

#根据Application状态过滤：ALL、NEW、NEW_SAVING、SUBMITTED、ACCEPTED、RUNNING、FINISHED、FAILED、KILLED
yarn application -list -appStates FINISHED

#Kill掉Application：
yarn application -kill application_1612577921195_0001

#查询Application日志
yarn logs -applicationId application_1612577921195_0001

#查询Container日志
yarn logs -applicationId application_1612577921195_0001 -containerId container_1612577921195_0001_01_000001

#列出所有Container
yarn container -list appattempt_1612577921195_0001_000001

#打印Container状态
yarn container -status container_1612577921195_0001_01_000001

#列出所有节点
yarn node -list -all

#打印队列信息
yarn queue -status default

#加载队列配置
yarn rmadmin -refreshQueues
```



### 高可用

/picture/高可用.docx 



# MapReduce

移动计算不移动数据



### 原理流程

一个MR 程序运行在YARN 上是一个应用，YARN 的 MR Application master开启map容器和reduce容器，进行调度，属于YARN内容，不再介绍。



[MapReduce原理优化.docx](D:\all\4bigdata\1hadoop\picture\MapReduce原理优化.docx)

MapReduce框架通常由三个操作（或步骤）组成：

`Map`：每个工作节点将 `map` 函数应用于本地数据，并将输出写入临时存储。

`Shuffle`：工作节点根据输出键（由 `map` 函数生成）重新分配数据，目的是属于一个键的所有数据都位于同一个工作节点上。发生在map容器。

`Reduce`：工作节点现在并行处理每个键的每组输出数据。



![image-20220124142933519](picture/image-20220124142933519.png)

![image-20220124143254448](picture/image-20220124143254448.png)



1.数据源的每个切片被`Map` 获取，在每台机器上并行处理。

2.向环形缓冲区（内存）中写入数据。

3.从环形缓冲区溢写到磁盘，发生`Partitioner` ，可以让 `Map` 根据`Key` 进行分区（把数据分成几部分），在分区内部排序，几个分区对应几个`Reduce`。

4.`Combiner` 是一个本地化的 `reduce` 操作，它与 `map` 在同一个主机上进行。可以提前做一个简单的合并重复key值的操作。不改变分区数量，减少文件数量。

5.下载数据到Reduce所在机器。

6.`Reduce`读取数据，进行归并和排序，进行reduce操作，输出。



### 代码示例

数据流向：kv -- map -- kv -- reduce -- kv 。map是对每一个数据执行map方法，reduce是对同key的每组数据执行reduce方法。

```java
package com.atguigu.mapreduce;
import java.io.IOException;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class WordcountMapper extends Mapper<LongWritable, Text, Text, IntWritable>{
    
	Text k = new Text();
	IntWritable v = new IntWritable(1);
    
	@Override
	protected void map(LongWritable key, Text value, Context context)	throws IOException, InterruptedException {
        
		String line = value.toString();// 1 获取一行
		String[] words = line.split(" ");// 2 切割
		for (String word : words) {
			k.set(word);
			context.write(k, v);// 3 输出
		}
	}
}
```

```java
package com.atguigu.mapreduce.wordcount;
import java.io.IOException;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class WordcountReducer extends Reducer<Text, IntWritable, Text, IntWritable>{
    int sum;
	IntWritable v = new IntWritable();

	@Override
	protected void reduce(Text key, Iterable<IntWritable> values,Context context) throws IOException, InterruptedException {
		
		sum = 0;// 1 累加求和
		for (IntWritable count : values) {
			sum += count.get();
		}
		
        v.set(sum);
		context.write(key,v);// 2 输出
	}
}
```

```java
package com.atguigu.mapreduce.wordcount;
import java.io.IOException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordcountDriver {
	public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
		// 1 获取配置信息以及封装任务
		Configuration configuration = new Configuration();
		Job job = Job.getInstance(configuration);
		// 2 设置jar加载路径
		job.setJarByClass(WordcountDriver.class);
		// 3 设置map和reduce类
		job.setMapperClass(WordcountMapper.class);
		job.setReducerClass(WordcountReducer.class);
		// 4 设置map输出
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(IntWritable.class);
		// 5 设置最终输出kv类型
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);
		// 6 设置输入和输出路径
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
		// 7 提交
		boolean result = job.waitForCompletion(true);
		System.exit(result ? 0 : 1);
	}
}
```



### 天龙八步

![image-20220124144650950](picture/image-20220124144650950.png)

![image-20220124145031417](picture/image-20220124144854286.png)

![image-20220124145031417](picture/image-20220124145031417.png)

![image-20220124145356882](picture/image-20220124145356882.png)

![image-20220124145421560](picture/image-20220124145421560.png)



### join实现

reduce join：map负责给不同来源数据打标签，reduce根据join字段分组，合并。

reduce join：reduce端压力过大，且容易数据倾斜。

```java
public class RJoinMapper extends Mapper<LongWritable,Text,Text,Text> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        /*super.map(key, value, context);*/
        //获取文件的名称
        FileSplit inputSplit =(FileSplit)context.getInputSplit();
        String name = inputSplit.getPath().getName();
        String[] split = value.toString().split(",");
        //判断当前是订单数据
        if(name.contains("order")){
            //订单数据第三个字段是产品id
            context.write(new Text(split[2]),value);
        }else{
            //否则是产品数据
            context.write(new Text(split[0]),value);
        }
    }
}

public class RJoinReducer  extends Reducer<Text,Text,Text,Text>{
    @Override
    protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
        /*super.reduce(key, values, context);*/
        String pro = "";
        String order = "";

        for (Text value : values) {
            String lines = value.toString();
            //如果是产品数据，以p开头
            if(lines.startsWith("p")){
                pro = lines;
            }else {
                //否则是订单数据
                order =lines;
            }
        }

        context.write(key,new Text(pro +"\t"+order));
    }
}
```



map join：小表join大表，map端缓存小表，直接join大表。

map join：reduce join对100个数据块在shuffle的分组，map join可能只需要70个，因为小表的数据块在map阶段就分组好了。减缓了reduce压力和数据倾斜

```java
ublic class MJoinMapper extends Mapper<LongWritable,Text,Text,Text> {
    //定义Hashmap接收数据
    Map<String,String> pro = new HashMap<String,String>();

    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
        /*super.setup(context);*/
        //通过上下文获取配置文件
        Configuration configuration = context.getConfiguration();
        //获取块缓存的地址
        URI[] cacheFiles = DistributedCache.getCacheFiles(configuration);
        //获取文件系统对象
        FileSystem fileSystem = FileSystem.get(cacheFiles[0], configuration);
        //获取数据的输入流
        FSDataInputStream inputStream = fileSystem.open(new Path(cacheFiles[0]));
        //处理流对象
        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
        String lines = "";
        //读取数据
        while ((lines = bufferedReader.readLine())!=null){
            String[] split = lines.split(",");
            //将数据放到map对象中
            pro.put(split[0],lines);

        }
    }

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        /*super.map(key, value, context);*/
        //value的值 订单的一行值
        String lines = value.toString();
        String[] split = lines.split(",");
        //产品id
        String pid = split[2];
        //返回产品的一行数据
        String product = pro.get(pid);
        context.write(new Text(pid),new Text(product+"\t"+lines));
    }
}


public class MJoinMain extends Configured implements Tool{
    @Override
    public int run(String[] args) throws Exception {
        //添加缓存文件,块缓存文件一定要在hdfs上，否则加载不到
        DistributedCache.addCacheFile(new URI("hdfs://node01:8020/data/mapjoin/cache/pdts.txt"),super.getConf());

        Job job = Job.getInstance(super.getConf(), "sadfasfd");

        job.setJarByClass(MJoinMain.class);
        //1
        job.setInputFormatClass(TextInputFormat.class);
        TextInputFormat.addInputPath(job,new Path("file:///H:\\map端join\\input\\orders.txt"));
        //2
        job.setMapperClass(MJoinMapper.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Text.class);
        //7
        //8
        job.setOutputFormatClass(TextOutputFormat.class);
        TextOutputFormat.setOutputPath(job,new Path("file:///H:\\传智播客\\Hadoop课程资料\\4、大数据离线第四天\\map端join\\output12"));

        boolean b = job.waitForCompletion(true);

        return b?0:1;
    }

    public static void main(String[] args) throws Exception {
        int run = ToolRunner.run(new Configuration(), new MJoinMain(), args);
        System.exit(run);
    }
}
```



### 数据倾斜

检查数据倾斜的原因：

1、空值过多：直接过滤空值。

2、Partitioner 分区倾斜：改变Partitioner 分区方法，自定义分区。

3、Partitioner 分区倾斜：改变key（把key加随机数打散，在根据其他方法聚合）。



优化但不根本解决的方法：

1、在map阶段提前处理数据：使用Combiner，使用MapJoin代替reducejoin。

2、增加reduce个数。



# 面试问题

datanode怎么保证数据完整性？

```
当DataNode读取Block的时候，它会计算CheckSum。
如果计算后的CheckSum，与Block创建时值不一样，说明Block已经损坏。
```



写数据过程中，优先传到最近的datanode中，怎么计算节点距离？

```
节点距离：两个进程所在节点到达最近的共同祖先的距离总和。
根 -- 集群 -- 机架 -- 节点 
```

基于机架感知的副本存储节点选择

```
第一个副本在客户端所处节点上
第二个副本在第一个副本同机架的节点上
第三个副本在第一个副本不同机架的节点上
```



namenode或者datanode挂了怎么办？

```
datanode挂了一台，直接配置一台新的datanode即可。
namenode高可用，自动切换即可。
namenode非高可用，启动一台新的namenode，将SecondaryNameNode元数据文件复制过来？
```



怎么解决1%的map任务特别慢的问题

```
推测执行机制
	1.为拖后腿的任务启动一个备份任务，谁快用用谁。
	2.每个Task只能有一个备份任务。当前Job已完成的Task必须不小于5%
	3.不能启用推测执行机制情况：任务间存在严重的负载倾斜；特殊任务，比如任务向数据库中写数据。
```



# 企业操作

### 数据压缩

不能直接使用HDFS进行压缩，要结合HDFS Java程序、MR、Hive等在读写数据时同HDFS结合进行压缩。

HDFS可以识别打开压缩文件，但是不能压缩和解压。



以下均为HDFS 同MR结合进行压缩。

MR使用压缩原则：计算密集型Job少压缩，IO运算型Job多压缩。

MR压缩位置：3个KV的位置都可以进行压缩。

| 压缩格式 | hadoop自带？ | 算法    | 文件扩展名 | 是否可切分 | 换成压缩格式后，原来的程序是否需要修改 |
| -------- | ------------ | ------- | ---------- | ---------- | -------------------------------------- |
| DEFLATE  | 是，直接使用 | DEFLATE | .deflate   | 否         | 和文本处理一样，不需要修改             |
| Gzip     | 是，直接使用 | DEFLATE | .gz        | 否         | 和文本处理一样，不需要修改             |
| bzip2    | 是，直接使用 | bzip2   | .bz2       | 是         | 和文本处理一样，不需要修改             |
| LZO      | 否，需要安装 | LZO     | .lzo       | 是         | 需要建索引，还需要指定输入格式         |
| Snappy   | 否，需要安装 | Snappy  | .snappy    | 否         | 和文本处理一样，不需要修改             |

Gzip：压缩完的每一个文件都比数据块小时，使用Gzip  。

bzip2：压缩速度慢，压缩速率高。

LZO： 支持切片，文件越大，有点越明显。 大部分企业都选用LZO压缩。



HDFS支持LZO压缩格式

将lzo的jar包放到 share/hadoop/common下

```shell
/opt/module/hadoop-3.1.3/share/hadoop/common/hadoop-lzo-0.4.20.jar
```

core-site.xml 配置

```xml
<configuration>
    <property>
        <name>io.compression.codecs</name>
        <value>
            org.apache.hadoop.io.compress.GzipCodec,
            org.apache.hadoop.io.compress.DefaultCodec,
            org.apache.hadoop.io.compress.BZip2Codec,
            org.apache.hadoop.io.compress.SnappyCodec,
            com.hadoop.compression.lzo.LzoCodec,
            com.hadoop.compression.lzo.LzopCodec
        </value>
    </property>

    <property>
        <name>io.compression.codec.lzo.class</name>
        <value>com.hadoop.compression.lzo.LzoCodec</value>
    </property>
</configuration>
```

为lzo文件手动创建索引，使lzo文件可切片，是一种优化

```
hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/common/hadoop-lzo-0.4.20.jar \
com.hadoop.compression.lzo.DistributedLzoIndexer /input/bigtable.lzo
```



MR开启压缩

```java
package com.atguigu.mapreduce.compress;
import java.io.IOException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.io.compress.BZip2Codec;	
import org.apache.hadoop.io.compress.CompressionCodec;
import org.apache.hadoop.io.compress.GzipCodec;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCountDriver {
	public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
		Configuration conf = new Configuration();

		// 开启map端输出压缩
		conf.setBoolean("mapreduce.map.output.compress", true);
		// 设置map端输出压缩方式
		conf.setClass("mapreduce.map.output.compress.codec", BZip2Codec.class,CompressionCodec.class);

		Job job = Job.getInstance(conf);
		job.setJarByClass(WordCountDriver.class);
		job.setMapperClass(WordCountMapper.class);
		job.setReducerClass(WordCountReducer.class);
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(IntWritable.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
        
        // 设置reduce端输出压缩开启
		FileOutputFormat.setCompressOutput(job, true);
		// 设置压缩的方式
	    FileOutputFormat.setOutputCompressorClass(job, BZip2Codec.class); 
        
		boolean result = job.waitForCompletion(true);
		System.exit(result ? 0 : 1);
	}
}
```



### HDFS 联邦

/picture/高可用.docx 



### 集群迁移

/picture/集群迁移.doc



# 源码解析

[源码解析V3.3.docx]: picture/源码解析V3.3.docx











































