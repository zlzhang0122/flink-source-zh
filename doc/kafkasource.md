### Kafka Source

在Flink 1.12中，基于Flip-27对KafkaSource重新的改造，不仅实现了批流一体，可以通过Bounded和UnBounded来统一实现批处理和流处理，而且还通过在
JobManager上增加Split Enumerator及在TaskManager上增加Source Reader来进行RPC通信，新的架构图如下图所示(图片来自Flink官网)：
![架构图](../images/kafka_source.svg "架构图")

在Flink 1.12以前，kafka的消息都是无界流，如果想要回溯指定时间段内的消息，就只能自己手动控制消费消息的边界，在消费完成后退出，其实在生产过程中还
是很有可能遇到这样的需求，毕竟某段时间的数据处理异常需要回溯数据是一个常见的需求，在1.12以前，我们在内部框架上自己实现了这样的一个功能。从Flink 
1.12开始，新的KafkaSource已经可以非常方便的实现这个需求了。

可以首先查看KafkaSource的源代码，在其最上面的类注释中可以看到一个非常常见的KafkaSource的调用方式，用到的一些参数都是消费kafka时常见的一些参数。
我们可以发现它其实是使用了KafkaSourceBuilder这个类来创建的，可以通过setUnbounded()/setBounded()来设置无界和有界，而从StreamGraphGenerator
类可以看出有界和无界其实会在最开始生成StreamGraph时就确定了。