基于 Hadoop 集群部署 Spark 集群

# 部署 Scala
下载 Scala，将 `scala-2.11.12.tgz` 放置到 `/usr/local/` 下，并解压。
```
tar -zxvf scala-2.11.12.tgz 
```

配置环境变量，在 `/etc/profile` 中增加以下内容：
```
export SCALA_HOME=/usr/local/scala-2.11.12
export PATH=$PATH:$SCALA_HOME/bin
```
重新载入：
```
source /etc/profile
```
验证：
```
[root@weilu131 local]# scala
Welcome to Scala 2.11.12 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_192).
Type in expressions for evaluation. Or try :help.

scala> 
```

# 部署 Spark

## 开放端口
**8080**：Master节点上的端口，提供Web UI
**8081**：Worker节点上的端口，提供Web UI
**7077**：Master 与 Worker 通信的端口
```
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --add-port=7077/tcp --permanent
firewall-cmd --add-port=8081/tcp --permanent
firewall-cmd --add-port=8030/tcp --permanent

firewall-cmd --add-port=30000-50000/tcp --permanent
firewall-cmd --reload
```
30000-50000之间的端口是Yarn用来通信的端口，不知道是随机的还是怎么弄的，试出来几十个，干脆就把这个区段都开放了。

## 下载&解压
从 [Apache Spark download page](https://spark.apache.org/downloads.html) 下载安装包。在选择安装包类型时，如果是针对某个版本的 Hadoop 的话，可以选择 `Pre-build for Apache Hadoop 2.6`，或 `Pre-build for Apache Hadoop 2.7 and later`。分别是针对 2.6 和 2.7 版本的。或者也可以选择 `Pre-build with user-provided Apache Hadoop`，表示适用于所有版本 Hadoop。

下载后解压到 `/usr/local/` 目录。
```
tar -zxvf spark-2.4.0-bin-hadoop2.6.tgz
```

## 配置 spark-env.sh

在 Spark 的 `conf` 目录下拷贝一份 `spark-env.sh` 文件：
```
cp spark-env.sh.template spark-env.sh
```
在文件最后添加以下内容：
```
export JAVA_HOME=/usr/local/jdk1.8.0_192
export SCALA_HOME=/usr/local/scala-2.11.12

export HADOOP_HOME=/usr/local/hadoop-2.6.0-cdh5.15.0
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop

export SPARK_WORKING_MEMORY=4g  #每一个worker节点上可用的最大内存
export SPARK_MASTER_IP=iflysse131   #驱动器节点IP

export SPARK_DIST_CLASSPATH=$(/usr/local/hadoop-2.6.0-cdh5.15.0/bin/hadoop classpath)
```

1、HADOOP_CONF_DIR
要让 Spark 与 YARN 资源管理器通信的话，需要将 Hadoop 的配置信息告诉 Spark，通过配置 `HADOOP_CONF_DIR` 环境变量实现。

## 配置 spark-defaults.conf
将 Spark 的 Master 设置为 Yarn。将 `conf` 目录下的配置文件模板拷贝一份：
```
cp spark-defaults.conf.template spark-defaults.conf
```
### spark.master
配置 Spark 的集群管理器为 yarn
```
spark.master                     yarn
```

### spark.yarn.jars
配置 Spark 的 jar 包。在配合 Hadoop 集群下提交任务时，会将 jar 包提交到 HDFS 上，为防止每次提交任务时都提交，所以在 HDFS 上上传一份公共的。

在 HDFS 上创建存放 jar 包的目录：
```
/usr/local/hadoop-2.6.0-cdh5.15.0/bin/hdfs dfs -mkdir /spark_jars
```
检查目录是否创建：
```
/usr/local/hadoop-2.6.0-cdh5.15.0/bin/hdfs dfs -ls /
```
将 spark 下的 jar 包上传到该目录下：
```
/usr/local/hadoop-2.6.0-cdh5.15.0/bin/hdfs dfs -put /usr/local/spark-2.4.0-bin-hadoop2.6/jars/* /spark_jars
```
使用如下命令检查是否上传成功：
```
/usr/local/hadoop-2.6.0-cdh5.15.0/bin/hdfs dfs -ls /spark_jars
```
在 `spark-defaults.conf` 中增加以下内容：
```
spark.yarn.jars         hdfs://iflysse131:9000/spark_jars/*
```

## 配置 slaves
配置从节点主机名（或者IP），在 Spark 的 `conf` 目录下拷贝一份 `slaves` 文件：
```
cp slaves.template slaves
```
在其中添加以下内容：
```
iflysse132
iflysse133
iflysse151
iflysse152
```

## 配置 history-server

开放Web UI 端口
```
firewall-cmd --add-port=18080/tcp --permanent
firewall-cmd --reload
```
创建日志目录：
```
/usr/local/hadoop-2.6.0-cdh5.15.0/bin/hdfs dfs -mkdir /spark_logs
```

### spark-defaults.conf
这部分配置的是 Spark 写入日志的信息。
```
spark.eventLog.enabled  true 
spark.eventLog.dir      hdfs://iflysse131:9000/spark_logs
spark.eventLog.compress true
```

### spark-env.sh

```
export SPARK_HISTORY_OPTS="-D**spark.history.ui.port**=18080 -D**spark.history.retainedApplications**=10 -D**spark.history.fs.logDirectory**=hdfs://iflysse131:9000/spark_logs"
```

启动日志服务器
```
start-history-server.sh
```

## 启动 Spark 集群

```
/usr/local/spark-2.4.0-bin-hadoop2.6/sbin/start-all.sh 
```

## 验证

### jps 命令
在 Master 和 Worker 节点上分别使用 jps 命令，可以分别看到 Master 和 Worker 进程。

### Web UI
访问Master：http://192.168.0.131:8080/
可以看到当前集群的状况。

## 启动 spark-shell

```
[root@iflysse131 ~]# /usr/local/spark-2.4.0-bin-hadoop2.6/bin/spark-shell 
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/local/spark-2.4.0-bin-hadoop2.6/jars/slf4j-log4j12-1.7.16.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/local/hadoop-2.6.0-cdh5.15.0/share/hadoop/common/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
18/11/15 15:20:46 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
Spark context Web UI available at http://iflysse131:4040
Spark context available as 'sc' (master = yarn, app id = application_1542266210770_0001).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.4.0
      /_/
         
Using Scala version 2.11.12 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_192)
Type in expressions to have them evaluated.
Type :help for more information.

scala> 
```

同时，在Yarn上可以看到这个Application

# 参考
[1] http://spark.apache.org/docs/latest/monitoring.html
[2] https://www.fwqtg.net/%E3%80%90spark%E5%8D%81%E5%85%AB%E3%80%91spark-history-server.html
[3] https://my.oschina.net/u/3754001/blog/1811243
<!--stackedit_data:
eyJoaXN0b3J5IjpbNTA3MDkyNzk5LDU2ODIxNjI0OV19
-->