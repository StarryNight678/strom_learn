# storm重要知识点

1. [关于Storm与JStorm的调度算法的讨论](http://m.blog.csdn.net/article/details?id=50433273)
1. [Storm 性能优化 例子](http://www.jianshu.com/p/f645eb7944b0)
1. [图书 Storm技术内幕与大数据实践 192页](http://detail.dangdang.com/23699059.html#catalog)
1. [Storm：大数据流式计算及应用实践](http://detail.dangdang.com/23668216.html#catalog)

## 性能提升

性能如何提升,提升了哪些方面?


[反压介绍](http://jobs.one2team.com/apache-storms/)

![](http://jobs.one2team.com/wp-content/uploads/2016/04/apache-750x566.png)

Storm 1.0.0说性能提升了16倍,延迟减少了60%

自动反压机制

[Storm 1.0.0 released](http://storm.apache.org/2016/04/12/storm100-released.html)

- Improved Performance

One of the main highlights in this release is a dramatice performance improvement over previous versions. Apache Storm 1.0 is **up to 16 times faster than previous versions, with latency reduced up to 60%**. Obviously topology performance varies widely by use case and external service dependencies, but for most use cases users can expect a 3x performance boost over earlier versions.

- Automatic Backpressure

In previous Storm versions, the only way to throttle the input to a topology was to enable ACKing and set topology.max.spout.pending. For use cases that don't require at-least-once processing guarantees, this requirement imposed a significant performance penalty.

Storm 1.0 includes a new automatic backpressure mechanism based on configurable high/low watermarks expressed as a percentage of a task's buffer size. If the high water mark is reached, Storm will slow down the topology's spouts and stop throttling when the low water mark is reached.

Storm's backpressure mechanism is implemented independently of the Spout API, so all existing Spouts are supported.

- zoonkeeper 是瓶颈.

Pacemaker - Heartbeat Server 自己处理心跳信息.

- HA Nimbus 多个Nimbus 自己选举


- 延迟

[讲解UI](http://www.malinga.me/reading-and-understanding-the-storm-ui-storm-ui-explained/)

Latency – 从收到tuple,到被标记为完成的时间.

![](http://www.malinga.me/wp-content/uploads/2015/04/Reading-and-Understanding-the-Storm-UI-Topology-stats.png)


Complete latency – The average time a Tuple “tree” takes to be completely processed by the Topology. A value of 0 is expected if no acking is done

完整的延迟:Tuple “tree” 完全处理的平均时间.标记为0表示,no acking.

![](http://www.malinga.me/wp-content/uploads/2015/04/Reading-and-Understanding-the-Storm-UI-Bolts.png)

Execute latency – The average time a Tuple spends in the execute method. The execute method may complete without sending an Ack for the tuple.

在执行方法中的平均时间


Process latency – The average time it takes to Ack a Tuple after it is first received. Bolts that join, aggregate or batch may not Ack a tuple until a number of other Tuples have been received.

tuple收到和确认的的平均时间.

1. Spout Complete Latency: the time a tuple is emitted until Spout.ack() is called.
1. Bolt Execution Latency: the time it take to run Bolt.execute().
1. Bolt Processing Latency: the time Bolt.execute() is called until the bolt acks the given input tuple.

[延迟讲解](http://stackoverflow.com/questions/33558613/storm-huge-discrepancy-between-bolt-latency-and-total-latency)
