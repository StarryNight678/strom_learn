# Storm技术内幕与大数据实践笔记

周健华 2016年10月

![](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTNjQT0Slvmv2q-Fq7L2-nFgEjIve1AW49IcHAGRYFm1TeiUmg4)


## 序

LinnkdIn 基于Kafka 开发了,Samza用于实时新闻推送,广告和复杂监控.

1号店使用经验.


## 1绪论

1. Nimbus 和 Supervisor 通信通过ZooKeeper完成.
1. storm 0.8版本开始**executor**为具体物理线程. 同一个spout/bolt的task可能会共享一个物理线程.


- Apache kafka
消息队列
消息队列技术是分布式应用间交换信息的一种技术。消息队列可驻留在内存或磁盘上, 队列存储消息直到它们被应用程序读走。通过消息队列，应用程序可独立地执行--它们不需要知道彼此的位置、或在继续执行前不需要等待接收程序接收此消息。在分布式计算环境中，为了集成分布式应用，开发者需要对异构网络环境下的分布式应用提供有效的通信手段。为了管理需要共享的信息，对应用提供公共的信息交换机制是重要的。常用的消息队列技术是 Message Queue。

Apache kafka 经常当做数据缓冲区.
- Jstorm

![](http://i.imgur.com/6ea5RVW.png)

## 2实时平台介绍

## 3Storm集群部署和配置

## 4Storm内部剖析

- 调度器

schedule(topologies,cluster)
topologies包含topology的静态信息
cluster包含topology的运行信息

- 默认调度器

defaultScheduler()

计算当前集群中可以分配的slot资源,并判断当前分配给运行topology的slot是否需要重新分配.然后对提交的topology进行资源分配.


- 均衡调度器


P49


## 5Storm运维监控
## 6Storm扩展
## 7Storm开发
## 8基于Storm的实时数据平台
## 9大数据应用案例
## 10使用经验和优化
