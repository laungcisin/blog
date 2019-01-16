---
title: storm集群安装并运行程序
date: 2099-12-31 00:00:05
tags:
---
## 安装依赖包
 在Nimbus和worker机器上安装`Java`和`Python`依赖包
```
#需安装Java7以上版本
java -version

#需安装python2.7以上版本
python --version
```

## 解压storm安装包
```
tar -zxvf /home/bigdata/software/apache-storm-1.1.0.tar.gz -C /export/servers/
cd /export/servers/
rm -rf storm
ln -s apache-storm-1.1.0 storm
```

## 修改配置文件
```
cp /export/servers/storm/conf/storm.yaml /export/servers/storm/conf/storm.yaml.bak
vi /export/servers/storm/conf/storm.yaml
```
```
#指定storm使用的zk集群
storm.zookeeper.servers:
     - "bigdata01"
     - "bigdata02"
     - "bigdata03"

#指定storm本地目录
storm.local.dir: "/export/data/storm"

#用于配置主控节点的地址，可以配置多个
nimbus.seeds: ["192.168.33.61"]

#指定nimbus启动JVM最大可用内存大小
nimbus.childopts: "-Xmx1024m"

#指定supervisor启动JVM最大可用内存大小
supervisor.childopts: "-Xmx1024m"

#指定supervisor节点上，每个worker启动JVM最大可用内存大小
worker.childopts: "-Xmx768m"

#指定ui启动JVM最大可用内存大小，ui服务一般与nimbus同在一个节点上。
ui.childopts: "-Xmx768m"

#指定supervisor节点上，启动worker时对应的端口号，每个端口对应槽，每个槽位对应一个worker
#指定每个机器上可以启动多少个worker，一个端口号代表一个worker
supervisor.slots.ports:
    - 6700
    - 6701
    - 6702
    - 6703
```

***设置环境变量***
```
#使用root用户执行
vi /etc/profile

...
#Storm
export STORM_HOME=/export/servers/storm
export PATH=$PATH:$STORM_HOME/bin
...

source /etc/profile
```

## 分发安装包到其它台机器
```
scp -r /export/servers/apache-storm-1.1.0 bigdata02:/export/servers
scp -r /export/servers/apache-storm-1.1.0 bigdata03:/export/servers

#然后分别在各机器上创建软连接，并设置环境变量
cd /export/servers/
ln -s apache-storm-1.1.0 storm
```

***设置环境变量***
```
#使用root用户执行
vi /etc/profile

...
#Storm
export STORM_HOME=/export/servers/storm
export PATH=$PATH:$STORM_HOME/bin
...

source /etc/profile
```

## 启动storm集群、ui界面、运行程序
***启动前，先保证zookeeper集群已经启动***
```
#在主节点启动nimbus
storm nimbus >/dev/null 2>&1 &

#在所有机器启动supervisor
storm supervisor >/dev/null 2>&1 &

#在主节点启动ui
storm ui >/dev/null 2>&1 &

#在supervisor启动
storm logviewer >/dev/null 2>&1 &

#运行程序
storm jar /home/bigdata/run-jar-dir/storm-0.0.1-SNAPSHOT.jar com.laungcisin.storm.WordCountTopology wordCountTopology

#kill topology
storm kill wordCountTopology
```

## 通过ui界面查看集群
访问 http://192.168.33.61:8080 ，即可看到storm的ui界面。