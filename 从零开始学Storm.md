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
# Trident详解
# 内部实现
# 相关项目
# 企业应用案例

Kafka