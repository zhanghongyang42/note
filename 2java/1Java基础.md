java面向对象

封装：类，只暴露公共的访问方式

继承：同一个方法不同类不用重复写，单继承

多态：方便写接口，提高了代码的拓展性



# 基础

### 注释

单行注释

格式： //注释文字



多行注释

格式： /* 注释文字 */



文档注释

格式：/** 注释文字 */



### 引号

''单引号表示字符

""双引号表示字符串



### 关键字

![image-20201208182000796](picture/image-20201208182000796.png)

数据类型：

byte 字节型、short 短整型、int 整型、long 长整型、float 浮点、double 双精度

char 字符型、boolean 布尔型、void 不声明类型

class 类、interface 接口、**enum 枚举

程序控制语句：

return 返回、default 默认

if 如果、else 反之、switch 开关、case 返回开关里的结果

while 循环、do 运行、for 循环、break 跳出循环、continue 继续

包相关：
import 引入、package 包

类间关系：

extends 扩允,继承、implements 实现

![image-20201208182012698](picture/image-20201208182012698.png)

变量引用：
super 父类,超类、this 本类、new 创建、**instanceof 实例？**

访问控制修饰符：
private 私有的、protected 受保护的、public 公共的、默认（什么都不写）

类、方法和变量修饰符：

final 终极,不可改变的、static 静态 、abstract 声明抽象、**synchronized 线程,同步**

异常处理：
try 捕获异常、catch 处理异常、finally 有没有异常都执行、throw 抛出一个异常对象、throws 声明一个异常可能被抛出、**assert 断言**

其他：

native 本地strictfp 严格,精准、transient 短暂、volatile 易失



##### 变量引用

**this**

使用场景： **局部变量和成员变量重名**时，想在局部变量作用域  **调用成员变量**，使用**this**

```java
public class Student {
	private int age;

	public void setAge(int age) {
		this.age = age;
	}
}
```



**super**

```java
public class Son extends Father {
	public int age = 20;
	
	public void printAge() {
		int age = 10;
		System.out.println(age);
		System.out.println(this.age);
		System.out.println(super.age);
	}
}
```



##### 权限修饰符

**权限修饰符修饰类**

public、default 决定类在包的可见范围



public：修饰的类可以在包外访问。

default：修饰的类只能在包内访问到。



**权限修饰符修饰成员**

public、protected、*default*  、private决定成员（成员变量，成员方法，内部类）在类的可见范围



private 本类
默认	本包                 
protect 本包 和 不同包子父类        子类能访问到父类
public  所有



##### 非访问控制修饰符

**final**

•    可以修饰类，成员变量，成员方法。

–    修饰类，类不能被继承

–    修饰变量，变量就变成了常量，命名大写

–    修饰方法，方法不能被重写

```java
public class Father {
	public final void method() {
		System.out.println("method father");
	}
}

public class Son extends Father {
	public final int age = 20;
	
	public void show() {
		System.out.println(age);
	}
	
	/*
	@Override
	public void method() {
		System.out.println("method son");
	}
	*/
}
```



**static**

用来修饰成员变量和成员方法

定义阶段：

–    静态成员方法中不能出现this,super这样的关键字。

–    静态方法只能访问所在类静态成员，非静态方法可以访问所在类一切成员

调用阶段：

–    随着类的加载而加载，优先于对象存在

–    被类的所有对象共享，可以通过类名调用

```java
package com.itheima_02;

//静态成员方法中不能出现this,super这样的关键字。
 
public class Student {
	private String name = "林青霞";
	private static int age = 30;
    
    public void show2() {}
    public static void show4() {}
    
    //非静态方法可以访问所在类一切
	public void show() {
		this.name = "刘德华";
		System.out.println(name);
		System.out.println(age);
		show2();
		show4();
	}
	//静态方法只能访问所在类静态成员
	public static void show3() {
		System.out.println(age);
		show4();
	}
}
```

```java
package com.itheima_01;

public class Student {
	public String name;
	public static String graduateFrom; 
	
	public void show() {
		System.out.println(name+"-"+graduateFrom);
	}
}

//调用演示
public class StaticDemo {
	public static void main(String[] args) {
		Student.graduateFrom = "传智学院";

		Student s1 = new Student();
		s1.name = "林青霞";
		s1.show();
		
		Student s2 = new Student();
		s2.name = "刘德华";
		s2.show();
	}
}
```



### 常量

–    字符常量  用单引号括起来的内容(‘a’,’A’,’0’)

–    字符串常量 用双引号括起来的内容(“HelloWorld”)

–    整数常量  所有整数(12,-23)

–    小数常量  所有小数(12.34)

–    布尔常量  较为特有，只有true和false

–    空常量    null



new class()



### 变量

##### 变量定义

数据类型 变量名 = 初始化值;



##### 数据类型

![image-20201208182416881](picture/image-20201208182416881.png)



##### 类型转换

基本数据类型在运算的时候会自动转换成取值范围最大的类型

| 情况   | 基本数据类型                                                 | 引用数据类型 |
| ------ | ------------------------------------------------------------ | ------------ |
| 小变大 | 隐式转换：数据类型 变量名 = 初始化值;<br />float x = 5       | 向上转型     |
| 大变小 | 强制类型转换：目标类型 变量名=(目标类型)(被转换的变量);<br />int x = int(5.8) | 向下转型     |



##### 命名规范

名称只能由字母、数字、下划线_、$符号组成.

不能以数字开头.

名称不能使用Java中的关键字.



包						所有字母小写，可以域名倒过来

类或者接口		首字母大写，驼峰命名

方法或者变量	首字母小写，驼峰命名

常量					全部大写，多个词加下划线



##### 作用域

–    变量在哪对大括号内，变量就属于哪对大括号



### 运算符

##### 表达式

用运算符把常量或者变量连接起来符号java语法的式子就可以称为表达式。



##### 运算符

算术运算符

+,-,*,/,%,++,-- 

赋值运算符

=，+=,-=,*=,/=,%=

关系运算符

==,!=,>,>=,<,<=

逻辑运算符

&&,||，！



### 流程控制

##### 顺序结构

从上往下，依次执行



##### 选择结构

```java
if(x >= 3) {
	y = 2*x+1;
}
else if(x>=-1 && x<3) {
	y = 2*x;
}
else if(x<-1) {
	y = 2*x-1;
}
else {
	y = 0;
	System.out.println("不存在这样的x的值");
}
```

switch不用



##### 循环结构

```java
for(int x=0; x<5; x++) {
	System.out.println(arr[x]);
}


while(x<=5) {
	System.out.println("HelloWorld");
	x++;
}


do {
	System.out.println("HelloWorld");
	x++;
}while(x<=5);

```



# 方法

### 定义/调用

```java
/* 
 * 定义格式：
 * 		修饰符 返回值类型 方法名(参数类型 参数名1,参数类型 参数名2,...) {
 * 			方法体;
 * 			return 返回值;
 * 		}
 */
public class MethodDemo2 {
	public static int sum(int a, int b) {
		return a + b;
	}

	public static void main(String[] args) {
		int s = sum(10, 20);
	}
}
```



### 参数传递

基本数据类型的参数传递的是值，引用数据类型的参数传递的是地址



### 重载

一个类中，名称相同，参数不同的两个方法

```java
public class MethodDemo {
	public static void main(String[] args) {
	}
	
	
	public static float sum(float a,float b) {
		return a + b;
	}

	public static int sum(int a,int b,int c) {
		return a + b + c;
	}
	
	public static int sum(int a,int b) {
		return a + b;
	}
}
```



# 类

一个packge或者说一个java文件 内可以有多个类，但只能有一个public类，public类与文件名相同。



###  成员变量/局部变量

 A:在类中的位置不同

成员变量：在类中，方法外

局部变量：在方法中或者方法声明上(形式参数)



C:生命周期不同

成员变量：随着对象的创建而存在，随着对象的消失而消失

局部变量：随着方法的调用而存在，随着方法的调用完毕而消失



D:初始化值不同

成员变量：有默认值（引用 是 null ）

局部变量：没有默认值，必须先定义，赋值，最后使用



### 成员方法

见方法



### 构造方法

 方法名和类名相同

 没有返回值类型和返回值

```java
public class Student {	
	public Student() {
		System.out.println("这是构造方法");
	}
}
```

编译器 赠送  没有构造方法 的 类一个空的构造方法

自己写构造方法就不送了，构造方法  可重载。



强制：每个类必须定义一个无参构造。



# 继承

继承发生在类与类之间 ，java只支持单继承、多层继承

```java
public class Student extends Person {
	public void study() {
		System.out.println("学生要好好学习");
	}
}
```



### 变量继承

局部变量、成员变量、父类成员变量 同名时 调用原则：	**就近原则**

调  **成员变量** 用 **this**

调 **父类变量** 用 **super**

```java
public class Son extends Father {
	public int age = 20;
	
	public void printAge() {
		int age = 10;
		System.out.println(age);
		System.out.println(this.age);
		System.out.println(super.age);
	}
}
```



强制：父类、子类 变量名 不同 



### 构造方法继承

子类中所有的构造方法默认都会访问父类中的无参构造。

也可在子类构造方法中指定 执行 父类 的带参构造。

```java
public class Son extends Father {
	public Son() {
		super("林青霞");
		System.out.println("Son无参构造方法");
	}
}
```



### 方法继承

方法继承的三种情况：

普通：父类子类方法不同名

重载：父类子类方法同名不同参

重写：父类子类方法同名同参，重写后的父类方法可以用super调用



**重写**

与父类中的私有方法相同不是重写。

注解 @Override ，检查重写是否有问题

强制：子类重写父类方法时，访问权限不能更低（建议一模一样），因为多态

强制：子类重写父类方法时，不能抛出比父类方法中更多范围更大的异常。

```java
public class Father {
	public void show() {
		System.out.println("Father show");
	}
}

public class Son extends Father {
	public void method() {
		System.out.println("Son method");
	}
	
	public void show() {
		System.out.println("Son show");
	}
}

public class ExtendsTest {
	public static void main(String[] args) {
		Son s = new Son();
		s.method();
		s.show();
	}
}
```



### 类的初始化过程

```
父类静态代码块初始化
父类静态方法
子类静态代码块初始化
父类代码块初始化
父类无参构造函数初始化完成
子类show()方法：i=1  //因为创建的是son实例，所以父类里的show方法被初始化时，实际调用的是子类show方法
子类代码块初始化
子类构造函数初始化完成
子类成员变量初始化完成：s=子类私有成员变量
子类show()方法：i=1
```



# 多态

多态：父类引用指向子类对象，类似向上转型



成员变量使用父类的，成员方法使用子类的。

 A:成员变量：	编译看左边，执行看左边。

 B:成员方法：	编译看左边，执行看右边。

```java
public class Animal {
	public int age = 40;
	
	public void eat() {
		System.out.println("吃东西");
	}
}

public class Cat extends Animal {
	public int age = 20;
	public int weight = 10;
	
	public void eat() {
		System.out.println("猫吃鱼");
	}
	
	public void playGame() {
		System.out.println("猫捉迷藏");
	}
}


public class DuoTaiDemo {
	public static void main(String[] args) {
		//多态
		Animal a = new Cat();
		System.out.println(a.age);
		a.eat();
	}
}
```



**类型转换**

把 new Cat() 理解成一个值，则引用类型的类型转换同基本类型的类型转换

```java
public class DuoTaiDemo {
	public static void main(String[] args) {
        //多态，向上转型
		Animal a = new Cat(); 
		a.eat();
		//向下转型，强制类型转换
		Cat c = (Cat)a;
		c.eat();
		c.playGame();
	}
}
```



# 抽象类

抽象类有构造方法，会在子类执行构造方法的时候访问到

abstract  不与final,private  static 共存

```java
//抽象类不一定有抽象方法，有抽象方法的类一定是抽象类
public abstract class Animal {
	//抽象方法，一个没有方法体的方法应该定义为抽象方法
	public abstract void eat();
	
	public void sleep() {
		System.out.println("睡觉");
	}
}

//抽象类的子类	要么重写所有抽象方法，要么还是抽象类
public class Cat extends Animal {
	@Override
	public void eat() {
		System.out.println("猫吃鱼");
	}
}

public abstract class Dog extends Animal {
}

//抽象类的实例化
public class AnimalDemo {
	public static void main(String[] args) {
		Animal a = new Cat();
		a.eat();
		a.sleep();
	}
}
```



# 接口

接口的主要作用是分离做什么和怎么做

```java
public interface Inter {
    //接口没有构造方法。
    
    //接口中变量只能是常量
	public static final int num = 30;//默认修饰符：public static final
	
    //接口中方法都是抽象方法
	public abstract void method();//默认修饰符：public abstract
}

//接口的实现类,要么重写接口中的所有的抽象方法,要么是一个抽象类
public class InterImpl extends Object implements Inter {
	public InterImpl() {
		super();
	}
    
	@Override
	public void method() {	
	}
}

public abstract class Dog implements Inter {
}

public class InterfaceDemo {
	public static void main(String[] args) {
		Inter i = new InterImpl();
        
		System.out.println(Inter.num);
        i.method();
	}
}
```



# 类与接口

### 类和接口的关系

```java
//接口与接口：可以单继承，也可以多继承。
public interface Mother {
}

public interface Father {
}

public interface Sister extends Father,Mother {
}

//类与类：单继承，多层继承
//类与接口：可以单实现，也可以多实现
public class Son extends Object implements Father,Mother {
}
```



### 抽象类和接口使用场景

- 抽象类是对 类本质的抽象，比如抽象类 *人*，子类*老师*。

  接口是对类动作的抽象，比如接口 *吃*，子类 *老师*。

- 接口更适合实现多态

- 实现公用的方法用抽象类，个性的方法用抽象方法。抽象类功能要远超过接口，但是接口方便设计



# 内部类

内部类：被定义在另一个类的内部的类。

内部类可以修改外部类的私有成员。



### 普通内部类

```java
// 定义
class Outer {
    private String name;

    Outer(String name) {
        this.name = name;
    }
	
    // 定义一个内部类
    class Inner {
        void hello() {
            System.out.println("Hello, " + Outer.this.name);
        }
    }
}

// 调用，内部类必须依靠外部类实例，实例化
public class Main {
    public static void main(String[] args) {
        Outer outer = new Outer("Nested"); // 实例化一个Outer
        Outer.Inner inner = outer.new Inner(); // 实例化一个Inner
        inner.hello();
    }
}
```



### 匿名内部类

```java
// 定义
class Outer {
    private String name;

    Outer(String name) {
        this.name = name;
    }

    void asyncHello() {
        //匿名内部类，类创建的同时，也直接创建了一个实例化对象。
        //new Runnable() {} ,new的可以是接口或者其他普通父类，大括号里面是类的内容，new是实例化。
        Runnable r = new Runnable() {
            @Override
            public void run() {
                System.out.println("Hello, " + Outer.this.name);
            }
        };
        new Thread(r).start();
    }
}

public class Main {
    public static void main(String[] args) {
        Outer outer = new Outer("Nested");
        outer.asyncHello();
    }
}
```



### 静态内部类

静态类不用实例化，处处都是一个实例，内部类可以访问外部类私有成员。

```java
class Outer {
    private static String NAME = "OUTER";
    private String name;

    Outer(String name) {
        this.name = name;
    }

    static class StaticNested {
        void hello() {
            System.out.println("Hello, " + Outer.NAME);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Outer.StaticNested sn = new Outer.StaticNested();
        sn.hello();
    }
}
```



# 数组

数组是**长度固定**的存储**同一数据类型**多个元素的容器。

```java
//新建一个存储3个int类型元素的数组
int[] arr = new int[3];
int[] arr = {1,2,3};

arr[0] = 100;
arr[2] = 200;

arr.length
    
//遍历数组
for (int i=0; i<ns.length; i++) {
    int n = ns[i];
    System.out.println(n);
}

//打印数组
System.out.println(Arrays.toString(ns));

//数组排序
import java.util.Arrays;
Arrays.sort(ns);
```



# string

字符串本质是一个字符数组，是一个常量



### 定义

```java
//new出来的在堆内存，双引号的在常量池（即只要双引号内容一样，就指向同一个地址）
public class StringDemo {
	public static void main(String[] args) {
		String s1 = new String("hello");
		
        char[] value = {'h','e','l','l','o'};
		String s2 = new String(value);
		String s3 = new String(value,0,3);
	
		String s4 = "hello";
	}
}

// “\” 在字符串中表示转义 
// "\r\n" 在Windows中换行
```



### 查

```java
public class StringTest2 {
	public static void main(String[] args) {
		String s = "abcde";
		
        for(int x=0; x<s.length(); x++) {
			System.out.println(s.charAt(x));
		}
	}
}
```



### StringBuilder

string是一个常量，每次拼接都会产生新的字符串，StringBuilder是一个容器，怎么拼接都是一个StringBuilder

```java
public class StringBuilderDemo {
	public static void main(String[] args) {
		StringBuilder sb = new StringBuilder();
		StringBuilder sb2 = new StringBuilder("helloworld");
        
        String s = sb.toString()

		sb.append("hello").append("world").append(true).append(100);//链式编程
		sb.reverse();
	}
}
```



### string转换

```
基本类型-字符串				+""
字节-字符串					new String(bys,0,len)
数组-字符串					Arrays.toString(int[])
Stringbuilder-字符串		 sb.toString() 


字符串-基本类型				Integer构造(string)	in.parseInt(s)
字符串-字节					 "ABCDE".getBytes()
字符串-字符数组				s.charAt(index)		s.toCharArray
字符串-Stringbuilder		   Stringbuilder构造(String) 
```



# object

object 是所有类的父类



自己写的类需要  重写 object 类的 get、set 方法，

重写 tostring 方法，打印名称而不是地址

重写equals方法用于比较对象内容是否相同，不重写比较地址是否相同



# 枚举类

```java
// 枚举类中的每一个元素都是 Weekday 实例
enum Weekday {
    SUN, MON, TUE, WED, THU, FRI, SAT;
}


public class Main {
    public static void main(String[] args) {
        Weekday day = Weekday.SUN;
        if (day == Weekday.SAT || day == Weekday.SUN) {
            System.out.println("Work at home!");
        } else {
            System.out.println("Work at office!");
        }
    }
}

String s = Weekday.SUN.name(); // "SUN"
int n = Weekday.MON.ordinal(); // 1
```

```java
// 可以给每一个枚举的实例加上属性，便于使用
enum Weekday {
    MON(1, "星期一"), TUE(2, "星期二"), WED(3, "星期三"), THU(4, "星期四"), FRI(5, "星期五"), SAT(6, "星期六"), SUN(0, "星期日");

    public final int dayValue;
    private final String chinese;

    private Weekday(int dayValue, String chinese) {
        this.dayValue = dayValue;
        this.chinese = chinese;
    }

    @Override
    public String toString() {
        return this.chinese;
    }
}

public class Main {
    public static void main(String[] args) {
        Weekday day = Weekday.SUN;
        if (day.dayValue == 6 || day.dayValue == 0) {
            System.out.println("Today is " + day + ". Work at home!");
        } else {
            System.out.println("Today is " + day + ". Work at office!");
        }
    }
}
```



# 包装类

为了扩展基本数据类型的操作，每个基本类型都有一个对应的包装类

| Byte      | byte    |
| --------- | ------- |
| Short     | short   |
| Integer   | int     |
| Long      | long    |
| Float     | float   |
| Double    | double  |
| Character | char    |
| Boolean   | boolean |

### 定义

```java
public class IntegerDemo {
	public static void main(String[] args) {
		int value = 100;
		Integer i = new Integer(value);
        int x = i.intValue();
        
		String s = "100";
		Integer ii = new Integer(s);
	}
}
```



### 自动装箱

```java
public class IntegerDemo {
	public static void main(String[] args) {
        //自动装箱
		Integer ii = 100; 	
        //自动拆箱,自动装箱
		ii = ii + 200; 
		System.out.println(ii);
		
		Integer ii = Integer.valueOf(100);
		ii = Integer.valueOf(ii.intValue() + 200);
		System.out.println(ii);
	}
}
```



### int和String相互转换

```java
public class IntegerDemo {
	public static void main(String[] args) {
        //int 转 string
		int number = 100;
		String s1 = "" + number;
        
        //String 转 int
        String s = "100";
        int y = Integer.parseInt(s);	
	}
}
```



# 集合

容器：**集合**、**StringBuilder**、**数组**

集合类长度可变

泛型指定什么类型，就严格传什么类型，不可传该泛型的子类

集合用多态的方法定义，自己写着费劲，但是好改子类的类型。

![image-20201230161217739](picture/image-20201230161217739.png)

（ArrayList、HashSet、HashMap）



## Collection

单列集合的顶层接口，一般不用。

list 和 set 接口继承了 collection 接口

```java
public class CollectionDemo2 {
	public static void main(String[] args) {
		Collection<String> c = new ArrayList<String>();

		c.add("hello");//增
		c.remove("world");//删
		c.clear();//清空
        
        c.contains("world");
		c.isEmpty();
		c.size();
        
        //遍历，通过迭代器
		Iterator<String> it = c.iterator();
		while(it.hasNext()){
			String s = it.next();
			System.out.println(s);
            
        for(String s :list) {//增强for，适用于所有容器类对象
			System.out.println(s);
        }
	} 
}
```



### List/ArrayList

有序，重复

实现类ArrayList 和 LinkedList 



ArrayList:底层数据结构是数组，查询快，增删慢

LinkedList:底层数据结构是链表，查询慢，增删快

```java
public class ListDemo2 {
	public static void main(String[] args) {
		List<String> list = new ArrayList<String>();
        //继承的Collection方法
        list.add("hello");
        list.remove("world");
        list.clear();
        list.contains("world");
		list.isEmpty();
		list.size();
        
        Iterator<String> it = list.iterator();
		while(it.hasNext()){
			String s = it.next();
			System.out.println(s);
        }
        
        for(String s :list) {//增强for，适用于所有容器类对象
			System.out.println(s);
        }
        
        //List特有的方法
		list.add(1, "javaee");//增
		System.out.println("remove:"+list.remove(1));//删
		System.out.println("get:"+list.get(1)); //查
		System.out.println("set:"+list.set(1, "javaee"));//改
        
        for (int i = 0; i < list.size(); i++) {//遍历的同时需要增删改时，不要用迭代器
			System.out.println(list.get(i));  
		}
	}
}
```



### set/HashSet

无序，唯一

实现类HashSet

```java
//将自定义类的对象存入HashSet去重复，类中必须重写hashCode()和equals()方法
public class SetDemo {
	public static void main(String[] args) {
		Set<String> set = new HashSet<String>();
        
        //继承的Collection方法
        set.add("hello");
        set.remove("world");
        set.clear();
        set.contains("world");
		set.isEmpty();
		set.size();
        
        Iterator<String> it = set.iterator();
		while(it.hasNext()){
			String s = it.next();
			System.out.println(s);
        }
        
        for(String s :set) {//增强for，适用于所有容器类对象
			System.out.println(s);
        }	
	}
}
```



## map/HashMap

实现类HashMap

```java
public class MapDemo2 {
	public static void main(String[] args) {
		Map<String,String> map = new HashMap<String,String>();

		map.put("张无忌", "赵敏");
		map.remove("郭靖");
		map.clear();
        map.get("张无忌")
		
		map.containsKey("郭靖");
        map.containsValue("赵敏")
		map.isEmpty();
		map.size();
	}
}

public class MapDemo3 {
	public static void main(String[] args) {
		Map<String,String> map = new HashMap<String,String>();
        
        //获取所有值的集合
        Collection<String> values = map.values();
		for(String value : values) {
			System.out.println(value);
		}
        
        //获取所有键的集合，并打印所有键值对
		Set<String> set = map.keySet();
		for(String key : set) {
			String value = map.get(key);
			System.out.println(key+"---"+value);
		}

        //获取所有键值对的集合，并打印所有键值对
        Set<Map.Entry<String,String>> set = map.entrySet();
		for(Map.Entry<String,String> me : set) {
			String key = me.getKey();
			String value = me.getValue();
			System.out.println(key+"---"+value);
		}
	}
}
```



# File

```java
public class FileDemo {
	public static void main(String[] args) {
        //创建文件对象
		File f1 = new File("d:\\aa\\b.txt");
        
        //创建文件
		File f1 = new File("d:\\a.txt");
		f1.createNewFile();
        //创建目录
		File f2 = new File("d:\\bb");
		f2.mkdir();
        
        //删除文件和目录，目录为空才能删
        File f1 = new File("b.txt");
		f1.delete();
        File f3 = new File("aa");
		System.out.println(f3.delete());
        
        //判断功能
		System.out.println("isDirectory:"+f.isDirectory());
		System.out.println("isFile:"+f.isFile());
		System.out.println("exists:"+f.exists());
		
		//获取功能
		System.out.println("getAbsolutePath:"+f.getAbsolutePath());
        System.out.println(new File("./").getCanonicalPath());//解析好的绝对路径
		System.out.println("getPath:"+f.getPath());
		System.out.println("getName:"+f.getName());    
	}
```



# IO

应用：文件复制、文件上传、文件下载

文件都是通过字节存储的，字节+不同解码方式（不同文件格式）就可以变成 字符、音频、视频等。

其中**字符**的编码方式有 ASCII 、UTF-8 、GBK 等。



### 字节缓冲流

FileOutputStream、BufferedOutputStream

FileInputStream、BufferedInputStream

```java
public class BufferedStreamDemo {
	public static void main(String[] args) throws IOException {
		BufferedInputStream bis = new BufferedInputStream(new FileInputStream("a.txt"));
        BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("a.txt",true));//追加

		byte[] bys = new byte[1024];
		int len;
        //读
		while((len=bis.read(bys))!=-1) {
            //写
			bos.write(bys,0,len)
		}
		
        bos.close();
		bis.close();
	}
}
```

```java
//编码
"ABCDE".getBytes()
//解码
new String(bys,0,len)
```



### 字符缓冲流

字符流的默认编码方式是utf-8 ，更改的话需要用转换流。

FileReader、BufferedReader

FileWriter、BufferedWriter

```java
public class TestBase {
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader("a.txt"));
        BufferedWriter bw = new BufferedWriter(new FileWriter("b.txt",true));//追加

        String line;
        //读一行
        while((line=br.readLine())!=null) {
            bw.write(line);
            bw.newLine();//换行
            bw.flush();//刷新
        }

        bw.close();
        br.close();
    }
}
```



### 转换流

转换流就是字符流。

字符 = 字节+解码。转换流需要传入字节流和编码方式。

```java
public class TestBase {
    public static void main(String[] args) throws IOException {
        InputStreamReader fReader = new InputStreamReader(new FileInputStream(filePathString),"UTF-8");
		BufferedReader reader = new BufferedReader(fReader);
        
        OutputStreamWriter fWriter = new OutputStreamWriter(new FileOutputStream(filePathString),"UTF-8");
		BufferedWriter writer = new BufferedWriter(fWriter);
    }
}
```



# 异常

## JDK的Throwable

error 是错误，程序本身无能为力。

exception是异常，程序可以处理。

编译期异常必须 try_catch 或者 throw 处理。运行期异常尽量通过修改代码处理。

![20190214200040693](picture/20190214200040693.jpg)



## 引发异常方法

Java中的**异常**可以是函数中的**语句执行时引发**的，也可以是程序员通过**throw 语句手动抛出**的。

```java
public class AllDemo{
    public static void main (String [] args ){
		Scanner scan = new Scanner ( System. in );
        int num1 = scan .nextInt () ;
        int num2 = scan .nextInt () ;
        int result = num1 / num2 ;
        System . out. println( "result:" + result) ;
        scan .close () ;   
    }
    
/*****************************************

----欢迎使用命令行除法计算器----
2
0
Exception in thread "main" java.lang.ArithmeticException : / by zero
     at com.example.AllDemo.devide( AllDemo.java:30 )
     at com.example.AllDemo.CMDCalculate( AllDemo.java:22 )
     at com.example.AllDemo.main( AllDemo.java:12 )

*****************************************/
```

```java
public class ExceptionDemo4 {
    public void save(User user){
      if(user  == null) 
          throw new IllegalArgumentException("User对象为空"); 
	}
}
```



## 异常处理方法

java处理异常是**termination model of exception handling**（终结式异常处理模式）：

即异常抛出点之后的代码不会被执行，而只会执行捕获这个异常处catch 及其之后同层级 的代码 或者 抛给jvm默认处理。



### JVM默认处理

异常发生时，因为函数是层级调用的，异常会层层打印上抛，形成异常追踪栈，最后抛给jvm进行处理。

A:把异常的名称，异常的原因，异常出现的位置等信息在控制台输出

B:让程序停止执行



### try_catch

catch捕获多种异常时，先catch子类异常。

finally无论怎样，最后都会执行。



```java
public class ExceptionDemo4 {
	public static void main(String[] args) {
        SimpleDateFormat sdf = null;
        
		try{
			String s = "abcd";
			sdf = new SimpleDateFormat("yyyy-MM-dd");
			Date d = sdf.parse(s);
		}catch(ParseException e) {
			e.printStackTrace();
		}finally{
           	System.out.println("程序结束。");  
            //不要在finally中抛出异常。
            //finally块仅仅用来释放资源是最合适的。
   		}
	}
}
```

强制：将所有的return写在函数的最后面，而不是try ... catch ... finally中。



### throws

写在方法后，声明可能抛出的异常

```java
public void testException() throws IOException
{
    //FileInputStream的构造函数会抛出FileNotFoundException
    FileInputStream fileIn = new FileInputStream("E:\\a.txt");
    
    //close方法会抛出IOException
    fileIn.close()
}
```



## 自定义异常

自定义异常需要4个构造函数：

- 一个无参构造函数
- 一个带有String参数的构造函数，并传递给父类的构造函数。
- 一个带有String参数和Throwable参数，并都传递给父类构造函数
- 一个带有Throwable 参数的构造函数，并传递给父类的构造函数。

```java
public class IOException extends Exception
{
    static final long serialVersionUID = 7818375828146090155L;

    public IOException()
    {
        super();
    }

    public IOException(String message)
    {
        super(message);
    }

    public IOException(String message, Throwable cause)
    {
        super(message, cause);
    }

    
    public IOException(Throwable cause)
    {
        super(cause);
    }
}
```



## 异常的链化

主要用于处理异常时再手动抛新异常，避免新抛的异常把之前捕获的异常掩盖掉

```java
public static void main(String[] args)
{
    System.out.println("请输入2个加数");
    int result;
    try
    {
        result = add();
        System.out.println("结果:"+result);
    } catch (Exception e){
        e.printStackTrace();
    }
}
//获取输入的2个整数返回
private static List<Integer> getInputNumbers()
{
    List<Integer> nums = new ArrayList<>();
    Scanner scan = new Scanner(System.in);
    try {
        int num1 = scan.nextInt();
        int num2 = scan.nextInt();
        nums.add(new Integer(num1));
        nums.add(new Integer(num2));
    }catch(InputMismatchException immExp){
        throw immExp;
    }finally {
        scan.close();
    }
    return nums;
}

//执行加法计算
private static int add() throws Exception
{
    int result;
    try {
        List<Integer> nums =getInputNumbers();
        result = nums.get(0)  + nums.get(1);
    }catch(InputMismatchException immExp){
        throw new Exception("计算失败",immExp);  /////////////////////////////链化:以一个异常对象为参数构造新的异常对象。
    }
    return  result;
}

/*
请输入2个加数
r 1
java.lang.Exception: 计算失败
    at practise.ExceptionTest.add(ExceptionTest.java:53)
    at practise.ExceptionTest.main(ExceptionTest.java:18)
Caused by: java.util.InputMismatchException
    at java.util.Scanner.throwFor(Scanner.java:864)
    at java.util.Scanner.next(Scanner.java:1485)
    at java.util.Scanner.nextInt(Scanner.java:2117)
    at java.util.Scanner.nextInt(Scanner.java:2076)
    at practise.ExceptionTest.getInputNumbers(ExceptionTest.java:30)
    at practise.ExceptionTest.add(ExceptionTest.java:48)
    ... 1 more

*/
```



# 工具类

工具类的设计思想：

构造方法私有。

成员都用static修饰



### Arrays

操作数组的工具类。

```java
import java.util.Arrays;

public class ArraysDemo {
	public static void main(String[] args) {
		int[] arr = {24,69,80,57,13};
		
		//public static String toString(int[] a):把数组转成字符串
		System.out.println(Arrays.toString(arr));
		
		//public static void sort(int[] a):对数组进行升序排序
		Arrays.sort(arr);
	}
}
```



### Scanner

```java
package com.zhy;
import java.util.Scanner;

public class TestBase {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        //接受数据类型为基本数据类型和字符串类型
        int x = sc.nextInt();
        System.out.println(x);
    }
}
```



### Random

```java
package com.itheima;
import java.util.Random;

public class RandomDemo {
	public static void main(String[] args) {
		Random r = new Random();
		int number = r.nextInt(10);
		System.out.println("number:"+number);
				
		//如何获取一个1-100之间的随机数呢?
		int i = r.nextInt(100)+1;
	}
}
```



### SimpleDateFormat

```java
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class SimpleDateFormatDemo {
	public static void main(String[] args) throws ParseException {
		//格式化: Date -- String
		Date d = new Date();
		SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss");
		String s = sdf.format(d);

		//解析:  String -- Date
		String str = "2080-08-08 12:23:45";
		SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		Date d = sdf.parse(str);
	}
}
```



# Tips

### 斜杠

1.在windows系统大部分情况下斜杠/和反斜杠\可以互用,特殊时候只能用反斜杠\

2.浏览器地址栏网址使用 斜杆/ ;

3.windows文件导航栏上使用 反斜杠\ ;

4.出现在html url() 属性中的路径，指定的路径是网络路径，所以必须用 斜杆/ ;

5.出现在普通字符串中的路径，如果代表的是windows文件路径，则使用 斜杆/ 和 反斜杠\ 是一样的；如果代表的是网络文件路径，则必须使用 斜杆/ 



### 路径

```
代码中的路径都写在字符串中。字符串中用 \ 代表转译，字符串中的反斜杠写作 \\ 

java项目中 需要指定路径的地方都参数化比较好，可以在提交jar包的时候指定。


路径分割符 【  / 】

【 ./ 】代表当前目录 ，java中的是 jvm的启动路径

【 ../ 】代表上一级目录


绝对路径：完整路径

相对路径：从jvm的启动路径开始写起，一般就是项目目录下
```



### classpath

`classpath`是JVM用到的一个环境变量，

`classpath`就是一组目录的集合，java按顺序在这些目录下查找编译好的.class文件，

基本不用设置。



### 模块

https://www.liaoxuefeng.com/wiki/1252599548343744/1281795926523938



# --------------------



# JavaBean

class满足以下规范，就被称为javaBean，javabean可以只读或者只写。

- 若干`private`实例字段；
- 通过`public`方法来读写实例字段。

```java
public class Person {
    private String name;
    private int age;

    public String getName() { return this.name; }
    public void setName(String name) { this.name = name; }

    public int getAge() { return this.age; }
    public void setAge(int age) { this.age = age; }
}

// 枚举javabean属性
public class Main {
    public static void main(String[] args) throws Exception {
        BeanInfo info = Introspector.getBeanInfo(Person.class);
        for (PropertyDescriptor pd : info.getPropertyDescriptors()) {
            System.out.println(pd.getName());
            System.out.println("  " + pd.getReadMethod());
            System.out.println("  " + pd.getWriteMethod());
        }
    }
}
```

JavaBean主要用来传递数据，即把一组数据组合成一个JavaBean便于传输



# 泛型

泛型就是参数化类型。把类型作为一个参数传递进去。

```
泛型的标志就是 <>
```

泛型既然是参数，就有形参和实参。

```java
//实参在调用的时候传入。类型实参就是各种类型，如String。
List<String> stringList = new ArrayList<String>();

//形参在定义类、接口和方法的时候使用，可以是任何一个大写字母
public interface List<E> extends Collection<E>{}
```

把类型作为参数可以传递给类，也可以传递给方法。但不论传给谁，都不改变类和方法的本质。

```java
//泛型的接口的定义
public interface Listai<E> {}
//泛型的类继承泛型的接口
public class Listac<E> extends Listai<E> {
    public void test(E e){
    	System.out.println(e.getclass().getTypeName());
    }
}
//泛型的类实例化和使用
Listac<String> aa = new Listac<String>();
aa.test("aaa");

//继承泛型的类时要指定实参，才能继承。默认object。
public class Listaoo extends Listac<String> {}

//泛型的类作为方法形参时，使用通配符代替泛型的类的大写字母。这里的通配符的作用是在不知道泛型的时候，限制泛型的类型
public void test(List<?> list){} //无限制
public List getList(List<? extends Test1> list) {} //限制泛型必须为Test1的子类的类型。
```

```java
//泛型的方法的定义
public <T> void test(T t){
    System.out.println(t.getclass().getTypeName());
}

//泛型方法的使用
Demo.test("aaa");
```

泛型在编译阶段，在执行阶段，所有的泛型都会变成具体数据类型，而类、接口和方法会恢复原样。这个叫做**泛型擦除**。



# JDBC

JDBC 是Java API 中用来操作数据库的一组接口规范。

驱动 ,数据库厂家写的一组实现JDBC 的实现类jar包。

JDBC一般用连接池工具类 C3P0、DRUID等来实现。



SQL注入：用户将SQL关键字放入输入内容中，达到操作数据库的目的。

防止SQL注入：使用preparedStatement等，过滤输入的关键字，将SQL语句预编译。

```java
1注册驱动
//java中的DriverManager类进行注册
//DriverManager.registerDriver(new Driver())
//Driver类中有静态代码块执行了注册动作,只要加载类就可以了
Class.forName("com.mysql.jdbc.Driver") 
		
2获得连接
Connection conn = DriverManager.getConnection(url,"rooot","root")
url = "jdbc:mysal://localhost:3366/day03"
							
3获得执行SQL语句的对象
Statement stmt = conn.creatStatement();
	
4执行SQL语句,返回结果
ResultSet rs = stmt.executeQuery("select*from category")
int		i	=	stmt.executeUpdate(增删改语句)
		
5处理结果
while(rs.next()){
    Integer cid = rs.getInt("cid");
	String cname = rs.getString("cname");
	System.out.println(cid + " , " + cname);
}
	
6释放资源
rs.close();
stmt.close();
conn.close();

//改动3和4，为了解决Sql注入问题
PreparedStatement ps = conn.preparedStatement(含站位符的sql)
ps.setxxx(第几个?,设置的值)
ResultSet rs = stmt.executeQuery()
int		i	=	stmt.executeUpdate()
    
//对1和2 用连接池进行优化
DataSource 是Java API 中用来管理连接池的一个接口规范
C3P0 jar包是用来实现DataSource的
导入包后编写C3P0连接池工具类配置,文件写在src下)

public class Utils {
    public static ComboPooledDataSource ds = new ComboPooledDataSource();
    //C3P0全自动调用配置文件 ,无需注册, 提供连接地址 和数据库账号密码,自己写要调用ds中的四个方法
    public static Connection getconnection() throws SQLException {
        return ds.getConnection();
    }   
    public static void close(ResultSet r, Statement s, Connection c) {
        if (r != null) {
            try {
                r.close();
            } catch (SQLException e) {
                throw new RuntimeException(e);
            }
        }
        if (s != null) {
            try {
                s.close();
            } catch (SQLException e) {
                throw new RuntimeException(e);
            }
        }
        if (c != null) {
            try {
                c.close();
            } catch (SQLException e) {
                throw new RuntimeException(e);
            }
        }
    }
}
```

```java
public class Demo {
    public static void main(String[] args) throws Exception {
        // 拿到连接
        Connection conn = JdbcUtils.getConnection();
        // 执行sql语句
        String sql = "INSERT INTO student VALUES (NULL, ?, ?, ?);";
        PreparedStatement pstmt = conn.prepareStatement(sql);
        pstmt.setString(1, "李四");
        pstmt.setInt(2, 30);
        pstmt.setDouble(3, 50);
        int i = pstmt.executeUpdate();
        System.out.println("影响的函数: " + i);
        // 关闭资源
        JdbcUtils.close(conn, pstmt);
    }
}
```













https://www.liaoxuefeng.com/wiki/1252599548343744/1255943543190176



























