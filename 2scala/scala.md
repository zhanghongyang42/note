官网：https://docs.scala-lang.org/zh-cn/tour/tour-of-scala.html

教程：http://twitter.github.io/scala_school/zh_cn/

博客：https://hongjiang.info/scala/



# 简介

Scala 运行在 Java 虚拟机上，并兼容现有的 Java 程序。

Scala是一种**纯面向对象**的语言。Scala是一门**函数式**语言。Scala中一切皆表达式。

Scala的**函数**也能当成**值**来使用。值也是**对象**。Scala中，对象由**类**和**特质**来描述。



# 安装

详见picture/安装



# 关键字

| >:      |
| ------- |
| forSome |
| lazy    |
| sealed  |
| yield   |
| <%      |
| :       |
| <:      |
| #       |
| -       |



# 运算符

运算符就是方法，任何具有一个参数的方法都可以当作运算符。



算术运算符

```scala
println("a + b = " + (a + b) );
println("a - b = " + (a - b) );
println("a * b = " + (a * b) );
println("b / a = " + (b / a) );
println("b % a = " + (b % a) );
```

关系运算符

```scala
println("a == b = " + (a == b) );
println("a != b = " + (a != b) );
println("a > b = " + (a > b) );
println("a < b = " + (a < b) );
println("b >= a = " + (b >= a) );
println("b <= a = " + (b <= a) );
```

逻辑运算符

```scala
println("a && b = " + (a&&b) );
println("a || b = " + (a||b) );
println("!(a && b) = " + !(a && b) );
```

位运算符

| 运算符 | 描述           | 实例                                                         |
| :----- | :------------- | :----------------------------------------------------------- |
| &      | 按位与运算符   | (a & b) 输出结果 12 ，二进制解释： 0000 1100                 |
| \|     | 按位或运算符   | (a \| b) 输出结果 61 ，二进制解释： 0011 1101                |
| ^      | 按位异或运算符 | (a ^ b) 输出结果 49 ，二进制解释： 0011 0001                 |
| ~      | 按位取反运算符 | (~a ) 输出结果 -61 ，二进制解释： 1100 0011， 在一个有符号二进制数的补码形式。 |
| <<     | 左移动运算符   | a << 2 输出结果 240 ，二进制解释： 1111 0000                 |
| >>     | 右移动运算符   | a >> 2 输出结果 15 ，二进制解释： 0000 1111                  |
| >>>    | 无符号右移     | A >>>2 输出结果 15, 二进制解释: 0000 1111                    |

赋值运算符

```scala
c = a + b;
c += a ;
c -= a ;
c *= a ;
c /= a ;
c %= a ;
c <<= 2 ;
c >>= 2 ;
c >>= a ;
c &= a ;
c ^= a ;
c |= a ;
```

优先级

| 类别 |               运算符               | 关联性 |
| :--: | :--------------------------------: | :----: |
|  1   |               () []                | 左到右 |
|  2   |                ! ~                 | 右到左 |
|  3   |               * / %                | 左到右 |
|  4   |                + -                 | 左到右 |
|  5   |             >> >>> <<              | 左到右 |
|  6   |             > >= < <=              | 左到右 |
|  7   |               == !=                | 左到右 |
|  8   |                 &                  | 左到右 |
|  9   |                 ^                  | 左到右 |
|  10  |                 \|                 | 左到右 |
|  11  |                 &&                 | 左到右 |
|  12  |                \|\|                | 左到右 |
|  13  | = += -= *= /= %= >>= <<= &= ^= \|= | 右到左 |
|  14  |                 ,                  | 左到右 |



# 表达式

表达式是进行计算的语句，由值和函数组成。如：1+1



常量：表达式可以赋值给常量

```scala
val myVal = "Hello, Scala!"
```

变量

```scala
var myVar = 10
```



# 代码块

`{}`包围起来的几个表达式，最后一个表达式的结果是代码块的值

```scala
{
val x = 1 + 1
x + 1
}
```



# 函数

函数是带有参数的表达式



匿名函数：左边是参数列表，右边是一个包含参数的表达式

```scala
(x: Int) => x + 1
```



函数

```scala
val addOne = (x: Int) => x + 1
```



高阶函数：参数或者返回值 是 函数或者方法 的函数

```scala
// 举例 map函数,参数是函数
val salaries = Seq(20000, 70000, 40000)
val newSalaries = salaries.map(x => x * 2) 
val newSalaries = salaries.map(_ * 2) // _可以代表匿名函数的参数

//返回值是函数
def urlBuilder(endpoint: String, query: String): (String, String) => String = {
  (endpoint: String, query: String) => endpoint + String
}
```



# 方法

方法和对象相关；

函数和对象无关。

Java中只有方法，C中只有函数，而C++里取决于是否在类中。



类似于函数，方法中可以嵌套方法。

```scala
def getSquareString(input: Double): String = {
    val square = input * input
    square.toString
}
```



柯里化：方法可以有多个参数列表，当使用较少的参数列表调用多参数列表的方法时，会产生一个新的函数，该函数接收剩余的参数列表作为其参数。

```scala
//多参数列表定义
def foldLeft[B](z: B)(op: (B, A) => B): B
def execute(arg: Int)(implicit ec: ExecutionContext) = ??? //定义方法需要隐式参数时使用多参数列表

//多参数列表调用
val numbers = List(1, 2, 3, 4, 5, 6, 7, 8, 9)
val res = numbers.foldLeft(0)(_ + _)

//柯里化调用，提高适用性，延迟执行，固定易变因素
val numbers = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
val numberFunc = numbers.foldLeft(List[Int]())_ //_把方法转化成函数
val squares = numberFunc((xs, x) => xs:+ x*x)
```



闭包

```
在方法作用域内读到了方法外部的变量（自由变量），即形成闭包。
闭包是一种概念特性，实现闭包的方法包括作用域链，把函数当作值传递等。
Scala中通过柯里化实现了闭包。
闭包无需刻意实现，在每个语言中自然写代码就实现了闭包。
闭包的作用就是当环境中的自由变量消失时，形成闭包的方法仍然能执行。
```



传名参数

传值参数，是计算好参数值，传入方法内部，方法内多次使用时避免计算

传名参数，是直接将参数表达式传入，再传入方法内部，方法内不使用不计算。

```scala
//传名参数用:=>而不是:
def calculate(input :=> Int) = input * 37
```



# 类

Scala中的类是用于创建对象的蓝图，其中包含了方法、常量、变量、类型、对象、特质、类，这些统称为成员。



访问修饰符 private，protected，public 修饰成员

```Scala
//distance除了对classa类可见之外，对其他所有类都私有
private[classa] val distance = 100
```



```scala
//定义，主构造方法中带有val和var的参数是公有的，否则仅仅在类中可见
class Greeter(prefix: String, suffix: String) {
    def greet(name: String): Unit = println(prefix + name + suffix)
}

//实例化
val greeter = new Greeter("Hello, ", "!")
```

```scala
//Getter/Setter
class Point {
    // 私有成员
    private var _x = 0
    //getter
    def x = _x
    //setter,特殊语法：在getter方法x后加_=，就是setter方法名
    def x_= (newValue: Int): Unit = {
        _x = newValue
    }
}

val point1 = new Point
point1.x = 99
```



辅助构造方法

```scala
//主构造器，直接定义在类上面
class Dog (name:String,age:Int){
   def this(name:String,age:Int,gender:String){
    //每个辅助构造器，都必须以其他辅助构造器，或者主构造器的调用作为第一句
    this(name:String,age:Int)
    this.gender = gender
  }

  private def this(name:String,age:Int,color:String,gender:String){
    this(name:String,age:Int)
    this.color = color
  }
}
```



内部类

```scala
//定义一个内部类
class Graph {
    class Node {def connectTo(node: Node) {}}
    var nodes: List[Node] = Nil
}

//初始化内部类实例，注意内部类类型由外部类实例决定 graph1.Node
val graph1: Graph = new Graph
val node1: graph1.Node = graph1.newNode

//内部类中用到的内部类类型变量，实例化后，受外部类实例的影响
val node2: graph1.Node = graph1.newNode
node1.connectTo(node2)//没问题
val graph2: Graph = new Graph
val node3: graph2.Node = graph2.newNode
node1.connectTo(node3)//java没问题，scala会报错

//除非定义的时候用 外部类#内部类 的形式
class Graph {
    class Node {def connectTo(node:Graph#Node) {}}
    var nodes: List[Node] = Nil
}
```



# case类

样例类一般用于不可变对象，可作值比较。

```scala
//定义，案例类的参数是都是public val
case class Point(x: Int, y: Int)

//实例化
val point = Point(1, 2)

//比较时是值比较，不是地址比较
if (point == anotherPoint) {println(point + " and " + anotherPoint + " are the same.")} 
```



# 单例对象

单例对象是一种特殊的类，第一次被使用时创建。

单例对象定义在一个类或方法中时，单例对象表现得和惰性变量一样。



```scala
object IdFactory {
    private var counter = 0
    def create(): Int = {
        counter += 1
        counter
    }
}

val newId: Int = IdFactory.create()

//多用于创建功能性方法后被引用
import logging.Logger.info
info("Created projects")
```



提取器对象

提取器对象是一个包含有 `unapply` 方法的单例对象。`apply` 方法用于object在首次使用时接收参数创建实例，`unapply` 方法接受实例对象然后返回参数。

提取器常用在模式匹配和偏函数中。

```scala
//定义提取器对象
object CustomerID {
    def apply(name: String) = name
    def unapply(customerID: String): Option[String] = {
        customerID
    }
}
//调用了apply方法
val customer2ID = CustomerID("Nico")
//调用了unapply方法
val CustomerID(name) = customer2ID  //等价于 val name = CustomerID.unapply(customer2ID).get
println(name)

//提取器对象用于模式匹配
customer2ID match {
  case CustomerID(name) => println(name)
  case _ => println("Could not extract a CustomerID")
}
```



伴生对象

当一个单例对象和某个类共享一个名称时，这个单例对象称为 *伴生对象*。这个类称为伴生类。类和它的伴生对象可以互相访问其私有成员。

使用伴生对象来定义那些在伴生类中不依赖于实例化对象而存在的成员变量或者方法。

类和它的伴生对象必须定义在同一个源文件里。

```scala
//伴生类
case class Circle(radius: Double) {
  import Circle._
  def area: Double = calculateArea(radius)
}
//伴生对象，用于定义伴生类的静态成员
object Circle {
  private def calculateArea(radius: Double): Double = Pi * pow(radius, 2.0)
}

//实例化伴生类
val circle1 = Circle(5.0)
//通过伴生对象的工厂方法fromString，用任意string实例化伴生类
val scalaCenterEmail = Email.fromString("scala.center@epfl.ch")
```



# 特质

特质 (Traits) 用于在类之间或对象 (Objects)之间共享程序接口和字段。类似于Java 8的接口。

特质不能被实例化，没有参数。



继承

```scala
//特质定义的时候，抽象方法不实现方法，泛型不指定类型，很常用
trait Iterator[A] {
  def hasNext: Boolean
  def next(): A
}

class IntIterator(to: Int) extends Iterator[Int] {
  override def hasNext: Boolean = 0 < to
  override def next(): Int = to + 5
}
```



多态

```scala
import scala.collection.mutable.ArrayBuffer

//特质
trait Pet {val name: String}
//子类
class Cat(val name: String) extends Pet
val cat = new Cat("Sally")

val animals = ArrayBuffer.empty[Pet]
animals.append(cat)
//多态
animals.foreach(pet => println(pet.name))
```



混入

一个类只能有一个父类但是可以有多个混入。

```scala
class B {
  val message = "I'm an instance of class B"
}
trait C {
  def loudMessage = message.toUpperCase()
}
class D extends B with C
```



自类型

```scala
//定义一个特质时可以使用自类型调用其他特质
trait User {
    def username: String
}

trait Tweeter {
    //自类型的形式，可以调用trait特质的成员
    this: User =>  
    def tweet(tweetText: String) = println(s"$username: $tweetText")
}

//使用了自类型的特质被继承时，所有用到的特质也要混入
class VerifiedTweeter(val username_ : String) extends Tweeter with User {
	def username = s"real $username_"
}
```



# 类型

Scala的函数也能当成值来使用。值也是对象。所有值都有类型。



层次结构

![Scala Type Hierarchy](picture/unified-types-diagram.svg)

Any 是所有类型的超类型，也称为顶级类型。它定义了一些通用的方法如`equals`、`hashCode`和`toString`。

AnyVal 基本类型：Byte、Short、Int、Long、Float、Double、Char  、Boolean、Unit。

AnyRef 引用类型，相当于java的Object。



类型转换

![Scala Type Hierarchy](picture/type-casting-diagram.svg)

Nothing和Null

没有一个值是`Nothing`类型的。它的用途之一是给出非正常终止的信号，如抛出异常、程序退出或者一个无限循环。

`Null`是所有引用类型的子类型。它有一个单例值由关键字`null`所定义。`Null`主要是使得Scala满足和其他JVM语言的互操作性。

Unit 类型也只有一个值()。



复合类型：可以给一个变量多个类型，用with

```scala
def cloneAndReset(obj: Cloneable with Resetable): Cloneable = {}
```



# 集合

不可变集合操作：+  -  ++ +:  ::: :: 操作 都会产生新的集合对象

可变集合操作：+= -= ++= --=    操作不会产生新的对象 

picture/conllction.xmind



数组

```scala
val array = new Array[Int](10)
val array = Array(1,2,3)
array(1) = 10
println(array(1))

val array3 = new ArrayBuffer[Int]()
array3.append(20)

val toBuffer = array.toBuffer
val toArray = array3.toArray

array6.sum
array6.max
array6.sorted
```

多维数组

```scala
val dim = Array.ofDim[Double](3,4)
dim(1)(1) = 11.11
println(dim.mkString(","))
```

元组：在 Scala 中，元组是一个可以容纳不同类型元素的类。 元组是不可变的。

```scala
//定义元组，元组类 Tuple2到Tuple22
val ingredient = ("Sugar" , 25):Tuple2[String, Int]
val (name, quantity) = ("Sugar" , 25)

//调用
println(ingredient._1)
println(name)
```

Map

```scala
val map2 = scala.collection.mutable.Map("hello" ->"world","name" -> "zhangsan","age" -> 18)

map2.+=("address" ->"地球")
val map3 = map2.-("address")
map2("address") ="火星"
map2.get("address")
map2.keys
map2.values

val  arrayMap = Array(("name","zhangsan"),("age",28))
val toMap = arrayMap.toMap
```

List

```scala
val list1 = List("hello",20,5.0f)
val  list6 = new ListBuffer[String]

val list1Result = list1(0) //查
List.range(2, 6)
val list2  = list1:+50	//尾部添加
val list3 = 100+:list1	//头部添加
val list4 = 1::2::3::list1::Nil //Nil 空列表
List(1, 2, 3)++List(4, 5, 6) //集合合并
nums drop 3 //丢弃前3个元素

nums.isEmpty
nums.head
nums.tail.head
nums.tail.tail.head
nums.last
nums.reverse
nums.splitAt(2)

nums.toArray
```

set

```scala
import scala.collection.mutable.Set
val set2 = Set(1, 2, 3)

set2 += 5
set3 -= 1
```

Queue

```scala
val queue1 = new  mutable.Queue[Int]()

queue1.enqueue(5,6,7)
val dequeue = queue1.dequeue()

println(queue1.head)
println(queue1.last)
```



# 包

```scala
package com.runoob 	// 定义包,一般与目录相同

// scala 导入包不在顶部，有作用范围。
import java.awt._  // 引入包内所有成员。

//如果存在命名冲突并且你需要从项目的根目录导入，请在包名称前加上 _root_：
import _root_.users._
```



包对象

Scala 提供包对象作为在整个包中方便的共享使用的容器。

包对象中可以定义任何内容，而不仅仅是变量和方法。 例如，包对象经常用于保存包级作用域的类型别名和隐式转换。 包对象甚至可以继承 Scala 的类和特质。

每个包都允许有一个包对象。 在包对象中的任何定义都被认为是包自身的成员。包对象的代码通常放在名为 `package.scala` 的源文件中。

```scala
//定义包对象，将 planted 和 showFruit 放入包 gardening中。
package gardening
package object fruits {
  val planted = List(Apple, Plum, Banana)
  def showFruit(fruit: Fruit): Unit = {
    println(s"${fruit.name}s are ${fruit.color}")
  }
}
```



# 循环结构

```scala
for( a <- 1 to 10){
    println( "Value of a: " + a );
}

while( a < 20 ){
    println( "Value of a: " + a );
    a = a + 1;
}
```



# 模式匹配

模式匹配是检查某个值（value）是否匹配某一个模式的机制。是Java中的`switch`语句的升级版，同样可以用于替代一系列的 if/else 语句。

```scala
def matchTest(x: Int): String = x match {
  case 1 => "one"
  case 2 => "two"
  case _ => "other"
}

//模式守卫
def matchTest(x: Int): String = x match {
  case 1 if true => "one"
  case 2 if true => "two"
  case _ => "other"
}
```

case class 模式匹配

```scala
abstract class Notification
case class Email(sender: String, title: String, body: String) extends Notification
case class SMS(caller: String, message: String) extends Notification
case class VoiceRecording(contactName: String, link: String) extends Notification

//仅匹配类型
def showNotification(notification: Notification): String = {
    notification match {
        case e:Email => s"You got an email from $sender with title: $title"
        case s:SMS => s"You got an SMS from $number! Message: $message"
        case v:VoiceRecording => s"you received a Voice Recording from $name! Click the link to hear it: $link"
  }
}
//匹配类型和参数
def showNotification(notification: Notification): String = {
    notification match {
        case Email(sender, title, _) =>s"You got an email from $sender with title: $title"
        case SMS(number, message) =>s"You got an SMS from $number! Message: $message"
        case VoiceRecording(name, link) =>s"you received a Voice Recording from $name! Click the link to hear it: $link"
  }
}
val someSms = SMS("12345", "Are you there?")
println(showNotification(someSms)) 
```

密封类：特质（trait）和类（class）可以用`sealed`标记为密封的，这意味着其所有子类都必须与之定义在相同文件中，从而保证所有子类型都是已知的。

```scala
// 对于模式匹配很有用，因为我们不再需要一个匹配其他任意情况的case
sealed abstract class Furniture
case class Couch() extends Furniture
case class Chair() extends Furniture

def findPlaceToSit(piece: Furniture): String = piece match {
  case a: Couch => "Lie on the couch"
  case b: Chair => "Sit on the chair"
}
```



# 泛型

类的泛型用法同java。



scala 的泛型可以是协变，逆变，不变的。使用型变，可以让我们直接把**泛型**的继承关系变成**泛型所修饰类**的继承关系。

```scala
//定义有继承关系的泛型，是准备工作
abstract class Animal {def name: String}
case class Cat(name: String) extends Animal
```

协变：协变就是泛型所修饰类的 父类可以接收子类。

协变List[+A]就代表两个类Printer[A]和Printer[A的子类]

```scala
//使用已有的协变类List[+A]演示协变用法
//演示开始
object CovarianceTest extends App {
    //printAnimalNames 方法只接收 List[Animal]
    def printAnimalNames(animals: List[Animal]): Unit = animals.foreach { animal =>println(animal.name)}
    //得到List[Cat]类型实例
    val cats: List[Cat] = List(Cat("Whiskers"), Cat("Tom"))
    //因为List[+A]是协变的，所以可以把 Cat 与 Animal的关系变为 List[Animal]和 List[Cat]的关系，使父类可以接收子类
    printAnimalNames(cats)
}
```

逆变：逆变就是泛型所修饰类 子类可以接收父类。

逆变Printer[-A]就代表两个类Printer[A]和Printer[A的父类]

```scala
//定义一个逆变类Printer[-A]演示逆变用法
abstract class Printer[-A] {def print(value: A): Unit}
class AnimalPrinter extends Printer[Animal] { def print(animal: Animal): Unit = println("The animal's name is: " + animal.name)}
class CatPrinter extends Printer[Cat] { def print(cat: Cat): Unit = println("The cat's name is: " + cat.name)}
//演示开始
object ContravarianceTest extends App {
    //printMyCat 方法只接收 Printer[Cat]
    def printMyCat(printer: Printer[Cat]): Unit = printer.print(Cat("Boots"))
    //得到Printer[Animal]类型实例
    val animalPrinter: Printer[Animal] = new AnimalPrinter
    //因为Printer[-A]是逆变的，所以可以把 Cat 与 Animal的关系变为 Printer[Animal]和 Printer[Cat]的关系，使子类可以接收父类
    printMyCat(animalPrinter)
}
```

不变：不能把泛型的继承关系变成泛型所修饰类的继承关系

不变Printer[A]就代表Printer[A]



泛型上界

```scala
//泛型P在使用时之能是Pet类的子类
abstract class Pet
class PetContainer[P <: Pet](p: P) {def pet: P = p}
```

泛型下界：https://docs.scala-lang.org/zh-cn/tour/lower-type-bounds.html



抽象类型：泛型变量

特质和抽象类可以包含抽象类型成员

```scala
trait Buffer {
  type T
  val element: T
}
```

https://docs.scala-lang.org/zh-cn/tour/abstract-type-members.html



# 隐式转换

隐式参数不用传，去隐式作用域找参数。

传参类型类型与定义不一致时，去隐式作用域转换类型。



隐式参数：定义时的隐式参数，使用时，不用传实参，自动从上下文的隐式值中推出。

```scala
abstract class Monoid[A] {
  def add(x: A, y: A): A
  def unit: A
}

object ImplicitTest {
    implicit val intMonoid: Monoid[Int] = new Monoid[Int] {
        def add(x: Int, y: Int): Int = x + y
        def unit: Int = 0
    }
  	
    //定义时，参数列表使用隐式参数
    def sum[A](xs: List[A])(implicit m: Monoid[A]): A = if (xs.isEmpty) m.unit else m.add(xs.head, sum(xs.tail))
    //调用隐式参数方法时，可以不传隐式参数，直接从上下文或者伴生对象中找到合适的隐式值 intMonoid 自动填进去。
    def main(args: Array[String]): Unit = println(sum(List(1, 2, 3)))     
}
```



隐式导入

```scala
class SwingType{ def  wantLearned(sw : String) = println("兔子已经学会了"+sw)}
object swimming{ implicit def learningType(s : AminalType) = new SwingType}

class AminalType
object AminalType{
    //隐式导入
    import com.mobin.scala.Scalaimplicit.swimming._
    val rabbit = new AminalType
    //调用不到wantLearned方法，去隐式导入的swimming中找到learningType，将AminalType转换成了SwingType，调用到了wantLearned方法
    rabbit.wantLearned("breaststroke")        
}
```



隐式类型转换

```scala
//定义一个隐式方法
implicit def intToString(x : Int) = x.toString

//调用foo(10)时，编译类型不通过，就去隐式作用域中找方法，将方法参数类型变成返回值类型。
def foo(msg : String) = println(msg)
foo(10)
```



































































