探索性数据分析最重要的就是深入理解每一列数据的业务含义





# AUTO_EDA

查看缺失值，结合分析出异常值，0值，查看高基数特征，查看整体相关性

```python
import numpy as np
import pandas as pd
import pandas_profiling
import matplotlib.pyplot as plt
import seaborn as sns
import itertools
df = pd.read_csv('churn.csv')

# pycharm执行代码
pfr = pandas_profiling.ProfileReport(df)
pfr.to_file("./example.html")
 
# 在jupyter执行代码
# pandas_profiling.ProfileReport(df)
```



# 基本查看

```python
df.head()
df.tail()
df.sample()

df.shape

df.info()

df.dtypes

df.describe()
```



# 正负样本比

```python
df['Survived'].value_counts()
```

画图

```python
df_vc = df['Survived'].value_counts()
plt.figure()
plt.pie(df_vc,labels=df_vc.index,autopct='%1.1f%%')
plt.show()
```



# 单列数据分析

频度统计

```python
df['a'].value_counts().sort_values()
df['a'].value_counts().count()
```

统计列值比例

```python
#选出值的种类少于总数一半的特征
df_i = [] 
for i in df: 
    if len(df[i].unique())<(len(df)/2):
        df_i.append(i)  
#选出某一个值占比超过50%的特征
list_0 = []
for i in df[df_i]:
    for j in df[i].value_counts():
        if (j/len(df[i]))>0.5:
            list_0.append(i)  
            print(i)
            continue
#选出值种类在5，1之间的特征和上面的，一起画比例图
for i in df:     
    if ((len(df[i].unique())<5) & (len(df[i].unique())>1)) | (i in list_0) :
        print()
        print("*******************查看特征",i,"比例***********************")
        print(df[i].value_counts())
        with sns.axes_style():
            df[i].value_counts().plot(x=None,y=None,kind='pie',autopct='%1.2f%%')
            plt.show()
    if len(df[i].unique())==2:
        print(str(round((df[i].value_counts()[0]/df[i].value_counts()[1]),2))+'倍')
        print()
        print()
del list_0,df_i
```

数据分布

```python
for i in df:
    if df[i].dtypes!='object':
    	print("*******************",i,"***********************")
    	sns.kdeplot(df[i], label=str(i))
    	plt.show()
```

统计指标

```python
categorical = pd.get_dummies(df.select_dtypes('object'))
categorical['Pclass'] = df['Pclass']
df = pd.merge(df,categorical,on='Pclass')

aa = df.groupby('Pclass').agg(['count', 'mean', 'max', 'min', 'sum']).reset_index()

columns = ['Pclass']
for var in aa:
    if var[0] != 'Pclass': 
        columns.append('%s_%s' % var)
aa.columns = columns
```



# 特征间相关性分析

0-0.09为没有相关性,0.5-1.0为强相关

```python
#所有相关性，类别型数据不计算
df.corr()

#相关性大于0.3的特征
temp = df.corr()
high_corr = []
for i in list(itertools.combinations(temp.columns,2)):
    if abs(temp.loc[i[0],i[1]])>0.3:
        high_corr.append(i)
print(high_corr)
del temp

#整体热力图
import seaborn as sns
import matplotlib.pyplot as plt
def heatmap_plot(df):
    dfcorr = df.corr()
    plt.figure()
    sns.heatmap(dfcorr, annot=True, vmax=1, square=True, cmap="Blues")
    plt.show()
heatmap_plot(df)

#相关性大于正负0.3的热力图
temp = df.corr()
temp = temp[(abs(temp)>0.3) & (abs(temp)<1)]
for i in temp:
    if len(temp[i].value_counts())==0:
        temp.drop([i],axis=1,inplace=True)
        temp.drop([i],axis=0,inplace=True)
temp = list(temp.columns)
temp = df[temp].corr()
sns.heatmap(temp)
del temp

#取出相关性高的两个特征
import itertools
corr = df.corr()
for col_name_2 in list(itertools.combinations(corr.columns,2)):
    if abs(corr.loc[col_name_2[0],col_name_2[1]])>0.5:
        print(col_name_2) 
```

```python
#通过画核密度图区分
import seaborn as sns
import matplotlib.pyplot as plt
def kde_target(df,lable,var_name):
    plt.figure()
    for i in df[lable].unique():
        sns.kdeplot(df.loc[df[lable]==i, var_name], label=str(i))
    plt.xlabel(var_name); plt.ylabel('Density'); plt.title('%s Distribution' % var_name)
    plt.legend()
    plt.show()

for i in df:
    if (df[i].dtypes!='object') & (i!='Survived'):
        kde_target(df,'Survived',i)
```

```python
#交叉表和分类直方图
for i in df:
    if (len(df[i].unique())<5) & (i!='Survived'):
        print("*******************",i,"***********************")
        print(pd.crosstab(df[i],df["Survived"]))
        with sns.axes_style():
            plt.figure()
            sns.countplot(x=i,hue="Survived",data=df)
            plt.show()
```







































