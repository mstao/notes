---
title: Linux 常用命令
date: 2020-11-10 22:27:31
tags:
- Linux
categories: 
- 运维
---



[TOC]





## 概述

### 命令设计

Postel 原则： 宽进严出
尽可能自由宽松接受输入格式，并输出格式良好的严谨输出格式。
宽进减少了过滤器在面对非预期输入时出错的可能性，以及特定情况下崩溃的可能性。
严出提高过滤器被其他程序用作输入的可能性。



命令的风格
- POSIX
- GNU
- MS-DOS   

```shell
# POSIX 集中短参数
tar -c -f xx.tar xx.txt yy.txt
tar -cfxx.tar xx.txt yy.txt
```



命令的参数:

- 可用选项  
- 选项参数: key=val  
- 子命令: 分割子命令显示不同的信息
- 密码参数: 防止根据 `history` 获取出用户密码
- 位置参数: 多个值按序解析 
- 支持多种类型的参数: 如发布的日志，传入文件名称自动解析文件内容作为参数，传入字符串  
- 支持自定义参数的解析:  可提供自定义的转换器，将参数映射成一个枚举项，指定的类对象
- 参数类型: `File`, `Date`, `URL`, `Pattern`..  

```shell
# Show interval and count 
vmstat 1 10
```



### 常见参数
- --usage:  给定短的使用消息  

- `-verbose`: 显示 Debug 日志  

- `-v` / `-vvv`: 的个数越多显示的日志越详细, 如 ansible 执行时通过 -vvv 显示运行时候的详细信息

- `-V`/`--version`: 显示版本信息  

- -a:
  所有项，与 `--all` 同名的选项
  添加（append), 与 tar 中一样

- -b
  缓冲区(buffer) 大小/block 的大小， du、df、tar 中使用
  批处理（batch）禁用提示/设置通过属性接受文件的输入

- -c
  命令（带参数），如 sh、python
  检查（check) 不带参数，检查命令/配置是否正确

- -d
  调试（debug)，带或不带参数，设置调试信息级别
  删除(delete)
  目录（directory)

- -D
  定义(define) 带参数

- -e
  执行(execute) (带参数)
  编辑(edit)，如 crontab
  排除(exclude)

- -f
  文件(file)(带参数)
  强制(force), 如 git push -f xxx、ssh

- -h
  表头（header)
  帮助(help)

- -i
  初始化
  交互执行，如 `docker exec -it <container-name> bash`

- -I
  包含

- -k: 保留，禁止资源的常规删除

- -l:
  列表
  加载：
  登陆： ssh 等要求网络身份



常用的命令缩写:  
-h: 主机    
-P: 端口  
-u: 用户  
-p: 密码   
-t: 测试配置文件  
-c: 指定配置文件地址  



常用的子命令  
`start`:  
`stop`:  
`restart`:   
`reload`:  



### 命令工具

**fzf**

> 模糊搜索工具
>
> 配置 Ctrl + R 快捷键自动根据 history 中的数据搜索

<img src="http://img.janhen.com/image-20201129000402250.png" alt="image-20201129000402250" style="zoom: 33%;" />



**the_silver_searcher**

> [GitHub](https://github.com/ggreer/the_silver_searcher)

替代 grep 的 Linux 工具，类似于ack的代码搜索工具，但速度更快。

可通过 `.gitignore`、 `.hgignore` 、`.ignore` 忽略指定文件

```shell
 yum -y install the_silver_searcher
```



**navi**

> [Github](https://github.com/denisidoro/navi)
>
> 命令行和应用程序启动器的交互式备忘单工具。

基于 fzf 对命令进行搜索, 可按照特定的标签进行组织命令，支持通过 `<name>` 指定参数

<img src="http://img.janhen.com/image-20201128235948928.png" alt="image-20201128235948928" style="zoom:33%;" />





**nohup**

> 允许用户退出帐户/关闭终端之后继续运行相应的进程

- /dev/null，代表linux的空设备文件，所有往这个文件里面写入的内容都会丢 失，俗称黑洞 
- 标准输入0，从键盘获得输入 /proc/self/fd/0 
- 标准输出1，输出到屏幕（控制台） /proc/self/fd/1 
- 错误输出2，输出到屏幕（控制台） /proc/self/fd/2
- `>/dev/null ` 标准输出1重定向到 /dev/null 中，此时标准输出不存在，没有任 何地方能够找到输出的内容 2>&1 错误输出将会和标准输出输出到同一个地方
- `>/dev/null 2>&1` 不会输出任何信息到控制台，也不会有任何信息输出到文件中

```shell
nohup flume-ng agent --conf /opt/apps/flume-1.9/conf \
  --conf-file /data/lagoudw/conf/flume-log2hdfs3.conf \
  -name a1 \
  -Dflume.root.logger=INFO,LOGFILE > /dev/null 2>&1 &
```



**lrzsz**

> 文件的上传和下载工具，只支持文件，不支持文件夹，配合 tar 命令使用，配合 SSH 连接使用

```shell
yum -y install lrzsz
# 选择本地文件上传到服务器
rz 
# 从服务器下载文件到本地
sz <file>
```



**ipvsadm**

> LVS 管理工具

```shell
# 查案负载情况
ipvsadm -l
```



**zsh**

> 辅助命令编写，带有提示

可进行 alias 指定, 主题选择，插件选择



**sort** 

用于排序。

- -f ：忽略大小写  
- -b ：忽略最前面的空格  
- -M ：以月份的名字来排序，例如 JAN，DEC  
- -n ：使用数字  
- -r ：反向排序  
- -u ：相当于 unique，重复的内容只出现一次  
- -t ：分隔符，默认为 tab  
- -k ：指定排序的区间  

```
du -sm * | sort -nr | head -n 10 
```



**uniq** 

可以将重复的数据只取一个。  

- -i ：忽略大小写
- -c ：进行计数
  

当前目录下的磁盘情况



**tee**  
输出重定向会将输出内容重定向到文件  

```
tee [-a] <file>
```



**paste**

字符转换

```
paste [-d] file1 file2
-d ：分隔符，默认为 tab
```



**rsync**

> 远程同步工具
>
> 速度快、避免复制相同内容和支持符号链接。
>
> rsync只对差异文件做更新。scp是把所有文件都复制过去。

- `-r`
- `-v`: 显示复制过程
- `-l`: 复制符号链接

```
yum install -y rsync
```

```shell
rsync -rvl /opt/module root@linux123:/opt/
```



## 基础命令

**cat**  

- `-n`: 显示文本的行号  
- `-T, --show-tabs`:  使用 `^I` 标示 Tab 字符      
- `-E, --show-ends `:  显示 `$` 作为每行的末尾  
Ctrl + D 终止输入    
```shell
cat > SampleTextFile.txt
```



**File**

显示文件字符集

```
file -i test_info.sh
```




**iconv:**  
文本字符集转换   

- -f, --from-code=NAME:  来源字符集编码  
- -t, --to-code=NAME： 目的字符集编码  
- -o, --output=FILE：  输出文件名  

```shell
iconv -f ISO-8859-1 -t UTF-8 in.txt > out.txt
```



### 信息查看

RedHat 类系统信息查看

```shell
cat /etc/os-release
```



**系统相关**

> 系统服务相关，包括查看状态、停止运行、重新加载、开机自启
>
> 主机域名映射，实现 hosts 
>
> 环境变量控制，实现增加环境变量

```shell
# 查看系统应用状态
# 停止系统应用
# 重新加载系统应用
# 将系统应用开机自启
# 查看失败的系统应用
systemctl status redis_6379
systemctl status docker
systemctl stop redis_6379
systemctl reload docker.service
systemctl enable docker.service
systemctl --failed
```



**系统基本信息**

```shell
# 查看系统总信息
# 查看系统的位数, 配合下载 URL 使用
# 查看系统内核版本
uname -a
uname -m
uname -r
```

**CPU 情况**

```shell
# 总核数 = 物理CPU个数 X 每颗物理CPU的核数 
# 总逻辑CPU数 = 物理CPU个数 X 每颗物理CPU的核数 X 超线程数
# 查看物理CPU个数
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l
# 查看每个物理CPU中core的个数(即核数)
cat /proc/cpuinfo| grep "cpu cores"| uniq
# 查看逻辑CPU的个数
cat /proc/cpuinfo| grep "processor"| wc -l
# 查看CPU信息（型号）
cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c
```

CPU 整体信息

```shell
lscpu
```

各个 CPU 的情况

```shell
mpstat -P ALL
```

连续的 CPU 情况

```shell
mpstat -P ALL 1 300
```


```shell
# 查看僵尸进程
ps -al | gawk '{print $2,$4}' | grep Z
```

查看进程实际使用的内存情况

```shell
ps -e -o 'pid,comm,args,pcpu,rsz,vsz,stime,user,uid' |  sort -nrk5
```

**运行负载情况**

```shell
top
top -p 3306
```



**内存使用情况**

```shell
free -m
```

```shell
# 查看内存使用百分比
free | sed -n '2p' | gawk 'x = int(( $3 / $2 ) * 100) {print x}' | sed 's/$/%/'
```

```shell
# 释放内存
echo 1 >/proc/sys/vm/drop_caches
echo 2 >/proc/sys/vm/drop_caches
echo 3 >/proc/sys/vm/drop_caches
```



**磁盘占用情况**

可以作为部署时候的参考

```shell
df -h
df -sh *
```



**文件句柄数**

```shell
ulimit -n
```



### 网络

查看端口使用

```shell
ss -nelp | grep 22
netstat -lntp | grep 22
```



查看IP 地址
```shell
# Show all interface ip addr
ifconfig | grep -Eo 'inet (addr:)?([0-9]*\.){3}[0-9]*' | grep -Eo '([0-9]*\.){3}[0-9]*' | grep -v '127.0.0.1'
```

```shell
# Specified interface
ifconfig eno1 | grep -Eo 'inet (addr:)?([0-9]*\.){3}[0-9]*' | grep -Eo '([0-9]*\.){3}[0-9]*'
ifconfig eth0 | grep -Eo 'inet (addr:)?([0-9]*\.){3}[0-9]*' | grep -Eo '([0-9]*\.){3}[0-9]*'
```

```shell
# 连接指定端口 
telnet IP PORT  
# dns 解析相关
nslookup sina.com    
# 从主机到互联网另一端的主机走的什么路径  
traceroute sina.com  

sar  
# 网络状况分析跟踪  
tcpdump
```

Tcp 端口监听情况

```shell
netstat -lntp | grep <port>
ps -ef | grep <port>
lsof -i tcp:<port>
```

Tcp 状态连接统计  
```shell
netstat -an | awk '/^tcp/ {++s[$NF]} END {for(a in s) print a,s[a]}' 
```

bridge-utils： yum install bridge-utils

```shell
# Linux 本地系统的转发支持，  0,1 语义..
# 设置开启转发
sysctl net.ipv4.ip_forward
sysctl -w net.ipv4.ip_forward=1
# iptables 关联命令
#   查看 NAT 的端口映射， 显示伪装地址，端口映射情况
iptables -t nat -nL
# 查看 bridge 情况
brctl show
# 网络运行情况
netstat -anop
```

```shell
# 查看本机网卡
nmcli d
```



**网卡配置**

打开文件**/etc/sysconfig/network**来修改主机名和DNS

```shell
vi /etc/sysconfig/network-scripts/ifcfg-ens160

ONBOOT=yes
BOOTPROTO=static
DNS1=172.17.0.211
DNS2=172.17.0.221
IPADDR=172.17.10.137    # 更改
NETMASK=255.255.240.0
GATEWAY=172.17.0.49

systemctl restart network
ping 8.8.8.8
ping 172.17.0.49
ping baidu.com
```



**域名主机名配置**

```shell
vi /etc/hostname
hostnamectl 
hostnamectl set-hostname cnode
echo "172.17.10.153 cnode1" >> /etc/hosts
```



**防火墙相关**

> 线上服务器必须使用，当前为 Centos7 自带防火墙，局域网一般可以关闭
>
> 开启、关闭防火墙，基本信息查看
>
> 开放或关闭某个|一个数据段的端口，
>
> 开放或关闭某个|多个服务

```shell
# 停止并关闭开机启动防火墙
systemctl stop firewalld
systemctl disable firewalld

# 查看防火墙版本号
# 查看防火墙状态
# 启动防火墙
# 列出所有zone
# 查看特定区域，不指定默认为 public
firewall-cmd --version
firewall-cmd --state
systemctl start firewalld.service
firewall-cmd --get-zones
firewall-cmd --get-default-zone
firewall-cmd --list-zone
firewall-cmd --list-all
firewall-cmd --zone=pulic
# 查看开放的端口
# 开启需要的端口
# 重新加载端口
# 查看开放端口验证
# 查看某个端口
firewall-cmd --list-port
firewall-cmd --permanent --zone=public --add-port=3306/tcp
firewall-cmd --permanent --zone=public --add-port=22/tcp
firewall-cmd --reload
firewall-cmd --list-port
firewall-cmd --query-port=22/tcp
firewall-cmd --add-port=22/tcp
# 查看开放的端口
# 删除某个|某段的端口
# 重新加载
# 验证开放端口情况
# 关闭防火墙
firewall-cmd --list-port
firewall-cmd --permanent --remove-port=2000-20002/tcp 
firewall-cmd --reload
firewall-cmd --list-port
firewall-cmd --zone=public --list-ports
# 开启|关闭某个服务
firewall-cmd --add-service ftp --permanent
firewall-cmd --remove-service --permanent
firewall-cmd --zone=public --list-services
```



**iptables 相关**

```shell
# 对于请求 8082 端口的全部丢弃掉
iptables -I INPUT -p tcp --dport 8082 -j DROP
```



**net-tools**

> 常用的网络工具

可以使用 Linux 旧有支持的命令，以及扩展的网络工具

route： 如 `route -n` 命令，等价于现在自带的 `ip -r` 命令

netstat： 常用的网络工具

ifconfig： 废弃命令，在 Linux 上使用 `ip address` 代替，简写成 `ip a`

```shell
# 添加默认网关模板以及对应的实例
# 验证设置网关的结果
route add default gw {IP-ADDRESS} {INTERFACE-NAME}
route add default gw 192.168.2.254 eth0
route -n
# Linux 自带命令
# 配置默认网关
# 验证执行情况
ip route add default via 192.168.1.254
ip r
```



### 磁盘


```shell
sudo find /var/lib/docker/containers -name *.log
sudo du -d1 -h /var/lib/docker/containers | sort -h
sudo du -d1 -h /var/lib/docker | sort -h
docker inspect --format='{{.LogPath}}' iwms-openapi
```

磁盘挂载

- `du -sh *`: 查看目录占用的空间情况
- `df -h`：查看磁盘占用情况
- `df -T`：查看所有磁盘的文件系统类型(type)
- `fdisk -l`：查看所有被系统识别的磁盘
- `mount -t type device dir`：挂载device到dir

```shell
fdisk /dev/sdb
# 格式化分区
mkfs -t ext4 /dev/vdb1


mkfs.xfs /dev/sdb1
```


查看指定目录下文件的大小，排序并只显示TOP10

```shell
du -sh * | sort -hr | head -n 10
```

- mkfs -t ext4 /dev/vdb1:  格式化分区

```shell
 umount /data   ##卸载data目录下分区
 # 分区的UUID
 ls -l  /dev/disk/by-uuid/
 # 开机自动挂载
 vim /etc/fstab 
   /dev/sdb    /data    ext4    defaults    0 0
```

```shell
# 查看硬盘以及分区情况
lsblk
```

```shell
# 比如要扩充 /var
# 在创建好文件系统后 新建临时挂载点 storage
mkdir /storage
# 将/dev/sdb1挂载到/storage下
mount /dev/sdb1  /storage
# 拷贝/var下的所有内容到新的硬盘
cp -pdr /var /storage
# 或在/var 目录下执行：find . -depth -print | cpio - pldvm /temp
# 删除当前/var目录下的内容
rm -rf /var/*
# 重新挂载硬盘到/var目录
umount /dev/sdb1
mount /dev/sdb1 /var
# 过程中若提示磁盘忙，使用fuser找出将正在使用磁盘的程序并结束掉；
fuser -m -v /var
```

```shell
partprobe /dev/sda
partx -a /dev/sdb
# 验证情况
```



### 软连接  

删除后，存放的数据不删除  

```shell
ln –snf  /var/www/test1   /var/test  
# 意不要在后面加 ”/”
rm –rf <软链接名称>
ln –snf  <新的源文件或目录> <目标文件或目录>
```



### 文件

> 重要的配置文件位置

**用户相关**

- /etc/passwd： 用户列表文件，是否是系统用户，是否可以登录
- /etc/group：    用户组列表文件

- /etc/sudoers: 可以有 sudo 权限的配置

- /etc/profile： 环境变量配置

- /etc/hosts：   host 文件位置

- /etc/hostnames: 主机名

- /etc/sysconfig/network-scripts/ifcfg-ensxx: 网卡设置

- /etc/yum.repos.d/xxx.repo： 软件安装源

- /var/logs： 日志存放位置



**执行路径**

/usr/local/bin、/usr/bin、/usr/local/sbin、/usr/sbin

/home/<user_name>/bin: 可执行路径，常通过 ln -s ... 的方式使全局可用



**文件操作**

基本操作， ls、touch、mkdir、cd、rm、cp、mv、pwd

```shell
# 复制、移动、重命名
cp -r /user/newTest /test 
mv /test/newTest /usr 
mv aaa bbb
rm -rf /user/newTst
# 文件的查看
cat -n /etc/passwd 
tail /etc/passwd
tail -3 /etc/passwd
tail -f nohup.out
# 文件压缩与解压
tar -cvf <zip_name> <folder_name>
# 复制并重命名
copy -a nginx_playbooks wordpress_playbooks
```



**文件上传下载**

> 文件下载： wget、curl
>
> 文件上传： scp

```shell
scp -r <local-path> <user>@<ip-or-host>:<remote-path>
```



**tar 命令**

- -z, --gzip, --gunzip, --ungzip   通过 gzip 过滤归档  
- -c, --create: 创建一个新归档
- -v, --verbose: 详细地列出处理的文件  
- -f, --file=ARCHIVE: 使用归档文件或 ARCHIVE 设备  
- -C, --directory=DIR: 改变至目录 DIR  
- -x, --extract, --get: 从归档中解出文件  
- -j, --bzip2 :  通过 bzip2 过滤归档  
- -t, --list: 列出归档内容

打包

```shell
tar -cvf /mydata/etc.tar /etc
```

  打包并压缩

```shell
tar -zcvf /mydata/etc.tar.gz /etc
```


用bzip2压缩文件夹/etc到文件/etc.tar.bz2：

```shell
tar -jcvf /mydata/etc.tar.bz2 /etc
```

解压缩到指定目录  

```shell
tar -zxvf etc.tar.gz -C /usr/local/
```


找出指定时间范围内的日志文件，打包传输  

```shell
find . -name "*04-29*" -exec cp {} 200428_200505 \;
find . -name "*04-28*" -exec cp {} 200428_200505 \;
find . -name "*04-30*" -exec cp {} 200428_200505 \;
find . -name "*04-31*" -exec cp {} 200428_200505 \;
find . -name "*05-01*" -exec cp {} 200428_200505 \;
find . -name "*05-02*" -exec cp {} 200428_200505 \;
find . -name "*05-03*" -exec cp {} 200428_200505 \;
find . -name "*05-04*" -exec cp {} 200428_200505 \;
find . -name "*05-05*" -exec cp {} 200428_200505 \;
tar -zcvf 200428_200505.tar.gz 200428_200505/
```

查看大文件的行数

```shell
sed -n '$=' orderinfo.txt
wc -l orderinfo.txt
```



## 常用工具
### find
- -maxdepth levels  
- -atime n File was last accessed n*24 hours ago.   
- *-mtime n File's  data  was  last  modified  n*24  hours ago.  
- -size n[cwbkMG]    
- -type c File is of type c  

```shell
# 找出指定目录下文件的个数  
find DIR_NAME -type f | wc -l
```

```shell
# 找出 /opt 目录下大于 800 M 的文件
find /opt -type f -size +800M -print0 | xargs -0 du -h | sort -nr  
# 找到文件并移到 opt 目录  
find / -name "*tower*" -exec rm {} \;
```

```shell
# 找出系统中占用容量最大的前 12 个目录  
du -hm --max-depth=2 | sort -nr | head -12
```

```shell
# 当前目录搜索lin开头的文件，然后用其搜索后的结果集，再执行ls -l的命令（这个命令可变，其他命令也可以），其中 -exec 和 {} \; 都是固定格式  
find . -name "lin*" -exec ls -l {} \;
```



### grep  

`-v`: 过滤掉

`-i`: 忽略大小写

`-n`: 打印行号

`-H, --with-filename`: 打印文件名

`-r`: 递归处理  

`-d, --directories=ACTION`: 目录处理策略, `read`,`recurse`,`skip`

`-c`/`--count`: 打印匹配的数量

`-w`: 匹配整个词

`-x`: 整行

查看指定进程并过滤掉 grep 自身

```shell
ps -ef | grep namenode | grep -v grep
```

正则匹配

```shell
grep "python|PYTHON" file
grep -E "python|PYTHON" file
grep -F "py.*" file
```

打印匹配行的前后5行 

```shell
grep -5 'parttern' INPUT_FILE 
```

```shell
grep Full gclogs/fanruan.gc.log.2020-07-26
grep -n Full gclogs/fanruan.gc.log.2020-07-26
```

搜索指定字符，并显示匹配到的行号

```shell
grep -n man /etc/man_db.conf
```

```
3:# This file is used by the man-db package to configure the man and cat paths.
4:# It is also used to provide a manpath for those without one by examining
5:# their PATH environment variable. For details see the manpath(5) man page.
```

查看单个/多个文件某字符出现的次数

```shell
grep Full gclogs/* | wc -l
# count
grep -c processor /proc/cpuinfo
# Show filename:count
grep -c Full gclogs/*
gclogs/fanruan.gc.log:1
gclogs/fanruan.gc.log.2020-07-17:434
gclogs/fanruan.gc.log.2020-07-18:1
gclogs/fanruan.gc.log.2020-07-19:3
...
```


查看某个配置文件，排除掉里面以 # 开头的注释内容：
```shell
grep '^[^#]' /etc/openvpn/server.conf
```
查看某个配置文件，排除掉里面以 # 开头和 ; 开头的注释内容：
```shell
grep '^[^#;]' /etc/openvpn/server.conf
```



### sed

>  轻量级流编辑器，一般用来处理文本类文件  
>  非交互式的编辑器, 它不会修改文件，除非使用 shell 重定向来保存结果。默认情况下，所有的输出行都被打印到屏幕上   
>  sed -i 会实际写入  

-   p 参数表示打印，一般配合 -n（安静模式）进行使用



- c. 替换
  
- s： 搜索并替换
  

```shell
# 显示网络接口的 Ip 地址  
ifconfig eth0 |grep 'inet' |sed 's/^.*inet//g' |sed 's/netmask.*$//g' |sed -n '1p'
192.168.199.183  
```

```shell
# 显示第 7 ~ 10 行内容
sed -n '7,10p' /opt/log4j2.properties
```

```shell
# 将文件中每一行以 # 开头的都替换掉空字符并展示
sed 's/^#*//g' /opt/log4j2.properties
```

```shell
# 将 1 ~ 4 行内容替换成 GitNavi.com
cat -n /opt/log4j2.properties |sed '1,4c GitNavi.com
```



### 日期和日历

- `date +%Y`:  显示当前年份
- `date +%m`: 显示当前⽉份
- `date +%d`: 显示当前是哪⼀天
- `date "+%Y-%m-%d %H:%M:%S"` : 按照指定格式显示

- `date -d '1 days ago'`: 显示前⼀天⽇期
- `date -d yesterday +"%Y-%m-%d"`: 同上
- `date -d next-day +"%Y-%m-%d"`: 显示明天⽇期
- `date -d 'next monday'`: 显示下周⼀时间

- `date -s 字符串时间`: 设置系统时间



cal
查看日历。
`cal -3`: 查看当前，上个，下个月的日历
`cal 2020`: 查看指令年份的






### 用户与权限

- open files: 可打开的文件限制  
- file size: 可创建单个文件的最大大小?  
- max memory size: 最大可用的内存大小  
- stack size:  最大可使用的栈大小  

```shell
ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 59459
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 59459
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

- `-`代表⽂件

* `d` 代表⽬录

* `c` 字符流，

* `s` socket

* `p` 管道

* `l` 链接⽂档(link ﬁle)

* `b` 设备⽂件

```shell
chmod [{ugoa}{+-=}{rwx}] [⽂件或⽬录] [mode=421 ] [⽂件或⽬录]
# 改变⽂件或者⽬录的所有者
chown [最终⽤户] [⽂件或⽬录] 
# 改变⽂件或者⽬录的所属组
chgrp [最终⽤户组] [⽂件或⽬录] 
```

```shell
useradd <user-name>
passwd <user-name>
# 切换⽤户，只能获得⽤户的执⾏权限，不能获得环境变量
su <user-name> 
# 切换到⽤户并获得该⽤户的环境变量及执⾏权限
su - <user-name>
# 删除⽤户但保存⽤户主⽬录
userdel <user-name>
# ⽤户和⽤户主⽬录，都删除
userdel -r ⽤户名 
```

```shell
# 让用户支持权限命令
visudo
%vagrant ALL=(ALL)   ALL
```

/etc/sudoers

Allow root to run any commands anywhere 

root ALL=(ALL) ALL 

hadoop ALL=(ALL) ALL

cat /etc/passwd 查看创建了哪些⽤户

```shell
usermod -g <⽤户组> <⽤户名>
groupadd <组名> <代表⽬录>
groupdel <组名>
groupmod -n <新组名> <⽼组名>
cat /etc/group
```



## Refs

- 《UNIX 编程艺术》

  