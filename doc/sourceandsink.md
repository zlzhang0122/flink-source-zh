### Source and Sink

Flink是批流一体的实时计算框架，其既可以做流处理，也可以做批处理，但是不管是哪种处理方式，都要有数据来源(输入)和数据汇集(输出)，前者叫做Source，
后者叫做Sink，Flink已经默认包含了最基本的Source和Sink(如，文件、Socket等)。另外，也还有一些常用的与其它第三方组件交互的Source和Sink，这些
叫做连接器，如与HDFS、Kafka、ElasticSearch等连接的连接器。

SourceFunction是定义Flink Source所有实现的根接口，其中定义的run()方法用于源源不断的产生源数据，所以重写的时候一般都会写成循环，然后用标志位
控制是否结束，cancel()方法则是用来打断run()方法中的循环，终止产生数据。

SourceFunction中还在内部嵌套定义了SourceContext接口，它表示这个Source对应的上下文，用于发射数据。其中起主要作用的是前三个方法：
  * collect()：发射一个不带自定义时间戳的元素。如果程序的时间特征(TimeCharacteristic)是处理时间，则元素没有时间戳；如果是摄入时间(IngestionTime)，
  元素会附带系统时间；如果是事件时间(EventTime)，那么初始时没有时间戳，但一旦要做与时间戳相关的操作(如，window)时，就必须用TimestampAssigner
  给它设定一个;

  * collectWithTimestamp()：发射一个带有自定义时间戳的元素。该方法对于时间特征为事件时间的程序是绝对必须的，如果为处理时间就会被直接忽略，如果
  为摄入时间就会被系统时间覆盖;

  * emitWatermark()：发射一个水印，仅对于事件时间有效。一个带有时间戳t的水印表示不会有任何t'<=t的事件再发生，如果真的发生，会被当做迟到事件忽略掉;

SourceFunction有一些其他实现，如：ParallelSourceFunction表示该Source可以按照设置的并行度并发执行；RichSourceFunction是继承自RichFunction，
表示该Source可以感知到运行时上下文(RuntimeContext，如Task、State及并行度的信息)，以及可以自定义初始化和销毁逻辑(通过open/close方法)。RichParallelSourceFunction
则表示的是以上两者的结合。

如同SourceFunction是定义Flink Source所有实现的根接口，SinkFunction是定义Flink Sink所有实现的根接口。但是它的定义比SourceFunction要简单，
只有一个invoke()方法，对收集来的每条数据都会用它来进行处理。SinkFunction也有对应的上下文对象Context，可以从中获得当前处理时间、当前水印和时间
戳，而且它也有衍生出来的RichSinkFunction函数版本。Flink内部提供了一个最简单的实现DiscardingSink。顾名思义，它就是将所有汇集的数据全部丢弃。


