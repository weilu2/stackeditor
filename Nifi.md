
# 

## 特质

## 模式匹配RDD 基本操作

## RDD 创建

**从本地文件系统加载数据**

```Scala
val lines = sc.textFile("file:///home/spark/mydata/word.txt")
```

**从分布式文件系统 HDFS 中加载数据**
```Scala
val lines = sc.textFile("hdfs://weilu131:9000/mydata/word.txt")
```

**从集合中创建 RDD**
使用 `sc.parallelize()` 方法可以将数组转换为 RDD
```Scala
val array = Array(1, 2, 3, 4, 5)
val rdd = sc.parallelize(array)
```
对于列表 `List` 同上。

## RDD 操作

### 转换操作（Transformation）

**filter(func)**
筛选出能够满足函数 func 的元素，并返回一个新的数据集。
```Scala
val lines = sc.textFile("hdfs://weilu131:9000/mydata/word.txt")
val res = lines.filter(line => line.contains("Spark")).count()
```

**map(func)**
将每个元素传递到函数func中，并将结果返回为一个新的数据集
```Scala
val lines = sc.textFile("hdfs://weilu131:9000/mydata/word.txt")
val res = lines.map(line => line.split(" ").size).reduce((a,b) => if (a > b) a else b)
```

**flatMap(func)**
与 map 类似，但每个输入元素都可以映射到0或多个输出结果

**groupByKey()**
应用于(K,V)键值对数据集时，返回一个新的 (K, Iterable) 形式的数据集

**reduceByKey(func)**
应用于 (K, V) 键值对的数据集时，返回一个新的 (K, V) 形式的数据集，其中的每个值是将每个 key 传递到函数 func 中进行聚合。

### 行动操作（Action）

**count()**
返回数据集中的元素个数

**collect()**
以数组的形式返回数据集中的所有元素

**first()**
返回数据集中第一个元素

**take(n)**
以数组的形式返回数据集中的前 n 个元素

**reduce(func)**
通过函数 func（输入两个参数并返回一个值）聚合函数集中的元素

**foreach(func)**
将数据集中的每个元素传递到函数 func 中运行



### 惰性机制
对于 RDD 而言，每一次转换都会形成新的 RDD，但是在转换操作过程中，只会记录轨迹，只有程序运行到行动操作时，才会真正的进行计算，这个被称为惰性求值。


### 持久化
```Scala
val list = List("Hadoop", "Spark", "Hive")
val rdd = sc.parallelize(list)

// 行动操作，触发一次计算
rdd.count()

// 行动操作，再次触发一次计算
rdd.collect().mkString(",")
```
在两次行动操作中每次触发的转换操作都是相同的，为了避免重复计算，可以对第一次转换的过程进行持久化。

**persist(MEMORY_ONLY)**
将 RDD 作为反序列化对象存储于 JVM 中，如果内存不足，按照 LRU 原则替换缓存中内容。

**persist(MEMORY_AND_DISK)**
将 RDD 作为反序列化的对象存储在 JVM 中，如果内存不足，超出部分会被存储在硬盘上

**cache()**
persist(MEMORY_ONLY) 的快捷方式

**unpersist()**
手动把持久化的 RDD 从缓存中删除

```Scala
val list = List("Hadoop", "Spark", "Hive")
val rdd = sc.parallelize(list)

// 标记为持久化
rdd.cache()

// 行动操作，触发一次计算，并缓存转换操作结果
rdd.count()

// 行动操作，直接使用缓存的转换操作结果
rdd.collect().mkString(",")
```

### 分区
RDD 分区的一个分区原则是使得分区的个数尽量等于整个集群中的CPU核心数目
对于不同的 spark 部署模式而言，都可以使用 spark.default.parallelism 这个参数设置

在调用 textFile 和 parallelize 方法时候手动指定分区个数即可。
- 对于 parallelize 而言，如果没有在方法中指定分区数，则默认为 spake.deafault.parallelism。
- 对于textFile 而言，如果没有在方法中指定分区数，则默认为 min(defaultParallelism, 2)，其中，defaultParallelism 对应的就是 spark.default.parallelism
- 如果时从 HDFS 中读取文件，则分区数为文件分片数（比如，128MB/片）

**textFile**

```Scala
sc.textFile(path, partitionNum)
```

**parallelize**

```
sc.parallelize(array, 2) // 设置两个分区
```

通过转换操作得到新的 RDD 时，直接调用 reparation 方法

**自定义分区**
```Scala
import org.apache.spark.{Partitioner, SparkContext, SparkConf}  
  
class UserPartitioner(numParts: Int) extends Partitioner {  
    override def numPartitions: Int = numParts  
  
    override def getPartition(key: Any): Int = {  
        key.toString.toInt % 10  
  }  
}  
  
object ManualPartition {  
    def main(args: Array[String]): Unit = {  
        val conf = new SparkConf()  
        val sc = new SparkContext(conf)  
  
        val data = sc.parallelize(1 to 5, 5)  
        val data2 = data.map((_, 1))  
        val data3 = data2.partitionBy(new UserPartitioner(10))  
        val data4 = data3.map(_._1)  
        data4.saveAsTextFile("hdfs://weilu131:9000/test/output")  
    }  
}
```

# Pair RDD（键值对 RDD） 

## 创建 PairRDD
```Scala
val list = List("Hadoop", "Hive", "HBase", "Spark", "Sqoop", "Spark")
val rdd = sc.parallelize(list)	// 创建 RDD
val pairRDD = rdd.map(word => (word, 1))

// 如果是在集群上运行 Spark 程序，那么这段代码不会打印任何内容
pairRDD.foreach(println)
// 需要先收集之后再打印
pairRDD.collect().foreach(println)
```
打印内容：
```
(Hadoop,1)
(Hive,1)
(HBase,1)
(Spark,1)
(Sqoop,1)
(Spark,1)
```

## reduceByKey(func)
```Scala
val list = List("Hadoop", "Hive", "HBase", "Spark", "Sqoop", "Spark")
val pairRDD = sc.parallelize(list).map(word => (word, 1))
pairRDD.reduceByKey((a,b) => a+b).collect().foreach(println)
```
打印结果：
```
(Hive,1)
(Sqoop,1)
(Spark,2)
(HBase,1)
(Hadoop,1)
```

## groupByKey()
key 相同，将值生成一个列表
```Scala
val list = List("Hadoop 2", "Spark 3", "HBase 5", "Spark 6", "Hadoop 1")
val rdd = sc.parallelize(list)

def split(value: String): {
	val pair = value.split(" ")
	(pair[0], pair[1])
}
rdd.map((item: String) => (pair = item.split(" "); (pair[0], pair[1]))).collect().foreach(println)

pairRDD.reduceByKey((a,b) => a+b).collect().foreach(println)
```
打印结果：
```
(Hive,1)
(Sqoop,1)
(Spark,2)
(HBase,1)
(Hadoop,1)
```

## values

## mapValues(func)

## keys

## sortByKey()

## join

## combineByKey

# 共享变量

# 数据读写


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTU0NTQ4NzIyNSwtMjY0OTc3OTJdfQ==
-->