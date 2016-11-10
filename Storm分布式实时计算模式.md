# Storm分布式实时计算模式

# 1分布式单词计数

## 并行度

- worker

1. 一个workder属于特定的topology。
1. 进程启动一个或多个executor。

- executor

1. 一个 executor 是由 worker 进程生成的一个线程。
1. executor 中可能会有一个或者多个 task
1. 一个task位同一个组件服务spout/bolt。
1. 默认一个executor分配一个task。

- task

1. task 是实际执行数据处理的最小工作单元（注意，task 并不是线程） 
1. 在拓扑的整个生命周期中每个组件的 task 数量都是保持不变的，不过每个组件的 executor 1. 数量却是有可能会随着时间变化。



一个executor的task执行相同的spout/bolt
可以设置worker数量，executor数量和task数量。


简单说一个task相当于一个spout/bolt实例.

在本地模式下增加worker数量不能提高速度.本地模式是在同一个JVM进程中执行.
增加task和exector才有效.

- 数据流分组

- 锚定tuple
建立tuple和衍生出的tuple间的对应关系.下游的tuple通过应答确认,报错或超时加入tuple对列.
非锚定的tuple处理失败,原始的tuple不会重新发送.



tuple一个或多个键值对的列表

- mvn构建项目进行测试
```
mvn archetype:generate -DgroupId=com.jz -DartifactId=jztest -Dpackage=com.jz -Dversion=1.0 -DarchetypeCatalog=internal

mvn package 

java -cp target/helloworld-1.0-SNAPSHOT.jar jz.App
```

- storm依赖
```
    <dependency>
      <groupId>org.apache.storm</groupId>
      <artifactId>storm-core</artifactId>
      <version>1.0.2</version>
      <scope>provided</scope>
    </dependency>
```
sed -i 's/import storm.blueprints.chapter1.v1.*;//g' *

sed -i 's/storm.blueprints.chapter1.v4/com.jz/g'  *
sed -i 's/backtype/org.apache/g'  *

sed -i 's/storm.blueprints.chapter1.v1/com.jz/g'  *


sed -i 's/storm.blueprints.utils.Utils;/com.jz.Utils.*;/g' *



## SentenceSpout


- declareOutputFields
所有的组件必须实现接口`declareOutputFields`,告诉storm会发射哪些流.数据流的tuple包括哪些字段.
open方法ISpout接口类中定义.所有的spout必须初始化调用.

- open
```
public void open(
Map config storm配置信息 , 
TopologyContext context topology中组件信息,
SpoutOutplsutCollector collector对象提供理论发射tuple方法
)
```

- nextTuple方法

## SplitSentenceBolt

- public void prepare(Map config, TopologyContext context, OutputCollector collector)
初始化

- public void declareOutputFields(OutputFieldsDeclarer declarer)
声明输出流,每个流包含字段word

- public void execute(Tuple tuple)
**核心功能**
每接收一个tuple调用该方法.

- WordCountBolt单词计数
- ReportBolt.java  上报
public void cleanup() storm在终止一个bolt前调用该方法.
不能保证一定会执行.

- 主函数
TopologyBuilder builder = new TopologyBuilder();



- 注意
topology在发布时,所有的组件首先进行序列化.如果在序列化前实现了无法序列化的实例常量.就会抛出异常.
本地模式,运行在一个JVM里面.  设置进程数量不会起作用.



## Group
1. Shuffle Grouping：随机分组，跨多个Bolt的Task
1. Fields Grouping： 按字段分组
1. Partial Key Grouping：与Fields grouping 类似，根据指定的Field的一部分进行分组分发，能够很好地实现Load balance，将Tuple发送给下游的Bolt对应的Task，特别是在存在数据倾斜的场景，使用 Partial Key  grouping能够更好地提高资源利用率
1. All Grouping：所有Bolt的Task都接收同一个Tuple（这里有复制的含义）
1. Global Grouping：所有的流都指向一个Bolt的同一个Task（也就是Task ID最小的）
1. None Grouping：不需要关心Stream如何分组，等价于Shuffle grouping
1. Direct Grouping：由Tupe的生产者来决定发送给下游的哪一个Bolt的Task  ，这个要在实际开发编写Bolt代码的逻辑中进行精确控制
1. Local or Shuffle rouping：(本地随机分组).将tuple分发到同一个worker的bolt task减少网络流量.其他情况下采用随机分组.


## topology命令
1. storm jar 提交
1. storm kill 关闭
1. storm deactivate topology_name  停止发送tuple
1. storm activate topology_name    恢复发送tuple
1. storm rebalancce topology_name  
1. storm remoteconfvalue topology_name  查看公共参数
1. storm list  查看运行的topology
1. storm  monitor   topology_name 查看拓扑组件
1. storm  monitor  -m  split-bolt word-count-topology 查看具体组件的执行情况

storm kill word-count-topology
storm deactivate word-count-topology


## 可靠性API


- spout可靠性
**msgId**
```
public void nextTuple() {
    Values values = new Values(sentences[index]);
    UUID msgId = UUID.randomUUID();
    this.pending.put(msgId, values);
    this.collector.emit(values, msgId);
    index++;
    if (index >= sentences.length) {
        index = 0;
    }
    Utils.waitForMillis(1);
}

public void ack(Object msgId) {
      this.pending.remove(msgId);
  }
  
public void fail(Object msgId) {
      this.collector.emit(this.pending.get(msgId), msgId);
  }   
```
spout接收tuple树上的消息。成功调用ack 失败调用fail。

- bolt可靠性
**tuple**

可靠的
collector.emit(tuple, new Values(word));


非可靠的tuple
this.collector.emit(new Values(word));


this.collector.ack(tuple);
this.collector.emit(tuple, new Values(word, count));

# 2配置storm集群


- supervisor进程
supervisor进程和worker进程工作在不同JVM进程上.

- zoonkeeper
提供分布式许多机制,如leader选举,分布锁,队列

nimbus和supervisor通信主要是靠zoonkeeper的状态变更和监控通知来处理.采用轻量级通信.
**重量级通信采用Thrift.**
topology组件间通信最影响效率地方,底层进行.经过优化.

- DRPC
请求-响应 范式

?????

storm快速失败,需要在出现错误时进行重启.  upstart系统非常适合该场景.
起几个端口就是控制运行几个worker进程.




## rebalance
storm在多个worker间重新平均分配任务.不需要关闭或重新提交.
新节点加入需要该命令.

```
jason@jason-HP:~/apache-storm-1.0.2/conf$ storm rebalance
Syntax: [storm rebalance topology-name [-w wait-time-secs] [-n new-num-workers] [-e component=parallelism]*]

    Sometimes you may wish to spread out where the workers for a topology
    are running. For example, let's say you have a 10 node cluster running
    4 workers per node, and then let's say you add another 10 nodes to
    the cluster. You may wish to have Storm spread out the workers for the
    running topology so that each node runs 2 workers. One way to do this
    is to kill the topology and resubmit it, but Storm provides a "rebalance"
    command that provides an easier way to do this.

    Rebalance will first deactivate the topology for the duration of the
    message timeout (overridable with the -w flag) and then redistribute
    the workers evenly around the cluster. The topology will then return to
    its previous state of activation (so a deactivated topology will still
    be deactivated and an activated topology will go back to being activated).

    The rebalance command can also be used to change the parallelism of a running topology.
    Use the -n and -e switches to change the number of workers or number of executors of a component
    respectively.
```

现有拓扑不会变化,在添加新节点时需要重新调度.先将拓扑deactivate
进行重新调度:

storm rebalance word-count-topology -w 3 -n 4 -e sentence-spout=4 -e split-bolt=4
-e 调整得是executor个数

## storm必行度






# 3trident和传感器数据

trident按照批处理数据.

[[翻译][Trident] Storm Trident 教程](http://blog.csdn.net/derekjiang/article/details/9126185)
Trident在处理输入stream的时候会把输入转换成若干个tuple的batch来处理.比如说，输入的sentence stream可能会被拆分成如下的batch：

![](https://github.com/nathanmarz/storm/wiki/images/batched-stream.png)



# 4实时趋势分析
# 5实时图形分析
# 6分工智能
# 7整合Druid进行金融分析
# 8自然语言处理
# 9Hadoop上部署Storm进行广告分析
# 10云环境下Storm