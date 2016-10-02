# storm learn


## 学习步骤
1. 环境搭建
1. 使用方式
1. API
1. 内部原理


## Jstorm
storm java 实现
[Jstorm github](https://github.com/alibaba/jstorm)
[封仲淹：Storm 2.0将会基于JStorm，阿里巴巴全程参与](http://www.infoq.com/cn/news/2015/11/jstorm-apache-alibaba)
[中文资料](https://github.com/alibaba/jstorm/wiki/JStorm-Chinese-Documentation)

## storm

数据流的实时处理,数据到达时立即在内存中处理


拓扑 topology

- stream 数据流
- spout	数据流生成者
- bolt 运算
- 核心数据结构 tuple(包含一个或多个键值对的列表)

	declareOutputFields
	open
	nextTuple

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
