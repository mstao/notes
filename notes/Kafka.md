[TOC]

Topic: 主题

Brokder:

Partition: 分区

Consumer Group: 避免被重复的处理





核心 API:

Produer API:

Consumer API:

Streams API:

Connector API:



消息中间件的模式，<font color="green">只有消息的拉取，没有推送</font>，可通过轮询实现消息的推送。



**优点**

持久化存储： 零拷贝、顺序读与顺序写、Linux 的页缓存



**应用场景**

日志、消息

用户活动跟踪

运营指标

流式处理





## **基本架构**

> 一个分布式、分区的、多副本的、多生产者、多订阅者，基 于zookeeper协调的分布式日志系统, 常见可以用于web/nginx日志、访问日 志，消息服务等。

支持**<font color="green">点对点传递模式、发布-订阅模式</font>**的消息传递模式。

每个记录由一个键，一个值和一个时间戳组成。

消息被分批写入Kafka。

分批发送： 批次为一组消息，消息属于同一个主题和分区。可对批次进行压缩。吞吐量大。

测试 Kafka 处理数据 200G/h。



两个反常识：

消息磁盘存储，非内存

分区副本保持数据一致性(高可用保证)，非过半机制，随机选择一个 ISR，依据配置可选一个 USR。



可对生产者、消费者、 Brokder、主题的分区进行横向扩展。

consumer 采用 pull 模式从 brokder 中读取数据，<font color="green">可自主控制消费消息的速率, 可控制消费方式.. </font>

推送, 保证及时性, RabbitMq 保证 um, 小而美。



<img src="http://img.janhen.com/image-20201121171728810.png" alt="image-20201121171728810" style="zoom: 50%;" />

Producer:

给定key, 无 key, 自定义分区器



Consumer:

再平衡： 某个 Consumer 挂掉，对多个分区进行重新分配。



Broker:

对与  topic 的 N partition，与 brokder 的数量...

集群控制器： 分区的分配、监控 broker



Broker 与集群： 单个 Brokder 可处理多个分区和每秒百万级的消息量。



集群管理器 Brokder： 监控 brokder



Topic:

类似 DB 的表，分库分表的<font color="green">逻辑表</font>。



Partiton:

主题实际存储的分片, 无法保证主题范围内消息的顺序，通过分区实现数据冗余与伸缩。

生产端通过轮询、哈希(基于Key)、指定放到特定分区(基于分区号)。

Replicas：

首领副本、追随者副本。主题分区的，不让相同分区的副本在同一台机器上。

Offset：

递增序列，可指定从0开始消费，或者指定的偏移量进行消费。



副本的细分：

分区的所有副本为 AR(Assigned Replicas)

同步副本 ISR(In-Sync Replicas)

不同步副本 OSR(Out-Sync Replicas)



偏移量 HW: 决定对外提供消息的最大偏移量，再 HW 前可见、可消费，之后不允许消费

下一条待写入消息的 offset(LEO): 



Consumer:

可横向扩展，需要保证不能够重复消费。一个分区只能被一个消费组中的一个消费者消费。



Consumer Group:

同一个消费组中的消费者不会对 Topic 的消息进行重复消费。

消费组之间...



## 安装



# 高级特性

## 生产

<img src="http://img.janhen.com/image-20201116223818141.png" alt="image-20201116223818141" style="zoom: 50%;" />

<p align="center">生产者发送过程</p>

### 序列化器

自定义序列化器需要实现 `org.apache.kafka.common.serialization.Serializer<T>` 接口，并实现其 中的 serialize 方法。

Avro 提供了一种紧凑的序列化格式，模式和消息体分开。当模式发生变化时，不需要重新生成代码，它还支持强类型和模式进化，其版本既向前兼容，也向后兼容。



### 分区器

生产端控制消息发送到哪个分区，有轮询、哈希、指定分区号等方式。



### 拦截器

Interceptor 可能被运行在多个线程中，在具体实现中**<font color="green">需要自行保证线程安全性</font>**



### 原理分析

<img src="http://img.janhen.com/image-20201117215502433.png" alt="image-20201117215502433" style="zoom: 60%;" />

<p align="center">生产者执行原理</p>

两个线程：

① 主线程：负责消息创建，拦截器，序列化器，分区器等操作，并将消息追加到消息收集器 RecoderAccumulator中

② Sender 线程：



### +消息确认

Request.required.acks

- 0 

- 1

- -1

- all





## 消费

消费消息的偏移量保存在Kafka的名字是 __consumer_offsets 的 主题中。



**消费组**

设置 gorup.id ，保证在一个消费族的时候



### **反序列化**

自定义需要实现 `org.apache.kafka.common.serialization.Deserializer<T>` 接口。

```
ByteBuffer.wrap(data);
```





### 拦截器

按照顺序执行

于将第三方组件引入 消费者应用程序，用于定制的监控、日志处理等。

ConsumerInterceptor 方法抛出的异常会被捕获、记录，但是不会向下传播。如果用户配置 了错误的key或value类型参数，消费者不会抛出异常，而仅仅是记录下来。



### **偏移量**

kafka会定期把group消费情况保存起来，做成一个offset map。

有默认的主题 `__consumer_offsets` 控制偏移量，位移条有同步和异步提交。

(1) 自动提交

可能导致重复消费

<font color="green">自动提交不会出现消息丢失，但会重复消费</font>



(2) 异步提交

异步提交不会进行自动重复。

推荐的方式，<font color="green">大部分进行异步，每个一段时间/数据进行一次同步提交。</font>

```java
try {
  while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
    // 处理消息 
    process(records); 
    // 使用异步提交规避阻塞 
    commitAysnc();
  }
} catch (Exception e) {
  handle(e); // 处理异常 
} finally {
  try {
    consumer.commitSync(); // 最后一次提交使用同步阻塞式提交 
  } finally {
    consumer.close();
  }
}
```

```SHELL
for i in `seq 60`; do echo "hello lagou $i" >> nm.txt; done
kafka-topics.sh --zookeeper linux121:2181/myKafka \
  --create --topic tp_demo_01 \
  --partitions 3 --replication-factor 1
kafka-console-producer.sh --broker-list linux121:9092 --topic tp_demo_01 < nm.txt
```

```shell
# 查看偏移量信息
kafka-console-consumer.sh --topic __consumer_offsets \
  --bootstrap-server linux121:9092 \
  --formatter "kafka.coordinator.group.GroupMetadataManager\$OffsetsMessageFormatter" \
  --consumer.config $KAFKA_HOME/config/consumer.properties \
  --from-beginning | head
```

```SHELL
# 查看有那些 group ID 正在进行消费
kafka-consumer-groups.sh --bootstrap-server linux121:9092 --list

# 查看偏移量信息
kafka-consumer-groups.sh --bootstrap-server linux121:9092 -describe --group user_consumer
# 偏移量设置为最早的
kafka-consumer-groups.sh --bootstrap-server linux121:9092 \
  --reset-offsets --group user_consumer \
  --to-earliest --topic tp_demo_02
# 偏移量设置为最新的
kafka-consumer-groups.sh --bootstrap-server linux121:9092 \
  --reset-offsets --group user_consumer \
  --to-latest --topic tp_demo_02
# 将指定主题的指定分区的偏移量向前移动10个消息
kafka-consumer-groups.sh --bootstrap-server linux121:9092 \
  --reset-offsets --group user_consumer \
  --shift-by 10 --topic tp_demo_02:2
kafka-consumer-groups.sh --bootstrap-server linux121:9092 \
  --reset-offsets --group user_consumer \
  --shift-by -10 --topic tp_demo_02:1
```

 



### **消费组管理**

满足正则表达式的 Topic 的匹配，需要弹性扩展。



**消费组的状态**

<img src="http://img.janhen.com/20210130164345image-20201118204202363.png" alt="image-20201118204202363" style="zoom: 50%;" />

Dead：组内已经没有任何成员的最终状态

Empty：组内无成员

PreparingRebalance：组准备开启新的rebalance，等待成员加入

AwaitingSync：正在等待leader consumer将分配方案传给各个成员

Stable：再均衡完成





### **再平衡**

一种协议

RangeAssignor 范围划分， RoundRobin 轮训、StickyAssignor 分配。

控制一个主题的多个分区对应一个消费组中的多个消费者。





**Group Coordinator**

为一个 Brokder

__consumer_xx partition 在哪， 对应的分区在哪，组协调器。







**Rebalance Generation**

主题分区与消费主中的映射关系的一个版本，主要<font color="green">隔离 xxx 无效的请求。</font>





**协议**

Heatbeat 请求

LeaveGorup

SyncGroup

JoinGroup

DescribeGroup: 用于管理

组协调器在再平衡中使用到前四种请求。









**再平衡的过程**

 SyncGroup

版本之间的演进，<font color="green">由客户端进行控制</font>，提供更好的灵活性。







## 主题

对应保存在 Zookeeper 的 `/<base-dr>/config/topics/<topic-name>` 下

```shell
# 创建
kafka-topics.sh --zookeeper linux121:2181/myKafka \
  --create --topic tp_test_01 \
  --partition 3 \
  --replication-factor 1
kafka-topics.sh --zookeeper linux121:2181/myKafka \
  --create --topic topic_test_02 \
  --partitions 3 --replication-factor 1 \
  --config max.message.bytes=1048576 \
  --config segment.bytes=10485760
# 查看指定 <topic-name> 的信息
kafka-topic.sh --zookeeper linux121:2181/myKafka \
  --describe --topic tp_test_01
  
# 查看覆盖过默认配置的主题
kafka-topics.sh --zookeeper linux121:2181/myKafka \
  --describe --topics-with-overrides
  
# 更改 Topic 的配置
kafka-topics.sh --zookeeper localhost:2181/myKafka \
  --alter --topic topic_test_01 \
  --config max.message.bytes=1048576
kafka-topics.sh --zookeeper localhost:2181/myKafka \
  --alter --delete-config max.message.bytes \
  --topic topic_test_01
  
# 删除 Topic,  先标记 后删除
kafka-topics.sh --zookeeper localhost:2181/myKafka --delete --topic topic_x
```









## 分区

> 为了做到均匀分布，通常partition的数量通常是 Broker Server数量的整数倍。

### *副本机制

允许 Follower 进行<font color="green">批处理</font>。

Follower 分区和普通 Kafka 消费者一样，消费 Leader 分区消息，并将其持久化到自己的日志。



**分区副本在 Broker 上的存放(LJ)**

副本因子

(1) 不考虑机架的分配方案：

第一轮顺序放

第二轮，Partiton1 --> Brokder1，Partition2 --> Brokder2，Partiton3 --> Broder3

第三轮, Partiton1 -->  Brokder, Partition2 --> Brokder3, Partiton3 --> Brokder4

(2) 考虑机架的分配情况

按照基本规则分配，达到特定情况区分于基本规则配置，防止某几个 Broker 总是出现问题。



**出现问题时选取 Partition**

由集群控制器控制。

① 有 ISR 存在，随机找一个 ISR 进行恢复

② 所有的 ISR 都宕机，等一个 ISR 恢复

③ 所有的 ISR 都宕机，在配置可以选择 OSR 作为 Leader 时选一个 OSR 



### 分区重新分配

> 向已经部署好的Kafka集群里面添加机器，从已经部署好的Kafka节点中复制相应的配置 件，把里面的broker id修改成全局唯一的，最后启动这个节点即可将它加入到现有Kafka集群中。
>
> ==>> 新添加的Kafka节点并不会自动地分配数据，无法分担集群的负载

**手动再均衡**

kafka-reassign-partitions.sh 重新分布分区, 三种模式, plan, execute, verify。

定义哪些 topic 需要重新分区, 文件 topics-to-move.json {

```json
{
  "topics": [
    {
      "topic":"tp_re_01"
    }
  ],
  "version": 1
}
```
执行分配，可根据需要修改生成后建议的计划。
```shell
kafka-reassign-partitions.sh --zookeeper node1:2181/myKafka 
  --topics-to-move-json-file topics-to-move.json \
  --broker-list "0,1" \
  --generate 

# 再分配, 给出建议的分配结果
kafka-reassign-partitions.sh --zookeeper linux121:2181/myKafka \
  --reassigment-json-file topics-to-execute.json \
  --execute
  
  
# 验证, 数据较多的情况下验证是否执行完毕
kafka-reassign-partitions.sh --zookeeper linux121:2181/myKafka \
  --reassigment-json-file topics-to-execute.json \
  --verify
```



**自动再均衡**

```shell
kafka-topics.sh --zookeeper node1:2181/myKafka 、
  --create --topic tp_demo_03 、
  --replica-assignment "0:1,1:0,0:1"
```

kafka-preferred-replica-election.sh 用于自动再均衡。

```shell
kafka-preferred-replica-election.sh --zookeeper linux121:2181/myKafka \
  --path-to-json-file preferred-replica.json
```

**修改副本因子**

```json
{
	"version": 1,
	"partitions": [

		{
			"topic": "tp_re_02",
			"partition": 0,
			"replicas": [0, 1]
		},

		{
			"topic": "tp_re_02",
			"partition": 1,
			"replicas": [0, 1]
		},

		{
			"topic": "tp_re_02",
			"partition": 2,
			"replicas": [1, 0]
		}
	]
}
```

```shell
kafka-reassign-partitions.sh --zookeeper linux121:2181/myKafka \
  --reassignment-json-file increase-replicationfactor.json \
  --execute
```



### +分区与消费者的再分配

> 再平衡的时候无法进行消费，导致消费能力下降。
>

**触发条件：**

① 消费者组内成员发生变更： 增减消费者

② 主题的分区数发生变更： 增加主题的分区

③ 订阅的主题发生变化： 消费组通过正则订阅主题, 消费组订阅的 Topic 数量变更



**再平衡**

某个消费者挂掉

消费者进行横向扩展

<font color="green">分区不能被多个消费者消费</font>, Topic 的分区越多，越有利于消费组的扩展。



**心跳机制**

Kafka 的心跳是 Kafka Consumer 和 Broker 之间的健康检查，只有当 Broker Coordinator 正常 时，Consumer 才会发送心跳。

控制Topic 的分区与消费组中的消费者对应关系



**再平衡的影响：**

重平衡过程中，<font color="green"><strong>消费者无法从kafka消费消息，对kafka的 TPS影响极大，重平衡可能会耗时极多，这段时间kafka基本处于不可用状态。</strong></font>



**再平衡的过程**

在协调器收集到所有成员请求前，它会把已收到请求放入一个叫purgatory(炼狱)的地方。

<font color="green">消费组的分区分配方案在客户端执行</font>。Kafka交给客户端可以有更好的灵活性。



**分区分配策略**

<font color="green">Kafka 默认采用 RangeAssignor 分配算法</font>

1、RangeAssignor

偏向于字典序小的消费组中的消费者比较 "贪婪"

```scala
  assign(topic, consumers) { // 对分区和Consumer进行排序 
    List<Partition> partitions = topic.getPartitions();
    sort(partitions);
    sort(consumers); // 计算每个Consumer分配的分区数 
    int numPartitionsPerConsumer = partition.size() / consumers.size(); // 额外有一些Consumer会多分配到分区 
    int consumersWithExtraPartition = partition.size() % consumers.size();
    // 计算分配结果 
    for (int i = 0, n = consumers.size(); i < n; i++) { // 第i个Consumer分配到的分区的index 
      int start = numPartitionsPerConsumer * i + Math.min(i, consumersWithExtraPartition); 
      // 第i个Consumer分配到的分区数 
      int length = numPartitionsPerConsumer + (i + 1 > consumersWithExtraPartition ? 0 : 1);
      // 分装分配结果
      assignment.get(consumersForTopic.get(i)).addAll(partitions.subList(start, start + length));
    }
  }
```

2、RoundRobinAssignor

轮询的方式进行指定。

3、StickyAssignor

<font color="green">分区的分配尽量的均衡，每一次重分配的结果尽量与上一次分配结果保持一致,</font>和 HashMap 的 rehash 对低位的不变相同思想。

4、自定义分配策略



**可控制再平衡的三个参数**

session.timout.ms: 控制心跳的超时时间, 6s 

heartbeat.interval.ms: 心跳的频率, 2s, 一般为上面值的三分之一

max.poll.interval.ms: poll 的间隔, 1分钟



### +提高Kafka速度

设置 ack = -1, 所有的 ISR 都发送后才...

提高 batchsize

增加 Partition、增加 Producer 实例



### Zookeeper

元数据存储

消费者的消费状态

```
brokders/ids
```





## 物理存储

### 存储机制

**LogSegment**

<font color="green">顺序写入</font>

基准偏移来那个，日志文件名称为<font color="green">第一条消息的偏移量</font>



文件的作用：

.index: 偏移量索引, RandomReader.seek(xx)

.timestamp: 时间戳索引文件，一般服务器生成, 客户端使用可能时间不一致

.log:

.snapshot:

.deleted:

.cleaned:

.swap:



**文件的切分**



**索引文件切分**

预分配大小，实际切分的时候进行实际的大小，<font color="green">降低代码逻辑的复杂性。</font>

offset 实际长度 64位，默认日志文件使用 20位

```shell
# 常见测试主题
kafka-topics.sh --zookeeper linux121:2181/myKafka --create -topic tp_demo_05 --partitions 1 --replication-factor 1 --config segment.bytes=104857600

# 生成消息
for i in `seq 10000000`; do echo "hello lagou $i" >> nmm.txt; done

# 文本文件导入到 kafka 中
kafka-console-producer.sh --broker-list linux121:9092 --topic tp_demo_05 <nmm.txt

kafka-run-class.sh kafka.tools.DumpLogSegments --files 00000000000000000000.log --print-data-log | head
```



**索引文件**

偏移量索引。相对偏移量(4B)和物理地址(4B)组成。

<font color="green">索引文件是稀疏索引，不会为每条日志都建立索引信息。</font>

```shell
# 查看偏移量索引文件
kafka-run-class.sh kafka.tools.DumpLogSegments --files 00000000000000000000.index --print-data-log | head
```

offset 与 position 没有直接关系，因为会删除数据和清理日志。



Q:  如何查看偏移量为23的消息？

Kafka 中存在一个 ConcurrentSkipListMap 来保存在每个日志分段，通过跳跃表方式，定位到在 00000000000000000000.index ，通过二分法在偏移量索引文件中找到不大于 23 的最大索引项，即 offset 20 那栏，然后从日志分段文件中的物理位置为320 开始顺序查找偏移量为 23 的消息。



**时间戳索引文件**

通过时间戳方式进行查找消息，需要通过查找时间戳索引和偏移量索引<font color="green">两个文件</font>。

Q: 查找时间戳为 1557554753430 开始的消息？

<font color="green">timestamp文件中的 offset 与 index 文件中的 relativeOffset 不是一一对应的。因为数据的 写入是各自追加。</font>



**日志清理**

删除、压缩

Log.cleanup.policy: 默认 delete, 可选择 compact，业务数据一般不删除。

基于时间删除

log.retention.hours/log.retention.minutes/log.retention.ms，默认 7 days。



### *磁盘存取

**零拷贝**

```
内核缓冲区 ==> 用户缓冲区(JVM 堆内存) ==> 内核缓冲区 ==> 网卡...
内核缓冲区 ==> 网卡
```



**页缓存**

操作系统级别的特性，实现的磁盘缓存。

<font color="green">Mmap 不可靠</font>， NIO 提供 MappedByteBuffer 实现映射。

mmap 的文件映射，在 Full GC 时才会进行释放， close 时需要手动清理内存映像文件。

<font color="green">消息先被写入页缓存，由操作系统负责刷盘任务。</font>判断是否有<font color="green">脏页</font>, 通过 Swap 虚拟内存。

内核缓冲区，启动异步线程进行落盘，**<font color="green">在生产和消费速率趋近一致时，页缓存中有需要的数据，直接从内核缓冲区区数据，从网卡发送出去，不需要访问磁盘。</font>**



**顺序写入机制**

多个文件相同的大小与一个文件进行传输，一个文件更快，读取的时候效率更高。

RocketMQ 在消费消息时，使用了 mmap。kafka 使用了 sendFile。



**Kafka 使用磁盘存储速度快的原因**

1. partition 顺序读写，充分利用磁盘特性；
2. Producer生产的数据持久化到broker，采用mmap文件映射，实现顺序的快速写入；
3. Customer从broker读取数据，采用 <font color="green">sendfile</font>，将磁盘文件读到OS内核缓冲区后，直接转到 socket buffer进行网络发送。







## 稳定性

### 事务

事务的使用场景：

分布式事务，引入 RPC，通过食物协调者。。。

需要一个内部 topic 保存事务日志， __transaction_state，一般存在时间短。

消息队列需要表示事务状态，Control Message。

TransactionId 控制 producer 挂掉或漂移到其他机器。

producer epoch 保证只有一个 TransactionId 进行消费。



**粉碎 "僵尸实例"**



**事务组**



**生产者 ID 与 事务组状态**



**事务协调器**

为 __transaction_state 主题特定个分区的 Leader 分区的 Broker。

存储正在处理的事务。

确保何种保留策略，都不会删除事务的消息。



**事务流程**

每个消息都有 TxId 和 TxCtl 字段。



**事务的终止**

发送业务消息的时候发生异常。



**主题的压缩**

会丢弃具有相同键的早期记录。



**事务相关的配置**

Transaction.id.timeout.ms: 事务 ID 的最长时间，默认 7 day。

Max.transaction.timeout.ms: 默认 15分钟

Transaction.state.log.replication.factor: 事状态 topic 的副本数量，默认 3.

Transaction.state.log.num.partitions: 默认50

TRANSACTION.STATTE.log.min.isr: 

..



生产者配置

transactional.id: 

..



消费组配置

Isolation.level: 级别





### **-幂等性**

> 防止重复消费，

<font color="green">为消息加一个唯一的标记</font>。

保证消息重发的时候，消费者不会重复处理，即使进行重复消息，要保证最终结果的一致性。

```
f(f(x)) = f(x)
```



数据的事务语义：

最多一次: 

最少一次: 失败重发(重试队列)

仅一次



幂等性实现：

ProducerID:

SequenceNumber:

```java
private void maybeWaitForPid() {

  if (transactionState == null) return;

  while (!transactionState.hasPid()) {
    try {
      Node node = awaitLeastLoadedNodeReady(requestTimeout);
      if (node != null) {
        ClientResponse response = sendAndAwaitInitPidRequest(node);
        if (response.hasResponse() && (response.responseBody() instanceof InitPidResponse)) {
          InitPidResponse initPidResponse = (InitPidResponse) response.responseBody();

          transactionState.setPidAndEpoch(initPidResponse.producerId(), initPidResponse.epoch());
        } else {
          log.error("Received an unexpected response type for an InitPidRequest from {}. " + "We will back off and try again.", node);
        }
      } else {
        log.debug("Could not find an available broker to send InitPidRequest to. " + "We will back off and try again.");
      }
    } catch (Exception e) {
      log.warn("Received an exception while trying to get a pid. Will back off and retry.", e);
    }
    log.trace("Retry InitPidRequest in {}ms.", retryBackoffMs);
    time.sleep(retryBackoffMs);
    metadata.requestUpdate();
  }
}
```





### 集群控制器

> 为一个 Broker，负责 Leader 分区的选举。

避免选择多个 Leader, 或者 Controller 再次上线。

==> 通过 Zookeeper 的分布式锁实现，进行选举控制器，监听 ids 。

控制器使用 <font color="green">epoch 来避免 "脑裂"</font>。

<font color="green">控制器需要那个 Broder 宕机了，宕机的 Broder 上负责的哪些分区的 Leader 副本分区。</font>

变动的临时节点 xx/broders/ids。

监听 ids 的节点，监听到哪些节点宕机。

xx/broders/topic: 非临时节点。

xx/broders/seqid: 

```
get /myKafka/brokers/ids/
```



**可靠性**

ISR: 

USR：

`replica.lag.time.ms`: 默认 10000ms，落后改时间，Follower 没有向 Leader 发送 fetch 请求。

`acks=all` 保证可靠性



**失效副本**

可通过编程方式获取

Follower 副本进程卡住，同步太慢...



**副本的复制**

提高高可用。



**副本角色**

LEOLog end offset，日志末端位移
HW: HW 不会大于 LEO 值

<img src="http://img.janhen.com/image-20201120222351177.png" alt="image-20201120222351177" style="zoom:50%;" />

Leader 收到消息更新自己的 LEO，在 Follower 执行 FETCH 请求的时候更新 Leader 中的 Follower 中的 LEO。

如果Follower的LEO大于Leader的HW，Follower HW值不会大于Leader的HW值。

Kafka 通过 HW 

数据丢失

Min.insync.replicas=1



**数据的离散**

Follow 与 Leader 的 中的数据顺序不一致。

<font color="green">延迟一轮 FETCH RPC 更新HW值的设计使follower HW值是异步延迟更新</font>



### Epoch

HW 的值是异步延迟的..

实际上是一堆值 `<epoch, offset>`

epoch 未 Leader 的版本好，Leader 变更一次， + 1。



依靠 Leader epoch 的信息可以有效地规避数据不一致的问题。

对于使用 `unclean.leader.election.enable = true` 设置的群集，该方案不能保证消息的一致性。

<font color="green">Kafka 通过  ISR 保证一致性， Zookeeper 通过过半机制保证数据一致性。</font>



### 重复消费

**生产阶段重复**

<img src="http://img.janhen.com/image-20201120230200106.png" alt="image-20201120230200106" style="zoom:80%;" />

Max.in.flight.requests.per.connnection 默认 5，单个连接上发送的未确认请求的最大数量。



**消费阶段重复**

消费者没有及时的提交 offset



### __consumer_offsets

> 默认 50 个 partition， Kafka 的集群控制器管理。

```
kafka-consumer-groups.sh --bootstrap-server linux121:9092 \
  --describe --group console-consumer-xx
```

```shell
# 指定格式化查看
kafka-console-consumer.sh --topic __consumer_offsets -bootstrap-server linux121:9092 --formatter "kafka.coordinator.group.GroupMetadataManager\$OffsetsMessageFormatter" -consumer.config config/consumer.properties --from-beginning
```

指定 consumer group 的位移信息。

```shell
kafka-simple-consumer-shell.sh \
  --topic __consumer_offsets \
  -partition 19 \
  --broker-list linux121:9092 \
  --formatter "kafka.coordinator.group.GroupMetadataManager\$OffsetsMessageFormatter"
```

```shell
...
[console-consumer-49366,tp_test_01,3]:: [OffsetMetadata[20,NO_METADATA],CommitTime 1596424702212,ExpirationTime 1596511102212] 
[console-consumer-49366,tp_test_01,4]:: [OffsetMetadata[20,NO_METADATA],CommitTime 1596424702212,ExpirationTime 1596511102212]
...
```



## 延时队列

> Kakfa 内部实现。

延迟拉取操作(DelayedFetch) 等待拉取到足够数量的消息。

Request.timeout.ms 默认30s，进行消息确认。                                                                                                            

没有消息放到炼狱中，等有消息后恢复 Follower。



## 重试队列

> kafka没有重试机制不支持消息重试，也没有死信队列，因此使用kafka做消息队列时，需要自己实现消息重试的功能。
>
> RabbitMQ 中自带重试队列，Kafka 中需要自定义实现重试队列。

实现重试队列：

创建新的kafka主题作为重试队列：

1. 创建一个topic作为重试topic，用于接收等待重试的消息。
2. 普通topic消费者设置待重试消息的下一个重试topic。
3. 从重试topic获取待重试消息储存到redis的zset中，并以下一次消费时间排序
4. 定时任务从redis获取到达消费事件的消息，并把消息发送到对应的topic
5. 同一个消息重试次数过多则不再重试



数据量大的情况下， Redis 中存放不了，可考虑使用 HBase 或 MySQL 实现。

大数据中比较少使用到 Kafka 的重试队列。





### 命令

**服务的启停**

```shell
# 前台启动
kafka-server-start.sh -daemon $KAFKA_HOME/config/server.properties
kafka-server-stop.sh

# 后台启动
curl linux123/log -d "message send to kafka ..0000000088888"

vi $KAFKA_HOME/config/server.properties
bootstrap.servers=linux121:9092,linux122:9092,linux123:9092
```

**主题操作**

```shell
# 查看主题
kafka-topic.sh --zookeeper loclhost:2181/myKafka --list

# broker count ====> replication-factor
kafka-topic.sh --zookeeper localhost:2181/myKafka \
  --create --topic topic_1 \
  --partitons 1 \
  --replication-factor 1
  
kafka-topic.sh --zookeeper localhost:2181/myKafka \
  --describe --topic topic_1
# 调高主题的分区数
```

**消费者操作**

```shell
kafka-console-consumer.sh --bootstrap-server localhost:9092 \
  --topic topic_1
  
# 从头消费
kafka-console-consumer.sh --bootstrap-server localhost:9092 \
  --topic topic_1 \
  --from-beginning
```

**生产者操作**

```shell
kafka-console-producer.sh --brokder-list localhost:9092 \
  --topic topic_1
```

<img src="http://img.janhen.com/image-20201116213957800.png" alt="image-20201116213957800" style="zoom: 67%;" />

acks: 副本分区进行...， 保证可靠性，    同步确认/异步确认

Retries: 重试的次数..



### 配置

**内外网隔离配置**

**listener**: 用于指定当前Broker向外发布服务的地址和端口。与 advertised.listeners 配合，用于做内外网隔离。

listerner.security.protocol.map:

inter.broker.listener.name

listeners

advertised.listeners

配置监听器名称与安全协议的映射关系, 内外部的监听器。

Broader.id: 建议与主机相关

Log.dirs: 持久化的数据目录, 可指定多个

**partition**

在需要严格保证消息的消费顺序的场景下，需要将partition数目设为1。





bootstrap.servers

key.deserializer

value.deserializer

interceptor.classes

gorup.id

client.id



**再平衡参数**

max.poll.interval.ms: 心跳

session.timeout.ms: 心跳超时时间

max.poll.interval.ms





auto.offset.reset： Kafka中没有初始偏移量或当前偏移量在服务器中不存在时的处理， earliest， latest, none, anything

enable.auto.commit: 设置为true，消费者会自动周期性地向服务器提交偏移量。

auto.commit.interval.ms: 自动提交间隔

fetch.min.bytes

fetch.max.wait.ms

fetch.max.bytes

connections.max.idle.ms

check.crcs

exclude.internal.topics

isolation.level： 控制如何读取事务消息。

heartbeat.interval.ms

max.poll.records



max.partition.fetch.bytes

send.buffer.bytes

retry.backoff.ms



offsets.topic.num.partitions： __consumer_offsets 分区个数

partition.assignment.strategy: 指定分配策略

unclean.leader.election.enable： 所有的 ISR 副本都失败，是否可立即选择非 ISR 集合中的可用副本







**主题相关参数**

Cleanup.policy: 默认 delete, 对旧日志的利用方式，可选 compact

Compress.type= 压缩格式 gzip, snappy， 默认是没有压缩的, producer 为...

delete.retention.ms

flush.ms

flush.messages

Max.message.bytes=512:  

Retention.bytes: 默认不配置, 日志保留的最大大小

Rendition.ms: 默认保留的日志时间

Segment.bytes: 默认 1G，持久化存储时分段

Segment.index.bytes: 10MB,   .log, .index, .idx，一般不关注

Segment.ms: 7days

Unclean.leader.election.enable: true 是否可让不在 ISR 中 replicas 设置作为 leader, 可能丢数据

min.cleanable.dirty.ratio=0.5: 避免清楚压缩率超过 50% 的数据

Min.insync.replicas=1: lacks 中对与 ISR 的确认..



**物理存储**

配置项(LJ)

log.index.interval.bytes: 4096(4K) 网络中的索引项达到此大小，写 .index

Log.roll.hours: 168(7天)

Log.index.size.max.bytes: 10485760(10MB) 限制索引文件的大小...



**生产参数**

```
org.apache.kafka.clients.producer.ProducerConfig
```

bootstrap.servers:

key.serializer:

Value.serializer:

Acks:  确认机制

Retries:

compression.type： 默认 NONE，gzip, snappy, lz4。。

partitioner.class

retry.backoff.ms

request.timeout.ms

batch.size

client.id: 生产者发送请求的时候传递给broker的id字符串。

send.buffer.bytes

buffer.memory

connections.max.idle.ms

linger.ms

max.block.ms





## 其他

MQ 之间的对比

RabbitMQ 遵循 AMQP 协议

RocketMQ 根据 Tag 进行消息过滤

Kafka 吞吐量高，实时性不如 RabbitMQ 好。

