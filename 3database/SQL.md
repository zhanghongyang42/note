# DDL(数据定义语言)

### 数据库

创建数据库            		create database 数据库名;

使用数据库            		use 数据库名;

删除数据库            		drop database 数据库名;

查看所有数据库            show databases;

选择数据库   				 selct datebada();



### 数据表

创建表          

```sql
create table 表名(
    列名 数据类型 约束,
    列名 数据类型 约束);
```

  

查看所有表     		 		show tables;

查看表结构					  desc 表名;

复制表							 create table 表名 like 被复制的表名;



删除表         

```sql
drop table 表名;   
delete from 表名;   
truncate table 表名;
```

​            		

修改表

```sql
-- 添加列名称			
alter table 表名 add 列名 数据类型 约束;
-- 修改列名称		
alter table 表名 change 旧列名 新列名 类型 约束;
-- 修改列的数据类型约束	
alter table 表名 modify 列名 数据类型 约束;
-- 删除列					
alter table 表名 drop列名;
```



# DML(数据操作语言)

添加数据      

```sql
intert into 表名 values(全列值),(全列值); 
intert into 表名(列1,列2)values(值1,值2),(值1,值2);
```

修改数据

```sql
update 表名 set 列名 = 值,列名 = 值 where 条件
```

删除数据

```sql
delete from 表名 where 条件; 
```



# DCL (数据控制语言)



# DQL(数据查询语言)

```sql
select [distinct]列名 [as 新名字]
		from 表名 where 条件 
		group by 分组字段 having 分组后条件 
		order by 排序字段 [asc]/desc
```

执行顺序		from--where--group by--having--select--order by

分组后，select只能查分组字段，或者其他字段的聚合。

查询的时候要注意null值进行运算还是null，会被where后的运算过滤掉。

子查询的子表要起别名，不然报错。



运算符

```
+-*/
=、>、<、>=、<=、<>
AND、OR、NOT
```

查询的时候要注意null值进行运算还是null，会被where后的运算过滤掉



关键字

```sql
SELECT * FROM Persons LIMIT 0,5;
SELECT * FROM Persons WHERE LastName IN ('Adams','Carter');
SELECT LastName,FirstName,Address FROM Persons WHERE Address IS NULL;
SELECT * FROM Persons WHERE LastName [NOT] BETWEEN 'Adams' AND 'Carter';
SELECT * FROM Persons WHERE City LIKE '%N%'
```



SQL连接

左右连接时，从表字段一定要是唯一的，不然会使主表字段增多

```sql
-- 笛卡尔积
SELECT * FROM Persons INNER JOIN Orders
-- 内连接
SELECT * FROM Persons Orders ON Persons.Id_P = Orders.Id_P
SELECT * FROM Persons INNER JOIN Orders ON Persons.Id_P = Orders.Id_P
-- 左外链接,左表数据全保留，条件筛选右表
select * from a left outer join b on 条件;   
-- 全连接，两表能连接的连接，不能的保留
select * from a full join b on 条件;    
```



查询后增加一个常量列

```sql
--增加了一个列名为class，值为‘A’的列
SELECT sno, name, 'A' AS class FROM student WHERE eng_score > 80
```



case表达式

```sql
-- case end 是用if，else筛选name值。when then是顺序执行的，所以要注意条件的顺序，上面满足了，下面就不会执行了。
SELECT name
  , CASE 
    WHEN math_score >= 80
    AND eng_score >= 80 THEN '优'
    WHEN math_score >= 60
    AND eng_score >= 60 THEN '良'
    WHEN math_score >= 60
    OR eng_score >= 60 THEN '中'
    WHEN math_score <= 60
    AND eng_score < 60 THEN '差'
    ELSE NULL
  END AS score_grade
FROM student
```



集合运算

```sql
-- 并集
(SELECT * FROM instructor WHERE name='smith')
UNION [all]
(SELECT * FROM instructor WHERE dept_name = 'history');

-- 交集
(SELECT dept_name FROM instructor WHERE name = 'smith')
intersect
(SELECT dept_name FROM department);

--差集
(SELECT dept_name FROM instructor)
EXCEPT 
(SELECT dept_name FROM department WHERE dept_name = 'biology');

-- mysql交集
SELECT DISTINCT dept_name FROM 
instructor
INNER JOIN 
department 
on instructor.dept_name=department.dept_name

-- mysql差集
SELECT dept_name FROM 
department
LEFT JOIN instructor 
on instructor.dept_name=department.dept_name
WHERE instructor.dept_name IS NULL ;
```



累计求和

```sql
-- 两表笛卡尔积，过滤条件主键>=，按求和列分组
SELECT max(a.id),sum(b.money) FROM 
nm a
JOIN 
nm b
ON
a.id>=b.id
GROUP BY a.id

-- 另一种方法见窗口函数
```



# 约束  

```sql
CREATE TABLE Persons(
    Id_P int CHECK (Id_P>0) AUTO_INCREMENT, -- 自增
    LastName varchar(255) NOT NULL,  -- 非空
    FirstName varchar(255) UNIQUE,  -- 唯一
    Address varchar(255) PRIMARY KEY , -- 主键:非空唯一
    City varchar(255) DEFAULT 'Sandnes', -- 默认值
    Id_P int FOREIGN KEY REFERENCES Orders(Id_O) --外键
)
```



```sql
-- 添加约束
ALTER TABLE Persons ADD UNIQUE (Id_P)
-- 删除约束
ALTER TABLE Persons DROP CONSTRAINT Id_P
```



**外键**

```
一个表的外键，一定指向另一个表的主键，两个表的主外键 列相同。
外键值只能是空值，或者是主键中出现的值。
```



# 表关系

一对一，一对多，使用外键即可。

多对多，使用中间表。



# 视图

经常查询的SQL语句结果，可以存储成视图。



# 函数

日期函数

```
now()         	返回当前的日期和时间
curdate()     	返回当前的日期
curtime()     	返回当前的时间
 
date()        	提取日期或日期/时间表达式的日期部分
extract()     	返回日期/时间按的单独部分
 
date_add()    	给日期添加指定的时间间隔
date_sub()    	从日期减去指定的时间间隔
 
datediff()    	返回两个日期之间的天数
date_format()  	用不同的格式显示日期/时间
```

 合计函数

```
avg(column)           返回某列的平均值
count(column)         返回某列的行数（不包括 null 值）
count(*)                             返回被选行数
first(column)         返回在指定的域中第一个记录的值
last(column)          返回在指定的域中最后一个记录的值
max(column)           返回某列的最高值
min(column)           返回某列的最低值
stdev(column)          
stdevp(column)         
sum(column)           返回某列的总和
var(column)            
varp(column)           
```

Scalar 函数

```
ucase(c)                      	将某个域转换为大写
lcase(c)                       	将某个域转换为小写
mid(c,start[,end])              从某个文本域提取字符
len(c)                          返回某个文本域的长度
instr(c,char)                  	返回在某个文本域中指定字符的数值位置
left(c,number_of_char)       	返回某个被请求的文本域的左侧部分
right(c,number_of_char)     	返回某个被请求的文本域的右侧部分
round(c,decimals)             	对某个数值域进行指定小数位数的四舍五入
mod(x,y)                     	返回除法操作的余数
datediff(d,date1,date2)       	用于执行日期计算
format(c,format)              	改变某个域的显示方式
```



# 索引

常用索引原理：B+ 树



**建立原则**

1. 更新频繁的列不应设置索引
2. 数据量小的表不要使用索引（毕竟总共2页的文档，还要目录吗？）
3. 重复数据多的字段不应设为索引（比如性别，只有男和女，一般来说：重复的数据超过百分之十五就不适合建索引）
4. 首先应该考虑对where 和 order by 使用的列上建立索引



创建

```sql
--建表时
CREATE TABLE 表名(
字段名 数据类型 [完整性约束条件],
       ……，
[UNIQUE | FULLTEXT | SPATIAL] INDEX
[索引名](字段名1  [ASC | DESC])
);

--建表后
ALTER TABLE 表名 ADD [UNIQUE| FULLTEXT | SPATIAL]  INDEX | KEY  [索引名] (字段名1 [(长度)] [ASC | DESC]) [USING 索引方法]；
 
--说明：
--UNIQUE:可选。表示索引为唯一性索引。
--FULLTEXT:可选。表示索引为全文索引。
--SPATIAL:可选。表示索引为空间索引。
--INDEX:用于指定字段为索引。
--索引名:可选。给创建的索引取一个新名称。
--字段名1:指定索引对应的字段的名称，该字段必须是前面定义好的字段。

--示例
CREATE TABLE classInfo(
    id INT AUTO_INCREMENT COMMENT 'id',
    classname VARCHAR(128) COMMENT '课程名称',
    classid INT COMMENT '课程id',
    classtype VARCHAR(128) COMMENT '课程类型',
    classcode VARCHAR(128) COMMENT '课程代码',
-- 主键本身也是一种索引
    PRIMARY KEY (id),
-- 给classid字段创建了唯一索引(注:也可以在上面创建字段时使用unique来创建唯一索引)
    UNIQUE INDEX (classid),
-- 给classname字段创建普通索引
    INDEX (classname),
-- 创建组合索引
    INDEX (classtype,classcode)
);

ALTER TABLE classInfo ADD UNIQUE INDEX (classid);
```

 删查

```sql
drop index classname on classInfo;
show index from classInfo;
```



# 优化

### explain



可以返回字段供我们分析查询速度：

```sql
explain select * from user_info where id = 2
explain format=json select * from user_info where id = 2
```



**select_type**表示查询的类型，它的常用取值有:

1. SIMPLE，表示此查询不包含 UNION 查询或子查询。
2. UNION, 表示此查询是使用UNION语句的第二个或后面的SELECT。
3. SUBQUERY, 子查询中的第一个 SELECT。
4. 。。。。。。



### show processlist

返回进程供我们分析



### 其他

```sql
--查看每个客户端的连接数
select client_ip,count(client_ip) as client_num 
from (
     selectsubstring_index(host,':' ,1) as client_ip 
     fromprocesslist ) as connect_info 
group by client_ip 
order by client_num desc;

--查看进程时间
select * 
from information_schema.processlist 
where Command != 'Sleep' 
order by Time desc;

--杀死超时进程
select concat('kill ', id, ';')
from information_schema.processlist 
where Command != 'Sleep' and Time > 300
order by Time desc;
```

 

# 常用sql

https://mp.weixin.qq.com/s/Z3xknCQEpuZRzQ5AiZaRlg







 



 





































