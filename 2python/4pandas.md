pandas是数据科学的核心包。以他为基础结合numpy、matplotlib等包来使用

![pandas速查](picture/pandas速查-1596596734569.jpg)



官网：https://www.pypandas.cn/

教程：https://www.zhihu.com/question/56310477/answer/797336453

教程：https://github.com/datawhalechina/joyful-pandas



# DataFrame（表）

### 读取/保存

```Python
df = pd.read_csv('aaa.csv')
df = pd.read_excel('aaa.xls')
```

```Python
df.to_csv('bbb.csv')
df.to_excel('bbb.xlsx')
```



### 创建

一般从**非结构化文件解析后**的数据需要用到这个创建方法

```python
#从list创建
df = pd.DataFrame([[]])

#从dict创建
df2 = pd.DataFrame({'F': 1,'A'：[10,100,1000,1], 'D': np.array([3] * 4), 'C': pd.Series(1, index=list(range(4))), 'B': pd.Timestamp('20130102'),'E': pd.Categorical(["test", "train", "test", "train"])})

#从ndarray创建
df = pd.DataFrame(np.random.randn(6, 4), index=pd.date_range('20130101',periods=6), columns=list('ABCD'))

#从tuple创建
#从series创建
```



### 表拼接

行拼接（纵向合并）

```python
df = pd.concat([df1,df2,df3])
```

列拼接（横向合并）

```python
df = pd.merge(left=h1,right=h2,left_index=True,right_index=True,how='inner') #行标签连接
df = pd.merge(left=h1,right=h2,left_on='a',right_on='a',how='inner')  #某一列连接
```



### 查看

```Python
df.head()
df.info()
df.describe()
df.dtypes
df.shape
df.sample(10)
```

```Python
df['访客数'].unique()
df['访客数'].value_counts()
df.sort_values(by='支付金额'，ascending=False)
```



# 增删改查

### 查

**iloc**	基于数字索引（位置）

```python
df.iloc[:100,:]
df.iloc[:,[0,4]]
```



**loc**	基于行列标签名称

```python
df.loc[:, ['A', 'B']]
df.loc[df['流量'] == '一级',:]
df.loc[df['E'].isin(['two', 'four']),:]
df.loc[df['a']>df['a'].mean() & df['b']>df['b'].mean(),:]
```



基于标签**快速**查找列

```python
df[['工资','姓名']]
```



多简单条件查询

```python
df.query("(a>0 and b<0.05) or c>b")
```



### 删

```python
#基于 查  删除，选择不删除的数据。
df.loc[df['流量'] == '一级',:]

#直接选择删除的列
df.drop(['新列']，axis=1,inplace=True)
```



### 增/改/替换

基于 查  增/改/替换，可以是 行 或 列 

```python 
#增加、替换行
df.loc[9, :] = [1,2,3,'22','aaa',5.7]

#增加、替换列
df.loc[:, 'aa'] = 1

#修改单个元素
df.loc[9, 'aa'] = 1
```

根据标签快速增加替换列

```python
df['date'] = pd.date_range('20130101',periods=6)
df['date']  =  pd.Series([1, 2, 3, 4, 5, 6], index=pd.date_range('20130102', periods=6))
```



# 操作列

### 类型转换

pandas  dtype  包括  【**object，category，int64，float64，bool，datetime64，timedelta**】   7种数据类型

对应关系如下

![9ebd4c2bgy1fto9860xy9j20sl0bfgrk](picture/9ebd4c2bgy1fto9860xy9j20sl0bfgrk.jpg)



数据类型转换方法有3种：

1.astype

```python
df["Customer Number"] = df["Customer Number"].astype("int64")
```



2.运用apply

```Python
df['aa'] = df['aa'].apply(lambda x:str(x))
df['bb'] = df['bb'].apply(lambda x:float(x))
#更复杂的处理需要把lambda表达式换成自定义函数
```



3.转换成时间类型

```Python
df["Start_date"] = pd.to_datetime(df[['Month', 'Day', 'Year']])
```



### 操作不同类型

```python
#字符串
df['地址'] = df['地址'].str.replace('-','')		#加上.str  操作同原生字符串操作

#数值
df['访客数'] = df['访客数']+1000+df['工资']		#直接计算

#时间类型
df['天数'] = pd.to_datetime('2019-12-31') - df['日期']  	#时间类型数据直接相减
```



### 根据类型选择列

```python
df.select_dtypes('object')
```



### 指标计算

```Python
df['访客数'].mean()
df['访客数'].median()
df['访客数'].max()
df['访客数'].min()
df['访客数'].std()	
df['访客数'].size()
```



# 功能操作

### 列重命名

```Python
rfmTable.rename(columns={'hist': 'recency',  'customer_id': 'frequency'}, inplace=True)
```



### 空值处理

```python
#填充
df['aa'] = df['aa'].fillna(df['aa'].mean())

#删除
df = df.dropna()
```



### 重复值处理

```python
df = df.drop_duplicates(['city'],keep='last')
```



### 哑编码处理

```python
train = pd.get_dummies(train)
```



### 分箱处理

```python
#等宽分箱
df['Fare'] = pd.cut(df['Fare'], 4, labels=range(4))

#等频分箱
df['Fare'] = pd.qcut(df['Fare'],4,labels=['bad','medium','good','awesome'])

#自定义分箱
bins=[0,200,1000,5000,10000]
df['amount1']=pd.cut(df['amount'],bins)
```



# 高级操作

### apply与lambda

```Python
# 对df所有元素进行操作
df.applymap(lambda x: x+1 if x%2==0 else x)

# 对某一列的元素进行操作
frame["id"].apply(lambda x:(x,'ok'))
```



### 分组与透视

分组

```python
#内置函数
df.groupby('aa',as_index=False).sum()	
table = df.groupby(['A', 'B']).agg({'hist': 'min',  'customer_id': 'count',  'tran_amount':'sum'})

#apply中的函数一定是可以操作一组数据的函数
table = df.groupby('Cabin',as_index=False)['Fare'].apply(max)
```

透视

```python
df = pd.pivot_table(df, values=['金额'], index=['类别'], columns=['月份'],aggfunc=np.sum)
```



### 堆叠与索引

堆叠

```python
# 把所有列变成2级索引，共两级索引
stacked = df.stack()

# 还原
stacked = stacked.unstack()  
```



多级行列索引

```Python
#创建		这里以列索引为例
df1 = pd.DataFrame(np.random.randint(80, 120, size=(2, 4)),
                   index= ['girl', 'boy'],
                   columns=[['English', 'English', 'Chinese', 'Chinese'],
                         ['like', 'dislike', 'like', 'dislike']])

#查找		多个列名称的索引等级必须对应，多级索引以元祖代替列名称
df1.loc[:,[('English','like'),('Chinese','like')]]
df1.loc[:,['English','Chinese']]

#行列索引互转
df.set_index('date', inplace=True) #用某一列数据的值来代替索引值，并删掉之前的索引
df.reset_index(inplace=True)	#把索引值还原成一列数据
```



### 爆炸函数

https://www.cnblogs.com/hider/p/15627064.html



# 日期相关

```Python
import pandas as pd

# 日期
pd.date_range('2020/11/21', periods=5)
pd.date_range('2020/11/21', periods=5,freq='M')

# 时间差
timediff = pd.Timedelta('2 days 2 hours 15 minutes 30 seconds')
timediff = pd.Timedelta(6,unit='h')

# 运算
df = pd.DataFrame(dict(A = pd.Series(pd.date_range('2018-1-1', periods=3, freq='D')), 
                       B = pd.Series([pd.Timedelta(days=i) for i in range(3) ])))
df['C']=df['A']+df['B']
```



# 多进程

Pandarallel：https://mp.weixin.qq.com/s/9tZLmwCEdUPjfQyvmR3SEA



















