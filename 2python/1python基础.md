官网：https://docs.python.org/zh-cn/3.8/tutorial/index.html

教程：https://www.liaoxuefeng.com/wiki/1016959663602400



# 小提示

```python
# 单行注释

''' 
多行注释,
保留格式。
'''   
```



```python
if ( a is b ): print ("1 - a 和 b 有相同的标识")
# is 用于判断两个变量引用对象是否为同一个， == 用于判断引用变量的值是否相等。
```



# 关键字

```python
#列出所有关键字35个，关键字的用法在其他内容中
import keyword
print(keyword.kwlist)

# import,from，as
# class,def
# return
# del,global，nonlocal（标识该变量是上一级函数中的局部变量，定义了变量，但是定义的变量并不是新变量）

# if，elif，else，for，in，while
# break,continue，pass（起站位作用）

# try，except，finally，raise
# with

# False,True
# None   方法没有返回值的话，默认返回None
# and,or，not，is

# assert 断言
# yield  函数会从yield位置返回值，停止执行函数，保存信息，并在next(fun())继续执行
# lambda 匿名函数

# async，await 协程
```



# 内置函数

```python
#python内置函数68个

#计算
abs(x)
divmod(7, 2) 【得到商和余数】
max(80, 100)
min(80, 100)
pow(100, 2)【100的平方】
round(80.264, 2)【四舍五入】
sum([0,1,2])

range(0, 5)

#类型转换
int(3)
float(x)
complex(1, 2)
bool(0)
str(s)
chr(97)
ord('A')

dict(a='a', b='b')
list( seq )
set('google')
tuple( iterable )

bin(3) 【转二进制】
hex(x) 【转十六进制】
oct(x) 【转八进制】

bytearray('dd') 【转字节码】
bytes(1)

#数据处理
filter(function, iterable)
map(function, iterable)

#操作迭代器
all(iterable) 【迭代器有假，返回False】
any(iterable) 【真，True】

sorted(iterable, key=None, reverse=False)   【排序】
reversed(seq) 				【反转序列】

next(iterator,'last') 		【每次输出一次迭代器，最后输出第二个参数】

#操作对象
len( s ) 【对象长度】

delattr(object, name) 【删除一个对象的属性】
setattr(object, name, value) 【设置属性值】
getattr(object, name) 【查看一个对象属性值】
dir(object) 【查看一个对象或者模块的所有属性】
hasattr(object, name) 【查看对象是否包含该属性】

id(object) 【得到对象的内存地址】

callable(object) 【类和方法可调用】【对象是否有callable方法】

issubclass(B,A) b是a的子类，返回True

#获取变量
globals() 【返回所有全局变量的词典】 
locals() 【返回当前位置所有局部变量的词典】
vars([object]) 【返回对象的所有属性的字典，不填对象就是locals()】

#输入输出
input("input:") 【输入】
print('gg') 【输出】

#其他
# breakpoint 调试器
# slice(start, stop[, step]) 切片函数
# __import__('a')  动态导入，import调用了这个方法
# ascii(object)， memoryview(object)，property(),repr(object)
```

enumerate

```python
seasons = ['Spring', 'Summer', 'Fall', 'Winter']
list(enumerate(seasons, start=1))
>>[(1, 'Spring'), (2, 'Summer'), (3, 'Fall'), (4, 'Winter')]
```

eval()/exec()

```python
eval()函数只能计算单个表达式的值，而exec()函数可以动态运行代码段。
eval()函数可以有返回值，而exec()函数返回值永远为None。

x = 1
eval('x+1')
>>2

x = 1
exec("x+1")
>>2
```

format

```python
print('站点列表 {0}, {1}, 和 {other}。'.format('Google', 'Runoob', other='Taobao'))
print('{0:10} ==> {1:10d}'.format(name, number))  #1是位置，10是长度
```

frozenset

```python
#生成不可变集合，当集合要作为另一个集合的元素时需要变成不可变集合
a = frozenset(range(10)) 
b = frozenset('runoob') 
```

hash

```python
#hash() 函数可以应用于数字、字符串和对象，不能直接应用于 list、set、dictionary，要先强转str
name1='正常程序代码'
name2='正常程序代码带病毒'
print(hash(name1)) # 2403189487915500087
print(hash(name2)) # -8751655075885266653
```

isinstance()/type()

```python
#type()不会认为子类是一种父类类型。isinstance()会认为子类是一种父类类型。
class A:
	pass
class B(A):
	pass
isinstance(A(), A)
True
type(A()) == A 
True
isinstance(B(), A)
True
type(B()) == A
False
```

iter

```python
from random import randint
def guess():
    return randint(0, 10)
num = 1
# 这里先写死心里想的数为5
for i in iter(guess, 5):
    print("第%s次猜测，猜测数字为: %s" % (num, i))
    num += 1
# 当 guess 返回的是 5 时，会抛出异常 StopIteration，但 for 循环会处理异常，即会结束循环
```

open

```python
open(file, mode='r')

f = open('test.txt')
f.read()
```

zip

```python
a = [1,2,3]
b = [4,5,6]
c = [4,5,6,7,8]
list(zip(a,c))		#[(1, 4), (2, 5), (3, 6)]
 
a1, a2 = zip(*zip(a,b))          # zip(*)，对zip更深一层的维度进行zip
list(a1)	#[1, 2, 3]
list(a2)	#[4, 5, 6]

#同时遍历两个或更多的序列
questions = ['name', 'quest', 'favorite color']
answers = ['lancelot', 'the holy grail', 'blue']
for q, a in zip(questions, answers):
     print('What is your {0}?  It is {1}.'.format(q, a))
```

super

```python
#代表子类的所有父类
class SongBird(Bird):
     def __init__(self):
          super(SongBird,self).__init__()
          self.sound = 'Squawk'
     def sing(self):
          print self.song()
```

@classmethod

```python
class A(object):
    bar = 1
    def func1(self):  
        print ('foo') 
    @classmethod
    def func2(cls):
        print ('func2')
        print (cls.bar)
        cls().func1()   # 调用 foo 方法
 
A.func2()               # 不需要实例化
```

@staticmethod

```python
#staticmethod也可以修饰类中类
class C(object):
    @staticmethod
    def f():
        print('runoob');
 
C.f();          # 静态方法无需实例化
cobj = C()
cobj.f()        # 也可以实例化后调用
```



# 数据类型

不可变数据（3 个）：Number（数字）、String（字符串）、Tuple（元组）

可变数据（3 个）：List（列表）、Dictionary（字典）、Set（集合）

有序：string、 tuple 、list 

元素类型不一致：Tuple（元组）、List（列表）、Dictionary（字典）、Set（集合）



#### 数字 number

```python
#int (整数),float (浮点数), complex (复数)如 1 + 2j,bool (布尔)，空值 (None)
2 / 4   【 除法，得到一个浮点数】
2 // 4 	【除法，得到一个整数】
17 % 3 	【取余】
2 ** 5  【乘方】

#数学函数，有一部分内置，一部分需要 import math
import math
abs(x)
math.ceil(4.1)
math.floor(4.9)
round(80.264, 2)【四舍五入】
pow(100, 2)【100的平方】【写分数就是开方】
math.log(100,10) 【默认以e为底】
math.modf(2.4) 【返回整数与小数部分】
divmod(7, 2) 【得到商和余数】
max(80, 100)
min(80, 100)
sum([0,1,2])

math.pi
math.e

#随机数函数
import random
random.random()				【0，1之间】
random.randint(1,10)		【范围内随机整数】
random.uniform(1.1,5.4)		【范围内随机实数】
random.choice(range(10))	【序列随机元素】
random.randrange(1,100,2)	【1到100 步长2 随机数】
random.shuffle([1,2,3,4])	【随机排序】

#三角函数

```



#### 字符串 string 

- python中单引号和双引号使用完全相同。

- 使用三引号('''或""")可以指定一个多行字符串。

  

- 字符串转义，转义符 '\\'， \n表示换行，\t表示制表符，\\\表示的字符就是\


```python
# 字符串中有变量
print(f'Hello {name}')

# 字符串定义
var0 = "Runoob"

# 字符串拼接
var0 = "Runoob"[1:5]+"aa"

# 字符串截取
aa = "Runoob"[1:5]

# 字符串替换
str.replace("is", "was",5)

# 字符串是否包含
str.find('f')

# 字符串计数
"1-2-3-4".count("-")

# 字符串分割
"1-2-3-4".split("-")

# 字符串分隔符拼接
"-".join(["1","2","3","4"])

# 字符串判断
str.islower()
str.isupper()
str.isspace()
str.isalpha()	
str.isnumeric()
str.isalnum()	#字母加数字
str.istitle()
str.endswith("d")
str.startswith("d")

# 字符串转换
str.lower()
str.upper()
str.swapcase() #大变小，小变大
str.capitalize() #首字母大写
str.title()

# 转为list
from ast import literal_eval
list1 = literal_eval("['aa','bb']") 

# 翻译
trantab = str.maketrans(intab, outtab)   # 制作翻译表
'aaa'.translate(trantab)	#翻译

# 删除字符串中数字
from string import digits
str.translate(str.maketrans('', '', digits))

# 去除两边的8
print("!!!8888888".strip('8'))	
print("!!!8888888".lstrip('8'))	
```



#### 元组  tuple

```python
#创建
tup1 = ()
tup2 = (20,)

#查
tup2[1:5] 

#改	只能拼接
tup3 = tup1 + tup2

#删	不能删元素
del tup
```



#### 列表  List

序列都可以进行的操作包括索引，切片，加，乘，检查成员

列表的数据项不需要具有相同的类型。

有序，重复

```python
#定义
list1 = []
list1 = ['Google', 'Runoob', 1997, 2000]
#查
list2[3]
list2[1:5]				
list.index(obj)			【从列表中找出某个值第一个匹配项的索引位置】
#改
list1[2] = 2001
a[2:5:-1] = []    		【最后-1表示逆向读取】
list3 = list1 + list2
#删
del list[2]
list.remove(obj)
list.pop(index = -1)	【移除列表中的元素（默认最后一个），并且返回该元素的值】
list.clear()清空列表
del list
#增
list1[4] = 2001
list.append(obj)
list.insert(index, obj)
list.extend(seq)		【用seq的元素添加到list，扩展列表】

#函数
list.count(obj)			【统计某个元素在列表中出现的次数】
list.reverse()
list.copy()
list.popleft()			【移除列表中第一个元素，返回该值】
list.sort(reverse=False)
  
#列表推导式		元组，列表，集合，字典，字符串都可用
[3*x for x in vec]
[3*x for x in vec if x > 3]
[x*y for x in vec1 for y in vec2]

#矩阵行列互换
[[row[i] for row in matrix] for i in range(len(matrix))]
```



#### 集合 set

无序，唯一

```python
#定义
a = set()
s = {'apple', 'orange', 'apple', 'pear', 'orange', 'banana'}

#查
'Rose' in s

#增
s.add("Facebook")
s.update({'apple', 'orange', 'apple', 'pear', 'orange', 'banana'})		

#删
s.remove("Facebook")
s.discard("Facebook")
s.pop() 	
s.clear()

#函数
set.copy()
x.issubset(y) 			【x是否是y的子集】		
x.isdisjoint(y)			【是否有交集】

#集合运算
print(a - b)			【差集】 
print(a & b)			【交集】
print(a | b)			【并集】
print(a ^ b) 			【交集的补集】

#集合推导式
{x for x in 'abracadabra' if x not in 'abc'}
```



#### 字典 Dictionary

```python
#定义     		键：唯一、不可变类型
d = {key1 : value1, key2 : value2}
dict([('Runoob', 1), ('Google', 2), ('Taobao', 3)])

{x: x**2 for x in (2, 4, 6)}
dict.fromkeys(seq，10)		【健为seq，值为10】

#查
dict['Name']
key in dict

#增改
dict.update(dict2)
dict['Age'] = 8

#删
del dict['Name'] 
site.pop('name')          		
dict1.clear()    
del dict        

#函数
dict1.copy()
list(dict.keys())
list(dict.values())
dict.setdefault('Sex', None)	【有值就是get（key），没有值就添加None】

#遍历
for k, v in dict.items():
    print(k, v)
```



# 运算符

```python
算术运算符
+、-、*、/、%、**、//（得到结果向下取整）


比较运算符
==、!=、>、<、>=、<=


赋值运算符
=、+=、-=、*=、/=、%=、**=、//=


位运算符
&（按位与）、|、^、~（按位取反，1变0，0变1）、<<、>>


逻辑运算符
and（逻辑与）、or、not


成员运算符
in、not in


身份运算符
is、is not


优先级
**、~ 、* / % //、+ -、>> <<、&、^ |、<= < > >=、== !=、= %= /= //= -= += *= **=、is is not、in not in、not and or
```



```python
Python 中 （&，|）和（and，or）之间的区别

如果a，b是数值变量
1 & 2       # 输出为0
1 | 2       # 输出为3  #二进制位运算
2 and 1     # 返回1    #and中含0，返回0； 均为非0时，返回后一个值
2 or 0   	# 返回2	 #or中， 至少有一个非0时，返回第一个非0

a, b是bool变量， 则两类的用法基本一致

DataFrame的切片过程，需要求得满足多个逻辑条件的数据时，要使用& 和|
```



# 语句

### 顺序



### 条件控制

```python
if condition_1:
    statement_block_1
elif condition_2:
    statement_block_2
else:
    statement_block_3
```



### 循环

```python
while <expr>:
    <statement(s)>
else:
    <additional_statement(s)>
    
    
for <variable> in <sequence>:
    <statements>
else:
    <statements>
 

break、continue、 pass
```



# 变量

public 可以被直接引用，比如：`abc`，`x123`，`PI`等；

特殊变量，\__xxx__，自己的变量一般不要用这种变量名

private  ` _xxx`和`__xxx`这样的函数或变量就是非公开的（），不应该被直接引用，比如`_abc`，`__abc`等；



# 函数

```python
#定义
def area(width, height):
    return width * height

#调用
area(3,5)

#参数传递
不可变类型（整数、字符串、元组）		类似值传递，修改不改变原对象
可变类型（列表，字典，集合）				类似引用传递，修改改变原对象

#参数类型
#定义的时候可以给参数赋初值，调用的时候可以用名字或者按顺序调用。
#默认参数一定要用不可变类型
必需参数（定义）def printme( str ):
默认参数（定义）def printinfo( name, age = 35 ):
关键字参数（调用）printinfo( age=50, name="runoob" )

不定长参数（定义）def printinfo( arg1, *vartuple ):
		  （调用）printinfo( 70, 60, 50 )			【调用时 所有剩下的变量 都变成一个元组vartuple】
        
        （定义）def printinfo( arg1, **vardict ):
            		print(vardict['b'])
   					print(vardict['a'])
   		  （调用）printinfo(1, a=2,b=3)				【调用时 所有剩下的变量 都变成一个字典vardict】

#匿名函数（使用 lambda 来创建）
sum = lambda arg1, arg2: arg1 + arg2                
```

```python
# 函数本身也可以赋值给变量，即：变量可以指向函数。
f = abs
f(-10)
# 函数名其实就是指向函数的变量
```



# 高级特性

使用高级特性可以简化代码。



### 切片

```
L[-2:]
```



### 生成器

generator：生成器，在调用的时候才生成下一个元素，与直接的列表相比，节省了空间

```python
# 定义
g = (x * x for x in range(10))

#获取下一个元素
next(g)

#迭代出所有元素
for n in g:
    print(n) 
```

如果一个函数定义中包含`yield`关键字，那么这个函数就不再是一个普通函数，而是一个generator函数，调用一个generator函数将返回一个generator对象

```python
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b
        a, b = b, a + b
        n = n + 1
    return 'done'

for n in fib(6):
    print(n)
    
# 获取generator函数的返回值
while True:
    try:
        x = next(g)
        print('g:', x)
    except StopIteration as e:
        print('Generator return value:', e.value)
        break
```

generator函数和普通函数的执行流程不一样。普通函数是顺序执行，遇到`return`语句或者最后一行函数语句就返回。而变成generator的函数，在每次调用`next()`的时候执行，遇到`yield`语句返回，再次执行时从上次返回的`yield`语句处继续执行



### 迭代器

Iterable：迭代器，可以调用next的就是迭代器，不用区分生成器和迭代器，会用就行。

```python
#序列类型 可用于创建迭代器
list=[1,2,3,4]
it = iter(list)    	【list变迭代器】
print (next(it,5))	【每次输出迭代器的一个元素，完成后一直输出5】

    
#创建一个迭代器类
class MyNumbers:
  def __iter__(self):
    self.a = 1
    return self
 
  def __next__(self):
    if self.a <= 20:
      x = self.a
      self.a += 1
      return x
    else:
      raise StopIteration
 
myclass = MyNumbers()
myiter = iter(myclass)
```



# 高级函数

一个函数就可以接收另一个函数作为参数，这种函数就称之为高阶函数。

```python
def add(x, y, f):
    return f(x) + f(y)
```



### map

对每个元素单独使用 f

```python
r = map(f, [1, 2, 3, 4, 5, 6, 7, 8, 9])
```



### reduce

两个元素的结果继续和第三个元素计算，以此类推

```python
reduce(lambda x, y: x * 10 + y, [1, 3, 5, 8, 9])
```



### filter

filter 的传入函数的返回值是Boolean

```python
list(filter(is_odd, [1, 2, 4, 5, 6, 9, 10, 15]))
```



### 返回函数

函数作为返回值

```python
def lazy_sum(*args):
    def sum():
        ax = 0
        for n in args:
            ax = ax + n
        return ax
    return sum

# 每次调用lazy_sum，返回值都会是一个新的sum函数
f1 = lazy_sum(1, 3, 5, 7, 9)
f2 = lazy_sum(1, 3, 5, 7, 9)


f = lazy_sum(1, 3, 5, 7, 9)
>>> <function lazy_sum.<locals>.sum at 0x101c6ed90>

f()
>>> 25
```

内部函数`sum`可以引用外部函数`lazy_sum`的参数和局部变量，当`lazy_sum`返回函数`sum`时，相关参数和变量都保存在返回的函数中，这叫做**闭包**。



### 闭包

使用闭包，就是内层函数引用了外层函数的局部变量。

调用内层函数时，变量才会传入。所以返回闭包时牢记一点：返回函数不要引用任何循环变量，或者后续会发生变化的变量。

内层函数改变外层函数局部变量值时，需要使用nonlocal声明该变量不是当前函数的局部变量。

```python
def inc():
    x = 0
    def fn():
        nonlocal x
        x = x + 1
        return x
    return fn
```



### 装饰器

Decorator，在代码运行期间动态增加功能的方式，是一个返回函数的高阶函数

```python
# 定义
def log(func):
    def wrapper(*args, **kw):
        print('call %s():' % func.__name__)
        return func(*args, **kw)
    return wrapper

# 使用
@log
def now():
    print('2015-3-25')
#相当于
now = log(now) 
```



# 模块

一个.py文件就称之为一个模块（Module）。

按目录来组织模块的方法，称为包（Package）。

每一个包目录下面都会有一个`__init__.py`的文件，这个文件是必须存在的，\__init__.py`本身就是一个模块，而它的模块名就是包名。

```ascii
mycompany
 ├─ web
 │  ├─ __init__.py
 │  ├─ utils.py
 │  └─ www.py
 ├─ __init__.py
 ├─ abc.py
 └─ utils.py
```

web既是一个包，也是一个模块。



```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

' a test module '

__author__ = 'Michael Liao'



import sys

def test():
    args = sys.argv
    if len(args)==1:
        print('Hello, world!')
    elif len(args)==2:
        print('Hello, %s!' % args[1])
    else:
        print('Too many arguments!')

if __name__=='__main__':
    test()
```

第1行和第2行是标准注释，第4行是一个字符串，表示模块的文档注释，第6行使用`__author__`变量把作者写进去



# 类和实例

python中 类是多重继承的



### 类和实例

```python
class Student(object):

    def __init__(self, name, score):
        self.name = name
        self.score = score

bart = Student('Bart Simpson', 59)

# 列出对象所有属性
dir('ABC')

# 给实例动态增加属性
s = Student()
s.name = 'Michael'

# 给实例动态增加方法
from types import MethodType
s.set_age = MethodType(set_age, s)
s.set_age(25)

# 给类动态增加方法
Student.set_score = set_score

# 限制只能增加2个属性，子类也要定义__slots__，定义了就继承父类的__slots__，否则不会继承父类的__slots__
class Student(object):
    __slots__ = ('name', 'age')
```



### get set 属性

```python
class Student(object):

    # 装饰器把方法变为可读属性
    @property
    def score(self):
        return self._score

    # # 装饰器把方法变为可写属性，可以限制属性的范围
    @score.setter
    def score(self, value):
        if not isinstance(value, int):
            raise ValueError('score must be an integer!')
        if value < 0 or value > 100:
            raise ValueError('score must between 0 ~ 100!')
        self._score = value
        
s = Student()
s.score = 60
s.score
```



### 枚举类

```python
# 定义
from enum import Enum, unique
Month = Enum('Month', ('Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'))

@unique		# unique 验证没有重复值
class Weekday(Enum):
    Sun = 0 
    Mon = 1
    Tue = 2
    Wed = 3
    Thu = 4
    Fri = 5
    Sat = 6

# 使用
for name, member in Month.__members__.items():
    print(name, '=>', member, ',', member.value)
    
Weekday.Mon
Weekday(1)
Weekday.Tue.value
```



# 异常

### 异常引发

程序错误引发

手动抛出引发

```python
raise NameError('HiThere')  
```



### 异常处理

```python
import logging

try:
    x = int(input("Please enter a number: "))
    break
except (RuntimeError, TypeError, NameError):
	print("Oops!  That was no valid number.  Try again   ")
except ZeroDivisionError as e:
	logging.exception(e)
	raise
else:
	print(arg, 'has', len(f.readlines()), 'lines')
finally:
	print('Goodbye, world!') 
```



### 异常自定义

```python
class MyError(Exception):
        def __init__(self, value):
            self.value = value
        def __str__(self):
            return repr(self.value)
```



# 打印调试

打印使用print，调试（debug）打断点。



python 打印错误信息的时候，最后的error 是最开始发生异常的地方。



### assert

凡是用`print()`来辅助查看的地方，都可以用断言（assert）来替代：

```
assert expression

等价于：
if not expression:
    raise AssertionError
```

```shell
python -O err.py #关闭断言
```



### logging

```python
import logging
logging.basicConfig(level=logging.INFO)

n = 10
logging.info('n = %d' % n)
```



### 单元测试

单元测试是用来对一个模块、一个函数或者一个类来进行正确性检验的测试工作。

以`test`开头的方法就是测试方法，不以`test`开头的方法不被认为是测试方法，测试的时候不会被执行。

```python
import unittest

from mydict import Dict

class TestDict(unittest.TestCase):

    def test_init(self):
        d = Dict(a=1, b='test')
        self.assertEqual(d.a, 1)
        self.assertEqual(d.b, 'test')
        self.assertTrue(isinstance(d, dict))

    def test_key(self):
        d = Dict()
        d['key'] = 'value'
        self.assertEqual(d.key, 'value')

    def test_attr(self):
        d = Dict()
        d.key = 'value'
        self.assertTrue('key' in d)
        self.assertEqual(d['key'], 'value')

    def test_keyerror(self):
        d = Dict()
        with self.assertRaises(KeyError):
            value = d['empty']

    def test_attrerror(self):
        d = Dict()
        with self.assertRaises(AttributeError):
            value = d.empty
            
if __name__ == '__main__':
    unittest.main()
```

```shell
python mydict_test.py

python -m unittest mydict_test
```

```python
class TestDict(unittest.TestCase):
    
	# 在每个测试方法前执行
    def setUp(self):
        print('setUp...')
	
    # 在每个测试方法后执行
    def tearDown(self):
        print('tearDown...')
```



# IO编程

IO：磁盘网络 与 内存 交换数据的过程

同步IO：CPU 等待IO 完成，再继续执行。

异步IO：CPU 不等待IO执行的结果，继续执行其他程序。异步IO效率更高，但是编程复杂。



# 读写文件

open(filename, mode)

```python
'''
'r'       open for reading (default)
'w'       open for writing, truncating the file first
'x'       create a new file and open it for writing
'a'       open for writing, appending to the end of the file if it exists
'b'       binary mode
't'       text mode (default)
'+'       open a disk file for updating (reading and writing)
'U'       universal newline mode (deprecated)
'''
#读	r+ 
f = open("/tmp/foo.txt", "r+")
str = f.read()
str = f.readline()
print(str)
f.close()

f = open("/tmp/foo.txt", "r+")
for line in f:
    print(line, end='')
f.close()

#清空写 w	追加写a+ 
f = open("/tmp/foo.txt", "w")
f.write( "Python 是一个非常好的语言。\n是的，的确非常好!!\n" )
f.flush()
f.close()
```



# 进程和线程

一个核同时只能进行一个线程或者两个线程。线程是资源，进程是应用。

一个应用就是一个进程，一个进程至少有一个线程。一个应用至少申请一个资源。

多进程就是多个应用，可以在一个核上轮流执行（并发）。

一个进程下面多个线程，线程如果在多个核上执行，就是并行。一般多线程就是指并行，但是python多线程无法并行。



同时执行多个任务（并行）的解决办法：

- 多进程模式；启动多个进程，每个进程虽然只有一个线程，但多个进程可以一块执行多个任务。

- 多线程模式；启动一个进程，在一个进程内启动多个线程，这样，多个线程也可以一块执行多个任务。
- 多进程+多线程模式：启动多个进程，每个进程再启动多个线程，这样同时执行的任务就更多了，当然这种模型更复杂，实际很少采用。



多线程与多进程的优缺点：

多进程模式最大的优点就是稳定性高，多进程模式的缺点是创建进程的代价大，一般一个系统就几千个进程。

多线程不稳定，一个线程奔溃，整个进程结束。python的多线程也只能用一个核。

线程切换（并发），也会消耗资源，任务太多，可能资源都用来切换了。



总结：

1、计算密集型任务使用python只能多进程，进程的数量应当等于CPU的核心数。即使这样，python对于计算密集型任务效率也很低。

2、IO密集型任务使用协程，支持异步IO。

3、即利用多核，又高效执行，使用多进程+协程。



Python既支持多进程，又支持多线程，但是多线程只能利用一个核，基本没用。



### 多进程

`jupyter 不支持多进程`

```python
from multiprocessing import Process
import os

# 子进程要执行的代码
def run_proc(name):
    print('Run child process %s (%s)...' % (name, os.getpid()))

if __name__=='__main__':
    print('Parent process %s.' % os.getpid())
    
    # 子进程
    p = Process(target=run_proc, args=('test',))
    p.start()	#执行子进程
    p.join()	#等待子进程执行结束再执行其他进程，便于进程同步
```

进程池

```python
from multiprocessing import Pool
import os, time, random

def long_time_task(name):
    print('Run task %s (%s)...' % (name, os.getpid()))
    start = time.time()
    time.sleep(random.random() * 3)
    end = time.time()
    print('Task %s runs %0.2f seconds.' % (name, (end - start)))

if __name__=='__main__':
    print('Parent process %s.' % os.getpid())
    p = Pool(4)
    for i in range(5):
        p.apply_async(long_time_task, args=(i,))
    p.close() #不能继续添加子进程
    p.join() #开始并行执行所有的子进程，等待子进程执行结束再执行其他进程，有几个核并行执行几个进程，多余的进程等有核了再执行。
    print('All subprocesses done.')
```

进程间通信

```python
from multiprocessing import Process, Queue
import os, time, random

# 写数据进程执行的代码:
def write(q):
    print('Process to write: %s' % os.getpid())
    for value in ['A', 'B', 'C']:
        print('Put %s to queue...' % value)
        q.put(value)
        time.sleep(random.random())

# 读数据进程执行的代码:
def read(q):
    print('Process to read: %s' % os.getpid())
    while True:
        value = q.get(True)
        print('Get %s from queue.' % value)

if __name__=='__main__':
    # 父进程创建Queue，并传给各个子进程：
    q = Queue()
    pw = Process(target=write, args=(q,))
    pr = Process(target=read, args=(q,))
    # 启动子进程pw，写入:
    pw.start()
    # 启动子进程pr，读取:
    pr.start()
    # 等待pw结束:
    pw.join()
    # pr进程里是死循环，无法等待其结束，只能强行终止:
    pr.terminate()
```

进程运行命令

```python
import subprocess
r = subprocess.call(['ping', 'www.baidu.com'])
```



### 多线程

任何进程默认就会启动一个线程，我们把该线程称为主线程，主线程又可以启动新的线程

```python
import time, threading

# 新线程执行的代码:
def loop():
    print('thread %s is running...' % threading.current_thread().name)

print('thread %s is running...' % threading.current_thread().name)
t = threading.Thread(target=loop, name='LoopThread')
t.start()
t.join()
print('thread %s ended.' % threading.current_thread().name)
```



多线程和多进程最大的不同在于，多进程中，同一个变量，各自有一份拷贝存在于每个进程中，互不影响，而多线程中，所有变量都由所有线程共享。

线程锁

```python
balance = 0
lock = threading.Lock()
 
def run_thread(n):
    lock.acquire()	 # 先要获取锁:
    
    # try代码段是要上锁的部分程序，finally保证最后锁的释放
    try:
        # 先存后取，结果应该为0:
    	global balance
    	balance = balance + n
    	balance = balance - n
	finally:
        lock.release()

t1 = threading.Thread(target=run_thread, args=(5,))
t2 = threading.Thread(target=run_thread, args=(8,))
t1.start()
t2.start()
t1.join()
t2.join()
print(balance)
```

 死锁：由于可以存在多个锁，不同的线程持有不同的锁，并试图获取对方持有的锁时，可能会造成死锁，导致多个线程全部挂起，既不能执行，也无法结束。



Python的线程虽然是真正的线程，但解释器执行代码时，有一个GIL锁：Global Interpreter Lock，任何Python线程执行前，必须先获得GIL锁。所以，多线程在Python中只能交替执行，即使100个线程跑在100核CPU上，也只能用到1个核。



### 分布式进程

https://www.liaoxuefeng.com/wiki/1016959663602400/1017631559645600



# 异步IO

### 协程

什么是协程

```
子程序，或者称为函数，在所有语言中都是层级调用，比如A调用B，B在执行过程中又调用了C，C执行完毕返回，B执行完毕返回，最后是A执行完毕。所以子程序调用是通过栈实现的，一个线程就是执行一个子程序。子程序调用总是一个入口，一次返回，调用顺序是明确的。

协程看上去也是子程序，但执行过程中，在子程序内部可中断，然后转而执行别的子程序，在适当的时候再返回来接着执行。
协程的效果就是在一个方法中任意位置任意次数的插入其他方法的代码段。
```

协程与多线程相比优点

```
1.极高的执行效率。因为子程序切换不是线程切换，而是由程序自身控制，因此，没有线程切换的开销，和多线程比，线程数量越多，协程的性能优势就越明显。
2.不需要多线程的锁机制，因为只有一个线程，也不存在同时写变量冲突，在协程中控制共享资源不加锁，只需要判断状态就好了，所以执行效率比多线程高很多。
```



多进程+协程，既充分利用多核，又充分发挥协程的高效率，可获得极高的性能。



原生协程：Python对协程的支持是通过generator（死循环+yield）实现的。

```python
def consumer():
    r = ''
    while True:
        n = yield r     # yield 第一次被调用返回一个值，结束方法。第二次被调用接收一个值，继续向下执行。             
        if not n:
            return
        print('[CONSUMER] Consuming %s...' % n)
        r = '200 OK'

def produce(c):
    c.send(None)	#启动生成器，运行到yield返回值，再次调用时在yield处继续执行									
    n = 0			#初始化n
    while n < 5:
        n = n + 1
        print('[PRODUCER] Producing %s...' % n)
        r = c.send(n)	# 调用消费者生成器，从yield处继续执行，并传递n值
        print('[PRODUCER] Consumer return: %s' % r)
    c.close()

c = consumer()
produce(c)
```



### asyncio

异步io是一个线程执行的，可以极大的提高并发。异步io通过协程实现。

```python
import threading
import asyncio

# 定义一个异步io的方法
@asyncio.coroutine	#这个注解可以吧生成器方法变成异步io方法
def hello():
    print('Hello world! (%s)' % threading.currentThread())
    yield from asyncio.sleep(1) # 模拟一个io耗时方法，这个方法也必须是coroutine的。
    #执行到这不会等待io方法执行完，线程直接去执行其他io任务。io方法执行完，又返回值继续执行
    print('Hello again! (%s)' % threading.currentThread())

# 执行异步io方法 
loop = asyncio.get_event_loop()
tasks = [hello(), hello()]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()
```

```python
# 简化写法，async代替注解，await代替yield
async def hello():
    print("Hello world!")
    r = await asyncio.sleep(1)
    print("Hello again!")
```

使用实例

```python
import asyncio

from aiohttp import web

async def index(request):
    await asyncio.sleep(0.5)
    return web.Response(body=b'<h1>Index</h1>')

async def hello(request):
    await asyncio.sleep(0.5)
    text = '<h1>hello, %s!</h1>' % request.match_info['name']
    return web.Response(body=text.encode('utf-8'))

async def init(loop):
    app = web.Application(loop=loop)
    app.router.add_route('GET', '/', index)
    app.router.add_route('GET', '/hello/{name}', hello)
    srv = await loop.create_server(app.make_handler(), '127.0.0.1', 8000)
    print('Server started at http://127.0.0.1:8000...')
    return srv

loop = asyncio.get_event_loop()
loop.run_until_complete(init(loop))
loop.run_forever()
```



# 正则表达式

https://www.liaoxuefeng.com/wiki/1016959663602400/1017639890281664



# os模块

### 系统环境

```python
import os

# 操作系统类型
os.name 
os.uname()

#环境变量
os.environ
os.environ.get('PATH')
```

### 操作目录

```python
#创建目录
os.mkdir('/Users/michael/testdir')

#删除目录
os.rmdir('/Users/michael/testdir')

#修改当前目录
os.chdir( "/tmp" )

#查看目录
os.path.abspath('.')

#查看当前目录
os.getcwd()

#查看所有目录下所有
os.listdir('.')

#是否是目录
os.path.isdir(x)

#查看所有目录
[x for x in os.listdir('.') if os.path.isdir(x)]

#拼接目录
os.path.join('/Users/michael', 'testdir')

#拆分目录
os.path.split('/Users/michael/testdir/file.txt')
```

### 操作文件

```python
#删除文件
os.remove('test.py')

#重命名
os.rename('test.txt', 'test.py')

#是否是文件
os.path.isfile(x)

#获取扩展名
os.path.splitext('/path/to/file.txt')

#所有.py 文件
[x for x in os.listdir('.') if os.path.isfile(x) and os.path.splitext(x)[1]=='.py']
```

### 权限操作

```python
# 权限查看
ret = os.access("/tmp/foo.txt", os.F_OK)

# 权限修改
os.chmod("/tmp/foo.txt", stat.S_IWOTH)
```

```
os.F_OK path是否存在
os.R_OK是否可读
os.W_OKh是否可写
os.X_OK是否可执行

stat.S_IXOTH: 其他用户有执行权0o001
stat.S_IWOTH: 其他用户有写权限0o002
stat.S_IROTH: 其他用户有读权限0o004
stat.S_IRWXO: 其他用户有全部权限(权限掩码)0o007
stat.S_IXGRP: 组用户有执行权限0o010
stat.S_IWGRP: 组用户有写权限0o020
stat.S_IRGRP: 组用户有读权限0o040
stat.S_IRWXG: 组用户有全部权限(权限掩码)0o070
stat.S_IXUSR: 拥有者具有执行权限0o100
stat.S_IWUSR: 拥有者具有写权限0o200
stat.S_IRUSR: 拥有者具有读权限0o400
stat.S_IRWXU: 拥有者有全部权限(权限掩码)0o700
stat.S_ISVTX: 目录里文件目录只有拥有者才可删除更改0o1000
stat.S_ISGID: 执行此文件其进程有效组为文件所在组0o2000
stat.S_ISUID: 执行此文件其进程有效用户为文件所有者0o4000
stat.S_IREAD: windows下设为只读
stat.S_IWRITE: windows下取消只读
```



# pickle

变量从内存中变成可存储或传输的过程称之为序列化，在Python中叫pickling，在其他语言中也被称之为serialization，marshalling，flattening等等。

序列化之后，就可以把序列化后的内容写入磁盘，或者通过网络传输到别的机器上。

```python
import pickle

#序列化保存
output = open('data.pkl', 'wb')
pickle.dump(selfref_list, output)
output.close()

#序列化加载
pkl_file = open('data.pkl', 'rb')
data1 = pickle.load(pkl_file)
pkl_file.close()
```

Pickle的问题，就是它只能用于Python，并且可能不同版本的Python彼此都不兼容，因此，只能用Pickle保存那些不重要的数据。



# json

| JSON类型   | Python类型 |
| :--------- | :--------- |
| {}         | dict       |
| []         | list       |
| "string"   | str        |
| 1234.56    | int或float |
| true/false | True/False |
| null       | None       |

```python
import json

# 变成json
d = dict(name='Bob', age=20, score=88)
json.dumps(d)

# 解析json
json_str = '{"age": 20, "score": 88, "name": "Bob"}'
json.loads(json_str)
```

对象变成json

```python
import json

class Student(object):
    def __init__(self, name, age, score):
        self.name = name
        self.age = age
        self.score = score
 
s = Student('Bob', 20, 88)
json.dumps(s, default=lambda obj: obj.__dict__)
```

json变成对象

```python
import json

def dict2student(d):
    return Student(d['name'], d['age'], d['score'])

json_str = '{"age": 20, "score": 88, "name": "Bob"}'
json.loads(json_str, object_hook=dict2student)
```



# datetime

```python
from datetime import datetime

now = datetime.now()

today = datetime.today()

dt = datetime(2015, 4, 19, 12, 20)
```

进行运算

```python
from datetime import datetime, timedelta

now - timedelta(days=1)
now + timedelta(days=2, hours=12)
```

与字符串转换

```python
# str转换为datetime
cday = datetime.strptime('2015-6-1 18:19:59', '%Y-%m-%d %H:%M:%S')

# datetime转换为str
now.strftime('%a, %b %d %H:%M')
```

与 timestamp 转换

```python
'''
全球各地的计算机在任意时刻的timestamp都是完全相同的，与时区无关。时间戳是秒数或者毫秒，python是秒
'''

# 把datetime转换为timestamp
dt.timestamp()

# 把timestamp转换为datetime
datetime.fromtimestamp(t)	#转换到系统时区datetime
datetime.utcfromtimestamp(t) #转化到0时区datetime
```

时区转换

```python
from datetime import datetime, timedelta, timezone

# 先把当前时间转换到标准时区时间
utc_dt = datetime.utcnow().replace(tzinfo=timezone.utc)

# 再转换成其他时区时间
bj_dt = utc_dt.astimezone(timezone(timedelta(hours=8)))
tokyo_dt = utc_dt.astimezone(timezone(timedelta(hours=9)))
```



# 编码解码

```python
print('qqq'.encode('utf-8'))
print(b'\xe4\xb8\xad\xe6\x96\x87'.decode('utf-8'))
```

https://www.liaoxuefeng.com/wiki/1016959663602400/1183255880134144

https://www.liaoxuefeng.com/wiki/1016959663602400/1017684507717184



# requests

```python
import requests

#发送get请求
r = requests.get('https://www.douban.com/')
r = requests.get('https://www.douban.com/search', params={'q': 'python', 'cat': '1001'})
r = requests.get('https://www.douban.com/', headers={'User-Agent': 'Mozilla/5.0 AppleWebKit'})
r = requests.get(url, cookies={'token': '12345', 'status': 'working'})
r = requests.get(url, timeout=2.5)

#发送post请求
r = requests.post('https://accounts.douban.com/login', data={'form_email': 'abc@example.com', 'form_password': '123456'})
r = requests.post(url, json={'key': 'value'}) # 内部自动序列化为JSON

#上传文件
r = requests.post(url, files={'file': open('report.xls', 'rb')}) 

#获取状态码
r.status_code

#获取编码
r.encoding

#获取响应头
r.headers

#获取返回内容
r.text

#获取json
r.json()

#获取字节对象
r.content

#获取cookies
r.cookies
```



# Pillow

图像处理模块

https://www.liaoxuefeng.com/wiki/1016959663602400/1017785454949568



# psutil

运维相关模块

https://www.liaoxuefeng.com/wiki/1016959663602400/1183565811281984



# 视图与副本

### 视图	

单纯返回一个指针，指向同一个物理内存。是之前变量的别称引用

```python
#ndarray的切片操作
import numpy as np

aa = np.array([1,2,3,6,7,8])
aa[0:2] = 0
print(aa)

#各数据结构的变量指向
aa = [1,2,3,4,5,6]
bb = aa
bb[0] = 0
print(aa)
print(bb)
```



### 副本

副本  是新的物理内存 ，分为 深浅拷贝

浅拷贝父对象是复制的新对象， 但是子对象都是公用的

```python
#ndarray/list 的copy()
import numpy as np

aa = np.array([1,2,[3,4,5],6,7,8])
bb = aa.copy()
aa[2][0] = 0
print(aa)
print(bb)

#copy 的浅拷贝
import copy
list1 = [1,2,3,[4,5,6],8]
list2 = copy.copy(list1)		
list1[3][0] = 0
print(aa)
print(bb)
```

深拷贝完全复制一个新对象

```python
import copy
list1 = [1,2,3,[4,5,6],8]
list2 = copy.deepcopy(list1)		
list1[3][0] = 0
print(aa)
print(bb)
```



# 性能度量

```python
#语句 性能比较
from timeit import Timer

#可以分别计算出两个语句需要的时间
Timer('t=a; a=b; b=t', 'a=1; b=2').timeit()
Timer('a,b = b,a', 'a=1; b=2').timeit()

#函数 性能比较
import profile

def one():                
	sum=0
	for i in range(10000):
		sum+=i
	return sum

def two():
	sum = 0
	for i in range(100000):
		sum += i
	return sum

profile.run("one()", "result")  【结果保存在文件中】
profile.run("two()")			【ncalls：次数，tottime：时间，percall：平均时间】
```



# 文件切割

```python
import os
import pandas as pd

# filename为文件路径，file_num为拆分后的文件行数
# 根据是否有表头执行不同程序，默认有表头的
def Data_split(filename, file_num, header=True):
    if header:
        # 设置每个文件需要有的行数,初始化为1000W
        chunksize = 10000
        data1 = pd.read_table(filename, chunksize=chunksize, sep=',', encoding='gbk')
        # print(data1)
        # num表示总行数
        num = 0
        for chunk in data1:
            num += len(chunk)
        # print(num)
        # chunksize表示每个文件需要分配到的行数
        chunksize = round(num / file_num + 1)
        # print(chunksize)
        # 分离文件名与扩展名os.path.split(filename)
        head, tail = os.path.split(filename)
        data2 = pd.read_table(filename, chunksize=chunksize, sep=',')
        i = 0
        for chunk in data2:
            chunk.to_csv('{0}_{1}{2}'.format(head, i, tail), header=None, index=False)
            print('保存第{0}个数据'.format(i))
            i += 1
    else:
        # 获得每个文件需要的行数
        chunksize = 10000
        data1 = pd.read_table(filename, chunksize=chunksize, header=None, sep=',')
        num = 0
        for chunk in data1:
            num += len(chunk)
            chunksize = round(num / file_num + 1)

            head, tail = os.path.split(filename)
            data2 = pd.read_table(filename, chunksize=chunksize, header=None, sep=',')
            i = 0
            for chunk in data2:
                chunk.to_csv('{0}_{1}{2}'.foemat(head, i, tail), header=None, index=False)
                print('保存第{0}个数据'.format(i))
                i += 1
                
filename = '文件路径'
# num为拆分为的文件个数
Data_split('E:\\data\\ctr_data\\train.csv',20, header=True)
```

