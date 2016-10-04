# storm learn



## Jstorm
storm java 实现

JStorm 比Storm更稳定，更强大，更快， Storm上跑的程序，一行代码不变可以运行在JStorm上。

[Jstorm github](https://github.com/alibaba/jstorm)

[封仲淹：Storm 2.0将会基于JStorm，阿里巴巴全程参与](http://www.infoq.com/cn/news/2015/11/jstorm-apache-alibaba)

[中文资料](https://github.com/alibaba/jstorm/wiki/JStorm-Chinese-Documentation)

- 结论

1. JStorm 0.9.0 在使用Netty的情况下，比Storm 0.9.0 使用Netty情况下，快10%， 并且JStorm 1. Netty是稳定的而Storm的Netty是不稳定的
1. 在使用ZeroMQ的情况下， JStorm 0.9.0 比Storm 0.9.0 快30%

- 原因

1. Zeromq减少一次内存拷贝
1. 增加反序列化线程
1. 重写采样代码，大幅减少采样影响
1. 优化ack代码
1. 优化缓冲map性能
1. Java比Clojure更底层

## Heron Twitter新的流处理利器(开源了)

[Twitter Heron的深入解析(Twitter Heron与Storm的比较)](http://www.blogchong.com/post/117.html)


[Twitter已经用Heron替换了Storm](http://www.infoq.com/cn/news/2015/06/twitter-storm-heron)
Twitter已经用Heron替换了Storm。此举将吞吐量最高提升了14倍，单词计数拓扑时间延迟最低降到了原来的1/10，所需的硬件减少了2/3。

Wednesday, May 25, 2016 Twitter宣布开源Heron
[Open Sourcing Twitter Heron](https://blog.twitter.com/2016/open-sourcing-twitter-heron)


[开源github twitter/heron](https://github.com/twitter/heron)


Karthik Ramasamy是Twitter Storm/Heron团队的负责人。据他介绍，为满足这些需求，他们已经考虑了多个选项：增强Storm、使用一种不同的开源解决方案或者创建一个新的解决方案。增强Storm需要花费很长时间，也没有其它的系统能够满足他们在扩展性、吞吐量和延迟方面的需求。而且，其它系统也不兼容Storm的API，需要重写所有拓扑。所以，最终的决定是创建Heron，但保持其外部接口与Storm的接口兼容。

Twitter已经用Heron完全替换了Storm。前者现在每天处理“数10TB的数据，生成数10亿输出元组”，在一个标准的单词计数测试中，“吞吐量提升了6到14倍，元组延迟降低到了原来的五到十分之一”，硬件减少了2/3。

论文[Twitter Heron: Stream Processing at Scale](http://dl.acm.org/citation.cfm?id=2742788)


几个说明
[深度解析 Twitter Heron 大数据实时分析系统](http://dataunion.org/19297.html)


[浅谈《【原创】深度分析Twitter Heron》](https://gist.github.com/maosongfu/c3aeb1bb5eb7b39fcdc5)

[Flying faster with Twitter Heron](https://blog.twitter.com/2015/flying-faster-with-twitter-heron) 中文翻译版如下:
[Twitter发布新的大数据实时分析系统Heron](http://geek.csdn.net/news/detail/33750)

## storm

数据流的实时处理,数据到达时立即在内存中处理


拓扑 topology

- stream 数据流
- spout	数据流生成者
- bolt 运算
- 核心数据结构 tuple(包含一个或多个键值对的列表)

	1. declareOutputFields
	1. open
	1. nextTuple

- 集群的主要组成部分
	- nodes 服务器

- 高可靠性
storm 保证spout发出的每条消息都能"完全处理",这也是storm区别于其他系统的地方.比如yahoo的S4.
消息树

- **事物性拓扑**

多语言协议
每个tuple处理时都需要进行编解码,处理吞吐量有很大的影响.

- 高效
使用zeroMQ作为底层的消息队列,消息能快速处理


- spout
	- nextTuple
	- ask  成功处理
	- fail 处理失败
- bolts
封装所有的处理逻辑
过滤,聚合,查询数据库
	- `OutputFieldsDeclarer.declareStrean`  定义Stream
	- `OutputCollector.emit` 选择要发射的Stream
- Stream Groupings
定义一个stream应该如何分配数据给bolts上面的多个task


- storm 论文翻译

[Storm@Twitter - SIGMOD’14 (Jun, 2014)](http://dl.acm.org/citation.cfm?id=2595641)

[Streaming@Twitter - Bulletin of the IEEE Computer Society Technical Committee on Data Engineering (Jul, 2016)](http://sites.computer.org/debull/A15dec/p15.pdf)

[Twitter Heron: Stream Processing at Scale - SIGMOD’15 (May, 2015)](http://dl.acm.org/citation.cfm?id=2742788)

架构
扩展性
容错
可扩展的:容易增删
弹性:容错
可扩展
效率
易于管理:关键组件
开发者Nathan Marz 
2012年开源


**YARN**在hadoop上使用  storm[Storm On YARN](http://dongxicheng.org/mapreduce-nextgen/storm-on-yarn/) Storm On YARN带来的好处相比于将Storm部署到一个独立的集群中，Storm On YARN带来的好处很多，主要有以下几个：

- 好处
	- 弹性计算资源。 将Storm运行到YARN上后，Storm可与其他应用程序（比如MapReduce批处理应用程序）共享整个集群中的资源，这样，当Storm负载骤增时，可动态为它增加计算资源，而当负载减小时，可释放部分资源，从而将这些资源暂时分配给负载更重的批处理应用程序。

	- 共享底层存储。 Storm可与运行在YARN上的其他框架共享底层的一个HDFS存储系统，可避免多个集群带来的维护成本，同时避免数据跨集群拷贝带来的网络开销和时间延迟

	- 支持多版本。可同时将多个Storm版本运行YARN上，避免一个版本一个集群带来的维护成本

- 数据模型和架构
	1. Nimbus 主节点:  分配和协调
	2. worker nodes运行1个或多个worker processes.
	1. worker processes在jvm上运行,运行1个或多个executors.
	1. Executors有1个或多个tasks.工作真正在task上执行
每一个worker上运行一个Supervisor监督进程,和主节点通信.
一个task是spout或bolt.一个task和一个executor

数据分发策略
内部构件
Supervisor和主节点相互沟通,报告情况,空闲资源.协调公馆zooKeeper

- Supervisor 每个节点上有监控进程
	1. 心跳信息,报告节点正常,每15s
	1. 同步监控,观察任务分配的改变.每10s.
	1. 同步进程,管理worker processes


每一个worker包含两过程

1. worker receive thread
1. worker send thread.

每一个executor包含两个线程

1. user logic thread 从in queue获取进来的tuple,执行工作.
1. executor send thread.



语义:

1. 至多一次
1. 至少一次

有向非循环图(directed acyclic graph,DAG)
64-bit “message id”每个tuple上.provenance tree.
通过异或的方式处理.

处理错误情况

今后

1. 状态不是在zookeeper就是在硬盘中.worker继续工作.提高稳定性.
1. 当主节点出问题,继续工作
1. 一个task不和executor严格绑定,得到更好效果.

***
## Twitter Heron论文
[Twitter Heron: Stream Processing at Scale](http://dl.acm.org/citation.cfm?id=2742788)

1. 扩展性更好
1. 性能更好
1. 更容易调试
1. 易于管理

## storm问题
- 难于调试
storm大量组件的工作乱塞进一个处理进程.难于调试.Heron更清晰的map图.
- storm需要专门的硬件去运行topology
- 笨拙的管理机器.

Heron

1. 兼容stormAPI
1. 高性能,资源少,调试,扩展性,易于管理

- storm缺点
一个节点可以运行大量work进程,但是每个都能属于不同拓扑.


- Storm worker 架构局限性
	- worker设计复杂
	- 每个线程需要完成许多工作
	- 调用多层,复杂度的相互作用,导致调度不确定性.
	- 多种任务在一个JVM里运行.
	- 多个任务将日志写到同一个文件中.
	- 一个未处理的错误,将导致整个work错误
	- 资源调度,storm认为每个worker相同.利用率低
	- debug困难
	- 并行度提升,每个组件试图和其他组件联系.
	- storm使用多个线程和队列使任务在task和worker移动.每个tuple有4个线程.

- Storm Nimbus问题
	1. 容易成为瓶颈.worker不相互隔离,互有影响.
	1. Zookeeper使用限制了topology的数量.Zookeeper成为瓶颈.

- 缺少Backpressure
如果处理不了就丢弃

- 效率
	- 垃圾收集时间长
	- 队列竞争
	- 效率低


## Heron

减轻管理的复杂性

- 架构概述
	- Aurora 调度器(twitter自己的,没有另外实现.), 调度抽象
	- 每个topolopg包含多个containers.
	- 元数据保存在zookeeper
	- 热备份Topology Master
	- Topology Master
	- Metrics Manager
	- Heron Instances
	- 优点
		- 多个container可以运行在一台机器上
		- 根据资源进行调度
		- standby Topology Master 没有单点故障
		- 通讯使用协议缓冲

- Topology Master(TM)
管理拓扑,提供发现拓扑状态的单点信息.启动时创建临时节点.(???)
	- 避免多个Topology Master成为同一个拓扑的master.提供统一视图
	- 允许任何属于拓扑的节点发现TM
不涉及处理过程,不是瓶颈.

- Stream Manager(SM)
有效管理tuples路由
Heron Instance(HI)同本地的SM取得和发送数据.
k个Stream Manager间相互连接,比n个Instance间相互连接,降低了复杂度.


- Topology Backpressure
使用Backpressure机制动态调整数据流经topology的速率.
可以调整数据流的速率,不同的组件可以按照不同的速率运行.
如果流入速率过快,将建立起过长的buffer对列或者丢弃tuples.
- 实现方法
	- TCP Backpressure
	使用TCP窗口机制.SM和HI在container中通过TCP socket通信.HI处理慢了接收buffer将很快填满.SM意识到,传播.只有当慢的HI赶上进度才得以清除.
	**容易实现,效果不好,阻塞清理十分缓慢,性能下降**
	- Spout Backpressure(已经实现)
	SM降低spout速度.当spout发送缓存填满.SM发送消息让其他是SMs降速.当慢的HI赶上来,发送消息让其他SM继续工作.
	**可能不是最优,有缺点.但是不论topology深度如何,反应时间很短.**
	- Stage-by-Stage Backpressure
	控制信息通过SMs交换.

	- Backpressure 实现
	实现了Spout Backpressure,运行良好.当到达高点标记时触发Backpressure,直到到达低点标记.
	**避免迅速震荡**
	**tuple从spout发射出去,就不会放弃它.除非机器错误,使tuple失败更加有确定性.**
	**运行的速度和最慢的组件相当**



- Heron Instance
Heron Instance是一个JVM进程,只运行单一的工作.易于debug,log等.
数据传输的复杂性交给SM了.HI更加简单.

- 两种实现HI的方式:

	- Single-threaded实现HI
		- TCP和loacl SM通信,等待tuples.
		- tuple到达,处理
		- 处理后将tuple缓存
		- 缓存到达阈值,发送给local SM

		- 优点: 简单 
		- 缺点: user code 可能因为很多原因被阻塞
		(1)系统sleep(2)读写调用(3)同步原语

		阻塞不理想,阻塞时间不可以预知.不知HI状态是否正常.

	- Two-threaded 实现HI
		- Gateway thread
		通信和数据出入HI.和SM和metrics manager通信.接收达到的tuple
		- Task Execution thread
		运行user code
		两种方法open和prepare
		若是bolt,调用execute
		若是spout,调用nextTuple
		收集运行的信息
		- 通信Gateway和 Task Execution通过单向对列进行通信
		Gateway 通过data-in:将tuple送到Task Execution
		Task Execution通过data-out将tuple送到gateway
		Task Execution通过metrics-out将收集的信息发送给gateway.
		- 垃圾收集问题
		定期检查对列的容纳能力,适当改变对列的大小.




- Metrics Manager 特征管理
收集系统和用户特征,发送到内部的监控系统上.


- 启动顺序和故障方案

1. 提交topology后,调度器scheduler调度topology containers到一些机器.
1. Stream Manager (TM)在第一个containers出现,被zookeeper发现.
1. 同时其他container的Stream Manager联系Zookeeper去发现Stream Manager.SM和TM间定期发送心跳信息.
1. 分配physical plan:所有的SM相互联系后.分配spout 和bolts到不同的containers.
1. 分配完,SM得到整个physical plan从TM.便于SM相互发现.然后SM相互发现,组成互连网络.
1. 同时,HI发现本地Sm,下载physical plan.开始执行数据开始流经整个topology.
1. 为了安全TM将physical plan写入到Zookeeper避免自己实效.


- 错误情况


1. TM失败,重启从Zookeeper恢复状态.standby TM成为主TM.重启的TM成为standby.
1. SM失败.和TM联系恢复.其他SM从TM那发现新的SM.
1. HI失败.从SM那得到physical plan,确定spout or bolt.
1. container安排到其他机器上,按照上面的方式联系TM.恢复SM和HI.


- 总结

1. 资源提供清楚的抽象.
1. HI仅允许单一任务,容易debug
1. 对失败和减慢透明.颗粒收集信息,容易找出问题.
1. 组件级资源分配,组件分配特定资源,避免浪费.
1. Topology Master允许每个拓扑独立管理.一个拓扑不影响其他.
1. backpressure机制,实现输出结果的一致速率.
**关键机制使topology从一组容器迁移到另外一组.**
1. 无单点故障


- 生产上使用
Heron Tracker
Heron UI
Heron Viz

- 实验Word Count Topology实验175Kword

1. Heron **10-14X**倍加速比storm in all these experiments.
1. Heron latency is **5-15X** lower than that of the Storm
1. CPU usage of Heron is **2-3X** lower than that of the Storm,


- 总结

Heron, while delivering **6-14X** improvements in throughput, and
**5-10X** reductions in tuple latencies