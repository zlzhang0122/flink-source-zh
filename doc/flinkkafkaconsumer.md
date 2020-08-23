### FlinkKafkaConsumer

FlinkKafkaConsumer是一个流式数据源，用于从Apache Kafka中获取并行数据流。其继承结构如下图所示：
![FlinkKafkaConsumer继承体系](../images/flinkkafka.png "FlinkKafkaConsumer继承体系")

从上图可以看到，FlinkKafkaConsumer继承于FlinkKafkaConsumerBase类，而FlinkKafkaConsumerBase类又实现了RichFunction接口
和SourceFunction接口(在Flink 1.11版本中进行了重构，实现的是ParallelSourceFunction接口)。由于实现了RichFunction接口，所以
我们可以分析下其open()方法和run()方法。

open()方法的实现在FlinkKafkaConsumerBase类中，主要是FlinkKafkaConsumer的初始化逻辑。
首先设置offset的提交模式，OffsetCommitMode是一个枚举类型，有以下三个取值：
  * DISABLED：完全禁用offset的提交;
  * ON_CHECKPOINTS：仅在checkpoint完成时提交offset;
  * KAFKA_PERIODIC：周期性提交，使用kafka客户端内部的自动提交功能;
具体判断OffsetCommitMode的逻辑被封装在OffsetCommitModes.fromConfiguration()方法中，该方法会先判断是否启用checkpoint，如果启用且
同时启用了checkpoint完成时提交offset，则返回ON_CHECKPOINTS；如果未启用checkpoint，同时启用了自动提交则返回KAFKA_PERIODIC，否则在
其他情况下都返回DISABLED。
接着便是创建和启动分区发现工具。
