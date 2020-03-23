### 内存管理

先来看一下Flink中TaskManager的内存布局：
![TaskManager内存布局](../images/memorymanage.jpg "TaskManager内存布局")

Flink为了减少对象存储开销，会先将对象进行序列化后再进行存储，序列化存储的最小单位是MemorySegment，底层的实现是数组，大小由taskmanager.memory.segment-size
配置项指定，默认32KB。下面分别进行介绍：
  * 网络缓存(Network Buffer)：用于进行网络传输及网络相关的操作(如shuffle、boradcast等)的内存块，由MemorySegment组成，自Flink 1.5版本开始