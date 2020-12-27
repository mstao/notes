[TOC]  

# *Hive 

## 基础说明

> **将 SQL 转换成 MapReduce 任务的工具**   
>
> 基于 Hadoop 的数据仓库工具，存储和处理海量结构化的数据，可将结构化的数据映射成一张表

**说明**

处理 HDFS 中的海量数据  

通过 SQL 完成计算

基于Hadoop的数据仓库工具，可以将结构化的数据文件映射为一张表，并提供类SQL查询功能.

最适合数据仓库程序   



缺点：

HQL 表达能力有限： 无法表示迭代计算，数据挖掘方面不擅长

效率低： 自动生成的 MR 效率，调优较困难



 

Pig: Hive 的替代品，apache 顶级项目  
一种数据流语言，不是查询语言  
常用语 ETL 中的一部分  


HBase: 已经可以结合 Hive 使用  


Thrift:  
提供了可远程访问其他进程的功能，提供了使用 JDBC, ODBC 访问 Hive 的功能  

HWI: 简单的网页界面  

HQL： 



**Hive 与 RDBMS 的比较**  

- 查询语言类型：

- 执行引擎： Hive 可为 MR/Tez/Spark/Flink，RDBMS 使用自己的执行引擎  

- 数据存储： RDBMS 使用本地文件系统， Hive 使用 HDFS  

- 执行速度： 

- 扩展型： Hive 支持水平扩展，RDBMS 支持垂直扩展，对水平扩展不友好  

- 数据更新： Hive 对数据更新不友好  





Hive:  

- 解释器: AST 抽象语法树
- 编译器
- 优化器
- 执行期
- MetaStore

Hive 的元数据默认存储在自带的 derby 数据库中  
derby: java 开发、但进程、单用户



### 架构原理

// TODO

Client： Hive、Beeline、Hue



元数据库： 存放元数据的地方，数据库、表、分区、列的名称和属性，数据所在位置等信息



Meta store 元数据服务： 提供统一的服务接口，Client 通过 metastore 访问元数据。三种模式，内嵌、本地、远程模式。



。。。



Hive Driver： 解释器、编译器、优化器、执行器



HSQL 转化为 MapReduce 的过程

HiveSQL 

--> AST 抽象语法树

--> QB 查询块

--> OperatorTree 操作树

--> 优化后的 OperatorTree

--> MapReduce 树

--> 优化后的 MapReduce 任务树





Q: Hive 关联表对应 MapReduce 如何实现？





<p align="center">HiveQL 于 SQL 的比较</p>

| 比较项        | SQL                        | HiveQL                                  |
| ------------- | -------------------------- | --------------------------------------- |
| ANSI SQL 更新 | 支持  UPDATEVINSERT\DELETE | 不完全支持  insert OVERWRITEVINTO TABLE |
| 事务          | 支持                       | 可支持(分桶、ORCFile 文件格式)          |
| 模式          | 写模式                     | 读模式                                  |
| 数据保存      | 块设备、本地文件系统       | HDFS                                    |
| 延时          | 低                         | 高                                      |
| 多表播入      | 不支持                     | 支持                                    |
| 子查询        | 完全支持                   | 只能用在 From 子句中                    |
| 视图          | Updatable                  | Read-only                               |
| 可扩展性      | 低                         | 高                                      |
| 数据规模      | 小                         | 大                                      |



### 数据类型

基本数据类型

- 整数类型： Integer  、TINYINT、SMALINT、INT、BIGINT

- 浮点数类型： FLOAT、DOUBLE、DECIMAL(17byte)

- 字符类型： STRING(任意长度)、VARCHAR(1-65535)

- 日期类型： 



类型转换

String 可隐式转换  
cast 进行强制类型转换，失败返回空  



四种集合类型  
- array: 有序的相同类型集合
- map:  key 为基本类型
- struct:  不同类型字段的集合
- union: 不同类型元素存储在统一字段的不同行中  



原始的 JSON 数据

```sql
{
    "name": "songsong",
    "friends": ["bingbing" , "lili"] ,       //列表Array,
    "children": {                      //键值Map,
        "xiao song": 18 ,
        "xiaoxiao song": 19
    }
    "address": {                      //结构Struct,
        "street": "hui long guan" ,
        "city": "beijing"
    }
}
```

格式好的行数据

```
songsong,bingbing_lili,xiao song:18_xiaoxiao song:19,hui long guan_beijing
yangyang,caicai_susu,xiao yang:18_xiaoxiao yang:19,chao yang_beijing
```

```sql
-- 表创建
create table test(
  name string,
  friends array<string>,
  children map<string, int>,
  address struct<street:string, city:string>
)
row format delimited fields terminated by ','
collection items terminated by '_'
map keys terminated by ':'
lines terminated by '\n';
```

```sql
-- 测试结构查询
SELECT friends[1], children['xiao song'], address.city 
FROM test WHERE name = 'songsong';
```



### +表类型
外部表： 指定 external 关键字，元数据 + 数据分开管理，删除表定义，数据不会删除

内部表： 删除表的时候数据会删除

分桶表： 实现 DML 事务时必须

分区表： 



Q: 为何分区？

可避免全表扫描，提高查询效率，通常根据事件、地区等信息进行分区



Q: 为何分桶？

分区或表数据量过大，分桶降数据划分成更细粒度。

通过 分桶字段.hashCode % 分桶个数。

不能使用 load data local inpath 方式加载数据





在数据仓库中

ODS 外部表，从外部进来

DW 内部表

ADS 内部表



计算过程中使用到的临时表，数据随用随删，使用内部表。







### 文本文件编码
支持自定义文件存储格式  



默认的文件分割

- 行与行 \n  
- 字段之间 ^A  
- 元素之间 ^B
- k-v 之间： ^C  



### 读时模式  
写时模式 -> 写数据检查 -> RDBMS  
读时模式 -> 读时检查 -> Hive   




## 命令操作
hive  
--service name 指定服务名称  
hive --help --service cli  





### 变量和属性
set -v

```
-- 打印当前数据库  
hive.cli.print.current.db=true
```

执行文件  ，一般为 h/hql 结尾  
hive -f xxx.hql

-i: 指定文件名  
启动时，首先执行这个文件，默认为 .hiverc  

.hivehistory: 记录hive 的历史命令

集成  Shell 使用  
集成 HDFS 使用  






## 元数据管理
通常是独立的 RDBMS  


## Thrift 服务
开启 hiveserver2，搭配 groovy / maven 使用      
默认情况下，管理表在 `/usr/hive/warehouse` 目录下

hive.start.cleanup.scratchdir  默认为 false  
每次重启 Hiveserver 时清理掉历史目录  



### HiveServer2

> 管理元数据，生产环境中常使用。



### beeline
beeline 可以连接 Hive， MySQL...  




### HCatalog
Hive 的元数据服务  
统一的元数据服务  

可不启动 MR 任务执行  
主要是 DDL 对元数据的操作  


hcat -e "create database tt2";

hcat -f xxx.hql




## +数据存储格式 
TEXTFILE（默认格式） 、
SEQUENCEFILE、
RCFILE、
ORCFILE、
PARQUET。

TEXTFILE、SEQUENCEFILE 的存储格式是基于行存储的；  
ORC和PARQUET 是基于列式存储的。


TEXTFILE  


通常先导入到 textfile，之后执行 insert ... select 到其他格式的表中  



**行和列的存储**   
行存储：  
insert 与 update 比较容易  
select 需要查询大多无用的数据  


textfile,sequencefile 行  
rcrile, orc, parout   列存储  



sequencefile:  
可分割  
可压缩  
record, none, block 压缩  ...




RCFile:  
烈士记录文件，结合列和行存储的优势  

先按水平划分，后让垂直划分  



### ORCFile  

> 表位 ORC 可支持事务操作

组成  
文件脚注(file footer)：  
postscript：  
stripe: 条带  ，默认 250M  

- Index Data:  1W行一个, 条带统计信息, 数据在条带中的位置  
- Row data:  水平 --> 垂直, 列为单位存储数据  
- Stripe Footer: stripe 元数据信息  

三个级别的索引：  

文件级别、条带级、行组级  

无需指定分割符，自动处理  



### Parquet  

> apache 顶级项目
>
> [site](https://parquet.apache.org/)

通用型强  

**与语言和平台无关**  

二进制存储的  

文件中包含数据和元数据  

Row group:  文件有多个行组组成，写入数据最大的缓存单元，50M ~ 1GB 之间    

Column Chunk: 存储当前行组内的某一行数据  
最小的 I/O 并发单元  

Page: 压缩读取数据的最小单元  
8K ~ 1M 之间，越大压缩率越高  

Footer: 数据的 schema 信息    
每个行组的元数据信息：  
每个 column chunk 的元数据信息：  



### 比较
压缩比  
ORC > Parquet > text  

执行查询 
ORC 与 Parquet  相当  

- TextFile文件更多的是作为跳板来使用(即方便将数据转为其他格式)  
- 有update、delete和事务性操作的需求，通常选择ORCFile  
- 没有事务性要求，希望支持 Impala、Spark，多种计算框架/查询引擎，建议选择 Parquet








## Hive 调优
### 架构优化
**执行引擎选择**

hive.execution.engine 控制  

MapReduce, Tez, Spark, Flink  

DAG 有向五环图  
Hontonworks 开源  





优化器的使用  
矢量化查询执行  
使用矢量化查询执行，必须用ORC格式存储数据  

```
-- 默认 false 
set hive.vectorized.execution.enabled = true;

-- 默认 false
set hive.vectorized.execution.reduce.enabled = true;
```





文件格式





数据压缩





分区、分桶表





### 参数优化



本地模式   



严格模式  



JVM重用   



并行执行   

Hive 将查询转换成一个或多个阶段，MapReduce 阶段、抽样阶段、合并阶段、Limit 阶段.. 

默认情况下一次只执行一个阶段，对于特定 Job 有多个阶段，阶段间非完全相互依赖，并行执行，可以缩短 job 的执行时间。

并行执行集群利用了对应增加。



推测执行   



合并小文件   



Fetch模式





### SQL 优化
**列裁剪和分区裁剪**  
sort by 代替 order by   



**group by 代替 count(distinct)**  

去重计算数据量大时不好处理，一般COUNT DISTINCT使用先GROUP BY再COUNT的方式替换。



**group by 配置调整**  



**join 基础优化**  

map join

分桶 join





**处理空值或无意义值**  

大表 Join 大表时，key 有大量的异常数据，结果 Join 的时候耗时长，可通过 SQL 对其进行过滤

空 Key 转化，ke y 非异常数据，为空 key 设置随机值，防止 ...



**单独处理倾斜key**  





**调整 Map 数**  

Q:  是不是map数越多越好？
  答案是否定的。如果一个任务有很多小文件（远远小于块大小128m），则每个小文件也会被当做一个块，用一个map任务来完成，而一个map任务启动和初始化的时间远远大于逻辑处理的时间，就会造成很大的资源浪费。而且，同时可执行的map数是受限的。

Q:  是不是保证每个map处理接近128m的文件块，就高枕无忧了？
  答案也是不一定。比如有一个127m的文件，正常会用一个map去完成，但这个文件只有一个或者两个小字段，却有几千万的记录，如果map处理的逻辑比较复杂，用一个map任务去做，肯定也比较耗时。

增加 Map 的方法：

调整 maxSize 最大值，使 maxSize 小于 blockSize 增加 map 个数

// TODO maxSize 对应的配置参数...

```java
computeSliteSize(Math.max(minSize,Math.min(maxSize,blocksize)))=blocksize=128M
```



**调整 Reduce 数**  







连续值问题

Hive 自带的序列化与反序列化

https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide#DeveloperGuide-HiveSerDe



### *json 数据处理

Hive 处理 json 数据的方式

- 内建的函数 get_json_object、json_string
- 自定义 UDF 函数
- 使用序列化反序列化工具



**方式一: 内建的函数处理**

处理简单的 json 串。

- `get_json_object(string json_string, string path)`: 解析json字符串json_string，返回path指定的内容；

- `json_tuple(jsonStr, k1, k2, ...)`: ：参数为一组键k1，k2，...和json字符串，返回值的元组。该方法比 get_json_object高效，可以在一次调用中输入多个键, 对嵌套结果的解析操作复杂；

- `explode` / `lateral view`，使用explod将Hive一行中复杂的 array 或 map 结构拆分成多行。

```sql
CREATE TABLE IF NOT EXISTS jsont1( 
  username string, 
  age int, 
  sex string, 
  json string 
) row format delimited fields terminated by ';';
load data local inpath '/root/lagoudw/data/weibo.json' overwrite into table jsont1;
```

```sql
-- get 单层值 
select username, age, sex, 
  get_json_object(json, "$.id") id, 
  get_json_object(json, "$.ids") ids, 
  get_json_object(json, "$.total_number") num 
from jsont1;
```

```sql
-- get 数组
select username, age, sex, 
  get_json_object(json, "$.id") id, 
  get_json_object(json, "$.ids[0]") ids0, 
  get_json_object(json, "$.ids[1]") ids1, 
  get_json_object(json, "$.ids[2]") ids2, 
  get_json_object(json, "$.ids[3]") ids3, 
  get_json_object(json, "$.total_number") num 
from jsont1;
```

```sql
-- json_tuple 一次处理多个字段
select json_tuple(json, 'id', 'ids', 'total_number') from jsont1;
```

含其他字段时，不能直接展开，需要使用 explod 展开

```sql
-- 拆分 json
select username, age, sex, id, ids, num 
from jsont1 
lateral view json_tuple(json, 'id', 'ids', 'total_number') t1 as id, ids, num;

-- 拆分 JSON -> 拆分 jsonarray
with tmp as( select username, age, sex, id, ids, num
            from jsont1 
            lateral view json_tuple(json, 'id', 'ids', 'total_number') t1 as id, ids, num ) 
select username, age, sex, id, ids1, num
from tmp 
lateral view explode(split(regexp_replace(ids, "\\[|\\]", ""), ",")) t1 as ids1;
```

**方式二: 使用 UDF 处理**

能处理大部分数据，更灵活。

```sql
-- 创建临时函数
add jar /root/lagoudw/jars/bigdata-hive-1.0-SNAPSHOT.jar;
create temporary function json_json_array as "com.janhen.bigdata.hive.ParseJsonArray";

select username, age, sex, parse_json_array(json, "ids") ids 
from jsont1;

select username, age, sex, ids1 
from jsont1 
lateral view explode(parse_json_array(json, "ids")) t1 as ids1;

select username, age, sex, id, num 
from jsont1 
lateral view json_tuple(json, 'id', 'total_number') t1 as id, num;
-- 合并
select username, age, sex, ids1, id, num 
from jsont1 
lateral view explode(parse_json_array(json, "ids")) t1 as ids1 
lateral view json_tuple(json, 'id', 'total_number') t1 as id, num;
```



**方式三: 使用SerDe处理**

对象的序列化用途：

- 把对象转换成字节序列后保存到文件中

- 对象数据的网络传送

Read : HDFS files => InputFileFormat => <key, value> => Deserializer => Row object

Write : Row object => Seriallizer => <key, value> => OutputFileFormat => HDFS files

```json
{"id": 1,"ids": [101,102,103],"total_number": 3}
{"id": 2,"ids": [201,202,203,204],"total_number": 4}
{"id": 3,"ids": [301,302,303,304,305],"total_number": 5}
{"id": 4,"ids": [401,402,403,304],"total_number": 5}
{"id": 5,"ids": [501,502,503],"total_number": 3}
```

```sql
create table jsont2(
  id int,
  ids array<string>,
  total_number int
)
ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe';
load data local inpath '/data/lagoudw/data/json2.dat' into table jsont2;
```



**JSON 处理方式比较**

1、简单格式的json数据，使用 `get_json_object`、`json_tuple` 处理

2、对于嵌套数据类型，可以使用 UDF

3、纯 json 串可使用 JsonSerDe 处理更简单





## 配置参数

```shell
hive \
  -hiveconf mapred.max.split.size=128000000 \
  -hiveconf  hive.exec.reducers.bytes.per.reducer=128000000 \
  -e  "$sql"
```



<p align="center"><strong>常用配置</strong></p>

| 变量                                 | 含义                     | 参数类型 | 取值作用                                                 |
| ------------------------------------ | ------------------------ | -------- | -------------------------------------------------------- |
| hive.execution.engine                | 配置执行引擎             | 架构     | mr, tez                                                  |
| hive.fetch.task.conversion           | Fetch 抓取               |          | more 在全局查找、字段查找、limit查找等都不走mapreduce    |
| hive.exec.mode.local.auto            | 本地模式                 |          | 在输入数据量小的情况下，在单台机器上处理所有任务。       |
| hive.map.aggr                        | Map 端聚合               |          | True，默认                                               |
| hive.groupby.mapaggr.checkinterval   | Map 端聚合条目数目       |          | 100000，默认                                             |
| hive.groupby.skewindata              | 数据倾斜时负载均衡       |          | 默认 false                                               |
|                                      |                          |          |                                                          |
| hive.input.format                    | InpuFormat               |          | org.apache.hadoop.hive.ql.io.CombineHiveInputFormat 默认 |
| hive.exec.reducers.bytes.per.reducer | 每个 Reduce 处理的数据量 |          | 默认 256MB                                               |
| hive.exec.reducers.max               | 任务最大的 reduce 数     |          | 默认 1009                                                |
|                                      |                          |          |                                                          |
| hive.exec.parallel                   | 并行执行（多阶段）       |          |                                                          |
| hive.cli.print.current.db            | 打印当前数据库           | 显示     |                                                          |
|                                      |                          |          |                                                          |
| hive.metastore.uris                  | 元数据地址               |          | 如果为空，则为本地模式                                   |
| mapred.max.split.size=256000000      |                          |          |                                                          |
| hive.exec.reducers.bytes.per.reducer | 256000000                |          |                                                          |
| hive.exec.reducers.max               | 调整reduce个数：         |          |                                                          |







