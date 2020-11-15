[TOC]  

# Hue
交互工具查询



  


# Flume 数据采集工具
> Cloudera提供的一个**高可用的，高可靠的，分布式的海量日志采集、聚合和传输的系统**, 有效的收集、聚合、移动大量的日志数据  

基于流式架构，灵活简单。

作用： 实时读取服务器本地磁盘的数据，将数据写入到HDFS。



优点：

可与任意存储进程集成，Redis？ Kafka？

输入的速率大于写入目的存储的速率，会被缓存，减少 HDFS 压力

事务基于 channel，两个十五模型（sender + receiver），保证消息被可靠发送



## Flume 拓扑结构
穿行模式:  
复制模式： 但source, 多 channel, sink 模式  

负载均衡模式： 单 source, 。。。

聚合模式：   


一个分布式、高可靠、高可用的海量日志采集、聚合传输的系统  

**实时采集日志的数据采集引擎**  

Cloudera 公司开发  

其他数据采集工具还有：dataX、kettle、Logstash、Scribe、sqoop。

### Flume 架构

// TODO 补图







**Event**

传输单元，以事件的形式将数据从源发送到目的。

Header: 附加的信息

Body：..



### **Agent** 

JVM 进程，Flume 数据传输的基本单元。

运行在日志手机节点服务器上。



#### **Source** 

负责接收数据到 Flume Agent 的组件。  
directory   
http  
kafka  
avro  
exec  
netcagt  
souce 组件： 采集数据源  

最广泛使用的是日志文件



#### **Channel**  

位于 Source 和 Sink 之间的缓冲区:  

线程安全的，可同时处理多个 Source 的写入和多个 Sink 的读取

- memory channel:  

- file channel:  

- kafka channel:  兼具 memory, file 的优点，支持扩展。

- jdbc channel:  



允许 Souce, Sink 运行不同的速率上  
可以同时处理多个 Souce, Sink

过滤器  
处理器  
选择器  



#### **Sink**  

Sink组件用来保存数据  

不断轮询 Channel 中的事件并批量的移除 

<u>完全事务性的</u>

HDFS sink： 使用较多 
Hive sink: Hive 事务写 events  
HBase sink:  
Kafka sink:  



一般采用 HDFS Sink 都会采用滚动生成文件的方式  
- 基于时间： 缺省 30s
- 基于文件大小： 缺省 1024byte  
- 基于 event 的数量： 缺省 10
- 基于文件空闲时间: 关闭非活动文件超时时间，缺省禁用    
- 基于 HDFS 副本数: 默认与 HDFS 副本数一致，最好将参数设置为 1，避免小文件和复制的网络开销  



其他：  
是否使用本地时间戳： 默认 false, 开启  

规避小文件参数  
```yaml
a1.sinks.k1.type=hdfs
a1.sinks.k1.hdfs.useLocalTimeStamp=true
a1.sinks.k1.hdfs.path=hdfs://linux121:9000/flume/events/%Y/%m/%d/ %H/%M

a1.sinks.k1.hdfs.minBlockReplicas=1
a1.sinks.k1.hdfs.rollInterval=3600
a1.sinks.k1.hdfs.rollSize=0
a1.sinks.k1.hdfs.rollCount=0
a1.sinks.k1.hdfs.idleTimeout=0
```



### 拓扑结构  

串行模式  



复制模式： 单Souce多Channel、Sink模式  



负载均衡模式(单Source、Channel多Sink)  



聚合模式： 最常用  
传送到一个集中收集日 志的flume，再由此flume上传到hdfs、hive、hbase、消息队列中  



### 执行过程
// TODO 数据流向图  

1. Source接收事件，交给其Channel处理器处理事件  

1. 处理器通过拦截器Interceptor，对事件一些处理,比如压缩解码，正则拦截，时 间戳拦截，分类等  
2. 经过拦截器处理过的事件再传给Channel选择器，将事件写入相应的Channel。 Channel Selector有两种：
3. 最后由Sink处理器处理各个Channel的事件  



## 高级特性
### *拦截器
**时间添加戳拦截器**  

**Host添加拦截器**  

**正则表达式过滤拦截器**  

**自定义拦截器**



Event 的 body 当做 string 处理  
Q: 匹配多行数据作为实体的属性??  
类似 logback 日志信息的收集?  





生产环境中常需要自定义




### 选择器
默认为复制的  

**复制选择器（Replicating Channel Selector)**   
可通过 `selector.optional` 指定哪些 channel 可选，空格分开，失败忽略不会进行实物回滚    



**多路复用选择器(Multiplexing Channel Selector)**  
按配置进行分发  




**自定义选择器**  
类： org.apache.flume.ChannelSelector   



### Sink 组逻辑处理器
默认无 sink 组    

**负载均衡**  
轮训，随机。。。   
退避机制,backoff 设置，失败的 sink 放入黑名单 ...  


**故障转移**  
N 中只有一个在工作，其他备用    
故障sink降级到一个池  
设置sink组的选择器为failove  
为每一个sink 设置一个唯一的优先级数值  



## 事务机制与可用性  
**Put 事务:**    
Source 到 Channel 之间   





**Take 事务:**   

Channel 与 Sink 之间  

保证原子性  



source、sink 的可靠不可控，随着具体使用 Source、Channel、Sink 的类型。  

生成常用 taildir source + sink



# Sqoop ETL 工具

> 数据清洗工具，在 RDBMS 和 HDFS 之间导入导出
>
> ETL(Extraction-Tranformation-Loading) 数据提取、转换和加载

可 RDBMS 进行数据清洗  
在Hadoop(Hive)与传统的数据库(mysql、 postgresql等)间进行数据的传递。  

将导入或导出命令转换为 MapReduce 程序来实现。翻译出的 MapReduce 中主要 是对 inputformat 和outputformat 进行定制。  




## 导入
### 筛选导入


### *CDC
CDC: 数据捕获  

#### 侵入式

侵入式: 对源系统产生性能影响  

**基于时间戳的 CDC:**  
无法实时  





基于触发器的 CDC:  
Insert, update, delete 语句，将数据保存到临时表中  ...    
基本不会被采用  




基于快照的 CDC:  



#### 非侵入式

基于日志的 CDC:  
binlog...  

使用阿里 Canal 解析 MySQL binlog 日志。



## 导出

// TODO





## JOB

// TODO








## FAQ

sqoop导入hive数据时可能会报这个错误：  
ERROR Could not register mbeans java.security.AccessControlException: access denied .  
解决方案一：hive-site.xml放入sqoop的conf目录下  
解决方案二：在jdk1.8的/opt/lagou/servers/jdk1.8/jre/lib/security
找到java.policy文件，添加一句
permission javax.management.MBeanTrustPermission "register";
到grant{}里边在执行  













# Redis

// TODO





# Kafka

// TODO

