<img src="picture/numpy速查表-1596592522285.png" alt="numpy速查表" style="zoom:200%;" />



官网：https://www.numpy.org.cn/

教程：https://www.runoob.com/numpy/numpy-tutorial.html



# ndarray

ndarray 对象是用于存放同类型元素的多维数组。



# 创建

```python
import numpy as np

a = np.array([[4, 5], [6, 1]])
a = np.array([1, 2, 3, 4, 5],ndmin=2,dtype=complex)		#【最小维度2维】 【数据类型复数】

x = np.zeros(5) 
x = np.ones(5)
z = np.zeros((2,2), dtype = [('x', 'i4'), ('y', 'i4')])
x = np.arange(5)
x = np.arange(10,20,2) 
a = np.random.random(5)
x=np.fromiter(iter(list), dtype=float)	#从iter创建 ndarray

# 每一列定义名称和类型，名称可不写
student = np.dtype([('name',np.string_,16), ('age', np.int_), ('marks', np.float_)]) 
a = np.array([('abc', 21, 50),('xyz', 18, 75)], dtype = student) 

#生成等比数列
a = np.logspace(1, 4,num = 4 ,base=2)
```



# 属性

```python
import numpy as np  
a = np.array([[1,2,3],[4,5,6]])

#维度
print (a.ndim) 

#形状
print (a.shape)
b = a.reshape(2,4,3,-1)

#类型：内置数据类型 bool_ 、int_ 、float_ 、complex_ 、str_ 、object_
print(a.dtype)
c = b.astype(int8)

#其他
print(a.size) 	# 25
print(a.itemsize) # 每个元素8
print(a.nbytes)	# 总字节200
```



# 增删改查

通过切片和索引

```python
import numpy as np
a = np.arange(10)

# 查
a[2:7:2]
x[x>5]
a[~np.isnan(a)]

#改
a[1:][1:]=1

# 数组索引
x=np.arange(32).reshape((8,4))
y = x[[0,1,2],  [0,1,0]]
```

axis=0操作最外层，1是第二层，一直到最里层，比如shape（5，6，2）的数组，axis=0操作最外层的5个，axis=1操作第二层的30个，axis=2操作所有

```python
#删
x = np.delete(x, [0,4]，axis=0)

#增				
q = np.append(Y,[[9,99],[10,100]], axis=1) 		
w = np.insert(Y,1,[4,5,6],axis=0)

#去重
u, counts = np.unique(a, return_counts=True)

#删除只有一个元素的维度
y = np.squeeze(x
```



# 函数-数组操作

```python
a.flatten()
a.T

np.rollaxis(a,2,1)	#【2轴滚动到1轴前， 即某一3维数组元素下标（001）变（010）】
np.swapaxes(a, 2, 0)  #【交换数组的两个轴，原理同上】

#连接数组
print (np.stack((a,b),1))  #【堆叠：两个维度相同数组（3，4），沿着1轴堆叠变（3，2，4）】【a第一个元素坐标变为（0,0,0）b第一个元素坐标变为（0,1,0）】
print (np.concatenate((a,b),axis = 1)) 	#【连接：两个维度相同数组（3，4），沿着1轴连接变（3，8）】【contenate可以 reshape 成 stack 效果】

#切割数组
b = np.split(a,3) 		【一维，将数组均分成3份】
b = np.split(a,[4,7])	【一维，将数组按位置切割成3份】
b = np.hsplit(harr, 3） 【二维，将数组按水平轴切割成3份新数组】
b = np.vsplit(a,2)		【二维，将数组按垂直轴切割成3份新数组】
```



# 函数-统计

字符串函数如np.char.center()， 数学函数如np.ceil(a)  ,numpy都包装了一下

```python
np.reciprocal(a)	#取倒数
np.amin(a,0)
np.amax(a,axis=1)
np.ptp(a, axis =  1) 	#【最大值减最小值】
np.percentile(a, 50)	
np.median(a)	
np.mean(a)		
np.average(a,weights = wts)  #【加权平均值，wts是各元素对应权重】
np.std([1,2,3,4])   
np.var([1,2,3,4])	
a.cumsum()
```



# 函数-筛选排序

```python
# 排序
np.sort(a, order='name')
np.lexsort((dv,nm)) 		#按照多列排序

# 过滤
np.extract(np.mod(x,2)==0, x)	#【x中满足条件的返回】
```



# 线性代数运算

https://www.runoob.com/numpy/numpy-linear-algebra.html

```python
import numpy as np
 
print(np.dot(a,b))			【第n行和第n列对应位置的数相乘求和】【得到一个矩阵】【矩阵乘法】
print (np.vdot(a,b))		【对应位置相乘求和（内积）】【得到一个数】
print (np.inner(a,b))		【第n行和第n行对应位置的数相乘求和】【得到一个矩阵】
print (np.linalg.det(a))	【求解行列式】
```

求解线性方程

```python
AX = B
X = A^(-1)B

a = np.array([[1,1,1],[0,2,5],[2,5,-1]]) 
ainv = np.linalg.inv(a)					【求解A逆】
b = np.array([[6],[-4],[27]]) 
x = np.linalg.solve(a,b) 				【X = A^(-1)B】
```



# 保存加载

```python
np.save('outfile.npy',a) 
np.savez("runoob.npz", a, b,)
b = np.load('outfile.npy') 

np.savetxt('out.txt',a) 
b = np.loadtxt('out.txt')  
```


