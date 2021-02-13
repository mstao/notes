# Docker 基础

> 类似精简的 Linux 环境，含 root 权限、进程空间、用户空间和网络空间，以及运行在其中的应用程序

![1566961968709](http://img.janhen.com/202101301651031566961968709.png)

Client： 客户端通过 CLI 命令与 Docker 交互
Docker daemon： 宿主机的守护进程，通过 RESTful 接口处理 Client 的命令，连接 Registry 进行镜像的拉取的推送，具体配置见 [Daemon配置](#Daemon 配置)
Registry： 保存 image 的地方，实现 image 的维护、复用
Image： 静态的镜像，可根据 Image 运行 container
Container： 依据 Image 生成的具体的容器，实际运行的程序



Docker 底层实现原理：

- Namespaces：做隔离pid，net，ipc，mnt，uts
- Control groups(cgroups)：做资源限制
- Union file systems: Container和image的分层，分层文件系统



## 镜像

> 一个特殊的文件系统，提供容器运行时所需的程序，同时包含一些为运行时准备的配置参数，无法更改

**镜像的获取**

- 根据 Dockerfile 构建镜像，配合 sh 脚本实现一些定制的初始化和参数判断逻辑，可重建
- 根据容器构建镜像，在只读镜像上操作可写容器重新打包成镜像，Docker 无状态，volume 不会打包进镜像，较少使用
- 从远程 Registry 拉取镜像

```shell
# 从远程 registry 拉取
docker commit wonderful_mendeleev janhen/centos-vim-gcc:1.0.0
# 从 Dockerfile 构建
docker build -t janhen/myimage:1.0 .
# 从容器创建
docker pull <registry_host>/<username OR project_name>/<image_name>:<image_tag>
```

**镜像 tag**

```shell
# image 的查、交互
docker images
docker history <image_id>
docker tag <image_old_name> <image_new_name>
```

**镜像清理**

处理同一个版本多次覆盖，默认查找顺序为 Local -> Registry 的问题

```shell
#   删除指定的 image
docker rmi <image_id OR image_name>
#   强制删除指定|全部 image
docker rmi -f $(docker images)
#   删除 <none> 的镜像(#)
docker rmi $(docker images -f "dangling=true" -q)
docker images | grep none | awk '{print $3}' | xargs docker rmi
```



## 容器

>  是镜像运行时的实体，构建在镜像上，可对容器进行写操作

**Container 的启动并运行**

单机上使用最多，控制部署时候的各种参数，包含网络、存储、密码、变量...

常用的启动指定：

- 指定网络，根据需要选择端口转发、单机桥接网络、多机网络、主机网络

- 指定文件映射，将程序中的配置文件、数据文件隔离出来，避免应容器销毁而丢失
- 指定命令，内部运行的程序自带的命令，如 Redis 中的命令控制持久化方式...
- 指定变量，通过命令方式、环境变量方式指定，让运行容器更加定制化

```shell
# 容器的运行
#   --name: 按照特定名称启动，作为容器标识
#   -d: 后台运行
#   -i: 交互式运行容器，打开STDIN，用于控制台交互 
#   -t: 终端方式交互, 通过 bash、shell... 进行命令式交互
#   -p: 映射宿主机与容器的端口号
#   --network=<value>: 指定网络连接类(#)
#   -v: 进行宿主机文件与容器文件的映射(#)
#   --<param>=<value>: 进行特定参数指定，传入中的参数，在容器中的文件处可引用
#   -e: 指定环境变量, 对应镜像提供，与 Dockerfile 中指定的 ENV 等同，可进入容器使用 env 查看(#)
#   --privileged=true: 给容器扩展的权限
#   --rm: 在容器终止运行后自动删除容器文件，避免磁盘浪费，常用于测试
#   --restart=<strategy>: 重启策略，与 --rm 参数冲突，提供多种策略
#   --entrypoint: 覆盖默认镜像的 ENTRYPOINT
#   --link: 添加链接到另一个 container, 不建议使用
#   -w: 指定工作目录，等价于 Dockerfile 中的 WORKDIR
# 启动过后执行一段 Shell 脚本, 用于测试环境类镜像使用
docker run ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
# 以命令行方式进入容器，查看镜像具体情况
docker run -it --entrypoint bash openjdk:7-jre 
# Dockerfile 中环境变量配合运行指定 JVM 运行参数、运行端口，参数名仿照 spring-boot maven 插件
docker run -d -p 7070:7070 -e JVM_OPTS="-Xms1024m -Xmx2048m" -e PROGRAM_ARGS="--server.port=7070" com.blinkfox/web-demo:1.0.0
docker run -e "JAVA_OPTS=-agentlib:jdwp=transport=dt_socket,address=5005,server=y,suspend=n" -p 8080:8080 -p 5005:5005 -t springio/gs-spring-boot-docker
docker run -d --name test1 \
  -e MYENV=AAAA \ 
  busybox /bin/sh -c "while true;do sleep 3600;done"
```



**容器的信息查看**

```shell
# 容器整体信息查询
docker info
docker info | grep "Docker Root Dir"
docker ps [(-a)|(-aq)]?
# 配置信息
docker inspect <container>
docker inspect -f {{xx.yy}} <container>
# 交互，调试
#   日志，  -f ： follow log output，持续实时显示日志，   -t:......
#   命令交互
#   在容器中执行特定命令
# 日志查看
docker logs <contain_id OR container_name>
docker logs -f <container_id OR container_name>
# 容器内部执行
docker exec -it <container_id> bash
docker exec <container_id> ip a
docker exec -it <container_id OR container_name> env
# 运行信息
docker stat <container>
```



**容器基础命令**

```shell
# 容器的启、停
docker container start|stop|restart <container_id OR contaienr_name>
# 导入导出
#   导出容器成指定的 tar 包
#   容器快照文件导入为*镜像*
#   URL/目录导入
docker export 7691a814370e > ubuntu.tar
cat ubuntu.tar | docker import - test/ubuntu:v1.0
docker import http://example.com/exampleimage.tgz example/imagerepo
```



**容器的清理**

```shell
# 删除|强制删除指定的容器
docker container rm <container_id OR container_name>
docker contaienr rm -f <container_id OR container_name>
# 删除所有容器
docker rm $(docker ps -aq)
docker rm -f $(docker ps -aq)
# 删除已停止运行的容器(#)
docker rm $(docker ps -f "status=exited" -q)
```



**Container 交互**

> 容器内部可执行的命令，特定目录存储的配置内容，可以通过 CLI 的监控命令

支持更改 /etc/hosts， /etc/hostname，/etc/resolv.conf ，只针对运行时，临时的更改

几种交互方式：

- 运行时直接进入交互、运行时直接执行命令交互，包含对文件的操作、内部命令执行

- 运行后按特定终端进入交互、运行后按特定命令交互，同上

- 日志交互，logs，支持最后几行、最近的时间点、实时显示

- 基本情况，inspect，返回运行情况 JSON 字符串，可通过 Go  Templete 获取特定情况

- 运行的资源情况，stats，实时显示 CPU、内存、网络、磁盘情况

```shell
# 进入容器内部
docker exec -it -u root jenkins sh
# 执行特定命令
#   创建之后执行
#   在已运行的容器中执行命令
docker run -it --rm ubuntu:18.04  ip a
docker run -it --rm ubuntu:18.04 --hostname=test.com --dns=172.16.3.3 ip a
docker run -it --rm ubuntu:18.04  cat /etc/resolv.conf
docker exec -it gitlab cat /etc/resolv.conf
docker exec -it gitlab cat /etc/hostname
docker exec -it gitlab cat /etc/hosts
# 容器内部执行
#   查看挂载情况
#   查看定义的环境变量
#   查看 dns 情况, 与在宿主机上的 /etc/docker/daemon.json 上配置 dns 类似?
#   查看容器IP地址配置
#   查看路由情况
mount
env
cat /etc/resolv.conf
ip addr show eth0
ip route
# logs 查看
#   特定时间偏移, 特定时间段
docker logs -f -t --since=40m --tail=10 jenkins
docker logs -t --since="2019-08-01T13:23:37" --until "2018-08-31T12:23:37" jenkins
# inpect 查看
docker inspect -f '{{.State.Pid}}' 1f1f4c1f931a
# stat 查看
docker stats <container_id OR container_name>
# 拷贝文件，作为 Dockerfile 中 COPY 的...
docker cp <host_machine file OR dir> <container_name>:<container_dir>
```



## Registry

> Docker 的私有仓库，实现容器的复用共享

发布镜像到 Registry 的方式：

- 发布镜像到仓库

    直接将本地已经构建好的镜像发布到仓库中

- 根据指定 Dockerfile 由 Docker hub 进行构建形成镜像

  自动在 git 发生变化的时候拉取数据进行构建重新发布到仓库上，自动构建发布，CICD

  保证镜像的可再生性

私有 Registry 搭建：

官方提供的 registry

Vmware 开源的 harbor，见 [工具与环境](./B-工具与环境.md#harbor)

```shell
# 登录 docker hub 账号和密码
# 推送镜像到 docker hub
# docker hub 关联 github or bitbucket
docker login 172.17.11.29:5111 -u admin -p Harbor12345

# 重命名镜像的名称(tag)
docker push 172.17.11.29:5111/centos-vim-gcc:1.0.0
docker tag janhen/centos-vim-gcc:1.0.0 172.17.11.29:5111/study-docker/centos-vim-gcc:1.0.0
docker push 172.17.11.29:5111/study-docker/centos-vim-gcc:1.0.0
```



## Docker 网络

> 进行容器之间的访问，包含单机上的访问，多台机器之间的访问； 含端口映射、容器互联
>
> 关联文档:  [使用网络](https://yeasy.gitbooks.io/docker_practice/network/)  |  [高级网络配置](https://yeasy.gitbooks.io/docker_practice/advanced_network/)

**Linux 上网络访问**

>  Linux 网络命名空间，进行网络的隔离

Veth pair： 进行网络命名空间的连接，实现两个 net namspce 连接通信

```shell
# 网络命名空间
# ip link
# 给命名空间分配 ip 地址, 默认情况下只有 mac 地址
# 启动接口
# 连接双方使其网络互通
ip netns list
ip netns delete test1
ip netns exec test2 ip link
ip netns exec testl ip link set dev veth-testl up
ip netns exec testz ip link set dev veth-test2 up
```



### Docker 网络访问

> 通过link 方式实现容器之间的访问，直接通过名称而非 IP，适用于单台机器

![1566531692115](http://img.janhen.com/202101301651101566531692115.png)

一个容器对应一个网络空间

类似局域网连接，通过中间的交换机实现两个容器之间的通信， docker0 的内网指定默认为 172.17.0/16，自定义为 172.17.18.0/16...

访问外部网络，需要经过 NAT 转换

```shell
# 查看容器网络， bridge 网络
docker network ls
sudo docker network inspect <network_id OR network_name>
ip a
yum install bridge-utils
# 展示系统当前桥接
brctl show
ip a
# 创建指定类型的网络
docker network create -d bridge net-my
```



**Docker link 网络连接**

> 通过命名 Docker 进行相连，类似网络命名空间中的 Veth pair，目前不推荐使用
>
> 命令格式： --link <container_name OR container_id>:<alias>
>
> 替代方案： docker-compose.yml 中使用 depends_on，使用 overlay 网络
>

```shell
# 类似给 net-test2 添加 DNS 记录
#   link 方向性; 使用少
docker run -d --name net-test2 \
  --link net-test1 busybox \
  /bin/sh -c "while true; do sleep 3600; done"
docker exec -it net-test2 /bin/sh
  ip a 
  ping net-test1
# -d 指定网络类型， bridge|overlay
docker network create -d bridge net-my
docker run -it --rm --name busybox1 --network my-net busybox sh
docker run -it --rm --name busybox2 --network my-net busybox sh
```

```shell
# --link <name> 支持通过名称访问容器 
docker run -d --name flask-redis \
  -p 5000:5000 \
  --link redis \
  -e REDIS_HOST=redis \
  janhen/flask-redis 
docker run -d --name test1 \
  -e PENG=testt1 \
  busybox 
docker run -d --name test2 \ 
  -e PENG=testest \
  busybox  \
  /bin/sh -c "while true; do sleep 3600; done"
```



**自定义网络连接**

> 避免使用 --link 进行容器之间网络的连接

```shell
dockernetwork create -d bridge net-demo
```



### Docker 单机网络

**Docker bridge 网络**

> 可以创建自己的桥接网络，进行区分，docker-compose 默认管理的容器共享同一个 bridge 网络

```shell
# 创建自己的桥接网络
docker network create -d bridge my-bridge
docker network ls
brctl show
docker run -d \
  -- name net-test3 \
  --network my-bridge busybox /bin/sh -c "while true; do sleep 3600; done"
docker network connect mybridge net-test2
```

bridge 性能一般，对性能要求较高，可使用个 SR-IOV 网卡嵌入容器内。



**Docker host 和 none网络**

> none 网络： 不会有网络信息，孤立的网络，用来做私有的工具，如保存密码??，使用场景少
>
> host 网络：无网络信息，与主机共享网络命名空间，存在端口冲突问题

```shell
docker run -d -p 80:80 nginx 
```



### Docker 多机网络

> 实现多个不同机器之间的容器进行通信


#### Overlay 网络

> 依赖一个分布式存储，保存对应的 IP，防止网络(172.18.0.0/16)、容器名称等的冲突

实现 Docker 的多机网络，见 [Internel 访问](#Internel 访问)

两台机器之间可以相互通信，为了实现不同容器之间的通信需要借助第三方的分布式存储

使用etcd 建立的 cluster 中容器名称不允许重复、Ip 地址不允许重复

```shell
# 创建 overlay 网络，实现多态主机之间的同步创建 overlay 网络
docker network ls
docker netword create -d overlay net-overlay-demo
# 查看网络情况，子网范围，容器情况
docker network inspect net-overlay-demo
# 启动容器指定到 overlay 网络
docker run -d --name node1-test1 \
  --net net-overlay-demo \
  busybox sh -c "while true; do sleep 3600; done"
docker run -d --name node2-test1 \
  --net net-overlay-demo \
  busybox sh -c "while true; do sleep 3600; done"
# 查看节点上容器的地址
# 查看 cluster 中网络的情况
docker exec node1-test1 ip a
docker exec node2-test1 ip a
docker network inspect net-overlay-demo
```



#### **Etcd 分布式存储**

> 存储分布式系统中的 key-val，开源免费，保证 overlay 网络中分配的容器与容器对应的IP地址在整个网络中唯一
>
> 关联： [GitHub](https://github.com/etcd-io/etcd)

```shell
# 在对应的两台机器上安装 etcd，容器安装/binary 安装

# 通过命令指定好集群启动

# 验证 cluster 的运行情况
# 进入 etcd 文件夹执行健康检查，两台机器同时执行
./etcdctl cluster-health

# 关闭 Docker 服务
# 使用 etcd 作为分布式存储启动 docker， 手动启动 dockerd 守护进程
# 验证
systemctl stop docker
sudo /usr/bin/dockerd -H tcp://0.0.0.0:2375 \
  -H unix://var/run/docker.sock \
  --cluster-store=etcd://192.168.xx.xx:2379 \
  --cluster-advertise=192.168.xx.xx:2375 &  
docker version
```



## Docker 持久化

> 将容器与数据存储隔离开，如 Mysql 运行程序与数据保存位置

![1566571502370](http://img.janhen.com/202012172237411566571502370.png)

两种持久化的方式：

- 本地 FS 的 Volumn

- 基于 plugin 的 Volume， 如 NAS



本机上三种持久化实现, -mount 选项选择数据卷：

- bind :挂载在 Linux FS 中任意位置

- volume：统一挂载在 daemon 设置的 docker 目录下，默认为 `/var/lib/docker/volumes/<unique_str_id OR volume_name>`

- tmpfs： 只挂载在内存中，易丢失



使用命令:

```shell
docker volume create -d local test
docker volume inspect <contaienr>
# 清理
docker volume prune <>
docker volume rm <>
docker run -d --mount type=bind, source=/data, destination=/redis/data xxxx
# 指定 :ro 容器无法对挂载数据卷内的数据进行修改
docker run -d -v /webapp:/opt/webapp:ro
```



**数据卷容器**

实现多个容器操作数据，任意容器修改都可被其他容器看到

--volumes-from 参数所挂载的数据卷容器无需处在运行状态

```shell
docker run -d --volumes-from dbdata xxx
```



### Volume

> 通过 Dockerfile 中的 Volumn 控制，在宿主机上 docker 文件下建立目录存放文件

建议 -v 参数指定在 docker 目录下 volume 的名称，默认为 `/var/lib/docker/volumes/<-v_name OR long_str>`

针对官方镜像，到 Docker Hub 上查看对应的 volume 挂载目录位置

```shell
# 创建 volume，查看所有|指定|删除volume
docker volume create volume1
docker volume ls
docker volume inspect volume1
docker volume rm volume2
# 运行->删除->验证
docker run -d -v mysql1:/var/lib/mysql  \
  --name mysql1  \
  -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql 
docker rm -f  mysql1 mysql2
docker run -d -v mysql1:/var/lib/mysql --name mysql1 -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql 
docker exec -it mysql2 /bin/bash
  mysql -u root
  show databases;
```



### Bind Mouting

> 指定容器目录与宿主机目录绑定，宿主机文件更改影响到容器中的运行

可以实现本台电脑  -->  虚拟机  -->  容器三者的目录映射

```shell
# -v: <宿主机目录>:<容器目录> 进行一一映射
docker run -d -v $(pwd):/usr/share/nginx/html -p 80:80 --name web janhen/my-nginx
# 使用 Docker 作为本地开发环境
docker run -d -p 80:5000  --name flask janhen/flask-skeleton 
```



## Dockerfile 编写

> 用于生成 Docker Image 的文件，一般只用于 `docker build -t janhen/xx:99 .` 命令执行使用 
>
> 关联： [Dockerfile 指令](https://yeasy.gitbooks.io/docker_practice/appendix/best_practices.html#dockerfile-指令)

### 语法

**Dockerfile 的基本语法**

> FROM,WORKDIR,ENV,COPY,ADD
>
> RUN,CMD,ENTRYPOINT
>
> VOLUME,EXPOSE

- `FROM`： 根据特定的镜像制作，从头制作、 根据指定环境制作、某个镜像作为构建阶段使用

- `RUN `： 运行命令脚本, 可以通过此安装一些环境并对环境进行配置，如安装 Node 环境，每运行一个命令增加一层 ==>  建议将多个命令合并成一个命令使用
- `WORKDIR`： 设定当前工作目录, 类似 cd 改变目录, 没有目录自动创建(#)
    直接通过绝对路径定位
    通过绝对路径+相对路径定位目录
- `ADD and COPY`： 将本地文件添加到 docker image 中,常 配合 WORKDIR 使用


```shell
FROM python:3.7
# LABEL 镜像的 metadata，帮助信息
LABEL maintainer="janhen <ipaam414@gmail.com>"

RUN yum update && yum install -y vim \
  python-dev
RUN apt-get update && apt-get install -y perl \
    pwgen --no-install-recommends && rm -rf \  
    /var/lib/apt/lists/*
RUN /bin/bash -c 'source $HOME/.bashrc;echo $HOME'

WORKDIR /root
WORKDIR /test
WORKDIR demo
RUN pwd

ADD hello /
ADD test.tar.gz /
WORKDIR /root
ADD hello test/
COPY hello test/

# ENV 设定环境变量, 建议使用
ENV MYSQL_VERSION 5.6
RUN apt-get install -y mysql-server = "${MYSQL_VERSION}" \
    && rm -rf /var/lib/apt/lists/*

# CMD
# 设置容器启动后默认执行的命令和参数
#   docker run 指定其他命令, CMD 被忽略
#   定义多个 CMD，只运行最后一个
docker run [image]                # CMD 会被执行
docker run -it [image] /bin/bash  # CMD 不会执行
CMD ["mongod"]

# ENTRYPOINT
# 设置容器启动时运行的命令
# 容器以应用程序/服务的形式运行
#   不会被忽略, 一定会执行
#   最佳实践: 通过 shell 脚本作为 entrypoint 
COPY docker-entrypoint.sh /usr/local/bin/     # 添加到容器中
ENTRYPOINT ["docker-entrypoint.sh"]           # 指定入口脚本
EXPOSE 27017
ENTRYPOINT ["scripts/dev.sh"]

# 进行宿主机与容器中文件的映射
# 映射容器中的 /tmp 到宿主机上，默认在 /var/lib/docker/volumes/<long_id OR name> 下建立对应的映射
VOLUME /tmp
```



**命令格式**

> 不同的命令执行写法，以及对应的区别

Shell 格式, 默认通过 shell 执行

Exec 格式,   ENTRYPOINT ["/bin/bash", "-c", "echo", "hello $name"]

针对 Exec 无法映射变量问题的处理： 通过命令方式编写语句

```shell
ENV name Docker
ENTRYPOINT ["/bin/bash", "-c", "echo", "hello $name"]
```



**命令区别**

1、RUN、CMD和ENTRYPOINT命令区别

RUN 运行在 image 的构建阶段执行，执行结果会被打包进 image 文件

CMD 在容器启动后执行，可用于在容器内启动某个服务、进程，只可使用一次，与 run 中年执行命令冲突

ENTRYPOINT 在容器启动后执行，出现多行不会忽略，一定执行，通常配合 COPY 到容器中的 sh 脚本使用

2、COPY 与 ADD 命令区别

ADD 可以获取网络资源，可以直接解压缩



**注意事项**

1、CMD 的最后一次有效性

官方镜像中大多最后运行 CMD，方便覆盖实现定制化的参数的启动

2、目录

COPY . /app 与 COPY . /app/ 映射不同





# Docker Compose

> 多容器管理，通过 yml 配置管理容器之间的依赖关系，底层 python 编写，前身为开源的 Fig 项目。主要用于本地开发使用。
>
> 关联文档：[Compose 模板文件](https://yeasy.gitbooks.io/docker_practice/compose/compose_file.html) | [Doc](https://docs.docker.com/compose/compose-file)

## 管理

> docker-compose 的启动、停止、交互

```shell
# compose 后台启动
# 启动并查看日志
docker-compose up
docker-compose up -d
docker-compose -f <compose_name> up -d 
# 停止服务
# 停止并删除 容器、网络、volumes
docker-compose stop <service>
docker-compose down <service>
docker-compose build
# compose 查看运行情况，状态、端口情况
# 查看 compose 中定义容器使用的 images
docker-compose ps
docker-compose images
# 进入 compose 中的 service
docker-compose exec mysql bash
# 扩展
docker-compose scale <service_name>=<count>
```

 

**service 的扩展**

>  实现水平扩展，负载均衡，在不存在端口冲突的情况下通过 haproxy 进行负载均衡，在 Docker Swarm 运行时可直接通过 deploy 中的参数指定复制扩展的个数

通过 docker-compose scale 命令进行扩展

处理 scale 中端口映射重复问题

在 docker-compose 中增加 dockercloud/haproxy 进行负载均衡
```shell
docker-compose up -d
# 启动时指定扩展
docker-compose up --scale web=3 -d
# 运行后进行扩展
docker-compose scale web=4
# 验证扩展情况
docker-compose ps
for i in `seq 10`; do curl 127.0.0.1:8080; done
```




## 语法

> 对应 docker-compose.yml 文件的语法
>

三大实体：

- service: 服务

- networks: 网络指定，指定网络类型，一般为 bridge、overlay，根据需要指定多个网络，进行一定的隔离

- volumes: 进行数据卷的映射

---

image 获取方式： 通过 image 获取本地的或是拉取远程的，或者通过 build 进行构建，传入 Dockerfile 的目录以及对应的 Dockerfile 名称

ports: 进行宿主端口与容器端口的映射

depends_on: 解决容器的依赖，启动先后问题

links: 服务之间的依赖关系，在容器内部可以直接使用依赖服务名称对应的 IP 地址，不建议使用

deploy: 进行部署，控制集群中的各种情况，用于 Docker swarm，version 3 支持

```shell
# 特定片段参考
# 设置网络, 可多个
# frontend, backend 前后端设置
networks:
  - frontend
  - backend
# 端口设置
# 直接引号设置
# 宿主机与容器端口映射
ports:
  - "6379"
ports:
  - 5000:80
# 依赖
depends_on:
  - mysql
```





# Docker Swarm

> Docker 自带的服务编排框架，大多数都由其中的 Manager 做管理，较难定制，不适合太多节点的部署

Docker Swarm 特点：

符合传统IT的管理模式

平台本身集成性好，可当成云管平台使用

内置太多不易进行定制化，不好 Debug，不易干预

**不提供存储选项**：Docker Swarm不提供将容器连接到存储的无障碍方式，其数据量需要在主机和手动配置上进行大量即兴创作

**监控不良**：Docker Swarm提供有关容器的基本信息，如果您正在寻找基本的监控解决方案，那么Stats命令就足够了。如果您正在寻找高级监控，那么Docker Swarm永远不是一个选择。虽然有像CAdvisor这样的第三方工具可以提供更多监控，但使用Docker本身实时收集有关容器的更多数据是不可行的。



## Swarm 架构

![1566634422450](http://img.janhen.com/202012172238091566634422450.png)

[Raft](http://thesecretlivesofdata.com/raft/) consensus group： 进行控制分布式场景下的协商:  内置的分布式的存储数据库，通过 Raft 协议进行同步，包含 Leader 选举、Log 复制

Internel distributed state store： 分布式存储数据库，功能如保证分布式场景下 Ip 等唯一，类似 etcd

Manager: 可以保存 Raft 关联的文件，用于 Secret 实现

Worker: 主要运行容器，通过 Gossip network 进行通信，保证分布式下的一致性

Gossip network： 各个 Worker 之间同步实现的协议



扩展：

  Service: 通过 swam manager 进行控制，具体 service 部署到哪个 node 上

  Replicas： 一个 Service 对应多个 Replicas，用于扩展



## 集群搭建管理

> 让几台服务器搭建成一个 Swarm Cluster

```shell
# 配置 Manager Node
docker swarm init --advertise-addr=192.169.xx.xx
# 配置 Worker Node 加入到特定的 Manager Node
docker swarm join --token xxxfsdfsdf <ip>:<port>
# 查看当前 Node 情况
# 节点查看
# 节点降级
docker node ls  
docker node inspect <node_name>
docker node demote <node_name>
docker node ps
```



## Swarm管理

**Swarm Services 管理**

> 单个 Service 的管理，一个 Service 可扩展到多个 cluster node 上的 Container 运行

```shell
# 创建容器，运行位置有 mananger 进行控制运行在哪个节点上
# 类似 docker run 命令，在本地创建 container
# 查看 service 情况
#   MODE: replicated
#   REPLICAS: 1/1 支持水平扩展，类似 docker compose 中的 scale
# 查看具体的 service 情况
#   运行在哪个节点上
# 扩展servie，通过复制的方式(#) 
docker service create --name demo busybox \
  sh -c "while true; do sleep 3600; done"
docker service ls
docker service ps demo
docker service scale demo=5
docker service ps demo 
# 本机查看 docker 容器运行
# 强制删除某个正在运行中的容器
# 集群自动恢复，确保一定数目的 scale 扩展有效，系统稳定运行时
# 显示节点中容器运行情况
# 删除服务，对应的集群节点容器删除
docker ps
docker rm -f e64432
docker service ls
docker service ps demo
docker service rm demo
```



## RoutingMesh

> Swarm 网络通信原理，管理集群服务间的通信，访问集群中任何一个节点特定端口都会被重定向到实际运行服务的节点上

![1566634990023](http://img.janhen.com/202012172238131566634990023.png)

DNS 服务发现，单机情况下可以通过 service 的名称进行相互访问，多机情况下通过 swarm 进行相互访问

VIP： 非真实机器的IP地址，避免多个IP地址变化问题，造成系统运行不稳定，一个 service 对应一个

LVS： 根据虚拟 IP 找出容器中的具体的 IP 地址



两种体现：

- Internel：容器之间通过 overlay 网络访问
- Ingress ：服务绑定接口的情况，此服务通过任意 Swarm 节点对应接口访问



### Internel 访问

> 容器间实现相互访问，通过 overlay 网络实现，实现 service 与 service 之间的通信

![1568514162156](http://img.janhen.com/202012172238171568514162156.png)

whoami 镜像： 提供 web 服务，访问 8000 端口，返回 container 的 hostname

```shell
# 创建 overlay 网络
# 创建 whoami 服务
#   后台运行
#   端口映射
#   网络指定
# 查看所有 service 
# 查看 whoami 服务运行位置
# 到对应机器上验证
docker network create -d overlay net-demo
docker service create -d \
  --name whoami \
  -p 8000:8000 \
  --network net-demo jwilder/whoami
docker service ls
docker service ps whoami
docker ps
curl 127.0.0.1:8000
# 创建 busybox 的容器
#   连接到同一个 overlay 网络
# 查看所有服务，当前 busybox service 是否启动完成
# 查看服务 client 服务具体位置
# 进入对应的机器查看对应运行的 container
# 进入容器
#   10.0.0.7 IP 地址，为虚拟 IP， 将 whoiam 通过 scale 扩展
# 通过 scale 进行扩展 whoami 
# 查看 whoami 位置，并进入
# 进入对应 client contaienr 中
#   连接 whoami 
#   查询 dns，只有一个虚拟IP 10.0.0.7
# 进入容器 whoami 查看网络地址
# 进入容器 whoami(另一) 查看网络地址
# 进入容器 client
#   查看 task.whoami，返回对应的多个节点，为真实的 IP 地址（#）
docker service create -d \
  --name client \
  --network net-demo busybox \
  sh -c "while true; do sleep 3600; done"
docker service ls
docker service ps client
docker ps
docerer exec -it <container_id> sh
  ping whoami    
docker service scale whoami=2    
docker service ps whoami
docker service ps client
docker exec -it <container_client_id> sh 
  ping whoami 
  nslookup whoami
docker exec 5b79 ip a
docker exec	df9 ip a
docerk exec -it <container_client_id> sh
  nslookup task.whoami
# 扩展 whoami 服务
# 查案 client 对应的 task.whoami，显示三个对应的(whoami)IP 地址
#   ==> 虚拟IP: 不会随 service 的扩展而变化, 包括增加、减少、机器之间的迁移不会变化(#)
# 访问多次服务 whoami，相应的对应机器上的容器会因为负载均衡而不同，通过 LVS 实现
docker service scale whoami=3
docker service ps whoami
---
  nslookup task.whoami
  wget whoami:8000
  more index.html
  rm -rf index.html
  wget whoami:8000
```

两种体现：

Internal: 容器键通过 overlay 网络(VIP)访问

Ingress: 服务绑定接口, 通过任意 swarm 节点的接口访问’



DNS + VIP + iptables + LVS 实现的过程图：

// todo 具体 Swarm 网络中数据的流动情况

小结： 容器之间连接到 overlay 网络进行通信，service 之间的通信通过  VIP + LVS 实现



### Ingress 负载均衡

> 绑定端口实现的容器之间的访问，通过 <node_ip>:<service_port> 直接访问服务
>
> 作用体现：集群中的 Node 对应的端口提供相同的服务，即使 Node 本地无服务也支持访问

Ingress Network 的数据包走向图

![1568515008492](http://img.janhen.com/202012172238261568515008492.png)

在  IPTables + IPVS 发往目的网络

```shell
# 常看网络桥接情况
# 查看机器的网络命令空间
# 进入 ingress_sbox 网络命名空间
iptables
brctl show
sudo ls /var/run/docker/netns
sudo nsenter --net=/var/run/docker/netns/ingress_sbox
ip a
iptables -nL -t mangle
# 安装 LVS 管理工具
# 
# 查看 LVS 情况，展示可选的服务 IP 地址，展示机器的 weight, 
yum install ipvsadm
sudo nsenter --net=/var/run/docker/netns/ingress_sbox
iptables -nL -t mangle
ipvsadm -l
```





##  Docker Stack 部署

> 进行多服务部署，可以使用 docker-compose.yml ，只能用于 swarm cluster，无法用于其他的服务编排框架

docker-compose.yml 文件更改

compose file version 3: 增加 [deploy](https://docs.docker.com/compose/compose-file/#deploy) 命令，具体参数如下

```yml
# deploy
#   endpoint_mode: vip 模式(默认), dnsrr 模式 循环访问(少用)
#   labels: 帮助描述信息
#   mode: global, replicated， global 全局唯一, 无法通过 scale 横向扩展，一般外部服务使用此种方式，如 mysql,nginx,redis等； replicated 默认，可通过复制来进行扩展
#   placement:   
#     constraint: 
#       - node.role == manager # 限制部署到 manager 节点上
#     preferences: 优先喜好
#       -
#   replicas: 在 mod 是 replicated 的时候定义初始化时候需要的 replicas
#   resources: 进行资源的限制
#     limits:
#       cpus: '0.50'  # CPU 使用限制
#       memeory: 50M  # 内存使用限制
#     reservations:   # 优先保留，最小的情况
#       cpus: '0.25'
#       memory: 20M
#   restart_policy:         # 容器宕机后的处理
#     conditon: on-failure  # 什么情况下重启
#     delay: 5s             # 延迟
#     max_attempts: 3       # 最大重试次数
#     window: 120s
#   update_config:          # service 更新的配置
#     parallelism: 2
#     delay: 10s
#     order: stop-first
```



部署的过程：

更改单机的 docker-compose.yml 为对应 cluster 部署(deploy)

按条件执行命令： 如下

验证： 通过访问任意一个 cluster 中的地址即可访问

```shell
# 整个 application 定义为一个 stack 为 wordpress
#   可通过 -c=docker-compose.yml 进行简化
# 查看运行情况
#   mysql: 限制只运行一个，只能运行在 manager 节点
# 通过 stack 查看服务情况
docker stack deploy wordpress --compose-file=docker-compose.yml
docker stack ls
docker stack ps wordpress
docker stack services wordpress
docker stack rm wordpress
```



## Docker Secret 管理

> 对一些密码进行管理， 处理 docker-compose.yml 中存储密码不安全问题，借助内部分布式存储数据库控制，只作用于 Docker Swarm
>
> 关联： [Doc-CLI](https://docs.docker.com/engine/reference/commandline/secret/)

![1566631044524](http://img.janhen.com/202012172238471566631044524.png)

Secret 类型： username password， SSH key, TLS 认证，不想让人看到的数据

生产环境至少要两个 Manager，分布式存储的天然加密环境



Secret 的管理：

将 Secret 存储在 Manager 中的分布式存储中的 Raft Database

Secret 给某个 service 指派



**Service 基本使用**

Secret 的创建方式：文件方式、输入方式。

存放在容器中的 `/run/secrets/<secret_file_name>` 文件中

```shell
# 按文件方式进行创建
# 删除文件，保证安全性
# 查看 secret
# 借助管道按照输入方式创建 secret
vim password
docker secret create my-file-pw password
rm -rf password
docker secret ls
echo "mypassword" | docker secret create my-input-pw
docker secret rm my-input-pw
# 通过 swarm service 创建过程中指定 secret 进行使用
# 进入容器查看指定目录，找到 manager 通过 Raft Database 保存的 secret
docker service create -d --name client \
  --secret my-file-pw busybox \
  sh -c "while true; do sleep 3600; done"
docker service ls 
docker service ps client
docker ps
docker exec -it <client_container_id> sh
  cd /run/secrets/
  ls
  cat my-file-pw # 原文
# 实际使用
# 在创建 service 的时候指定好 secret，并在环境变量中指定在容器中的位置
docker service create -d --name db \
  --secret my-file-pw \ 
  -e MYSQL_ROOT_PASSWORD_FILE=/run/secrets/my-file-pw mysql
docker service ps db
--
docker exec -it <db_container_id> sh
ls /run/secrets
cat /run/secrets/my-file-pw
mysql -u root -p
```



**在 Stack 中的使用**

在服务配置下增加 secrets，指定对应的 Secret

对应的密码参数使用指定的 secrets 在容器中的位置

可以连通创建 secret 一起使用，不建议

```yml
# -c 简化 --compose-file 
# 查看服务是否全部启动完成
docker stack deploy wordpress \
  -c=docker-compose.yml
docker stack services wordpress
```



## Docker Service 更新

> 在运行过程中对 service 依赖的镜像进行升级，实现升级过程中不会中断原来的服务

### 单 Service 更新

进行 service 的更新，不会暂停运行的项目

@Q: 存在一段时间有 1.0和2.0并存的情况，如何处理??

```shell
# 创建 overlay 网络，启动服务
# 等待服务启动完毕
docker network create -d overlay net-demo
docker network ls
docker service create -d --name web \
  --publish 8080:5000 \
  --network net-demo
  janhen/python-flask-demo:1.0.0
docker service ps web
# 扩展服务
# 检查服务运行情况
# 编写测试脚本方便验证
docker service scale web=2
docker service ps web
curl 127.0.0.1:8080
sh -c "while true; do curl 127.0.0.1:8080&&sleep 1; done"
# 更新镜像，一般通过 Dockerfile 进行构建，指定对应更新的版本，发布到私有 registry
# 运行环境拉取镜像，执行更新命令
docker service update --image janehn/python-plask-demo:2.0.0 web
```

```shell
# 更新镜像并设置参数, 覆盖 docker-compose.yml
docker service update --image westos.org/game2048 \
  --update-parallelism 10 \
  --update-delay 10s \
  nginx
```



### 端口更新

>  对 service 与宿主机的端口映射进行更改

删除掉原来的端口映射，无法做到更新时业务不中断，通过 VIP + 端口实现原理导致的

```shell
docker service update --publish-rm 8080:500 \
  --publish-add 8088:5000 web
```



### Stack 更新

> 更改 Swarm Cluster 中多个容器中对于镜像、网络、部署配置的更新，关联 <service>.deploy.update_config 下的配置

可更改 docker-compose.yml 中 deploy 下的 update_config 控制更新时的细节，允许几个 scale 进行更新，延迟信息。。。

第一次通过 deploy 进行启动进行了多 service 的部署

第二次通过 deploy 部署时，自动检测到 docker-compose.yml 的变化，进行更新	

```shell
docker stack deploy wordpress --compose-file docker-compose.yml 
```



## Docker Swarm 监控

> 实现对 Docker Swarm Cluster 中运行节点上容器的监控

### CAdvisor+InfluxDB+Grafana

> docker swarm集群的监控方案，开源免费
>
> cAdvisor：数据收集模块，需要部署在集群中的每一个节点上，当然前提条件是节点接受task。
>
> InfluxDB：数据存储模块
>
> Grafana：数据展示模块



**Docker Universal Control Plane(UCP)**

> docker原厂的可视化集群管理GUI，企业级的，只支持docker EE



**portainer**

> 在集群中部署portainer的service，只能被调度给manager角色的节点
>
> 关联： [Web](https://www.portainer.io)



# 其他

## Daemon 配置

> 对容器的 dockerd 守护线程进行配置
>
> 关联： [Configure the daemon](https://docs.docker.com/config/daemon/)

**dameon.json** 

配置文件编写：

源镜像地址配置

私有源非 Https 配置

Debug 模式开启

`/etc/docker/daemon.json`

ip: 永久绑定到某个固定的 IP 地址

bridge： 将 Docker 默认桥接到创建的网桥上

```json
{
  "registry-mirrors": [
    "https://dockerhub.azk8s.cn",
    "https://reg-mirror.qiniu.com"
  ],
  "insecure-registries": [
    "172.17.11.29:80",
    "172.17.11.29:5111",
    "192.168.205.23:80",
    "192.168.205.23:5111",
     "172.17.10.150:80"
  ],
  "debug": false,
  "dns" : [
    "114.114.114.114",
    "8.8.8.8"
  ],
  "ip": "0.0.0.0",
  "bridge": "bridge-my"
}
```

```shell
# 更改后使其生效
sudo systemctl daemon-reload
sudo systemctl restart docker.service
sudo systemctl status docker -l
sudo docker info
```



**设置运行时目录，存储驱动**



**设置 Http/Https 代理**

加快拉取国外访问、处理国内制作镜像无法访问国外资源问题

```shell
# 添加配置
mkdir -p /etc/systemd/system/docker.service.d
vim /etc/systemd/system/docker.service.d/http-proxy.conf

[Service]    
Environment="HTTP_PROXY=https://172.17.10.18:5720/" "NO_PROXY=localhost,127.0.0.1"

# 配置生效
systemctl daemon-reload
systemctl restart docker
systemctl show --property=Environment docker


# 重置
rm -f /etc/systemd/system/docker.service.d/http-proxy.conf
systemctl daemon-reload
systemctl restart docker
systemctl show --property=Environment docker
```

 [Docker systemd http-proxy](https://docs.docker.com/engine/admin/systemd/#/http-proxy) 




## 监控与管理

> 人工进行容器的管理、**监控**、资源调整、**故障排除**，包括日志查看、容器实时运行情况、资源重分配，在无法使用或没有监控方案情况下使用

**dockerd 支持**

> 在发生故障后，通过设置 Docker 守护线程的一些参数方便调试

```shell
# 开启守护线程的 debug 模式，给出更多的信息提示
dockerd --debug \
  --tls=true \
  --tlscert=/var/docker/server.pem \
  --tlskey=/var/docker/serverkey.pem \
  --host tcp://192.169.9.2:2376
```



**容器的运行日志查看**

使用 Go 模板尽心格式化日志输出

可使用日志驱动程序插件，企业版支持统一格式查看远程的日志，默认双重日志

[Format command and log output](https://docs.docker.com/config/formatting/)

```shell
docker container inspect --format '{{ .NetworkSettings.IPAddress }}' ${CID}
# 获取某个镜像对应的全部容器
docker container ls | grep <image> | awk '{print $1}
docker inspect -f '{{.HostConfig.LogConfig.Type}}' <CONTAINER>
docker logs -f <container_id>
# 通过 Go 的模板语法进行格式化展示数据
docker image ls --format "{{.ID}}: {{.Repository}}"
```



**容器日志**

> 在 daemon 中的日志配置，根据实际需要进行优化，可选择日志插件

指定容器日志最大大小                20M

最大的文件个数                            5

压缩，                                           开



**容器的运行情况**

> 通过 stats 实时查看容器的资源信息

```shell
# 实时查看容器统计信息，CPU、内存、网络、磁盘
docker stats 13b9203f9f0b d42877298134 44fb90cd2f2c
docker container stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```



**容器的资源分配**

> 容器使用多少宿主机的资源，可以通过 docker-compose.yml 中设置

```shell
# 通过参数限定容器访问内存、CPU
docker run --help | grep cpu 
docker run --help | grep memory
```



**容器可用性**

容器支持重启，可以通过 `--restart` 指定重启策略，保证可用性



**容器网络**

```shell
# 查看所有网络
docker network ls
# 查看容器网络映射
docker port nostalgic_morse 5000
```



**.dockerignore** 

针对非 SpringBoot 项目，如前端项目需要忽略一些文件。

```
.git
node_modules
npm-debug.log
```



**默认的重要文件：**

/var/run/docker.sock

/var/lib/docker/volumes/<volume_name OR long_uuid>




## **Docker 卸载**

```shell
# 停止并删除容器
docker rm -f `docker ps -aq`

# 删除安装
yum list installed|grep docker
 yum -y remove docker-ce.x86_64
 yum -y remove docker-ce-cli.x86_64
 yum -y remove containerd.io.x86_64
 
# 所有镜像、Volume删除
 # 删除 docker-compose
 rm -rf /var/lib/docker
 rm -rf /hdapp
 rm -rf /etc/docker
 rm -f /usr/local/bin/docker-compose
 
 # 测试卸载情况
 yum list installed|grep docker
 
 # 删除 docker0 网卡
 yum install bridge-utils
 ip link set dev docker0 down
 brctl delbr docker0
```



## 远程访问

```shell
iptables -I INPUT -p tcp --dport 3440 -j ACCEPT
```



[无法远程访问docker端口](https://www.jianshu.com/p/109474ee6d66)



# Refs

[Docker-guide](https://jiajially.gitbooks.io/dockerguide/content/dockerIND.html): 中文的 GitBook

[官方镜像示例](https://github.com/docker-library/docs)： Docker Hub 中一些镜像的开源示例

[play-with-docker](https://labs.play-with-docker.com/p/blga8164315g00e59gs0) ： 方便环境搭建， 保存 docker 4h，在网站上创建多个网络，进行互通访问

[10分钟看懂Docker和K8S](https://zhuanlan.zhihu.com/p/53260098)： 快速入门

[docker学习笔记](https://zhuyasen.com/post/docker_note.html)： 他人博客笔记

[Docker -- 从入门到实践](https://yeasy.gitbooks.io/docker_practice/)

[如何调试 Docker](https://yeasy.gitbooks.io/docker_practice/appendix/debug.html)： 总结调试方法

[关于对docker run --link的理解](https://www.jianshu.com/p/21d66ca6115e)： Docker 桥接网络理解

[工具和示例](https://yeasy.gitbooks.io/docker_practice/advanced_network/e)： Docker 相关工具

[Dockerfile 最佳实践](https://yeasy.gitbooks.io/docker_practice/appendix/best_practices.html#dockerfile-最佳实践)： 编写 Dockerfile 的一些建议 

[CentOS7.4—构建LVS+Keepalived高可用群集](https://blog.51cto.com/12227558/2096290)： LVS 

[docker镜像操作](https://www.centos.bz/2018/02/docker镜像操作/)： 容器制作、本地导入、镜像导出