## Storm实战 构建大数据实时计算
***

ZeroMQ

sudo yum install maven

## 1简介
- 使用场景

1. 实时分析
1. 在线机器学习
1. 持续计算
1. 分布式RPC
1. ETL
保证每个消息都得到处理,速度快每个节点每秒百万次消息.

- 实体

1. 工作进程:每台机器上多个
1. exector:每个进程多个
1. 任务:每个exector多个任务
 spot bolt

 storm 0.7版本引入**事物拓扑**解决,严格要求每个事物仅处理一次.

 - 多语言协议,每个tuple处理时需要进行JSON编解码.**吞吐量有影响**
 - ZeroMQ作为底层消息对列,消息快速处理.
ZeroMQ是一个为可伸缩的分布式或并发应用程序设计的高性能异步消息库。但是与面向消息的中间件不同，ZeroMQ的运行不需要专门的消息代理（message broker）。该库设计成常见的套接字风格的API。ZeroMQ是由iMatix公司和大量贡献者组成的社区共同开发的。ZeroQ通过许多第三方软件支持大部分流行的编程语言，从Java和Python到Erlang和Haskell。
- 支持动态增加节点,但是现有的任务不会自动负载均衡.
- 图形化监控
- 中间状态查询与存储

1. 处理流的结果,无法直接取得.导入MySQL或HBase中.
1. 计算逻辑类的快照,便于错误恢复.
但是有些业务需要保存中间状态,利用MySQL实时存储中间状态.崩溃从最近状态恢复.将数据源存储到HBase中,恢复后取出未处理的结果.利用HBase支持前后定位.


## 2Storm初体验
 
 - 节点类型
 	- 主控节点master
 	Nimbus的后台程序,分发代码,分配任务,监控状态.
 	- 工作节点 worker
 	运行一个Supervisoer后台程序,监听Nimbus分配的任务.启动或停止进程.
 	一个Topology由分布在不同工作节点上的多个工作进程组成.
 	Nimbus和Supervisoer间协调通过zookeeper

	Nimbus和Supervisoer是快速失败和无状态.结束后,要么在zookeeper要么在硬盘上,**拥有不可思议的稳定性**.

## 3构建Topology
- Topology
Topology不会结束,MR会结束.
Topology时Thrift(跨语言框架).
- 流
一个消息流就是一个没有边界的tuple抽象.
- sqout

1. 方法`nextTuple()`发射一个tuple到topology中.`nextTuple()`不能被阻塞,UI个exector调用所有消息源的`spout`方法.
1. `ack()`tuple成功处理
1. `fail()`tubples处理失败.
**只对可靠的spout调用ack和fail**

- Bolts
所有的消息处理逻辑.

	- 过滤
	- 聚合
	- 查询数据库

1. `OutputFieldsDeclarer.declareStream()`定义stream.
1. `OutputCollector.emit()`选择发射的Stream
1. execute处理tuple.
1. OutputCollector发射tuple.为每个处理的tuple调用ack方法.通知storm该tuple处理完毕.

- Stream Grouping 
Stream Grouping 定义一个stream如何分配bolts上面的多个task.

7种类型的Stream Grouping 

1. shuffle 随机,每个bolt数目大致相同
1. fields 字段分组
1. all 广播发送,每个tuple所有的bolts收到
1. global 全局分组,tupe分配到id值最低的task
1. non 随机,放到bolt的同一个exector执行.
1. direct 直接,特备.指定接受者的task
1. local or shuffle bolt有1个或多个task在同一个进程中,随机分.否则和shuffle grouping 行为一致.

- 可靠性

- tasks

- workers

	- 一个topology有多个worker(进程)
	- 每个worker是一个物理JVM
	- 并行度300的topology 50个进程的话.每个进程处理6个tasks.均分.

## 4Topology并行度
一台机器为多个topology运行多个进程.
**一个进程属于一个特定的topology**
一个进程为topology启动多个exector.
每个exector会为**特定**spout/bolt 运行一个或多个任务.
默认每个exector执行一个任务.


设置每个spout/bolt启动几个executor.默认启动1个exector.
配置任务数,每个bolt/spout执行多少个任务.

- 动态增加或减少exector数或进程数.不需要重启集群或者topology

## 5消息的可靠处理

确保spout发出的每个消息都被完整处理.
tuple tree超时值默认30s.


























