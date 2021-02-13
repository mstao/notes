[TOC]



# Docker 部署总结

## Dockerfile 编写

**不在其中进行修改文件**

镜像修改会新增一个层，对于修改文件或目录权限，会将文件复制一份

==> 在 Dockerfile 之前设置好或是通 entrypoint 进行修改



**使用 dockerignore 文件**

执行 docker build 时忽略特定的路径和文件，加速构建过程




**合并多个RUN减少镜像层数**

 合并多个 RUN 成一个，因为每次 RUN 都是在原有的镜像上增加一层，可有效减少镜像中的层数

```dockerfile
FROM debian
RUN apt-get update \
  && apt-get -y install \
    openjdk-8-jdk ssh vim 
COPY target/app.jar /app
CMD ["java", "-jar", "/app/app.jar"]
```



**选取合适的基础镜像**

使用 alpine 架构的 Linux 一般生成的镜像比较小，根据需要选择可以满足运行条件的最小镜像，自己构建镜像，使用公司内部镜像，借助 Dockerfile 更改部分内容适配需求

```
node:10.16.3-alpin   74M
node:10.16.3         904M
node:10.16.3-stretch
node:10.16.3-bruster
```



**可重建性**

制作镜像时将源码打包进入镜像中，在镜像提供的环境中进行源码的编译打包，之后运行此种方式配合容器的挂载映射，实现每次运行时的配置都是最新的

```dockerfile
FROM maven:3.6-jdk-alpine
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn -e -B package
CMD ["java", "-jar", "/app/app.jar"]
```



**多阶段构建**

使用某个镜像作为构建环境，之后引用其中的文件，不会将该辅助镜像打包进最后生成的镜像中; 将编译和运行过程分离开，与通过 Jenkins 的 docker agent 进行编译镜像实现类似

方便控制构建环境的版本，不易出现版本问题

可以使用配有公司 Maven 私服地址的 Maven 镜像作为编译环境，之后在容器中编译打包，从构建包传递到 JDK 环境的运行包中，执行最终的构建

```dockerfile
# jar 的打包和运行
# As Builder 指定构建过程中的镜像
FROM maven:3.6-jdk-8-alpine As builder 
WORKDIR /app 
COPY pom.xml .
RUN mvn -e -B dependency:resolve 
COPY src ./src 
RUN mvn -e -B package 
# 实际运行的环境
FROM openjdk:8-jre-alpine 
# 从 Builder 中复制
COPY --from=builder /app/target/app.jar /
CMD ["java","-jar","/app.jar"]
```

```dockerfile
# Web 项目的打包和运行
FROM node:10.16.3 as builder
WORKDIR /usr/src/app/
USER root
COPY package.json ./
RUN npm install
COPY ../Doc ./
RUN npm run build:docker
FROM nginx
WORKDIR /usr/share/nginx/html/
COPY ./docker/nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /usr/src/app/dist  /usr/share/nginx/html/
EXPOSE 80
```



### 运行参数和动态控制

**COPY 实现自定义配置**

可以实现容器内配置的覆盖，一般配合 WORKDIR 使用

可以实现相对路径下的文件拷贝到容器中，存在时覆盖原有文件

可自动实现文件的重命名

NOTE: 

- 对于可执行文件，需要进行赋权
- 对于配置文件，特定组件需要特定的权限，如 MySQL 的配置文件

```dockerfile
# 工程文件复制到镜像
COPY ../devops ./
# 配置覆盖
COPY ./docker/nginx.conf /etc/nginx/conf.d/default.conf
# 当前目录下文件作为容器中的部分配置
COPY nginx.conf /etc/nginx/conf.d/default.conf
# 相对路径下的所有文件拷贝到指定目录，不包含该路径文件夹
COPY ./dist  /usr/share/nginx/html/
```

```dockerfile
# Dockerfile_rabbit3.6
FROM docker.elastic.co/logstash/logstash:7.2.0
# 别名指定
RUN alias ll='ls -l'
ADD logstash.yml /usr/share/logstash/config/logstash.yml
ADD jvm.options /usr/share/logstash/config/jvm.options
EXPOSE 9600
```



动态参数和配置

**RUN + echo 实现自定义配置**

通过 RUN 执行 shell 命令，将需要的配置插入到配置文件的尾部，从而覆盖之前编写的配置，实现自定义配置

只适合使用 key=val 方式的配置文件，对于 json 格式的配置文件无法使用

```dockerfile
FROM mysql:5.6
...
RUN echo "lower_case_table_names=1" >> /etc/mysql/mysql.conf.d/mysqld.cnf && \
    echo "max_allowed_packet=100M" >> /etc/mysql/mysql.conf.d/mysqld.cnf && \
    echo "innodb_log_file_size=500M" >> /etc/mysql/mysql.conf.d/mysqld.cnf && \
    echo "wait_timeout=57600" >> /etc/mysql/mysql.conf.d/mysqld.cnf && \
    echo "interactive_timeout=57600" >> /etc/mysql/mysql.conf.d/mysqld.cnf && \
	echo "transaction_isolation=READ-COMMITTED" >> /etc/mysql/mysql.conf.d/mysqld.cnf && \
    echo "event_scheduler=1" >> /etc/mysql/mysql.conf.d/mysqld.cnf && \
    echo "max_connections=500" >> /etc/mysql/mysql.conf.d/mysqld.cnf && \
    echo "innodb_lock_wait_timeout=500" >> /etc/mysql/mysql.conf.d/mysqld.cnf
```



**COPY + Shell + Sed + ENTRYPOINT 自定义配置**

将带有一定模板值的多个配置文件 COPY 进镜像中，将通过 sed 进行配置替换的脚本 COPY 进镜像，将配置替换脚本作为 ENTRYPOINT;

即使配置文件对应的应用不支持传入变量值和环境变量也可以通过此种方式使其支持;

实际使用：

运行在 Nginx 容器中的 web 项目，动态替换框架打包的 umi 中的配置，实现环境的指定；

运行在 tomcat 容器中的 war Java 项目，在运行启动后，对应的 war 包的内容被解压开来，更改配置文件的位置，一般是 xx.properties、xxxx.yml，之后重启整个容器应用更改 



**ENV 指定实现特定功能**

三种语法

配合 Python 脚本将传入的环境变量进行解析成 Ansible 连接的信息

运行时指定环境变量实现动态运行控制

```dockerfile
# 指定路径、版本...
ENV MAVEN_HOME=/apache-maven-3.3.9
ENV MYSQL_VERSION 5.6.45-1debian9
ENV NODE_VERSION 8.9.4
# 指定系统环境变量
ENV PATH ${PATH}:${MAVEN_HOME}/bin
# 指定一组环境变量参数
ENV ANSIBLE_HOSTS="test1:127.0.0.1" \
    ANSIBLE_PORT="22" \
    ANSIBLE_USER="root" \
    ANSIBLE_PASSWORD="123456" \
    ANSIBLE_FILE="centos_init_test"
```



**多用途构建文件拆分**

通过 Dockerfile、Dockerfile.hub、Dockerfile.dev、Dockerfile.in 增加文件后缀方式区分构建的用途

通过` Dockerfile_<iamge_name><image_version>_<env>_for_<gole>` 指明所属环境版本、目的、特定版本

```
Dockerfile  Dockerfile.hub、Dockerfile.dev、Dockerfile.in
Dockerfile_alpine_3.5
Dockerfile_mysql5.6_v6
Dockerfile_hd_tomcat8539-jre811-for-prod-qianfan_v1
```



**配合构建工具实现一键发布**

对于 Maven 项目，通过 docker-maven-plugin 实现自定义选择阶段，只需在命令后增加 `docker:build`， `docker:push` 即可，同时可以将阶段绑定到对应 mvn 命令，如 package、deploy

对于 Node 项目，可以自定义 npm run 命令，在 package.json 中定义好 scripts 指定好某个命令的执行指令，并支持多个命令的合并运行，命令之间顺序任意，以此方便项目的 CI

```json
"docker-hub:build": Dockerfile_jenkins_jdk8-maven3.6.1,
"docker:tag": "docker tag ant-design-pro harborka.qianfan123.com/iwms/iwms-web",
"docker-registry:login":  "docker login harborka.qianfan123.com -u zhangsai -pzs1234ZS",
"docker:push": "npm run docker-hub:build && npm run docker:tag && npm run docker-registry:login && docker push harborka.qianfan123.com/iwms/iwms-web",
```

```shell
mvn deploy
mvn clean package docker:push -Pdocker
```



**ENTRYPOINT + CMD 参数填充**

指定命令参数覆盖 -?，实现指定参数的运行，配合特定运行 jar 包随着 main 类传入参数定制容器的行为

适合 "一次性命令"，启动的容器目的只是依赖于容器内部的工具执行特定的脚本，rdbver 实现数据库的安装、redis-trib 构建集群便是此种方式

```dockerfile
ENTRYPOINT ["java","-jar","setup.jar"] 
CMD ["-?"]
```

```shell
docker run --rm harborka.qianfan123.com/iwms/account-db-setup \
all -d "jdbc:mysql://${MYSQL_ADDR}:3306/iwmsaccount?useUnicode=true&characterEncoding=utf8" -u root -p Heading123iwms --skip-version-check
docker run -it --rm \
--net host \
harborka.qianfan123.com/iwms/redis-trib \
ruby redis-trib.rb create 172.17.10.133:6379 172.17.10.137:6379 172.17.10.137:6381
```



**ENTRYPOINT + docker-entrypoint.sh + 环境变量实现定制化运行**

在 docker-entrypoint.sh 中指定多种后缀类型的文件执行不同的处理

通过执行一个初始化脚本，对于 MySQL 环境初始项目需要使用的数据库，并实现权限的分配，当前 iwms 中 rdbver 不负责创建数据库

```dockerfile
COPY docker-entrypoint.sh /opt/app/
RUN chmod +x /opt/app/docker-entrypoint.sh
ENTRYPOINT ["sh", "/opt/app/docker-entrypoint.sh"]
```



[docker:mysql启动时自动执行初始建表脚本](https://blog.csdn.net/10km/article/details/79046864)： 通过 sh +2·1·1346·5·2 sql 的方式

[让docker中的mysql启动时自动执行sql](https://www.cnblogs.com/han-1034683568/p/6941337.html):  使用免密等了方式




## Docker Compose 编写

**相对路径文件绑定**

默认情况下不会对宿主机的文件进行相对路径绑定，仅能够实现相对路径为 /var/... 某个路径下的相对，在高版本中通过指定绑定的类型支持相对路径文件绑定，实现配置的映射

支持配置是否只读，如不可更改的配置文件，对于存储的数据需要写权限，可指定多个 bind、volume 类型的绑定

[deviantony/docker-elk](https://github.com/deviantony/docker-elk/blob/master/docker-compose.yml)

```yaml
elasticsearch:
  build:
    context: elasticsearch/
    args:
      ELK_VERSION: $ELK_VERSION
  volumes:
    - type: bind
      source: ./elasticsearch/config/elasticsearch.yml
      target: /usr/share/elasticsearch/config/elasticsearch.yml
      read_only: true
    - type: volume
      source: elasticsearch
      target: /usr/share/elasticsearch/data
```



**日志配置**

配置每个日志文件的最大大小，以及最多拥有的配置文件个数，文件个数由对应的日志框架配置的规则生成

```yaml
logging:
  options:
  max-size: "10m"
  max-file: "10"
```



**多阶段配置**

通过 `docker-compose.yml`  和 `docker-compose.dev.yml` 后缀来区分阶段

[ant-design-pro]() 和 [elk]() ... 项目采取这种方式



**指定拆分网络**

自己定义的网络有时会被自动加上当前用户名，可直接指定名称

方便以后的部署升级，停止某个服务，自己手动指定命令指定网络为某个命名网络，实现通信

对项目进行前后端网络的分离，避免某个网络容器数过多不好管理

```
networks:
  front:
  backend:
  eureak-net1:
    name: eureka-net1
    driver: bridge
  default:
    name: eureka
```



**环境类参数指定**

在 Dockerfile 中通过 `ARG ELK_VERSION` 指定参数，之后在 docker-compose.yml 中指定具体的值

通过在 docker-compose.yml 同路径下定义 .env 中定义环境变量，自动将值注入进 docker-compose.yml

```dockerfile
ARG ELK_VERSION
# https://github.com/elastic/logstash-docker
FROM docker.elastic.co/logstash/logstash:${ELK_VERSION}
```

```yaml
ogstash:
    build:
      context: logstash/
      args:
        ELK_VERSION: $ELK_VERSION
    volumes:
      - type: bind
        source: ./logstash/config/logstash.yml
        target: /usr/share/logstash/config/logstash.yml
        read_only: true
      - type: bind
        source: ./logstash/pipeline
        target: /usr/share/logstash/pipeline
        read_only: true
    environment:
      LS_JAVA_OPTS: "-Xmx256m -Xms256m"
```

环境变量来源方式:

1、Linux 系统自带的环境变量，包含通过 `export <env_name>=<env_value`  方式指定的环境变量

2、Dockerfile 中指定的参数， `ENV <env_name> <env_value`

3、在 docker-compose.yml 配置中明文写明的变量

4、默认在 docker-compose.yml 同级目录下的 .env 文件中的内

5、通过 env_file 指定的文件内容

6、通过 docker secret 进行加密后的 secret





**运行时参数指定**

一般是运行在容器内部的进程支持的运行参数，如 Java 运行参数

```dockerfile
ENTRYPOINT ["java","-jar","setup.jar"] 
CMD ["-?"]
```

默认执行的是 java -jar setup.jar -?

运行如下命令后，自动覆盖 -? 参数

```shell
docker run <contianer> all -d "jdbc:mysql://172.17.10.127:3306/iwmsaccount?useUnicode=true&characterEncoding=utf8" -u root -p Heading123iwms --skip-version-check
```



## Docker-Compose 部署

> 在单台机器上使用 Docker Compose 管理多个容器，包含服务的网络、目录挂载、服务

**配置检查**

检查配置的语法问题，以及将环境变量注入进配置文件中，进行预览调试，配置较多的时候执行防止低级错误

```shell
docker-compose config
```



**环境变量控制**

1、注入环境变量的方式

- 直接写入，在 docker-compose.yml 中直接配置
- 在 docker-compose.yml 同级目录下新建 `.env` 文件，在其中定义好需要的环境变量，以 piggyMetrics 中环境变量为例
- 通过 env_file 手动指定环境变量位置，可指定多个文件

```
CONFIG_SERVICE_PASSWORD=123456
NOTIFICATION_SERVICE_PASSWORD=123456
STATISTICS_SERVICE_PASSWORD=123456
ACCOUNT_SERVICE_PASSWORD=123456
MONGODB_PASSWORD=123456
```

2、不同方式下的优先级

Compose file ==>> Shell env var > env file > Dockerfile

3、Shell 中引用

在编写的 shell 脚本中使用 .env 中定义的变量，实现配置的替换等定制操作

```shell
source .env
```



**分阶段执行**

将 docker-compose 文件拆分成开发和实际运行，通过 docker-compose.dev.yml 和 docker-compose.yml 进行区分

先 build，从远程拉取出镜像、或是根据 Dockerfile 构建镜像；之后启动

```shell
docker-compose build
docker-compose up -d
```



**继承方式运行**

```shell
# docker-compose 之间 "继承" 关系操作
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up
```



**扩展 service 容器数量**

前提是没有设置容器名、设置的网络不是 host 网络

```shell
docker-compose -d --scale web=3,db=2 up
```



## 镜像构建打包发布

一键构建发布

通过 node 命令合并，maven 插件集成，zsh 终端别名命令定制，Jenkins CICD 操作 Shell 脚本



**镜像构建**

1、构建方式

(1) Dockerfile 一键构建，构建环境为当前运行环境

(2) Dockerfile + Builder 容器进行构建，构建环境为 Builder 指定的容器环境

(3) Jenkins + Agent 配合数据卷容器构建，构建环境为agent 指定的或是新构建的容器环境



2、常见构建目标

(1) 中间件镜像构建，对于 redis、rabbitmq、nginx、haproxy、mysql 对外提供服务，需要更改启动方式为非后台启动，在配置中或是命令运行时进行更改处理 

(2) 运行环境构建，Centos、Debian、Alpine 操作系统，JRE 、Tomcat、Nginx、Ruby、MySql 程序运行环境

(3) 构建工具构建，Maven、Node、Gradle 等工具镜像构建



3、镜像构建与 Git 配合

Docker 提供环境、Git 仓库提供最新的部署脚本，entrypoint 中指定运行特定的脚本(python)

```shell
git clone <git_url> -b <branch>
chmod 777 <git_master>/docker-entrypoint.sh
docker build -t <> .
docker push <>
```





**镜像打包**

`日期 + <short_commid_id>`: 持续交付方式的镜像打包，保留以往的

`<mvn_version>`: mvn 版本，不断的覆盖，随模块版本稳定而稳定

```
0.0.0-YYmmddHHMMSS-abddfd
```



## 容器运行与调试

### 运行

**容器运行**

(1) 命令式环境扩展，在容器运行后增加特定的命令，辅助镜像 *外部*  环境构建，适合有限几条命令的工具类容器

(2) 配置文件映射 + 环境变量定制，在配置文件支持环境变量映射的情况下使用，实现配置的定制化运行

(3) 内嵌脚本 + ENTRYPOINT|CMD，制作镜像时内置可运行脚本，辅助镜像 *内部* 环境构建

(4) 资源限制，通过 `--cpuset-cpus` 等参数限制容器使用的 cpu、内存，适合在宿主机上运行多个不同功能的容器防止容器争夺宿主机资源造成宕机时使用，默认情况下使用的 cpu、内存硬件资源都是宿主机的全部



**数据卷容器**

创建数据卷容器专门用于其他容器进行挂载，其他容器通过 `--volumes-from` 参数进行挂载，，实现多个容器之间的数据共享，如 Jenkins 中 agent 为 docker 容器、Dockerfile 中从构建容器中 COPY 数据

```shell
# 运行 Jenkins 容器
docker run -d --restart always --name iwms-jenkins\
  -p 8111:8080 \
  -p 50000:50000 \
  -e JAVA_OPTS="-XX:MaxMetaspaceSize=256m -Xms256m -Xmx2048m" \
  -v /hdapp/jenkins:/hdapp \
  -v /hdapp/tools:/tools \
  -v /root/.ssh:/root/.ssh \
  -v $(which docker):/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  harbor.qianfan123.com/jenkins/master:2.164.3-jdk8
```

```groovy
// Jenkins pipeline 中定义的 agent
stage('编译 & 打包 & 发布') {
  agent {
    docker {
      image 'harbor.qianfan123.com/base/maven:3.3.9_jdk8'
      args '-v /var/run/docker.sock:/var/run/docker.sock -v /hdapp/java/.m2:/hdapp/java/.m2'
      reuseNode true
    }
  }
  steps {
    echo "开始编译项目"
    sh "mvn clean install -Dmaven.skip.test=true"
  }
}
```

```shell
# Jenkins 执行过程中的日志输出
# 22fbb383a85cf8c4753cc60a2f6e3aa92eac3f38d3b6a7b6d2f1766965b02575: 容器的 ID
# VOLUME $JENKINS_HOME: 为制作 jenkins 的 Dockerfile 指定的, 为 /hdapp
# -w /hdapp/workspace/iwms-pom: 自动将当前 Jenkins 从 git 拉取的项目位置
docker run -t -d -u 0:0 -v /var/run/docker.sock:/var/run/docker.sock -v /hdapp/java/.m2:/hdapp/java/.m2 -w /hdapp/workspace/iwms-pom --volumes-from 22fbb383a85cf8c4753cc60a2f6e3aa92eac3f38d3b6a7b6d2f1766965b02575 -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** 
```



**宿主机配置脚本执行**

方式一： 在构建 Dockerfile 的时候将脚本 COPY 到 镜像之中，使用 RUN 或 ENTRYPOINT 执行

可用于指定初始化脚本，指定环境变量控制初始化逻辑

方式二： 在容器未启动时，通过 `--entrypoint` `-v` `--workspace` 参数进行数据映射，并执行入口点的指定位置的脚本

方式三： 容器已经运行，通过 `docker cp ./shell.sh <container>:/usr/local/run-shell`，之后运行 `docker exec -it <container> bash run-shell`即可在容器内部执行命令



[数据卷容器](https://jenkins.io/doc/book/pipeline/syntax/)



### 调试

**进入无 pid=1 的镜像**

对于编译环境内的容器，启动后直接退出，在没有对应 Dockerfile 的情况下，通过执行一个无线循环方便进入容器

为方便调试，加上 `--rm` 参数，停止后自动删除容器，适合多次进入容器内部查看情况

为方便调试，不后台运行，时候运行程序进行简单的运行情况查看

```shell
docker run -dit --rm --name test-maven harbor.qianfan123.com/base/maven:3.3.9_jdk8 /bin/sh -c "while true; do sleep 3600;done"
# 查看运行情况
docker run -it harborka.qianfan123.com/iwms/account-db-setup all -d "...."...
```



**使用控制台工具**

1、Dokcer 自带的 CLI 

```shell
docker exec
docker logs -f <container_name>
docker stats <container_name>
docker inspect -f <go_template> <container_name>
```

2、第三方命令行工具

```shell
lazydocker
```



**文件 COPY**

从宿主机拷贝到容器中，从容器中拷贝到宿主机，容器未运行的情况下同样支持

对于容器内部不支持 vim 编辑时，可以使用此种方式进行文件的覆盖，作为补偿

```shell
docker cp test-mysql-off:/var/lib/mysql ./
```



**图形化工具**

Rancher、


## 常见问题(#)

Mysql 容器运行必须制定 MYSQL_ROOT_PASSWORD

制作 Redis 镜像，需要将配置文件的 daemonize 改为 no

制作 Nginx 镜像，需要将 nginx 改为非后台启动



**后台进程与闪退问题**

容器启动检测 pid 为 1 的进程情况，该进程没有停止则容器一直运行，对于一些服务性质的组件，需要更改其启动方式为非后台启动，如 nginx、redis

```shell
CMD ["mysqld"]
CMD ["nginx", "-g", "daemon off;"]
# 配置 + 命令
CMD ["redis-server", "user/local/etc/redis/redis.conf"]
```



**启动顺序问题**

容器启动完毕不代表内部的进程启动完毕，如 MySQL、RabbitMQ 启动后需要一定的时间加载服务，在没有初始化之前连接都是拒绝的，使用一些自动化工具、搭建集群的时候需要注意;

可以给对应的依赖容器一定的循环尝试重启策略处理该问题



**配置文件目录挂载**

在不了解镜像中配置文件目录的情况下最好不要随意挂载，存在配置覆盖问题，以及重启后挂载目录得不到清理



**Permission denied**

`cannot open directory .: Permission denied`， setlinux 问题....

方式一: 对于 Centos 系统关闭 setlinux ，选择临时或永久关闭，选择修改 setlinux 规则

方式二: docker run 指定 `--privileged=true` 



**字符集问题**

对于以 alpine 为基础制作的镜像，对应支持的字符集需要在制作其他镜像的时候加入，只有镜像中有该字符集才可以使用



**挂载容器中的目录不存在**

对于使用 war 包部署的项目，在 tomcat 没有运行前对应的目录不存在，启动后快速失败



**通过 `RUN mkdir -p <path>` 方式创建并 `VOLUME` 无法在对应目录下创建文件**









[使用 Docker 过程中曾经掉过的坑](https://www.jianshu.com/p/3bcb0fe9ed28)

[docker volume 容器卷的那些事（一）](https://www.jianshu.com/p/dd2fecfd8edf)

[Docker Volume - 目录挂载以及文件共享](https://kebingzao.com/2019/02/25/docker-volume/)



# Docker 部署

## Docker swarm 部署

> 在 Swarm Cluster 部署多个 service，需要搭建 Swarm Cluster 集群，构建 overlay 网络实现多个容器之间的通讯

**Service 部署 wordpress**

在 Swarm  Cluster 中部署单个 Service

以往一： mysql 容器 + wordpress 容器

两个不同容器之间的通信问题，通过 overlay 网络进行通信，不论是否在同一台机器上都可以访问

输入对应其他机器的地址，实现 manager node, worker node 集群上都可以访问

最佳同步网络，只要需要的时候同步网络，在某个 node 上没有时，无需进行同步网络，即无需在该机器上创建 dockker 的 overlay 网络

swarm 底层实现 overlay 网络通信，无需外置 key-val 存储，已有内置的

在指定了 port 后，只能够在不同机器上部署

```shell
# 创建 overlay 的网络
# 创建对应的 mysql 服务
#   指定service 名称
#   指定MYSQL的密码
#   指定Mysql的数据库名
#   指定 service 的网络
#   指定挂载： 类似本地运行docker 的 -v 挂载参数 (#)
#     type: 挂载的类型
#     source: 挂载本地的 volume 的名称
#     destination: 容器内部的文件夹地址
# 创建对应的 wordpress 服务
#   指定service名称
#   进行宿主机端口映射
#   指定 wordpress 数据库密码
#   指定 wordpress 数据库 host
#   指定 service 的网络
docker network create -d overlay net-demo
docker network ls
docker service create --name mysql \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=wordpress \
  --network net-demo \
  --mount type=volume,source=mysql-data,destination=/var/lib/mysql mysql
docker service create --name wordpress \
  -p 80:80 \
  -e WORDPRESS_DB_PASSWORD=root \
  -e WORDPRESS_DB_HOST=mysql  \
  --network net-demo wordpress
```



**Stack 部署 wordpress**

进行多个服务的应用构建

部署情况验证： 通过访问任意一个 cluster 中的地址即可访问

```shell
# 更改服务名
# 整个 application 定义为一个 stack 为 wordpress
# 查看运行情况
#   mysql: 限制只运行一个，只能运行在 manager 节点
# 通过 stack 查看服务情况
docker stack deploy wordpress --compose-file=docker-compose.yml
docker stack ls
docker stack ps wordpress
docker stack services wordpress
docker stack rm wordpress
```



**Stack 部署 Vote**

进行多个服务的应用构建，增加可视化服务，方便查看 Swarm Cluster 中的容器情况

指定 image 从本地获取镜像或从 registry 拉取镜像

网络端口： 需要与宿主机端口的映射

networks: 指定 frontend ，特定需要 frontend, backend

depends_on： 指定依赖的服务，外部中间件服务或者内部业务服务， redis, db

deploy： 可复制实例增加，更新的配置，重试策略条件

  限制部署限制在 manager



增加 visulizer 服务，可以对 swarm 进行查看容器，有多少，什么情况，镜像:  dockersamples/visualizer:stable

```shell
docker stack deploy example --compose-file docker-compose.yml
docker stack ls
docker stack services example
# 通过 stack 名 + 对应服务名方式进行扩展
docker services scale example_vote=3
docker stack rm example
```



# 监控工具

## lazydocker

> 本地的监控容器运行情况的命令行工具，可实时查看容器日志、运行情况
>
> 关联： [GitHub](https://github.com/jesseduffield/lazydocker)     

```shell
# 官方包
rz
tar -xvf lazydocker_0.7.4_Linux_x86_64.tar.gz 
mv lazydocker /usr/local/bin/
rm lazydocker_0.7.4_Linux_x86_64.tar.gz 
```


## portainer
> 可视化 Docker 管理工具，支持 Docker Swarm，轻量级，配合对应的 Edge Agent 收集多台机器的信息


[使用Portainer集中管理多地域内外网运行的Docker实例](https://www.iamle.com/archives/2727.html)

## Rancher
> 较为强大的 Docker 管理工具, 支持部署 k8s



## ELK 日志..

[ELK-docker](https://github.com/sqshq/ELK-docker)： Elasticsearch 进行日志分析， Docker 部署



# 其他

## 运行环境问题

### Java 环境

针对 Java 项目当前已知三种方式进行环境区分：

**1、Spring 的 profile**

更细致的是 Condition + 各种判断控制，控制较为灵活

```shell
application-dev.properties
application-test.properties
```

**2、Maven 的 Profile**

指定 Maven 的 resource 为不同的目录，仅能够支持配置文件和一些 Maven 插件配置

```
resource  # 所有的通用配置
resource.dev  # 开发配置
resource.test  # 测试配置
resource.prod  # 生产环境配置
resource.docker # docker ... 可等价于 生产环境
```

**3、Git 的分支控制**

更改分支上的配置内容实现环境的拆分，可以实现非配置文件的更改，直接修改代码

```
git checkout -b prod
git checkout -b dev
git checkout -b test
git checkout -b mock
git checkout -b docker
```

**4、Jenkins 构建变量**

在 Jenkins 中添加条件变量，在每次构建的时候选择需要构建的环境，配合 Shell 脚本将源码中的文件进行替换，实现和 Git 分支控制一样的效果

如在 WEB-INFO 下的 db.properties，无法通过 上述1、2处理，可以在对应目录下再建一个 db.prod.properties，Jenkins 判断为 prod 环境时，对 db.properties 删除，db.prod.properties 进行重命名即可





### 前端环境

在 package.json 中指定 scripts ，控制某个run指令执行特定环境的配置

```shell
npm run start
npm run start:test
npm run build
npm run build:docker
```



## 环境模拟

方式一： vagrant 虚拟机模拟

```shell
# cluster 宣告的地址，作为 swarm manager 节点
# 执行成功后返回对应 work node 加入的命令，在另一台机器上执行该命令
# 显示当前 swarm 中的节点
docker swarm init --advertise-addr=192.169.xx.xx
docker swarm join --token xxxfsdfsdf <ip>:<port>
docker node ls 
```

方式二： docker-machine 模拟

```shell
docker-machine create swarm-manager
docker-machine create swarm-manager2
docker-machine create swarm-manager3
docker-machine ssh swarm-manager
```

方式三： [play-with-docker](https://labs.play-with-docker.com/p/blga8164315g00e59gs0) 网站，4 h

方式四： 多台安装了 Linux 系统的机器

​    废弃电脑安装、树莓派安装、内网服务器安装、多台线上服务器

方式五:  图形化界面使用 Vmware、Virtual Box



# 运维

容器与宿主机的 UID问题

容器的共享目录实现



## 镜像构建

```shell
docker commit -m “<description” -a “zhangjiang <zhangjiang@hd123.com>” <container_id> <image_name>:<image_tag>
docker commit <id> temp
```



## 参数传递

通过在 Dockerfile 中运行程序时指定参数

通过 Dockerfile 中定义的环境变量，配合文件或明文传输环境变量实现执行，命令 `docker run --env-file`

通过运行程序默认的环境变量指定覆盖实现， 如 Java 的 SPRING_PROFILES_ACTIVE

```shell
ENTRYPOINT ["sh","-c","java $PARAMS -jar server.jar --spring.profiles.active=$PROFILES "]
docker run -d -p 8080:80 --name xxxx -e PARAMS="myParams" -e PROFILES="myProfiles" static_web
```



## 数据持久化

docker commit 不会将数据卷中的内容 commit 到镜像中

Docker 设计是无状态的



TT

```
docker system prune
mv /var/lib/docker /root/data/docker
ln -s /root/data/docker /var/lib/docker
```

[Spring Boot with Docker](https://spring.io/guides/gs/spring-boot-docker/?spm=a2c4e.10696291.0.0.36b419a4li6Wxi)

[部署PiggyMetrics分布式微服务](https://www.cnblogs.com/lyhero11/p/8966498.html)

[PiggyMetrics项目分析(1)-生产模式下部署应用](https://cloverat.github.io/2018/07/10/项目分析/[项目分析]PiggyMetrics项目分析(1)/)

[Intro Guide to Dockerfile Best Practices](https://blog.docker.com/2019/07/intro-guide-to-dockerfile-best-practices/)： 官方文档中对于 Dockerfile 的最佳实践总结

[使用 Docker 搭建 Tomcat 运行环境](https://www.centos.bz/2018/01/使用-docker-搭建-tomcat-运行环境/)： 从 centos + java + tomcat 逐步制作、配置环境变量

[java – 将参数传递给docker入口点](https://codeday.me/bug/20190505/1056680.html)： 命令格式， ENTRYPOINT + CMD

[Docker-compose命令详解](https://blog.csdn.net/wanghailong041/article/details/52162293)： 常见的命令

[springboot之docker启动参数传递](https://www.cnblogs.com/bigben0123/p/8875573.html)： 通过环境变量方式注入参数

[docker学习笔记-启动镜像输入参数](https://juejin.im/post/5b405c96e51d45194a51f842)： ...

[Docker Volume - 目录挂载以及文件共享](https://kebingzao.com/2019/02/25/docker-volume/)