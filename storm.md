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

[Jstorm简介（参考storm的实时流式计算框架）](http://xinklabi.iteye.com/blog/2232257)

重要的文章Jstorm和Heron对比
[阿里中间件 揭开 Heron 性能面纱](http://jm.taobao.org/2016/08/04/heron-performance/)

他们告知 他们也就只能跑20w qps， 并且告诉 一个很大问题， container内部的stream－manager的瓶颈是50w qps， 也就是一个container所有内部通信和外部通信的总和上限就是50w。
性能分析

heron 一直号称是storm 10倍性能， 但heron 对比的对象是storm 0.8.2, 这是3年前的storm，是上一代的storm， 而最新版storm 1.0.2 早已经是storm 0.8.2 的十倍性能。

- heron在性能上存在2个致命缺陷
	- 失去了整个业界性能优化很大的一个方向， 流计算图优化。其核心思想就是让task尽量绑在一个进程中， 这样task之间的数据，可以直接走进程内通信，无需反序列化和序列化。
	- 为了提高稳定性， heron将每个task 独立成为一个进程， 则会产生一个新的问题，就是task之间的通信都不会有进程内通信， 所有task通信都是走网络， 都要经过序列化和反序列化， 引入了大量额外的计算.
	- 如果想要图优化， 则heron必须引入一层新的概念， 将多个task 链接到一个进程中， 但这个设计和heron的架构设计理念会冲突

每个container 的stream manager 会成为瓶颈， 一个container 内部的所有task 的数据（无论数据对外还是对内）通信都必须经过stream manager， 一个进程他的网络tps是有上限的， 而stream-manager的上限就是50w qps， 则表示一个container的内部通道和外部通道总和就是50w qps. 大家都必须抢这个资源。

原来的一次网络通信， 现在会变成3次网络通信， task －》 当前container的streammanager －》 目标container的stream manager －》 目标task

但今天的storm已经今非昔比，而jstorm更是不一样了。 jstorm反压早就做到了第三版， 当下游数据发生堆积时， 上游spout早就做限流降级， 应用无需申请超量的资源。

今天jstorm-on-yarn/jstorm-on-docker, jstorm-on-yarn 已经上线，就是在大集群上部署多个逻辑集群， 让大集群削峰填谷和资源隔离都非常成熟。

jstorm 0.9.0/storm0.9.5 开始就有了task粒度资源调度器，就是task按自己需要，申请多少cpu和多少内存就分配多少内存。**但jstorm从0.9.5 开始，调度的资源从task粒度恢复到worker粒度**， 原因是：

1. 集群跑一段时间后，容易出现碎片， 即有的机器上有cpu slot但没有内存 slot， 有的机器上有内存slot但没有cpu slot
1. 业务方很少遵守task 粒度去申请资源，反而偏爱worker粒度，简单粗暴方式。
2. 业务方有时超量申请资源， 只需要10个cpu slot，但却申请20个cpu

最终jstorm的方案是， 资源的粒度是到worker级别，但每台机器上配置动态监测， 实时根据负载情况调整自己的资源池策略， 很有效解决上述问题。

![](http://i.imgur.com/Yiyc94X.png)

另外Heron 资源上其实会引入1个小问题， 单个heron container会比单个jstorm／storm worker更消耗资源。
假设3个task运行在一个worker或container中，每个task 需要2g内存， 如果是jstorm或storm， 可能5g 内存就够了， 每个task之间可以临时share一下， 而heron container 则需要7g 甚至8g， 每个task 都需要2g，而且不能相互share， 另外一个container中还有streammanager／metricsmanager， 他们都需要内存。
原本worker级别的公共线程，在heron中现在需要在每个task进程中都配置上， 比如netty进程池，心跳线程， metrics 线程等等， 这些都在消耗cpu。

[深度分析Twitter Heron](http://www.longda.us/2015/06/04/heron/)
论文分析的很好.
最后总结：
Heron更适合超大规模的机器， 超过1000台机器以上的集群。 在稳定性上有更优异的表现， 在性能上，表现一般甚至稍弱一些，在资源使用上，可以和其他编程框架共享资源，但topology级别会更浪费一些资源。
另外应用更偏向于大应用，小应用的话，会多一点点资源浪费， 对于大应用，debug-ability的重要性逐渐提升。 另外对于task的设计， task会走向更重更复杂， 而JStorm的task是向更小更轻量去走。
未来JStorm可以把自动降级策略引入， 通过实现阿里妈妈的ASM， debug-ability应该远超过storm， 不会逊色于Heron， 甚至更强。

来自Twitter的反击
[浅谈《【原创】深度分析Twitter Heron》](https://gist.github.com/maosongfu/c3aeb1bb5eb7b39fcdc5)



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



[Flying faster with Twitter Heron](https://blog.twitter.com/2015/flying-faster-with-twitter-heron) 中文翻译版如下:
[Twitter发布新的大数据实时分析系统Heron](http://geek.csdn.net/news/detail/33750)


每一个worker包含两过程

1. worker receive thread
1. worker send thread.

每一个executor包含两个线程

1. user logic thread 从in queue获取进来的tuple,执行工作.
1. executor send thread.


今后

1. 状态不是在zookeeper就是在硬盘中.worker继续工作.提高稳定性.
1. 当主节点出问题,继续工作
1. 一个task不和executor严格绑定,得到更好效果.

***
## Twitter Heron论文
[Twitter Heron: Stream Processing at Scale](http://dl.acm.org/citation.cfm?id=2742788)

[论文Youtube视频讲解,英文](https://www.youtube.com/watch?v=pUaFOuGgmco)

中文视频:[witter Heron作者之一的符茂松也曾在QCon 2015 上海站介绍Heron，深入了解Heron的设计思路与架构之道。](https://v.qq.com/iframe/preview.html?vid=m03024pc6ql&amp)


1. 扩展性更好
1. 性能更好
1. 更容易调试
1. 易于管理



Heron

1. 兼容stormAPI
1. 高性能,资源少,调试,扩展性,易于管理


Storm架构
![storm archietecture](http://i.imgur.com/ZBHXsAV.png)

- storm缺点
一个节点可以运行大量work进程,但是每个都能属于不同拓扑.
难于调试,storm大量组件的工作乱塞进一个处理进程.

- Storm worker 架构局限性
	- worker设计复杂
	- 每个线程需要完成许多工作
	- 调用多层,复杂度的相互作用,导致调度不确定性.
	- 多种任务在一个JVM里运行.
	- 多个任务将日志写到同一个文件中.
	- 一个未处理的错误,将导致整个work错误
	- 资源调度,storm认为每个worker相同.利用率低.经常导致超量配置.
	- debug困难
	- 并行度提升,每个组件试图和其他组件联系.
	- storm使用多个线程和队列使任务在task和worker移动.每个tuple有4个线程.

worker:复杂的层级,难于调试,难于调优
![worker](http://i.imgur.com/6DD1FZU.png)


storm worker数据流

1. 队列争夺
1. 大量语言实现.不同语言实现不同的库
![storm worker数据流](http://i.imgur.com/wmMpFuK.png)

- Storm Nimbus问题

	- 容易成为瓶颈.worker不相互隔离,互有影响.
	- Server UI 负载重,比较慢.

- Zookeeper问题
负载重使用限制了topology的数量.Zookeeper成为瓶颈.

twitter 尝试分离zoonkeeper,分离支持数百节点.还不够.
![separate zoonkeeper](http://i.imgur.com/M25UBWV.png)

zoonkeeper负载
![load](http://i.imgur.com/9x0zVsa.png)


降低负载
![](http://i.imgur.com/xaS7N9a.png)

- 缺少Backpressure
如果处理不了就丢弃

- 效率
	- 垃圾收集时间长
	- 队列竞争
	- 效率低




## Heron

减轻管理的复杂性

架构
![](http://i.imgur.com/VPcaHQm.png)


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
管理拓扑,提供发现拓扑状态的单点信息.启动时创建和zookeeper沟通的唯一节点.
	- 避免多个Topology Master成为同一个拓扑的master.提供统一视图
	- 允许任何属于拓扑的节点发现TM
不涉及处理过程,不是瓶颈.
![TM](http://i.imgur.com/ra4Kudl.png)



- Stream Manager(SM)

**C++实现,复杂的内存操作.手动内存管理.Instance相当于手机的话,SM相当于手机的信号塔**


有效管理tuples路由
Heron Instance(HI)同本地的SM取得和发送数据.
k个Stream Manager间相互连接,比n个Instance间相互连接,降低了复杂度.

SM的一个网络的可视说明
![data-flow](http://twitter.github.io/heron/img/data-flow.png)


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

- 例子

![backpressure](http://twitter.github.io/heron/img/backpressure1.png)

A中的B3成为最慢的.A中的SM拒绝C和D的输入.导致填满C和D的socket buffers.

在这种情况下,反压机制介入:

A中的SM向B,C,D发送消息.B,C,D检查[physical-plan](http://twitter.github.io/heron/docs/concepts/topologies#physical-plan)减少输入匹配B3的速度.
当B3恢复,A中SM通知其他组件速度也将恢复正常.

![backpressure2](http://twitter.github.io/heron/img/backpressure2.png)



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

![](http://i.imgur.com/KYvTtLt.png)



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

![](http://i.imgur.com/KNuEDrD.png)



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

## Heron 其他

- 组件调度器
Scheduler — Heron requires a scheduler to run its topologies. It can be deployed on an existing cluster running alongside other big data frameworks. Alternatively, it can be deployed on a cluster of its own. Heron currently supports several scheduler options:

	- [Aurora 开源](http://twitter.github.io/heron/docs/operators/deployment/schedulers/aurora)
	[github.com/apache/aurora](https://github.com/apache/aurora)
	- Local
	- [Slurm 开源(Experimental)](http://twitter.github.io/heron/docs/operators/deployment/schedulers/slurm)
	- [YARN (Experimental)](http://twitter.github.io/heron/docs/operators/deployment/schedulers/yarn)


参考:
[Twitter 开源了数据实时分析平台 Heron](https://www.oschina.net/news/73811/twitter-open-source-heron)


