
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
# 基本概念和架构

## 基本概念
### 1. RDD
Resillient Distributed Dataset 弹性分布式数据集，是分布式内存的一个抽象概念，提供了一种高度受限的共享内存模型

### 2. DAG
Directed Acyclic Graph 有向无环图，反应 RDD 之间的依赖关系

### 3. Executor
运行在工作节点（WorkerNode）上的一个进程，负责运行 Task

### 4. Application
用户编写的 Spark 应用程序

### 5. Task
运行在 Execotr 上的工作单元

### 6. Job
一个 Job 包含多个 RDD 及作用于相应 RDD 上的各种操作

### 7. Stage
是 Job 的基本调度单位，一个 Job 会分为多组 Task，每组 Task 成为 Stage，或者 TaskSet，代表一组关联的，相互之间没有 Shuffle 依赖关系的任务组成的任务集

## 运行架构

![运行架构图](/A01.png)

Spark有点：
1、利用多线程执行具体的任务，减少任务启动开销
2、Executor 中有一个 BlockManager 存储模块，结合内存和磁盘作为存储设备，减少 IO 开销

## Spark 运行流程

![运行流程图](/A02.png)

### STEP 1
- 为应用构建基本的运行环境，由 Driver 创建一个 SparkContext 进行资源的申请、任务的分配和监控。

### STEP 2
- 资源管理器为 Executor 分配资源，并启动 Executor 进程。

### STEP 3
- SparkContext 根据 RDD 的依赖关系构建 DAG 图，DAG 图提交给 DAGScheduler 解析成 Stage，然后把一个个 TaskSet 提交给底层调度器 TaskSchedule 处理。

- Executor 向 SprakContext 申请 Task。

- TaskScheduler 将 Task 分发给 Executor ，并提供应用程序代码

### STEP 4
- Task 在 Excutor 上运行，把执行结果反馈给 TaskScheduler，然后反馈给 DAGScheduler，运行完成之后写入数据并释放资源。

## Spark 运行特点
1、每个 Application 都有自己专属的 Executor 进程，并且该进程在 Application 运行期间一直驻留。Executor进程以多线程的方式运行 Task。
2、Spark 运行过程与资源管理器无关，只要能够获取 Executor 进程并保持通讯即可
3、Task 采用了数据本地性和推测执行优化机制

# RDD

## RDD 执行过程
1、RDD 读取外部数据进行创建

2、经过一系列转换（Transformation）操作，每次都会产生新的 RDD，提供给下一次转换操作使用

3、最后一个 RDD 经过“动作”操作进行转换，并输出到外部数据源

一般讲一个 DAG 的一系列处理成为一个 Lineage（血缘关系）

## RDD 的依赖关系

### 窄依赖

1、一个父亲 RDD 的一个分区，转换得到一个儿子 RDD 的一个分区
2、多个父亲 RDD 的若干个分区，转换得到一个儿子 RDD 的一个分区

### 宽依赖
1、一个父亲 RDD 的一个分区，转换得到多个儿子 RDD 的若干个分区

### Stage 划分

DAG 中进行反向解析，遇到宽依赖就断开，遇到债依赖就把当前 RDD 加入到 Stage 中。将窄依赖尽量划分在同一个 Stage 中，实现流水线计算。

![Stage划分](/A03.png)


# RDD 基本操作

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
key 相同，将值按照传入逻辑计算
```Scala
val list = List("Hadoop 2", "Spark 3", "HBase 5", "Spark 6", "Hadoop 1")
val rdd = sc.parallelize(list)

val split = (line : String) => {
    val res = line.split(" ")
    (res(0), res(1).toInt)
}

val pairRdd = rdd.map(split)
pairRdd.collect().foreach(println) // 打印测试:1

val res = pairRdd.reduceByKey((a,b) => a+b)
res.collect().foreach(println)		// 打印测试：2
```
打印结果：
```
// 第一次
(Hadoop,2)
(Spark,3)
(HBase,5)
(Spark,6)
(Hadoop,1)

// 第二次
(Spark,9)
(HBase,5)
(Hadoop,3)
```

## groupByKey()
key 相同，将值生成一个列表
```Scala
val list = List("Hadoop 2", "Spark 3", "HBase 5", "Spark 6", "Hadoop 1")
val rdd = sc.parallelize(list)

val split = (line : String) => {
    val res = line.split(" ")
    (res(0), res(1).toInt)
}

val pairRdd = rdd.map(split)
pairRdd.collect().foreach(println) // 打印测试:1

val res = pairRdd.groupByKey()
res.collect().foreach(println)		// 打印测试：2
```
打印结果：
```
// 第一次
(Hadoop,2)
(Spark,3)
(HBase,5)
(Spark,6)
(Hadoop,1)

// 第二次
(Spark,CompactBuffer(6, 3))
(HBase,CompactBuffer(5))
(Hadoop,CompactBuffer(1, 2))
```

## keys、values
仅仅把 PairRDD 中的键或者值单独取出来形成一个 RDD

## sortByKey()
```Scala
val list = List("Hadoop 2", "Spark 3", "HBase 5", "Spark 6", "Hadoop 1")
val rdd = sc.parallelize(list)

val split = (line : String) => {
    val res = line.split(" ")
    (res(0), res(1).toInt)
}

val pairRdd = rdd.map(split)

val res = pairRdd.sortByKey(true)
res.collect().foreach(println)		// 打印测试：1

val res = pairRdd.sortByKey(false)
res.collect().foreach(println)		// 打印测试：2
```
打印结果：
```
# 1
(HBase,5)
(Hadoop,2)
(Hadoop,1)
(Spark,6)
(Spark,3)

# 2
(Spark,3)
(Spark,6)
(Hadoop,1)
(Hadoop,2)
(HBase,5)
```

## mapValues(func)
对 PairRDD 中的每个值进行处理，不影响 key.


## join
将两个 PairRDD 根据 key 进行连接操作

## combineByKey


# 共享变量
主要用于节省传输开销。
当Spark在集群的多个节点上的多个任务上并行运行一个函数时，它会吧函数中涉及到的每个变量在每个任务中生成一个副本。但是，有时需要在多个任务之间共享变量，或者在任务和任务控制节点之间共享变量。

为满足这种需求，Spark提供了两种类型的变量：广播变量（broadcast variables）和累加器（accumulators）。
广播变量用来把变量在所有节点的内存之间进行共享；
累加器则支持在所有不同节点之间进行累加计算（比如计数、求和等）

## 广播变量
允许程序开发人员在每个机器上缓存一个只读变量，而不是在每个机器上的每个任务都生成一个副本。
Spark的“行动”操作会跨越多个阶段（Stage），对每个阶段内的所有任务所需要的公共数据，Spark会自动进行广播。

可以使用 broadcast() 方法封装广播变量
```Scala
val broadcastVar = sc.broadcast(Array(1, 2, 3))

println(broadcastVar.value)
```
## 累加器
Spark 原生支持数值型累加器，可以通过自定义开发对新类型支持的累加器。

### longAccumulator & doubleAccumulator
Spark 自带长整型和双精度数值累加器，可以通过以上两个方法创建。创建完成之后可以使用 add 方法进行累加操作，但在每个节点上只能进行累加操作，不能读取。只有任务控制节点可以使用 value 方法读取累加器的值。

```Scala
val accum = sc.longAccumulator("OneAccumulator")
sc.parallelize(Array(1, 2, 3)).foreach(x => accum.add(x))
accum.value
```

# 数据读写

## 文件系统数据读写
**读写本地文件**
```Scala
val aFile = sc.textFile("file:///home/spark/somewords.txt")

// 保存时会生成一个目录，内容被跌倒这个目录中
aFile.saveAsTextFile("file:///home/spark/something.txt")
```


**读写HDFS文件**
```Scala
val aFile = sc.textFile("hdfs://weilu131:9000/home/spark/somewords.txt")

// 保存时会生成一个目录，内容被跌倒这个目录中
aFile.saveAsTextFile("hdfs://weilu131:9000/home/spark/something.txt")
```

## HBase 读写

上传 JAR 包

```
/usr/local/hadoop-2.6.0-cdh5.15.0/bin/hdfs dfs -put /usr/local/hbase-1.2.0-cdh5.15.0/lib/hbase*.jar /spark_jars
/usr/local/hadoop-2.6.0-cdh5.15.0/bin/hdfs dfs -put /usr/local/hbase-1.2.0-cdh5.15.0/lib/guava-12.0.1.jar /spark_jars
/usr/local/hadoop-2.6.0-cdh5.15.0/bin/hdfs dfs -put /usr/local/hbase-1.2.0-cdh5.15.0/lib/htrace-core-3.2.0-incubating.jar /spark_jars
/usr/local/hadoop-2.6.0-cdh5.15.0/bin/hdfs dfs -put /usr/local/hbase-1.2.0-cdh5.15.0/lib/protobuf-java-2.5.0.jar /spark_jars
```

`

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTAxMzE5MTY4MywtNDIxNzcxMDQzLDEwOT
U5MzM0OTYsLTMwODQxNTQ2NywxNDE2NzI2MDAwXX0=
-->