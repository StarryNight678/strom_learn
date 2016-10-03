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

## Heron Twitter新的流处理利器(开源了)

Heron
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


