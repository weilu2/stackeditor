# 安装配置修改配置文件Configure Hadoop

## 下载Download

View the page `http://archive.cloudera.com/cdh5/cdh/5/` and select a version, such as `hadoop-2.6.0-cdh5.15.0.tar.gz` :
```
wget http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.15.0.tar.gz
```

copy and uncompress to `/usr/local/` :
```
cp hadoop-2.6.0-cdh5.15.0.tar.gz /usr/local/hadoop-2.6.0-cdh5.15.0.tar.gz

tar -zxvf /usr/local/hadoop-2.6.0-cdh5.15.0.tar.gz
```

# 安装配置 Hive

## 下载

```
wget http://archive.cloudera.com/cdh5/cdh/5/hive-1.1.0-cdh5.15.0.tar.gz
```

```
export HADOOP_HOME=/usr/local/hadoop-2.6.0-cdh5.9.3
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

something changed

## hadoop-env.sh
文件位置：
```
{hadoop_root}/etc/hadoop/hadoop-env.sh
```
修改配置文件，修改JAVA_HOME 为当前实际目录
```
export JAVA_HOME=/usr/local/jdk1.8.0_181
```

## core-site.xml
hadoop2默认端口为 `8020`。并且默认hadoop数据是存储在临时目录中的，系统重启后，数据会丢失，因此还需要指定一个数据存储目录。
```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://127.0.0.1:9000</value>
    </property>

    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/hadoop/app/tmp</value>
    </property>
</configuration>
```

## hdfs-site.xml
```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

# 格式化文件系统
仅在第一次搭建的时候格式化
```
bin/hdfs namenode -format
```

# 启动hdfs
```
sbin/start-dfs.sh
```

验证是否启动成功：
1、查看进程
```
[root@weilu_125 etc]# jps
7602 NameNode
9014 Jps
7737 DataNode
7915 SecondaryNameNode
```

2、访问地址：http://192.168.0.125:50070/ 

# 停止 hdfs
```
sbin/stop-dfs.sh
```



<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEzMjk1NjY0MzUsLTEzMDI2MjY2MjRdfQ
==
-->