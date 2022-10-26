官网地址：http://flume.apache.org/

文档地址：http://flume.apache.org/FlumeUserGuide.html

下载地址：http://archive.apache.org/dist/flume/



# 安装

解压

```shell
tar -zxf /opt/software/apache-flume-1.9.0-bin.tar.gz -C /opt/module/
mv /opt/module/apache-flume-1.9.0-bin /opt/module/flume
```

兼容hadoop

```shell
rm /opt/module/flume/lib/guava-11.0.2.jar
```

配置环境变量

```shell
cd conf
mv flume-env.sh.template flume-env.sh
vi flume-env.sh
```

```shell
export JAVA_HOME=/opt/module/jdk1.8.0_212
```

分发flume

配置flume

配置拦截器

启动脚本

```shell
#! /bin/bash

case $1 in
"start"){
        for i in hadoop102 hadoop103
        do
                echo " --------启动 $i 采集flume-------"
                ssh $i "nohup /opt/module/flume/bin/flume-ng agent --conf-file /opt/module/flume/conf/file-flume-kafka.conf --name a1 -Dflume.root.logger=INFO,LOGFILE >/opt/module/flume/log1.txt 2>&1  &"
        done
};;	
"stop"){
        for i in hadoop102 hadoop103
        do
                echo " --------停止 $i 采集flume-------"
                ssh $i "ps -ef | grep file-flume-kafka | grep -v grep |awk  '{print \$2}' | xargs -n1 kill -9 "
        done

};;
esac
```

```shell
chmod u+x f1.sh
f1.sh start
f1.sh stop
```



单点启动

```shell
nohup /opt/module/flume/bin/flume-ng agent \
--conf-file /opt/module/flume/conf/kafka-flume-hdfs.conf 
--name a1 \
-Dflume.root.logger=INFO,LOGFILE >/opt/module/flume/log2.txt   2>&1
```



# **HA** Flume



# 架构

agent：核心角色，包含三个组件。多个agent之间可以串联。

Source：采集组件，用于跟数据源对接，以获取数据。

Channel：传输通道组件，用于从source将数据传递到sink。

Sink：下沉组件，用于往下一级agent传递数据或者往最终存储系统传递数据。



# 组件选型

TailDir Source：断点续传、多目录。

`batchSize大小：Event 1K左右时，500-1000合适（默认为100）`



Kafka Channel：可以不用sink，直接存入kafka。

MemoryChannel：传输快，易丢失。用于不重要数据。

FileChannel：传输慢，不易丢失，且有索引，易恢复。用于重要数据



HDFS Sink：存入HDFS，配置时要注意小文件优化。



# 配置flume

导入流程：TailDir Source -- 格式验证拦截器 --Kafka Channel --KafkaSource --FileChannel-- HDFS Sink

配置第一级flume，在node01，node02上

```shell
#进入conf目录
vim file-flume-kafka.conf
```

```shell
#为各组件命名
a1.sources = r1
a1.channels = c1

#描述source
a1.sources.r1.type = TAILDIR # source类型
a1.sources.r1.filegroups = f1
a1.sources.r1.filegroups.f1 = /opt/module/applog/log/app.* #监控的文件
a1.sources.r1.positionFile = /opt/module/flume/taildir_position.json #监控文件的索引存放位置

#拦截器设置，source - 拦截器 - channel
a1.sources.r1.interceptors =  i1
a1.sources.r1.interceptors.i1.type = com.atguigu.flume.interceptor.ETLInterceptor$Builder #拦截器是一个java类，这是全类名

#描述channel
a1.channels.c1.type = org.apache.flume.channel.kafka.KafkaChannel # channel类型
a1.channels.c1.kafka.bootstrap.servers = hadoop102:9092,hadoop103:9092 #kafka集群
a1.channels.c1.kafka.topic = topic_log #kafka topic
a1.channels.c1.parseAsFlumeEvent = false # Kafka 不接收日志的文件头

#绑定source和channel以及sink和channel的关系
a1.sources.r1.channels = c1
```



配置第二级flume，在node03上

```shell
#进入conf目录
vim kafka-flume-hdfs.conf
```

```shell
#为各组件命名
a1.sources=r1
a1.channels=c1
a1.sinks=k1

## source1
a1.sources.r1.type = org.apache.flume.source.kafka.KafkaSource
a1.sources.r1.batchSize = 5000
a1.sources.r1.batchDurationMillis = 2000
a1.sources.r1.kafka.bootstrap.servers = hadoop102:9092,hadoop103:9092,hadoop104:9092
a1.sources.r1.kafka.topics=topic_log

## channel1
a1.channels.c1.type = file
a1.channels.c1.checkpointDir = /opt/module/flume/checkpoint/behavior1
a1.channels.c1.dataDirs = /opt/module/flume/data/behavior1/

## sink1
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = /origin_data/gmall/log/topic_log/%Y-%m-%d
a1.sinks.k1.hdfs.filePrefix = log-
a1.sinks.k1.hdfs.round = false

#控制生成的小文件
a1.sinks.k1.hdfs.rollInterval = 10
a1.sinks.k1.hdfs.rollSize = 134217728
a1.sinks.k1.hdfs.rollCount = 0

## 拼装
a1.sources.r1.channels = c1
a1.sinks.k1.channel= c1
```



# flume拦截器

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.flume</groupId>
        <artifactId>flume-ng-core</artifactId>
        <version>1.9.0</version>
        <scope>provided</scope>
    </dependency>

    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.62</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>2.3.2</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
        <plugin>
            <artifactId>maven-assembly-plugin</artifactId>
            <configuration>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
            </configuration>
            <executions>
                <execution>
                    <id>make-assembly</id>
                    <phase>package</phase>
                    <goals>
                        <goal>single</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

```java
//工具类,判断json格式是否正确。
package com.atguigu.flume.interceptor;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONException;

public class JSONUtils {
    public static boolean isJSONValidate(String log){
        try {
            JSON.parse(log);
            return true;
        }catch (JSONException e){
            return false;
        }
    }
}
```

```java
package com.atguigu.flume.interceptor;
import com.alibaba.fastjson.JSON;
import org.apache.flume.Context;
import org.apache.flume.Event;
import org.apache.flume.interceptor.Interceptor;
import java.nio.charset.StandardCharsets;
import java.util.Iterator;
import java.util.List;

public class ETLInterceptor implements Interceptor {
    @Override
    public void initialize() {}

    // 用于接收每条信息
    @Override
    public Event intercept(Event event) {
        byte[] body = event.getBody();
        String log = new String(body, StandardCharsets.UTF_8);
        if (JSONUtils.isJSONValidate(log)) {
            return event;
        } else {
            return null;
        }
    }
    
    //迭代器删除不合格的event
    @Override
    public List<Event> intercept(List<Event> list) {

        Iterator<Event> iterator = list.iterator();
        while (iterator.hasNext()){
            Event next = iterator.next();
            if(intercept(next)==null){
                iterator.remove();
            }
        }
        return list;
    }

    public static class Builder implements Interceptor.Builder{
        @Override
        public Interceptor build() {
            return new ETLInterceptor();
        }
        @Override
        public void configure(Context context) {}

    }
    
    @Override
    public void close() {}
}
```

打包

部署 flume-interceptor-1.0-SNAPSHOT-jar-with-dependencies.jar 部署到 flume/lib

分发



# 经典问题

零点漂移问题：通过时间戳拦截器解决。

内存溢出：最大内存一般设置为4-6G

```
/opt/module/flume/conf/flume-env.sh
export JAVA_OPTS="-Xms100m -Xmx2000m -Dcom.sun.management.jmxremote" 
```









































