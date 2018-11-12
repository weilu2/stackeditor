Hadoop 集群部署方案

# 环境说明
配置环境准备四台机器，四台机器环境是 `CentOS 7.5`，IP和主机名配置如下：
```
192.168.0.131 weilu131
192.168.0.132 weilu132
192.168.0.135 weilu135
192.168.0.151 weilu151
```
**注意：主机名不可以有下划线**

# 前置配置

## 免密登录
生成密钥：
```
ssh-keygen
```
这个会生成在 `/root/.ssh/` 目录下，然后进入该目录，将公钥拷贝到其它两台机器上：
```
ssh-copy-id -i id_rsa.pub root@weilu132
```

具体可以参考：https://weilu2.github.io/2018/10/08/CentOS-7-4-%E9%85%8D%E7%BD%AESSH%E5%85%8D%E5%AF%86%E7%99%BB%E5%BD%95/


## JDK环境
从本地将JDK包拷贝到机器上：
```
[root@weilu_125 packages]# scp jdk-8u181-linux-x64.tar.gz root@192.168.0.131:/usr/local/
root@192.168.0.131's password: 
jdk-8u181-linux-x64.tar.gz                    100%  177MB  11.1MB/s   00:15    
```

解压到当前目录：
```
tar -xvzf jdk-8u181-linux-x64.tar.gz
```

配置环境变量：
```
vim /etc/profile
```
在其中末尾添加以下内容：
```
export JAVA_HOME=/usr/local/jdk1.8.0_181  
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar  
export PATH=$PATH:$JAVA_HOME/bin
```
然后重新加载配置文件：
```
source /etc/profile
```

检查配置结果：
```
[root@weilu_132 local]# java -version
java version "1.8.0_181"
Java(TM) SE Runtime Environment (build 1.8.0_181-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.181-b13, mixed mode)
```
用相同的方法将其余两台机器也配置好。

## 防火墙配置

如果开启了防火墙，那么就需要开启以下端口

### 9000
这个端口是 Hadoop 集群中 NameNode 与 DataNode 通信的端口。

### 8031
这个端口是 Yarn 的 ResourceManager 与 NodeManager 通信的端口。

# 配置 Hadoop

下载：http://archive.cloudera.com/cdh5/cdh/5/

文件放置在 `/usr/local/` 下，解压缩：
```
tar -zxvf hadoop-2.6.0-cdh5.15.0.tar.gz
```

## hadoop-env.sh
修改 hadoop 环境配置中的 JDK 配置：
```
vim /usr/local/hadoop-2.6.0-cdh5.15.0/etc/hadoop/hadoop-env.sh 
```
修改其中的 `JAVA_HOME`
```
export JAVA_HOME=/usr/local/jdk1.8.0_181
```

## core-site.xml

配置 NameNode 的URI：
```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://weilu131:9000</value>
    </property>
</configuration>
```

## hdfs-site.xml

**配置 NameNode**
配置 NameNode 在本地文件系统中存放内容的位置。首先创建目录：
```
mkdir -p /home/hadoop/tmp/dfs/name
```
**配置 DataNode**
配置 DataNode 在本地文件系统中存放内容的位置。首先创建目录：
```
mkdir -p /home/hadoop/tmp/dfs/data
```
这个目录是自己定义的。
```
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/home/hadoop/tmp/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/home/hadoop/tmp/dfs/data</value>
    </property>
</configuration>
```

# 配置 Yarn

## yarn-site.xml

```
<configuration>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>weilu131</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

# 配置 MapReduce

## mapred-site.xml

在每个节点上配置以下内容。

从模板中拷贝一份 mapred-site.xml 文件：
```
cp mapred-site.xml.template mapred-site.xml
```
在新文件中添加以下内容：
```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

# 配置从节点

## slaves
在 slaves 文件中列出从节点的主机名或IP：

```
weilu132
weilu135
weilu151
```

# 分发配置文件
```
scp slaves root@weilu_132:/usr/local/hadoop-2.6.0-cdh5.15.0/etc/hadoop
scp slaves root@weilu_135:/usr/local/hadoop-2.6.0-cdh5.15.0/etc/hadoop
scp slaves root@weilu_151:/usr/local/hadoop-2.6.0-cdh5.15.0/etc/hadoop
```

## 格式化文件系统

```
[root@weilu_131 bin]# /usr/local/hadoop-2.6.0-cdh5.15.0/bin/hdfs namenode -format
...
18/10/30 21:12:08 INFO common.Storage: Storage directory /home/hadoop/tmp/dfs/name has been successfully formatted.
```
看到有这行输出表明格式化成功。

## 启动
```
[root@weilu_131 sbin]# ./start-all.sh 
This script is Deprecated. Instead use start-dfs.sh and start-yarn.sh
18/10/30 22:17:05 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Starting namenodes on [weilu131]
weilu131: starting namenode, logging to /usr/local/hadoop-2.6.0-cdh5.15.0/logs/hadoop-root-namenode-weilu131.out
weilu132: starting datanode, logging to /usr/local/hadoop-2.6.0-cdh5.15.0/logs/hadoop-root-datanode-weilu_132.out
weilu135: starting datanode, logging to /usr/local/hadoop-2.6.0-cdh5.15.0/logs/hadoop-root-datanode-weilu_135.out
weilu151: starting datanode, logging to /usr/local/hadoop-2.6.0-cdh5.15.0/logs/hadoop-root-datanode-weilu_151.out
Starting secondary namenodes [0.0.0.0]
0.0.0.0: starting secondarynamenode, logging to /usr/local/hadoop-2.6.0-cdh5.15.0/logs/hadoop-root-secondarynamenode-weilu131.out
18/10/30 22:17:20 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
starting yarn daemons
starting resourcemanager, logging to /usr/local/hadoop-2.6.0-cdh5.15.0/logs/yarn-root-resourcemanager-weilu_131.out
weilu132: starting nodemanager, logging to /usr/local/hadoop-2.6.0-cdh5.15.0/logs/yarn-root-nodemanager-weilu_132.out
weilu135: starting nodemanager, logging to /usr/local/hadoop-2.6.0-cdh5.15.0/logs/yarn-root-nodemanager-weilu_135.out
weilu151: starting nodemanager, logging to /usr/local/hadoop-2.6.0-cdh5.15.0/logs/yarn-root-nodemanager-weilu_151.out

```

# 验证
在 weilu131上：
```
[root@weilu131 sbin]# jps
3927 NameNode
4520 Jps
4254 ResourceManager
4111 SecondaryNameNode
```
因为在 weilu131 上部署了 NameNode 和 ResourceManager，因此使用 jps 命令应该能够看到这几个进程。

在其余几个节点上：
```
[root@weilu132 hadoop]# jps
1290 DataNode
1531 Jps
1391 NodeManager
```
因为其余几个节点上面只部署了 DataNode 和 NodeManager，因此使用 jps 命令应该能够看到这几个进程。

# 问题

## 集群正常启动 50070 页面显示没有 Live Node

NameNode 和 DataNode 都正常启动，但是访问 50070 页面发现检测不到任何节点。查看 DataNode 的日志，发现如下内容：
```
2018-10-30 23:02:04,907 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: weilu131/192.168.0.131:9000. Already tried 1 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
2018-10-30 23:02:05,909 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: weilu131/192.168.0.131:9000. Already tried 2 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
2018-10-30 23:02:06,911 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: weilu131/192.168.0.131:9000. Already tried 3 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
2018-10-30 23:02:07,912 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: weilu131/192.168.0.131:9000. Already tried 4 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
```
一种可能性是由于 NameNode 的防火墙开着，并且不允许访问 9000 端口，这样 DataNode 就没法向 NameNode 报告状态，将9000端口开启后，该问题马上解决了，验证了该猜想。

## 集群正常启动 Yarn 页面上看不到活动节点
查看 NodeManager 节点上的 Yarn 启动日志，可以看到以下错误信息：
```
org.apache.hadoop.yarn.exceptions.YarnRuntimeException: java.net.NoRouteToHostException: No Route to Host from  weilu132/192.168.0.132 to weilu131:8031 failed on socket timeout exception: java.net.NoRouteToHostException: 没有到主机的路由; For more details see:  http://wiki.apache.org/hadoop/NoRouteToHost
	at org.apache.hadoop.yarn.server.nodemanager.NodeStatusUpdaterImpl.serviceStart(NodeStatusUpdaterImpl.java:215)
...
```

问题还是一样的问题，无法和 ResourceManager 节点进行通讯，yarn 使用的是 8031 端口，设置一下，然后重启集群。

# 部署 Zookeeper
四台机器中，仅在 `weilu132/weilu135/weilu151`三台上部署 zookeeper。

## 开放端口

```
firewall-cmd --add-port=2888/tcp --permanent
firewall-cmd --add-port=3888/tcp --permanent

firewall-cmd --add-port=2181/tcp --permanent

firewall-cmd --add-port=16010/tcp --permanent
firewall-cmd --add-port=16020/tcp --permanent

firewall-cmd --add-port=60000/tcp --permanent
firewall-cmd --add-port=50010/tcp --permanent
firewall-cmd --add-port=60020/tcp --permanent

firewall-cmd --add-port=60010/tcp --permanent
firewall-cmd --add-port=60030/tcp --permanent
firewall-cmd --reload
```


将安装包拷贝到 `/usr/local/` 目录下，解压：
```
tar -zxvf zookeeper-3.4.5-cdh5.15.0.tar.gz
```

## zoo.cfg
在 zookeeper 包的根目录创建一个文件夹，随意命名：
```
/usr/local/zookeeper-3.4.5-cdh5.15.0/zookeeperDataDir
```

进入 `/usr/local/zookeeper-3.4.5-cdh5.15.0/conf` 目录，拷贝一份 `zoo_sample.cfg` 文件，命名为 `zoo.cfg`：
```
cp zoo_sample.cfg zoo.cfg
```
在 `zoo.cfg` 中配置以下内容：
```
# The number of milliseconds of each tick
tickTime=2000

# The number of ticks that the initial synchronization phase can take
initLimit=10

# The number of ticks that can pass between sending a request and getting an acknowledgement
syncLimit=5

# the directory where the snapshot is stored.
dataDir=/usr/local/zookeeper-3.4.5-cdh5.15.0/zookeeperDataDir

# the port at which the clients will connect
clientPort=2181

server.1=weilu132:2888:3888
server.2=weilu135:2888:3888
server.3=weilu151:2888:3888
```

## myid
在每台 ZooKeeper 节点的数据目录（dataDir目录）下存放一个 `myid` 文件，文件中存放一个集群中唯一的ID，表明这台机器的ID，ID范围是 `1~255`

## 分发配置
将 zoo.cfg 和 myid 分发到其他及台机器上，对应修改 myid。

```
scp zoo.cfg root@weilu135:/usr/local/zookeeper-3.4.5-cdh5.15.0/conf
```
分发目录时增加参数 `-r`
```
scp -r zookeeperDataDir/ root@weilu151:/usr/local/zookeeper-3.4.5-cdh5.15.0/zookeeperDataDir
```

## 启动 zookeeper
启动时需要分别在每个节点使用 `bin/zkServer.sh` 脚本启动：
```
./zkServer.sh start
```
在所有节点启动完成之前，如果使用 `./zkServer.sh status` 查看状态时，会提示
```
JMX enabled by default
Using config: /usr/local/zookeeper-3.4.5-cdh5.15.0/bin/../conf/zoo.cfg
Error contacting service. It is probably not running.
```
如果都启动之后，再次查看会得到：
```
JMX enabled by default
Using config: /usr/local/zookeeper-3.4.5-cdh5.15.0/bin/../conf/zoo.cfg
Mode: follower
```


# 部署 HBase

将安装包拷贝到 `/usr/local/` 目录下，解压 Hbase：
```
tar -zxvf hbase-1.2.0-cdh5.15.0.tar.gz
```

## hbase-env.sh

配置 JDK 环境：
```
export JAVA_HOME=/usr/local/jdk1.8.0_181
```

配置不使用 HBase 默认自带的 ZooKeeper：
```
export HBASE_MANAGES_ZK=false
```

## hbase-site.xml
创建临时文件目录：
```
mkdir -p /home/hbase/tmp
```

配置
```
<configuration>
    <property>
        <name>hbase.tmp.dir</name>
        <value>/home/hbase/tmp</value>
    </property>
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://weilu131:9000/hbase</value>
    </property>
    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>weilu132,weilu135,weilu151</value>
    </property>
    
    <property>
        <name>hbase.master.info.port</name>
        <value>60010</value>
    </property>
    <property>
        <name>hbase.regionserver.info.port</name>
        <value>60030</value>
    </property>
</configuration>
```

## regionservers
配置 Region 服务器列表，在 `regionservers` 文件中配置以下内容，类似于 Hadoop 集群的 slaves 列表：
```
weilu132
weilu135
weilu151
```

## backup-masters
在 `conf/backup-masters` 中用来配置备份 master 服务器。
```
weilu132
```

## 分发配置
```
# hbase-env.sh 文件
scp hbase-env.sh root@weilu132:/usr/local/hbase-1.2.0-cdh5.15.0/conf/

# hbase-site.xml
scp hbase-site.xml root@weilu132:/usr/local/hbase-1.2.0-cdh5.15.0/conf/

# regionservers
scp regionservers root@weilu132:/usr/local/hbase-1.2.0-cdh5.15.0/conf/

#backup-masters
scp backup-masters root@weilu132:/usr/local/hbase-1.2.0-cdh5.15.0/conf/
```

## 启动服务
```
./start-hbase.sh
```


## 验证
访问：http://192.168.0.131:60010


## 问题

### zookeeper.MetaTableLocator: Failed verification of hbase:meta

```
2018-11-05 22:27:04,735 INFO  [weilu131:60000.activeMasterManager] zookeeper.MetaTableLocator: Failed verification of hbase:meta,,1 at address=weilu135,60020,1541427295657, exception=org.apache.hadoop.hbase.NotServingRegionException: Region hbase:meta,,1 is not online on weilu135,60020,1541428018822
	at org.apache.hadoop.hbase.regionserver.HRegionServer.getRegionByEncodedName(HRegionServer.java:2997)
	at org.apache.hadoop.hbase.regionserver.RSRpcServices.getRegion(RSRpcServices.java:1069)
	at org.apache.hadoop.hbase.regionserver.RSRpcServices.getRegionInfo(RSRpcServices.java:1349)
	at org.apache.hadoop.hbase.protobuf.generated.AdminProtos$AdminService$2.callBlockingMethod(AdminProtos.java:22233)
	at org.apache.hadoop.hbase.ipc.RpcServer.call(RpcServer.java:2191)
	at org.apache.hadoop.hbase.ipc.CallRunner.run(CallRunner.java:112)
	at org.apache.hadoop.hbase.ipc.RpcExecutor$Handler.run(RpcExecutor.java:183)
	at org.apache.hadoop.hbase.ipc.RpcExecutor$Handler.run(RpcExecutor.java:163)
```

这个问题是 zookeeper 的数据出错导致的，将 zookeeper 集群都停掉，然后将 Datadir目录中除了配置的 `myid` 以外的文件都删掉，然后启动 zookeeper，应该就可以了。
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTMyMTIwMTc4OF19
-->