

# HBase Shell
使用以下命令可以进入 HBase 的命令行模式：
```
/usr/local/hbase-1.2.0-cdh5.15.0/bin/hbase shell
```
打开后如果如下反馈，则表明成功：
```
[root@weilu131 ~]# /usr/local/hbase-1.2.0-cdh5.15.0/bin/hbase shell
2018-11-21 10:37:11,476 INFO  [main] Configuration.deprecation: hadoop.native.lib is deprecated. Instead, use io.native.lib.available
2018-11-21 10:37:12,197 WARN  [main] util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 1.2.0-cdh5.15.0, rUnknown, Thu May 24 04:30:05 PDT 2018

hbase(main):001:0> 
```

# 表操作

## 创建表 create
创建一张存放医疗问答数据的表，其中包含几个列族：
```
create 'medicalqa', 'http', 'department', 'question', 'user', 'answers'
```
其中 `medicalqa` 是表名，后面跟的几个都是列族。

执行成功后的反馈如下：
```
hbase(main):001:0> create 'medicalqa', 'http', 'department', 'question', 'user', 'answers'
0 row(s) in 4.7790 seconds

=> Hbase::Table - medicalqa
```

## 增加列族
增加列族实际上就是修改表结构，并且在修改是是写入到每个 Region 中的，因此会影响到正在使用的 Region，在修改之前要先对表进行 `disable`。
```
alter 'medicalqa', 'newcolumnfamily'
```
执行结果：
```
hbase(main):006:0> alter 'medicalqa', 'newcolumnfamily'
Updating all regions with the new schema...
0/1 regions updated.
1/1 regions updated.
Done.
0 row(s) in 3.9190 seconds
```
可以看到这个表在一个 Region 上。

## 删除列族
删除列族，同样的在执行之前应该先 `disable` 表
```
alert 'medicalqa', 'delete' => 'newcolumnfamily'
```
反馈：
```
hbase(main):009:0> alert 'medicalqa', 'delete' => 'newcolumnfamily'
NoMethodError: undefined method `alert' for #<Object:0x35bfa1bb>

hbase(main):010:0> alter 'medicalqa', 'delete' =>'newcolumnfamily'
Updating all regions with the new schema...
0/1 regions updated.
1/1 regions updated.
Done.
0 row(s) in 4.1230 seconds
```

## 查看表名 list
使用命令 `list` 可以罗列出当前所有的表。
```
hbase(main):002:0> list
TABLE
medicalqa
student
2 row(s) in 0.0350 seconds

=> ["medicalqa", "student"]
```

## 查看表信息 describe
使用命令 `describe` 可以查看表结构信息：
```
describe 'medicalqa'
```
主要是列族的信息，反馈的内容比较乱：
```
hbase(main):005:0> describe 'medicalqa'
Table medicalqa is ENABLED                                                
medicalqa                                                                 
COLUMN FAMILIES DESCRIPTION                                               
{NAME => 'answers', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}                      
{NAME => 'department', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}                   
{NAME => 'http', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}                         
{NAME => 'question', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}                     
{NAME => 'user', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}                         
5 row(s) in 0.0980 seconds
```
取其中一个列族的信息结构化之后是如下内容：
```JSON             
{
    NAME => 'department', 
    BLOOMFILTER => 'ROW', 
    VERSIONS => '1', 
    IN_MEMORY => 'false', 
    KEEP_DELETED_CELLS => 'FALSE', 
    DATA_BLOCK_ENCODING => 'NONE', 
    TTL => 'FOREVER', 
    COMPRESSION => 'NONE', 
    MIN_VERSIONS => '0', 
    BLOCKCACHE => 'true', 
    BLOCKSIZE => '65536', 
    REPLICATION_SCOPE => '0'
}
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTM0NzkyMjc3MV19
-->