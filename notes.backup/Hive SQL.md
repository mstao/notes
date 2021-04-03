## 练习

### 练习

```sql
-- ==================================================
-- 数据准备
-- ==================================================

create database if not exists module2;
create table if not exists t1
(
    team string,
    year int
) row format delimited fields terminated by ',';
load data local inpath "/root/data/t1.dat" into table t1;

create table if not exists t2
(
    id    string,
    time  string,
    price double
) row format delimited fields terminated by ',';
load data local inpath "/root/data/t2.dat" into table t2;

create table if not exists t3
(
    id       string,
    dt       string,
    browseid string
) row format delimited fields terminated by '\t';
load data local inpath "/root/data/t3.dat" into table t3;

-- ==================================================
-- SQL
-- ==================================================
```



1、找出全部夺得3连贯的队伍

```
team,year
活塞,1990
公牛,1991
公牛,1992
公牛,1993
火箭,1994
火箭,1995
公牛,1996
公牛,1997
公牛,1998
马刺,1999
湖人,2000
湖人,2001
湖人,2002
马刺,2003
活塞,2004
马刺,2005
热火,2006
马刺,2007
凯尔特人,2008
湖人,2009
湖人,2010
```



-- 连续值问题
-- 1、使用 row_number 进行数据排序(分组排序..)
-- 2、使用某列 - row_number = gid, 这个 gid 可以作为下一步分组的依据
-- 3、使用 gid 作为分区的依据，将数据分组，求的最终的结果

```sql
-- select team, year,
--        row_number() over (partition by team order by year),
--        year - (row_number() over (partition by team order by year))
-- from t1;
select team, count(*) countNum
from (
         select team,
                year,
                row_number() over (partition by team order by year),
                year - (row_number() over (partition by team order by year)) gid
         from t1
     ) tmp
group by team, gid
having countNum >= 3;
```



2、找出一天之内所有的波峰与波谷值

找出每个id在在一天之内所有的波峰与波谷值

```
id,time,price
sh66688,9:35,29.48
sh66688,9:40,28.72
sh66688,9:45,27.74
sh66688,9:50,26.75
sh66688,9:55,27.13
sh66688,10:00,26.30
sh66688,10:05,27.09
sh66688,10:10,26.46
sh66688,10:15,26.11
sh66688,10:20,26.88
sh66688,10:25,27.49
sh66688,10:30,26.70
sh66688,10:35,27.57
sh66688,10:40,28.26
sh66688,10:45,28.03
sh66688,10:50,27.36
sh66688,10:55,26.48
sh66688,11:00,27.41
sh66688,11:05,26.70
sh66688,11:10,27.35
sh66688,11:15,27.35
sh66688,11:20,26.63
sh66688,11:25,26.35
sh66688,11:30,26.81
sh66688,13:00,29.45
sh66688,13:05,29.41
sh66688,13:10,29.10
sh66688,13:15,28.24
sh66688,13:20,28.20
sh66688,13:25,28.59
sh66688,13:30,29.49
sh66688,13:35,30.45
sh66688,13:40,30.31
sh66688,13:45,30.17
sh66688,13:50,30.55
sh66688,13:55,30.75
sh66688,14:00,30.03
sh66688,14:05,29.61
sh66688,14:10,29.96
sh66688,14:15,30.79
sh66688,14:20,29.82
sh66688,14:25,30.09
sh66688,14:30,29.61
sh66688,14:35,29.88
sh66688,14:40,30.36
sh66688,14:45,30.88
sh66688,14:50,30.73
sh66688,14:55,30.76
sh88888,9:35,67.23
sh88888,9:40,66.56
sh88888,9:45,66.73
sh88888,9:50,67.43
sh88888,9:55,67.49
sh88888,10:00,68.34
sh88888,10:05,68.13
sh88888,10:10,67.35
sh88888,10:15,68.13
sh88888,10:20,69.05
sh88888,10:25,69.82
sh88888,10:30,70.62
sh88888,10:35,70.59
sh88888,10:40,70.40
sh88888,10:45,70.29
sh88888,10:50,70.53
sh88888,10:55,70.92
sh88888,11:00,71.13
sh88888,11:05,70.24
sh88888,11:10,70.37
sh88888,11:15,69.79
sh88888,11:20,69.73
sh88888,11:25,70.52
sh88888,11:30,71.23
sh88888,13:00,72.85
sh88888,13:05,73.76
sh88888,13:10,74.72
sh88888,13:15,75.48
sh88888,13:20,75.80
sh88888,13:25,76.74
sh88888,13:30,77.22
sh88888,13:35,77.12
sh88888,13:40,76.90
sh88888,13:45,77.80
sh88888,13:50,78.75
sh88888,13:55,78.30
sh88888,14:00,78.68
sh88888,14:05,78.99
sh88888,14:10,78.35
sh88888,14:15,78.37
sh88888,14:20,78.07
sh88888,14:25,78.80
sh88888,14:30,79.78
sh88888,14:35,79.72
sh88888,14:40,80.71
sh88888,14:45,79.92
sh88888,14:50,80.49
sh88888,14:55,80.44
```

```
最终结果与此类似：
id time   price  feature
sh66688    10:05  27.09  波峰
sh66688    10:15  26.11  波谷
sh66688    10:25  27.49  波峰
sh66688    10:30  26.7   波谷
sh66688    10:40  28.26  波峰
sh66688    10:55  26.48  波谷
sh66688    11:00  27.41  波峰
sh66688    11:05  26.7   波谷
```

-- 分别按照 id 分组找出最大值最小值对应的 time 和 price
-- 两表 union 后，按照特定排序得出最终的结果
-- 性能： 扫描表两次，最后 union join 后需全局排序进行格式化输出
-- 前一时刻的值(lag)、后一时刻的值(lead)
use module2;
with tmp as (select id, price,time,
        lag(price) over (partition by id order by time) lagprice,
        lead(price) over (partition by id order by time) leadprice
    from t2)
select id, price, time, feature
from (select id, price, time,
          case when price - nvl(lagprice, 0) >0 AND price - nvl(leadprice, 0) >0
          then '波峰'
          when price - nvl(lagprice, 0) <0 AND price - nvl(leadprice, 0) <0
          then '波谷'
          else null
          end feature
      from tmp) tmp2
where feature is not null;


-- ERROR
-- 每个id在的最大和最小
select id, time, price, feature
from (
         (select id, time, price, '波谷' as feature
          from (
              select id, time, price,
              rank() over (partition by id order by price) as rank
              from t2
              ) tmp
          where rank = 1)
         union all
         (select id, time, price, '波峰' as feature
          from (
              select id, time, price,
              rank() over (partition by id order by price desc) as rank
              from t2
              ) tmp2
          where rank = 1)
     ) unioned
order by id, price desc;



3.1、每个id浏览时长、步长

-- 字符日期解析并转换成 minute
-- 按照 Id 分组，进行最值、数量聚合得出结果

```sql
select id, max(timestamp_min) - min(timestamp_min) browse_time, count(*) step_length
from (select id, unix_timestamp(dt, 'yyyy/MM/dd HH:mm') / 60 timestamp_min
      from t3) tmp
group by id;
```



3.2 带有间隔的时长

-- 未作出
-- 得到 id, 时间(分钟), 间隔
-- 注入分组属性，带有循环的 case when? 配合间隔超过30的中间表数据? 自定义聚合函数？
-- lag      求差   与30比较(大于等于30=1)   求累计和(分组依据)
-- 1	2020/05/28 17:02	httpid1     null      0           0                     0
-- 1	2020/05/28 17:03	httpid2    17:02      1           0                     0
-- 1	2020/05/28 17:09	httpid3    17:03      6           0
-- 1	2020/05/28 17:12	httpid4    17:09      3           0
-- 1	2020/05/28 17:14	httpid5    17:12      2           0
-- 1	2020/05/28 17:16	httpid6    17:14      2           0
-- 1	2020/05/28 17:48	httpid7    17:16      32          1                     1
-- 1	2020/05/28 17:49	httpid8    17:48      1           0                     1
-- 1	2020/05/28 17:54	httpid9    17:49      5           0                     1
-- 1	2020/05/28 17:56	httpid0
-- 1	2020/05/28 17:57	httpid10
-- 1	2020/05/28 17:59	httpid11
-- select id, timestamp_min,
--        nvl(timestamp_min - lag(timestamp_min) over (partition by id order by timestamp_min), 0) timediff
-- from (select id, cast(unix_timestamp(dt, 'yyyy/MM/dd HH:mm') / 60 as int) timestamp_min
--          from t3) tmp;

```sql
with tmp2 as (
    select id, timestamp_min,
           nvl(timestamp_min - lag(timestamp_min) over (partition by id order by timestamp_min), 0) timediff
    from (select id, cast(unix_timestamp(dt, 'yyyy/MM/dd HH:mm') / 60 as int) timestamp_min
          from t3) tmp
)
select id, max(timestamp_min) - min(timestamp_min) resultdiff, count(*) -- 重新分组求值
from (select id, timestamp_min, timediff,
             sum(diff) over (partition by id order by timestamp_min rows between unbounded preceding and current row) sumdiff     -- 换个属性
      from (
               select id, timestamp_min, timediff,
                      case when timediff >= 30 then 1
                           else 0 end diff
               from tmp2   -- 加个属性
           ) tmp3) tmp4
group by id, sumdiff

-- ,  sum(diff);
```





会员分析中查找最近七天，连续3天会员数

```sql
with tmp as (select uid,
                    date_sub(dt, row_number() over (partition by uid order by dt)) gid
             from dws.dws_member_start_day
             where dt >= date_add(next_day('2020-07-23', 'mo'), -7) and dt <= '2020-07-23'
)
select count(*)
from (select uid, count(*) countNum
      from tmp
      group by uid, gid
      having countNum >= 3) tmp2;
```



## DDL

```sql
schematool -dbType mysql -initSchema

# 先启动 HDFS 服务、Yarn 服务
hive
SHOW functions;
SHOW databases;
create database test;

# ================================================
# DDL
# ================================================
create database if not exists mydb2;
create database if not exists mydb3;

CREATE DATABASE IF NOT EXISTS mydb1
COMMENT 'test comment'
LOCATION '/usr/hive/test/mydb.db';

drop database mydb1;
drop database mydb2;
drop database mydb3;

# ================================================
# DDL-建表
# ================================================





# ================================================
# DDL-内部表和外部表
# ================================================
# 默认内部表创建，外部表 external
# 删除内部表，定义和数据同时删除
# 外部表，不删除数据
-- Inner table
CREATE TABLE t1 (
    id int,
    name string,
    hobby array<string>,
    addr map<string, string>
)
row format delimited
fields terminated by ";"
collection items terminated by ","
map keys terminated by ":"


desc t1;
desc formatted t1;

-- Load data
load data local inpath '/home/hadoop/data/t1.dat' into table t1;
SELECT * FROM t1;
dfs -ls /usr/hive/warehouse/mydb.db/t1
dfs -cat /usr/hive/warehouse/mydb.db/t1

CREATE external TABLE t1 (
    id int,
    name string,
    hobby array<string>,
    addr map<string, string>
)



-- 内部表与外部表的转换
ALTER TABLE t1 set tblproperties("EXTERNAL"="TRUE");
DESC t1;

# ================================================
# DDL-分区表
# ================================================
-- 只查询部分分区数据时，可避免全表扫描，提高查询效率
-- 生产中用的较多， 按照时间、地域进行划分
create table if not exists t3 (
    id int,
    name string,
    hobby array<string>,
    addr map<string,string>
)
partition by (dt string)
row format delimited
fields terminated by ':'
collection items terminated by ','
map keys terminated by ':';

-- 静态导入指定的分区
load data local inpath "/home/hadoop/data/t1.dat"
into table t3
partition(dt="2020-06-01");

load data local inpath "/home/hadoop/data/t1.dat"
into table t3
partition(dt="2020-06-02");

-- 分区字段不是表中已经存在的数据，伪列，目录性质



-- 查看分区表中的内容
dfs -ls /usr/hive/warehouse/mydb.db/t3/dt=2020-06-01;
show partititon t3;

alter table add partition(dt="2020-06-03");
alter table add partition(dt="2020-06-04");

dfs -ls /usr/hive/warehouse/mydb.db/t3/dt=2020-06-01;
dfs -cp /usr/hive/warehouse/mydb.db/t3/dt=2020-06-01 /usr/hive/warehouse/mydb.db/t3/dt=2020-06-08;

-- 加载指定位置下的分区
alter table add partition(dt="2020-06-07")
location '/usr/hive/warehouse/mydb.db/t3/dt=2020-06-07';

-- 更改分区表的位置
alter table t3 partition(dt="2020-06-07") set location '/usr/hive/warehouse/mydb.db/t3/dt=2020-06-08'

-- 删除分区表中的分区
alter table t3 drop partition (dt="2020-06-04"),partition(dt="2020-06-05");



# ================================================
# DDL-分桶表
# ================================================
-- 分桶字段.hashCode % 桶的个数
create table course (
    id int,
    name string,
    score int
)
clustered by (id) into 3 buckets
row format delimited fields terminated by "\t";


create table course_common(
    id int,
    name string,
    score int
)
row format delimited fields terminated by "\t";
--
load data local inpath '/home/hadoop/data/course.dat' into table course_common;


-- insert .. select .. 将桶中的数据加载进桶中
insert into table course select * from course_common;
dfs -ls /usr/hive/warehouse/mydb.db/course/xx


# ================================================
# DDL-修改和删除表
# ================================================
alter table course_common rename to course_common1;
alter table course_common1 change column name subject string;

alter table course_common1 change column id id string;
alter table course_common1 change column score score string;
-- string => int ...
alter table course_common1 change column score score int;
alter table course_common1 add columns (common string);
alter table course_common1 replace colomns (id string, subject string, score string);

drop table course_common1;

-- 表
-- 内部表
-- 外部表
-- 分区表
-- 分桶表
```



## DML

```sql
-- ================================================
-- HQL-HML - 事务操作
-- ================================================
-- 这些参数也可以设置在hive-site.xml中
SET hive.support.concurrency = true;

-- Hive 0.x and 1.x only
SET hive.enforce.bucketing = true;
SET hive.exec.dynamic.partition.mode = nonstrict;
SET hive.txn.manager = org.apache.hadoop.hive.ql.lockmgr.DbTxnManager;

create table if not exists zxz_data(
    name string,
    nid int,
    phone string,
    ntime date
)clustered by (nid) into 5 buckets
stored as orc
tblproperties ('transactional'='true');


create table if not exists temp1(
    name string,
    nid int,
    phone string,
    ntime date
)row format delimited fields terminated by ",";
load data local inpath '/home/hadoop/data/zxz_data.txt'
overwrite into table temp1;

select * from zxz_data;
dfs -ls /user/hive/warehouse/mydb.db/zxz_data ;


delete from zxz_data
where nid = 3;

dfs -ls /user/hive/warehouse/mydb.db/zxz_data ;


-- insert into zxz_data values ("name3", 3, "010-83596208", current_date);

insert into zxz_data values ("name3", 3, "010-83596208", "202006-01"); -- 执行

insert into zxz_data
select "name3", 3, "010-83596208", current_date;
dfs -ls /user/hive/warehouse/mydb.db/zxz_data ;


insert into zxz_data
values ("name6", 6, "010-83596208", "2020-06-02"),
       ("name7", 7, "010-83596208", "2020-06-03"),
       ("name8", 9, "010-83596208", "2020-06-05"),
       ("name9", 8, "010-83596208", "2020-06-06");
dfs -ls /user/hive/warehouse/mydb.db/zxz_data ;
update zxz_data
set name=concat(name, "00")
where nid > 3;
dfs -ls /user/hive/warehouse/mydb.db/zxz_data ;



-- 分桶字段不能修改，下面的语句不能执行

-- Updating values of bucketing columns is not supported
update zxz_data set nid = nid + 1;
```



### 导入导出

```sql
-- ================================================
-- HQL-装载数据
-- ================================================
-- load data [local] inpath 'filepath'
create table tabA(
    id int,
    name string,
    area string
) row format delimited fields terminated by ',';

~/data/sourceA.txt

1,fish1,SZ
2,fish2,SH
3,fish3,HZ
4,fish4,QD
5,fish5,SR

hdfs fs -put sourceA.txt data/


-- Local FS 原始数据保留
load data local inpath '/home/hadoop/data/sourceA.txt' into table tabA;

-- HDFS 中的数据被移动  HDFS 一般都有副本, 防止过多的 replication
load data inpath 'data/sourceA.txt' into table tabA;

load data inpath 'data/sourceA.txt'
overwrite into table tabA;

-- 创建表与加载数据合并
hdfs dfs -mkdir /usr/hive/tabB
hdfs dfs -put sourceA.txt /usr/hive/tabB

create table tabB(
    id int,
    name string,
    area string
) row format delimited fields terminated by ','
location '/usr/hive/tabB';


-- ================================================
-- HQL-插入数据
-- ================================================
create table tabC(
    id int,
    name string,
    area string
)
partitioned by (month string)
row format delimited fields terminated by ',';

-- insert 多条数据
insert into table tabC partition(month='2020001')
values(5, 'wangwu', 'BJ'),(4, 'lishi', 'SH'),(3, 'zhangsan', 'TJ');
-- 多一个伟列
select * from tabC;


-- 插入多个分区中的数据
from tabC
insert overwrite table tabC partition(month='202003')
select id, name, area, where month='202002'
insert overwrite table tabC partition(month='202004')
select id, name, area where month='202002';



create table if not exists tabD
as select * from tabC;


import table student2 partition(month='201709')
from '/usr/hive/warehouse/exort/student';
```



数据导出

```sql
-- ================================================
-- HQL-数据导出
-- ================================================
-- 导出到 Local FS
insert overwrite local directory '/home/hadoop/data/tabC'
select * from tabC;

-- 格式化导出到 Local FS
insert overwrite local directory '/home/hadoop/data/tabC2'
row format delimited fields terminated by ' '
select * from tabC;

-- 导出到 HDFS
insert overwrite directory '/usr/hadoop/data/tabC3'
row format delimited fields terminated by ' '
select * from tabC;
hdfs dfs -cat /usr/hadoop/data/tabC;

-- 直接从 HDFS 中复制出来
dfs -get /usr/hive/warehouse/mydb.db/tabc/month=202001 /home/hadoop/data/tabC4;

-- 使用 Hive 导出，将结果重定向到文件  默认使用 defacult 数据库
hive -e "select * from mydb.tabC" > a.log;

-- export 导出整个表
-- 包含表的元数据信息
export table tabC to '/usr/hadoop/data/tabC4';

-- 将 export 的数据从新导入
-- like <table-name> 表结构与原来表一致， 分区表
-- 使用 create ... as select .. 结构可能不一致
create table tabE like tabC;
import table tabE from '/usr/hadoop/data/tabC4';

-- 截断表，近适用于内部表
truncate table tabE;
alter table tabC set tblproperties("EXTERNAL"="TRUE");
-- 执行失败
truncate table tabC;


-- ================================================
-- 其他方式
-- ================================================
-- Sqoop 、DataX 进行数据的导入导出
```



## DCL

create role

grant role

revoke role

drop role





## 函数

```sql
SHOW FUNCTIONS;
DESC FUNCTION <function_name>;
DESC FUNCTION EXTENDED <function_name>;
```



### *日期函数

```SQL
-- ================================================
-- 日期函数(##)
-- ================================================
select current_date;
select current_timestamp();
-- select unix_timestamp();

-- unix 时间戳转换
select from_unixtime(11111111);
select from_unixtime(11111, 'yyyyMMdd');
select from_unixtime(1111111, 'yyyy-MM-dd HH:mm:ss');
-- 日期转成时间戳
select unix_timestamp('2019-09-15 14:23:00');
-- 时间差
select datediff('2020-04-18', '2019-11-21');
select abs(datediff('2020-04-18', '2019-11-21'));
select datediff('2019-11-21', '2020-04-18');

-- 月的相对查询
select dayofmonth(current_date);
select last_day(current_date);
-- 当月第一天
select date_sub(current_date, dayofmonth(current_date) -1);
-- 下月第一天
select add_months(date_sub(current_date, dayofmonth(current_date) - 1), 1)

-- must 字符串转换成 date yyyy-MM-dd
select to_date('2020-01-01');
select to_date('2020-01-01 12:12:12');

-- 日期格式化成指定的字符串
select date_format(current_timestamp(), 'yyyy-MM-dd HH:mm:ss');
select date_format(current_date(), 'yyyyMMdd');
select date_format('2020-06-01', 'yyyy-MM-dd HH:mm:ss');

-- 计算每个人的工龄
select * ,
  round(datediff(current_date, hiredate) / 365, 1) workingyears
from emp;
```



### **条件函数**

if

Case when: 使用较多

Coalesce:

Is null / isnotnull:

nvl

nulif:

Explode: 配合 lateral view 于 explode 联用，解决 uDTF 不能添加额外列的问题



```sql
-- ================================================
-- 条件函数(#)
-- ================================================
-- if, case when, coalesce, isnull/isnotnull, nvl,nullif

-- if (boolean testCondition, T valueTrue, T valueFalseOrNull)
-- 将emp表的员工工资等级分类：0-1500、1500-3000、3000以上
select sal,
  if(sal <= 1500, 1, if(sal <= 3000, 2, 3))
from emp;

-- 复杂条件
select sal, case when sal <= 1500 then 1
                 when sal <= 3000 then 2
                 else 3 end sallevel
from emp;

-- 以下语句等价                     常用
select ename, deptno,
       case  when deptno=10 then 'accounting'
             when deptno=20 then 'research'
             when deptno=30 then 'sales'
             else 'unknown' end deptname
from emp;

select ename, deptno,
       case deptno when 10 then 'accounting'
                   when 20 then 'research'
                   when 30 then 'sales'
                   else 'unknown' end deptname
from emp;


-- COALESCE(T v1, T v2, ...)
-- 返回参数中的第一个非空值；如果所有值都为 NULL，那么返回NULL
select sal, coalesce(comm, 0) from emp;

-- isnull(a) isnotnull(a)          常用
select * from emp where isnull(comm);
select * from emp where isnotnull(comm);

-- 空值转换函数 nvl(T value, T default_value)
select empno, ename, job, mgr, hiredate, deptno, sal + nvl(comm, 0) sumsal
from emp;

-- nullif(x, y) 相等为空，否则为x  ??
SELECT nullif("b", "b"), nullif("b", "a");
```



### 字符串函数

```sql
-- ================================================
-- 字符串函数
-- ================================================
select lower("HELLO WORLD");
select lower(ename), ename from emp;
select length(ename), ename from emp;

-- 子串截取
select split("www.lagou.com", "\\.");

-- 字符串拼接  指定分割符拼接 concat_ws(separator, [string | array(string)]+)
select empno || " " || ename idname from emp;
select concat(empno, " " ,ename) idname from emp;
SELECT concat_ws('.', 'www', array('lagou', 'com'));
select concat_ws(" ", ename, job) from emp;


SELECT substr('www.lagou.com', 5);
SELECT substr('www.lagou.com', -5);

-- 字符分割
SELECT substr('www.lagou.com', 5, 5);
```



### 数学函数

```sql
-- ================================================
-- 数学函数
-- ================================================
-- 四舍五入
select round(314.15926);
select round(314.15926, 2);
select round(314.15926, -2);

-- 向上取整。ceil
select ceil(3.1415926);

-- 向下取整。floor
select floor(3.1415926);
```



### *分析函数

***窗口函数：**

聚集函数：

序列函数：

排名函数：



窗口函数

```sql
create table if not exists emp
(
    empno    int,
    ename    string,
    job      string,
    mgr      int,
    hiredate date,
    sal      int,
    comm     int,
    deptno   int
) row format delimited fields terminated by ",";
-- ================================================
-- 窗口函数
-- ================================================
-- 用于计算基于组的某种聚合值，对于每个组返回多行，而聚合函数对于每个组只返回一行
-- 指定了分析函数工作的数据窗口大小，这个数据窗口大小可能会随着行的 变化而变化

-- 窗口函数是针对每一行数据的
-- over中没有参数，默认的是全部结果 集；
-- 通过over()进行开窗

-- over 关键字，为 分析函数
select ename, sal, sum(sal) over () salsum
from emp;

select ename, sal, sum(sal) over () salsum,
       concat(round(sal / sum(sal) over () * 100, 1), '%') ratiosal
from emp;


-- =======================
-- partition by
-- =======================
-- 在over窗口中进行分区，对某一列进行分区统计，窗口的大小就是分区的大小
-- -- 查询员工姓名、薪水、部门薪水总和
select ename, sal,
    sum(sal) over(partition by deptno) salsum
from emp;


-- =======================
-- order by 子句
-- =======================
-- sum：从分组的第一行到当前行求和
select ename, sal, deptno,
       sum(sal) over(partition by deptno order by sal) salsum
from emp;

-- unbounded preceding: 组内第一行数据
-- n preceding: 组内当前行的前n行数据
-- current row: 当前行数据
-- n following: 组内当前行的后n行数据
-- unbounded following: 组内最后一行数据


-- =======================
-- window 子句
-- =======================
-- rows between ... and ...
select ename, sal, deptno,
       sum(sal) over (partition by deptno order by ename)
from emp;
-- 等价。组内，第一行到当前行的和
select ename, sal, deptno,
       sum(sal) over (partition by deptno order by ename rows between unbounded preceding and current row )
from emp;

-- 组内，第一行到最后一行的和
select ename, sal, deptno,
       sum(sal) over (partition by deptno order by ename rows between unbounded preceding and unbounded following )
from emp;

-- 组内，前一行 + 当前行 +后一行
select ename, sal, deptno,
       sum(sal) over(partition by deptno order by ename rows between 1 preceding and 1 following )
from emp;
```

排名函数 

```sql
-- =======================
-- 排名函数 - 常用 TopN
-- =======================
-- row_number(): 排名顺序增加不会重复；如1、2、3、4、... ...
-- RANK():       排名相等会在名次中留下空位；如1、2、2、4、5、... ...
-- DENSE_RANK(): 排名相等会在名次中不会留下空位 ；如1、2、2、3、4、... ...
create table if not exists t2(
    cname string,
    sname string,
    score int
) row format delimited fields terminated by ' ';
load data local inpath '/home/hadoop/data/t2.dat' into table t2;
select * from t2;

-- 按照班级，使用3种方式对成绩进行排名
select cname, sname, score,
       row_number() over (partition by cname order by score desc) rank1,
       rank() over (partition by cname order by score desc) rank2,
       dense_rank() over (partition by cname order by score desc) rank3
from t2;

-- 求每个班级前3名的学员  语法格式
select cname, sname, score, rank
from (select cname, sname, score,
             dense_rank() over (partition by cname order by score) rank
      from t2) tmp
where rank <= 3;
```

序列函数

```sql
-- =======================
-- 序列函数
-- =======================
-- lag:          返回当前数据行的上一行数据       常用
-- lead:         返回当前数据行的下一行数据      常用
-- first_value:  取分组内排序后，截止到当前行，第一个值
-- last_value:   分组内排序后，截止到当前行，最后一个值
-- ntile:        将分组的数据按照顺序切分成n片，返回当前切片值
create table userpv (
    cid string,
    ctime date,
    pv int
) row format delimited fields terminated by ',';
load data local inpath '/home/hadoop/data/userpv.dat' into table userpv;
select * from userpv;

-- lag, lead
select cid, ctime, pv,
       lag(pv) over (partition by cid order by ctime) lagpv,
       lead(pv) over (partition by cid order by ctime)  leadpv
from userpv;

select cid, ctime, pv,
       lag(pv, 2) over (partition by cid order by ctime) lagpv,
       lead(pv, 3) over (partition by cid order by ctime)  leadpv
from userpv;

-- first_value / last_value
select cid, ctime, pv,
       first_value(pv) over (partition by cid order by ctime
           rows between unbounded preceding and unbounded following) firstpv,
       last_value(pv) over (partition by cid order by ctime
           rows between unbounded preceding and unbounded following) lastpv
from userpv;

-- ntile 按照cid进行分组，每组数据分成2份
select cid, ctime, pv,
       ntile(2) over (partition by cid order by ctime) ntile
from userpv;
```





### 自定义函数

UDF

```sql
-- ================================================
-- 自定义函数
-- ================================================
-- UDF（User Defined Function）。用户自定义函数，一进一出
-- UDAF（User Defined Aggregation Function）。用户自定义聚集函数，多进一 出；类似于：count/max/min
-- UDTF（User Defined Table-Generating Functions）。用户自定义表生成函数，一进多出；类似于：explode

-- =======================
-- 临时性使用
-- =======================
-- hive command line
add jar /home/hadoop/hiveudf.jar;

create temporary function mynvl as "com.janhen.bigdata.hive.nvl";
-- 基本功能还有
select mynvl(comm, 0) from mydb.emp;

-- 测试扩充的功能
select mynvl("", "OK");
select mynvl(" ", "OK");


-- =======================
-- 永久函数
-- =======================
// language shell
-- hdfs dfs -put hiveudf.jar jar/

create function mynvl2 as 'com.janhen.bigdata.hive.nvl'
    using jar 'hdfs:/user/hadoop/jar/hiveudf.jar';

show functions;
drop function mynvl2;
show functions;
```



## *DQL

排序字句：

order by: 全局有序

sort by： 每个 MR 内部有序

distribute by: 分区排序，将数据按照 distribute by 字段分区

Cluster by： distrubute by 于 sort by 同一个字段时使用



1、where 子句

**<font color="red">where 子句中不能使用列的别名；</font>**

`[NOT] BETWEEN ... AND ...`: 范围的判断，使用NOT关键字结果相反。

`RLIKE`、 `REGEXP`: 基于java的正则表达式，匹配返回TRUE，反之返回FALSE。匹配使用的是JDK中的正则表达式接口实现的，因为正则也依据其中的规则。

<font color="green">NULL<=>NULL的结果为true</font>

```sql
-- 使用 rlike。正则表达式，名字以A或S开头 
select ename, sal 
from emp 
where ename rlike '^(A|S).*';
```

2、group by 子句

```sql
-- 计算emp每个部门中每个岗位的最高薪水 
select deptno, job, max(sal) 
from emp 
group by deptno, job;
```

```sql
-- 求每个部门的平均薪水大于2000的部门 
select deptno, avg(sal) 
from emp 
group by deptno 
having avg(sal) > 2000;
```

4、join 子句
Hive总是按照从左到右的顺序执行，Hive会对每<font color="green">对 JOIN 连接对象启动一个 MapReduce 任务</font>。

```sql
-- 多表连接查询，查询老师对应的课程，以及对应的分数，对应的学生
select *
from techer t 
  left join course c on t.t_id = c.t_id 
  left join score s on s.c_id = c.c_id 
  left join student stu on s.s_id = stu.s_id;
```



### 排序

1、全局排序

<font color="green">ORDER BY执行全局排序，只有一个reduce；</font>

```sql
-- 按别名排序 
select empno, ename, job, mgr, sal + nvl(comm, 0) salcomm, deptno 
from emp 
order by salcomm desc;
```

```sql
-- 排序字段要出现在select子句中。以下语句无法执行（因为select子句中缺少 deptno）： 
select empno, ename, job, mgr, sal + nvl(comm, 0) salcomm 
from emp 
order by deptno, salcomm desc;
```

```sql
-- ================================================
-- HQL-单表查询
-- ================================================
-- 测试数据 /home/hadoop/data/emp.dat
create table if not exists emp(
    empno int,
    ename string,
    job string,
    mgr int,
    hiredate date,
    sal int,
    comm int,
    deptno int
) row format delimited fields terminated by ",";
load data local inpath '/home/hadoop/data/emp.dat' into table emp;
select * from emp;

-- 函数和语句基本计算
select 8*888;
select current_date;

-- 结果分页，限制
select * from emp limit 3;

-- null 判断
select * from emp where comm is null;

-- 正则匹配，使用 rlike。正则表达式，名字以A或S开头
select ename,sal from emp where ename rlike '^(A|S).*';


-- 分组查询 - 多个分区
select deptno, job, max(sal)
from emp
group by deptno, job;

-- 分组筛选
select deptno, avg(sal)
from emp
group by deptno
having avg(sal) > 2000;



-- ================================================
-- HQL-连接查询
-- ================================================
-- 多表 join
select *
  from techer t left join course c on t.t_id=c.t_id
                left join score s on s.c_id = c.c_id
                left join student stu on s.s_id = stu.s_id;

-- 按照从左到右的顺序
-- Hive 会对每一个 Join 连接对象 启动一个 MapReduce 任务



-- =======================
-- 笛卡尔积
-- =======================
-- 默认不支持，需要手动开启
set hive.strict.checks.cartesian.product=false;
select * from u1, u2;



-- ================================================
-- HQL-排序(重点）
-- ================================================
-- =======================
-- 全局排序
-- =======================
-- 排序字段需要出现在 select 字段中
-- Error..
-- select empno ename, job, mgr, sal + nvl(comm, 0) salcomm
-- from emp
-- order by deptno, salcomm desc;


-- =======================
-- MR 的内部排序(sort by)
-- =======================
-- sort by 每个 reduce 产生排序文件，局部有序
-- reduce 内部排序
set mapreduce.job.reduces = 2;

select * FROM emp SORT BY sal DESC;

-- reduce 中的文件局部有序
insert overwrite local directory '/home/hadoop/output'
select * from emp sort by sal desc;


-- =======================
-- MR 分区排序(distribute by)
-- =======================
-- 将特定的行发送到特定的 reducer 中
-- distribute by 要写在sort by之前
-- 可结合sort by操作，使分区数据有序
-- 类似与 MR 中的分区操作
-- 按照指定的条件将数据分组，常结合 sort by 使用

-- 先按 deptno 分区，在分区内按照 sal + comm 排序
set mapreduce.job.reduces=2;
select empno, ename, job, deptno, sal + nvl(comm, 0) salcomm
  from emp
distribute by deptno
sort by salcomm desc;

insert overwrite local directory '/home/hadoop/output/disBy'

-- deptno % 2  20，30，40。。。 都放到同一个分区中，结果只有一个有数据的文件


set mapreduce.job.reduces=3;
insert overwrite local directory '/home/hadoop/output/distBy1'
select empno, ename, job, deptno, sal + nvl(comm, 0) salcomm
  from emp
distribute by deptno
sort by salcomm desc;


-- =======================
-- MR - Cluster By
-- =======================
-- distribute by 与 sort by 为同一个字段时，使用 cluster by 简化语法
-- 只能是剩下的，不能指定排序规则
select * from emp distribute by deptno sort by deptno;
select * from emp cluster by deptno;
```





