# Storm开发


## 问题:
消息处理 best effort方式.
[可靠性 Guaranteeing Message Processing](http://storm.apache.org/releases/1.0.2/Guaranteeing-message-processing.html)

## 并行度

一个进程属于特定的topology。
进程启动一个或多个线程。
一个task认为是一个spout或者bolt实例。
默认一个executor分配一个task。
可以设置worker数量，executor数量和task数量。


## 编程

IDEA 运行storm-starter's

1. Follow the steps in storm-starter's: [Using storm-starter with IntelliJ IDEA](https://github.com/apache/storm/tree/master/examples/storm-starter#intellij-idea)
1. Open Maven's pom.xml file and remove <scope>provided</scope> line from storm dependency. This enables IntelliJ to compile storm dependency on build.
1. Go to /src/jvm/storm/starter/, right click on ExclamationTopology file and Run 'ExclamationTop....main()'


## 运行旧版程序
In Storm 1.0 , Java package naming moved from backtype.storm to org.apache.storm.
If you intend to run any topologies that used to run previous versions of Storm in Storm 1.0, you can do so by using one of two options:

You can rebuild the topology by renaming the imports of the backtype.storm package to org.apache in all of your topology code.

or

You can add config 

`Client.jartransformer.class = org.apache.storm.hack.StormShadeTransformer`

 to storm.yaml.

## 概念

本地模式

```
import org.apache.storm.LocalCluster;

LocalCluster cluster = new LocalCluster();
//关闭
cluster.shutdown();
```

创建topology
```
TopologyBuilder builder = new TopologyBuilder();

builder.setSpout("1", new TestWordSpout(true), 5);
builder.setSpout("2", new TestWordSpout(true), 3);
builder.setBolt("3", new TestWordCounter(), 3)
         .fieldsGrouping("1", new Fields("word"))
         .fieldsGrouping("2", new Fields("word"));
builder.setBolt("4", new TestGlobalCount())
         .globalGrouping("1");

Map conf = new HashMap();
conf.put(Config.TOPOLOGY_WORKERS, 4);

StormSubmitter.submitTopology("mytopology", conf, builder.createTopology());
```
本地debug模式,10s后停止.
```
TopologyBuilder builder = new TopologyBuilder();

builder.setSpout("1", new TestWordSpout(true), 5);
builder.setSpout("2", new TestWordSpout(true), 3);
builder.setBolt("3", new TestWordCounter(), 3)
         .fieldsGrouping("1", new Fields("word"))
         .fieldsGrouping("2", new Fields("word"));
builder.setBolt("4", new TestGlobalCount())
         .globalGrouping("1");

Map conf = new HashMap();
conf.put(Config.TOPOLOGY_WORKERS, 4);
conf.put(Config.TOPOLOGY_DEBUG, true);

LocalCluster cluster = new LocalCluster();
cluster.submitTopology("mytopology", conf, builder.createTopology());
Utils.sleep(10000);
cluster.shutdown();
```

spout the main method on spouts is `nextTuple. nextTuple `

[可靠性](http://storm.apache.org/releases/1.0.2/Guaranteeing-message-processing.html)
消息处理过程:
Storm offers several different levels of guaranteed message processing, includeing 

1. best effort
1. at least once
1. exactly once through Trident.

调度器
默认调度器和资源隔离的调度器.

1. Pluggable scheduler
1. Isolation Scheduler

[Nimbus HA Design document,Highly Available Nimbus Design](http://storm.apache.org/releases/1.0.2/nimbus-ha-design.html)
选举和状态保存

依赖
```
<dependency>
  <groupId>org.apache.storm</groupId>
  <artifactId>storm-core</artifactId>
  <version>1.0.2</version>
  <scope>provided</scope>
</dependency>
```


[CGroup Enforcement](http://storm.apache.org/releases/1.0.2/cgroups_in_storm.html)
Integration with Resource Aware Scheduler
CGroups can be used in conjunction with the Resource Aware Scheduler. CGroups will then enforce the resource usage of workers as allocated by the Resource Aware Scheduler. To use cgroups with the Resource Aware Scheduler, simply enable cgroups and be sure NOT to set storm.worker.cgroup.memory.mb.limit and storm.worker.cgroup.cpu.limit configs.


Pacemaker
当storm变大时,zoonkeeper成为瓶颈.维护一致性时导致大量的磁盘和网络开销.
Pacemaker是简单的键值存储.维护心跳信息,心跳信息不需要存储到硬盘中,在内存中存储.
Pacemaker作为单个守护程序实例运行，使其成为潜在的单点故障。

相比zoonkeeper使用的资源更少. Gigabit networking下,the real limit is likely around 2000-3000 nodes.


 
