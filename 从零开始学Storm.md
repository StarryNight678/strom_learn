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

在客户端主机创建Bolt,序列化到拓扑,提交到主控节点.集群启动worker,反序列化Bolt,prepare调用,开始处理元组.

复合锚定,通过发射元组列表来实现.


declareOutputFields 都需要声明字段.



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

Trident 有5种类型的操作

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

- 投影 **projection**

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

mystream.aggregator

### 流分组

groupBy

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

需要保存以前值.

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

![](https://github.com/nathanmarz/storm/wiki/images/spout-vs-state.png)


State只有两个方法 begincommit 和 commit


[TridentTopology](http://storm.apache.org/releases/1.0.2/javadocs/index.html)

TridentTopology方法

Stream  newStream(String txId, IBatchSpout spout) 

[Stream](http://storm.apache.org/releases/1.0.2/javadocs/index.html)



1. GroupedStream groupBy(Fields fields)
1. Stream  filter(Filter filter)
1. Stream  each(Fields inputFields, Filter filter)
1. Stream  each(Fields inputFields, Function function, Fields functionFields) 
1. Stream  parallelismHint(int hint)
    Applies a parallelism hint to a stream.
1. Stream  stateQuery(TridentState state, Fields inputFields, QueryFunction function, Fields functionFields) 
1. Stream  stateQuery(TridentState state, QueryFunction function, Fields functionFields) 
1. TridentState  persistentAggregate(StateFactory stateFactory,
    CombinerAggregator agg, Fields functionFields) 

- StateFactory

```java
public interface StateFactory
extends Serializable

State makeState(Map conf, IMetricsContext metrics, 
  int partitionIndex, int numPartitions) 
```

```java
public abstract class BaseStateUpdater<S extends State>
extends BaseOperation
implements StateUpdater<S>
```


persistentAggregate  是  partitionPersist上的另外一层抽象.
通过利用Trident聚合器来更新源状态.

public interface MapState<T>
extends ReadOnlyMapState<T>

void  multiPut(List<List<Object>> keys, List<T> vals) 
List<T> multiUpdate(List<List<Object>> keys, List<ValueUpdater> updaters) 


public class MemoryMapState<T>
extends Object
implements Snapshottable<T>, ITupleCollection, MapState<T>, RemovableMapState<T>

1. void  beginCommit(Long txid) 
1. void  commit(Long txid) 
1. T get() 
1. Iterator<List<Object>>  getTuples() 
1. List<T> multiGet(List<List<Object>> keys) 
1. void  multiPut(List<List<Object>> keys, List<T> vals) 
1. void  multiRemove(List<List<Object>> keys) 
1. List<T> multiUpdate(List<List<Object>> keys, List<ValueUpdater> updaters) 
1. void  set(T o) 
1. T update(ValueUpdater updater) 


public interface IBackingMap<T>

1. List<T> multiGet(List<List<Object>> keys) 
1. void  multiPut(List<List<Object>> keys, List<T> vals) 


Trident提供了一个QueryFunction接口用来实现Trident中在一个source state上查询的功能。

```
public interface QueryFunction<S extends State,T>
extends EachOperation

List<T> batchRetrieve(S state, List<TridentTuple> args) 
void  execute(TridentTuple tuple, T result, TridentCollector collector) 
```

- BaseQueryFunction

```
public abstract class BaseQueryFunction<S extends State,T>
extends BaseOperation
implements QueryFunction<S,T>
```

同时还提供了一个StateUpdater来实现Trident中更新source state的功能。

```java
public abstract class BaseStateUpdater<S extends State>
extends BaseOperation
implements StateUpdater<S>
```


### Implementing Map States

在Trident中实现MapState是非常简单的，它几乎帮你做了所有的事情。OpaqueMap, TransactionalMap, 和 NonTransactionalMap 类实现了所有相关的逻辑，包括容错的逻辑。你只需要将一个**IBackingMap**的实现提供给这些类就可以了。IBackingMap接口看上去如下所示：

Trident还提供了一种CachedMap类来进行自动的LRU cache。


大家可以看看 [MemcachedState](https://github.com/nathanmarz/trident-memcached/blob/master/src/jvm/trident/memcached/MemcachedState.java)
的实现，从而学习一下怎样将这些工具组合在一起形成一个高性能的MapState实现。MemcachedState是允许大家选择使用opaque transactional, transactional, 还是 non-transactional 语义的。





# 内部实现

查看struct内容

[storm.thrift](https://github.com/StarryNight678/storm/blob/master/storm-core/src/storm.thrift)

通过nimbus的Thrift接口完成Jar上传.



# Kafka

Kafka是由LinkedIn开发的一个分布式的消息系统，使用Scala编写


Kafka是一种分布式的，基于发布/订阅的消息系统。主要设计目标如下：

1. 以时间复杂度为O(1)的方式提供消息持久化能力，
    即使对TB级以上数据也能保证常数时间复杂度的访问性能。
1. 高吞吐率。即使在非常廉价的商用机器上也能做到单机支持每秒100K条以上消息的传输。
1. 支持Kafka Server间的消息分区，及分布式消费，同时保证每个Partition内的消息顺序传输。
1. 同时支持离线数据处理和实时数据处理。
1. Scale out：支持在线水平扩展。

##　拓扑结构:

Broker
Kafka集群包含一个或多个服务器，这种服务器被称为broker

Topic
每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic。**（物理上不同Topic的消息分开存储，逻辑上一个Topic的消息虽然保存于一个或多个broker上但用户只需指定消息的Topic即可生产或消费数据而不必关心数据存于何处）**

Partition
Parition是物理上的概念，每个Topic包含一个或多个Partition.

Producer
负责发布消息到Kafka broker

Consumer
消息消费者，向Kafka broker读取消息的客户端。

Consumer Group
每个Consumer属于一个特定的Consumer Group（可为每个Consumer指定group name，若不指定group name则属于默认的group）。


![](http://cdn1.infoqstatic.com/statics_s1_20170221-0307u1/resource/articles/kafka-analysis-part-1/zh/resources/0310020.png)


如上图所示，一个典型的Kafka集群中包含若干Producer（可以是web前端产生的Page View，或者是服务器日志，系统CPU、Memory等），若干broker（Kafka支持水平扩展，一般broker数量越多，集群吞吐率越高），若干Consumer Group，以及一个Zookeeper集群。Kafka通过Zookeeper管理集群配置，选举leader，以及在Consumer Group发生变化时进行rebalance。Producer使用push模式将消息发布到broker，Consumer使用pull模式从broker订阅并消费消息。

Topic & Partition

**每个Partition在物理上对应一个文件夹**
不同的消息可以并行写入不同broker的不同Partition里.

Topic在逻辑上可以被认为是一个queue，每条消费都必须指定它的Topic，可以简单理解为必须指明把这条消息放进哪个queue里。为了使得Kafka的吞吐率可以线性提高，物理上把Topic分成一个或多个Partition，每个Partition在物理上对应一个文件夹，该文件夹下存储这个Partition的所有消息和索引文件。若创建topic1和topic2两个topic，且分别有13个和19个分区，则整个集群上会相应会生成共32个文件夹（本文所用集群共8个节点，此处topic1和topic2 replication-factor均为1），如下图所示。


![](http://cdn1.infoqstatic.com/statics_s1_20170221-0307u1/resource/articles/kafka-analysis-part-1/zh/resources/0310022.png)


这个log entries并非由一个文件构成，而是分成多个segment，每个segment以该segment第一条消息的offset命名并以“.kafka”为后缀。另外会有一个索引文件，它标明了每个segment下包含的log entry的offset范围，如下图所示。

**因为每条消息都被append到该Partition中，属于顺序写磁盘，因此效率非常高（经验证，顺序写磁盘效率比随机写内存还要高，这是Kafka高吞吐率的一个很重要的保证）**

对于传统的message queue而言，一般会删除已经被消费的消息，而Kafka集群会保留所有的消息，无论其被消费与否。当然，因为磁盘限制，不可能永久保留所有数据（实际上也没必要），因此Kafka提供两种策略删除旧数据。一是基于时间，二是基于Partition文件大小。例如可以通过配置$KAFKA_HOME/config/server.properties，让Kafka删除一周前的数据，也可在Partition文件超过1GB时删除旧数据，配置如下所示。

同一Topic的一条消息只能被同一个Consumer Group内的一个Consumer消费，但多个Consumer Group可同时消费这一消息。

- 为何使用消息系统

解耦
在项目启动之初来预测将来项目会碰到什么需求，是极其困难的。消息系统在处理过程中间插入了一个隐含的、基于数据的接口层，两边的处理过程都要实现这一接口。这允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。

冗余
有些情况下，处理数据的过程会失败。除非数据被持久化，否则将造成丢失。消息队列把数据进行持久化直到它们已经被完全处理，通过这一方式规避了数据丢失风险。许多消息队列所采用的"插入-获取-删除"范式中，在把一个消息从队列中删除之前，需要你的处理系统明确的指出该消息已经被处理完毕，从而确保你的数据被安全的保存直到你使用完毕。

扩展性
因为消息队列解耦了你的处理过程，所以增大消息入队和处理的频率是很容易的，只要另外增加处理过程即可。不需要改变代码、不需要调节参数。扩展就像调大电力按钮一样简单。

灵活性 & 峰值处理能力
在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见；如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。

可恢复性
系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。

顺序保证
在大多使用场景下，数据处理的顺序都很重要。大部分消息队列本来就是排序的，并且能保证数据会按照特定的顺序来处理。Kafka保证一个Partition内的消息的有序性。

缓冲
在任何重要的系统中，都会有需要不同的处理时间的元素。例如，加载一张图片比应用过滤器花费更少的时间。消息队列通过一个缓冲层来帮助任务最高效率的执行———写入队列的处理会尽可能的快速。该缓冲有助于控制和优化数据流经过系统的速度。

异步通信
很多时候，用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们。


- Redis 轻量级队列

Redis是一个基于Key-Value对的NoSQL数据库，开发维护很活跃。虽然它是一个Key-Value数据库存储系统，但它本身支持MQ功能，所以完全可以当做一个轻量级的队列服务来使用。对于RabbitMQ和Redis的入队和出队操作，各执行100万次，每10万次记录一次执行时间。测试数据分为128Bytes、512Bytes、1K和10K四个不同大小的数据。实验表明：入队时，当数据比较小时Redis的性能要高于RabbitMQ，而如果数据大小超过了10K，Redis则慢的无法忍受；出队时，无论数据大小，Redis都表现出非常好的性能，而RabbitMQ的出队性能则远低于Redis。

- ZeroMQ

ZeroMQ号称最快的消息队列系统，尤其针对大吞吐量的需求场景。ZeroMQ能够实现RabbitMQ不擅长的**高级/复杂的队列**，但是开发人员需要自己组合多种技术框架，技术上的复杂度是对这MQ能够应用成功的挑战。ZeroMQ具有**一个独特的非中间件的模式**，你不需要安装和运行一个消息服务器或中间件，因为你的应用程序将扮演这个服务器角色。你只需要简单的引用ZeroMQ程序库，可以使用NuGet安装，然后你就可以愉快的在应用程序之间发送消息了。但是ZeroMQ仅提供非持久性的队列，也就是说如果宕机，数据将会丢失。其中，Twitter的Storm 0.9.0以前的版本中默认使用ZeroMQ作为数据流的传输（Storm从0.9版本开始同时支持ZeroMQ和Netty作为传输模块）。


- Kafka/Jafka

Kafka是Apache下的一个子项目，是一个高性能跨语言分布式发布/订阅消息队列系统，而Jafka是在Kafka之上孵化而来的，即Kafka的一个升级版。具有以下特性：快速持久化，可以在O(1)的系统开销下进行消息持久化；高吞吐，在一台普通的服务器上既可以达到10W/s的吞吐速率；完全的分布式系统，Broker、Producer、Consumer都原生自动支持分布式，自动实现负载均衡；**支持Hadoop数据并行加载**，对于像Hadoop的一样的日志数据和离线分析系统，但又要求实时处理的限制，这是一个可行的解决方案。Kafka通过Hadoop的并行加载机制统一了在线和离线的消息处理。Apache Kafka相对于ActiveMQ是一个非常轻量级的消息系统，除了性能非常好之外，还是一个工作良好的分布式系统。