---
title: 2-redis高可用服务缓存架构-软件安装
date: 2100-01-01 08:08:07
tags: 
categories: 
---


## 7.1 安装单机版redis
### 7.1.1 安装单机版redis

1. 更改`/usr/local`的`group`和`user`权限

	```
	sudo chown bigdata:bigdata /usr/local -R
	```

2. 安装`tcl`。本文安装`tcl8.6.1`版本

	```
	
	cd /home/bigdata/software/
	wget http://downloads.sourceforge.net/tcl/tcl8.6.1-src.tar.gz
	chmod 777 tcl8.6.1-src.tar.gz
	tar -zxvf /home/bigdata/software/tcl8.6.1-src.tar.gz -C /export/servers
	cd /export/servers/tcl8.6.1/unix/
	./configure
	make && make install
	
	```
3. 安装`redis`。本文安装`redis-3.2.8`版本

	```
	
	cd /home/bigdata/software/
	wget http://download.redis.io/releases/redis-3.2.8.tar.gz
	chmod 777 redis-3.2.8.tar.gz
	tar -zxvf /home/bigdata/software/redis-3.2.8.tar.gz  -C /export/servers
	cd /export/servers
	ln -s redis-3.2.8 redis
	cd /export/servers/redis
	make && make test && make install
	
	```

### 7.1.2 `redis`的生产环境启动方案（非`root`用户启动）

1. 将 `/export/servers/redis/utils/redis_init_script`脚本拷贝至`/etc/init.d` ，重命名为`redis_6379`，`6379`是`redis`实例监听的端口号。

	```
	#root用户执行
	cp -r /export/servers/redis/utils/redis_init_script /etc/init.d
	cd /etc/init.d
	mv redis_init_script redis_6379
	```

2. 创建三个目录，一个存放`redis`配置文件，一个存放`redis`的持久化文件，一个存放`redis`日志
    * 创建存放`redis`的配置文件目录
	```
	#非root用户执行
	mkdir -p /export/data/redis/conf
	```

    * 创建存放`redis`的持久化文件
	```
	#非root用户执行
	mkdir -p /export/data/redis/6379
	```

    * 创建存放`redis`日志
	```
	#非root用户执行
	mkdir -p /export/data/redis/logs
	```

3. 修改 `/etc/init.d/redis_6379` 脚本内容
    * 修改 `/etc/init.d/redis_6379`的`REDISPORT`为`6379`，默认是`6379`，可不修改
    * 设置 `PIDFILE=/export/data/redis/redis_${REDISPORT}.pid`
    * 设置 `CONF="/export/data/redis/conf/${REDISPORT}.conf"`
	

4. 修改`redis`配置文件，并重命名
	将`/export/servers/redis/redis.conf`拷贝至`/export/data/redis/conf`，并重命名成`6379.conf`
	```
	#非root用户执行
	cp -r /export/servers/redis/redis.conf /export/data/redis/conf
	cd /export/data/redis/conf
	mv redis.conf 6379.conf
```
**修改配置**

| 配置项    | 配置值                            | 说明                                                         |
| :-------- | --------------------------------- | ------------------------------------------------------------ |
| daemonize | yes                               | 让redis以daemon进程运行                                      |
| pidfile   | /export/data/redis/redis_6379.pid | 设置redis的pid文件位置                                       |
| port      | 6379                              | 设置redis的监听端口号                                        |
| dir       | /export/data/redis/6379           | 设置持久化文件的存储位置                                     |
| bind      | 127.0.0.1 192.168.33.61           | 设置绑定地址，使用redis-cli命令时， 使用-h参数时， 要指定ip地址 |

5. 启动`redis`，关闭`redis`
    * `root`用户启动`redis`:
	```
	#用root用户执行
	cd /etc/init.d
	chmod 777 redis_6379
	./redis_6379 start
	```

    * 非`root`用户启动`redis`:
	```
	/usr/local/bin/redis-server /export/data/redis/conf/6379.conf
	```

    * 非`root`用户关闭`redis`:
	```
	/usr/local/bin/redis-cli -p 7200 shutdown
	```

6. 确认`redis`进程是否启动
	```
	ps -ef | grep redis
	```
7. 让`redis`跟随系统启动  
	修改`/etc/init.d/redis_6379`脚本
	在`/etc/init.d/redis_6379`脚本中，最上面，加入下面两行注释
	```
	#用root用户执行
	vi /etc/init.d/redis_6379

	# 添加下面两行注释
	# chkconfig:   2345 90 10
	# description:  Redis is a persistent key-value database

	#执行以下命令，设置开机服务自动启动
	cd /etc/init.d/
	chkconfig redis_6379 on

	#如果要关闭自动启动，执行以下命令
	cd /etc/init.d/
	chkconfig redis_6379 off
	```

8. `redis cli`的使用

	```
	#连接本机的6379端口停止redis进程
	redis-cli SHUTDOWN

	redis-cli -h 192.168.33.61 -p 6379

	#指定要连接的ip和端口号
	redis-cli -h 127.0.0.1 -p 6379 SHUTDOWN

	#ping redis的端口，看是否正常
	redis-cli PING

	redis-cli，进入交互式命令行

	#设置键值
	SET k1 v1

	#获取值
	GET k1
	```
	
	
## 7.2 搭建`redis-cluster`集群	

### 7.2.1 安装集群前的准备
***CentOS安装redis集群提示**`redis requires ruby version 2.2.2`**的解决方案***
1. 安装RVM
```
#具体RVM安装命令地址，参考：http://rvm.io/
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB 

curl -sSL https://get.rvm.io | bash -s stable
source /usr/local/rvm/scripts/rvm
```
1. 查看rvm库中已知的ruby版本
```
rvm list known
```
1. 安装一个ruby版本
```
rvm install 2.5.1
```
1. 使用一个ruby版本
```
rvm use 2.5.1
```
1. 设置默认版本
```
rvm use 2.5.1 --default
```
1. 卸载一个已知版本
```
rvm remove 1.8.1
rvm remove 2.3.4
rvm remove 2.4.1
```
1. 查看ruby版本
```
ruby --version
```
1. 安装redis
```
gem install redis
```

### 7.2.2 安装`redis`集群
>集群配置3台机器，一台机器配置一主一从。  
>redis的配置文件：`/export/data/redis/conf/700*.conf`  
>redis的持久化文件夹：`/export/data/redis/700*`

**3台机器：**
>192.168.33.61  
>192.168.33.62  
>192.168.33.63

1. 创建目录
在3台机器上执行如下命令：
```
mkdir -p /export/data/redis-cluster
mkdir -p /export/data/redis-cluster-log
mkdir -p /export/data/redis/7001
mkdir -p /export/data/redis/7002
mkdir -p /export/data/redis/7003
mkdir -p /export/data/redis/7004
mkdir -p /export/data/redis/7005
mkdir -p /export/data/redis/7006
```
    >`/export/data/redis-cluster`为集群目录  
    >`/export/data/redis-cluster-log`为集群日志目录

1. 准备配置文件  
**在1台机器上执行如下命令，准备好配置文件**
```
cp -r /export/servers/redis/redis.conf /export/data/redis/conf/7001.conf
cp -r /export/servers/redis/redis.conf /export/data/redis/conf/7002.conf
cp -r /export/servers/redis/redis.conf /export/data/redis/conf/7003.conf
cp -r /export/servers/redis/redis.conf /export/data/redis/conf/7004.conf
cp -r /export/servers/redis/redis.conf /export/data/redis/conf/7005.conf
cp -r /export/servers/redis/redis.conf /export/data/redis/conf/7006.conf

```

1. 修改配置  
***
**机器端口分配情况：**

| 机器ip             | 分配的端口号                         |
| :----------------- | ----------------------------------- |
| 192.168.33.61      | 7001 ~ 7002                         |
| 192.168.33.62      | 7003 ~ 7004                         |
| 192.168.33.63      | 7005 ~ 7006                         |


---
根据如下表格，并根据机器ip、端口分配情况，修改**`/export/data/redis/conf/700*.conf`**相应配置  

| 配置项    | 配置值                            | 说明                                                         |
| :-------- | --------------------------------- | ------------------------------------------------------------ |
| port      | 700*                              | 设置redis的监听端口号                                      |
| cluster-enabled | yes                         | 开启集群                                      |
| cluster-config-file | /export/data/redis-cluster/nodes-700*.conf                         | 集群配置文件 |
| cluster-node-timeout | 15000                               | 超时时长                                      |
| daemonize | yes                               | 让redis以daemon进程运行                                      |
| pidfile   | /export/data/redis/redis_700*.pid | 设置redis的pid文件位置                                       |
| dir       | /export/data/redis/700*           | 设置持久化文件的存储位置                                     |
| logfile       | /export/data/redis-cluster-log/700*.log           | 设置集群日志持久化文件的存储位置 |
| bind      | 192.168.33.6*           | 设置绑定地址，最好不要绑定127.0.0.1，否则集群启动不起来|
| appendonly      | yes           | 开启AOF功能 |

---
配置完后，将配置文件分发到另外两台机器
```
scp -r /export/servers/redis/700* bigdata@192.168.33.62:/export/data/redis/conf/
scp -r /export/servers/redis/700* bigdata@192.168.33.63:/export/data/redis/conf/
```

1. 准备生产环境的启动脚本  
在`/etc/init.d`目录下，准备6个启动脚本，分别为: `redis_7001, redis_7002, redis_7003, redis_7004, redis_7005, redis_7006`，在每个启动脚本内，都修改对应的端口号。
```
#!/bin/sh
# chkconfig:   2345 90 10
# description:  Redis is a persistent key-value database
#
# Simple Redis init.d script conceived to work on Linux systems
# as it does use of the /proc filesystem.

#redis服务器监听的端口，根据实际情况修改
REDISPORT=700*
#EXEC=/usr/local/bin/redis-server
#CLIEXEC=/usr/local/bin/redis-cli

#PIDFILE=/var/run/redis_${REDISPORT}.pid
#CONF="/etc/redis/${REDISPORT}.conf"

#服务端所处位置，在make install后默认存放于`/usr/local/bin/redis-server`，如果未make install则需要修改该路径，下同。
EXEC=/usr/local/bin/redis-server
#客户端位置
CLIEXEC=/usr/local/bin/redis-cli

#Redis的PID文件位置
PIDFILE=/export/data/redis/redis_${REDISPORT}.pid
#配置文件位置，需要修改
CONF="/export/data/redis/conf/${REDISPORT}.conf"

case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
                echo "$PIDFILE does not exist, process is not running"
        else
                PID=$(cat $PIDFILE)
                echo "Stopping ..."
                $CLIEXEC -p $REDISPORT shutdown
                while [ -x /proc/${PID} ]
                do
                    echo "Waiting for Redis to shutdown ..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
    *)
        echo "Please use start or stop as first argument"
        ;;
esac

```

1. 在3台机器上，启动6个redis实例 

    >**如果有启动redis实例，先停止实例**
```
/usr/local/bin/redis-cli -p 7001 shutdown
/usr/local/bin/redis-cli -p 7002 shutdown
/usr/local/bin/redis-cli -p 7003 shutdown
/usr/local/bin/redis-cli -p 7004 shutdown
/usr/local/bin/redis-cli -p 7005 shutdown
/usr/local/bin/redis-cli -p 7006 shutdown
```

    >**清理集群状态**
```
rm -rf /export/data/redis-cluster/*
rm -rf /export/data/redis-cluster-log/*

rm -rf /export/data/redis/7001/*
rm -rf /export/data/redis/7002/*
rm -rf /export/data/redis/7003/*
rm -rf /export/data/redis/7004/*
rm -rf /export/data/redis/7005/*
rm -rf /export/data/redis/7006/*

rm -rf /export/data/redis/redis_700*.pid

#在192.168.33.61机器上执行redis-trib.rb create，所以要在清理192.168.33.61上的配置
redis-cli -h 192.168.33.61 -p 7001 flushdb 
redis-cli -h 192.168.33.61 -p 7001 flushall
redis-cli -h 192.168.33.61 -p 7001 cluster reset
redis-cli -h 192.168.33.61 -p 7002 flushdb 
redis-cli -h 192.168.33.61 -p 7002 flushall
redis-cli -h 192.168.33.61 -p 7002 cluster reset
redis-cli -h 192.168.33.61 -p 7003 flushdb 
redis-cli -h 192.168.33.61 -p 7003 flushall
redis-cli -h 192.168.33.61 -p 7003 cluster reset
redis-cli -h 192.168.33.61 -p 7004 flushdb 
redis-cli -h 192.168.33.61 -p 7004 flushall
redis-cli -h 192.168.33.61 -p 7004 cluster reset
redis-cli -h 192.168.33.61 -p 7005 flushdb 
redis-cli -h 192.168.33.61 -p 7005 flushall
redis-cli -h 192.168.33.61 -p 7005 cluster reset
redis-cli -h 192.168.33.61 -p 7006 flushdb 
redis-cli -h 192.168.33.61 -p 7006 flushall
redis-cli -h 192.168.33.61 -p 7006 cluster reset
```

    >使用`flushall`和`cluster reset`命令，避免出现`ERR Slot 0 is already busy (Redis::CommandError)`错误

   >**192.168.33.61机器执行以下命令：**
```
/usr/local/bin/redis-server /export/data/redis/conf/7001.conf
/usr/local/bin/redis-server /export/data/redis/conf/7002.conf
```

   >**192.168.33.62机器执行以下命令：**
```
/usr/local/bin/redis-server /export/data/redis/conf/7003.conf
/usr/local/bin/redis-server /export/data/redis/conf/7004.conf
```

   >**192.168.33.63机器执行以下命令：**
```
/usr/local/bin/redis-server /export/data/redis/conf/7005.conf
/usr/local/bin/redis-server /export/data/redis/conf/7006.conf
```

    >**查看redis实例启动日志：**
```
cat /export/data/redis-cluster-log/700*
```


1. 创建集群  

    >**执行以下命令：**
```
cp /export/servers/redis/src/redis-trib.rb /usr/local/bin
redis-trib.rb create --replicas 1 192.168.33.61:7001 192.168.33.61:7002 192.168.33.62:7003 192.168.33.62:7004 192.168.33.63:7005 192.168.33.63:7006
```
   >--replicas 1: 每个master有1个slave

   >**如果出现以下错误：**
[ERR] Not all 16384 slots are covered by nodes，  
执行以下命令：
```
redis-trib.rb fix 192.168.33.61:7001
```
   >**查看集群状态：**
```
redis-cli -h 192.168.33.61 -c -p 7001 cluster nodes
```

1. 设置开机启动  
>使用root用户执行  
>**192.168.33.61机器执行以下命令：**
```
#执行以下命令，设置开机服务自动启动
cd /etc/init.d/
chkconfig redis_7001 on
chkconfig redis_7002 on

#如果要关闭自动启动，执行以下命令
cd /etc/init.d/
chkconfig redis_7001 off
chkconfig redis_7002 off
```

   >**192.168.33.62机器执行以下命令：**
```
#执行以下命令，设置开机服务自动启动
cd /etc/init.d/
chkconfig redis_7003 on
chkconfig redis_7004 on

#如果要关闭自动启动，执行以下命令
cd /etc/init.d/
chkconfig redis_7003 off
chkconfig redis_7004 off
```

   >**192.168.33.63机器执行以下命令：**
```
#执行以下命令，设置开机服务自动启动
cd /etc/init.d/
chkconfig redis_7005 on
chkconfig redis_7006 on

#如果要关闭自动启动，执行以下命令
cd /etc/init.d/
chkconfig redis_7005 off
chkconfig redis_7006 off
```

