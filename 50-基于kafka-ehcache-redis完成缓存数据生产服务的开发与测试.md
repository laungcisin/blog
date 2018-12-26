---
title: 50-基于kafka-ehcache-redis完成缓存数据生产服务的开发与测试
date: 2018-12-26 14:25:26
categories: redis
tags:
---
## kafka相关配置
```
vi /export/servers/kafka/config/server.properties

#远程消费的话，配置此参数
listeners=PLAINTEXT://192.168.33.61:9092
```
## kafka测试命令
```
#生成cache-message主题
kafka-topics.sh --create --zookeeper bigdata01:2181 --replication-factor 3 --partitions 3 --topic cache-message

#查看cache-message的详情
kafka-topics.sh --topic cache-message --describe --zookeeper bigdata01:2181

#生产数据
#注意事项，要远程消费数据，配置/export/servers/kafka/config/server.properties中listeners=PLAINTEXT://192.168.33.61:9092
kafka-console-producer.sh --broker-list bigdata01:9092,bigdata02:9092,bigdata03:9092 --topic cache-message
kafka-console-producer.sh --bootstrap-server 192.168.33.61:9092,192.168.33.62:9092,192.168.33.63:9092 --topic cache-message

#往console输入数据
{"serviceId": "productInfoService", "productId": 5}
{"serviceId": "shopInfoService", "shopId": 1}

#停止kafka
kafka-server-stop.sh
```
## 代码注意点
```
Properties props = new Properties();
// 确保 rebalance.max.retries * rebalance.backoff.ms > zookeeper.session.timeout.ms
props.put("zookeeper.connect", "192.168.33.61:2181,192.168.33.62:2181,192.168.33.63:2181");
props.put("group.id", "eshop-cache-group");
props.put("zookeeper.session.timeout.ms", "40000");
props.put("zookeeper.connection.timeout.ms", "40000");
props.put("zookeeper.sync.time.ms", "200");
props.put("rebalance.backoff.ms", "20000");
props.put("rebalance.max.retries", "10");
props.put("auto.commit.interval.ms", "1000");
return new ConsumerConfig(props);
```


