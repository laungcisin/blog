---
title: 单机启动redis集群
date: 2099-12-31 00:00:01
tags:
---

```

#停止redis进程
/usr/local/bin/redis-cli -h 192.168.33.61 -p 7001 shutdown
/usr/local/bin/redis-cli -h 192.168.33.61 -p 7002 shutdown
/usr/local/bin/redis-cli -h 192.168.33.61 -p 7003 shutdown
/usr/local/bin/redis-cli -h 192.168.33.61 -p 7004 shutdown
/usr/local/bin/redis-cli -h 192.168.33.61 -p 7005 shutdown
/usr/local/bin/redis-cli -h 192.168.33.61 -p 7006 shutdown

#启动redis集群
/usr/local/bin/redis-server /export/data/test-env-redis/redis/conf/7001.conf
/usr/local/bin/redis-server /export/data/test-env-redis/redis/conf/7002.conf
/usr/local/bin/redis-server /export/data/test-env-redis/redis/conf/7003.conf
/usr/local/bin/redis-server /export/data/test-env-redis/redis/conf/7004.conf
/usr/local/bin/redis-server /export/data/test-env-redis/redis/conf/7005.conf
/usr/local/bin/redis-server /export/data/test-env-redis/redis/conf/7006.conf

#查看redis集群启动日志
cat /export/data/test-env-redis/redis-cluster-log/700*



#清理redis集群
mkdir -p /export/data/test-env-redis/redis-cluster
mkdir -p /export/data/test-env-redis/redis-cluster-log
mkdir -p /export/data/test-env-redis/redis/7001
mkdir -p /export/data/test-env-redis/redis/7002
mkdir -p /export/data/test-env-redis/redis/7003
mkdir -p /export/data/test-env-redis/redis/7004
mkdir -p /export/data/test-env-redis/redis/7005
mkdir -p /export/data/test-env-redis/redis/7006


rm -rf /export/data/test-env-redis/redis-cluster/*
rm -rf /export/data/test-env-redis/redis-cluster-log/*

rm -rf /export/data/test-env-redis/redis/7001/*
rm -rf /export/data/test-env-redis/redis/7002/*
rm -rf /export/data/test-env-redis/redis/7003/*
rm -rf /export/data/test-env-redis/redis/7004/*
rm -rf /export/data/test-env-redis/redis/7005/*
rm -rf /export/data/test-env-redis/redis/7006/*
rm -rf /export/data/test-env-redis/redis/redis_700*.pid

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



#拷贝命令至bin目录
#cp /export/servers/redis/src/redis-trib.rb /usr/local/bin

#创建集群
redis-trib.rb create --replicas 1 192.168.33.61:7001 192.168.33.61:7002 192.168.33.61:7003 192.168.33.61:7004 192.168.33.61:7005 192.168.33.61:7006





```
