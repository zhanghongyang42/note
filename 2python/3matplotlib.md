官网：https://www.matplotlib.org.cn/



# 基本用法

```python
import numpy as np
import matplotlib.pyplot as plt

x = np.linspace(-3, 3, 50)
y1 = 2*x + 1
y2 = x**2
```

```python
plt.figure(figsize=(8, 5))
plt.plot(x, y1, color='red', linewidth=1.0, linestyle='--')		【曲线宽度，曲线类型】
plt.xlim((-1, 2))								【x轴显示范围】
plt.ylim((-2, 3))
plt.xlabel('I am x')
plt.ylabel('I am y')
plt.yticks([-2,3],['really bad','really good'])		【定义要显示的y轴坐标数，给坐标起别名】
ax = plt.gca()										【获取坐标轴】
ax.spines['left'].set_color('red')					【设置坐标轴颜色】
ax.spines['bottom'].set_position(('data', 0))		【将x轴放到y=0的地方】
ax.xaxis.set_ticks_position('top')					【设置x坐标数显示位置】
plt.show()
```



# 图例与标注

```python
import numpy as np
import matplotlib.pyplot as plt

x = np.linspace(-3,3,50)
y1 = 2*x+1
y2 = pow(x,2)

plt.figure()

plt.xlim((-1,2))
plt.ylim((-2,3))
new_sticks = np.linspace(-1,2,5)
plt.xticks(new_sticks)
plt.yticks([-2, -1.8, -1, 1.22, 3],['really bad','bad','normal','good','really good'])

l1 = plt.plot(x,y1,label='linear line')
l2 = plt.plot(x,y2,color='red',linewidth=1.0,linestyle='--',label='square line')
#图例
plt.legend()						

plt.show()
```

```python
import numpy as np
import matplotlib.pyplot as plt

x0 = 1
y0 = 2*x0 + 1

plt.figure(num=1, figsize=(8, 5),)

ax = plt.gca()
ax.spines['right'].set_color('none')
ax.xaxis.set_ticks_position('bottom')
ax.spines['bottom'].set_position(('data', 0))
ax.spines['left'].set_position(('data', 0))

plt.plot([x0, x0,], [0, y0,], linewidth=2.5)		【绘制一个线段】

plt.scatter([x0, ], [y0, ], s=50, color='b')		【绘制一个点】

#标注
plt.annotate(r'$2x+1=%s$' % y0, xy=(x0, y0), xytext=(+30, -30),textcoords='offset points', 					fontsize=16，xycoords='data')	【显示内容，显示位置，偏置，是否显示，字体，默认参数】

plt.show()
```

显示刻度，不被图像遮挡

```python
import numpy as np
import matplotlib.pyplot as plt

x = np.linspace(-3, 3, 50)
y = 0.1*x

plt.figure()

plt.plot(x, y, linewidth=10, zorder=1)
plt.ylim(-2, 2)
ax = plt.gca()
ax.spines['bottom'].set_position(('data', 0))
ax.spines['left'].set_position(('data', 0))

for label in ax.get_xticklabels() + ax.get_yticklabels():			#获取xy所有刻度
    label.set_fontsize(12)	
    #设置透明度，背景白，边框无，透明度0.7
    label.set_bbox(dict(facecolor='white', edgecolor='None', alpha=0.7))
   
plt.show()
```



# 绘图

### 1.折线图/散点图/条形图

曲线图

```
from matplotlib import pyplot as plt
year = [2010, 2012, 2014, 2016]
people = [20, 40, 60, 100]
plt.plot(year, people)
```

散点图

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

条形图-双变量-y值直接对应x

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



### 2.饼图

显示一个特征各个值得比例

```python
import matplotlib.pyplot as plt
plt.rcParams['font.sans-serif']=['SimHei'] #用来正常显示中文标签

labels = ['娱乐','育儿','饮食','房贷','交通','其它']
sizes = [2,5,12,70,2,9]
plt.pie(sizes,labels=labels,autopct='%1.1f%%')
plt.show() 
```



### 其他

等高线图

https://www.kesci.com/home/project/5b7fce5631902f000f5cfa87

```python
#需要x，y，z三个坐标画等高线图， 在xy平面上，连接相同z即为等高线
import matplotlib.pyplot as plt
import numpy as np

#生成z
def f(x,y):
    return (1 - x / 2 + x**5 + y**3) * np.exp(-x**2 -y**2)

n = 256
x = np.linspace(-3, 3, n)
y = np.linspace(-3, 3, n)
X,Y = np.meshgrid(x, y)         #生成256*256个点，每个点坐标分成x，y返回了

plt.contourf(X, Y, f(X, Y), 8, alpha=.75, cmap=plt.cm.RdBu)  #画等高线图

plt.show()
```

像素点画图

```python
import matplotlib.pyplot as plt
import numpy as np

a = np.array([0.313660827978, 0.365348418405, 0.423733120134,
              0.365348418405, 0.439599930621, 0.525083754405,
              0.423733120134, 0.525083754405, 0.651536351379]).reshape(3,3)

plt.imshow(a, interpolation='nearest', cmap='RdBu', origin='lower')  #a代表像素点矩阵
plt.colorbar(shrink=.92)        #图例

plt.show()
```



# 多图合并显示

```python
import matplotlib.pyplot as plt
plt.figure()

#会自动把一列变得和两列一样宽
plt.subplot(2,1,1)
plt.plot([0,1],[0,1])

plt.subplot(223)
plt.plot([0,1],[0,3])

plt.subplot(224)
plt.plot([0,1],[0,4])

plt.show()
```



图中图

```python
import matplotlib.pyplot as plt

x = [1, 2, 3, 4, 5, 6, 7]
y = [1, 3, 4, 2, 5, 8, 6]

fig = plt.figure()

#根据坐标确定绘图位置，起点（1，1），长8 宽8 的矩形位置。默认10*10画布，否则为比例
left, bottom, width, height = 0.1, 0.1, 0.8, 0.8
ax1 = fig.add_axes([left, bottom, width, height])
ax1.plot(x, y, 'r')

left, bottom, width, height = 0.2, 0.6, 0.25, 0.25
ax2 = fig.add_axes([left, bottom, width, height])
ax2.plot(y, x, 'b')

plt.show()
```



# 保存图片

```python
plt.savefig('resultPlot.png', format="png")
```



# 3D作图与动画

https://www.kesci.com/home/project/5b81491d31902f000f5ece52



# 问题

绘图时需要看的x轴名称太多 互相遮挡怎么办

​	1.删掉一些x坐标，或者不显示x坐标：比如热力图只找到相关性大于0.3的特征进行绘制

​	2.x，y调换

​	3.调整坐标，旋转显示角度，减少显示数量

```python
plt.yticks([-2,3],['really bad','really good'],rotation=90)		#定义要显示的y轴坐标数，给坐标起别名
```


