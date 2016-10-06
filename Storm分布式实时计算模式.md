## Storm分布式实时计算模式

简单说一个task相当于一个spout/bolt实例.

在本地模式下增加worker数量不能提高速度.本地模式是在同一个JVM进程中执行.
增加task和exector才有效.

- 数据流分组

- 锚定tuple
建立tuple和衍生出的tuple间的对应关系.下游的tuple通过应答确认,报错或超时加入tuple对列.
非锚定的tuple处理失败,原始的tuple不会重新发送.

