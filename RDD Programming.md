
# HBase 读写

# 环境准备

## 集群环境

**Hadoop环境**
参考：[Hadoop 集群部署方案](https://weilu2.github.io/2018/10/30/Hadoop%E9%9B%86%E7%BE%A4%E9%83%A8%E7%BD%B2%E6%96%B9%E6%A1%88/)

**Zookeeper&HBase环境**
参考：[基于 Hadoop 集群部署 ZooKeeper 和 HBase 集群](https://weilu2.github.io/2018/11/05/%E5%9F%BA%E4%BA%8E-Hadoop-%E9%9B%86%E7%BE%A4%E9%83%A8%E7%BD%B2-ZooKeeper-%E5%92%8C-HBase-%E9%9B%86%E7%BE%A4/)

**Spark环境**
参考：[基于 Hadoop 和 Yarn 集群部署 Spark 集群](https://weilu2.github.io/2018/11/15/%E5%9F%BA%E4%BA%8E-Hadoop-%E5%92%8C-Yarn-%E9%9B%86%E7%BE%A4%E9%83%A8%E7%BD%B2-Spark-%E9%9B%86%E7%BE%A4/)

## 运行环境
整个过程是在本地 Idea 中使用 Scala 编写 Spark 程序，使用 SBT 打包后，通过 `spark-submit` 提交到 Yarn 中运行。

之前在部署 Spark 环境时，有设置过一个变量：
```
spark.yarn.jars		hdfs://weilu131:9000/spark_jars/*
```
并且已经将 Spark 的相关 JAR 包上传到 HDFS 上的这个目录中。这是 Spark 程序的运行环境。由于需要连接 HBase，因此程序的运行环境还需要有 HBase 相关的 JAR 包。

将 HBase 根目录下的以下 JAR 包上传到 HDFS 的这个目录中：
```
/usr/local/hadoop-2.6.0-cdh5.15.0/bin/hdfs dfs -put /usr/local/hbase-1.2.0-cdh5.15.0/lib/hbase*.jar /spark_jars

/usr/local/hadoop-2.6.0-cdh5.15.0/bin/hdfs dfs -put /usr/local/hbase-1.2.0-cdh5.15.0/lib/guava-12.0.1.jar /spark_jars

/usr/local/hadoop-2.6.0-cdh5.15.0/bin/hdfs dfs -put /usr/local/hbase-1.2.0-cdh5.15.0/lib/htrace-core-3.2.0-incubating.jar /spark_jars

/usr/local/hadoop-2.6.0-cdh5.15.0/bin/hdfs dfs -put /usr/local/hbase-1.2.0-cdh5.15.0/lib/protobuf-java-2.5.0.jar /spark_jars

/usr/local/hadoop-2.6.0-cdh5.15.0/bin/hdfs dfs -put /usr/local/hbase-1.2.0-cdh5.15.0/lib/metrics-core-2.2.0.jar /spark_jars 
```

这里需要注意，由于使用的 HBase 和 Spark 各自有对一些 JAR 包的不同版本的依赖，因此需要都上传上去，如果没有可能会报找不到类的错误。在末尾会有相关示例。


# 数据准备
进入 HBase shell 环境：
```
/usr/local/hbase-1.2.0-cdh5.15.0/bin/hbase shell
```

## 建表
创建一张名为 `student` 的表，包含一个 `info` 列族。
```
create 'student','info'
```

## 添加数据
HBase 中添加数据是按照单元格添加，通过行键、列族、列名确定一个单元格。
```
put 'student','e018565c-ebaa-11e8-b4ec-3c970e0087f3','info:name','wcwang'
put 'student','e018565c-ebaa-11e8-b4ec-3c970e0087f3','info:gender','F'
put 'student','e018565c-ebaa-11e8-b4ec-3c970e0087f3','info:age','22'

put 'student','e0182f4a-ebaa-11e8-a353-3c970e0087f3','info:name','lx'
put 'student','e0182f4a-ebaa-11e8-a353-3c970e0087f3','info:gender','M'
put 'student','e0182f4a-ebaa-11e8-a353-3c970e0087f3','info:age','21'
```

# 编写运行程序
创建一个 Scala 项目，参考[配置 Intellij Idea 和 Sbt 开发、打包、运行 Spark 程序](https://weilu2.github.io/2018/11/16/%E9%85%8D%E7%BD%AE-Intellij-Idea-%E5%92%8C-Sbt-%E5%BC%80%E5%8F%91%E3%80%81%E6%89%93%E5%8C%85%E3%80%81%E8%BF%90%E8%A1%8C-Spark-%E7%A8%8B%E5%BA%8F/)。

## 代码

然后在 `src/main/scala/` 目录下创建一个 Scala 脚本，填充以下代码：

```Scala
import org.apache.spark._  
import org.apache.hadoop.hbase.HBaseConfiguration  
import org.apache.hadoop.hbase.mapreduce.TableInputFormat  
import org.apache.hadoop.hbase.util.Bytes  
  
object DataImport {  
  def main(args: Array[String]): Unit = {  
    val sparkConf = new SparkConf().setMaster("yarn").set("spark.app.name", "MedicalQA")  
    sparkConf.set("spark.serializer","org.apache.spark.serializer.KryoSerializer")  
    val sc = new SparkContext(sparkConf)  
  
    val hbaseConf = HBaseConfiguration.create()  
    hbaseConf.set("hbase.zookeeper.property.clientPort", "2181")  
    hbaseConf.set("hbase.zookeeper.quorum", "192.168.0.131,192.168.0.132,192.168.0.133,192.168.0.151,192.168.0.152")  
    hbaseConf.set("hbase.master", "192.168.0.131")  
    hbaseConf.set(TableInputFormat.INPUT_TABLE, "student")  
  
    val studRDD = sc.newAPIHadoopRDD(hbaseConf, classOf[TableInputFormat], classOf[org.apache.hadoop.hbase.io.ImmutableBytesWritable], classOf[org.apache.hadoop.hbase.client.Result])  
    val count = studRDD.count()  
    println(count)  
  
    studRDD.cache()  
    studRDD.collect().foreach({  
      row => {  
        val result = row._2  
        val key = Bytes.toString(result.getRow)  
        val name = Bytes.toString(result.getValue("info".getBytes(), "name".getBytes()))  
        val gender = Bytes.toString(result.getValue("info".getBytes(), "gender".getBytes()))  
        val age = Bytes.toString(result.getValue("info".getBytes(), "age".getBytes()))  
        println(key + "  " + name + "  " + gender + "  " + age)  
      }  
    })  
  }  
}
```
在这段代码中首先统计了 student 表中数据的条数，然后将两条数据分别打印出来。

## 项目配置
将 Hbase 下面的 `hbase-site.xml` 文件拷贝到项目中的 `src/main/scala` 目录中。

## 打包运行
打包好的 JAR 包上传到服务器中，然后提交到 Spark 中运行：
```
/usr/local/spark-2.4.0-bin-hadoop2.6/bin/spark-submit --deploy-mode cluster /home/workspace/MedicalQAImport.jar
```

如果命令行和 Yarn 中没有抛异常，那就OK。然后检查 Yarn 中的日志。
```
2
e0182f4a-ebaa-11e8-a353-3c970e0087f3  lx  M  21
e018565c-ebaa-11e8-b4ec-3c970e0087f3  wcwang  F  22
```
注意，这里输出的日志不是在控制台输出，而是在所运行的那台服务器的日志中，可以通过 Yarn 的界面查看。

# 遇到问题

## java.lang.NoClassDefFoundError: com/yammer/metrics/core/Gauge

这里就是之前所说的 HBase 和 Spark 依赖于同一个类库的不同版本的问题。在上传 Spark 的 JAR 包时，已经将 `metrics-core-3.1.5.jar` 上传了。但实际上 HBase 依赖的是 `metrics-core-2.2.0.jar` 的版本，因此还需要将这个包也上传。

**部分异常**
```
org.apache.hadoop.hbase.client.RetriesExhaustedException: Failed after attempts=36, exceptions:
Tue Nov 20 08:55:43 CST 2018, null, java.net.SocketTimeoutException: callTimeout=60000, callDuration=68474: row 'student,,00000000000000' on table 'hbase:meta' at region=hbase:meta,,1.1588230740, hostname=iflysse151,60020,1542616359146, seqNum=0
Caused by: java.net.SocketTimeoutException: callTimeout=60000, callDuration=68474: row 'student,,00000000000000' on table 'hbase:meta' at region=hbase:meta,,1.1588230740, hostname=iflysse151,60020,1542616359146, seqNum=0
	
Caused by: java.io.IOException: com.google.protobuf.ServiceException: java.lang.NoClassDefFoundError: com/yammer/metrics/core/Gauge

Caused by: com.google.protobuf.ServiceException: java.lang.NoClassDefFoundError: com/yammer/metrics/core/Gauge

Caused by: java.lang.NoClassDefFoundError: com/yammer/metrics/core/Gauge

Caused by: java.lang.ClassNotFoundException: com.yammer.metrics.core.Gauge
```

**完整异常信息**
```
18/11/20 08:55:44 ERROR ApplicationMaster: User class threw exception: org.apache.hadoop.hbase.client.RetriesExhaustedException: Failed after attempts=36, exceptions:
Tue Nov 20 08:55:43 CST 2018, null, java.net.SocketTimeoutException: callTimeout=60000, callDuration=68474: row 'student,,00000000000000' on table 'hbase:meta' at region=hbase:meta,,1.1588230740, hostname=iflysse151,60020,1542616359146, seqNum=0

org.apache.hadoop.hbase.client.RetriesExhaustedException: Failed after attempts=36, exceptions:
Tue Nov 20 08:55:43 CST 2018, null, java.net.SocketTimeoutException: callTimeout=60000, callDuration=68474: row 'student,,00000000000000' on table 'hbase:meta' at region=hbase:meta,,1.1588230740, hostname=iflysse151,60020,1542616359146, seqNum=0

	at org.apache.hadoop.hbase.client.RpcRetryingCallerWithReadReplicas.throwEnrichedException(RpcRetryingCallerWithReadReplicas.java:320)
	at org.apache.hadoop.hbase.client.ScannerCallableWithReplicas.call(ScannerCallableWithReplicas.java:247)
	at org.apache.hadoop.hbase.client.ScannerCallableWithReplicas.call(ScannerCallableWithReplicas.java:62)
	at org.apache.hadoop.hbase.client.RpcRetryingCaller.callWithoutRetries(RpcRetryingCaller.java:210)
	at org.apache.hadoop.hbase.client.ClientScanner.call(ClientScanner.java:327)
	at org.apache.hadoop.hbase.client.ClientScanner.nextScanner(ClientScanner.java:302)
	at org.apache.hadoop.hbase.client.ClientScanner.initializeScannerInConstruction(ClientScanner.java:167)
	at org.apache.hadoop.hbase.client.ClientScanner.<init>(ClientScanner.java:162)
	at org.apache.hadoop.hbase.client.HTable.getScanner(HTable.java:867)
	at org.apache.hadoop.hbase.client.MetaScanner.metaScan(MetaScanner.java:193)
	at org.apache.hadoop.hbase.client.MetaScanner.metaScan(MetaScanner.java:89)
	at org.apache.hadoop.hbase.client.MetaScanner.allTableRegions(MetaScanner.java:324)
	at org.apache.hadoop.hbase.client.HRegionLocator.getAllRegionLocations(HRegionLocator.java:88)
	at org.apache.hadoop.hbase.util.RegionSizeCalculator.init(RegionSizeCalculator.java:94)
	at org.apache.hadoop.hbase.util.RegionSizeCalculator.<init>(RegionSizeCalculator.java:81)
	at org.apache.hadoop.hbase.mapreduce.TableInputFormatBase.getSplits(TableInputFormatBase.java:256)
	at org.apache.hadoop.hbase.mapreduce.TableInputFormat.getSplits(TableInputFormat.java:240)
	at org.apache.spark.rdd.NewHadoopRDD.getPartitions(NewHadoopRDD.scala:130)
	at org.apache.spark.rdd.RDD$$anonfun$partitions$2.apply(RDD.scala:253)
	at org.apache.spark.rdd.RDD$$anonfun$partitions$2.apply(RDD.scala:251)
	at scala.Option.getOrElse(Option.scala:121)
	at org.apache.spark.rdd.RDD.partitions(RDD.scala:251)
	at org.apache.spark.SparkContext.runJob(SparkContext.scala:2126)
	at org.apache.spark.rdd.RDD.count(RDD.scala:1168)
	at DataImport$.main(DataImport.scala:18)
	at DataImport.main(DataImport.scala)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.spark.deploy.yarn.ApplicationMaster$$anon$2.run(ApplicationMaster.scala:678)
Caused by: java.net.SocketTimeoutException: callTimeout=60000, callDuration=68474: row 'student,,00000000000000' on table 'hbase:meta' at region=hbase:meta,,1.1588230740, hostname=iflysse151,60020,1542616359146, seqNum=0
	at org.apache.hadoop.hbase.client.RpcRetryingCaller.callWithRetries(RpcRetryingCaller.java:169)
	at org.apache.hadoop.hbase.client.ResultBoundedCompletionService$QueueingFuture.run(ResultBoundedCompletionService.java:80)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
Caused by: java.io.IOException: com.google.protobuf.ServiceException: java.lang.NoClassDefFoundError: com/yammer/metrics/core/Gauge
	at org.apache.hadoop.hbase.protobuf.ProtobufUtil.getRemoteException(ProtobufUtil.java:334)
	at org.apache.hadoop.hbase.client.ScannerCallable.openScanner(ScannerCallable.java:408)
	at org.apache.hadoop.hbase.client.ScannerCallable.call(ScannerCallable.java:204)
	at org.apache.hadoop.hbase.client.ScannerCallable.call(ScannerCallable.java:65)
	at org.apache.hadoop.hbase.client.RpcRetryingCaller.callWithoutRetries(RpcRetryingCaller.java:210)
	at org.apache.hadoop.hbase.client.ScannerCallableWithReplicas$RetryingRPC.call(ScannerCallableWithReplicas.java:397)
	at org.apache.hadoop.hbase.client.ScannerCallableWithReplicas$RetryingRPC.call(ScannerCallableWithReplicas.java:371)
	at org.apache.hadoop.hbase.client.RpcRetryingCaller.callWithRetries(RpcRetryingCaller.java:136)
	... 4 more
Caused by: com.google.protobuf.ServiceException: java.lang.NoClassDefFoundError: com/yammer/metrics/core/Gauge
	at org.apache.hadoop.hbase.ipc.AbstractRpcClient.callBlockingMethod(AbstractRpcClient.java:240)
	at org.apache.hadoop.hbase.ipc.AbstractRpcClient$BlockingRpcChannelImplementation.callBlockingMethod(AbstractRpcClient.java:336)
	at org.apache.hadoop.hbase.protobuf.generated.ClientProtos$ClientService$BlockingStub.scan(ClientProtos.java:34094)
	at org.apache.hadoop.hbase.client.ScannerCallable.openScanner(ScannerCallable.java:400)
	... 10 more
Caused by: java.lang.NoClassDefFoundError: com/yammer/metrics/core/Gauge
	at org.apache.hadoop.hbase.ipc.AbstractRpcClient.callBlockingMethod(AbstractRpcClient.java:225)
	... 13 more
Caused by: java.lang.ClassNotFoundException: com.yammer.metrics.core.Gauge
	at java.net.URLClassLoader.findClass(URLClassLoader.java:382)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	... 14 more
18/11/20 08:55:44 INFO ApplicationMaster: Final app status: FAILED, exitCode: 15, (reason: User class threw exception: org.apache.hadoop.hbase.client.RetriesExhaustedException: Failed after attempts=36, exceptions:
Tue Nov 20 08:55:43 CST 2018, null, java.net.SocketTimeoutException: callTimeout=60000, callDuration=68474: row 'student,,00000000000000' on table 'hbase:meta' at region=hbase:meta,,1.1588230740, hostname=iflysse151,60020,1542616359146, seqNum=0

	at org.apache.hadoop.hbase.client.RpcRetryingCallerWithReadReplicas.throwEnrichedException(RpcRetryingCallerWithReadReplicas.java:320)
	at org.apache.hadoop.hbase.client.ScannerCallableWithReplicas.call(ScannerCallableWithReplicas.java:247)
	at org.apache.hadoop.hbase.client.ScannerCallableWithReplicas.call(ScannerCallableWithReplicas.java:62)
	at org.apache.hadoop.hbase.client.RpcRetryingCaller.callWithoutRetries(RpcRetryingCaller.java:210)
	at org.apache.hadoop.hbase.client.ClientScanner.call(ClientScanner.java:327)
	at org.apache.hadoop.hbase.client.ClientScanner.nextScanner(ClientScanner.java:302)
	at org.apache.hadoop.hbase.client.ClientScanner.initializeScannerInConstruction(ClientScanner.java:167)
	at org.apache.hadoop.hbase.client.ClientScanner.<init>(ClientScanner.java:162)
	at org.apache.hadoop.hbase.client.HTable.getScanner(HTable.java:867)
	at org.apache.hadoop.hbase.client.MetaScanner.metaScan(MetaScanner.java:193)
	at org.apache.hadoop.hbase.client.MetaScanner.metaScan(MetaScanner.java:89)
	at org.apache.hadoop.hbase.client.MetaScanner.allTableRegions(MetaScanner.java:324)
	at org.apache.hadoop.hbase.client.HRegionLocator.getAllRegionLocations(HRegionLocator.java:88)
	at org.apache.hadoop.hbase.util.RegionSizeCalculator.init(RegionSizeCalculator.java:94)
	at org.apache.hadoop.hbase.util.RegionSizeCalculator.<init>(RegionSizeCalculator.java:81)
	at org.apache.hadoop.hbase.mapreduce.TableInputFormatBase.getSplits(TableInputFormatBase.java:256)
	at org.apache.hadoop.hbase.mapreduce.TableInputFormat.getSplits(TableInputFormat.java:240)
	at org.apache.spark.rdd.NewHadoopRDD.getPartitions(NewHadoopRDD.scala:130)
	at org.apache.spark.rdd.RDD$$anonfun$partitions$2.apply(RDD.scala:253)
	at org.apache.spark.rdd.RDD$$anonfun$partitions$2.apply(RDD.scala:251)
	at scala.Option.getOrElse(Option.scala:121)
	at org.apache.spark.rdd.RDD.partitions(RDD.scala:251)
	at org.apache.spark.SparkContext.runJob(SparkContext.scala:2126)
	at org.apache.spark.rdd.RDD.count(RDD.scala:1168)
	at DataImport$.main(DataImport.scala:18)
	at DataImport.main(DataImport.scala)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.spark.deploy.yarn.ApplicationMaster$$anon$2.run(ApplicationMaster.scala:678)
Caused by: java.net.SocketTimeoutException: callTimeout=60000, callDuration=68474: row 'student,,00000000000000' on table 'hbase:meta' at region=hbase:meta,,1.1588230740, hostname=iflysse151,60020,1542616359146, seqNum=0
	at org.apache.hadoop.hbase.client.RpcRetryingCaller.callWithRetries(RpcRetryingCaller.java:169)
	at org.apache.hadoop.hbase.client.ResultBoundedCompletionService$QueueingFuture.run(ResultBoundedCompletionService.java:80)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
Caused by: java.io.IOException: com.google.protobuf.ServiceException: java.lang.NoClassDefFoundError: com/yammer/metrics/core/Gauge
	at org.apache.hadoop.hbase.protobuf.ProtobufUtil.getRemoteException(ProtobufUtil.java:334)
	at org.apache.hadoop.hbase.client.ScannerCallable.openScanner(ScannerCallable.java:408)
	at org.apache.hadoop.hbase.client.ScannerCallable.call(ScannerCallable.java:204)
	at org.apache.hadoop.hbase.client.ScannerCallable.call(ScannerCallable.java:65)
	at org.apache.hadoop.hbase.client.RpcRetryingCaller.callWithoutRetries(RpcRetryingCaller.java:210)
	at org.apache.hadoop.hbase.client.ScannerCallableWithReplicas$RetryingRPC.call(ScannerCallableWithReplicas.java:397)
	at org.apache.hadoop.hbase.client.ScannerCallableWithReplicas$RetryingRPC.call(ScannerCallableWithReplicas.java:371)
	at org.apache.hadoop.hbase.client.RpcRetryingCaller.callWithRetries(RpcRetryingCaller.java:136)
	... 4 more
Caused by: com.google.protobuf.ServiceException: java.lang.NoClassDefFoundError: com/yammer/metrics/core/Gauge
	at org.apache.hadoop.hbase.ipc.AbstractRpcClient.callBlockingMethod(AbstractRpcClient.java:240)
	at org.apache.hadoop.hbase.ipc.AbstractRpcClient$BlockingRpcChannelImplementation.callBlockingMethod(AbstractRpcClient.java:336)
	at org.apache.hadoop.hbase.protobuf.generated.ClientProtos$ClientService$BlockingStub.scan(ClientProtos.java:34094)
	at org.apache.hadoop.hbase.client.ScannerCallable.openScanner(ScannerCallable.java:400)
	... 10 more
Caused by: java.lang.NoClassDefFoundError: com/yammer/metrics/core/Gauge
	at org.apache.hadoop.hbase.ipc.AbstractRpcClient.callBlockingMethod(AbstractRpcClient.java:225)
	... 13 more
Caused by: java.lang.ClassNotFoundException: com.yammer.metrics.core.Gauge
	at java.net.URLClassLoader.findClass(URLClassLoader.java:382)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	... 14 more
)
18/11/20 08:55:44 INFO SparkContext: Invoking stop() from shutdown hook
```

## had a not serializable result

**解决**
在 Spark 的环境中配置：
```Scala
sparkConf.set("spark.serializer","org.apache.spark.serializer.KryoSerializer")  
```

**异常信息**
```
18/11/20 09:55:25 ERROR TaskSetManager: Task 0.0 in stage 1.0 (TID 1) had a not serializable result: org.apache.hadoop.hbase.io.ImmutableBytesWritable
Serialization stack:
	- object not serializable (class: org.apache.hadoop.hbase.io.ImmutableBytesWritable, value: 65 30 31 38 35 36 35 63 2d 65 62 61 61 2d 31 31 65 38 2d 62 34 65 63 2d 33 63 39 37 30 65 30 30 38 37 66 33)
	- field (class: scala.Tuple2, name: _1, type: class java.lang.Object)
	- object (class scala.Tuple2, (65 30 31 38 35 36 35 63 2d 65 62 61 61 2d 31 31 65 38 2d 62 34 65 63 2d 33 63 39 37 30 65 30 30 38 37 66 33,keyvalues={e0182f4a-ebaa-11e8-a353-3c970e0087f3/info:age/1542606358169/Put/vlen=2/seqid=0, e0182f4a-ebaa-11e8-a353-3c970e0087f3/info:gender/1542606352584/Put/vlen=1/seqid=0, e0182f4a-ebaa-11e8-a353-3c970e0087f3/info:name/1542606346782/Put/vlen=2/seqid=0}))
	- element of array (index: 0)
	- array (class [Lscala.Tuple2;, size 2); not retrying
18/11/20 09:55:25 INFO YarnClusterScheduler: Removed TaskSet 1.0, whose tasks have all completed, from pool 
18/11/20 09:55:25 INFO YarnClusterScheduler: Cancelling stage 1
18/11/20 09:55:25 INFO YarnClusterScheduler: Killing all running tasks in stage 1: Stage cancelled
18/11/20 09:55:25 INFO DAGScheduler: ResultStage 1 (collect at DataImport.scala:23) failed in 0.338 s due to Job aborted due to stage failure: Task 0.0 in stage 1.0 (TID 1) had a not serializable result: org.apache.hadoop.hbase.io.ImmutableBytesWritable
Serialization stack:
	- object not serializable (class: org.apache.hadoop.hbase.io.ImmutableBytesWritable, value: 65 30 31 38 35 36 35 63 2d 65 62 61 61 2d 31 31 65 38 2d 62 34 65 63 2d 33 63 39 37 30 65 30 30 38 37 66 33)
	- field (class: scala.Tuple2, name: _1, type: class java.lang.Object)
	- object (class scala.Tuple2, (65 30 31 38 35 36 35 63 2d 65 62 61 61 2d 31 31 65 38 2d 62 34 65 63 2d 33 63 39 37 30 65 30 30 38 37 66 33,keyvalues={e0182f4a-ebaa-11e8-a353-3c970e0087f3/info:age/1542606358169/Put/vlen=2/seqid=0, e0182f4a-ebaa-11e8-a353-3c970e0087f3/info:gender/1542606352584/Put/vlen=1/seqid=0, e0182f4a-ebaa-11e8-a353-3c970e0087f3/info:name/1542606346782/Put/vlen=2/seqid=0}))
	- element of array (index: 0)
	- array (class [Lscala.Tuple2;, size 2)
18/11/20 09:55:25 INFO DAGScheduler: Job 1 failed: collect at DataImport.scala:23, took 0.346124 s
18/11/20 09:55:25 ERROR ApplicationMaster: User class threw exception: org.apache.spark.SparkException: Job aborted due to stage failure: Task 0.0 in stage 1.0 (TID 1) had a not serializable result: org.apache.hadoop.hbase.io.ImmutableBytesWritable
Serialization stack:
	- object not serializable (class: org.apache.hadoop.hbase.io.ImmutableBytesWritable, value: 65 30 31 38 35 36 35 63 2d 65 62 61 61 2d 31 31 65 38 2d 62 34 65 63 2d 33 63 39 37 30 65 30 30 38 37 66 33)
	- field (class: scala.Tuple2, name: _1, type: class java.lang.Object)
	- object (class scala.Tuple2, (65 30 31 38 35 36 35 63 2d 65 62 61 61 2d 31 31 65 38 2d 62 34 65 63 2d 33 63 39 37 30 65 30 30 38 37 66 33,keyvalues={e0182f4a-ebaa-11e8-a353-3c970e0087f3/info:age/1542606358169/Put/vlen=2/seqid=0, e0182f4a-ebaa-11e8-a353-3c970e0087f3/info:gender/1542606352584/Put/vlen=1/seqid=0, e0182f4a-ebaa-11e8-a353-3c970e0087f3/info:name/1542606346782/Put/vlen=2/seqid=0}))
	- element of array (index: 0)
	- array (class [Lscala.Tuple2;, size 2)
org.apache.spark.SparkException: Job aborted due to stage failure: Task 0.0 in stage 1.0 (TID 1) had a not serializable result: org.apache.hadoop.hbase.io.ImmutableBytesWritable
Serialization stack:
	- object not serializable (class: org.apache.hadoop.hbase.io.ImmutableBytesWritable, value: 65 30 31 38 35 36 35 63 2d 65 62 61 61 2d 31 31 65 38 2d 62 34 65 63 2d 33 63 39 37 30 65 30 30 38 37 66 33)
	- field (class: scala.Tuple2, name: _1, type: class java.lang.Object)
	- object (class scala.Tuple2, (65 30 31 38 35 36 35 63 2d 65 62 61 61 2d 31 31 65 38 2d 62 34 65 63 2d 33 63 39 37 30 65 30 30 38 37 66 33,keyvalues={e0182f4a-ebaa-11e8-a353-3c970e0087f3/info:age/1542606358169/Put/vlen=2/seqid=0, e0182f4a-ebaa-11e8-a353-3c970e0087f3/info:gender/1542606352584/Put/vlen=1/seqid=0, e0182f4a-ebaa-11e8-a353-3c970e0087f3/info:name/1542606346782/Put/vlen=2/seqid=0}))
	- element of array (index: 0)
	- array (class [Lscala.Tuple2;, size 2)
	at org.apache.spark.scheduler.DAGScheduler.org$apache$spark$scheduler$DAGScheduler$$failJobAndIndependentStages(DAGScheduler.scala:1887)
	at org.apache.spark.scheduler.DAGScheduler$$anonfun$abortStage$1.apply(DAGScheduler.scala:1875)
	at org.apache.spark.scheduler.DAGScheduler$$anonfun$abortStage$1.apply(DAGScheduler.scala:1874)
	at scala.collection.mutable.ResizableArray$class.foreach(ResizableArray.scala:59)
	at scala.collection.mutable.ArrayBuffer.foreach(ArrayBuffer.scala:48)
	at org.apache.spark.scheduler.DAGScheduler.abortStage(DAGScheduler.scala:1874)
	at org.apache.spark.scheduler.DAGScheduler$$anonfun$handleTaskSetFailed$1.apply(DAGScheduler.scala:926)
	at org.apache.spark.scheduler.DAGScheduler$$anonfun$handleTaskSetFailed$1.apply(DAGScheduler.scala:926)
	at scala.Option.foreach(Option.scala:257)
	at org.apache.spark.scheduler.DAGScheduler.handleTaskSetFailed(DAGScheduler.scala:926)
	at org.apache.spark.scheduler.DAGSchedulerEventProcessLoop.doOnReceive(DAGScheduler.scala:2108)
	at org.apache.spark.scheduler.DAGSchedulerEventProcessLoop.onReceive(DAGScheduler.scala:2057)
	at org.apache.spark.scheduler.DAGSchedulerEventProcessLoop.onReceive(DAGScheduler.scala:2046)
	at org.apache.spark.util.EventLoop$$anon$1.run(EventLoop.scala:49)
	at org.apache.spark.scheduler.DAGScheduler.runJob(DAGScheduler.scala:737)
	at org.apache.spark.SparkContext.runJob(SparkContext.scala:2061)
	at org.apache.spark.SparkContext.runJob(SparkContext.scala:2082)
	at org.apache.spark.SparkContext.runJob(SparkContext.scala:2101)
	at org.apache.spark.SparkContext.runJob(SparkContext.scala:2126)
	at org.apache.spark.rdd.RDD$$anonfun$collect$1.apply(RDD.scala:945)
	at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:151)
	at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:112)
	at org.apache.spark.rdd.RDD.withScope(RDD.scala:363)
	at org.apache.spark.rdd.RDD.collect(RDD.scala:944)
	at DataImport$.main(DataImport.scala:23)
	at DataImport.main(DataImport.scala)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.spark.deploy.yarn.ApplicationMaster$$anon$2.run(ApplicationMaster.scala:678)
18/11/20 09:55:25 INFO ApplicationMaster: Final app status: FAILED, exitCode: 15, (reason: User class threw exception: org.apache.spark.SparkException: Job aborted due to stage failure: Task 0.0 in stage 1.0 (TID 1) had a not serializable result: org.apache.hadoop.hbase.io.ImmutableBytesWritable
Serialization stack:
	- object not serializable (class: org.apache.hadoop.hbase.io.ImmutableBytesWritable, value: 65 30 31 38 35 36 35 63 2d 65 62 61 61 2d 31 31 65 38 2d 62 34 65 63 2d 33 63 39 37 30 65 30 30 38 37 66 33)
	- field (class: scala.Tuple2, name: _1, type: class java.lang.Object)
	- object (class scala.Tuple2, (65 30 31 38 35 36 35 63 2d 65 62 61 61 2d 31 31 65 38 2d 62 34 65 63 2d 33 63 39 37 30 65 30 30 38 37 66 33,keyvalues={e0182f4a-ebaa-11e8-a353-3c970e0087f3/info:age/1542606358169/Put/vlen=2/seqid=0, e0182f4a-ebaa-11e8-a353-3c970e0087f3/info:gender/1542606352584/Put/vlen=1/seqid=0, e0182f4a-ebaa-11e8-a353-3c970e0087f3/info:name/1542606346782/Put/vlen=2/seqid=0}))
	- element of array (index: 0)
	- array (class [Lscala.Tuple2;, size 2)
	at org.apache.spark.scheduler.DAGScheduler.org$apache$spark$scheduler$DAGScheduler$$failJobAndIndependentStages(DAGScheduler.scala:1887)
	at org.apache.spark.scheduler.DAGScheduler$$anonfun$abortStage$1.apply(DAGScheduler.scala:1875)
	at org.apache.spark.scheduler.DAGScheduler$$anonfun$abortStage$1.apply(DAGScheduler.scala:1874)
	at scala.collection.mutable.ResizableArray$class.foreach(ResizableArray.scala:59)
	at scala.collection.mutable.ArrayBuffer.foreach(ArrayBuffer.scala:48)
	at org.apache.spark.scheduler.DAGScheduler.abortStage(DAGScheduler.scala:1874)
	at org.apache.spark.scheduler.DAGScheduler$$anonfun$handleTaskSetFailed$1.apply(DAGScheduler.scala:926)
	at org.apache.spark.scheduler.DAGScheduler$$anonfun$handleTaskSetFailed$1.apply(DAGScheduler.scala:926)
	at scala.Option.foreach(Option.scala:257)
	at org.apache.spark.scheduler.DAGScheduler.handleTaskSetFailed(DAGScheduler.scala:926)
	at org.apache.spark.scheduler.DAGSchedulerEventProcessLoop.doOnReceive(DAGScheduler.scala:2108)
	at org.apache.spark.scheduler.DAGSchedulerEventProcessLoop.onReceive(DAGScheduler.scala:2057)
	at org.apache.spark.scheduler.DAGSchedulerEventProcessLoop.onReceive(DAGScheduler.scala:2046)
	at org.apache.spark.util.EventLoop$$anon$1.run(EventLoop.scala:49)
	at org.apache.spark.scheduler.DAGScheduler.runJob(DAGScheduler.scala:737)
	at org.apache.spark.SparkContext.runJob(SparkContext.scala:2061)
	at org.apache.spark.SparkContext.runJob(SparkContext.scala:2082)
	at org.apache.spark.SparkContext.runJob(SparkContext.scala:2101)
	at org.apache.spark.SparkContext.runJob(SparkContext.scala:2126)
	at org.apache.spark.rdd.RDD$$anonfun$collect$1.apply(RDD.scala:945)
	at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:151)
	at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:112)
	at org.apache.spark.rdd.RDD.withScope(RDD.scala:363)
	at org.apache.spark.rdd.RDD.collect(RDD.scala:944)
	at DataImport$.main(DataImport.scala:23)
	at DataImport.main(DataImport.scala)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.spark.deploy.yarn.ApplicationMaster$$anon$2.run(ApplicationMaster.scala:678)
)
```

参考：https://segmentfault.com/q/1010000007041500
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTQyMTc3MTA0MywxMDk1OTMzNDk2LC0zMD
g0MTU0NjcsMTQxNjcyNjAwMF19
-->