官网地址：[http://sqoop.apache.org](http://sqoop.apache.org/docs/1.4.7/index.html)

下载地址：http://mirrors.hust.edu.cn/apache/sqoop/1.4.6/



# 安装

解压

```shell
tar -zxf sqoop-1.4.6.bin__hadoop-2.0.4-alpha.tar.gz -C /opt/module/
mv sqoop-1.4.6.bin__hadoop-2.0.4-alpha/ sqoop
```

配置文件

```shell
cd sqoop/conf
mv sqoop-env-template.sh sqoop-env.sh
vim sqoop-env.sh 
```

```shell
export HADOOP_COMMON_HOME=/opt/module/hadoop-3.1.3
export HADOOP_MAPRED_HOME=/opt/module/hadoop-3.1.3
export HIVE_HOME=/opt/module/hive
export ZOOKEEPER_HOME=/opt/module/zookeeper-3.5.7
export ZOOCFGDIR=/opt/module/zookeeper-3.5.7/conf
```

拷贝JDBC驱动

```shell
cp mysql-connector-java-5.1.48.jar /opt/module/sqoop/lib/
```



cdh 安装

云服务器要保证hive mysql 的机器网是通的

上传jar包：https://www.jianshu.com/p/ac2b616c54ad

hdfs权限解除 ，https://blog.csdn.net/weixin_41229271/article/details/87858334

FIELDS TERMINATED BY参数使用默认https://www.cnblogs.com/sabertobih/p/13959940.html

密码要带引号



# 命令

```shell
#帮助
bin/sqoop help
#连接数据库
bin/sqoop list-databases --connect jdbc:mysql://hadoop102:3306/ --username root --password 000000
#mysql 到 hive
bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/gmall \
--username root \
--password '000000' \
--query ‘select * from aa where $CONDITIONS’
--target-dir /test \
--delete-target-dir \
--fields-terminated-by '\t' \
--num-mappers 2 \
--split-by id
```



# 同步策略

https://zhuanlan.zhihu.com/p/133300693



不会发生变化的表，只需**同步一次**：

`地区表、省份表`



数据量大且只会增加的表，**增量同步**：

`订单详情表、产品评论表`

```sql
where
date_format(operate_time,'%Y-%m-%d')='$do_date'
```



数据量大且会增加变化的表，**新增和变化同步**：

`用户表、订单表`

```sql
where date_format(create_time,'%Y-%m-%d')='$do_date'
or date_format(operate_time,'%Y-%m-%d')='$do_date')
```



数据量小且会增加变化的表，**全量同步**：

`编码字典表、品牌表、商品一级分类表、收藏表、加购表`

```sql
where 1=1
```



首日全量同步脚本

```shell
vim mysql_to_hdfs_init.sh
```

```shell
#! /bin/bash

APP=gmall	#数据库库名
sqoop=/opt/module/sqoop/bin/sqoop

if [ -n "$2" ] ;then
   do_date=$2
else 
   echo "请传入日期参数"
   exit
fi 

#一个函数，函数中的$1需要调用的时候传参
import_data(){
$sqoop import \
--connect jdbc:mysql://hadoop102:3306/$APP \
--username root \
--password 000000 \
--target-dir /origin_data/$APP/$1/$do_date \
--delete-target-dir \
--query "$2 where \$CONDITIONS" \
--num-mappers 1 \
--fields-terminated-by '\t' \
--compress \
--compression-codec lzop \
--null-string '\\N' \
--null-non-string '\\N'

hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/common/hadoop-lzo-0.4.20.jar com.hadoop.compression.lzo.DistributedLzoIndexer /origin_data/$APP/$1/$do_date
}

#函数，调用了上面的函数
import_sku_sale_attr_value(){
	import_data sku_sale_attr_value "select id,sku_id,spu_id,sale_attr_value_id,sale_attr_name,from sku_sale_attr_value"
	}

# 传表名，即会同步
case $1 in
  "sku_attr_value")
      import_sku_attr_value
;;
  "sku_sale_attr_value")
      import_sku_sale_attr_value
;;
  "all")
   import_refund_payment
   import_sku_attr_value
   import_sku_sale_attr_value
;;
esac
```

```shell
chmod +x mysql_to_hdfs_init.sh
mysql_to_hdfs_init.sh all 2020-06-14
```

https://blog.csdn.net/bird3014/article/details/93122974



每日同步脚本

```shell
vim mysql_to_hdfs.sh
```

```shell
#! /bin/bash

APP=gmall
sqoop=/opt/module/sqoop/bin/sqoop

if [ -n "$2" ] ;then
    do_date=$2
else
    do_date=`date -d '-1 day' +%F`
fi

import_data(){
$sqoop import \
--connect jdbc:mysql://hadoop102:3306/$APP \
--username root \
--password 000000 \
--target-dir /origin_data/$APP/$1/$do_date \
--delete-target-dir \
--query "$2 and  \$CONDITIONS" \
--num-mappers 1 \
--fields-terminated-by '\t' \
--compress \
--compression-codec lzop \
--null-string '\\N' \
--null-non-string '\\N'

hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/common/hadoop-lzo-0.4.20.jar com.hadoop.compression.lzo.DistributedLzoIndexer /origin_data/$APP/$1/$do_date
}

#增量更新
import_order_info(){
  import_data order_info "select
                            id, 
                            total_amount, 
                            order_status, 
                            user_id, 
                            payment_way,
                            delivery_address,
                            out_trade_no, 
                            create_time, 
                            operate_time,
                            expire_time,
                            tracking_no,
                            province_id,
                            activity_reduce_amount,
                            coupon_reduce_amount,                            
                            original_total_amount,
                            feight_fee,
                            feight_fee_reduce      
                        from order_info
                        where (date_format(create_time,'%Y-%m-%d')='$do_date' 
                        or date_format(operate_time,'%Y-%m-%d')='$do_date')"
}

#全量更新
import_sku_info(){
  import_data sku_info "select 
                          id,
                          spu_id,
                          price,
                          sku_name,
                          sku_desc,
                          weight,
                          tm_id,
                          category3_id,
                          is_sale,
                          create_time
                        from sku_info where 1=1"
}

case $1 in
  "order_info")
     import_order_info
;;
  "sku_info")
     import_sku_info
;;
"all")
   import_order_info
   import_sku_info
;;
esac
```

```shell
chmod +x mysql_to_hdfs.sh
mysql_to_hdfs.sh all
```



# MySQL减压

MySQL如果是生产库，sqoop直接连接生产库会造成压力。

1. 最好用生产库的从库，不要用主库。
2. 脚本执行时间，设置在凌晨。
3. 控制并发，不要同时启动多个mapper。
4. 抽取的时候按照索引split。
5. 注意事务锁定？



















