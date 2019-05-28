---
title: 07-单机版redis的安装以及redis生产环境启动方案
date: 2018-11-02 10:56:16
tags: 
categories: redis
---


## 1. 安装单机版redis

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
## 2. `redis`的生产环境启动方案（非`root`用户启动）
1. 将 `redis/utils/redis_init_script`脚本拷贝至`/etc/init.d` ，重命名为`redis_6379`，`6379`是`redis`实例监听的端口号。
```
#root用户执行
cp -r /export/servers/redis/utils/redis_init_script /etc/init.d
cd /etc/init.d
mv redis_init_script redis_6379
```

2. 修改 `/etc/init.d/redis_6379` 脚本内容
    * 修改 `/etc/init.d/redis_6379`的`REDISPORT`为`6379`，默认是`6379`，可不修改
    * 设置 `PIDFILE=/export/data/redis/redis_${REDISPORT}.pid`
    * 设置 `CONF="/export/data/redis/conf/${REDISPORT}.conf"`


3. 创建两个目录，一个存放`redis`配置文件，一个存放`redis`的持久化文件
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