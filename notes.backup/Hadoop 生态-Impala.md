[TOC] 

# Impala

![image-20201102200733355](http://img.janhen.com/20210130165731image-20201102200733355.png)

交互式查询工具    
支持 parquet 存储  

按照阶段划分⼀个⼤数据开发任务，会有：  
数据采集(⽇志⽂件，关系型数据库中)，  
数据清洗 (数据格式整理，脏数据过滤等)，  
数据预处理(为了后续分析所做的⼯作)，  
数据分析：离线处理(T+1分 析)，  
实时处理(数据到来即分析)，  
数据可视化，  
机器学习，  
深度学习等  





Hive 对交互式查询的场景⽆能为⼒  

抛弃了MapReduce 使⽤了类似于传统的 **MPP数据库技术**  

Impala：在执⾏程序之间使⽤流的⽅式传输中间结果，避免数据落盘。尽可能使⽤内存避免磁盘 开销  

处理数据量在PB级别最佳  


Impala 使用 Java、C++ 编写  

在 PB 级别，比 Hive 快 3-90 倍  

Join 优化  

Cloudera 提供的，参考 Google 新





使用的元数据为 Hive 的 ...  



**移动数据不如移动计算**





MR

### *MPP

每个节点独享磁盘和内存  

MR 慢的原因：  

- Shuffer 开销大，读写磁盘的 I/O 开销大  
- Shuffle 默认对 key 进行分区排序, 即使不需要  



MPP: 

- 提前启动好， MR 每次启动时候进行启动  

- 中间结果： Hive 需要落盘，Impala 使用流的方式传输中间结果  

- 排序： 可不对数据进行排序



缺点：  

- 只能是百节点级  

- 并发查询达到 20，处于系统的满负载  
- 资源不能通过 Yarn 进行资源调度  



Impala 需要引用 Hive 依赖包，需要 Hadoop 支持 C 的调用接口...   

rpm 安装需要管理包的依赖关系  



### 短路读取

数据节点与 xx 在同一个节点上  
直接访问本地的数据  

短路读取本地中转站  
好的方式配置 DN 与 HDFS 客户气通信  
快之间元数据。。   



### 安装

22000： 可用的 catalog 进程端口  

25000： 管理界面  。。。  



### 与 Hive 比较

Impala 与 Hive 的元数据关系？

Hive 对元数据的跟新操作不能被 Impala 感知。

Impa la 对元数据的更新操作可悲 Hive 感知



`inviolate metadata` 手动同步 Hive 元数据命令



### Imapala 架构原理

与 Hive 类似的数据分析工具，非数据库





Impalad

- Query Planner: 生成查询计划

- Query Cordinator: 查询协调器，协调多个节点上的执行

- Query Executor:

对节点获取健康情况





Catalog:

某个 impala d 更新元数据后，将此次更新同步给其他 impala d 

集群启动时，拉取元数据信息同步到 impala d 进程，后续只有执行 ·inviolate metadata`  后拉取元数据





#### *Impala 查询

一般不会问，有助于理解。





Query Planner 生成单机执行计划、分布式执行计划



分布式并行执行计划：





Hash Join



Impala 发现为小表时，采用 broadcast join，将数据复制到所有节点上



局部聚合完成后，按照分组字段分发数据，保证分组字段相同的数据去往相同节点进行全局汇总



### 执行计划生成

....   


## Impala 交互式查询  

集成了 Hive 的 sql 语法  
重点在于查询，事务性、更新操作不适合  
一般数据存储在 HDFS  

### Impala shell  

外部命令  
-r: 元数据的刷新  
-f: 指定查询文件  
-i: 连接到哪个 impala ， `-i linux122`  
-o: 将执行结果保存到文件   



展示 Impala 默认内置汉书需要进入 Impala 默认系统数据库中，其他数据库中无法查看   



更新指定的表, 非更新所有的元数据, 建议使用  

```
refresh default.t1
```

连接到其他机器

```shell
connect linux122
```


explain  
查看执行计划  
显示读取文件大小、个数、表、分区的信息    

```shell script
set explain_level=3
```


profile  
任务执行后调用  
更详细，物理层面的资源消耗信息...  

```shell script
# Planner TImeline  
# Query Timeline  
# 
```



### Impala SQL  

text 格式存储时，不支持 array 数据类型  
对于 paxxx 支持  




view 支持  

```sql
create view if not exist  u2_view as select * from u2 where name = 'd'
```



order by 排序  
NULL FIRST , NULL LAST



limit offset  

```shell script
select * from employee order by salary limit 2 offset 0;
```


### 数据导入与 JDBC 查询  

load data 方式不建议在 impala 中使用  
先使用 load dagta 导入到 hive，之后插入到 impala 表中  










## Impala 进阶

### Impala 负载均衡

对 Impalad 节点进行负载均衡  
DNS 负载均衡最简单，性能一般  

生产中选择一个非 impala 节点安装 HAProxy   



### Impala 优化

文件格式：大数据量下最佳为 Parquet    
避免小文件   
合理分区力度： 分区数量在 3W 以下造成元数据管理性能下降  

分区列数据类型最好是证书类型：  

获取表的统计指标，追求性能 / 大数据查询时，获取统计指标 compute stats   

减少客户端数据量：  

使用 explanin 查看执行计划...  







