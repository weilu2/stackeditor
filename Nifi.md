
# 变量

## 可变和不可变量
Scala 中变量有两种不同类型：
1）val：不可变，在声明时必须初始化，且之后不能再复制
2）var：可变，声明时需要初始化，之后可以再次赋值

## 数据类型
scala 可以根据赋值推断数据类型
```Scala
val myName = "dcwang"
```

也可以指定类型
```Scala
val myName2 : String = "dcwang"
```

还可以利用全限定名指定类型，Scala中使用的是Java类型
```Scala
val myName3 : java.lang.String = "dcwang"
```

所有 scala 文件默认会导入 java.lang 下面所有的包，在 scala中表示为：
```Scala
import java.lang._
```
用下划线表示所有

```Scala
var price = 1.2
var price2 : Double = 2.2
```
在 scala 中所有的基本类型都是 类

# 基本运算
在 scala 中所有运算都是调用函数
```Scala
val sum1 = 5 + 3
```
相当于
```Scala
val sum2 = (5).+(3)
```

## 富包装类

## Range
```Scala
1 to 5
res1: scala.collection.immutable.Range.Inclusive = Range(1, 2, 3, 4, 5)
```
1 to 5 相当于 1.to(5)
```Scala
scala> 1 until 5
res2: scala.collection.immutable.Range = Range(1, 2, 3, 4)\
```
```Scala
scala> 1 to 10 by 2
res3: scala.collection.immutable.Range = Range(1, 3, 5, 7, 9)
```

# 控制结构
## 判断
```Scala
val x = 6

if (x > 0) {
	// do something
} else {
	// do something
}

if (x > 0) {
	//
} else if (x == 0) {
	//
} else {
	//
}
```
scala 中可以将 if 中判断的值赋值给变量

## while 循环
```Scala
var i = 9

while (i > 0) {
	i -= 1
	printf("i is %d \n", i)
}
```

## for 循环

```Scala
for (i <- 1 to 5)
	println(i)

for (i <- 1 to 5 by 2)
	println(i)

// 守卫（guard）表达式
for (i <- 1 to 5 if i % 2 == 0)
	println(i)

for (i <- 1 to 5; j <- 1 to 3)
	println(i * j)
```

# 文件操作

## 文本文件读写
Scala 需要调用 java.io.PrintWriter 实现文件写入
```Scala
import java.io.PrintWriter

val out = new PrintWriter("output.txt")

for (i <- 1 to 5)
	out.println(i)

out.close()
```

使用 Scala.io.Source 的 getLines 方法实现对文件中所有行的读取
```Scala
import scala.io.Source

val inputFile = Source.fromFile("output.txt")

val lines = inputFile.getLines

for (line <- lines)
	println(line)
```

# 异常捕获

Scala 不支持 Java 中的 checked exception，将所有异常当做运行时异常
Scala 仍然使用 try-catch 捕获异常

```Scala
import java.io.FileReader
import java.io.FileNotFoundException
import java.io.IOException

try {
	val file = new FileReader("NotExistFile.txt")
} catch {
	case ex: FileNotFoundException =>
		// do something
	case ex: IOException =>
		// do something
} finally {
	file.close()
}
```

# 容器 Collections
Scala 提供了一套丰富的容器库，包括列表、数组、集合、映射等
Scala 用三个包来组织容器，分别是 
```Scala
scala.collection
scala.collection.mntable 
scala.collection.immutable
```

## 列表 List
共享相同类型的不可变的对象序列，定义在 scala.collection.immutable 中
List 在声明时必须初始化
```Scala
var strList = List("BigData", "Hadoop", "Spark")

// 返回头部第一个元素
strList.head

// 返回除了第一个之外的其他元素（一个新的List）
strList.tail

// 连接操作（从右侧开始）
var oneList = List(1, 2, 3)
var otherList = List(4, 5)
var newList = oneList::otherList

// Nil 是一个空列表对象
var intList = 1::2::3::Nil
```

## 集合 Set
集合中的元素插入式无序的，以哈希方法对元素的值进行组织，便于快速查找

集和包括可变和不可变集和，分别位于 `scala.collection.mntable` 和 `scala.collection.immutable` 包中，默认情况下是不可变集和。

```Scala
var mySet = Set("Hadoop", "Spark")
mySet += "Scala"
```
变量是可变的，集合不可变，加操作导致生成了一个新的集合。

可变集合，在原集合中增加一个元素。
```Scala
import scala.collection.mutable.Set

val myMutableSet = Set("BigData", "Spark")
myMutableSet += "Scala"
```

## 映射 Map
映射时一系列键值对的容器。也有可变和不可变两个版本，默认情况下是不可变的。

```Scala
val university = Map("XMU" -> "Xiamen University", "THU" -> "Tsinghua University", "PKU" -> "Peking University")
```

可变映射
```Scala
import scala.collection.mutable.Map

val university = Map("XMU" -> "Xiamen University", "THU" -> "Tsinghua University", "PKU" -> "Peking University")

university("XMU") = "Ximan University"	// update
university("FZU") = "Fuzhou University"	// add
```

### 遍历映射
```Scala
for ( (k,v) <- university) {
	// do something
}

for ( k <- university.keys ) {
	// do something
}

for ( v <- university.values ) {
	// do something
}
```

## 迭代器 Iterator
迭代器不是一个集合，而是一种访问集合的方法。

迭代器有两个基本操作：next 和 hasNext。

```Scala
val iter = Iterator("Hadoop", "Spark", "Scala")

while (iter.hasNext) {
	println(iter.next())
}

for (elem <- iter) {
	println(elem)
}
```

### grouped & sliding
grouped 返回元素的增量分块
```Scala
scala> val xs = List(1,2,3,4,5)
xs: List[Int] = List(1, 2, 3, 4, 5)

scala> val git = xs grouped 3
git: Iterator[List[Int]] = non-empty iterator

scala> git.next()
res0: List[Int] = List(1, 2, 3)

scala> git.next()
res1: List[Int] = List(4, 5)
```

sliding 生成一个滑动元素的窗口
```Scala
scala> val sit = xs sliding 3
sit: Iterator[List[Int]] = non-empty iterator

scala> sit.next()
res3: List[Int] = List(1, 2, 3)

scala> sit.next()
res4: List[Int] = List(2, 3, 4)

scala> sit.next()
res5: List[Int] = List(3, 4, 5)
```

## 数组 Array
是一种可变的、可索引的、元素具有相同类型的数据集合，Scala提供了类似于Java中泛型的机制指定数组类型。也可以不指定类型。

```Scala
val intValueArr = new Array[Int](3)
intValueArr(0) = 23
intValueArr(1) = 34
intValueArr(2) = 45

val strArr = Array("BigData", "Hadoop", "Spark")
```

### 多维数组
定义多维数组使用 `ofDim()` 方法
```Scala
val myMatrix = Array.ofDim[Int](3,4) // 三行四列
```

访问元素
```Scala
myMatrix(0)(1)
```

### 不定长数组
```Scala
import scala.collection.mutable.ArrayBuffer

val aMutableArr = ArrayBuffer(10, 20, 30)
aMutableArr += 40
println(aMutableArr)			// ArrayBuffer(10, 20, 30, 40)

aMutableArr.insert(2, 60, 40)
println(aMutableArr)			// ArrayBuffer(10, 20, 60, 40, 30, 40)

aMutableArr -= 40
println(aMutableArr)			// ArrayBuffer(10, 20, 60, 30, 40)

var temp = aMutableArr.remove(2)
println(aMutableArr)			// ArrayBuffer(10, 20, 30, 40)
```

## 元组 Tuple
不同类型的值的集合。
```Scala
val tuple = ("BigData", 2015, 45.0)
println(tuple._1)
println(tuple._2)
```

# 面向对象
## 类 

### 基本类结构
**简单类**
```Scala
class Counter {
	private var value = 0
	
	def increment(): Unit = {
		value += 1
	}
	
	def current(): Int = { value }
}
```

定义方法可以省略返回类型：
```Scala
class Counter {
	private var value = 0
	
	def increment() {
		value += 1
	}
	
	def current(): Int = { value }
}
```

**方法传参**
```Scala
class Counter {
	private var value = 0
	
	def increment(step: Int) {
		value += step
	}
	
	def current(): Int = { value }
}
```

**创建对象**
```Scala
val myCounter = new Counter
myCounter.increment()
println(myCounter.current())

val myCounter2 = new Counter()
```

### 编译&执行
如下定义的这样一个类在执行时，直接使用 scala 解释器执行即可，不需要编译。
```Scala
class Counter {
	private var value = 0
	
	def increment() {
		value += 1
	}
	
	def current(): Int = { value }
}
```
如果要编译，需要创建一个单例对象：
```Scala
class Counter {
	private var value = 0
	
	def increment() {
		value += 1
	}
	
	def current(): Int = { value }
}

object MyCounter {
	def main(args:Array[String]) {
		val myCounter = new Counter
		myCounter.increment()
		println(myCounter.current)
	}
}
```

**编译**（后面跟的是文件名）：
```
scalac counter.scala 
```
编译之后会产生一些文件：
```
Counter.class
MyCounter.class
MyCounter$.class
```

**执行**
执行时后面跟的是包含 `main` 方法的对象名称
```
scala -classpath . MyCounter
```

### getter & setter
```Scala
class Person {
	private var privateName = "dcwang"
	private var privateAge = 25
	
	// 
	def name = privateName

	def setName(newName : String) {
		privateName = newName
	}

	def age = privateAge

	def setAge(newAge : Int) {
		privateAge = newAge
	}

	def grown(step : Int) {
		privateAge += step
	}
}

object SomeOne {
	def main(args:Array[String]) {
		val someOne = new Person
		println(someOne.name)

		someOne.setName("yz")
		println(someOne.name)

		println(someOne.age)
		someOne.grown(2)
		println(someOne.age)
	}
}
```

### 构造器
Scala构造器包含一个主构造器和若干个辅构造器
辅助构造器的名称为 `this` ，每个辅助构造器都必须调用一个已有的构造器

```Scala
class Person {
	private var privateName = "dcwang"
	private var privateAge = 25
	
	def this(name : String) {
		this() // invoke main constructor
		this.privateName = name
	}

	def this(name : String, age : Int) {
		this(name)
		this.privateAge = age
	}

	def name = privateName

	def setName(newName : String) {
		privateName = newName
	}

	def age = privateAge

	def setAge(newAge : Int) {
		privateAge = newAge
	}

	def grown(step : Int) {
		privateAge += step
	}
}

object SomeOne {
	def main(args:Array[String]) {
		val someOne = new Person("yz", 18)
		println(someOne.name)
		println(someOne.age)
	}
}
```

## 对象

### 单例对象
```Scala
object Person {
	private var id = 0

	def newId() = {
		id += 1
		id
	}
}

println(Person.newId())
println(Person.newId())
println(Person.newId())
```

### 伴生对象
在 Java 中经常用到同时包含实例方法和静态方法的类，在 Scala 中可以用伴生对象来实现
类和它的伴生对象必须在同一个文件中，并且可以相互访问私有成员
当单例对象与某个类具有相同的名称时，它被成为这个类的伴生对象

```Scala
class Person {
	private val id = Person.newPersonId()
	private var name = ""

	def this(name : String) {
		this()
		this.name = name
	}

	def info() { println("The id of %s is %d. \n".format(name, id)) }
}

object Person {
	private var lastId = 0

	private def newPersonId() = {
		lastId += 1
		lastId
	}

	def main(args : Array[String]) {
		val person1 = new Person("yz")
		val person2 = new Person("yj")

		person1.info()
		person2.info()
	}
}
```

### applay 方法

### update 方法

## 继承

在子类中重写超类抽象方法时不需要使用 `override` 关键字
重写一个非抽象方法必须使用 `override` 修饰符
只有主构造器可以调用超类的主构造器
可以重写超类中的子段

### 抽象类
```Scala
// 抽象类，不能直接实例化
abstract class Car {
	// 抽象子段，不需要初始化
	val carBrand : String
	// 抽象方法，不需要使用 abstract 关键字
	def info()
	def greeting() { println("Welcome to my car!") }
}
```

### 继承抽象类
```Scala
class BMWCar extends Car {
	override val carBrand = "BMW"
	// 重写抽象方法不用加 override
	def info() {
		printf("This is a car")
	}

	// 重写非抽象方法必须使用 override
	override def greeting() {
		printf("something")
	}
}
```

## 特质（trait）
在 Scala 中没有接口的概念，而是提供了 trait，它实现了接口的功能，以及许多其他特性
trait 是 Scala 中代码重用的基本单元，可以同时拥有抽象方法和具体方法
在 Scala 中一个类只能继承一个超类，但是可以实现多个 trait，从而拥有 trait 中的方法和字段，实现多重继承。


```Scala
trait CarId {
	var id : Int
	def currentId() : Int
}
```

## 模式匹配
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTY0ODMxMTkxM119
-->