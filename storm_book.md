## Storm实战 构建大数据实时计算
***

ZeroMQ

sudo yum install maven

- 使用场景

1. 实时分析
1. 在线机器学习
1. 持续计算
1. 分布式RPC
1. ETL
保证每个消息都得到处理,速度快每个节点每秒百万次消息.

- 实体

1. 工作进程:每台机器上多个
1. 线程:每个进程多个
1. 任务:每个线程多个任务
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


# Storm初体验
 
 - 节点类型
 	- 主控节点master
 	Nimbus的后台程序,分发代码,分配任务,监控状态.
 	- 工作节点 worker
 	运行一个Supervisoer后台程序,监听Nimbus分配的任务.启动或停止进程.
 	一个Topology由分布在不同工作节点上的多个工作进程组成.
 	Nimbus和Supervisoer间协调通过zookeeper
	Nimbus和Supervisoer是快速失败和无状态.结束后,要么在zookeeper要么在硬盘上,拥有不可思议的稳定性.

## 构建Topology






