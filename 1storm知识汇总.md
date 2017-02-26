# storm重要知识点

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

