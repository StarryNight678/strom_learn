## Storm分布式实时计算模式

简单说一个task相当于一个spout/bolt实例.

在本地模式下增加worker数量不能提高速度.本地模式是在同一个JVM进程中执行.
增加task和exector才有效.

- 数据流分组

