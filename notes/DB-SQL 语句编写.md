# 结构定义

**字段**

（1）  数值

1\. 整数数值
int、long、tinyint

MySQL 中无 boolean，通过 int(1) 来实现的；

```
id int(11)
parent_id int(11)
status tinyint(1)
sort_order int(4)
```
2\. 小数数值

float

decimal(a, b)
decimal(20,2)



（2）  字符

1\. 一般字符
varchar、char

```
name varchar(50)
```
2\. 大字符
text ： 存储 json 格式数据， 存储 html 格式文件



（3）  时间

- date
- datetime ：作为时间戳，记录出现的点，作为 "后悔药"

- time:

// TODO 对应所占的字节数

// TODO 实现类型的转换

```sql
create_time datetime
update_time datetime
-- MySQL5.7 支持的语句进行自动更新
```



索引

（1） unique
作用 ：在分布式中保证唯一确定性，主要用于约束，如用户名、订单号等；

可作用于多个属性，如秒杀中 (user_id, goods_id) ；

```sql
UNIQUE KEY `user_name_unique` (`username`) USING BTREE
-- 组合起来的唯一索引
UNIQUE KEY `udx_userId_orderId` (`user_id`,`goods_id`) USING BTREE
```

（2） 普通索引
作用 ：仅用于提高查询速度

```sql
KEY `order_no_index` (`order_no`) USING BTREE
```

（3） 组合索引
常查询的进行建立，减少查询时间；

需要符合最左匹配原则使用；

```sql
KEY `order_no_user_id_index` (`user_id`,`order_no`) USING BTREE
```



（4） 主键索引

用于作为主键，可以包含多个字段；







约束

（1） 普通的约束

1\. not null
2\. default，defalut '23'
default null
3\. unique
4\. primary key

```

```



2） 外键约束

constrant key xxx_fk 



（3） 触发器完成约束的维护





**表**

创建表

```SQL
CREATE TABLE mytable (
	id INT NOT NULL AUTO_INCREMENT,
    col1 INT NOT NULL DEFAULT 1,
    col2 VARCHAR(45) NULL,
    col3 DATE NULL,
    PRIMARY KEY(`id`)
);
```



# 更新

```
rename database hivemetadata to hivemetadatabackup;
```

批量重命名表

```shell
old_db="hivemetadata"
new_db="hivemetadatabackup"
password="$MYSQL_ROOT_PASSWORD"
for table in $(mysql -u root -p$password -s -N -e "use $old_db;show tables from $old_db;");
do
  mysql -u root -p$password -s -N -e "use $old_db;rename table ${old_db}.$table to ${new_db}.$table;";
done
```



## 结构更新

**表的操作**

（4） 删除表

```sql
DROP TABLE mytable;
```

() 删除表的数据

// TODO DROP、TRUNCATE、DELETE 三种操作的区别

```SQL
-- ?? 
TRUNCATE TABLE mytable;
```



**修改表**

（1） 添加列

```sql
ALTER TABLE mytable ADD col CHAR(20);
```

（2） 修改列属性

```sql
ALTER TABLE 
-- 增大 varchar 长度
-- 更改列的名称
-- 更改列的字符集
-- 删除该列
```

（3） 删除列

```sql
ALTER TABLE mytable DROP COLUMN col;
```





## 数据更新

**插入数据**

（1）  普通插入

```sql
-- 全量插入
INSERT INTO mytable VALUES(val1, val2);			
-- 插入部分字段
INSERT INTO mytable(col1, col2) VALUES (val1, val2);         
```



（2）  插入检索出来的数据

在有 95% 是无用数据的情况下，可用 SELECT 筛选出 5% 有用的数据，将其插入到一张新的表中。

```sql
-- 指定的两列，主键必须存在
INSERT INTO mytable(col1, col2)
  SELECT col1, col2
  FROM mytables;
-- 插入所有
INSERT INTO Table2 SELECT  *  FROM Table1
```



（3） 筛选出数据并插入到创建的新表中

包含表的构建，类似原型构建

```sql
-- 无需显示指明列的类型和名称，自动根据查询结构进行构建，连同对应的数据
CREATE TABLE newtable AS 
  SELECT * FROM mytable
```





**更新特定字段的值**

```sql
UPDATE mytable
  SET col = val 
  WHERE id = 1;
```





 DELETE

返回被删除的记录数、可进行 group by、limit、where 进行选择限定、添加 where 自增字段可不恢复为 1

```sql
DELTE FROM mytabe
  WHERE id=1;
```



TRUNCATE

清空表，删除所有行

① 重置 AUTO_INCREMENT 标识为1

② 性能高

```sql
TRUNCATE TABLE mytable;
```



DROP

属于 DDL ，定义语言，能够将表的结构和数据全部删除



DELETE、DROP、TRUNCATE 的区别

① 性能： TRUNCATE 高

② AUTO_INCREMENT： TRUNCATE 重置为1

③ 命令格式，操作的，定义的







# 查询

SQL 中 SELECT 语句完整执行顺序：

> from ---> where --> group by ---> having ---> 计算所有的表达式--->order by--->select输出

**where 过滤**

范围过滤

① 大小： >=, <=, !=, >, <

② 指定几个： IN, NOT IN

③ 范围： BETWEEN xx AND xx

④ NULL 值： 通常用于 LEFT JOIN 中进行控制，对于一些函数自动跳过对于 NULL 值的列，从而得到最终的结果。  特定的面向对象封装也要注意对于 NULL 的 where 判断

⑤ AND： 多个逻辑条件

⑥ OR： 任意一个满足条件即可，在索引使用上使用该关键字使得索引失效，同时 OR 关键字是以往进行 SQL 注入攻击的一种方式， 也可以作为以往动态参数的拼接 OR 1 = 1；





**字段筛选**

用来作为 SELECT 后面接着的情况

① 简单计算： 通过进行计算一些列来实现，参数可为各个不同的列，也可以为指定的数；

② 函数计算：  通过一些字符串函数、统计函数实现拼接展示使用；

③ 查询语句结果字段： 直接写一个 SQL 语句，让其结果作为展示字段；

```sql
-- 计算的结果
SELECT col1 * col2 AS alias
FROM mytable;
```

```sql
-- 字符拼接的结果
SELECT CONCAT(TRIM(col1), '(', TRIM(col2), ')') AS concat_col
FROM mytable
-- 去重的结果
SELECT DISTINCT col FROM mytable;
SELECT COUNT(DISTINCT col) FROM mytable;
```







**GROUP By**

   (1) 出现在select后面的字段 要么是是聚合函数中的,要么就是group by 中的.
   (2) 要筛选结果 可以先使用where 再用group by 或者先用group by 再用having



分组规定

（1） 语句顺序

WHERE -- GROUP BY 



（2） SELECT 字段

除了 汇总字段外， SELECT 中的每一个字段都必须在 GROUP BY 子句中给出；



（3） NULL 处理

NULL 会单独分为一组



 GROUP BY

拼接

HAVING

Statistics: COUNT, SUM, AVG, MAX, MIN





GROUP BY 限制： 

- SELECT 子句中的列明必须包含 分组列 或是该列的函数(COUNT...)
- 列函数对于 GROUP BY 子句定义的每个组各返回一个结果
- 出现表的字段， 列为 GROUP BY 中的列 OR COUNT().. OR 别的表的列

```sql
-- 查询所有同学的学号、选课树、总成集
GROUP BY student_id                                     -- 为组
SELECT student_id, COUNT( course_id ), SUM( score )    -- 字段
FROM score                                    -- 来自哪些表
⇒ 
SELECT student_id, COUNT( course_id ), SUM( score )
FROM score
GROUP BY student_id;
```

```sql
-- 查询所有同学的学号、姓名、选棵数、总成绩
SELECT student_id, name, COUNT( course_id ), SUM( score )
FROM score s, student stu
WHERE s.student_id=stu.student_id      -- WHERE before GROUP BY
GROUP BY s.student_id;
```

```sql
EXPLAIN SELECT student_id, COUNT( course_id ), SUM( score )
FROM score
GROUP BY student_id;       -- 
```



![1551857013614](.\assets\1551857013614.png)







多个字段

>  GROUP BY X, Y意思是将所有具有相同X字段值和Y字段值的记录放到一个分组里。

我们下面再接着要求统计出每门学科每个学期有多少人选择，应用如下SQL：

```SQL
SELECT Subject, Semester, Count(*)
FROM Subject_Selection
GROUP BY Subject, Semester
```

```SQL
SELECT barcode, COUNT( barcode ) AS total
FROM db.shopSku
WHERE shop=#{} AND barcode in (#{}, #{}, ...)
GROUP BY shop, barcode     -- two column group
```





HAVING

```sql
# 查询平均成绩大于 60 分的同学的学号和平均成绩
SELECT student_id, AVG( score )
FROM score
GROUP BY studnet_id
HAVING AVG( score )>60
```

```sql
# 查询所有没有学全所有课的同学的学号、姓名
SELECT stu.student_id, stu.name
FROM student stu, score s
WHERE stu.id=s.id
GROUP BY stu.student.id 
HAVING COUNT(*) < 
(SELECT * FROM course)
```





**ORDER BY 的排序**

相同列时的特殊处理情况

```sql
SELECT *
FROM mytable
ORDER BY col1 DESC, col2 ASC;
-- 
ORDER BY  f1 DESC, f2 DESC
```







## 子查询

子查询作为 SELECT 字段

子查询结果作为 WHERE 过滤条件





## 表连接查询

内连接

又称等值连接
在没有条件语句的情况下返回笛卡尔积。



自然连

自然连接是把同名列通过等值测试连接起来的，同名列可以有多个。

内连接和自然连接的区别：内连接提供连接的列，而自然连接自动连接所有同名列。

```sql
SELECT A.value, B.value
FROM tablea AS A 
NATURAL JOIN tableb  AS B
```





外连接

外连接保留了没有关联的那些行。



（1） 左连接

注意在查询的字段中存在多张表的情况下的使用，看最终想要保存的结果集，在两个表中都含有相同的字段的情况下，选择左连接左边那张表的字段，不能够是右边那张表。

以左表为基准进行查询,左表数据会全部显示出来,右表如果和左表匹配的数据则显示相应字段的数据,如果不匹配,则显示为NULL;





## 模糊分页查询

模糊查询

通过 LIKE 关键字实现

- `%` 匹配 >= 0 个任意字符

- `_` 匹配 == 1 个任意字符

- `[]` 匹配集合内的字符， ^ 代表非

```sql
SELECT *
FROM mytable
WHERE col LIKE '[^AB]%'; -- 不以 A 和 B 开头的任意文本
```



限制不同

通过 DISTINCT 实现，可以作用在 SELECT 中字段的前面，也可作为内置函数的 参数传入

（1） COUNT( DISTINCT( col ) )

相同值只会出现一次。它作用于所有列，也就是说所有列的值都相同才算相同。

```sql
SELECT DISTINCT col1, col2
FROM mytable
```





限制个数

LIMIT offset count

（1） 查找倒数第二个

ORDER BY col DESC

LIMIT 1 1

```sql
SELECT * 
FROM mytable
ORDER BY col DESC
LIMIT 1 1;
```

（2） 找出前5 个

ORDER BY col ASC

LIMIT 5

```sql
SELECT * 
FROM mytable
ORDER BY col
LIMIT 5
```

// TODO 对于分页的优化

不需要具体的页数，只有下一页

缓存前几页的数据







# 内置机制

## 函数

如何在插入一行数据后，让当前线程立即获得该ID？
insert之后这个字段立马有值了。

内置函数

- now()   ： 处理时间戳
  返回yyyy-MM-dd hh-mm-ss格式的数据
   |-用于自动处理时间戳
   |-
- LAST_INSERT_ID()：自动返回最后一个insert或者update查询， 为auto_increment列设置的第一个发生的值

 password()用于生成加密的数据，作为密码





**聚合函数**

① COUNT() 

② SUM()

③ MAX()

④ MIN()

⑤ AVG()： 会忽略到 NULL 行，不显示在结果中

上述配合 DISTINCT 进行对应的逻辑过滤

```sql
SELECT AVG(DISTINCT col1) AS avg_col
FROM mytable;
```







流程控制函数

 case 有两种一种等值 一种值比较

 case 值 when 比较值 then 值 .... else 值 end

 值比较利用比较运算符

 case when 比较过程 then 值 .... else 值 end  

 if(func,a,b) 等价于三目运算 func 为真 返回a 为假返回 b

 ifnull(a,b) 判断a是否为null 是返回b不是返回a

 nullif(a,b) 判断a，b是否相等 是就null 不是就a

 

语句

（） IF(expr1, expr2, expr2)

```sql
SELECT IF(1 > 2, 2, 3);   -- 3
SELECT IF(STRCMP(test, test1), yes, no);       -- no
SELECT IF(0.1, 1, 0)              -- 0
SELECT IF(0.1 <> 0, 1, 0);         -- 1
```

（1） IFNULL

IFNULL(expr1, expr2)

如果expr1不是NULL，IFNULL()返回expr1，否则它返回expr2。IFNULL()返回一个数字或字符串值，取决于它被使用的上下文环境。   

```sql
SELECT IFNULL(1, 0);     -- 1
SELECT IFNULL(0, 10);     -- 0
SELECT IFNULL(1/0, 10);     -- 10
SELECT IFNULL(1/0, yes);   -- yes
```

返回最新插入的 ID

```java
@Insert("INSERT INTO order_info "
		+ "VALUES (null, #{userId}, #{goodsId}, null, #{goodsName}, #{goodsCount}, #{goodsPrice}, #{orderChannel}, #{status}, #{createDate}, null)")
@SelectKey(keyColumn = "id", keyProperty = "id", resultType = long.class, before = false, statement = "select last_insert_id()")
long insertOrderInfo(OrderInfo orderInfo);
```





文本处理函数

① TRIME | LTRIM() | RTRIM():  

② LOWER() | UPPER()

③ LENGTH() | LEFT() | RIGHT()： 





**自定义函数**

实现自动控制



## 存储过程

存储过程可以看成是对一系列 SQL 操作的批处理。

使用存储过程的好处：

- 代码封装，保证了一定的安全性；
- 代码复用；
- 由于是预先编译，因此具有很高的性能。



注意事项

（1） 分号

命令行中创建存储过程需要自定义分隔符，因为命令行是以 ; 为结束符，而存储过程中也包含了分号，因此会错误把这部分分号当成是结束符，造成语法错误。

（2） 参数种类

包含 in、out 和 inout 三种参数。

给变量赋值都需要用 select into 语句。

每次只能给一个变量赋值，不支持集合的操作。

```sql
DELIMITER
CREATE PROCEDURE myprocedure(OUT ret IN) 
	BEGIN
		DECLARE y INT;
		SELECT sum(col1)
		FROM mytable
		INTO y;
```



存储过程是一组予编译的SQL语句，

 它的优点有：

​          允许模块化程序设计，就是说只需要创建一次过程，以后在程序中就可以调用该过程任意次。

​          允许更快执行，如果某操作需要执行大量SQL语句或重复执行，存储过程比SQL语句执行的要快。

​          减少网络流量，例如一个需要数百行的SQL代码的操作有一条执行语句完成，不需要在网络中发送数百行代码。

​          更好的安全机制，对于没有权限执行存储过程的用户，也可授权他们执行存储过程。





 一些使用

```sql
DELIMITER
CREATE PROCEDURE idata() 
BEGIN
	declare i int;
	set i = 0;
	while i<10000 do
	 INSERT INTO words(word) values(CONCAT(97+(i idv 1000)), char(97+(i % 1000 div 100)), char(97+(i%1000 div 100)));
	 SET i=i+1;
	END WHILE;
END;;
DELIMITER ;
CALL idata();
```



**触发器**





**游标**

（1）

在存储过程中使用游标可以对一个结果集进行移动遍历。

游标主要用于交互式应用，其中用户需要对数据集中的任意行进行浏览和修改。





（2） 使用游标的四个步骤：

1. 声明游标，这个过程没有实际检索出数据；
2. 打开游标；
3. 取出数据；
4. 关闭游标；





## 视图

不能够对其进行索引操作 

视图是虚拟的表，本身不包含数据，也就不能对其进行索引操作。

对视图的操作和对普通表的操作一样。



视图的好处

- 简化复杂的 SQL 操作，比如复杂的连接；
- 只使用实际表的一部分数据；
- 通过只给用户访问视图的权限，保证数据的安全性；
- 更改数据格式和表示。

```sql
CREATE VIEW myview AS
SELECT Concat(col1, col2) AS concat_col, col3*col4 AS compute_col
FROM mytable
WHERE col5=val;
```





## *窗口函数

> MySQL8.0+ 支持，Hive、Oracle 默认支持。

大多数商业数据库都支持，部分开源数据库支持的


主要用于统计分析使用

分类：
按照行进行统计
按照范围进行统计
按照百分比
按照排名


语法：
`Window w AS (partition by <col> order by <col2>)`: 按照什么进行分组, 并按照什么进行排序


基于行：
CURRENT ROW 边界是当前行，一般和其他范围关键字一起使用

UNBOUNDED PRECEDING 边界是分区中的第一行

UNBOUNDED FOLLOWING 边界是分区中的最后一行

expr PRECEDING 边界是当前行减去 expr 的值

expr FOLLOWING 边界是当前行加上 expr 的值



基于范围：
CUME_DIST()

DENSE_RANK()

LAG()

LEAD()

NTILE()

PERCENT_RANK()

RANK()

ROW_NUMBER()



\## 序号函数
row_number() / rank() / dense_rank()。



\## 分布函数
分布函数——percent_rank()/cume_dist()。
percent_rank()

用途：和之前的 RANK() 函数相关，每行按照如下公式进行计算：

(rank - 1) / (rows - 1)

其中，rank 为 RANK() 函数产生的序号，rows 为当前窗口的记录总行数。

应用场景：没想出来…… 感觉不太常用，看个例子吧↓
cume_dist()

用途：分组内小于等于当前 rank 值的行数 / 分组内总行数，这个函数比 percen_rank 使用场景更多。

应用场景：大于等于当前订单金额的订单比例有多少。


\## 前后函数
前后函数——lead(n)/lag(n)。

\*  用途：分区中位于当前行前 n 行（lead）/ 后 n 行 (lag) 的记录值。

\*  使用场景：查询上一个订单距离当前订单的时间间隔。


\## 头尾函数
头尾函数——first_val(expr)/last_val(expr)。
用途：得到分区中的第一个 / 最后一个指定参数的值。

使用场景：查询截止到当前订单，按照日期排序第一个订单和最后一个订单的订单金额。

\## 其他函数
其他函数

其他函数——nth_value(expr,n)/nfile(n）。

nth_value(expr,n)

用途：返回窗口中第 N 个 expr 的值，expr 可以是表达式，也可以是列名。

应用场景：每个用户订单中显示本用户金额排名第二和第三的订单金额。
nfile(n)

用途：将分区中的有序数据分为 n 个桶，记录桶号。

应用场景：将每个用户的订单按照订单金额分成 3 组。

聚合函数作为窗口函数

用途：在窗口中每条记录动态应用聚合函数 (sum/avg/max/min/count)，可以动态计算在指定的窗口内的各种聚合函数值。

应用场景：每个用户按照订单 id，截止到当前的累计订单金额 / 平均订单金额 / 最大订单金额 / 最小订单金额 / 订单数是多少？






# 其他

把IP地址存成 UNSIGNED INT
很多程序员都会创建一个 VARCHAR(15) 字段来存放字符串形式的IP而不是整形的IP。如果你用整形来存放，只需要4个字节，并且你可以有定长的字段。而且，这会为你带来查询上的优势，尤其是当你需要使用这样的WHERE条件：IP between ip1 and ip2



固定长度的表会更快
如果表中的所有字段都是“固定长度”的，整个表会被认为是 “static” 或 “fixed-length”。 例如，表中没有如下类型的字段： VARCHAR，TEXT，BLOB。只要你包括了其中一个这些字段，那么这个表就不是“固定长度静态表”了，这样，MySQL 引擎会用另一种方法来处理。



当只要一行数据时使用 LIMIT 1
当你查询表的有些时候，你已经知道结果只会有一条结果，但因为你可能需要去fetch游标，或是你也许会去检查返回的记录数。

在这种情况下，加上 LIMIT 1 可以增加性能。这样一样，MySQL数据库引擎会在找到一条数据后停止搜索，而不是继续往后查少下一条符合记录的数据。







线上配置



**权限管理**

查看特定用户权限

show grants for zx_root;

 

授予

grant all privileges on YQ.* to wise;

 

修改

rename user feng to newuser；//mysql 5之后可以使用，之前需要使用update 更新user表

 

删除

dropuser newuser;

//mysql5之前删除用户时必须先使用revoke 删除用户权限，然后删除用户，mysql5之后drop 命令可以删除用户的同时删除用户的相关权限

更改密码

set password for zx_root = password('xxxxxx');

update mysql.user set password=password('xxxx') where user='otheruser'

 

刷新特权， 在下次急需使用时

flush privileges;

 

如何解决root用户的Mysql只可以本地连接， 对外拒绝连接的情况？  处理线上环境的配置？

建立一个用户为monitor密码admin权限为和root一样。

允许任意主机连接。这样你可以方便进行在本地远程操作数据库了。

CREATE USER 'monitor'@'%' IDENTIFIED BY 'admin';

GRANT ALL PRIVILEGES **ON \*.\*** TO 'monitor'@'%' IDENTIFIED BY 'admin'**WITH GRANT OPTION** MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0;

 

DROP USER 'monitor'@'%';

DROP DATABASE IF EXISTS `monitor` ;





配置用户刚安装好的 MySQL 只有一个密码为空的root用户，首先需要给该用户设置一个密码：

mysqladmin -u root password 你的密码
然后使用设置好的密码登录 MySQl：

mysql -u root -p
 输入刚才设置的密码
由于root用户只能用于本地登录，无法远程登录，所以还需要创建一个用于远程登录的用户：

```
GRANT ALL ON *.* TO 你的用户名 @'%' IDENTIFIED BY '你的密码' WITH GRANT OPTION;
```

这条语句的意思是创建一个用户，赋予它以任何方式访问，在任何数据库的所有操作权限，但是实际这里的任何访问方式其实不包括localhost，所以还需要单独设定：

```
GRANT ALL ON *.* TO 你的用户名 @localhost IDENTIFIED BY '你的密码' WITH GRANT OPTION;
```

与上一条语句一样，只是将任何访问方式改为了以localhost访问。



**线上环境配置**

1）基本搭建

```shell
# 机器上是否安装验证
sudo rpm –qa | grep mysql-server
# 安装
sudo yum –y install mysql-server       //-y   yes
```

*优化对应的配置*， /etc/my.cnf

|-更改server端的编码

​       |-默认创建编码

//[mysqld]

character-set-server=utf8

default-character-set=utf8

 

配置自启动

```java
chkconfig mysqld on            //sudo
sudo chkconfig –list mysqld      //查看状况
重启
sudo service mysqld restart
```



2）mysql配置

```sql
-- 查看当前用户  mysql.user中保存着这些信息
select user,host,password from mysql.user
-- 修改root**密码,password()内置函数
set password for root@localhost = password(‘newrootpassword’);
-- 刷新
flush privileges;
-- 插入新的用户, password通过内置函数
insert into mysql.user(host,user,password) values("localhost", "mmall", password("mmall"));
```

 

**数据库相关**

创建db

```sql
create database mmall_net default character set utf8 COLLATE utf8-general_ci;
-- 授予用户对于该db的权限, 在之前可能需要flush privileges; 才能立即使用
grant all privileges on mmall_net.* to mmall@localhost identified by 'mmallpassword';
flush privileges

source /developer/mmall.sql;
select * from mmall_user\G;            
```







[后端程序员必备：书写高质量SQL的30条建议](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247486461&idx=1&sn=60a22279196d084cc398936fe3b37772&chksm=cea24436f9d5cd20a4fa0e907590f3e700d7378b3f608d7b33bb52cfb96f503b7ccb65a1deed&token=1987003517&lang=zh_CN%23rd)

[MYSQL 8.0 窗口函数：用非常规思维简易实现 SQL 需求](https://dbaplus.cn/news-11-2258-1.html)

[MySQL高性能优化规范建议](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485117&idx=1&sn=92361755b7c3de488b415ec4c5f46d73&chksm=cea24976f9d5c060babe50c3747616cce63df5d50947903a262704988143c2eeb4069ae45420&token=79317275&lang=zh_CN%23rd)