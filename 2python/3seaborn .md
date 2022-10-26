官方文档：https://www.cntofu.com/book/172/index.html
教程：https://zhuanlan.zhihu.com/p/33977558



# 区别记忆

### 直方图、条形图、柱状图区别

直方图是条形图（柱状图）的连续分组，是横坐标的不同

横坐标：条形图是一个数，直方图是连续值分组。条形图无序，直方图有序

纵坐标：一个变量（直方图面积为数量，条形图y数量） 。两个变量各种统计

用途：直方图强调的是数值连续的变化规律（数据分布情况）,条形图强调各组的数值大小



### 按what分

**单变量图**

饼图：x无序。x不变（离散），y是 x值/x总值 														plt.pie(x)

条形图（单变量）：x无序。x唯一y数量											  					sns.countplot(x)

直方图：xy有序。x分组 ，y是频数，面积是数量													 sns.distplot(x)

核密度图：上图 更好的显示																						  sns.kdeplot(x)



箱型图：y无意义，x有序。x分位点。反应异常值情况。										sns.boxplot(x)



**双变量图**

曲线图：xy有序。xy无需加工																						plt.plot(x,y)

散点图：xy有序。xy无需加工																						plt.scatter(x,y)

条形图（双变量-x值y值）：xy有序。xy无需加工														plt.bar(x,y)

条形图 （双变量-x唯一y均值）：x无序。x唯一y指标												sns.barplot(x=x,y=y,data=df)



分簇散点图：x无序。x唯一y不变																				 sns.swarmplot(x=x,y = y,data=df)



**多变量图**

热力图：xy无序。xy是特征																							sns.heatmap(corr) 

点对图：直方图、核密度图、散点图的结合																sns.pairplot(df)



### 按how分

**指标显示**

饼图：显示不分组时x值占比指标															plt.pie(x)

条形图（单变量-x唯一y数量）：显示分组后该变量指标					sns.countplot(x)



条形图 （双变量-x唯一y均值）：显示分组后其他变量指标				sns.barplot(x=x,y=y,data=df)



**分布情况**

直方图（单变量-y频数）：反应数据分布情况										sns.distplot(x)

核密度图：反应数据分布情况																	sns.kdeplot(x)



曲线图：反应出数据变化的趋势																  plt.plot(x,y)

条形图（双变量-x值y值）：反应出数据变化的趋势								 plt.bar(x,y)

分簇散点图：反应数据在不同x时分布情况												sns.swarmplot(x=x,y = y,data=df)



**异常值情况**

箱型图：反应异常值情况																				sns.boxplot(x)



**相关性**

散点图：反应xy的相关性																				sns.lmplot(x=x,y =y,data=df)



热力图：反应数据相关性																				sns.heatmap(corr) 

点对图：反应数据相关性和分布情况															sns.pairplot(df)





# 绘图



## 准备

```python
import numpy as np
from scipy.stats import *
import matplotlib.pyplot as plt
import seaborn as sns
import pandas as pd
df = pd.read_csv('train.csv')
```



## 单变量图

### 饼图

```python
import matplotlib.pyplot as plt
plt.rcParams['font.sans-serif']=['SimHei'] #用来正常显示中文标签

labels = ['娱乐','育儿','饮食','房贷','交通','其它']
sizes = [2,5,12,70,2,9]
plt.pie(sizes,labels=labels,autopct='%1.1f%%')
plt.show() 
```



### 条形图（单变量）

```python
sns.countplot(df['Fare'])
```

![png](picture/output_24_1-1596595778347.png)

### 直方图

```python
sns.distplot(df['Fare'],bins=100,fit=norm)
```

![png](picture/output_20_1-1596595786933.png)

### 核密度图

```python
sns.kdeplot(df['Fare'])
```

![png](picture/output_21_1-1596595795447.png)

```python
sns.kdeplot(df['Fare'],cumulative=True) #累积分布
```

![png](picture/output_22_1-1596595807551.png)

### 箱型图

```python
sns.boxplot(df['Fare'])
```

![png](picture/output_28_1-1596595812588.png)





## 双变量图

### 曲线图

```
from matplotlib import pyplot as plt
year = [2010, 2012, 2014, 2016]
people = [20, 40, 60, 100]
plt.plot(year, people)
```



### 散点图

```python
import matplotlib.pyplot as plt
import numpy as np

n = 1024   
X = np.random.normal(0, 1, n) # 每一个点的X值
Y = np.random.normal(0, 1, n) # 每一个点的Y值
T = np.arctan2(Y,X) 

plt.scatter(X, Y, s=75, c=colors)	#散点图，颜色可以是一个 或 和 点数量相同数量的值
plt.xlim(-1.5, 1.5)
plt.ylim(-1.5, 1.5)

plt.show()
```

```python
sns.jointplot(x="Age",y = "Fare",data=df)
```

![png](picture/output_7_1-1596595823440.png)

带回归线的散点图

```python
sns.lmplot(x="Age",y = "Fare",data=df)
```

![png](picture/output_9_1-1596595828607.png)

### 条形图（双变量-x值y值）

```python
import matplotlib.pyplot as plt
import numpy as np

n = 12
X = np.arange(n)
Y1 = (1 - X / float(n)) * np.random.uniform(0.5, 1.0, n)

plt.bar(X, +Y1, facecolor='#FFCCCC')	#柱状图

#给每个数据进行标注
for x, y in zip(X, Y1):
    plt.annotate('%.2f'%y, xy=(x, y), xytext=(-20, +10),textcoords='offset points', fontsize=16,xycoords='data', )    
    
plt.show()
```



### 条形图 （双变量-x唯一y均值）

```python
sns.barplot(x='Pclass',y="Survived",data=df)
```

![png](picture/output_26_1-1596595837367.png)

### 分簇散点图

```python
sns.swarmplot(x="Survived",y = "Age",data=df)#x是类别变量得散点图，并且点不重合   
```

![png](picture/output_10_2-1596595842704.png)





## 多变量图

### 热力图

```python
corr = df.corr()
sns.heatmap(corr) 
```

![png](picture/output_30_1-1596595846853.png)

### 点对图

```python
sns.pairplot(df[["Survived","Age","Fare","Pclass"]])
```

![png](picture/output_12_2-1596595851073.png)

```python
sns.pairplot(df[["Survived","Age","Fare","Pclass"]],kind="reg",diag_kind="kde")
```



![png](picture/output_13_2-1596595856719.png)











































​												



​														



