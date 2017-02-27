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


## 程序
```java
public class SentenceSpout extends BaseRichSpout {
    private SpoutOutputCollector collector;
    private String[1] sentences = {
        "my dog has fleas",
        "don't have a cow man",
        "i don't think i like fleas"
    };
    private int index = 0;
    public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("sentence"));
    }
    public void open(Map config, TopologyContext context, 
            SpoutOutputCollector collector) {
        this.collector = collector;
    }
public void nextTuple() {
//没有可靠性
        this.collector.emit(new Values(sentences[index));
        index++;
        if (index >= sentences.length) {
            index = 0;
        }
        Utils.waitForMillis(1);
    }
}
```
```java
    private SpoutOutputCollector collector;
    private String[1] sentences = {
        "my dog has fleas",
        "don't have a cow man",
        "i don't think i like fleas"
    };
    private int index = 0;
    public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("sentence"));
    }
    public void open(Map config, TopologyContext context, 
            SpoutOutputCollector collector) {
        this.collector = collector;
    }
public void nextTuple() {
//没有可靠性
        this.collector.emit(new Values(sentences[index));
        index++;
        if (index >= sentences.length) {
            index = 0;
        }
        Utils.waitForMillis(1);
    }
}
//可靠性使用
   public void open(Map config, TopologyContext context, 
            SpoutOutputCollector collector) {
        this.collector = collector;
        this.pending = new ConcurrentHashMap<UUID, Values>();
    }
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
```java
public class SplitSentenceBolt extends BaseRichBolt{
    private OutputCollector collector;
    public void prepare(Map config, TopologyContext context, OutputCollector collector) {
        this.collector = collector;
    }
    public void execute(Tuple tuple) {
        String sentence = tuple.getStringByField("sentence");
        String[] words = sentence.split(" ");
        for(String word : words){
            this.collector.emit(new Values(word));
        }
    }
    public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("word"));
    }
}
```
```java
可靠性方法
  public void execute(Tuple tuple) {
        String sentence = tuple.getStringByField("sentence");
        String[] words = sentence.split(" ");
        for(String word : words){
            this.collector.emit(tuple, new Values(word));
        }
     this.collector.ack(tuple);
    }
```
```java
public class WordCountBolt extends BaseRichBolt{
    private OutputCollector collector;
    private HashMap<String, Long> counts = null;
    public void prepare(Map config, TopologyContext context, 
            OutputCollector collector) {
        this.collector = collector;
        this.counts = new HashMap<String, Long>();
    }
    public void execute(Tuple tuple) {
        String word = tuple.getStringByField("word");
        Long count = this.counts.get(word);
        if(count == null){
            count = 0L;
        }
        count++;
        this.counts.put(word, count);
        this.collector.emit(new Values(word, count));
    }
    public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("word", "count"));
    }}
```
```java
public class ReportBolt extends BaseRichBolt {
    private HashMap<String, Long> counts = null;
    public void prepare(Map config, TopologyContext context, OutputCollector collector) {
        this.counts = new HashMap<String, Long>();
    }
    public void execute(Tuple tuple) {
        String word = tuple.getStringByField("word");
        Long count = tuple.getLongByField("count");
        this.counts.put(word, count);
    }
    public void declareOutputFields(OutputFieldsDeclarer declarer) {        // this bolt does not emit anything    }
    @Override
    public void cleanup() {
        System.out.println("--- FINAL COUNTS ---");
        List<String> keys = new ArrayList<String>();
        keys.addAll(this.counts.keySet());
        Collections.sort(keys);
        for (String key : keys) {
            System.out.println(key + " : " + this.counts.get(key));        }
        System.out.println("--------------");
    }
}
```
```java
public class WordCountTopology {
    private static final String SENTENCE_SPOUT_ID = "sentence-spout";
    private static final String SPLIT_BOLT_ID = "split-bolt";
    private static final String COUNT_BOLT_ID = "count-bolt";
    private static final String REPORT_BOLT_ID = "report-bolt";
    private static final String TOPOLOGY_NAME = "word-count-topology";
    public static void main(String[] args) throws Exception {
        SentenceSpout spout = new SentenceSpout();
        SplitSentenceBolt splitBolt = new SplitSentenceBolt();
        WordCountBolt countBolt = new WordCountBolt();
        ReportBolt reportBolt = new ReportBolt();
        TopologyBuilder builder = new TopologyBuilder();
        builder.setSpout(SENTENCE_SPOUT_ID, spout)
.setNumTasks(4);
        // SentenceSpout --> SplitSentenceBolt
        builder.setBolt(SPLIT_BOLT_ID, splitBolt)
.setNumTasks(4)
                .shuffleGrouping(SENTENCE_SPOUT_ID);
        // SplitSentenceBolt --> WordCountBolt
        builder.setBolt(COUNT_BOLT_ID, countBolt)
                .fieldsGrouping(SPLIT_BOLT_ID, new Fields("word"));
                   // WordCountBolt --> ReportBolt
        builder.setBolt(REPORT_BOLT_ID, reportBolt)
                .globalGrouping(COUNT_BOLT_ID);
        Config config = new Config();
config.setNumWorkers(2);
config.put(Config.TOPOLOGY_DEBUG, false);
//本地模式
        LocalCluster cluster = new LocalCluster();
        cluster.submitTopology(TOPOLOGY_NAME, config, builder.createTopology());
        waitForSeconds(10);
        cluster.killTopology(TOPOLOGY_NAME);
        cluster.shutdown();
//提交方式
StormSubmitter.submitTopology(TOPOLOGY_NAME, config, builder.createTopology());
    }
}
```



## Spout
BaseRichSpout 是Ispout和IComponent 简单实现
open()在Ispout定义,Spout初始化调用.

## Bolt
BaseRichBolt 是IBolt和IComponent 简单实现
prepare()在IBolt定义,bolt初始化调用.
IBolt.cleanup() 不保证会执行

BaseBasicBolt 封装了读取 发射 执行的模式.

元组发送到`BasicOutputCollectorare`自动锚定.

##	流分组:

流分组定义了元组在bolt间分发的方式.

1.	shuffle group随机分组: 随机分
2.	fields group按字段分组: 按字段分组
3.	all grouping全复制分组: 发给所有的task
4.	globle grouping 全局分组: 唯一的task(最小task ID)
5.	none grouping不分组,随机,为将来预留.
6.	direct grouping指向分组: 指向型数据流上使用,执行组件.
7.	local or shffle grouping本地或随机分组: 随机类似.发送给同一个worker内的bolt task.
8.  **PartialKeyGrouping 部分关键字分组,考虑到下游bolt均衡情况.**

![](http://shiyanjuncn.b0.upaiyun.com/wp-content/uploads/2016/02/storm-topology-tasks.png)

### 自定义分组


```java
public interface CustomStreamGrouping
extends Serializable
List<Integer>	chooseTasks(int taskId, List<Object> values)
This function implements a custom stream grouping.
void prepare(WorkerTopologyContext context, GlobalStreamId stream, List<Integer> targetTasks)
Tells the stream grouping at runtime the tasks in the target bolt.
```

# 可靠性
## 锚定

**锚定tuple: 建立输入tuple和衍生出的tuple间的对应关系.下游的tuple可以进行应答确认,超时或报错.**

## acker

Storm有一个acker的特殊任务跟踪DAG图消息.当一个消息被创建时(Spout或bolt中)分配64位id.

每个消息都知道跟消息的ID,生成一个消息时,根消息的ID复制到消息中.

该Bolt调用OutputCollector.ack()时，Storm会做如下操作：
将anchor tuple列表中每个已经ack过的和新创建的Tuple的id做异或(XOR)。假定Spout发出的TupleID是tuple-id-0，该Bolt新生成的TupleID为tuple-id-1，那么，`tuple-id-0 XOR tuple-id-0 XOR tuple-id-1`

Storm根据该原始TupleID进行一致性hash算法，找到最开始Spout发送的那个acker，然后把上面异或后得出的ack val值发送给acker

- 调整可靠性
在某些特定的情况下，你或许想调整Storm的可靠性。例如，你并不关心数据是否丢失，或者你想看看后面是否有某个Bolt拖慢了Spout的速度？
那么，有三种方法可以实现：

1. 在build topology时，设置acker数目为0，即conf.setNumAckers(0);
2. 在Spout中，不指定messageId，使得Storm无法追踪；
3. 在Bolt中，使用Unanchor方式发射新的Tuple。


## 并行度

- executors <= tasks

[What is the “task” in Storm parallelism](http://stackoverflow.com/questions/17257448/what-is-the-task-in-storm-parallelism)

- 多个taks的原因:

1. 灵活的通过`rebalance`进行**不离线**系统伸缩.
	**task数量运行后不能变动**
1. 便于测试,大规模系统正确性.两个线程运行多个task.
1. 实际运行1个 executor 1 task  .

## Thrift

由 Facebook 开发的远程服务调用框架 Apache Thrift，它采用接口描述语言定义并创建服务，支持可扩展的跨语言服务开发，所包含的代码生成引擎可以在多种语言中，如 C++, Java, Python, PHP, Ruby, Erlang, Perl, Haskell, C#, Cocoa, Smalltalk 等创建高效的、无缝的服务

其传输数据采用二进制格式，相对 XML 和 JSON 体积更小，对于高并发、大数据量和多语言的环境更有优势。

[Apache Thrift - 可伸缩的跨语言服务开发框架](https://www.ibm.com/developerworks/cn/java/j-lo-apachethrift/)


## Hook 钩子

在storm内部插入自定义的代码来运行任意数量的事件.

storm的hook也是一个典型的钩子，当某些事情发生时(比如说执行execute方法，执行ack方法时)，相应的代码会自动被调用。

通过继承BaseTaskHook，并覆盖其方法来创建一个hook。


注册方法:

1. 在spout的open方法或者bolt的prepare方法中调用： 
TopologyContext.addTaskHook(new **Hook()) 
1. 在storm的配置文件中修改topology.auto.task.hooks项，这会自己注册到每一个spout和bolt。这种情况对于一些集成应用或者监控之类的有用。

- BaseTaskHook方法

```java
void	boltAck(BoltAckInfo info) 
void	boltExecute(BoltExecuteInfo info) 
void	boltFail(BoltFailInfo info) 
void	cleanup() 
void	emit(EmitInfo info) 
void	prepare(Map conf, TopologyContext context) 
void	spoutAck(SpoutAckInfo info) 
void	spoutFail(SpoutFailInfo info) 
```

```java
//先创建hook 
//这个hook很简单，就是当execute或者ack方法被调用时，将相应的信息打印出来：
package com.lujinhong.demo.storm.hook;
import backtype.storm.hooks.BaseTaskHook;
import backtype.storm.hooks.info.BoltAckInfo;
import backtype.storm.hooks.info.BoltExecuteInfo;
public class TraceTaskHook extends BaseTaskHook {
    @Override
    public void boltExecute(BoltExecuteInfo info) {
        super.boltExecute(info);
        System.out.println("executingTaskId:" + info.executingTaskId);
        System.out.println("executedLatencyMs:" + info.executeLatencyMs);
        System.out.println("execute msg:" + info.tuple.getString(0));
    }
    @Override
    public void boltAck(BoltAckInfo info) {
        super.boltAck(info);
        System.out.println("ackingTaskId:" + info.ackingTaskId);
        System.out.println("processLatencyMs:" + info.processLatencyMs);
        System.out.println("ack msg:" + info.tuple.getString(0));
    }
}
```



```java

```


```java

```