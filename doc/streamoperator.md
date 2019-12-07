StreamOperator
-------------

StreamOperator是Stream operators的基础接口，是任务执行过程中的实际处理类，其上层由StreamTask调用，下层调用用户所实现的具体方法，它的
实现类是实现OneInputStreamOperator或TwoInputStreamOperator接口中的一种，分别表示处理一个输入、两个输入的Operator，其中包含了
processElement/processWatermark/processLatencyMarker方法，并用来创建算子处理数据。OneInputStreamOperator实现类StreamMap、
WindowOperator、KeyedProcessOperator等单流入处理Operator，TwoInputStreamOperator实现类CoStreamMap、KeyedCoProcessOperator、
IntervalJoinOperator等多流处理Operator。AbstractStreamOperator是StreamOperator
的基础抽象实现类，所有的operator都必须继承该抽象类，它为生命周期和属性方法提供了默认的实现。AbstractUdfStreamOperator是继承
AbstractStreamOperator的抽象实现类，其内部包含了userFunction，在Task的生命周期都会调用userFunction中对应的方法。

其层级结构如下图：

 ![StreamOperator](../images/stream-operator.png "StreamOperator")

列举一些常见的StreamOperator：
 * env.addSource对应StreamSource;
 * dataStream.map对应StreamMap;
 * dataStream.window对应WindowOperator;
 * dataStream.addSink对应StreamSink;
 * dataStream.keyBy(...).process对应KeyedProcessOperator;

StreamOperator继承的接口有：
 * CheckpointListener接口，其中的notifyCheckpointComplete方法表示checkpoint完成后的回掉函数;
 * KeyContext接口，用于当前key的切换，用于KeyedStream中state的key的设置;
 * Disposable接口，dispose方法主要用于对象销毁和资源释放
 * Serializable序列化接口

