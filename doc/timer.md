### Timer定时器

Timer(定时器)是Flink Streaming API提供的用于感知并利用processing time(或event time)变化的机制。我们最常见的接触它的地方就是KeyedProcessFunction，
在这里我们通过在processElement()方法中注册Timer，然后覆盖onTimer()方法并在其中添加Timer触发时的回调逻辑。根据时间特征的不同，可分为两种情况：
  * Processing Time：通过调用context.timerService().registerProcessingTimeTimer()方法注册，onTimer()方法在系统时间戳达到Timer设定的时间戳
  时进行触发;

  * Event Time：通过调用context.timerService().rigisterEventTimeTimer()注册，onTimer()方法在Flink内部水印达到或超过Timer设定的时间戳
  时触发;

