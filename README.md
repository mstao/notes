个人学习笔记，主要为数据结构与算法、后端、大数据相关。  
汇总散落在为知笔记(Windows 时使用)、有道云笔记(Mac 时使用)、Typora + 坚果云同步(系统性整理)中的内容，统一使用 Notion 进行管理，博客辅助知识的整理， 笔记整理说明见 [link](https://www.notion.so/janhen/a7cba23355f648a89ebd455f4f985cd9)。

## 数据结构与算法
见 [这里](https://github.com/Janhen/dsal)

## Java
[Java-集合类源码](https://www.notion.so/janhen/Java-13de9f29afb34f34bab5ae9e604eae77)  
> JDK 中 ArrayList,LinkedList,Vector,CopyOnWriteArrayList,PriorityQueue,ArrayDeque,Collections,Arrays 源码分析比较，关注于
> 集合类型的底层结构、扩容机制、线程安全、复杂度震荡、迭代方式等方面。

[Java-Map 类源码](https://www.notion.so/janhen/Java-Map-fe72eca8c00c49a39111d38dddefd92c)  
> JDK 中 HashMap,LinkedHashMap,ConcurrentHashMap,HashTable,ThreadLocalMap,WeakHashMap 源码分析比较，JDK7 和 JDK8 版本下的不同，
> 关注 Map 的 Hash 函数与元素定位、Hash 冲突的处理、NULL 值的处理、扩容与 rehash 机制、结构的变更、线程安全的保证(全局锁,分段锁,CAS)

[JVM-内存和GC](https://www.notion.so/janhen/JVM-GC-a5644802b46b4d34bdf95af864a16059)  
> 内存、GC 配置参数，内存划分，内存问题，内存回收...

[Arthas](https://www.notion.so/janhen/Arthas-1874b06e2d2b45b0a9ed9412ca1ddeb1)  
> 使用 arthas 在未有 jdk 自带工具的 Docker 中获取堆 dump，获取类中的静态变量，获取运行在 Spring 中的实际配置参数，
> 获取运行时候的方法请求和返回参数，获取方法的耗时统计情况

## MySQL
[SQL 语句编写](https://www.notion.so/janhen/SQL-af130cfc54d3463ab5bd08b77d51c12d)  
> sql 标准的语法，where 正则过滤，group by 去重，子查询，join，limit 分页，常用的函数...

[MySql 索引](https://www.notion.so/janhen/MySql-f1dd30907e1d45da92351a121a441446)   
> 聚簇索引、普通索引、唯一索引、联合索引，最左匹配原则。
> 字符串索引，前缀索引，列的区分度，区分度不足的处理。
> 常见的索引失效问题，索引选择异常问题，索引重建。
> 索引实现，自适应 hash 索引， b 树、b+ 树。

[MySQL 事务](https://www.notion.so/janhen/MySQL-1cc42fe5d18549d1a72a2dab6226342c)
> 事务的四大特性，并发问题，隔离级别，事务实现方式，事务分类，两阶段提交，Innodb 在 RR 下解决幻读，MVCC 机制。

[MySQL 锁](https://www.notion.so/janhen/MySQL-41e378ebba7349c5b3d9c18c1f505b62)
> 锁的分类、粒度、实现算法，乐观锁、悲观锁、死锁等

[MySQL 结构|索引|SQL的优化](https://www.notion.so/janhen/MySQL-SQL-678d9fce85cf4528b3e79b64e1ae8b6b)
> decimal,varchar,char,date,datetime,time 等字段的占用，避免索引失效、确定索引顺序、索引列的选取、清理重复索引等，SQL 执行过程，慢查询分析，
> 特定条件的 SQL 优化等。

[Mysql 日志](https://www.notionhttps://www.notion.so/janhen/MySQL-SQL-678d9fce85cf4528b3e79b64e1ae8b6b.so/janhen/Mysql-1f6ef9a1d4ad451db0066ed7af74cb01)  
> 慢查询日志，慢查询日志的分析，explain 各个字段的含义，慢查询分析工具 mysqldumpslow,pt-query-digest，配置...
> binlog，两阶段提交，与 redo 日志的比较，实时获取 binlog 内容，binlog 的清理...
> redo log，wal, 脏页的刷新，配置...
> relay log、error log、general log

[MySQL 备份与恢复](https://www.notion.so/janhen/MySQL-7fae8af4ddf149f1bfaa558f430c28d7)  
> 备份的内容，恢复的策略，基于 binlog 的恢复，导入导出方式，mysqldump 进行 全量、条件、表结构、压缩等备份恢复，xtrabackup 进行全量、条件、压缩、增量备份和恢复...

[MySQL 管理](https://www.notion.so/janhen/MySQL-d8fe7b26db344cb6b60a5d34ac8b2f91)
> 常用的 MySQL 运维管理操作整理，mysql 制作报表，用户与权限管理，运行情况查看与监控，碎片整理，在线表结构变更；服务器配置参数、连接参数、归档数据库配置说明；大表数据的删除；

[Percona-Toolkit](https://www.notion.so/janhen/Percona-Toolkit-e9ea157626d24777babdaacf2babd301)  
> Percona 公司开发用于管理 MySQL 的工具使用整理，在线更改表结构工具 pt-online-schema-change, 慢查询分析工具 pt-query-digest，主从表同步工具 
> pt-table-sync，重复索引发现工具 pt-duplicate-key-checker ...

## 大数据
[HDFS](https://www.notion.so/janhen/Hadoop-HDFS-e34843132aff4d839f9ace7d3e33c53c)  
> HDFS 的元数据管理，secondary name node 处理流程与配置参数，元数据存储的 FSImage 和 edit 文件，HDFS 的读写过程，常用参数...

[MapReduce](https://www.notion.so/janhen/Hadoop-MapReduce-335e290ddadf495b9c87145ca205c07d)
  
[Hive 基础知识](https://www.notion.so/janhen/Hive-48d40c2239fc44568abc8dd340324d0f)  
[Hive - HQL](https://www.notion.so/janhen/Hive-HQL-a2e5e48a5e7743378c07c610e504ae21)  
> 数据导入导出，表结构修改，内外部表、分区表、分桶表创建，lateral view，join，union，各个排序子句，HQL 转换为 MR，Join 与 MR，JSON 数据的处理...

[Hive 中的函数](https://www.notion.so/janhen/Hive-07129ed27e8845c2b7ec7af785eea19f)  
> 内嵌的函数： 日期函数、条件函数、字符串函数、数学函数、集合处理函数。
> 聚集函数、表生成函数、窗口函数、排名函数、序列函数。
> 自定义函数的导入与使用。

[Hive 管理](https://www.notion.so/janhen/Hive-5106f099c6494252a24040383b09641b)  

[数据仓库](https://www.notion.so/janhen/9e989680fb0246b9b09cd799f5e9a5fc)  
> 数据仓库的特征，作用，数仓的分层，OLAP 与 OLTP 的比较，各种 OLAP 的比较。
> 范式建模，维度建模，指标体系和指标，缓慢变化维和周期性事实表，拉链表。
> 数据采集，主题分析，任务调度系统，元数据管理，数据质量监控，数据可视化。

[Scala 语言学习笔记](https://www.notion.so/janhen/Scala-e42fbae5e80b41fd9e9979c8cbce229b)  

[SparkCore](https://www.notion.so/janhen/SparkCore-54457ccfa35045288bb3b3689c931f68)  

[SparkSQL](https://www.notion.so/janhen/SparkSQL-5db6b2b73e474e7bbc1314f5a32ba830)  

## 其他
[Linux 常用命令](https://www.notion.so/janhen/Linux-7993a60a6636411484c0832795383d2e)  
[Docker 学习笔记](https://www.notion.so/janhen/Docker-22568256c6894b879f587ca34358af39)
> Docker 镜像、容器、仓库，单机网络、多机网络、overlay 网络、etcd 存储，数据持久化，Dockerfile 语法，Docker compose...  

[Docker 镜像构建](https://www.notion.so/janhen/Docker-b6d1d65453934331933244d8c9504d2e)  
> 构建前端项目、maven 项目，使用容器作为编译环境，镜像打包，将参数在运行时传入，编写 Dockerfile 的一些约定(规范)
