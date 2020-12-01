[parent](README.md)  

[TOC]  

## 端口

组件的安装以及配置  

Hadoop 核心组件安装  

Hive 的安装配置  

Hue 的安装配置  

Sqoop 的安装配置  

Flume 的安装配置、启动  

组件的端口整理


| 组件      | 端口  | 模块                   | 协议 | 用途                                                         |
| --------- | ----- | ---------------------- | ---- | ------------------------------------------------------------ |
| Hadoop    | 8020  | HDFS-NN                |      | Hdfs 管理端口                                                |
|           | 8485  | DN 共享编辑端口        |      | dfs.namenode.shared.edits.dir                                |
|           | 8088  | Yarn-ResourceManager   | HTTP | ResourceManager 的 Web 管理端口                              |
|           | 9000  | HDFS-NameNode          |      | HDFS 管理端口，Java Client 连接 HDFS 的端口                  |
|           | 10020 | 历史服务器             | HTTP | Job 历史服务器端口                                           |
|           | 19888 | 历史服务器             | HTTP | Yarn Job 历史管理端口                                        |
|           | 50010 | HDFS                   |      | 文件上传相关的?                                              |
|           | 50070 | HDFS-DataNode          | HTTP | DataNode 的 http 地址, 可查看 HDFS 的目录内容<br>            |
|           | 50075 | Datanode-webhdfs       |      | 访问文件内容或者进行打开、上传、修改、下载等操作<br>不区分端口，直接使用namenode的IP和端口进行所有的webhdfs操作<br>需要在所有的datanode上都设置hdfs-site.xml中的 <br> `dfs.webhdfs.enabled` 为true |
|           | 50090 | HDFS-2NN               | HTTP | 2NN 的 http 端口                                             |
| Hive      | 9083  | Metastore              | TCP  | Hive 中 thrift 的 metastore 服务                             |
|           | 10000 | HiveServer2            | TCP  | Hive 连接的端口                                              |
|           | 10002 | HiveServer2            | HTTP | hiveserver2 浏览器端口                                       |
| Hbase     | 16010 | HMaster                |      | HMaster 的 Web管理界面                                       |
| Hue       |       |                        |      |                                                              |
| Impala    | 21000 |                        |      | Impalad Shell 端口                                           |
|           | 21050 |                        |      | impala 对应 jdbc 连接端口                                    |
|           | 22000 | Impalad                |      | Impalad 地址                                                 |
|           | 25000 | impalad                | HTTP | impalad 的管理界⾯                                           |
|           | 25010 | statestored            | HTTP | statestored 的管理界⾯                                       |
| Haproxy   | 1080  | #                      |      | HAProxy 统计的 Http 端口                                     |
| Zookeeper | 2181  | 提供服务的端口         |      |                                                              |
|           | 2888  | 服务器之间通信端⼝     |      |                                                              |
|           | 3888  | 服务器之间投票选举端⼝ |      |                                                              |
|           |       |                        |      |                                                              |
| Azkaban   | 8081  | Web                    |      | Azkaban 浏览器 Web 访问                                      |
| MySQL     | 3306  | #                      |      |                                                              |
| Redis     | 6379  | #                      |      |                                                              |
|           | 26379 | #                      |      |                                                              |
| Kafka     | 9092  | #                      |      |                                                              |





大数据的 5V 特征
- Volume: 数据量大
- Velocity: 速度快，个性化推荐要求实时处理
- Variety: 多样、复杂: 音频、图片、日志、地理位置...
- Veracity: 真实
- Value: 低价值，高度分析后产生新价值  



技术驱动：
存储： 文件 => 分布式存储
单机 => 分布式


商业驱动：


主要内容
- 数据采集： Flume Sqoop  
- 数据存储： Hadoop  
- 数据处理、分析、挖掘： Hadoop、Spark 。。。
- 可视化：  
  



1TB = 1024 GB

1PB = 1024 TB 

1EB = 1024 PB

1ZB = 1024 EB 

1YB = 1024 ZB 

1BB = 1024 YB

1NB = 1024 BB



ETL工程师，数据仓库工程师，实时流处理工程师，用户画像工程师，数据挖掘，算法 工程师，推荐系统工程。



典型应用：  

```
count/sum/avg  
    ||
group by/join
    ||
窗口分析函数
    ||
异常/欺诈检测  
    ||
人工智能
```

```
报表
用户细分
指标监控
指标预警
```

数据采集:  Flume  
数据仓库：   Hive  
函数式编程：  Scala  
内存计算引擎： Spark  

新一代计算引擎：  Flink  

数据中台大屏  

**Hadoop 基础**

**Hadoop**

HDFS  
MapReduce: 机器上分布式并行计算  
Yarn: 资源管理、作业调度   

**HDFS**

文件、块、副本  
Distributed File System  


fault-toerant: 容错  
low-cost hardware:  
high throught  
large data sets  


单机与分布式 FS:  
文件横跨 N 个机器  


设计目标：  
- Hardware failure  
- Stream access:  batch processing    
- Moving Computation is Cheaper than Moving Data: 移动计算比移动数据更..  
  

**HDFS 架构**  

- NameNode and DataNodes  
- Master/Slave  
- NN: the fs namespace  
- DN: storage  
- a file is split into on or more blocks: blocksize 128M  
- blocks are stored in a set of DataNodes  
- determines the mapping of blocks to DataNodes: 块与数据节点映射关系  
![i78c5q](http://janhen.com/img/i78c5q.jpg)

**HDFS 副本机制**

Data Replication  

**YARN**

资源调度系统  



**Hadoop 生态**

去 Ioe，可以部署在廉价的机器上  
![E9nFeV](http://janhen.com/img/E9nFeV.png)


发行版本的选择：  
- Apache 不同版本/框架整合，jar 冲突
- CDH: https://www.cloudera.com
  cm(cloudera manager)通过页面一键安装、文档较为全面  
  cm 不开源  
- HDP:  
原装  
- MapR:  



**重要目录文件**

etc/hadoop/core-site.xml:  
etc/hadoop/hdfs-site.xml:  
sbin/start-dfs.sh  
sbin/stop-dfs.sh  

etc/hadoop/mapred-site.xml:  
etc/hadoop/yarn-site.xml:  

sbin/start-yarn.sh  
sbin/stop-yarn.sh


etc/hadoop/hadoop-env.sh： 全局使用的 Hadoop Shell 命令  
etc/hadoop/hadoop-user-functions.sh：  覆盖一些 Shell 函数实现  
~/.hadooprc：  存储个人环境信息  

sequence files, HAR files 使用二进制方式存储  

// TODO Hadoop 的发展过程



**课程**

新技术的实现  
架构的思维  

内推  
每月一次的内推机会  

学习报告  

技术内训  



视屏完整观看  
Xmind 总结笔记  
周期性复习  
多练多巧  
和谐的技术交流氛围  

  

**资源**

文档地址

http://hadoop.apache.org  
http://hive.apache.org  
http://hbase.apache.org  
http://spark.apache.org  
http://flink.apache.org  
http://storm.apache.org  


CDH:  
http://archive.cloudera.com/cdh5/cdh/5/  
http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.15.1/

```
http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.15.1.tar.gz
```







数据ETL  
数据聚合  



![KrXoDq](http://janhen.com/img/KrXoDq.png)  
NameNode:
2NN:



**x**

Cloudera 发行版本： CDH，生产环境建议使用

Hortonworks 发行版： 免费开源，提供一整套的 web 管理界面

Apache Hadoop:





Apache 下载的地址

http://archive.apache.org/dist/



## 环境搭建

[版本支持](https://cwiki.apache.org/confluence/display/HADOOP/Hadoop+Java+Versions)  




### 管理命令  
管理 Hadoop 集群的命令  


daemonlog  
设置节点上指定类的日志级别   
非持久的  
```
hadoop daemonlog -setlevel 127.0.0.1:9870 org.apache.hadoop.hdfs.server.namenode.NameNode DEBUG
hadoop daemonlog -getlevel 127.0.0.1:9871 org.apache.hadoop.hdfs.server.namenode.NameNode DEBUG -protocol https
```



FS Shell

```
hadoop fs -appendToFile localfile1 localfile2 /user/hadoop/hadoopfile
```

**MapReduce 命令**  

archive  
distcp  


job：  
于 Map Reduce Job 进行交互  
```
 mapred job | [GENERIC_OPTIONS] | [-submit <job-file>] | [-status <job-id>] | [-counter <job-id> <group-name> <counter-name>] | [-kill <job-id>] | [-events <job-id> <from-event-#> <#-of-events>] | [-history [all] <jobHistoryFile|jobId> [-outfile <file>] [-format <human|json>]] | [-list [all]] | [-kill-task <task-id>] | [-fail-task <task-id>] | [-set-priority <job-id> <priority>] | [-list-active-trackers] | [-list-blacklisted-trackers] | [-list-attempt-ids <job-id> <task-type> <task-state>] [-logs <job-id> <task-attempt-id>] [-config <job-id> <file>]
```

pipes: 运行 pipes Job  
```
mapred pipes [-conf <path>] [-jobconf <key=value>, <key=value>, ...] [-input <path>] [-output <path>] [-jar <jar file>] [-inputformat <class>] [-map <class>] [-partitioner <class>] [-reduce <class>] [-writer <class>] [-program <executable>] [-reduces <num>]
```


queue:  
交互并查看作业 Job 队列信息  
```
 mapred queue [-list] | [-info <job-queue-name> [-showJobs]] | [-showacls]
```



**hdfs**

```
hadoop fs -ls /
hadoop fs -put
hadoop fs -copyFromLocal
hadoop fs -moveFromLocal
hadoop fs -cat
hadoop fs -text
hadoop fs -get
hadoop fs -mkdir
hadoop fs -mv 
hadoop fs -getmerge
hadoop fs -rm
hadoop fs -rmdir
hadoop fs -rm -r
```



**配置与脚本**

主要时 Xml 配置、日志配置  
避免使用  hadoop, io, ipc, fs, net, file, ftp, kfs, ha, file, dfs, mapred, mapreduce, and yarn 前缀的键  


```
core-default.xml
hdfs-default.xml
hdfs-rbf-default.xml
mapred-default.xml
yarn-default.xml
```


```
~/.hadoop-env  用户的环境变量  
```

制品

```
hadoop-client
hadoop-client-api
hadoop-client-minicluster
hadoop-client-runtime
hadoop-hdfs-client
hadoop-hdfs-native-client
hadoop-mapreduce-client-app
hadoop-mapreduce-client-common
hadoop-mapreduce-client-core
hadoop-mapreduce-client-jobclient
hadoop-mapreduce-client-nativetask
hadoop-yarn-client
```

环境变量

HADOOP_OPTS： 由外部提供  

重要的环境变量  

终端用户变量：  

HADOOP_CLIENT_OPTS:  JVM 选项或是 Hadoop 选项  
(command)_(subcommand)_OPTS
HADOOP_CLASSPATH
Auto-setting of Variables  

YARN_CLIENT_OPTS 定义且运行 yarn 时候，会被覆盖  
```
HADOOP_CLIENT_OPTS="-Xmx1g -Dhadoop.socks.server=localhost:4000" hadoop fs -ls /tmp
```
一般比 `HADOOP_CLIENT_OPTS` 优先  
```
MAPRED_DISTCP_OPTS="-Xmx2g"
```


用冒号分隔的目录，文件或通配符位置的列表  
```
HADOOP_CLASSPATH=${HOME}/lib/myjars/*.jar hadoop classpath
```



管理公共的配置环境变量  
默认放在 `${HOME}/.hadoop-env` 文件中  
使用的时 bash 语法  
```
#
# my custom Apache Hadoop settings!
#

HADOOP_CLIENT_OPTS="-Xmx1g"
MAPRED_DISTCP_OPTS="-Xmx2g"
HADOOP_DISTCP_OPTS="-Xmx2g"

if [[ -n ${HADOOP_SERVER} ]]; then
  HADOOP_CONF_DIR=/etc/hadoop.${HADOOP_SERVER}
fi
```

管理的环境变量：  

(command)_(subcommand)_OPTS
(command)_(subcommand)_USER  


```
HDFS_DATANODE_USER=root
HDFS_DATANODE_SECURE_USER=hdfs
```



其他

3.x 新特性

MapReduce 基于 内存和 IO + 磁盘处理数据







Hdoop Common 改进



监控报警

大数据 airflow 监控报告

数据比对





Q&A

Could not find or load main class org.apache.hadoop.util.VersionInfo  
https://stackoverflow.com/questions/21212629/could-not-find-or-load-main-class-org-apache-hadoop-util-versioninfo



### Hadoop

```shell
hadoop namenode -format

hdfs dfs -ls /
hdfs dfs -mkdir /test
hdfs dfs -mkdir /input
hdfs dfs -mkdir /out
```

```shell
# Hadoop运行时产生文件的存储目录
/opt/lagou/servers/hadoop-2.9.2/data/tmp
# NN 数据
/opt/lagou/servers/hadoop-2.9.2/hdfs/namenode

mv /opt/lagou/servers/hadoop-2.9.2/data/tmp.backup
mkdir -p /opt/lagou/servers/hadoop-2.9.2/data/tmp
```





# 资源链接

## 文档链接
Fsimage 文件的内容  
https://hadoop.apache.org/docs/r2.9.2/hadoop-project-dist/hadoophdfs/HdfsImageViewer.html

MR 中的属性  
https://hadoop.apache.org/docs/r2.9.2/hadoop-mapreduce-client/hadoop-mapreduceclient-core/mapred-default.xml



Hive  
Hive 参数说明的官方文档：  
https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties  



Hue官方网站：https://gethue.com/   
HUE官方用户手册：https://docs.gethue.com/   
官方安装文档：https://docs.gethue.com/administrator/installation/install/   
HUE下载地址：https://docs.gethue.com/releases/  



Flume    
Flume官网地址：http://flume.apache.org/   
文档查看地址：http://flume.apache.org/FlumeUserGuide.html   
下载地址：http://archive.apache.org/dist/flume/   
选择的版本 1.9.0  

中文帮助文档  
https://flume.liyifeng.org/  


Sqoop  
Sqoop 官网：http://sqoop.apache.org/   
Sqoop下载地址：http://www.apache.org/dyn/closer.lua/sqoop/  



Zookeeper  
稳定版本地址  
http://zookeeper.apache.org/releases.html  


Hbase  
http://archive.apache.org/dist/hbase/1.3.1/  





Hive 练习题





### 下载链接

http://archive.cloudera.com/cdh5/repo-as-tarball/5.7.6/