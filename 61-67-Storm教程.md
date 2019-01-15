---
title: 61-67-Storm教程
date: 2019-01-08 18:01:34
tags:
categories: storm
---


# ***Storm到底是什么？***

## mysql、hadoop与storm
>mysql：事务性系统，面临海量数据的尴尬
>hadoop：离线批处理
>storm：实时计算

## 我们能不能自己搞一套storm？
1. 花费大量的时间在底层技术细节上：如何部署各种中间队列，节点间的通信，容错，资源调配，计算节点的迁移和部署，等等
1. 花费大量的时间在系统的高可用上问题上：如何保证各种节点能够高可用稳定运行
1. 花费大量的时间在系统扩容上：吞吐量需要扩容的时候，你需要花费大量的时间去增加节点，修改配置，测试，等等

## storm的特点是什么？
1. 支撑各种实时类的项目场景
>实时处理消息以及更新数据库，基于最基础的实时计算语义和API（实时数据处理领域）；对实时的数据流持续的进行查询或计算，
>同时将最新的计算结果持续的推送给客户端展示，同样基于最基础的实时计算语义和API（实时数据分析领域）；
>对耗时的查询进行并行化，基于DRPC，即分布式RPC调用，单表30天数据，并行化，每个进程查询一天数据，最后组装结果

1. 高度的可伸缩性
>如果要扩容，直接加机器，调整storm计算作业的并行度就可以了，storm会自动部署更多的进程和线程到其他的机器上去，无缝快速扩容
>扩容起来，超方便

1. 数据不丢失的保证
>storm的消息可靠机制开启后，可以保证一条数据都不丢
>数据不丢失，也不重复计算

1. 超强的健壮性
>从历史经验来看，storm比hadoop、spark等大数据类系统，健壮的多的多，因为元数据全部放zookeeper，不在内存中，随便挂都不要紧
>特别的健壮，稳定性和可用性很高

1. 使用的便捷性
>核心语义非常的简单，开发起来效率很高
>用起来很简单，开发API还是很简单的

![mysql-hadoop与storm的关系](mysql-hadoop与storm的关系.png 'mysql-hadoop与storm的关系')

---

# Storm集群架构与核心概念

## Storm的集群架构

***Nimbus，Supervisor，ZooKeeper，Worker，Executor，Task***
![storm集群架构](storm集群架构.png 'storm集群架构')

## Storm的核心概念

***Topology，Spout，Bolt，Tuple，Stream***

>* 拓扑(Topology)：务虚的一个概念

>* Spout：数据源的一个代码组件，就是我们可以实现一个spout接口，写一个java类，在这个spout代码中，我们可以自己尝试去数据源获取数据，比如说从kafka中消费数据

>* bolt：一个业务处理的代码组件，spout会将数据传送给bolt，各种bolt还可以串联成一个计算链条，java类实现了一个bolt接口

>* 一堆spout+bolt，就会组成一个topology，就是一个拓扑，实时计算作业，spout+bolt，一个拓扑涵盖数据源获取/生产+数据处理的所有的代码逻辑，topology

>* tuple：就是一条数据，每条数据都会被封装在tuple中，在多个spout和bolt之间传递

>* stream：就是一个流，务虚的一个概念，抽象的概念，源源不断过来的tuple，就组成了一条数据流
![storm核心概念](storm核心概念.png 'storm核心概念')

## Storm的并行度和流分组

***并行度和流分组***

* 并行度：Worker->Executor->Task，没错，是Task

* 流分组：Task与Task之间的数据流向关系

>* Shuffle Grouping：随机发射，负载均衡
>* Fields Grouping：根据某一个，或者某些个fields，进行分组，那一个或者多个fields如果值完全相同的话，那么这些tuple，就会发送给下游bolt的其中固定的一个task
你发射的每条数据是一个tuple，每个tuple中有多个field作为字段
比如tuple，3个字段，name，age，salary
{"name": "tom", "age": 25, "salary": 10000} -> tuple -> 3个field，name，age，salary

>* All Grouping：全分组，发射给下游的每个task
>* Global Grouping：全局id最小的
>* None Grouping 与 Shuffle Grouping相同
>* Direct Grouping：直接指定
>* Local or Shuffle Grouping：同一个executor中

![storm并行度和流分组](storm并行度和流分组.png 'storm并行度和流分组')