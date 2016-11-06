# Storm分布式实时计算模式

简单说一个task相当于一个spout/bolt实例.

在本地模式下增加worker数量不能提高速度.本地模式是在同一个JVM进程中执行.
增加task和exector才有效.

- 数据流分组

- 锚定tuple
建立tuple和衍生出的tuple间的对应关系.下游的tuple通过应答确认,报错或超时加入tuple对列.
非锚定的tuple处理失败,原始的tuple不会重新发送.

## 1分布式单词计数

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

sed -i 's/backtype/org.apache/g'  *

sed -i 's/storm.blueprints.chapter1.v1/com.jz/g'  *


sed -i 's/com.jz.Utils.*/com.jz.Utils.*;/g' *


- 提交任务
```
storm jar  ./target/jztest-1.0.jar  com.jz.WordCountTopology
```

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


storm kill word-count-topology
storm deactivate word-count-topology


## 可靠性API


- spout可靠性
**msgId**
UUID msgId = UUID.randomUUID();
this.pending.put(msgId, values);
this.collector.emit(values, msgId);

- bolt可靠性
**tuple**

可靠的
collector.emit(tuple, new Values(word));


非可靠的tuple
this.collector.emit(new Values(word));


