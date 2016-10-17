## Jstorm

- 为什么启动Jstorm项目

1. 现有storm调度太简单粗暴，无法定制化
1. 雪崩问题一直没有解决
1. 监控太简单
1. 对ZK 访问频繁

RPC OOM（OOM - Out of Memory，内存溢出 ——俗称雪崩问题）一直没有解决

原生Storm RPC：Zeromq 使用堆外内存，导致OS 内存不够，Netty 导致OOM；
JStorm底层RPC 采用netty + disruptor，保证发送速度和接受速度是匹配的，彻底解决雪崩问题

- 更稳定（1） -- nimbus HA

Nimbus 实现HA:当一台nimbus挂了，自动热切到备份nimbus


- 更稳定（2）彻底解决Storm雪崩问题

1. 底层RPC 采用netty + disruptor
1. 保证发送速度和接受速度是匹配的

- 更稳定（3）-- 数据流稳定
	- 现有Storm
	添加supervisor时， 会触发任务rebalance
	Supervisor shutdown时， 触发任务rebalance
	提交新任务时，当worker数不够时，触发其他任务做rebalance

上叙问题不会在Jstorm中发生

- 更稳定（4） – 任务之间影响小

新上线的任务不会冲击老的任务

1. 新调度从cpu，memory，disk，net 四个角度对任务进行分配，
1. 已经分配好的新任务，无需去抢占老任务的cpu，memory，disk和net

- 更稳定（5） -- more catch

1. Supervisor主线程
1. Spout/Bolt 的open/prepare
1. 所有IO, 序列化，反序列化

- 更稳定（6）

减少对ZK的访问量：

1. 去掉大量无用的watch
1. task的心跳时间延长一倍
1. Task心跳检测无需全ZK扫描
1. 不将动态数据存储在ZK,特别是metrics和hearbeat
1. ZK can’t support more than 400 Storm nodes .
1. ZK can support 2000 Jstorm node, current in Alibaba, a lot of Jstorm ZK support 800 node.


- 性能更快

1. 底层使用ZeroMq， 比storm快30%
1. 底层使用netty时， 和storm快10%，并且稳定非常多 

- 为什么更快

1. Zeromq 减少一次内存拷贝
1. 增加反序列化线程
1. 重写采样代码，大幅减少采样影响
1. 优化ack代码
1. 优化缓冲map性能
1. Java 比clojure更底层
1. Smart Batch Policy
1. Add one thread to deserialize Tuple in every task
在每个任务中添加一个线程来反序列化元组
1. Remove total send/receive stage
1. Separate send and receive operation in Spout
1. Fix several bug which leading to CPU empty run.
1. Reduce metrics system performance influence.
1. Tuning Acker code
1. Tuning GC


- Cgroups

Cgroups是control groups的缩写，是Linux内核提供的一种可以限制、记录、隔离进程组（process groups）所使用的物理资源（如：cpu,memory,IO等等）的机制。最初由google的工程师提出，后来被整合进Linux内核。Cgroups也是LXC为实现虚拟化所使用的资源管理手段，可以说没有cgroups就没有LXC。

Cgroups可以做什么？
Cgroups最初的目标是为资源管理提供的一个统一的框架，既整合现有的cpuset等子系统，也为未来开发新的子系统提供接口。现在的cgroups适用于多种应用场景，从单个进程的资源控制，到实现操作系统层次的虚拟化（OS Level Virtualization）。Cgroups提供了一下功能：

1. 限制进程组可以使用的资源数量（Resource limiting ）。比如：memory子系统可以为进程组设定一个memory使用上限，一旦进程组使用的内存达到限额再申请内存，就会出发OOM（out of memory）。
2. 进程组的优先级控制（Prioritization ）。比如：可以使用cpu子系统为某个进程组分配特定cpu share。
3. 记录进程组使用的资源数量（Accounting ）。比如：可以使用cpuacct子系统记录某个进程组使用的cpu时间
4. 进程组隔离（isolation）。比如：使用ns子系统可以使不同的进程组使用不同的namespace，以达到隔离的目的，不同的进程组有各自的进程、网络、文件系统挂载空间。
5. 进程组控制（control）。比如：使用freezer子系统可以将进程组挂起和恢复。


