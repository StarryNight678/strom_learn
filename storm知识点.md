# storm知识点

1. [关于Storm与JStorm的调度算法的讨论](http://m.blog.csdn.net/article/details?id=50433273)
1. [Storm 性能优化 例子](http://www.jianshu.com/p/f645eb7944b0)
1. [图书 Storm技术内幕与大数据实践 192页](http://detail.dangdang.com/23699059.html#catalog)
1. [Storm：大数据流式计算及应用实践](http://detail.dangdang.com/23668216.html#catalog)

## storm1.0.0性能提升

Storm 1.0.0说性能提升了16倍,延迟减少了60%
性能如何提升,提升了哪些方面?

- 自动反压机制	[反压介绍](http://jobs.one2team.com/apache-storms/)
- zoonkeeper 是瓶颈.
- Pacemaker - Heartbeat Server 自己处理心跳信息.
- HA Nimbus 多个Nimbus 自己选举


## 延迟含义

[延迟讲解](http://stackoverflow.com/questions/33558613/storm-huge-discrepancy-between-bolt-latency-and-total-latency)

1. 完整的延迟:Tuple “tree” 完全处理的平均时间.标记为0表示,no acking.
1. Execute latency : 在执行方法中的平均时间,运行Bolt.execute().时间.
1. Process latency : tuple收到和确认的的平均时间.

## Spouts

数据源泉，获取数据，是整个拓扑数据的生产者。

1. nextTuple：发射新tuple到topology/无tuple时返回；
2. ack：storm检测到一个tuple被整个topology处理成功时调用；
3. fail：storm检测到一个tuple被整个topology处理失败时调用；

## Bolts

调用ack通知storm：已经处理过本tuple;

具体任务的执行者



## 消息分发策略

消息分发策略
Stream groupings,为定义一个流的分发策略，也就是说Spouts产生的数据怎么向Bolts流，目前支持六种分发策略：

1. Shuffle Grouping: 随机分组， 随机派发stream里面的tuple， 保证每个bolt接收到的tuple数目相同.
2. Fields Grouping：按字段分组， 比如按userid来分组， 具有同样userid的tuple会被分到相同的Bolts， 而不同的userid则会被分配到不同的Bolts.
3. All Grouping： 广播发送， 对于每一个tuple， 所有的Bolts都会收到.
4. Global Grouping: 全局分组，这个tuple被分配到storm中的一个bolt的其中一个task.再具体一点就是分配给id值最低的那个task.
5. Non Grouping: 不分组，意思是说stream不关心到底谁会收到它的tuple.目前他和Shuffle grouping是一样的效果,有点不同的是storm会把这个bolt放到这个bolt的订阅者同一个线程去执行.
6. Direct Grouping: 直接分组,这是一种比较特别的分组方法，用这种分组意味着消息的发送者由消息接收者的哪个task处理这个消息.只有被声明为Direct Stream的消息流可以声明这种分组方法.而且这种消息tuple必须使用emitDirect方法来发射.消息处理者可以通过TopologyContext来或者处理它的消息的taskid (OutputCollector.emit方法也会返回taskid)

## Trident  


## DRPC

## Thrift服务框架

Thrift是一个跨语言的可扩展的服务框架，它通过一个中间语言(IDL，接口定义语言)来定义RPC的接口和数据类型，然后通过一个编译器生成RPC客户端和服务器通信的无缝跨编程语言。
Storm中Thrift的应用场景：

1. 客户端向nimbus提交topology任务；
1. supervisor从nimbus下载topology任务（代码和序列化文件）；
1. storm ui从nimbus获取topology运行的统计信息。

## ZeroMQ

ZeroMQ是一个基于消息的嵌入式网络编程库，可作为并发框架连接多个应用程序，支持N-to-N的连接，多种工作模式（Request-reply，Publish-subscribe，Pipeline等），支持多种语言，ZeroMQ使得编写高性能网络应用程序极为简单。
Storm中ZeroMQ的应用场景：Spout与Bolt、Bolt与Bolt之间tuple消息的传输。

## netty

## Java序列化

Java**序列化**技术可以实现将Java对象保存为二进制文件，
而**反序列化**过程则可以在另一个Java进程中将此二进制文件恢复为Java对象，其缺点在于不能很好的解决版本变化。

Storm中Java序列化的应用场景：

1. 客户端提交topology任务后，Storm将topology任务序列化并发送给nimbus；
1. supervisor从Zookeeper取得任务信息后，从nimbus下载序列化文件和jar包，启动worker进程并反序列化得到提交任务时的topology对象。

# 相关开源软件

## Zookeeper

Storm中使用Zookeeper主要用于Storm集群各节点的分布式协调工作，具体功能如下：

1. 存储客户端提供的topology任务信息，nimbus负责将任务分配信息写入Zookeeper，supervisor从Zookeeper上读取任务分配信息；
1. 存储supervisor和worker的心跳（包括它们的状态），使得nimbus可以监控整个集群的状态， 从而重启一些挂掉的worker；
1. 存储整个集群的所有状态信息和配置信息。


## Heron