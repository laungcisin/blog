---
title: zookeeper-kafka集群的安装部署
date: 2099-12-31 00:00:04
tags:
---

# zookeeper集群搭建
## 解压文件
```
tar -zxvf /home/bigdata/software/zookeeper-3.4.11.tar.gz -C /export/servers/
cd /export/servers/
ln -s zookeeper-3.4.11 zookeeper
```

## 修改配置
### 修改zookeeper数据目录和日志目录
```
cd /export/servers/zookeeper/conf
cp zoo_sample.cfg zoo.cfg
vi /export/servers/zookeeper/conf/zoo.cfg

#zookeeper数据目录
dataDir=/export/data/zookeeper/workdir/data

#zookeeper日志目录
dataLogDir=/export/data/zookeeper/workdir/log
```

### 设置zookeeper集群信息
```
vi /export/servers/zookeeper/conf/zoo.cfg

...
#设置集群信息,此处的bigdata0x可以用ip地址代替
server.1=bigdata01:2888:3888
server.2=bigdata02:2888:3888
server.3=bigdata03:2888:3888
...
```

```
#执行
rm -rf /export/data/zookeeper/

mkdir -p /export/data/zookeeper/workdir/data
mkdir -p /export/data/zookeeper/workdir/log

echo "1" > /export/data/zookeeper/workdir/data/myid
```

### 将zookeeper拷贝到其它机器上
```
#将zookeeper拷贝至bigdata02机器上
scp -r /export/servers/zookeeper-3.4.11 bigdata@bigdata02:/export/servers

#进入bigdata02机器执行以下命令
mkdir -p /export/data/zookeeper/workdir/data
echo "2" > /export/data/zookeeper/workdir/data/myid

#将zookeeper拷贝至bigdata03机器上
scp -r /export/servers/zookeeper-3.4.11 bigdata@bigdata03:/export/servers

#进入bigdata03机器执行以下命令
mkdir -p /export/data/zookeeper/workdir/data
echo "3" > /export/data/zookeeper/workdir/data/myid
```

### 配置环境变量
```
#切换成root用户执行，修改profile文件
vi /etc/profile
#Zookeeper
export ZOOKEEPER_HOME=/export/servers/zookeeper
export PATH=$PATH:${ZOOKEEPER_HOME}/bin


#使配置生效
source /etc/profile
```

## zk命令
```
#启动时，切换成bigdata用户
su bigdata

#启动zookeeper
zkServer.sh start

#查看zookeeper状态
zkServer.sh status
```

# kafka集群搭建
搭建kafka集群前，先保证zookeeper集群已搭建成功。

## 解压文件
```
tar -zxvf /home/bigdata/software/kafka_2.11-1.0.0.tgz -C /export/servers/
cd /export/servers/
ln -s kafka_2.11-1.0.0 kafka
```
## 修改配置
### 修改zookeeper数据目录和日志目录
```
cp /export/servers/kafka/config/server.properties /export/servers/kafka/config/server.properties.bak
vi /export/servers/kafka/config/server.properties

#根据实际情况修改
#当前机器在集群中的唯一标识，和zookeeper的myid性质一样
broker.id=0

#当前kafka对外提供服务的端口默认是9092
port=9092

#这个参数默认是关闭的，在0.8.1有个bug，DNS解析问题，失败率的问题。
host.name=192.168.33.61

#这个是borker进行网络处理的线程数
num.network.threads=3

#这个是borker进行I/O处理的线程数
num.io.threads=8

#消息存放的目录，这个目录可以配置为","逗号分割的表达式，
#上面的num.io.threads要大于这个目录的个数这个目录，
#如果配置多个目录，新创建的topic他把消息持久化的地方是，当前以逗号分割的目录中，那个分区数最少就放那一个
log.dirs=/export/data/kafka/logs

#发送缓冲区buffer大小，数据不是一下子就发送的，先回存储到缓冲区了到达一定的大小后在发送，能提高性能
socket.send.buffer.bytes=102400

#kafka接收缓冲区大小，当数据到达一定大小后在序列化到磁盘
socket.receive.buffer.bytes=102400

#这个参数是向kafka请求消息或者向kafka发送消息的请请求的最大数，这个值不能超过java的堆栈大小
socket.request.max.bytes=104857600

#默认的分区数，一个topic默认1个分区数
num.partitions=1

#默认消息的最大持久化时间，168小时，7天
log.retention.hours=168

#消息保存的最大值5M
message.max.byte=5242880


#kafka保存消息的副本数，如果一个副本失效了，另一个还可以继续提供服务
default.replication.factor=2

#取消息的最大直接数
replica.fetch.max.bytes=5242880

#这个参数是：因为kafka的消息是以追加的形式落地到文件，当超过这个值的时候，kafka会新起一个文件
log.segment.bytes=1073741824

#每隔300000毫秒去检查上面配置的log失效时间（log.retention.hours=168 ），
#到目录查看是否有过期的消息如果有，删除
log.retention.check.interval.ms=300000

#是否启用log压缩，一般不用启用，启用的话可以提高性能
log.cleaner.enable=false

#设置zookeeper的连接端口
zookeeper.connect=bigdata01:2181,bigdata02:2181,bigdata03:2181

#远程消费的话，配置此参数
listeners=PLAINTEXT://192.168.33.61:9092
```
## 分发安装包
```
#将kafka安装包分发到其它机器上
scp -r /export/servers/kafka_2.11-1.0.0 bigdata@bigdata02:/export/servers
scp -r /export/servers/kafka_2.11-1.0.0 bigdata@bigdata03:/export/servers

#然后分别在各机器上创建软连
cd /export/servers/
ln -s kafka_2.11-1.0.0 kafka
```
## 修改其它机器上的kafka配置
```
#修改其它机器的kafka配置信息
vi /export/servers/kafka/config/server.properties

...
将broker.id的值，修改成相应的数字
...
```

## 添加环境变量
```
#切换成root用户执行，修改profile文件
vi /etc/profile

#Kafka
export KAFKA_HOME=/export/servers/kafka
export PATH=$PATH:$KAFKA_HOME/bin

#使配置生效
source /etc/profile
```

## kafka命令
```
#依次在各节点上启动kafka
kafka-server-start.sh /export/servers/kafka/config/server.properties > /dev/null 2>&1 &

#停止kafka
kafka-server-stop.sh

#查看当前服务器中的所有topic
kafka-topics.sh --list --zookeeper bigdata01:2181

#创建topic
kafka-topics.sh --create --zookeeper bigdata01:2181 --replication-factor 3 --partitions 3 --topic test

#删除topic
kafka-topics.sh --delete --zookeeper bigdata01:2181 --topic test
需要server.properties中设置delete.topic.enable=true否则只是标记删除或者直接重启。

#通过shell命令发送消息
kafka-console-producer.sh --broker-list bigdata01:9092 --topic test


#通过shell消费消息
kafka-console-consumer.sh --zookeeper bigdata01:2181 --from-beginning --topic test

#查看消费位置
kafka-run-class.sh kafka.tools.ConsumerOffsetChecker --zookeeper bigdata01:2181 --group testGroup

#查看某个Topic的详情
kafka-topics.sh --topic test --describe --zookeeper bigdata01:2181

```