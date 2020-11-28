# Linux 环境

> 常用 Linux 环境的安装配置

```shell
yum -y update
# 安装开发工具包
yum groupinstall -y 'development tools'
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
```

```shell
# 查看系统版本
cat /etc/redhat-release
```

```shel
# 处理 No package python-pip available.
yum -y install epel-release
```



## Docker 

[安装脚本](./script-)

**Docker 环境安装**

```shell
# 通过官方推荐的脚本安装
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

```

**Docker Compose 安装**

> 安装 Docker Compose 
>
> 关联：[Install Docker Compose](https://docs.docker.com/compose/install/)

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
# 可选
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
# 验证
docker-compose --version
```



## Anisible 安装

基于 Python 环境

安装方式：

- 通过 Yum 包管理安装
- Git 源代码安装

**源码安装 + virtualenv 环境隔离**

```shell
useradd deploy && su - deploy
git clone https://github.com/ansible/ansible.git
source .py3-a2.5-env/bin/active
pip3 --default-timeout=500 install paramiko PyYAML jinja2
mv ansible .py3-a2.5-env/
cd .py3-a2.5-env/ansible/
git checkout stable-2.5
# 每次进入环境进行 ansible ...
source /home/deploy/.py3-a2.5-env/ansible/hacking/env-setup -q
ansible --version
```



**yum 安装**

[Centos7安装配置ansible运维自动化工具](https://www.centos.bz/2017/08/centos7-install-ansible/)

删除：

```shell
rpm -qa | grep ansible | xargs rpm -e
rpm -qa | grep -i epel | xargs
```

## Python 安装

```shell
yum -y groupinstall "Development tools"
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel

yum install gcc
yum install zlib zlib-devel -y
wgt ....... tar.xz      rz
xz -d Python-3.6.5.tar.xz
tar -xvf Python-3.6.5.tar
cd Python-3.6.5
mkdir -p /usr/local/python/python3.6
./configure --prefix=/usr/local/python/python3.6
make && make install




./configure --prefix=/usr/local --with-ensurepip=install --enable-shared LDFLAGS="-Wl,-rpath /usr/local/lib"
make && make altinstall
```

**pip 安装**

```shell
yum -y install epel-release
yum install -y yum-utils && yum-config-manager --enable epel
yum -y install python-pip
pip install --upgrade pip
```

```shell
pip list
```

**virtualenv 安装使用**

```shell
pip3 install virtualenv
# 处理国内下载超时问题
pip3 --default-timeout=200 install virtualenv
# 安装管理
pip --default-timeout=100 install virtualenvwrapper
useradd - deploy && su deploy
virtualenv -p /usr/local/python/python3.6/bin/python3.6 .py3-a2.5-env
# 退出虚拟环境
deactivate
```

[在一个centos6上安装多个不同版本python](https://blog.51cto.com/leesbing/1827517)

[CentOS 7.4 安装python3及虚拟环境](https://www.cnblogs.com/contiune/p/10575113.html)

[Centos7安装Python3与pip3[简教程]](https://www.jianshu.com/p/758b592387d1)

[Python 虚拟环境 virtualenv](https://www.cnblogs.com/zh605929205/p/7705192.html)

[linux centos7 安装python3.6 替换默认python2.7](https://www.cnblogs.com/fixdq/p/9879285.html)： 软连接，关联 python2.7 执行配置更改



## Java 环境

> 在 Linux 机器上安装 Java 环境，主要用于 CICD 中 Maven 的构建
>
> 关联： [JDK8-Download](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

```shell
# /usr/local/java/jdk8
yum update
rpm -qa | grep -E '^open[jre|jdk]|j[re|dk]'
yum remove <already_installed_jdk_version>
tar -xvf jdk-8u111-linux-x64.tar.gz
mkdir /usr/local/java
mv jdk1.8.0_221/ /usr/local/java/jdk8
sudo vim /etc/profile
# 在 export 开头的行下添加如下内容
export JAVA_HOME=/usr/local/java/jdk8
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

source /etc/profile
java -version
```



## Maven 环境

> 在 Linux 机器上安装 Maven 环境，主要用于 CICD 中对源码的构建打包
>
> 关联： [Maven-Download](https://maven.apache.org/download.cgi)

```shell
# /usr/local/maven/maven3.6
tar -zxvf apache-maven-3.6.1-bin.tar.gz 
mkdir /usr/local/maven
mv apache-maven-3.6.1 /usr/local/maven/maven3.6
sudo vi /etc/profile
export MAVEN_HOME=/usr/local/maven/maven3.6
export PATH=$PATH:$MAVEN_HOME/bin
source /etc/profile
mvn -version     //验证
```



Maven 使用

```shell
# 使用多核进行打包编译
mvn -T 4 clean install 
-Dmaven.compile.fork=true 
# 打包项目中指定模块，指定 db-setup，指定 server 的打包
# 脚本传参控制单条还是多条
mvn -U -pl ${MODULE} -am clean package
```



## Git 环境

> 在 Linux 机器上安装 Git 环境，主要用于 CICD 中拉取源码

```shell
yum -y install zlib-devel openssl-devel cpio expat-devel gettext-devel curl-devel perl-ExtUtils-CBuilder perl-ExtUtils-MakeMaker
yum install gcc
tar -zxvf git-2.8.3.tar.gz
cd git-2.8.3
./configure prefix=/usr/local/git/git2.8
make && make install
vim /etc/profile
  export GIT_HOME=/usr/local/git/git2.2
  export PATH=$PATH:$GIT_HOME/bin
source /etc/profile
git --version

# 配置
git config --global user.name "zhangjiang"
git config --global user.email "zhangjiang@hd123.com"
git config --global --list
```



## Mysql 环境

> 安装 MySql 数据库

通过软件源安装

通过运维工具安装，如通过宝塔连接上直接安装

```shell
wget 'https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm'
sudo rpm -Uvh mysql57-community-release-el7-11.noarch.rpm
yum repolist all | grep mysql
vim /etc/yum.repos.d/mysql-community.repo
# 更改 enbale,选择想要安装的版本
sudo yum install mysql-community-server
systemctl start mysqld
systemctl status mysqld
# 修改字符集
vim /etc/mysql.cnf
sudo systemctl restart mysqld
```

重要文件目录：

`/var/lib/mysql`： 默认存放数据目录

`/var/lib/mysql/mysql.sock`： 默认的 Socket

`/etc/my.cnf`： mysql 启动时的配置



 [Centos 7 安装 MySQL](https://www.jianshu.com/p/7cccdaa2d177)



## Node 环境

//todo node 的编译安装



## Nginx 环境 

### 安装配置

1、安装

2、验证

```shell
nginx -v
# 查看编译情况的参数
nginx -V
```

3、查看目录

```shell
# 查看 rpm 方式安装的软件位置
rpm -ql nginx
```

(1) 配置目录
/etc/logrotate.d/nginx: Nginx 日志伦准，logrotate 服务的日志切割
/etc/nginx/nginx.conf: 主配置文件
/etc/nginx/conf.d/default.conf:  自定义的配置目录下的默认加载

(2) 安装目录
/etc/nignx/fastcgi_params
/etc/nginx/uwsgi_params
/etc/nginx/scgi_params

(3) Http 协议
/etc/nginx/mime.types: 设置 http 协议的 Content-Type 与扩展名对应关系

(4) 守护进程管理器方式管理：
/usr/lib/systemd/system/nginx-debug.service
/usr/lib/systemd/system/nginx.service
/etc/sysconfig/nginx
/etc/sysconfig/nginx-debug

(5) 模块安装目录
/usr/lib64/nginx/modules
/etc/nginx/modules:

(6) 命令相关文件
/usr/sbin/nginx:
/usr/sbin/nginx-debug:

(7) 缓存服务目录
/var/cache/nginx: Nginx 的缓存目录

(8) 日志目录
/var/log/nginx/error.log:  错误日志位置
/var/log/nginx/access.log: Http 协议访问日志

4、安装编译参数
--prefix=/etc/nginx
--sbin-path=/usr/sbin/ngins
-modules-path=/usr/lib64/nginx/modules
--conf-path=/etc/nginx/nginx.conf
--error-log-path=/var/log/nginx/error.log
--http-log-path=/var/log/nginx/access.log
--pid-path=/var/run/nginx.pid
--lock-path=/var/run/nginx.lock

--user=nginx:  用户指定
--group=nginx

--with-cc-opt=parameters: 设置额外的参数...
--with-ld-opt=parameters: 附加参数，链接系统库，多种方式可实现..



## Redis 安装

> 安装软件常见位置 `/usr/local/redis`

前提： 安装好对应的环境

```shell
yum install -y epel-release gcc
```

```shell
# 4核 cpu 编译
# 将可执行文件添加到启动目录中


vim /etc/init.d/redis_6379
tar -zvxf redix-4.0…ta.gz
mv redix-4.0.2 /usr/local/redis        
make -j 4               
make install 
# 编辑配置文件
#   修改ip绑定     bind 0.0.0.0          #注释 bing 127.0.0.1
#   后台启动       daemonize yese
#   增加密码       requirepass <password>
#   修改日志文件    logfile  "/var/log/redis/6379.log"
#   修改数据文件目录 dir  "/opt/redis/data/"
#   修改文件名称     dbfilename "dump-${port}.rdb"
#   修改集群参数     cluster-enabled yes
#   节点参数       cluster-config-file nodes-$(port}.conf
vim redis.conf
# 新建日志、数据目录
mkdir -p /var/log/redis
mkdir -p /opt/redis/data
# 使用指定配置启动 
redis-server ./redis.conf    

# 验证
ps -ef | grep redis             
redis-cli -h 127.0.0.1 -p 6379       
exit            
```

作为系统服务并开放：

```shell
cd /<redis_install_path>/utils/install_server.sh
./install_server.sh
# 指定端口、配置文件、log、data directory、executable path
# 6379
# /usr/local/redis/redis.conf
# /usr/local/redis/redis.log
# /usr/local/redis/data
--
# 验证服务
# 查看状态
chkconfig --list | grep redis        
systemctl status redis_6379            
systemctl stop redis_6379
systemctl start redis_6379
ps -ef | grep redis
# 开放端口
# 重新加载端口
# 验证端口开放情况
firewall-cmd --permanent  --zone=public  --add-port=8083/tcp        
firewall-cmd --reload
firewall-cmd --list-port                
```

Redis 基准测试

```shell
# 测试特定大小包性能
# 测试指定命令
# 只测试某些数值存取性能
redis-benchmark -h 127.0.0.1 -p 6379 -c 100 -n 100000               
redis-benchmark -h 127.0.0.1 -p 6379 -q -d 100
redis-benchmark -t set, lpush -q -n 10000                     
redis-benchmark -n 100000 -q script load "redis.call('set', 'foo', 'bar')"
```



**redis 的管理**

```shell
# 终止某个 host 下某个 port 的 redis 
redis-cli -h 127.0.0.1 -p 6379 shutdown
redis-server ./redis.conf
```



[Centos7安装Redis](https://segmentfault.com/a/1190000017780463)



## RabbitMQ 安装

> 关联： [error-version](https://www.rabbitmq.com/which-erlang.html) | [errlang-download](http://www.erlang.org/downloads/20.1) | [errlang-solutions](https://www.erlang-solutions.com/resources/download.html) | [errlang-rpm](https://github.com/rabbitmq/erlang-rpm/releases)
>
> [RabbitMQ-download](http://www.rabbitmq.com/download.html)  [Github-rabbitmq-server](https://github.com/rabbitmq/rabbitmq-server)

依赖 erlang
『erlang』 ： 编程语言，具有应对高并发的特性

### 源码安装

**安装**

（1） erlang 安装

Erlang 的配置参数： 
--enable-hipe 
--enable-threads 
--enable-smp-support 
--enable-kernel-poll 
--without-javac

```shell
# 安装 erlang 语言依赖
# 解压进入 otp_src_xx, 进行配置
# 编译
# 安装
yum install ncurses-devel
./configure --prefix=/usr/locaI/erlang20  --without-javac
make -j 2
make install
# 验证
# 进入安装位置
# 执行命令
cd /usr/local/elang20
cd bin
./erl
```

（2） RabbitMQ 安装

RabbitMQ 预编译完成

```shell
# 安装依赖 python
# 安装simplejson
yum install python-y  
yum install xmlto 
yum install python -simplgson             
# 解压
# 启动 rabbitMQ server
./rabbitmq-server            
netstat -nap l grep 5672
```

Linux 非命令安装
https://www.erlang-solutions.com/resources/download.html



ST2 : 安装 RabbitMQ

```shell
# 解压， note: xz
xz -d rabbitmq-server-generic-unix-3.6.14.tar.xz
# tar 解压
tar xf rabbitmq-server-generic-unix-3.6.14.tar            
 # move 指定位置
mv rabbitmq_server-3.6.14 /usr/local/rabbitmq             
 
# 安装依赖
yum install python -y
yum install xmlto -y
yum install python-simplejson -y
 
# 相关配置 ( /etc/profile )   以及验证
export PATH=….:/usr/local/erlang20/bin:/usr/local/rabbitmq/sbin
       cd rabbit/sbin
./rabbitmq-server// 启动
       netstat -nap | grep 5672
./rabbitmqctl stop               // stop
```





**配置使用**

环境变量配置

```
vim /etc/profile
ERLANG_HOME=/usr/local/erlang20
export ERLANG_HOME
export RABBITMQ_HOME
export PATH=$PATH:$ERLANG_HOME/bin:$RABBITMQ_HOME/sbin
```

netstat -nap | grep 567

```
firewall-cmd --permanent  --zone=public  --add-port=8083/tcp        
firewall-cmd --reload
firewall-cmd --list-port          
```



### rpm 安装

根据对应 github, errlang solution 下载对应 rpm，已有esl-erlang_21.0-1_centos_7_amd64.rpm， rabbitmq-server-3.7.15-1.el7.noarch.rpm

```shell
rpm -ivh erlang-21.3.8.6-1.el7.x86_64.rpm
rpm -ivh socat-1.7.3.2-1.1.el7.x86_64.rpm
rpm -ivh rabbitmq-server-3.7.15-1.el7.noarch.rpm
# 更改配置文件
vi /usr/lib/rabbitmq/lib/rabbitmq_server-3.7.15/ebin/rabbit.app
  配置替换：
  {default_user, <<"guest">>},     ==>> {default_user, <<"iwms">>},
  {default_pass, <<"guest">>},     ==>> {default_pass, <<"iwms">>},
  {loopback_users, [guest]},       ==>> {loopback_users, [iwms]},
# 启用 rabbit 管理插件
rabbitmq-plugins enable rabbitmq_management
# 更改 hostname
vi /etc/hostname
# 启动 rabbit, 查看日志文件位置 /var/log/rabbitmq/rabbit@<hostname>.log
rabbitmq-server start &
# web 验证
http://172.17.10.150:15672
```

[官网解决guest 默认不可远程连接](http://www.rabbitmq.com/access-control.html)





## Tomcat 安装

Linux安装配置

1.远程下载或本地上传

2.解压并配置环境变量

3.重加载使其生效

4.配置解决乱码和端口问题

5.启动验证

```shell
wget http://dowm.......
tar -zxvf appache-tomcat-…..
sudo vim /etc/profile  
  export CATALINA_HOME=/developer/apache-tomcat……//CATALINA_HOME为安装
source /etc/profile
sudo vim server.xml    //找到对应位置编辑
  <Connector port="8080" protocol="HTTP/1.1"
    connectionTimeout="20000"
    redirectPort="8443" URIEncoding="UTF-8" />
…/bin]$./startup.sh
```

[Docker可视化与管理工具](https://yangbingdong.com/2018/docker-visual-management-and-orchestrate-tools/)： 介绍常用的可视化工具，Rancher、Shipyard、Portainer

[Windows10终端优化方案：Ubuntu子系统+cmder+oh-my-zsh](https://zhuanlan.zhihu.com/p/34152045)



## 服务器初始化

fzf

```
git clone --depth 1 https://gitee.com/janhen/fzf.git ~/.fzf
~/.fzf/install
```

lrzsz

```
yum install lrzsz -y
```

lazydocker

```
file_name=lazydocker_0.8_Linux_x86_64.tar.gz  
wget http://app2.iwms.hd123.cn:8081/ftp/deploy/iwms/tools/$file_name 
tar zxvf $file_name
mv lazydocker /usr/local/bin
lazydocker
```

arthas

```
mkdir -p /opt/arthas
wget -O /opt/arthas/arthas-packaging-3.2.0-bin.zip http://app2.iwms.hd123.cn:8081/ftp/deploy/iwms/tools/arthas-packaging-3.2.0-bin.zip
cd /opt/arthas
unzip arthas-packaging-3.2.0-bin.zip
```

Yum

```
yum -y install yum-utils 
```


```
cd /etc/yum.repos.d/
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

wget http://mirrors.163.com/.help/CentOS7-Base-163.repo
yum makecache
yum -y update
```

Centos VPN 连接

**ppty**  


```
pptpsetup --create sjvpn \
  --server 183.63.53.132:10443 \
  --username wangcb --password Hdydcwcb12#$56 \
  --encrypt --start
  
pon sjvpn

poff sjvpn
```

  

**FortiClient**  
/etc/forticlient  
/var/log/forticlient  
/usr/bin/forticlient  
- fctsched
- fortitray
- FortiClient
- epctrl
```
ftp_prefix=http://app2.iwms.hd123.cn:8081/ftp/Deploy/iWMS/tools/
#file_name=FortiClientFullVPNInstaller_6.4.0.0851.rpm
file_name=forticlient_6.0.8.0140_x86_64.rpm
wget $ftp_prefix/$file_name
yum install $file_name -y

# Remove
yum remove forticlient
```



https://repo.fortinet.com/repo/centos/7/os/x86_64/Packages/



奶牛上传

```
wget http://app2.iwms.hd123.cn:8081/ftp/Deploy/iwms/tools/cowtransfer-uploader_0.4.11_linux_amd64.tar.gz
tar -zxvf cowtransfer-uploader_0.4.11_linux_amd64.tar.gz
mv cowtransfer-uploader /usr/local/bin/
rm -rf cowtransfer-uploader_0.4.11_linux_amd64.tar.gz
```

软件工具包

```
yum install -y lsof netstat
yum install -y git
```



# Linux 服务

### selinux

状态

```
sestatus -v
```

关闭

```
setenforce 0
```

更改配置

```
vi /etc/selinux/config
```



### 时间校准

```
timedatectl list-timezones
# 将硬件时钟调整为与本地时钟一致, 0 为设置为 UTC 时间
timedatectl set-local-rtc 1 
timedatectl status
timedatectl set-timezone Asia/Shanghai
```

```
yum -y install ntp
ntpdate ntp1.aliyun.com
```


How To Sync Time With Timezone On CentOS 7  
https://www.techbrown.com/sync-time-timezone-centos-7/  



## SSH

默认安装好客户端

```shell
yum install openssh-server
serice sshd start
```

SSH Config

存放在 `~/.ssh/config`

```
vim ~/.ssh/config

host "anode1"
  HostName 172.17.10.150
  User root
  Port 22
host "anode2"
  HostName 172.17.10.158
  User root
  Port 22
  
ssh anode1
```

端口更改

```shell
vim /etc/ssh/sshd_config
service sshd restart
```

SSH 免密码密钥认证

```shell
ssh-keygen -t rsa
# 将公钥传递给远程 root 
ssh-copy-id -i /root/.ssh/id_rsa.pub root@172.17.10.158

ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.80.207
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.80.179
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.80.134
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.80.62

ssh-copy-id -i /root/.ssh/id_rsa.pub -oPort=6000 root@127.0.0.1

ssh-copy-id -i /c/Users/Lenovo/.ssh/id_rsa root@192.168.80.24

ssh root@172.17.10.158
```

