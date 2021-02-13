[TOC] 

# Flume 数据采集工具

> Cloudera提供的一个**分布式、高可靠、高可用的海量日志采集、聚合和传输的系统**, 有效的收集、聚合、移动大量的日志数据。
>
> 基于流式架构，灵活简单。
>
> **实时采集日志的数据采集引擎**

作用： 实时读取服务器本地磁盘的数据，将数据写入到HDFS。



特点

- 可与任意存储进程集成，Redis、 Kafka
- 输入的速率大于写入目的存储的速率，会被缓存，减少 HDFS 压力
- 事务基于 channel，两个十五模型（sender + receiver），保证消息被可靠发送
- 易用性： 配置较为繁琐，使用人员专业技术要求高



Flume 1.6

- 提供的 spolling directory source, exec source 不满足实时收集的需求



Flume 1.8+

- 提供 Taildir Source， 使用该 source， 可监控多个目录，对目录中心加入的数据进行实时采集



## Flume

![image-20201226102711887](http://img.janhen.com/20210130165358image-20201226102711887.png)

Agent： JVM 进程，Flume 数据传输的基本单元。运行在日志手机节点服务器上。

**Event**

传输单元，以事件的形式将数据从源发送到目的。

Header: 附加的信息

Body：..



### 拓扑结构  

串行模式  

![image-20201226103429649](http://img.janhen.com/20210130165403image-20201226103429649.png)





复制模式： 单Souce多Channel、Sink模式  

![image-20201226103451571](http://img.janhen.com/20210130165406image-20201226103451571.png)



负载均衡模式(单Source、Channel多Sink)  

解决负载 均衡和故障转移问题。

![image-20201226103514789](http://img.janhen.com/20210130165410image-20201226103514789.png)



聚合模式： 最常用  
传送到一个集中收集日志的flume，再由此flume上传到hdfs、hive、hbase、消息队列中  

![image-20201226103600691](http://img.janhen.com/20210130165413image-20201226103600691.png)



### 执行过程

![image-20201226103744350](http://img.janhen.com/20210130165418image-20201226103744350.png)

1. Source接收事件，交给其Channel处理器处理事件  

2. 处理器通过拦截器Interceptor，对事件一些处理,比如压缩解码，正则拦截，时 间戳拦截，分类等  
3. 经过拦截器处理过的事件再传给Channel选择器，将事件写入相应的Channel。 Channel Selector有两种：
4. 最后由Sink处理器处理各个Channel的事件  



## Source

> 负责接收数据到 Flume Agent 的组件。  
>
> 可以处理各种类 型、各种格式的日志数据，

- directory   
- http  
- kafka  
- avro source: 监听 Avro 端口来接收外部 avro 客户端的事件流。  
- exec: 可以将命令产生的输出作为 source。如ping、tail。 可监控实时追加的文件，无法保证数据不丢。
- Netcat source: 用来监听一个指定端口，并接收监听到的数据。
- spooling directory source: 将指定的文件加入到“自动搜集”目录中。
- taildir souce(1.7)： 监控指定的多个文件



### *Taildir source

> 1.7.0 引入，支持断点续传、可监控多目录。
>
> 相当于 spooldir source + exec source。

特点：

- 支持正则表达式匹配目录中的文件名
- 监控日志文件，有文件写入
- 高可靠，agent 重启后不会数据丢失
- 不会对跟踪文件有任何处理，不会重命名、删除
- 不支持 Windows, 不能读取二进制文件，支持按照行读取文本文件。



**配置**

```properties
a1.sources.r1.type = TAILDIR 
a1.sources.r1.positionFile = /data/lagoudw/conf/startlog_position.json 
a1.sources.r1.filegroups = f1 
a1.sources.r1.filegroups.f1 = /data/lagoudw/logs/start/.*log
```



**HDFS sink 配置**

滚动方式：

基于时间

基于文件daxiao基于 EVENT 数量

基于文件空闲时间

```properties
a1.sinks.k1.type = hdfs 
a1.sinks.k1.hdfs.path = /user/data/logs/start/%Y-%m-%d/ 
a1.sinks.k1.hdfs.filePrefix = startlog.

# 配置文件滚动方式（文件大小32M）, 默认 1024b, 生产中 >>
a1.sinks.k1.hdfs.rollSize = 33554432 
# 基于时间的数量，默认10
a1.sinks.k1.hdfs.rollCount = 0 
# 基于时间的滚动，默认30s
a1.sinks.k1.hdfs.rollInterval = 0 
# 基于文件空闲时间
a1.sinks.k1.hdfs.idleTimeout = 0 
# 默认值与 hdfs 副本数一致, 设为1是为了让 Flume 感知不到hdfs的块复制
a1.sinks.k1.hdfs.minBlockReplicas = 1

# 向hdfs上刷新的event的个数, 默认100 
a1.sinks.k1.hdfs.batchSize = 100

# 使用本地时间 
a1.sinks.k1.hdfs.useLocalTimeStamp = true
```



**agent 配置**

```properties
a1.sources=r1
a1.sinks = k1
a1.channels = c1
# taildir source
a1.sources.r1.type = TAILDIR
a1.sources.r1.positionFile = /data/lagoudw/conf/startlog_position.json
a1.sources.r1.filegroups = f1
a1.sources.r1.filegroups.f1 = /data/lagoudw/logs/start/.*log
# memorychannel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 100000
a1.channels.c1.transactionCapacity = 2000
# hdfs sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = /user/data/logs/start/%Y-%m-%d/
a1.sinks.k1.hdfs.filePrefix = startlog.
# 配置文件滚动方式（文件大小32M）
a1.sinks.k1.hdfs.rollSize = 33554432
a1.sinks.k1.hdfs.rollCount = 0
a1.sinks.k1.hdfs.rollInterval = 0
a1.sinks.k1.hdfs.idleTimeout = 0
a1.sinks.k1.hdfs.minBlockReplicas = 1
# 向hdfs上刷新的event的个数
a1.sinks.k1.hdfs.batchSize = 1000
# 使用本地时间
a1.sinks.k1.hdfs.useLocalTimeStamp = true
# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```



使用自定义拦截器

```shell
flume-ng agent --conf-file /root/lagoudw/conf/flumelog2hdfs1.conf -name a1 -Dflum e.roog.logger=INFO,console
```

报错

```
20/12/27 10:17:01 INFO hdfs.BucketWriter: Creating /user/data/logs/start/2020-12-27//startlog..1609035421335.tmp
Exception in thread "PollableSourceRunner-TaildirSource-r1" java.lang.OutOfMemoryError: GC overhead limit exceeded
        at java.util.jar.Attributes.read(Attributes.java:394)
        at java.util.jar.Manifest.read(Manifest.java:234)
        at java.util.jar.Manifest.<init>(Manifest.java:81)
        ...
```

处理

```shell
vi $FLUME_HOME/conf/flume-env.sh 

export JAVA_OPTS="-Xms2048m -Xmx2048m -Dcom.sun.management.jmxremote"


# 使配置文件生效，还要在命令行中指定配置文件目录 
flume-ng agent --conf $FLUME_HOME/conf \
	--conf-file /root/lagoudw/conf/flume-log2hdfs1.conf \
	-name a1 -Dflume.roog.logger=INFO,console
```

```shell
flume-ng agent --conf $FLUME_HOME/conf \
	--conf-file /root/lagoudw/conf/flume-log2hdfs2.conf \
	-name a1 -Dflume.roog.logger=INFO,console
```





## Channel 

>  位于 Source 和 Sink 之间的缓冲区:  
>
> 线程安全的，可同时处理多个 Source 的写入和多个 Sink 的读取

- memory channel:  缓存到内存中，宕机数据丢失

- file channel:  缓存到文件中

- **<font color="blue">kafka channel</font>**:  缓存到 Kafka 中。兼具 memory, file 的优点，支持扩展。速度快，容量大。

- jdbc channel:  缓存到关系型数据库中



允许 Souce, Sink 运行不同的速率上  

可以同时处理多个 Souce, Sink

- 过滤器  
- 处理器  
- 选择器  



### *拦截器

> 在运行时对event进行修改或丢弃

**时间添加戳拦截器**

向每个event的header中添加一个时间戳属性进去，key默认是 “timestamp ”，value就是当前的毫秒 

```properties
a1.sources.r1.interceptors = i1 
a1.sources.r1.interceptors.i1.type = timestamp
# 是否保留Event header中已经存在的同名时间戳，缺省值false
a1.sources.r1.interceptors.i1.preserveExisting= false
# 向event header中添加时间戳键值对的key
a1.sources.r1.interceptors.i1.header= time
```





**Host添加拦截器:** 

把当前Agent的 hostname 或者 IP 地址写入到Event的header中

`type`: host

`preserveExisting`: header 中存在同名属性是否保留，默认 false

`userIp`: ip 地址，默认 true, 为 false 时使用 hostname

`hostHeader`: 默认为 host, Event header 中添加 host 键值对的 key



**正则表达式过滤拦截器:**  

把Event的body当做字符串来处理，并用配置的正则表达式来匹配。



**自定义拦截器**

生产环境中常需要自定义拦截器, `org.apache.flume.interceptor.Interceptor`

```java
public interface Interceptor {
  void initialize();
  Event intercept(Event var1);
  List<Event> intercept(List<Event> var1);
  void close();
  public interface Builder extends Configurable {
    Interceptor build();
  }
}
```




### 选择器

> 确定 source 以何种方式向多个 channel 写， 默认为复制的  

**复制选择器（Replicating Channel Selector)**   
可通过 `selector.optional` 指定哪些 channel 可选，空格分开，失败忽略不会进行实物回滚    

```properties
a1.sources.r1.selector.type = replicating
```

**多路复用选择器(Multiplexing Channel Selector)**  
按配置进行分发  





**自定义选择器**  
接口 `org.apache.flume.ChannelSelector` 的实现类, 实现类以及依赖的jar包在启动时候都必须放入Flume的classpath。

```properties
a1.sources = r1 
a1.channels = c1 
a1.sources.r1.selector.type = com.janhen.bigdata.flume.MyChannelSelector
```



## Sink  

>  Sink组件用来保存数据  ，不断轮询 Channel 中的事件并批量的移除。
>
>  将这些事件批量写入到 存储或索引系统、或者被发送到另一个Flume Agent。 

<u>完全事务性的</u>

- HDFS sink： 使用较多 
- Hive sink: Hive 事务写 events  
- HBase sink:  
- Kafka sink:  
- Logger sink: 信息显示在标准输出上，用于测试。
- Avro sink: 转换为 Avro events



### HDFS Sink

滚动生成文件配置

- 基于时间： `hdfs.rollInterval` 缺省 30s
- 基于文件大小： `hdfs.rollSize` 缺省 1024byte  
- 基于 event 的数量：`hdfs.rollCount`  缺省 10
- 基于文件空闲时间:  `hdfs.idleTimeout` 关闭非活动文件超时时间，缺省禁用    
- 基于 HDFS 副本数:  `hdfs.minBlockReplicas` 默认与 HDFS 副本数一致，最好将参数设置为 1，避免小文件和复制的网络开销  

```properties
a2.sinks.k2.hdfs.path = hdfs://linux121:9000/flume/%Y%m%d/%H%M
# 上传文件的前缀
a2.sinks.k2.hdfs.filePrefix = log

# 是否使用本地时间戳
a2.sinks.k2.hdfs.useLocalTimeStamp = true
# 积攒500个Event才flush到HDFS一次
a2.sinks.k2.hdfs.batchSize = 500
# 设置文件类型，支持压缩。DataStream没启用压缩
a2.sinks.k2.hdfs.fileType = DataStream
# 1分钟滚动一次
a2.sinks.k2.hdfs.rollInterval = 60
# 128M滚动一次
a2.sinks.k2.hdfs.rollSize = 134217700
# 文件的滚动与Event数量无关
a2.sinks.k2.hdfs.rollCount = 0
# 最小冗余数
a2.sinks.k2.hdfs.minBlockReplicas = 1
```



其他参数：  

- `hdfs.useLocalTimeStamp` 是否使用本地时间戳： 默认 false, 开启  

- `hdfs.round`: 时间戳是否四舍五入, 默认 false

- `hdfs.roundValue`:

- `hdfs.roundUnit`:

  

规避小文件参数  

```properties
a1.sinks.k1.type=hdfs
a1.sinks.k1.hdfs.useLocalTimeStamp=true
a1.sinks.k1.hdfs.path=hdfs://linux121:9000/flume/events/%Y/%m/%d/ %H/%M

a1.sinks.k1.hdfs.minBlockReplicas=1
a1.sinks.k1.hdfs.rollInterval=3600
a1.sinks.k1.hdfs.rollSize=0
a1.sinks.k1.hdfs.rollCount=0
a1.sinks.k1.hdfs.idleTimeout=0
```



### Sink 组逻辑处理器

> 把多个sink分成一个组， Sink组逻辑处理器可以对这同一个组里的几个sink进行 负载均衡 或者 其中一个sink发生故障后将输出Event的任务转移到其他的sink上。
>
>  默认无 sink 组    

**负载均衡**  
轮训，随机。。。   
退避机制,backoff 设置，失败的 sink 放入黑名单 ...  



**故障转移**  
N 中只有一个在工作，其他备用    
故障sink降级到一个池  
设置sink组的选择器为failove  
为每一个sink 设置一个唯一的优先级数值  





## 其他

### *事务机制与可用性  

> 保证数据在 Flume 流动过程中数据不丢失。

**Put 事务:**    

> Source 到 Channel 之间    

把这一批 Event 放到一个事务中，这批event一次性的放入Channel 中。



**Take 事务:**   

>  Channel 与 Sink 之间  

把一批event组成的事务统一拿出来到 sink 放到 HDFS 上。

source、sink 的可靠不可控，随着具体使用 Source、Channel、Sink 的类型。  

生成常用 taildir source + sink

doTake方法会将channel中的event剪切到takeList中。



### Flume 命令操作

Agent JVM heap 一般为 4G ~ 8G，部署在单独的服务器上。

```SHELL
$FLUME_HOME/bin/flume-ng agent --name a1 \
  --conf-file $FLUME_HOME/conf/flume-netcat-logger.conf \
  -Dflume.root.logger=INFO,console
```

```shell
export JAVA_OPTS="-Xms4000m -Xmx4000m -Dcom.sun.management.jmxremote"
flume-ng agent --conf /opt/apps/flume-1.9/conf \
  --conf-file /data/lagoudw/conf/flume-log2hdfs1.conf \
  -name a1 -Dflume.roog.logger=INFO,console
```

```shell
flume-ng agent --conf /opt/apps/flume-1.9/conf \
  --conf-file conf/flume-log2hdfs1.conf \
  -name a1 -Dflume.root.logger=INFO,console
```



### 数据采集优化

- channel 选取 Kafka, 兼具 File Channel

- 配置负载均衡

- 替换成其他的日志采集组件，Logstash, FileBeat。

- 在 Flume 中过滤数据，只收集 json 数据。





## Refs

- [中文flume帮助文档](https://flume.liyifeng.org/)




