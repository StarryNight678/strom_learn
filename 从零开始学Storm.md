# 从零开始学Storm

Storm简介

- 学习

# 1基本知识

- 应用方向:

1. 流处理
1. 连续计算
1. 分布式RPC

[storm-state](https://github.com/stormprocessor/storm-state) 管理大量的内存状态

0.8 版本引入State

# 2拓扑详解

TopologyBuilder


# 3组件详解

Map conf Storm配置

### IComponent 所有组件的接口

```java
void	declareOutputFields(OutputFieldsDeclarer declarer)
	    Declare the output schema for all the streams of this topology.
Map<String,Object>	getComponentConfiguration()
	   获取组件配置
```

### ISpout

storm同一个线程中执行ack,fail,nextTuple方法

1. void	ack(Object msgId)
	Storm has determined that the tuple emitted by this spout with the msgId identifier 1. has been fully processed.
1. void	activate()
	Called when a spout has been activated out of a deactivated mode.
1. void	close()
	Called when an ISpout is going to be shutdown.
1. void	deactivate()
	Called when a spout has been deactivated.
1. void	fail(Object msgId)
	The tuple emitted by this spout with the msgId identifier has failed to be fully processed.
1. void	nextTuple()
	When this method is called, Storm is requesting that the Spout emit tuples to the 1. output collector.
1. void	open(Map conf, TopologyContext context, SpoutOutputCollector collector)
	初始化

### IBolt

1. void	cleanup()
	Called when an IBolt is going to be shutdown.
1. void	execute(Tuple input)
	Process a single tuple of input.
1. void	prepare(Map stormConf, TopologyContext context, OutputCollector collector)
	 Called when a task for this component is initialized within a worker on the cluster.


### IRichSpout 和 IRichBolt

主要继承接口ISpout/IBolt和IComponent


```java
public Interface IRichSpout extends ISpout, IComponent{}
public interface IRichBolt extends IBolt, IComponent{}
```

### IBasicBolt

方法和IRichBolt相同

IBasicBolt 方法会自动出来acking机制.
All acking is managed for you. Throw a FailedException if you want to fail the tuple.

```java
public interface IBasicBolt extends IComponent
```

### IStateSpout &　IRichStateSpout

```java
public interface IStateSpout extends Serializable{}

void	close() 
void	nextTuple(StateSpoutOutputCollector collector) 
void	open(Map conf, TopologyContext context) 
void	synchronize(SynchronizeOutputCollector collector) 
```

`public interface IRichStateSpout extends IStateSpout, IComponent`


# 基本抽象类

### BaseComponent

```java
public abstract class BaseComponent
extends Object
implements IComponent
```


### BaseRichSpout

```java
public abstract class BaseRichSpout
extends BaseComponent
implements IRichSpout
```

### BaseRichBolt

```java
public abstract class BaseRichBolt extends BaseComponent
implements IRichBolt
```

###　BaseBasicBolt

```java
public abstract class BaseBasicBolt
extends BaseComponent
implements IBasicBolt
```

# 事务接口

```java
public interface IPartitionedTransactionalSpout<T>
extends IComponent

IPartitionedTransactionalSpout.Coordinator	getCoordinator(Map conf, TopologyContext context) 
IPartitionedTransactionalSpout.Emitter<T>	getEmitter(Map conf, TopologyContext context) 
```



### IBatchBolt接口

```java
void	execute(Tuple tuple) 
void	finishBatch() 
void	prepare(Map conf, TopologyContext context, 
	BatchOutputCollector collector, T id) 
```

当一个batch上的元组处理完调用finishBatch.

### BaseBatchBolt

```java
public abstract class BaseBatchBolt<T>
extends BaseComponent
implements IBatchBolt<T>
```





# 4Spout详解

拓扑使用广播拓扑,任何bolt失败,fail方法都会调用.

输入源JMS,Redis,Kafka



# 5Bolt详解

在客户端主机创建Bolt,序列化套拓扑,提交到主控节点.集群启动worker,反序列化Bolt,prepare调用,开始处理元组.

复合锚定,通过发射元组列表来实现.


declareOutputFields 都需要声明字段.
```java

```


# 6Zoonkeeper详解

Storm通过将状态信息保存在ZooKeeper中.

Nimbus写入分配信息.

Supervisor,task 通过从ZK 来读取状态信息.

Supervisor,task发送心跳信息到ZK.

P127 说明了ZK上Storm的目录.



# DRPC详解

分布式过程调用,DRPC详解

![](http://storm.apache.org/releases/1.0.1/images/drpc-workflow.png)

客户端发送请求给DRPC服务器,请求发送到Topology,服务器接收结果,返回给客户端.

客户端来看,DRPC和普通的RPC没有什么不同.

```java
LinearDRPCTopologyBuilder
```

1. LinearDRPCInputDeclarer	addBolt(IBasicBolt bolt)
 
1. LinearDRPCInputDeclarer	addBolt(IBasicBolt bolt, Number parallelism)
 
1. LinearDRPCInputDeclarer	addBolt(IBatchBolt bolt)
 
1. LinearDRPCInputDeclarer	addBolt(IBatchBolt bolt, Number parallelism)
 
1. LinearDRPCInputDeclarer	addBolt(IRichBolt bolt)

1. LinearDRPCInputDeclarer	addBolt(IRichBolt bolt, Number parallelism)

1. StormTopology	createLocalTopology(ILocalDRPC drpc)

1. StormTopology	createRemoteTopology()

```java
public class BasicDRPCTopology {
  public static class ExclaimBolt extends BaseBasicBolt {
    @Override
    public void execute(Tuple tuple, BasicOutputCollector collector) {
      String input = tuple.getString(1);
      collector.emit(new Values(tuple.getValue(0), input + "!"));
    }
    @Override
    public void declareOutputFields(OutputFieldsDeclarer declarer) {
      declarer.declare(new Fields("id", "result"));
    }
  }
  public static void main(String[] args) throws Exception {
    LinearDRPCTopologyBuilder builder = new LinearDRPCTopologyBuilder("exclamation");
    builder.addBolt(new ExclaimBolt(), 3);
    Config conf = new Config();
    if (args == null || args.length == 0) {
      try (LocalDRPC drpc = new LocalDRPC();
           LocalCluster cluster = new LocalCluster()) {
        cluster.submitTopology("drpc-demo", conf, builder.createLocalTopology(drpc));
        for (String word : new String[]{ "hello", "goodbye" }) {
          System.out.println("Result for \"" + word + "\": " + drpc.execute("exclamation", word));
        }
        Thread.sleep(10000);
      }
    }
    else {
      conf.setNumWorkers(3);
      StormSubmitter.submitTopologyWithProgressBar(args[0], conf, builder.createRemoteTopology());
    }
  }
}
```

- 客户端例子

DRPCClient,client.execute

```java
public class DRPCClientDemo {
    public static void main(String[] args) throws TException, DRPCExecutionException {
        DRPCClient client = new DRPCClient("hd181", 3772);
        String result = client.execute("drpcFunc", "aaa");
        System.out.println(result);
    }
}
```

bolt接收连个参数,id和请求参数.发送结果[id,result].所有的元组都必须包含id作为第一个字段.



# 事物拓扑

事务拓扑0.7 版本引入特性,0.8版本封装为Trident.

将元组封装为Batch实现.

可以指定某些bolt为Committer,保证Committer的finishBatch操作按照严格不降序的顺序实现.


实现:

1. 处理阶段,并行处理Batch
1. 提交阶段,严格有序.

storm自动处理以下的事情:

1. 管理状态: id,batch元数据.
1. 协调事务: 决策相关
1. 故障检测: 重发Batch
1. 封装批处理API:

kafka系统作为消息队列系统.

# Trident详解

### 具有功能

1. 连接
1. 聚合
1. 分组
1. 函数
1. 过滤器

有一致性和恰好一次语义.

- IBatchSpout

```
public interface IBatchSpout
extends ITridentDataSource
```
1.  void	ack(long batchId) 
1.  void	close() 
1.  void	emitBatch(long batchId, TridentCollector collector) 
1.  Map<String,Object>	getComponentConfiguration() 
1.  Fields	getOutputFields() 
1.  void	open(Map conf, TopologyContext context) 


- FixedBatchSpout

```java
public class FixedBatchSpout
extends Object
implements IBatchSpout
```

1.  void	ack(long batchId) 
1.  void	close() 
1.  void	emitBatch(long batchId, TridentCollector collector) 
1.  Map<String,Object>	getComponentConfiguration() 
1.  Fields	getOutputFields() 
1.  void	open(Map conf, TopologyContext context) 
1.  void	setCycle(boolean cycle) 

- each()

- persistentAggregate 存储更新聚合结果

```java
TridentState wordCounts = topology.newStream("spout1", spout)
.parallelismHint(16)
.each( new Fields("sentence"),new Split(), new Fields("word") )
.groupBy(new Fields("word"))
.persistentAggregate(new MemoryMapState.Factory(),new Count(), new Fi("count"))
.parallelismHint(16);
```


### 高性能执行一个拓扑

1. 读写状态操作,自动Batch化.
1. 聚合器高度优化.发送元组前会进行局部聚合.


### 状态 State

1. 每个batch有唯一ID
1. 状态更新在Batch间顺序执行

5种类型的操作

1. 本地分区: 应用本地到每个分区
1. 重新分区: 重新分区一个流,不改变内容
1. 聚合:     网络传输是操作一部分
1. 流分组
1. 合并与连接

### 本地分区

```java
public abstract class BaseFunction
extends BaseOperation
implements Function
```

- 函数

	输入一些字段,发射零个或多个元组作为输出.

	输出元组字段附加到原输入元组字段后面.

- 过滤器

	一个元组作为输入,决定是否保留元组.

- 分区聚合

每个分区运行一个函数,输出元组会替换输入元组

1. CombinerAggregator返回一个单一输出字段单一元组.
	网络传输前进行部分聚合来自动优化.
	**CombinerAggregator 效率较高**

1. ReducerAggregator


1.  Interface Aggregator<T>

	1.  void	aggregate(T val, TridentTuple tuple, TridentCollector collector) 
	1.  void	complete(T val, TridentCollector collector) 
	1.  T	init(Object batchId, TridentCollector collector) 




- 状态查询和分区持久化

stateQuery 用于查询状态
分区持久化 partitionPersist  用于更新状态源

- 投影

投影操作值保留操作中指定的字段.

### 重新分区

重新分区需要网络传输

1. shuffle方法,随机,数据均匀分配.
1. broadcast  元组复制到所有目标分区.
1. partitionBy 接收输入字段,根据字段进行语义分区.总是发送到同一个目标分区.
1. global 元组发送到相同的分区
1. batchGlobal 所有元组发送到相同的分区,不同Batch可能发送到不同分区.
1. partition 自定义实现


### 聚合


### 流分组


### 合并与连接


topology.merge(stream1,stream2,stream3,...)

合并以第一个流的输出字段来命名

第二个方法

topology.join()



## Trident状态

仅处理一次,快速,持续聚合方法



## 事务

每个Batch指定唯一一个id.

1. tid相同,batch相同
1. 元组batch间没有重叠.

失败时必须发送完全相同的Batch

- 处理逻辑:

1. tid相同,跳过更新.
1. tid不同,进行更新.

## 不透明事务

不能保证不同tid,Batch保持不变.
保证,Batch间没有重叠.每个元组只能在一个batch中成功处理.

不透事务,失去源节点也是容错的. 实现了一次且仅一次的语义.


需要保存

```
Value=4
prevalue=1
tid=2
```

1. 当前tid与数据库不同

现在 tid=3,count=2;
更新:
```
Value=6
prevalue=4
tid=3
```


1. 当前tid与数据库相同

现在 tid=2,count=2;
```
Value=3   (prevalue+count)
prevalue=4
tid=2
```

##  非事务Spout

没有保证



## 实现恰好一次语义

1. state 事务 只能与 spout 事务类型 相结合.
1. state 不透明事务 可以与spout 事务或不透明事务.


State只有两个方法 begincommit 和 commit




# 内部实现
# 相关项目
# 企业应用案例

Kafka