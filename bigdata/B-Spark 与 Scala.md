[parent](README.md)  

[TOC]  

内存计算引擎，用于替换 Hadoop 中的 MapReduce  



MR 的缺点：

基于数据集计算，面向数据，数据需要落盘。

与 Hadoop 紧密耦合，无法动态获取

无法满足对数据的迭代计算







Spark 基于内存计算，基于 Scala 语法开发，适合迭代计算。

基于 Hadoop 1.x 架构思想，采用自己的方式改善 Hadoop 1.x 的问题。

只适用于计算，可通过 Yarn 链接 Spark 和 Hadoop 使用



## Scala 编程
Scala语言基础、数据结构、高阶语法特性等  
函数式编程  


## Spark





### 架构

Spark SQL： 支持多种数据源，如 Hive表、Parquet、JSON，统一处理关系表和 RDD，通过 SQL 进行外部查询。

Spare Code：

Spark Streaming 实时计算：



MLib 机器学习：  MLBase 专注于机器学习，主要分为四个部分， MLib、MLI、ML Optimizer、MLRuntime。

GraghX 图计算：图和图的并行计算

独立调度器：

Yarn：

Mesos：



集群管理器： 支持在各种集群管理器上工作，包裹 Hadoop Yarn、Apache Mesos，以及 Spark 自带的一个简易调度器。





特点：

与 MR 相比，Spark基于内存的运算要快100倍以上，基于硬盘的运算也要快10倍以上。

实现了高效的 DAG 执行引擎，通过基于内存的方式处理数据流，计算的中间结果存放在内存中。





支持 Java、Python、Scala 的 API，支持超过 80 种高级算法。交互式的 Python和 Scala 的 shell，可方便在这些 shell 中使用 Spark 集群验证解决问题的方法。









两种算子：

Transformation 转化算子

Action 执行算子



哪些聚合类算子，应该尽量避免什么类型的算子

尽量避免 reduceByKey、join、distinct、repartition 会进行 shuffle 的算子

尽量使用 map 类的非 shuffl e 算子。



### RDD

> 分布式弹性数据集

所有算子都基于 RDD 执行。

RDD 执行的过程中会形成 DAG 图，然后形成 Lineage 保证容错性。

物理角度看，RDD 存储的是 block 和 node 之间的映射。

不支持增量迭代计算，Flink 支持。





### 配置参数

| 参数                         | 含义                        | 值设置                                              |
| ---------------------------- | --------------------------- | --------------------------------------------------- |
| spark.default.parallelism    | 每个 Stage 的默认 Task 数量 | 官网建议task个数为CPU的核数*executor的个数的2~3倍。 |
| spark.shuffle.memoryFraction |                             |                                                     |
|                              |                             |                                                     |



### 集群角色

Driver 驱动器节点、Executor 执行器节点







**Driver**





**Executor**

在 Spark 作业中运行任务，任务间相互独立。







命令参数：

--class：

--master：

--deploy-mode：

--conf：

Application-jar：

Application-arguments：

```shell
#  100 该数字代表迭代次数
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--executor-memory 1G \
--total-executor-cores 2 \
./examples/jars/spark-examples_2.11-2.1.1.jar \
100   
```

```shell
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://hadoop102:7077 \
--executor-memory 1G \
--total-executor-cores 2 \
./examples/jars/spark-examples_2.11-2.1.1.jar \
100
```



```shell
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master yarn \
--deploy-mode client \
./examples/jars/spark-examples_2.11-2.1.1.jar \
100
```

