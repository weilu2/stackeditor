# 数据模型  
  
**Namespace(表命名空间）**  
不是必须的，如果需要分组管理表，就可以使用表命名空间。  
  
**Table（表）**  
由一个或多个列族构成。  
  
**Column Family（列族）**  
列的集合。数据属性定义的基本单位。  
  
**Column Qualifier（列限定符，简称列）**  
多个列构成一个列族，同一个列族中，不同行数据的列可以不同。  
  
**Row（行）**  
存在一个行键，以及对应每个列族和列的值。  
  
**Cell（单元格）**  
存放数据的单位，一个单元格由一个行键、一个列族、一个列限定符确定。  
  
**Timestamp（时间戳）**  
每个单元格的数据可以存储多个版本，通过时间戳确定不同版本的数据。  
  
## 表命名空间  
如果增加了命名空间，表名就变成了：  
```  
[Namespace]:[Table]  
```  
  
表命名空间可以填补 HBase 无法在一个实例上分库的缺陷。通过表命名空间可以将不同的表进行分组，然后设定不同的环境，比如配额管理、安全管理等。  
  
保留表空间  
- Hbase：系统表空间，用于 HBase内部表  
- defalut：那些没有定义表空间的表都被自动分配到这个表空间下  
  
  
# 集群架构  
  
一个常规的基于 HDFS 的 HBase 集群架构主要包括：Master、RegionServer、Region、HDFS 和 ZooKeeper。  
  
## Master  
负责启动时候分配 Region 到具体的 RegionServer，执行各种管理操作。  
  
## RegionServer  
上面有一个或多个 Region，读写的数据就存储在 Region 上。如果部署的 HBase 是基于 HDFS 的，那么 Region 的所有数据存取操作都是调用了 HDFS 的客户端接口实现的。  
  
## Region  
表的一部分数据。 HBase 是一个会自动分片的数据库。一个 Region 就相当于关系型数据库中分区表的一个分区，或者 MongoDB 的一个分片。  
  
## HDFS  
数据的实际载体。  
  
## ZooKeeper  
ZooKeeper 在 HBase 中的重要性甚至超过了 Master。读取数据所需要的元数据表 hbase:meata 的位置存储在 ZooKeeper 上。  
  
# RegionServer 架构  
  
## WAL  
预写日志，WAL 是 Write-Ahead Log 的缩写。当操作到达 Region 时，HBase 先将操作写入 WAL 中。然后将数据写入基于内存的 Memstore 中，当数据到达一定数量时，刷写（flush）到最终的 HFile 中。  
  
如果在这个过程中服务器宕机，数据就丢失了，就会从 WAL 中恢复数据。  
  
## Region  
Region 相当于一个数据库分片。每一个 Region 都有起始 rowkey 和结束 rowkey，代表了存储的 row 范围。每台 RegionServer 中可以有多个 Region。  
  
## Store  
位于 Region 中存储数据的单元。每一个 Region 内部都包含多个 Store 实例。一个 Store 对应一个列族的数据。  
  
如果一个表有两个列族，那么在一个 Region 中就会有两个 Store。  
  
Store 内部由 MemStore 和 HFile 两个部分构成。  
  
### MemStore  
每个 Store 中都有一个 MemStore 实例。数据写入 WAL 后，就会被放入 MemStore 中。MemStore 是内存的存储对象，只有当 MemStore 存满之后，数据才会写入 HFile。  
  
  
  
### HFile  
在 Store 中由多个 HFile。是数据的存储实体。
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEzMzUyMzU0ODFdfQ==
-->