# Shell

## 语法

**字符转义**

对与语句中的引号转义问题



**变量**

### **String 的处理**

> 类似 Java String 处理，删除、替换、查找、子串

```shell
${变量#匹配规则}:  删除  →  最短
${变量##匹配规则]: 删除  →  最长
${变量%匹配规则}:  删除  ←  最短
${变量%%匹配规则}: 删除  ←  最长
# 替换
${变量/旧字符串/新字符串}:   替换  第一个
${变量//旧字符串/新字符串}: 替换  全部

# 字符索引位置
expr index "$string" substr

# 字符长度
${#string}
expr length "$string" # 字符中存在空格，必须使用 ""
# 子串抽取
${string:position}
${string:position:length}
${string: -position}
${string:(position)}
expr substr $string $position $length
```



**命令替换**

> 通过特定取值获取到特定的值，表达式为 Linux 命令
>
> 常用为 XX_DIR=\`pwd\`、TIME=、COMMIT_ID=
>

```shell
# 命令替换
`command`
$(command)
$(()): 整数运算


# 获取系统所有用户并输出
#   -d: 分隔符
#   -f: 属性列表
cat /etc/passwd | cut -d ":" -f 1
num1=20;num2=30
((num++))

# 遍历
#!/bin/bash index=1
for user in `cat /etc/passwd | cut-d ":"-f 1`
do 
  echo "This is $index user:$user"
  index=$(($index +1))
done

# 日期格式截取
man date
echo "$(date +%Y)"
echo "$(($(date +%Y) + 1))"
echo "$(date +%j) day"
echo "$(($(date +%j)/7)) weeks"
echo "$(((365 - $(date +%j))/7) weeks"

# 统计个数， -l: 行的个数
ps -ef | grep nginx | grep -v grep
ps -ef | grep nginx | grep -v grep | wc -l
nginx_process_num = $(ps -ef | grep -v grep | wc -l)
if [ nginx_process_num -eq 0 ]; then
  systemctl start nginx
fi
```



### 数据类型

> 整型、数组，环境变量

declare、typeset

```shell
# -r: 只读
# -i: 整形
# -a: 数组
# -f: 查看所有的系统函数,  -F: 只查看函数名
# -x: 环境变量，可以在终端、脚本中使用
man declare
declare  
num1=10
num2=$num1 + 20
echo $num2
expr $num1 + 10
```

数值运算

> expr 针对整数的操作

```shell
expr $num1 operator $num2
$(($num1 operator $num2))

# 转移，大小比较使用 expr，不使用 $(())
expr $num1 \| $num2
expr $num1 \& $num2
expr $num1 \< $num2
expr $num1 \< $num2
expr $num1 \<= $num2
expr $num1 \> $num2
expr $num1 \>= $num2
expr $num1 = $num2     
expr $num1 != $num2

num3=`expr $num1 + $num2`
num3=`expr $num1 \* $num2`

# @PRATICE
# sum 1+2+...+n
# 判断整数
while true
do 
  read -p "a numnber" num
  expr $num + 1 &> /dev/null
  if [ $? -eq 0 ]; then
    if [`expr $num \> 0` -eq 1]
      echo "+Integer"
      for ((i=2;i<=$num;i++))
      do
        sum=`expr $sum + $i`
      done
      echo "1+2+..+$num: $sum"
      exit
    else
      echo "error, not +Integer"
    fi
  fi 
  echo "error, not Integer"
  continue
done
```

浮点数运算

> 较少使用，bs 为 bash 内建的运算器， scale 精确度默认为0

```shell
which bc
echo "23.3+23.1" | bc
echo "scale=4;25.3*2.5/3" | bc
# PARTICE
read -p "in num" num1
read -p "in num" num2
echo "$num1 / $num2 | bc"
num3=`echo ""`
```

数组操作

```shell
request_addr_arr=(
  "linux121"
  "linux122"
  "linux123"
)
for request_addr in "${request_addr_arr[@]}";
do
  echo "request_addr: ${request_addr}"
done
```



### 参数

`$#`: 输入的参数个数

```shell
# 获取文件名称
basename <file-name>
```

```shell
# 获取当前用户
whoami
```





函数

> 参数名通过 $1,..$n
>
> func_name $1 $2...
>
> 函数传递参数：
>
> 函数的返回值： 两种
>
> 变量的作用范围： local 声明局部变量
>
> 函数库： 为 .lab 后缀，无可执行权限，无需和脚本统计目录，第一行一般 echo

```shell
#!/bin/echo
myFunction() {
}
function myFunction
{}
# @PRATICE
# 监控某个进程，自动启动，脚本本身也有 nginx (#)
#   通过进程判断
#   通过curl 判断
#   通过网络控制  netstat -tnlp | grep :80
# 执行进程的子进程的 pid
this_pid=$$
status=`ps -ef | grep nginx | grep -v grep | grep -v this_pid &> /dev/null`
if [ $status -eq 0 ]; then
  echo "healthy"
  sleep 3
else 
  systemctl start nginx
  echo "bag con, start ..."
fi


nohup sh nginx_daemon.sh &
tail -f nohup.out

# @PRATICE
# 计算器
function calc 
{
  case $2 in
    +)
      echo "`expr $1 + $3`"
      ;;
    -)
      echo "`expr $1 - $3`"
      ;;
    \*)
      echo "`expr $1 \* $3`"
      ;;
    /) 
      echo "`expr $1 / $3`"
     ;;
   esac
}
calc $1 $2 $3


# 返回值
# return:   ∈[1-255], 通常仅返回 0 或 1; 0->SUCCESS; 1->FAIL
# echo:     ∈{字符串结果, 列表}


# @PRACTICE
this_pid=$$
function is_nginx_running
{
  ps -ef | grep nginx | grep -v grep | grep -v this_pid &> /dev/null
  if [ $? -eq 0 ]; then
  	return
  else 
    return 1
  fi
}
is_nginx_running && echo "Nginx is running" || "Nginx is stoped"
# 查看执行过程
sh -x nginx.sh

# @PRACTICE
# 获取所有用户
function get_users
{
  users=`cat /etc/passwd | cut -d ":" -f1`
  echo $users
}
user_list=`get_users`

index=1
for u in $user_list
do 
  echo "NO: $index, User: $u"
  index=$(($index++))
done

# @PRACTICE: 函数库
#   算数操作
#   系统的内存，磁盘使用情况
function add 
{
  echo "`expr $1 + $2`"
}
function sub
{
  echo "expr $1 - $2"
}
function mul
{
  echo "expr $1 \* $2"
}
function div
{
  echo "expr $1 / $2"
}
function sys_info
{
  echo "Memo Info"
  echo
  free -m
  echo 
  
  echo "Disk Usage Info"
  echo
  df -h
  echo
}

. /develop/base_function
add 12 34
sub 33 21
```

```shell
# switch
case $choice
  2)
  	;;
  3)
  	;;
  q|Q)
    ;;
  *)
    ;;
# 读取数值到某个变量
read -p "xxxx" choice
# 循环
while true
do 
 ...
done

# 函数
function myMethod {

}
```



**自动输入**

expect 实现自动化输入，处理 fdisk、telnet、ftp 连接不适合脚本化的自动执行问题，默认情况下没有安装

```
yum install expect
```



**find**(#)

> 命令格式: find <path> <option> <operation>
>
> -name:  按照文件或目录进行搜索，支持通配符，如 find / -name "*.conf"
>
> -iname: 忽略大小写进行文件或目录的搜索
>
> -type: 文件类型查找，f 文件，d 目录
>
> -size -n +n： 按照文件所占磁盘空间进行查找，大于或小于某个特定值
>
> -mtime -n|+n: 根据文件更改时间查找，n 表示天数 (#)
>
> -mmin -n|+n： 根据文件修改时间查找，n表示分钟数
>
> -mindepth n： 控制搜索的目录深度
>
> -prune
>
> -user，-group
>
> -perm: 根据权限进行查找
>
> -exec：  -exec <commd> {} \       使用 `{}` 表示之前查找结果，对查找到的目录或文件进行执行特定的命令，如删除
>
> -ok: 作用同 exec ，给出选项确定是否执行

-nogroup

-nouser

-newer file1 ! file2

-maxdepth n

逻辑运算： -a： 与(默认)， -o： 或, -not|!： 非

```shell
# -name: 模糊查询
# -iname: 忽略大小写
find /etc -name '*.conf'
find ./ -iname 'aa'
# -type:   f: 文件，  d： 目录，l 链接文件， p: 管道文件
find . -type f
find . -type d
# -size: 大小，   +n: 大于n， -n:
find /etc -size +1M
find /etc-size 100k
# 块的大小	
dd if=/dev/zero of=123 bs=512k count=2
# -mtime:  -n: 小于n天修改, +n:      (#)
# -mmin:   -n: 小于n分钟修改
find /etc -mtime -3
find /etc -mtime -5 -name '*.conf'
find /develop/ -mmin -3 -type f -name '*.md'
find . -mindepth 2 -type f
# user
find . -nouser
find . -nogroup
# -perm: 权限
# 查找777
find . -perm 644
find -perm 777
# -prune: 排除，  日志、配置文件
# 与 -path <path> -prune 配合使用， 使用较少，优化使用?
find . -path ./test -prune -o -type f
find . -path ./test_1 -prune -o -path ./test_2 -prune -o -type f
# -newer: 比某个文件较新的文件
find . -newer 123

# ----------------------
# 操作
# -print: 
# -exec：  (#)
#   删除更改时间在7天以上的文件 (#)
find ./etc -name '*.conf' -exec rm -rf {} \;
find ./etc -size +1M -exec cp {} ./test \;
find /var/log/ -name '*.log' -mtime +7 exec rm -rf {} \;
```



**其他查找命令**

> locate: 在数据库文件中查找，定时任务更新该数据库， 默认部分匹配。  (无?)  单一，速度快，没有时安装 mlocate
>
>  updatedb： 
>
> ​      用户更新  /var/lib/mlocate/mlocate.db，
>
> ​      配置 /etc/mlocate.conf，在后台 crom 计划任务定期执行
>
> whereis
>
>  -b: 二进制文件
>
>  -m: 帮助文档
>
>  -s： 源码文件
>
> which： 仅查找二进制文件                用于查找程序的绝对路径

```shell
locate my.conf
updatedb
whereis mysql
which mysql
```





## sed

> 流编辑器，对标准输出或文件逐行处理
>
> 命令格式： sed [option] ,..

方式一： stdout | sed [option] "pattern command"

方式二： sed [option] "pattern command"

选项：

-n: 只打印模式匹配行，静默模式，非两行

-e: 在 sed 中编辑

-f: 指定文件获取匹配模式，   '/<pattern>/<command>'

-r: 支持扩展正则表达式

-i: 直接修改文件内容，生产中常用，配置文件，管理上千台服务器配置更改 (#)



'p'： 打印命令

's': 命令，  格式    `s/<old_str>/<new_str>/g`



匹配模式：  '/<pattern>/'

```shell
# 多个 -e 持续的作为选项 
# -f 从文件中获取匹配模式
# -r 扩展正则，使用 |
sed 'p' sed.txt
sed -n 'p' sed.txt
sed -n -e '/python/p' -e '/PYTHONE/p' sed.txt
sed -n -f edit.sed Test.txt 
sed -n -r '/python|PYTHON/p' Test.txt 
# 
# 对源文件进行修改  (#)
sed -n 's/love/like/g' sed.txt 
sed -n 's/love/like/g;p' sed.txt 
sed -i 's/love/like/g;p' sed.txt 
```

 

**pattern(7)**

pattern 用法表(7 种)

// todo 添加用法表

按行号进行匹配，少用

`/<pattern1>/<command>`： 使用最多 (#)

`/<pattern1/,/<pattern2>/<command>`： 次使用最多

```shell
# 按行号打印
sed -n '17p' /etc/passwd
sed -n '3,20p' /etc/passwd
sed -n '3,+17p' /etc/passwd 
# 字符匹配
# 字符转移匹配
# 正则匹配(#)
sed -n '/bash/p' /etc/passwd
sed -n '/\/sbin\/nologin/p' /etc/passwd
sed -n '/^root/p' /etc/passwd
# 匹配的范围(#)
sed -n '/^root/,/^yarn/p' /etc/passwd
# 从某行开始直到匹配到特定模式的行， 无法匹配则全部 :+1:
sed -n '4,/^root/p' /etc/passwd
sed -n '/sync,89p' /etc/passwd
sed -n '/sync/,3p' /etc/passwd
```



**编辑命令**

编辑命令对照表：

// todo md 表格合并式





**基础操作**

查找：

删除：

增加：

a: 行后追加

i: 行前增加

r:  将制定文件的内容追加到匹配行的后面

w： 将匹配到的行提取保存到另外的文件中

查找

```shell

```

**删除**

```shell
# 删除第一行，不会对原文件进行操作
# 删除并删除原文件
# 删除特定模式的字符
# 删除特定模式区间的字符
sed '1d' passwd
sed -i '1d' passwd
sed -i '1,5d' passwd
sed -i '/\/sbin\/nologin/d' passwd
sed -i '/^mail/,/^ftp/d' passwd
cat -n passwd
```

**增加**

> 

```shell
# 对满足特定模式的字符添加字符
sed -i '/\/bin\/bash/a # 这个用户可以登录系统' passwd
sed -i '/^bin/,/^rpc/i # 行前增肌的内容' passwd
# 将文件中的内容添加到匹配的行后
sed -i '/rpc/r ./list' passwd
# 将系统该用户保存到特定的文件中
sed -n 
sed -n '/\/bin\/bash/w /develop/test.txt' passwd

```

**修改**

> s/old_str/new_str/g    .....

```shell
# 匹配到行中的全部(g) 替换为特定的行 (#)
# 只替换行中匹配到的第一个
# 2g 从第二个开始的
# 2  只是第二个匹配到的替换
# ig 不区分大小写的匹配
sed -i 's/\/bin\/bash/\/BIN\/BASH/g' passwd
sed -i 's/root/ROOT/' passwd
sed -i 's/HADOOP/hadoop/2g'
sed -i 's/HADOOP/hadoop/2'
sed -i 's/hadoop/ig'

# = 显示行号
sed -n '/\/sbin\/nologin/=' passwd
```



**反向引用**

> 与正则表达式分组匹配类似，使用 `&`， `\<num>`

```shell
# 反向引用，通过 & 引用
# 通过 \1 
# 部分替换控制
sed -i 's/had..p/&s/g' str.txt
sed -i 's/\(had..ps\)/\1O/g'
sed -i 's/\(had\)..../\1doop/g'
```



**匹配变量**

> 通过 sh 定义的变量进行操作，使用双引号

```shell
#!/bin/bash
old str=hadoop
new str=HADOOP 

# 使用双引号进行变量控制
sed-i "s/$old_str/$new_str/g" str. txt
sed -i 's/'$old_str'/'$new_str'/g' str.txt
```



**ACTION**

```shell
sed -n '/^hdfs/p' passwd
sed -n '/^root/,/^hdfs/p' passwd
......
```

**查找 my.conf 中的段，以及每个段的配置总行数**

查找大的范围

通过 grep 进一步筛选

通过 for 循环遍历

```shell
# @PRACTICE
# 查找 my.conf 中的段，以及每个段的配置总行数
# 期望结果
#   1: client 2
#   2: server 12
#!/bin/bash

FILE_NAME=/xx/shell/my.conf

# @return 段的列表
function get_all_segment
{
  # 返回一个列表
  # 会保留 [] 辅助的查询，    类似正则表达式中 ?= 进行忽略辅助查询字符
  # Note: 命令替换 :+1:
  echo "`sed -n '/\[.*\]/p' $FILE_NAME | sed -e 's/\[//g' | sed -e 's/\]//g'``"
}
# 辅助测试，需要替换 ``
for seg in `get_all_segments`
do 
  echo "Config: $seg"
done


# 排除注释行、空行找出段的总行数
# 
# $1: 段的名称
# @return: 段对应的行数
function count_items_in_segment
{
  # 排除 {#开头，空行^$，[.*]}
  items=`sed -n '/$1/,/\/[.*\]/p' $FILE_NAME | grep -v "^#" | grep -v "^$" | grep -v "\[.*\]"`
  index=0
  for item in $items
  do
    index=`expr $index + 1`
  done
  echo $index
}
sum=`count_items_in_segment server`
echo sum


num=0
for segment in `get_all_segments`
do
  items_count=`count_items_segment $segment`
  echo "$num: $segment  $items_count"
done
```





删除命令对照表

```shell
sed -i '15d' passwd
sed -i '8,13d' passwd
sed -i 
```



// todo 



## awk

> 文本处理工具，"报表生成器"，对输出结果进行精细控制
>
> 方式一： awk 'BEGIN{}pattern{commands}END{}' file_name
>
> 方式二：stdout| awk 'BEGIN{}pattern{commands}END{}' file_name

**内置变量**

内置变量对照表

// todo 添加



$0: 整行数据，类似 正则分组匹配

$1...$n： 分组的数据



NF(Number Field): 字段的个数，使用Tab和空格分隔    (#)

NR: 处理的行号

FNR：File Number Row，每个文件单独技术



FS: 字段分隔符，指定非默认的 Tab 键和空格分隔

RS: 行分隔符，不是默认的 \n 分隔



OFS: 输出字段分隔符

ORS： 输出行分隔符





ARGC: 参数个数(少用)

ARGV: 数组(少用)



使用 FS, RS 较少，一般使用 printf 控制输出



命令： 

print: 类似 echo



默认以 空格和 Tab键分隔，混合任然当做一个变量处理

内部变量无需使用 $ 进行引用

```shell
awk 'print $0' /etc/passwd
awk 'BEGIN{FS=":"}{print $1}' /etc/passwd
awk '{print $1}' list
# 使用 NR, NF
awk '{print NF}' list /etc/passwd
awk '{print NR}' list /etc/passwd
awk 'print FNR' list /etc/passwd
# 限定分隔符，多次分隔符控制
awk 'BEGIN{FS="|"}{print $2}' list
awk 'BEGIN{FS=":"}{print $2}' list
awk 'BEGIN{;}{}' list
# FS: 字段分隔符，指定非默认的 Tab 键和空格分隔
# RS: 行分隔符，不是默认的 \n 分隔
awk 'BEGIN{RS="--"}{print $0}' list
awk 'BEGIN{RS="--";FS="|"]{print $3}}' list
# OFS: 输出字段分隔符
# ORS： 输出行分隔符
# 控制输出，默认使用,分隔才起作用
#   没指定，默认使用空格分隔
awk 'BEGIN{RS="--";FS="|";ORS="&"}{print $3}' list
awk 'BEGIN{RS="--";FS="|";ORS="&";OFS=""}{print $1,$3}' list
awk 'BEGIN{RS="--";FS="|";ORS="&";OFS=""}{print $1 $3}' list
awk 'BEGIN{RS="--";FS="|";ORS="&";OFS=":"}{print $1,$3}' list


# 输出每行中的最大字段
awk 'BEGIN{FS=":"}{print $NF}' /etc/passwd
Hadoop I Spark |Flume--JavalPython|ScalalGo--Allen|Mike|Meqgie
```



**格式化输出**

>%s, %d, %f, -，+，同 python、C、Java 格式化输出

```shell
awk 'BEGIN{FS=":"}{printf "%s",$1}' /etc/passwd
awk 'BEGIN{FS=":"}{printf "%s\n",$1}' /etc/passwd
awk 'BEGIN{FS=":"}{printf "%s %s\n",$1,$7}' /etc/passwd
# 空格补齐
awk 'BEGIN{FS=":"}{printf "%+20s %+20s\n",$1,$7}' /etc/passwd
awk 'BEGIN{FS=":"}{printf "%-20s %-20s\n",$1,$7}' /etc/passwd
```

```shell
# @PRACTICE
awk 'BEGIN{FS=":"}{printf "%s\n\n",$7}' /etc/passwd
awk 'BEGIN{FS=":"}{printf "%-20\n",$3}' /etc/passwd
awk 'BEGIN{FS=":"}{print "%0.2f\n",$3}' /etc/passwd
```



**实际使用**

```shell
# 获取容器的 Pid
 docker top mysql | awk '{print $2}'
# 获取程序的 pid
ps -ef | \
  grep -v grep | \
  grep 'java -jar -Dspring.profiles.active=test ips-bill-server.jar'| \
  awk '{ print $2 }'`
```



[linux-cmmand](https://github.com/jaywcjlove/linux-command)： GitHub 开源项目，Linux命令大全搜索工具，内容包含Linux命令手册、详解、学习、搜集。

[TLCL](https://github.com/billie66/TLCL/blob/gh-pages/README.md)： The Linux Command Line 一书的中文翻译

[shell获取当前系统时间](https://blog.csdn.net/sjin_1314/article/details/8561734)



# 运维

## 操作DB

> 对数据库进行备份....

```shell
# -e: 执行 SQL 语句
# -D: 连接的数据库
# ----
# 输出： 配合 Excel 制表
#   -B: 使用 Tab 替换分隔符
#   -N: 不输出列信息
#   -E: 垂直输出，展示格式
#   -H: 以 HTML 输出
#   -X: 以 XML 输出
# 连接 DB 执行 SQL，让结果按照特定格式显示，方便 awk, sed 处理
mysql -udbuser -p123456 -h<ip> -D school -e "SELECT * FROM student;"
mysql -udbuser -p123456 -h<ip> -D school -N -B -e "SELECT * FROM student;"
mysql -udbuser -p123456 -h<ip> -D school -N -H -B -e "SELECT * FROM student;" > result.html
```

```shell
# @PRACTICE
# 对某个数据库执行特定的 SQL 语句
# operate_mysql.sh
user="dbuser"
password="123456"
host="192.168.1.1"
db_name="$1"
SQL="$2"

mysql -u"$user" -p"$password" -h"$host" -D "$db_name" -B -e "$SQL"
# sh operate_mysql.sh school "SELECT * FROM score" > result.txt


# @PRACTICE
# 将文本中的数据插入到 MySQL 中
# 对文本中的数据进行筛选，将其插入到 MySQL 中
# import_mysql.sh
user="dbuser"
password="123456"
host="192.168.1.1"

mysql_conn="mysql -u"$user" -p"$password" -h"$host""
  # 自动分隔赋值
cat data.txt | while read id name birth sex
do
  $mysql_conn -e "INSERT INTO school.student VALUES(''$id', '$name', '$birth', '$sex')"
done
```



## 数据库备份

mysqldump常用参数详解：
-u用户名
-p密码
-h服务器IP地址
-d等价于--no-data只导出表结构
-t等价于--no-create-info只导出数据，不导出建表语句
-A等价于--all-databases
-B等价于--databases 导出一个或多个数据库r，  (#)





备份某个数据库中的某张表



将备份的数据通过 ftp 传输到某个远程主机



## 常见写法

获取某个进程的 PID

```shell
cat redis.conf | grep -v "#" | grep -v "^"
# 获取 redis 进程 ID
redis-cli -h 172.17.10.150 -p 7000 info server | grep process_id
kill -9 
# 获取某个 Java 程序的 PID
#
# @$1: 进程名称
# @return: 某个程序的 PID
function get_java_pid {
  process_name=$1
  pid=ps aux|grep java  | grep $process_name |awk '{print $2}'
  echo $pid
}

function replace_redis_conf {
  ori_port=$1
  new_port=$2
  master_ip=$3
  master_port=$4
  sed "s/$ori_port/$new_port/g" redis.cnf > redis-$new_port.cnf
  echo "slaveof $master_ip $master_port" >> redis-$new_port.conf
}
```



### Vim 编辑

```
# 临时显示行号
：set number
```

```
vim ~/.vimrc
```

**基础命令**

> 作为文本编辑器一般都有的

- yy： 复制

- p： 粘贴

- dd： 删除一行

- u： 撤销

- gg|G：移动到首行和尾行

- j|k：向下向上移动



区块选择：

v：

V：

[Ctrl] + v：

y：反白的复制

d：反白的删除

p：反白的粘贴



**额外功能**

**多文件编辑**

:n： 编辑下一个

:N： 编辑上一个

:files： 列出可选的

```shell
# 实现多个文件的编辑，可以保证复制内容在各个文件之间共享
vim hosts /etc/hosts
```



**多窗口编辑**

[ctrl] + w + j：

[ctrl] + w + k：

```
# 分为上下三屏
:sp /etc/hosts
:sp /etc/passwd
:sp /etc/hosts
```

```shell
# 从当前行到最后一行将 HADOOP 替换为 hadoop
.,$s/HADOOP/hadoop

# 当前行到最后一行删除
.,$d
```



**配置**

> 配置 vim
>
> ~/.vimrc：   默认不存在，设置每次通过 vim 打开的设置，使用 " 作为注释
>
> ~/.viminfo： vim 会主动的将你曾经做过的行为登录下来，该文件便存放这些记录

```
vim ~/.vimrc
set hlsearch "高亮度反白 
set backspace=2 "可随时用退格键删除 
set autoindent "自动缩排 
set ruler "可显示最后一列的状态 
set showmode "左下角那一列的状态 
set nu "显示行号
set bg=dark "显示不同的底色色调 
syntax on "进行语法检验，颜色显示。
```
