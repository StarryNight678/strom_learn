# 从零开始学Storm

Storm简介

- 学习

1. 容错
1. 并行度
1. javadoc
1. 序列化
1. 钩子


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

IComponent 所有组件的接口

```java
void	declareOutputFields(OutputFieldsDeclarer declarer)
	Declare the output schema for all the streams of this topology.
Map<String,Object>	getComponentConfiguration()
	获取组件配置
``


AbstractEsBolt, AbstractHBaseBolt, AbstractHdfsBolt, AbstractJdbcBolt, AbstractMongoBolt, AbstractRankerBolt, AbstractRedisBolt, AvroGenericRecordBolt, BaseBasicBolt, BaseBatchBolt, BaseCassandraBolt, BaseComponent, BaseOpaquePartitionedTransactionalSpout, BasePartitionedTransactionalSpout, BaseRichBolt, BaseRichSpout, BaseStatefulBolt, BaseStatefulBoltExecutor, BaseStatefulWindowedBolt, BaseTransactionalBolt, BaseTransactionalSpout, BaseWindowedBolt, BasicBoltExecutor, BasicDRPCTopology.ExclaimBolt, BatchBoltExecutor, BatchCassandraWriterBolt, BatchNumberList, BatchProcessWord, BatchRepeatA, BlobStoreAPIWordCountTopology.FilterWords, BlobStoreAPIWordCountTopology.RandomSentenceSpout, BlobStoreAPIWordCountTopology.SplitSentence, BoltTracker, CassandraWriterBolt, CheckpointSpout, CheckpointTupleForwarder, ClojureBolt, ClojureSpout, CoordinatedBolt, CountingBatchBolt, CountingCommitBolt, DRPCSpout, EsIndexBolt, EsLookupBolt, EsPercolateBolt, EventHubBolt, EventHubSpout, ExclamationTopology.ExclamationBolt, FastWordCountTopology.FastRandomSentenceSpout, FastWordCountTopology.SplitSentence, FastWordCountTopology.WordCount, FeederSpout, FixedTupleSpout, FluxShellBolt, FluxShellSpout, GlobalCountBolt, HBaseBolt, HBaseLookupBolt, HdfsBolt, HdfsSpout, HdfsSpoutTopology.ConstBolt, HiveBolt, IdentityBolt, InOrderDeliveryTest.Check, InOrderDeliveryTest.InOrderSpout, IntermediateRankingsBolt, JdbcInsertBolt, JdbcLookupBolt, JoinResult, KafkaBolt, KafkaSpout, KafkaSpout, KeyedCountingBatchBolt, KeyedCountingCommitterBolt, KeyedFairBolt, KeyedSummingBatchBolt, LogInfoBolt, ManualDRPC.ExclamationBolt, MasterBatchCoordinator, MemoryTransactionalSpout, MongoInsertBolt, MongoUpdateBolt, MqttBolt, MqttSpout, MultipleLoggerTopology.ExclamationLoggingBolt, OpaqueMemoryTransactionalSpout, OpaquePartitionedTransactionalSpoutExecutor, PartialCountBolt, PartitionedTransactionalSpoutExecutor, PrepareBatchBolt, PrepareRequest, PrinterBolt, PythonShellMetricsBolt, PythonShellMetricsSpout, RandomIntegerSpout, RandomSentenceSpout, ReachTopology.CountAggregator, ReachTopology.GetFollowers, ReachTopology.GetTweeters, ReachTopology.PartialUniquer, RedisLookupBolt, RedisStoreBolt, ResourceAwareExampleTopology.ExclamationBolt, ReturnResults, RichShellBolt, RichShellSpout, RichSpoutBatchTriggerer, RollingCountAggBolt, RollingCountBolt, SequenceFileBolt, SingleJoinBolt, SlidingWindowSumBolt, SolrUpdateBolt, SpoutTracker, StatefulBoltExecutor, StatefulTopology.PrinterBolt, StatefulWindowedBoltExecutor, StatefulWordCounter, SubtopologyBolt, TestAggregatesCounter, TestConfBolt, TestEventLogSpout, TestEventOrderCheckBolt, TestGlobalCount, TestPlannerBolt, TestPlannerSpout, TestPrintBolt, TestWindowBolt, TestWordBytesCounter, TestWordCounter, TestWordSpout, ThroughputVsLatency.FastRandomSentenceSpout, ThroughputVsLatency.SplitSentence, ThroughputVsLatency.WordCount, TotalRankingsBolt, TransactionalGlobalCount.BatchCount, TransactionalGlobalCount.UpdateGlobalCount, TransactionalSpoutBatchExecutor, TransactionalSpoutCoordinator, TransactionalWords.BucketCountUpdater, TransactionalWords.Bucketize, TransactionalWords.KeyedCountUpdater, TridentBoltExecutor, TridentSpoutCoordinator, TridentSpoutExecutor, TupleCaptureBolt, TwitterSampleSpout, WindowedBoltExecutor, WordCounter, WordCountTopology.SplitSentence, WordCountTopology.WordCount, WordCountTopologyNode.RandomSentence, WordCountTopologyNode.SplitSentence, WordCountTopologyNode.WordCount


# 4Spout详解
# 5Bolt详解
# 6Zoonkeeper详解
# 7安装配置
# 8Storm集群搭建
# 9准备开发环境
# 10开发自己的应用
# 11Storm-start
# 12DRPC详解
# 13事物拓扑
# 14Trident详解
# 15内部实现
# 16相关项目
# 17企业应用案例

Kafka