
# 4. HBase

## 4.1. 简介

## 4.2. HBase的数据类型
每一个值都是未经解释的字符串，也就是 Bytes 数组

**行键**
每行数据有一个行键

**列族**
一个 HBase 表可以包含多个列族

**列限定符**
一个列族中可以包含多个列限定符

**时间戳**

在 HBase 中需要通过这四个纬度才能唯一定位一个单元格中的数据。

## 4.3. HBase 的实现原理

### 4.3.1. 功能组件
**库函数**
一般用于链接每个客户端

**Master服务器**
充当管家的作用
1、对 HBase 中的分区信息进行维护和管理
2、维护一个 Region 服务器列表
3、对 Region 进行分配
4、负载均衡

**Region服务器**
存放每个不同的 Region
客户端直接与 Region 服务器沟通，获取要访问的数据。

### 4.3.2. 实现原理
一个 HBase 表被划分成多个 Region
同一个 Region 只能放在一个服务器上

### 4.3.3. HBase 三级寻址
HBase 的三级寻址结构：
1、第一层：Zookeeper 文件，记录了 -ROOT- 表的位置信息。
2、第二层：-ROOT-表，记录了 .META. 表的 Region 位置信息，-ROOT-表只能有一个 Region。通过-ROOT-表，就可以访问.META.表中的数据。
3、第三层：.META.表，记录了用户数据表的 Region 位置信息，.META.表可以有多个 Region，保存了 HBase 中所有用户数据表的 Region 位置信息

## 4.4. HBase 运行机制

### 4.4.1. HBase的系统架构

**客户端**
访问 HBase 的接口

**Zookeeper服务器**
实现协同管理服务
提供管家的功能，维护和管理整个 HBase 集群

**Master主服务器**
1、对表进行增删改查
2、负责不同 Region 服务器的负载均衡
3、负责调整分裂、合并后 Region 的分布
4、负责重新分配故障、失效的Region服务器

**Region服务器**
存取具体的数据

1、每一个Region服务器存储10～1000个Region
2、一台Region服务器上的Region共用一个HLog日志文件
3、HBase 的数据存储是按照列族为单位的，每一个Region会按照列族进行拆分，每一个列族是单独存储为一个Store
4、在写 Store 时，不是直接写到文件中，而是先写到 MemStore 缓存中，写满后刷写到 StoreFile 文件中，这个StoreFile是HBase中文件的表示形式，然后这个文件是通过HDFS的HFile进行存储的。

### 4.4.2. Region 服务器的工作原理

### 4.4.3. Store 的工作原理



### 4.4.4. HLog的工作原理

## 4.5. HBase 应用方案
### 4.5.1. HBase 在实际应用中的性能优化方法

### 4.5.2. HBase 如何检测性能
**Master-status**

**Ganglia**
UC Berkeley 发起的一个开源集群监视项目

**OpenTSDB**


**Ambari**
是 Hadoop 架构上的一个产品，作用是创建管理监视整个 Hadoop 集群，HBase 是整个集群的一部分，所以也可以对 HBase 进行监视。

### 4.5.3. HBase之上如何构建 SQL 引擎和 HBase 二级索引


# 8. Hive
## 8.1. 数据仓库概念

**数据仓库概念**
一个面向主题的、集成的、相对稳定的、反映历史变化的数据集和，用于支撑管理决策。

## 8.2.  Hive简介
1、传统数据仓库同时职称数据的存储和处理分析
2、Hive本身并不支持数据的存储和处理，而是相当于提供了一种编程语言。
3、Hive基于HDFS和Mapreduce进行数据的存储和处理
4、借鉴 SQL 语言设计实现了新的查询语言 HiveQL

### Hive两个方面特性
**采用批处理方式处理海量数据**
1、Hive会将HiveQL转为 MapReduce 任务进行处理
2、数据仓库存储的是静态数据，适合使用批处理方式，不需要快速给出响应结果，数据本身也不会频繁变化

**Hive提供了一系列对数据进行提取、转换、加载（ETL）的工具**

### Hive对外访问接口
1、CLI
一种命令行工具

2、HWI
Hive Web Interface

3、JDBC和ODBC

4、Thrift Server
基于Thrift架构开发的接口，允许外界通过该接口，实现对Hive仓库的RPC调用

### Hive驱动模块
包含编译器、优化器、执行器，负责把 HiveQL 语句转化成一系列的 MapReduce 作业

### Hive元数据
Hive的元数据是存储在一个独立的关系型数据库中的，可以存储在外部数据库，也可以存储在自带的数据库中。

### Hive HA基本原理
Hive High Avaliability，存在多个 Hive 实例，通过一个代理对外提供一个统一的接口。

# 9. Hadoop 再探讨
## 9.1. Hadoop 的优化与发展
**Hadoop 1.x 版本**
1、抽象层次低，需要人工编码
2、表达能力有限，将所有逻辑抽象为 Map 和 Reduce 两个函数，能够表达的功能有限
3、开发者自己管理作业（Job）之间的依赖关系
4、难以看到程序的整体逻辑，只能通过阅读代码的方式理解逻辑
5、执行迭代操作效率低，每个作业之间的数据需要落地到HDFS上，不能直接进行传输
6、资源浪费
7、实时性差

**Hadoop 2.x 版本**
1、设计了 HDFS HA，提供名称节点（NameNode）的热备机制，解决单一名称节点，节点失效问题
2、设计了 HDFS Federation 管理多个命名空间，解决单一命名空间，无法实现资源隔离的问题
3、设计了新的资源调度框架 YARN，解决了资源管理效率低的问题

**Hadoop 生态圈的发展**
**Pig**

## 9.4. Hadoop 生态圈的一些主要框架

### 9.4.1. Pig


# 10. Spark

## 10.1. Spark简介
1、执行速度快
使用DAG执行引擎以支持循环数据流与内存计算

2、容易使用
支持 Scala、Java、Python和R语言进行编程，还可以通过 Spark Shell 进行交互式编程

3、通用性
提供了完整而强大的技术栈，包括SQL查询、流式计算、机器学习和图算法组件

4、运行模式多样
可运行于独立的集群模式中，可运行与 Hadoop 中；
可以访问 HDFS、HBase、Hive等多种数据源；

Scala特性
具备强大的并发性，支持函数式编程，可以很好的支持分布三系统；
Scala兼容Java

**Spark 相比于 Hadoop 的优点**
1、Spark计算模式也属于 MapReduce，但不局限于此，Spark还提供了多种数据集操作类型，编程模型比 Hadoop MapReduce 更灵活。

2、Spark 提供了内存计算，可将中间结果放到内存中，对于迭代运算效率更高。

3、Spark 基于 DAG 的任务调度执行机制，要优于 Hadoop MapReduce 的迭代执行机制。



## 10.2. Spark生态系统
企业内的主要应用：
1、复杂的批量数据处理
2、基于历史数据的交互式查询
3、基于实时数据流的数据处理

**Spark Core**
提供内存计算

**Spark SQL**
提供交互式查询分析

**Spark Streaming**
提供流式计算

**Mllib** 
提供机器学习算法库组件

**GraphX**
提供图计算

## 10.3. Spark运行架构

**RDD**
Resillient Distributed Dataset，弹性分布式数据集，是分布式内存的一个抽象概念，提供了一种高度受限的共享内存模型。

**DAG**
Directed Acyclic Graph，有向无环图的简称，反映 RDD 之间的依赖关系。

**Executor**
是运行在工作节点（WorkerNode）上的一个进程，负责运行 Task

**Application**
用户编写的 Spark 应用程序。

**Task**
运行在 Executor 上的工作单元。

**Job**
一个 Application 包含多个 Job，一个Job包含多个 RDD 及作用于相应 RDD 上的操作。

**Stage**
是 Job 的基本调度单位，一个 Job 会分为多组 Task，每组 Task 被称为 Stage，或者也被成为 TaskSet，代表了一组关联的/相互之间没有 Shuffle 依赖关系的任务组成的任务集

### Spark 运行架构特点
1、每个 Application 都有自己专属的 Executor 进程，并且该进程在 Application 运行期间一直驻留。 Executor 进程以多线程的方式运行 Task

2、Spark 运行过程于资源管理其无关，只要能够获取 Executor 进程并保持通信即可

3、Task采用了数据本地性和推测执行等优化机制

### RDD 的概念
**设计背景**
1、许多迭代式算法（比如机器学习、图算法等）和交互式数据挖掘工具，共同之处是不同计算阶段会重用中间结果
2、目前 MapReduce 框架将中间结果写入磁盘，带来大量数据复制、磁盘 IO和序列化开销

RDD 为解决这些问题而设计。

**RDD概念**
一个 RDD 就是一个分布式对象集和，本质上是一个只读的分区记录集和，每个 RDD 可以分为多个分区，每个分区就是一个数据集片段，并且一个 RDD 的不同分区可以被保存到集群中的不同节点上，从而可以在集群中的不同节点上进行计算

1、RDD 提供了一组丰富的操作以支持常见的数据运算，分为“动作（Action）” 和“转换（Transformation）”两种类型。
2、RDD 提供的转换接口都非常简单，都是类似 map、filter、groupBy、join等粗力度的数据转换操作，而不是针对某个数据项的细粒度修改
3、表面上 RDD 的功能受限，实际上 RDD 已经被实践证明可以高效的表达许多框架的编程模型（比如 MapReduce、SQL、Pregel）

### 10.3.5. RDD 的依赖关系和运行过程

**窄依赖**
一个父亲RDD分区对应一个孩子RDD分区；多个父亲RDD分区对应于一个孩子RDD分区；

**宽依赖**
一个父亲RDD分区对应多个孩子RDD分区；

**Stage 的划分**
1、ShuffleMapStage
2、ResultStage


## 10.4. Spark SQL
代替 Hive

## 10.5. Spark 的部署和应用方式

### Standalone

### Spark on Mesos
Mesos 是一种资源管理框架，是官方推荐的框架

### Spark on Yarn

# 11. 流计算

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTM0NzM2NzQ4NF19
-->