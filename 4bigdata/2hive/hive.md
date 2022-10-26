尚硅谷

文档：https://cwiki.apache.org/confluence/display/Hive/GettingStarted

官方手册:https://www.cnblogs.com/fanzhenyong/p/9746796.html



# Hive 安装

[anzhuang.docx](picture/anzhuang.docx)



# Hive 配置

https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties

全局配置：vim hive-site.xml																		全局有效

命令行配置：bin/hive   -hiveconf    *hive.root.logger=INFO,console*		  对 hive 启动实例有效

hive.sql配置：set   *mapred.reduce.tasks=100U*;										对 hive 的连接 session 有效



# Hive 架构

![img](picture/16eab91453ae3874~tplv-t2oaga2asx-watermark.awebp)

   ![img](picture/clip_image023.gif)

 

用户接口：WEB UI , hive 命令行，JDBC。

`开发人员通过JDBC连接到hive，进行开发。Thrift可以让不同语言调用hive接口。`

`WEB UI 需要额外的配置，hive并不主动提供。`



元数据：hive元数据存在独立数据库中。元数据包括表列类型，分区，目录等。

`hive自带derby数据库。我们生产中用MySQL以支持三种访问元数据的方式。`



驱动：编译器（Compiler），优化器（Optimizer），执行器（Executor）

`解释器：解释器的作用是将 HiveSQL 语句转换为抽象语法树（AST）。`

`编译器：编译器是将语法树编译为逻辑执行计划。`

`优化器：优化器是对逻辑执行计划进行优化。`

`执行器:调用底层的运行框架执行逻辑执行计划。被调用的计算框架 默认MR 或者 spark。`



数据存储：HDFS 。

数据计算：MR、Spark。



# 工作原理

![How Hive Works](picture/1-14122R10220b9.jpg)



- 步骤1：UI 调用 driver 的接口；

- 步骤2：driver 为查询创建会话句柄，并将查询发送到 COMPILER(编译器)生成执行计划。解析sql

- 步骤3和4：编译器从元数据存储中获取本次查询所需要的元数据，该元数据用于对查询树中的表达式进行类型检查，以及基于查询谓词修建分区；

- 步骤5：编译器生成的计划是分阶段的DAG，每个阶段要么是 map/reduce 作业，要么是一个元数据或者HDFS上的操作。将生成的计划发给 DRIVER。

- 步骤6、7、8：执行引擎将这些阶段提交给适当的组件。使用MR或者Spark运行。

- 步骤7、8和9：最终的临时文件将移动到表的位置，确保不读取脏数据(文件重命名在HDFS中是原子操作)。



步骤2的COMPILER(编译器)的工作

https://mp.weixin.qq.com/s/_MZsDfJq7klL2D-tJgCZBA



tips：对于用户的无计算的查询，临时文件的内容由执行引擎直接从HDFS读取，然后通过Driver发送到UI



# 使用方式

命令行交互

```shell
bin/hive
```

shell语句执行

```shell
bin/hive -e "use myhive;select * from test;"
bin/hive -f hive.sql
```

jdbc连接

```shell
#这是命令行模拟的JDBC连接，还可以用其他JDBC驱动连接。
#bin/hive --service hiveserver2
#bin/beeline
#beeline> !connect jdbc:hive2://node03.hadoop.com:10000
```



# 数据类型

### 基本数据类型

| Hive 数据类型 | Java 数据类型 | 长度                                                   | 例子                    |
| ------------- | ------------- | ------------------------------------------------------ | ----------------------- |
| TINYINT       | byte          | 1byte 有符号整数                                       | 20                      |
| SMALINT       | short         | 2byte 有符号整数                                       | 20                      |
| INT           | int           | 4byte 有符号整数                                       | 20                      |
| BIGINT        | long          | 8byte 有符号整数                                       | 20                      |
| BOOLEAN       | boolean       | 布尔类型，true  或者  false                            | TRUE   FALSE            |
| FLOAT         | float         | 单精度浮点数                                           | 3.14159                 |
| DOUBLE        | double        | 双精度浮点数                                           | 3.14159                 |
| STRING        | string        | 字符系列。可以指定字 符集。可以使用单引号或者双 引号。 | ‘ now  is ’   “for men” |
| TIMESTAMP     |               | 时间戳，内容格式：yyyy-mm-dd hh:mm:ss                  |                         |
| DATE          |               | 日期，内容格式：YYYY­MM­DD                             |                         |
| BINARY        |               | 字节数组                                               |                         |

对于 Hive 的 String 类型相当于数据库的 varchar 类型，该类型是一个可变的字符串，理论上它可以存储 2GB 的字符数。



### 集合数据类型

复杂数据 类型允许任意层次的嵌套。

| 数据类型  | 定义                                   |
| --------- | -------------------------------------- |
| STRUCT    | struct<street:int, city:int>           |
| MAP       | map<string, int>                       |
| ARRAY     | array<string>                          |
| UNIONTYPE | 联合体 UNIONTYPE<data_type, data_type> |

```sql
-- 定义
array<string>
map<string,string>
struct<id:int,name:string,age:int>

--取值
array[1]
map['key']
named_struct.n1

--赋值
collect_set(id1) --存同一列的不同值，一般group by后可以把分组用这个函数变成数组
collect_list（id1）--与collect_set相比，不去除重复值

named_struct('id', id, 'task_name', name) --多列变成一列
```



### 数据类型转换

**隐式类型转换：**

1. 任何整数类型都可以隐式地转换为一个范围更广的类型，如 TINYINT 可以转换成INT，INT 可以转换成 BIGINT。
2. 所有整数类型、FLOAT 和 STRING 类型都可以隐式地转换成 DOUBLE。
3. TINYINT、SMALLINT、INT 都可以转换为 FLOAT。
4. BOOLEAN 类型不可以转换为任何其它的类型。



**CAST显式类型转换：**

例如 CAST('1' AS INT)将把字符串'1'  转换成整数 1；

如果强制类型转换失败，如执行 CAST('X' AS INT)，表达式返回空值 NULL。



# 数据库操作

### 创建数据库

```sql
-- 数据库在 HDFS 上的默认存储路径是/user/hive/warehouse/db_hive.db
create database db_hive; 

-- 指定存储位置
create database db_hive2 location '/db_hive2.db'; 

create database if not exists db_hive;

-- 模板
CREATE DATABASE [IF NOT EXISTS] database_name [COMMENT database_comment] [LOCATION hdfs_path]
[WITH DBPROPERTIES (property_name=property_value, ...)];
```



###  查询数据库

```sql
-- 查询所有数据库
show databases; 
show databases like 'db_hive*'; 

-- 查看数据库信息
desc database db_hive;
desc database extended db_hive; 
```



### 切换数据库

```sql
use db_hive; 
```



### 指定数据库属性

```sql
-- 可以在创建数据库的时候指定，也可以之后设置
alter database db_hive set dbproperties('createtime'='20170830');
```



### 删除数据库

```sql
-- 删除空数据库
drop database db_hive2; 
drop database if exists db_hive2;

-- 强制删除数据库
drop database db_hive cascade;
```



# 数据表操作

### 创删改表

创建表

```sql
-- 通用写法
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name
[(col_name data_type [COMMENT col_comment], ...)] [COMMENT table_comment]
[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)] 

[CLUSTERED BY (col_name, col_name, ...) --分桶列只能一个
[SORTED BY (col_name [ASC|DESC], ...)]  --桶内排序，可选任意列
 INTO num_buckets BUCKETS]  

[ROW FORMAT row_format]  --指定行列分割符
[STORED AS file_format] 
[LOCATION hdfs_path]
[TBLPROPERTIES (property_name=property_value, ...)] --设置属性，如压缩方式
```

```sql
-- 正常建表
CREATE EXTERNAL TABLE if not exists dim_sku_info (
    `id` STRING COMMENT '商品id'
	`price` DECIMAL(16,2) COMMENT '商品价格') COMMENT '商品维度表'
PARTITIONED BY (`dt` STRING COMMENT '日期')
CLUSTERED BY (id) SORTED BY(price) INTO 3 BUCKETS
row format delimited fields terminated by '\t' 
STORED AS PARQUET
LOCATION '/warehouse/gmall/dim/dim_sku_info/'
TBLPROPERTIES ("parquet.compression"="lzo");
```

```sql
-- 复制表结构，分区信息也会丢失
create table if not exists student3 like student; 
```

```sql
-- 复制表结构和字段值，但会丢失分区等信息
create table if not exists student2 as select id, name from student; 
```

删除表

```sql
drop table dept; 
truncate table student; --删数据，保留表结构
```

修改表

```sql
--重命名表
ALTER TABLE table_name RENAME TO new_table_name 
```



### 增删改行列

下列操作需要验证，不可直接使用线上数据进行操作。



添加列

```sql
-- 原来每条数据的新增列显示为 null，新增数据的新增列正常显示
alter table dept add columns(deptdesc string comment 'asd'，aa string comment 'fds'); 

-- 分区表新增字段，历史分区中该字段一直为null，无法修改。可以先删除分区，再重新新建分区插入数据，即可生效。
```

修改列

```sql
alter table dept change column col_old_name col_new_name string; --修改字段名称、类型、注释、顺序
alter table table_name change c_time c_time string after column_1 ;  -- 移动到指定位置,column_1字段的后面
```

删除字段

```sql
-- 尽量不要删除，删除中间字段可能会出现错列的情况。新表旧表的对应字段类型要一致。
alter table table_name replace columns(column_1 string);    --column_2不写，即删除column_2，保留column_1，不写的字段全部被删除
```



删除行

```sql
-- 通过重新插入想保留数据的方法删除
insert overwrite table dim.dim_common_date_once select * from dim.dim_common_date_once where date_id!='date_id';
```



### 查询表

hive使用子查询的时候要起别名，hive对子查询支持不好，尽量使用临时表



查看表

```sql
show tables; 

desc student2; 
desc formatted student2; 
```

查数据

```sql
SELECT select_expr, select_expr
FROM table_reference
[WHERE where_condition] 
[GROUP BY col_list] 
[ORDER BY col_list] 
[cluster by col_list]
[LIMIT number]
```

```sql
select * from emp;

-- 给列起别名
select ename AS name, deptno dn from emp; 

--列进行运算
select sal+1 from emp; 

--列函数运算
select count(*) cnt from emp;
select max(sal) max_sal from emp; 

--模糊查询，_是所有任意个数字符，%单个字符
 select * from emp where ename LIKE '_A%'; 
 
 --正则查询
 select * from emp where ename RLIKE '[A]'; 
 
 --抽样查询
select * from stu tablesample(0.1 percent) ;
select * from stu_buck tablesample(bucket 1 out of 4 on id);
```

```sql
-- 分组查询
select deptno, avg(sal) avg_sal from emp group by deptno having avg_sal > 2000;
-- 执行顺序
group by 分组字段 select 分组字段，聚合字段 having 对select结果筛选
```



比较运算符

```
=、>、<、>=、<=、<>、!=
AND、OR、NOT
```

关键字

```sql
SELECT * FROM Persons WHERE LastName IN ('Adams','Carter');
SELECT LastName,FirstName,Address FROM Persons WHERE Address IS NULL;
SELECT * FROM Persons WHERE LastName [NOT] BETWEEN 'Adams' AND 'Carter';
```



### 导入导出

从文件加载数据

```sql
load data [local] --数据来自本地还是hdfs
inpath '数据的 path' 
[overwrite] into table student; --是否覆盖加载
```

插入数据

```sql
insert into table student_par values(1,'wangwu'),(2,'zhaoliu')；
insert overwrite table student_par select id, name from student;
```

导出数据

```sql
insert overwrite [local] directory '/opt/module/hive/data/export/student1'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
select * from student;
```



### 内外表

内部表：删除内部表，删除表元数据和数据。

外部表：删除外部表，删除元数据，不删除数据。

建表时，内部表和外部表都会移动数据到hive管理的数据目录目录下，可以location指定。



外部表场景：为了数据安全，使用外部表。多数据库处理同一 HDFS 数据 或者 同一 HDFS 数据有不同表结构。

```sql
CREATE EXTERNAL TABLE table_name 
(col_name data_type) 
[LOCATION hdfs_path]
```



内外表互转

```sql
alter table student2 set tblproperties('EXTERNAL'='TRUE'); 
alter table student2 set tblproperties('EXTERNAL'='FALSE'); 
```



### 分区表

分区字段不一定是表的某一字段，可以建表时自定义分区字段，导入查询数据时指定分区值即可。

分区实际就是表文件夹下的分区文件夹。



分区

```sql
--创建分区
CREATE TABLE <table_name> (<col_name> <data_type>)
    PARTITIONED BY  (<partition_key> <data_type>) ;
    
--添加分区值。不支持添加分区字段。如果多个分区字段，分区值不能单独添加。
alter table tablename add partition(col=value)

--删除分区值，分区字段不可删除
alter table tablename drop partition(col=value)

--修复分区，hdfs目录有分区文件夹，hive表没有，修复
msck repair table tablename

-- 查看分区
show  partitions  aaa;
```

分区表加载数据

```sql
-- 加载数据到分区表，表是分区表，加载数据必须指定分区
load data local inpath '/opt/module/hive/datas/dept_20200401.log' into table dept_partition partition(day='20200401');

-- 静态分区，覆盖加载数据时指定分区值
INSERT OVERWRITE TABLE <table_name> 
	PARTITION (<partition_key>=<partition_value>) 
    SELECT <select_statement>;

--动态分区，覆盖加载数据时根据select的最后列值确定此条数据所在分区
insert overwrite table dwd_comment_info partition (dt)
select id,create_time,date_format(create_time,'yyyy-MM-dd') dt from ods_comment_info

--动态分区参数
hive.exec.dynamic.partition=true --开启动态分区
hive.exec.dynamic.partition.mode=nonstrict --所有分区字段都可以使用动态分区
hive.exec.max.dynamic.partitions=1000 --最多动态分区数量
hive.exec.max.created.files=100000 --MRjob最多可以创建多少个文件
```



### 分桶表

分桶针对的是数据文件，将同一个分区下的数据分成多个文件，就是分桶。

对分桶字段的值进行哈希，然后除以桶的个数求余的方式决定该条记录存放在哪个桶当中。

```sql
--创建分桶表
create table stu_buck(id int, name string) 
clustered by(id) into 4 buckets; --分桶列只能一个

--加载数据，无需处理
load data inpath '/student.txt' into table stu_buck; 

--插入数据
insert into table stu_buck select * from student_insert;

--有排序需求时，分桶查询更快，可以加快全局排序速度
SELECT select_expr FROM table_reference
cluster by col_list --会把查询结果分桶到不同reduce加对分桶字段排序。分桶与排序字段不同时，用distribute by + sort by 代替cluster by。

--join时，on的字段是分桶字段时，会加快join速度
```

reduce 的个数设置为-1，需要大于分桶桶数。

hdfs 数据进分桶表，使用load的形式，不能手动然后修复表。



分桶的优点：

（1）因为桶数量是固定的，所以他没有数据波动；

（2）桶对抽样非常适合；

（3）分桶有利于执行高效的map-side join。



### 自定义函数

删除自定义函数

```sql
drop [temporary] function [if exists] [dbname.]function_name; 
drop function function_name；
```



1.maven

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.hive</groupId>
        <artifactId>hive-exec</artifactId>
        <version>3.1.2</version>
    </dependency>
</dependencies>
```

2.编写函数

```java
package com.atguigu.hive;

import org.apache.hadoop.hive.ql.exec.UDFArgumentException;
import org.apache.hadoop.hive.ql.exec.UDFArgumentLengthException;
import org.apache.hadoop.hive.ql.exec.UDFArgumentTypeException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDF;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectIn spectorFactory;

/**
 * 自定义 UDF 函数，需要继承 GenericUDF 类
 * 需求: 计算指定字符串的长度
 */
public class MyStringLength extends GenericUDF {
    /**
     * @param arguments 输入参数类型的鉴别器对象
     * @return 返回值类型的鉴别器对象
     * @throws UDFArgumentException
     */
    @Override
    public ObjectInspector initialize(ObjectInspector[] arguments) throws UDFArgumentException {
// 判断输入参数的个数
        if (arguments.length != 1) {
            throw new UDFArgumentLengthException("Input Args Length Error!!!");
        }
// 判断输入参数的类型

        if (!arguments[0].getCategory().equals(ObjectInspector.Category.PRIMITIVE)
        ) {
            throw new UDFArgumentTypeException(0, "Input Args Type Error!!!");
        }
//函数本身返回值为 int，需要返回 int 类型的鉴别器对象
        return PrimitiveObjectInspectorFactory.javaIntObjectInspector;
    }

    /**
     * 函数的逻辑处理
     *
     * @param arguments 输入的参数
     * @return 返回值
     * @throws HiveException
     */
    @Override
    public Object evaluate(DeferredObject[] arguments) throws HiveException {
        if (arguments[0].get() == null) {
            return 0;
        }
        return arguments[0].get().toString().length();
    }

    @Override
    public String getDisplayString(String[] children) {
        return "";
    }
}
```

3.添加jar包

```sql
add jar /opt/module/data/myudf.jar; 
```

4.创建关联

```sql
create temporary function my_len as "com.atguigu.hive. MyStringLength";
```

5.使用udf

```sql
select ename,my_len(ename) ename_len from emp; 
```



# 内置函数

### 内置函数

```sql
show functions; 
desc function upper; 
desc function extended upper; 
```



### 基础函数

```sql
select count(1) from score;
select max(s_score) from score;
select min(s_score) from score;
select sum(s_score) from score;
select avg(s_score) from score;
```



### 字符串函数

```sql
-- length
select length('iteblog') from iteblog;

-- reverse
select reverse('iteblog') from iteblog;

--concat 字符串拼接函数
select concat('www','.iteblog','.com') from iteblog;

-- concat_ws 字符串拼接函数，带分隔符
select concat_ws('.','www','iteblog','com') from iteblog;

-- substr 字符串截取函数，开始位置，长度
select substr('iteblog',3) from iteblog;
select substr('abcde',3,2) from iteblog;

-- upper
select upper('abCd') from dual;

-- lower
select lower('abCd') from dual;

-- trim 去除字符串的空格
select trim(' abc ') from dual;

-- ltrim，rtrim
select trim(' abc ') from dual;

-- regexp_replace 字符串替换函数，使用正则表达式，有的需要转义
select regexp_replace('foobar', 'oo|ar', '') from dual;

-- regexp_extract 按照正则表达式提取字段，0拿所有匹配，1拿第一个括号匹配上的值
select regexp_extract('foothebar', 'foo(.*?)(bar)', 1) from dual;

-- parse_url，第二个参数可取HOST, PATH, QUERY, REF, PROTOCOL, AUTHORITY, FILE，第三个参数可省略
hive> select parse_url('http://facebook.com/path1/p.php?k1=v1&k2=v2#Ref1', 'QUERY', 'k1') from dual;

-- get_json_object， json 解析函数，第二个参数用$取出想要的值
SELECT get_json_object('[{"name":"大郎","sex":"男","age":"25"},{"name":"西门庆","sex":"男","age":"47"}]',"$[0].age") from aa;

-- space 空格字符串
select space(10) from dual;

-- repeat 重复字符串 ，把字符串重复5次
select repeat('abc',5) from dual;

-- lpad 左补足函数，将td补足在abc左边直至10位。rpad
select lpad('abc',10,'td') from dual;

-- split
select split('abtcdtef','t') from dual;

-- find_in_set，找到ab的下标 2
select find_in_set('ab','ef,ab,de') from dual;

-- INSTR 搜索b的下标 2
select instr("abcde",'b') from dual;

-- str_to_map 字串变成json格式，第一个分隔符分割键，第二个分隔符分割值
select str_to_map('aaa:123&bbb:456', '&', ':') from dual;
```



### 时间函数

```sql
select date_format('2020-06-14','yyyy-MM');

-- 当前天的下一天
select date_add('2020-06-14',-1);

-- 取当前天的下一个周一
select next_day('2020-06-14','MO');

-- 取当前周的周一
select date_add(next_day('2020-06-14','MO'),-7);

-- 求当月最后一天日期）
select last_day('2020-06-14');

DATEDIFF('2022-04-17',shop_add_time)

current_timestamp
from_unixtime
```



### 选择函数

多值取非空

```sql
-- nvl，第一个参数为空时，取第二个参数值
select nvl(comm,-1) from emp; 

-- coalesce，作用同nvl，只是可以传多个参数
select coalesce(id,id2,id3) from emp; 
```



case when 

```sql
select dept_id,
sum(case when sex='男' then 1 else 0 end) male_count,
sum(case when sex='女' then 1 else 0 end) female_count 
from emp_sex
group by dept_id;

case
	when then 
	when tnen
	else
end
```



### 列转行函数

一列变多行

```sql
-- split将字符串切割成数组
-- explode将一个数据数组爆炸成多行
-- ateral VIEW 列 表 as 新列名 ，视图合并
-- 一个数组列变成多行
SELECT movie, category_name 
FROM movie_info 
lateral VIEW explode(split(category,",")) movie_info AS category_name; --因为explode后不能获取原视图，所以视图合并回原表
```



### 行转列函数

多行变一列

```sql
-- concat_ws 将多个字符串连接成一个字符串,第一个参数是分隔符
-- collect_list 将多个值变成一个数组
-- id相同的行变成一个字符串
select 
user_id,
concat_ws(',',collect_list(order_id)) as order_value 
from col_lie
group by user_id
```



### 窗口函数

```sql
--开窗函数 over，执行顺序同select中函数，一般和聚合函数一起使用，开窗函数不会减少表数据
select id,count(*) over() from aa -- 将统计结果给每一条数据带上
select id,count(*) over(partition by id order by id) from aa --将分组，排序，聚合函数后的结果给每一条数据
```

排序函数

```mysql
-- 常和窗口函数一起使用，对分组后的数据进行分组
ROW_NUMBER()  	排序时，给序号，序号不会重复。
RANK()  		排序时值相同时序号会重复，序号按照数据量跳跃增加。
DENSE_RANK()  	排序时值相同时序号会重复，序号按照123增加。
```



### 抽样函数

数据块抽样

```sql
-- select语句不能带where条件且不支持子查询，可通过新建中间表或使用随机抽样解决
select * from xxx tablesample(10 percent)
tablesample(n M) 
tablesample(n rows)
```

分桶抽样

```sql
-- 将表随机分成10组，抽取其中的第一个桶的数据
select * from table_01 tablesample(bucket 1 out of 10 on rand())
select * from table_01 tablesample(bucket 1 out of 10 on bucketed_column)
```

随机抽样

```sql
select * from table_name where col=xxx distribute by rand() sort by rand() limit num;
select * from table_name where col=xxx order by rand() limit num; --耗时更长
```



### 集合类型函数

```sql
-- 数组是否包含某元素
select ARRAY_CONTAINS(array,'141') from ods.ods_sys_tbl_sys_config_full

-- 数组中某元素的索引
select
stu.stu_pos,
stu.stu_value,
from table_name
lateral view posexplode(student_list) stu as stu_pos,stu_value
```



# 常用SQL

with ......as 临时表 

```sql
with 
	aaa as (select...)
	bbb as (select...)
select * from aaa join bbb on aaa.id=bbb.id
```



group by 后统计每组数量

```sql
SELECT COUNT(*) over() from dwd.orders group by member_id limit 1;
```



# 存储和压缩

### 数据存储

Hive 数据存储在HDFS中，Hive 所有表都是对HDFS文件的映射，HDFS支持数据格式：TextFile、SequenceFile、ORC、PARQUET

其中 ORC最节省空间，其次是PARQUET。



hive可以在建立表时指定存储格式，则加载到表的数据都以此种格式存储。

或者直接load文件进表，则存储格式就是文件格式。



### 数据压缩

| 压缩格式 | 工具  | 算法    | 文件扩展名 | 是否可切分 |
| -------- | ----- | ------- | ---------- | ---------- |
| DEFAULT  | 无    | DEFAULT | .deflate   | 否         |
| Gzip     | gzip  | DEFAULT | .gz        | 否         |
| bzip2    | bzip2 | bzip2   | .bz2       | 是         |
| LZO      | lzop  | LZO     | .lzo       | 否         |
| LZ4      | 无    | LZ4     | .lz4       | 否         |
| Snappy   | 无    | Snappy  | .snappy    | 否         |



一般创建表时设置压缩格式即可，不用专门配置参数

```sql
-- parquet默认使用snappy压缩
create table log_parquet_snappy( 
    track_time string,
    url string, 
    session_id string, 
    referer string,
    ip string, 
    end_user_id string, 
    city_id string
)
row format delimited fields terminated by '\t' 
stored as parquet 
```

MR引擎 压缩参数设置

```shell
# 开启中间传输数据压缩
set hive.exec.compress.intermediate=true; 
# 开启输出数据压缩
set hive.exec.compress.output=true; 

# map输出压缩
set mapreduce.map.output.compress=true; 
set mapreduce.map.output.compress.codec= org.apache.hadoop.io.compress.SnappyCodec;
# reduce 阶段压缩
set mapreduce.output.fileoutputformat.compress=true; 
set mapreduce.output.fileoutputformat.compress.type=BLOCK;
set mapreduce.output.fileoutputformat.compress.codec = org.apache.hadoop.io.compress.SnappyCodec;
```



### 存储压缩总结

https://www.studytime.xin/article/hive-knowledge-storage-format.html

除了ods层和app层外，其他层的压缩及存储格式尽量一致。



# Hive 优化

### explain

explain：查看执行计划的基本信息

```sql
explain select sum(id) from test1;
```

```sql
-- 划分为两个 stage
STAGE DEPENDENCIES:
  Stage-1 is a root stage
  Stage-0 depends on stages: Stage-1

STAGE PLANS:
  Stage: Stage-1
    Map Reduce
      Map Operator Tree:
          TableScan	-- 加载表
            alias: test1 -- 表名称
            Statistics: Num rows: 6 Data size: 75 Basic stats: COMPLETE Column stats: NONE -- 数据量6条，数据大小75
            Select Operator
              expressions: id (type: int) -- 需要的字段名称及字段类型
              outputColumnNames: id
              Statistics: Num rows: 6 Data size: 75 Basic stats: COMPLETE Column stats: NONE
              Group By Operator
                aggregations: sum(id) -- 聚合函数
                mode: hash -- 聚合模式。hash：随机聚合，partial：局部聚合；final：最终聚合
                outputColumnNames: _col0
                Statistics: Num rows: 1 Data size: 8 Basic stats: COMPLETE Column stats: NONE
                Reduce Output Operator
                  sort order: -- 值为空 不排序；值为 + 正序排序，值为 - 倒序排序
                  Statistics: Num rows: 1 Data size: 8 Basic stats: COMPLETE Column stats: NONE
                  value expressions: _col0 (type: bigint)
      Reduce Operator Tree:
        Group By Operator
          aggregations: sum(VALUE._col0)
          mode: mergepartial
          outputColumnNames: _col0
          Statistics: Num rows: 1 Data size: 8 Basic stats: COMPLETE Column stats: NONE
          File Output Operator -- 文件输出操作
            compressed: false -- 是否压缩
            Statistics: Num rows: 1 Data size: 8 Basic stats: COMPLETE Column stats: NONE
            table:
                input format: org.apache.hadoop.mapred.SequenceFileInputFormat
                output format: org.apache.hadoop.hive.ql.io.HiveSequenceFileOutputFormat
                serde: org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe

  Stage: Stage-0
    Fetch Operator	-- 客户端获取数据
      limit: -1	-- 不限制
      Processor Tree:
        ListSink
```



explain dependency：藐视SQL的数据来源，哪张表，哪个分区

```sql
explain dependency select s_age,count(1) num from student_orc;
```

```json
// input_partitions：来源表分区，空代表没有分区表
// input_tables：来源表
{"input_partitions":[],
 "input_tables":[{"tablename":"default@student_tb _orc","tabletype":"MANAGED_TABLE"}]}
```



explain authorization：查看输入输出位置

```sql
explain authorization 
select variance(s_score) from student_tb_orc;
```

```
INPUTS: 
  default@student_tb_orc 
OUTPUTS: 
  hdfs://node01:8020/tmp/hive/hdfs/cbf182a5-8258-4157-9194- 90f1475a3ed5/-mr-10000 
CURRENT_USER: 
  hdfs 
OPERATION: 
  QUERY 
```



- explain vectorization：查看SQL的向量化描述信息，显示为什么未对Map和Reduce进行矢量化。从 Hive 2.3.0 开始支持；

- explain analyze：用实际的行数注释计划。从 Hive 2.2.0 开始支持；
- explain cbo：输出由Calcite优化器生成的计划。CBO 从 Hive 4.0.0 版本开始支持；
- explain locks：这对于了解系统将获得哪些锁以运行指定的查询很有用。LOCKS 从 Hive 3.2.0 开始支持；
- explain ast：输出查询的抽象语法树。AST 在 Hive 2.1.0 版本删除了，存在bug，转储AST可能会导致OOM错误，将在4.0.0版本修复；
- explain extended：加上 extended 可以输出有关计划的额外信息。这通常是物理信息，例如文件名，这些额外信息对我们用处不大；



### 参数优化

##### Fetch 抓取

Fetch 抓取是指，Hive 中对某些情况的查询可以不必使用 MapReduce 计算，而可以直接抓取数据。

```xml
<property>
	<name>hive.fetch.task.conversion</name>
	<value>more</value>
</property>
```



##### 本地模式

Hive 可以通过本地模式在单台机 器上处理所有的任务。对于小数据集，执行时间可以明显被缩短。

```java
//开启本地模式
set hive.exec.mode.local.auto=true; 
//设置 local mr 的最大输入数据量，当输入数据量小于这个值时采用 local mr 的方式，默认 为 134217728，即 128M
set hive.exec.mode.local.auto.inputbytes.max=50000000;
//设置 local mr 的最大输入文件个数，当输入文件个数小于这个值时采用 local mr 的方式，默 认为 4
set hive.exec.mode.local.auto.input.files.max=10;
```



##### 并行执行

资源空闲的时候允许同一个sql的不同阶段，可以并行执行的就并行执行

```shell
set hive.exec.parallel=true;
set hive.exec.parallel.thread.number=16; #同一个 sql 允许最大并行度，默认为8。
```



##### 严格模式

```shell
#查询分区表时必须在where条件中含有分区字段，禁止全表扫描分区表
hive.strict.checks.no.partition.filter=true 
#要求使用order by 查询时，必须使用limit。全局排序时会用一个reduce处理，不用limit，每次都会数据全部排序
hive.strict.checks.orderby.no.limit=true 
#限制笛卡尔积查询，即join where操作。因为hive不像关系型数据库可以把join where 优化为join on
hive.strict.checks.cartesian.product=true
```



### map 数量调整

map 数量影响因素：input 的文件总个数，input 的文件大小，集群设置的文件块大小。

`配置中设置map数并不会真正影响map数量`



map数量控制原则：使大文件量利用合适的map数；使单个map任务处理合适的数据量。

`每个map的执行时长至少要保持一分钟`



大量小文件，导致生成大量的map，会有资源的浪费，此时应该减少map数量。

```shell
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat; #开启小文件合并
set mapred.max.split.size=100000000; 		  #每个map处理的最大文件大小100m
set mapred.min.split.size.per.node=100000000; #节点中可以处理的最小文件大小
set mapred.min.split.size.per.rack=100000000; #机架中可以处理的最小文件大小
#合并逻辑，首先文件读取，每100m生成map。剩余部分合并，根据节点处理文件大小，每100m一个map。
```

如果每个map处理的数据都是只有两列，但是很多行，此时一个map处理会慢，应该增加map数量

```sql
-- 当对表a进行复杂查询时，表a只有一个128m文件，一个map处理很慢。
-- 生成一个临时表，把表a的一个文件变成10个文件，再进行复杂查询。
set mapred.reduce.tasks=10;

create table a_1 as
select * from a distribute by rand(123);
```



### reduce 数量调整

reduce数量控制原则：使大数据量利用合适的reduce数；使单个reduce任务处理合适的数据量。



不进行设置的时候hive 会推测reduce数量。

一般1G数据一个rudece，最多999个。



直接设置

```sql
set mapred.reduce.tasks = 15;
set hive.exec.reducers.bytes.per.reducer=500000000; -- 设置500M数据一个reduce
```



有一些全局操也会使reduce只有一个，尽量避免

```sql
-- 全局计数，可以改写成第二个sql
select count(1) from popt_tbaccountcopy_mes where pt = '2012-07-04';
select pt,count(1) from popt_tbaccountcopy_mes where pt = '2012-07-04' group by pt;

-- 全局排序  Order by

-- 进行了笛卡尔积运算，尽量避免，join的时候要使用on，不能用where或者不加
```



### 合并小文件

```shell
# 开启小文件合并
set hive.input.format= org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;
# map-only任务结束时合并小文件
SET hive.merge.mapfiles = true;
# reduce 任务结束时合并小文件
SET hive.merge.mapredfiles = true; 
#小文件大小256M
SET hive.merge.size.per.task = 268435456; 
# 输出文件平均大小小于该值时，进行合并
SET hive.merge.smallfiles.avgsize = 16777216; 
```



### 语句优化

Count(Distinct col) ：去重统计，使用group by count 代替，避免一个reduce数据量过大。



join的时候将where条件写在子查询里，再join，不要join完再where，避免全表关联。两种方式得到的数据也可能不一样。



连续full join ，注意连接条件要写成 nvl（id1，id2）= id3 。



left join 注意从表的连接字段要唯一，最好是主键，否则数据会增多。



### group by 数据倾斜

同一个key，被发到同一个reduce进行计算

```shell
# 在map端聚合计算一部分
set hive.map.aggr = true 
set hive.groupby.mapaggr.checkinterval = 100000 #聚合条数

#开启两个阶段reduce，前一个打散部分聚合，后一个整体聚合，达到负载均衡的目的
set hive.groupby.skewindata = true 
```



### 小表join大表

已经被hive自动优化



### 大表join大表

```sql
-- 可能是空key或者其他key对应数据过多导致的

-- 空key过滤
select * from 
(select * from nullidtable where id is not null) n 
left join bigtable o 
on n.id = o.id;

--空key转换
select * from nullidtable n 
full join bigtable o 
on nvl(n.id,rand()) = o.id;
```

```sql
-- 通过分桶表优化速度，两个大表关联字段作为分桶字段创建表
-- 开启配置，正常join 即可
set hive.optimize.bucketmapjoin = true;
set hive.optimize.bucketmapjoin.sortedmerge = true; 
set hive.input.format=org.apache.hadoop.hive.ql.io.BucketizedHiveInputFormat;
```



### 数据倾斜

发现数据倾斜 ，查看cm cpu 集群监控 cpu 使用率，当（单台）最大使用率和（集群）平均使用率相差过大的话，即可怀疑发生了数据倾斜。

通过查看cpu使用异常时间，可以找到是哪个脚本任务发生了数据倾斜。

具体分析该脚本倾斜的原因，按照上面的优化方式进行优化。



# Hive 事务

https://www.modb.pro/db/43858

https://www.infoq.cn/article/guide-of-hive-transaction-management

https://blog.csdn.net/weixin_42123844/article/details/125439963

https://blog.csdn.net/ainizfb/article/details/122252311



# Hive on Spark

Hive on Spark 是spark开发的 使用hive运行spark引擎的方式。原理是在hive上集成一个spark。



Hive on Spark：Hive 存储元数据。Spark采用RDD执行。HQL语法，Hive 负责SQL的解析优化。

Spark on Hive : Hive 存储元数据。Spark采用DataFrame执行。Spark SQL语法，Spark 负责SQL解析优化。



修改hive-site.xml

```shell
vim /opt/module/hive/conf/hive-site.xml
```

```xml
<property>
    <name>hive.execution.engine</name>
    <value>spark</value>
</property>

<!--Spark依赖位置，需要提前上传不包含hive的对应spark的jar包到相应位置-->
<property>
    <name>spark.yarn.jars</name>
    <value>hdfs://hadoop102:8020/spark-jars/*</value>
</property>
```



设置spark配置，可以同 spark 的 spark-defaults.conf

```shell
vim /opt/module/hive/conf/spark-defaults.conf
```

```shell
spark.master						yarn
spark.eventLog.enabled				true
spark.executor.memory				1g
spark.driver.memory					1g
```



测试

```shell
bin/hive
```

```shell
create table student(id int, name string);
insert into table student values(1,'abc');
```



# 源码解析

picture/尚硅谷大数据技术之Hive-03（源码）.pdf



# 面试题

picture/尚硅谷大数据技术之Hive-04（面试题）.pdf



























































