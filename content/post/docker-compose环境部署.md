---
title: "docker-compose部署环境"
date: 2023-09-15T20:37:56+08:00
lastmod: 2023-09-15T20:37:56+08:00
draft: false
tags: ["docker", "mysql", "kafka"]
categories: ["笔记"]
author: "dcq"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'

---

> <span style="display:block;text-align:center;color:orangered;">**部署<font face="Times New Roman">Docker</font>环境注意**</span>
> 二维码发送端只需要安装<font face="Times New Roman">Redis,MySql</font>, 接收端安装<font face="Times New Roman">Redis,MySql,Rabbitmq,Canal</font>
> 配置文件都从192.168.1.178：/data目录拷贝下来，解压放到根目录下面

### 1.<font face="Times New Roman">Docker</font>

```sh
$sudo apt insall docker.io
#基本命令如下
$sudo docker ps -a  #查看容器运行
$sudo docker stop <CONTAINER ID> #停止容器
$sudo docker rm <CONTAINER ID>  #删除容器
$sudo docker logs -f <CONTAINER ID>
$sudo docker rmi 镜像ID #删除镜像
$sudo docker images #查看镜像
$sudo exec -it CONTAINER ID 或者--name 指定容器名 #进入docker 容器
$sudo exit #退出docker
$sudo docker stats #查看容器占用资源
$sudo docker save -o images.tar image1 image2 #多个镜像一起打包
$sudo docker load < images.tar #加载镜像
```

### 2.<font face="Times New Roman">Redis</font>

```sh
 $sudo docker pull redis
 $sudo mkdir -p /data/redis/{conf,data}
 $sudo vim /etc/redis/conf/redis.conf
 #加入如下两行
 protected-mode no
 appendonly yes
 $wq #保存
 $sudo docker run -p 6379:6379 --name redis \
  -v /data/redis/conf/redis.conf:/etc/redis/redis.conf  \
  -v /data/redis/data:/data -d redis redis-server /etc/redis/redis.conf  
```

### 3.<font face="Times New Roman">Rabbitmq</font>

```sh
$sudo mkdir -p /data/rabbitmq/data
$sudo docker pull rabbitmq:management
$sudo docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 \
 -v /data/rabbitmq/data:/var/lib/rabbitmq --hostname rabbitmq \
 -e RABBITMQ_DEFAULT_VHOST=/ -e RABBITMQ_DEFAULT_USER=admin  \
 -e RABBITMQ_DEFAULT_PASS=admin rabbitmq:management
#登陆http://ip:15672/
#用户名：admin
#密码：admin
#创建exchange: canal_exchange
#创建queue: canal_queue
#点击canal_queue: bind canal_exchange 和 canal_key
```

### 4.<font face="Times New Roman">MySql</font>

```sh
$sudo mkdir -p /data/mysql/{data,conf,logs}
$sudo docker pull mysql:5.7
$sudo docker run -d -p 3306:3306 --name mysql-service \
 -e MYSQL_ROOT_PASSWORD=root  mysql:5.7
$sudo docker exec -it mysql-service bash
$mysql --help|grep my.cnf #查找配置文件
#/etc/mysql/mysql.conf.d
$sudo docker cp mysql-service:/etc/msyql/msyql.conf.d/msyqld.conf /data/mysql/conf/ #拷贝容器配置文件到宿主机
$sudo docker stop <CONTAINER ID>
$sudo docker rm <CONTAINER ID>
#以下映射本地配置到docker容器
$sudo docker run -p 3307:3306 --name mysql \
 -v /data/mysql/conf/mysqld.cnf:/etc/mysql/mysql.conf.d/mysqld.cnf \ 
 -v /data/mysql/logs:/logs -v /data/mysql/data:/mysql_data \
 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```

### 5.<font face="Times New Roman">Canal</font>

```sh
$sudo mkdir -p /data/canal/conf
$sudo mkdir -p /data/canal/example
$sudo docker pull canal/canal-server
$sudo docker run --name canal-service  --rm -d canal/canal-server:latest
$sudo docker cp canal-server:/home/admin/canal-server/conf/canal.properties  /data/canal/conf   #从容器复制到宿主机
$sudo docker cp canal-server:/home/admin/canal-server/conf/example/instance.properties /data/canal/conf/example # 同上
#修改canal.properties文件

$sudo vim /data/canal/conf/canal.properties
# tcp, kafka, rocketMQ, rabbitMQ, pulsarMQ
canal.serverMode =rabbitMQ  
# rabbitmq配置
rabbitmq.host =192.168.1.178
rabbitmq.virtual.host =/    
rabbitmq.exchange =canal_exchange  #同管理界面创建
rabbitmq.username =admin           
rabbitmq.password =admin
rabbitmq.deliveryMode =direct

# 修改example.properties配置
$sudo vim /data/canal/conf/example/instance.properties 
canal.instance.master.address=192.168.1.178:3307  #摆渡机接收端数据库地址
canal.instance.master.journal.name=mysql-bin.000003  #binlog文件
canal.instance.master.position=922123  #binlog偏移位置，具体参见7，8点命令
canal.instance.dbUsername=admin
canal.instance.dbPassword=123456
canal.instance.filter.regex=qr_recv.recv_data_tb #监听那个表
canal.mq.topic=canal_key #同管理平台bind
# rm /canal/config/meta.dat



$sudo docker run --name canal -p 11111:11111 \
 -v /data/canal/conf/example/instance.properties:/home/admin/canal-server/conf/example/instance.properties \
 -v /data/canal/conf/canal.properties:/home/admin/canal-server/conf/canal.properties \
 -v /data/canal/logs/:/home/admin/canal-server/logs/ -d canal/canal-server:latest
```

### 6.非<font face="Times New Roman">Docker</font>版本<font face="Times New Roman">MySql</font>设置

```sh
$vim /etc/mysql/mysql.conf.d/mysqld.cnf
增加配置：
log_bin=mysql-bin
binlog_format=ROW
server_id=1
expire_logs_days=7
$wq
$/etc/init.d/mysql restart
$mysql -u admin -p
```

### 7.是否开启日志

```sh
$show variables like '%log_bin%';
```

### 8.查看日志文件及偏移量

```sh
$SHOW MASTER STATUS;
```

### 9.<font face="Times New Roman">Docker</font> 容器开机自启

* 开启自启
  
  ```sh
  $sudo docker update --restart=always <CONTAINER ID>
  ```

* 取消自启
  
  ```sh
  $sudo docker update --restart=no <CONTAINER ID>
  ```
  
  ### 10.<font face="Times New Roman">Docker</font>桥接
  
  - Bridge：桥接网络。默认情况下启动的Docker容器，都是使用Bridge，Docker安装时创建的桥接网络，每次Docker容器重启时，会按照顺序获取对应的IP地址，这个就导致重启下，Docker的IP地址就变了。
  
  - None：无指定网络。使用 --network=none，Docker容器就不会分配局域网的IP。
  
  - Host：主机网络。使用--network=host，此时，Docker容器的网络会附属在主机上，两者是互通的。例如，在容器中运行一个Web服务，监听8080端口，则主机的8080端口就会自动映射到容器中。
    
    ```sh
    $sudo docker network create --subnet=172.18.0.0/16 mynetwork
    ```

### 11.<font face="Times New Roman">Kafka</font>

```sh
$sudo docker pull wurstmeister/zookeeper
$sudo docker pull wurstmeister/kafka
$sudo docker pull sheepkiller/kafka-manager
$sudo mkdir -p /data/zookeeper/{data, log}
$sudo docker run -d --name zookeeper -p 2181:2181 
\ -v /data/zookeeper/data:/data -v /data/zookeeper/log:/datalog zookeeper
$sudo docker exec -it zookeeper bash
$cd bin
$bash zkCli.sh
$ls /
$ls /brokers
#启动kafka管理，添加集群
$sudo docker run -d --name kafka-manager \
--link zookeeper:zookeeper \
--link kafka:kafka -p 9001:9000 \
--restart=always \
--env ZK_HOSTS=zookeeper:2181 \
sheepkiller/kafka-manager
$sudo docker run -d --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=0
\  -e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 --link zookeeper
\ -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.1.178:9092
\ -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -t wurstmeister/kafka
$sudo mkdir -p /data/kafka/logs
#单机启动
$sudo docker run -d --name kafka --publish 9092:9092 \
   --link zookeeper \
   --env KAFKA_ZOOKEEPER_CONNECT=192.168.200.178:2181 \
   --env KAFKA_ADVERTISED_HOST_NAME=192.168.200.178 \
   --env KAFKA_ADVERTISED_PORT=9092  \
   --env KAFKA_LOG_DIRS=/kafka/kafka-logs-1 \
   -v /data/kafka/logs:/kafka/kafka-logs-1  \
   wurstmeister/kafka


$sudo chmod 666 /var/run/docker.sock  #启动报权限问题解决
#-e KAFKA_BROKER_ID=0 在kafka集群中，每个kafka都有一个BROKER_ID来区分自己
#-e KAFKA_ZOOKEEPER_CONNECT=10.9.44.11:2181    /kafka 配置zookeeper管理kafka的路径10.9.44.11:2181/kafka
#-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://10.9.44.11:9092    把kafka的地址端口注册给zookeeper
#-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 配置kafka的监听端口
#-v /etc/localtime:/etc/localtime 容器时间同步虚拟机的时间
```

### 12.Docker compose install

```sh
   $sudo apt install docker-compose
   #启动服务
   $sudo docker-compose -f docker-compose.yml -d
   #停止服务
   $sudo docker-compose -f docker-compose.yml stop
   #停止并删除服务
   $sudo docker-compose -f docker-compose.yml down
   # 重启单个容器
   $sudo docker-compose restart rabbitmq
```

### 13.Docker 国内镜像地址

```sh
   $sudo vim /etc/docker/daemon.json
   {
 "registry-mirrors":[
     "https://9cpn8tt6.mirror.aliyuncs.com",
     "https://registry.docker-cn.com"
 ]
 }
 $sudo service docker restart
```

### docker问题

```bash
   Just in case anyone else stumbles upon the same error message. I also received the error: docker.socket: Failed to resolve group docker. And for me the solution as posted in this issue resolved the error:

   sudo groupadd docker
   sudo usermod -aG docker $USER
   sudo systemctl enable docker
   sudo systemctl start docker
```

#### 14.docker-compose 启动kafka

```yml
$ docker compose -f docker-compose.yml up -d
```
