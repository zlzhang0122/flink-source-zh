### 内存管理

先来看一下Flink中TaskManager的内存布局：
![TaskManager内存布局](../images/memorymanage.jpg "TaskManager内存布局")

Flink为了减少对象存储开销，会先将对象进行序列化后再进行存储，序列化存储的最小单位是MemorySegment，底层的实现是数组，大小由taskmanager.memory.segment-size
配置项指定，默认32KB。下面分别进行介绍：
  * 网络缓存(Network Buffer)：用于进行网络传输及网络相关的操作(如shuffle、broadcast等)的内存块，由MemorySegment组成，自Flink 1.5版本开始，网络缓存固定分
  配在堆外，这样可以充分利用零拷贝等技术;

  * 托管内存：用于Flink内部所有算子逻辑的内存分配，及中间数据的存储，同样是由MemorySegment组成，通过Flink的MemoryManager组件进行管理。默认堆内分配，如果开启
  堆外内存分配的开关，也可以在堆外、堆内同时进行分配。相比于Spark，它没有对存储内存和执行内存进行区分，所以也更加的灵活;

  * 空闲内存：实际上用于存储用户代码和数据结构，固定分配在堆内，一般可以认为堆内内存减去托管内存后剩下的就是空闲内存;

YARN部署的per job集群的启动调用的是YarnClusterDescriptor.deployJobCluster()方法，其中的ClusterSpecification对象持有该集群的3个基本参数：JobManager内存
大小、TaskManager内存大小、每个TaskManager的slot数量，deployInternal()方法在其中调用YarnClusterDescriptor.validateClusterResources()方法对资源进行校
验。clusterSpecification.getMasterMemoryMB()返回的是JobManager的内存，默认是768MB，可以通过参数jobmanager.heap.size设置。clusterSpecification.getTaskManagerMemoryMB()
返回的是TaskManager的内存，默认是1024MB，可以通过参数taskmanager.heap.size设置。

以上是Flink1.10版本之前的内存管理方案，其实际上的配置比较复杂，有很多隐藏的细节需要注意，并且对于批处理作业和流处理作业分别有一套不同的配置方法，因此社区一直在探究
并提出了一套新的统一的内存管理模型和配置，并在Flink 1.10版本中进行了发布。下面就让我们来分析下吧，先来看下新的内存布局：
![新版TaskManager内存布局](../images/memorynew.jpg "新版TaskManager内存布局")

