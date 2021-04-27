### Kafka Source

在Flink 1.12中，基于Flip-27对KafkaSource重新的改造，不仅实现了批流一体，可以通过Bounded和UnBounded来统一实现批处理和流处理，而且还通过在
JobManager上增加Split Enumerator及在TaskManager上增加Source Reader来进行RPC通信，新的架构图如下图所示(图片来自Flink官网)：
![架构图](../images/kafka_source.svg "架构图")

