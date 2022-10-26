官网：https://zookeeper.apache.org/doc/current/index.html

教程：https://www.runoob.com/w3cnote_genre/zookeeper/page/2

https://mp.weixin.qq.com/s/W6QgmFTpXQ8EL-dVvLWsyg



# 简介

zookeeper本质是一个**分布式**的**小文件**(每个文件最大1M)**存储系统** ,辅助其他的框架解决应用系统当中的**一致性**问题。



zookeeper作用：https://mp.weixin.qq.com/s/W6QgmFTpXQ8EL-dVvLWsyg

- 数据发布/订阅
- 负载均衡
- master 管理：避免脑裂问题，采用奇数节点过半机制
- 集群管理：分布式配置管理(solr的配置集中管理)
- 统一命名服务(dubbo)
- 分布式协调/通知
- 分布式锁
- 分布式队列



特性：

全局数据一致：每个 server 保存一份相同的数据副本。

可靠性：如果消息被其中一台服务器接受，那么将被所有的服务器接受。

全局有序：如果在一台服务器上消息 a 在消息 b 前发布，则在所有 Server 上消息 a 都将在消息 b 前被发布。

偏序有序：如果一个消息 b 在消息 a 后被同一个发送者发布， a 必 将排在 b 前面。

数据更新原子性：一次数据更新要么成功（半数以上节点成功），要么失败，不存在中间状态；

实时性： Zookeeper 保证客户端将在一个时间间隔范围内获得服务器的更新信息，或者服务器失效的信息。



# 安装

下载：http://archive.apache.org/dist/zookeeper/

上传：zookeeper-3.4.5-cdh5.14.0.tar.gz上传到   /export/softwares

解压：

```
cd /export/softwares
zookeeper-3.4.5-cdh5.14.0.tar.gz -C ../servers/
```

配置：

```
cd /export/servers/zookeeper-3.4.5-cdh5.14.0/conf/
cp zoo_sample.cfg zoo.cfg
mkdir -p /export/servers/zookeeper-3.4.5-cdh5.14.0/zkdatas/
vim  zoo.cfg
```

```
改：dataDir=/export/servers/zookeeper-3.4.5-cdh5.14.0/zkdatas/
放开：autopurge.snapRetainCount=3
放开：autopurge.purgeInterval=1
增：
server.1=node01:2888:3888
server.2=node02:2888:3888
server.3=node03:2888:3888
```

myid：

```
cd /export/servers/zookeeper-3.4.5-cdh5.14.0/zkdatas/
echo 1 > myid
```

分发

```
scp -r  /export/servers/zookeeper-3.4.5-cdh5.14.0/ node02:/export/servers/
scp -r  /export/servers/zookeeper-3.4.5-cdh5.14.0/ node03:/export/servers/

第二台：
cd /export/servers/zookeeper-3.4.5-cdh5.14.0/zkdatas/
echo 2 > myid

第三台：
cd /export/servers/zookeeper-3.4.5-cdh5.14.0/zkdatas/
echo 3 > myid
```

启动：

```
所有机器：
/export/servers/zookeeper-3.4.5-cdh5.14.0/bin/zkServer.sh start
/export/servers/zookeeper-3.4.5-cdh5.14.0/bin/zkServer.sh status
```



zookeeper 的三个端口作用:

`1、2181 : 对 client 端提供服务`

`2、2888 : 集群内机器通信使用`

`3、3888 : 选举 leader 使用`



# 架构

![image-20210210160643703](picture\image-20210210160643703.png)

### Leader

leader 是事务（写操作）的唯一调度和处理者，来保证集群事务处理的顺序性。

所有写操作被统一转发给leader ，leader 决定顺序，然后分发进行操作。



### Follower

处理客户端非事务（读操作） 请求；转发事务请求给 Leader。



### Observer

增加并发处理读请求的能力。不参与leader选举投票，不会对集群产生任何影响。



# 数据模型

### 数据结构

森林模型 / 树形层次结构

![zknamespace](picture/zknamespace.jpg)

图中的每个节点称为一个 Znode，**Znode兼具文件和目录两种特点**。每个 Znode 由 3 部分组成:

`① stat：此为状态信息, 描述该 Znode 的版本, 权限等信息`

`② data：与该 Znode 关联的数据`

`③ children：该 Znode 下的子节点`

**Znode通常以 KB 为大小单位**

**Znode 具有原子性操作**，读操作将获取与节点相关的所有数据，写操作也将 替换掉节点的所有数据。

**Znode路径是绝对的,由斜杠字符来开头，也要是唯一的。**

`字符串"/zookeeper"用以保存管理 信息，比如关键配额信息`



### 节点属性

```
get 节点名称
```

dataVersion：数据版本号，每次对节点进行 set 操作，dataVersion 的值都会增加 1

cversion ：子节点的版本号。当 znode 的子节点有变化时，cversion 的值就会增加 1。

aclVersion ：权限控制的版本号。

ctime：节点创建时的时间戳。

mtime：节点最新一次更新发生时的时间戳。

mZxid ：Znode 被修改的事务 id，即每次对 znode 的修改都会更新 mZxid。

`对于 zk 来说，每次的变化都会产生一个唯一的事务 id，zxid（ZooKeeper Transaction Id）。通过 zxid，可以确定更新操作的先后顺序。例如，如果 zxid1小于 zxid2，说明 zxid1 操作先于 zxid2 发生。**zxid 对于整个 zk 都是唯一的**，即使操作的是不同的 znode。`

ephemeralOwner:如果该节点为临时节点, ephemeralOwner 值表示与该节点绑定的 session id. 如果不是, ephemeralOwner 值为 0.

`在 client 和 server 通信之前,首先需要建立连接,该连接称为 session。连接建立后,如果发生连接超时、授权失败,或者显式关闭连接,连接便处于 CLOSED状态, 此时 session 结束。`



### 节点类型

Znode 分为临时节点和永久节点。节点的类型在创建时即被确定，并且不能改变。

`临时节点：该节点生命周期依赖于创建它们的会话,一旦会话结束,临时 节点将被自动删除,当然可以也可以手动删除。临时节点不允许拥有子节点`

`永久节点：该节点的生命周期不依赖于会话，并且只有在客户端显示执行删除操作的时候，他们才能被删除。`

 

Znode 创建的时候可以指定序列化，会在名字后面追加一个序列号。

序列号对于此节点的父节点来说是唯一的，这样便会记录每个子节点创建的先后顺序。

 

所以有4种节点类型：

PERSISTENT：永久节点

EPHEMERAL：临时节点

PERSISTENT_SEQUENTIAL：永久节点、序列化

EPHEMERAL_SEQUENTIAL：临时节点、序列化



# shell

进入shell

```shell
cd /export/servers/zookeeper-3.4.5-cdh5.14.0/bin
zkCli.sh -server localhost:2181
```

创建节点： create [-s] [-e] path data acl

```shell
#创建永久节点
create /testp 123
#创建临时节点
create -e /testtemp 123
#创建顺序节点
create -s /testse 123
```

读取节点

```shell
#子节点信息
ls path 
#数据和节点属性信息
get path 
#子节点和节点属性信息
ls2 path
```

更新节点数据：set path data 

```shell
set /test0000000000 45ss7
```

删除节点

```shell
#删除节点
delete path 
#递归删除节点
rmr path
```

节点限制：setquota -n|-b val path

```shell
#限制最大子节点为2
setquota -n 2 /test
#限制数据最大长度为10
setquota -b 10 /test

#查看节点限制
listquota path

#删除节点限制
delquota path
```

其他

```sh
help
history
redo   #history中的历史命令编号可以通过redo快速执行
```



# javaAPI

创建maven工程，导入依赖

curator是zookeeper的封装框架：http://curator.apache.org/。

```xml
<!-- 
<repositories>
	<repository>
    	<id>cloudera</id>
		<url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>
	</repository>
</repositories> 
-->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.11</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.8</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>4.0.0</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>4.0.0</version>
</dependency>
```

```java
@Test
public void createNode() throws Exception {
    //获取客户端对象
    RetryPolicy retryPolicy = new  ExponentialBackoffRetry(1000, 1);
	//调用start开启客户端操作
    CuratorFramework client = CuratorFrameworkFactory.newClient
				("192.168.52.100:2181,192.168.52.110:2181,192.168.52.120:2181", 1000, 1000, retryPolicy);
	client.start();
    
	//创建永久节点
	client.create().creatingParentsIfNeeded().withMode(CreateMode.PERSISTENT).forPath("/hello3/world");
	//创建临时节点
    client.create().creatingParentsIfNeeded().withMode(CreateMode.EPHEMERAL).forPath("/hello5/world");
    
    //节点数据查询
    byte[] forPath = client.getData().forPath("/hello5");
	System.out.println(new String(forPath));
	
    //修改节点数据
    client.setData().forPath("/hello5", "hello7".getBytes());
    
    //关闭客户端
    client.close();
}
```



# 权限控制

https://www.runoob.com/w3cnote/zookeeper-acl.html



# watch机制

Watcher机制过程：

客户端向服务端注册 Watcher（订阅）、服务端事件发生触发 Watcher（发布）、客户端回调 Watcher 得到触发事件情况。



特点：

- 先注册再触发：Zookeeper 中的 watch 机制，必须客户端先去服务端注册监听，这样事件发送才会触发监听，通知给客户端。

- 一次性触发：订阅一次，事件触发一次后发送 watcher event  到客户端，监听即结束。


- event 异步发送：watcher 的通知事件从服务端发送到客户端是异步的。

- 事件封装：ZooKeeper 使用 WatchedEvent 对象来封装服务端事件并传递。

`WatchedEvent 包含了每一个事件的三个基本属性：通知状态（keeperState），事件类型（EventType）和节点路径（path）`



![20180915160333209](picture/20180915160333209.png)

其中连接状态事件(type=None, path=null)不需要客户端注册，客户端只要有需要直接处理就行了。



### shell

```shell
#设置一种监听
get /test0000000000 watch
#触发监听
set /test0000000000 45ss7
```

### javaAPI

```JAVA
//Curator帮我们实现了永久注册监听
//TreeCacheEvent是监听事件的主要类
@Test
public void watchNode() throws Exception {
    //获取客户端对象
    RetryPolicy retryPolicy = new  ExponentialBackoffRetry(1000, 1);
	//调用start开启客户端操作
    CuratorFramework client = CuratorFrameworkFactory.newClient
				("192.168.52.100:2181,192.168.52.110:2181,192.168.52.120:2181", 1000, 1000, retryPolicy);
	client.start();

    // ExecutorService pool = Executors.newCachedThreadPool();  
    //设置节点的cache  
    TreeCache treeCache = new TreeCache(client, "/hello5");  
    
    
    //设置监听器和处理过程  
    treeCache.getListenable().addListener(new TreeCacheListener() {  
        @Override  
        public void childEvent(CuratorFramework client, TreeCacheEvent event) throws Exception {  
            ChildData data = event.getData();  
            if(data !=null){  
                switch (event.getType()) { 
                    case NODE_ADDED:  
                        System.out.println("NODE_ADDED:"+data.getPath()+"数据:"+new String(data.getData()));
                        break;  
                    case NODE_REMOVED:  
                        System.out.println("REMOVED:"+data.getPath() +"数据:"+new String(data.getData()));  
                        break;  
                    case NODE_UPDATED:  
                        System.out.println("UPDATED:"+data.getPath()+"数据:"+new String(data.getData()));  
                        break;  
                    default:  
                        break;  
                }  
            }else{  
                System.out.println( "data is null : "+ event.getType());  
            }  
        }  
    });  
    
    //开始监听  
    treeCache.start();  
    Thread.sleep(50000000);
}
```















