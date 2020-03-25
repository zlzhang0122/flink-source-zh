### Timer定时器

Timer(定时器)是Flink Streaming API提供的用于感知并利用processing time(或event time)变化的机制。我们最常见的接触它的地方就是KeyedProcessFunction，
在这里我们通过在processElement()方法中注册Timer，然后覆盖onTimer()方法并在其中添加Timer触发时的回调逻辑。根据时间特征的不同，可分为两种情况：
  * Processing Time：通过调用context.timerService().registerProcessingTimeTimer()方法注册，onTimer()方法在系统时间戳达到Timer设定的时间戳
  时进行触发;

  * Event Time：通过调用context.timerService().rigisterEventTimeTimer()注册，onTimer()方法在Flink内部水印达到或超过Timer设定的时间戳
  时触发;

除了在KeyedProcessFunction中使用外，Timer在窗口机制中也有着重要的地位。提及窗口，最容易想到的便是Trigger触发器，相关内容在[Flink源码阅读8：窗口触发器](./trigger.md)
中进行过分析，此处不再赘述。

负责执行KeyedProcessFunction的算子是KeyedProcessOperator，在其中以内部类的形式实现了KeyedProcessFunction需要的上下文类Context，由此
可见timerService()方法返回的是外部传入的TimerService的实例，而Context这个类是在KeyedProcessOperator类的open()方法中被实例化的，所以
timerService的实例也是在此处传入的SimpleTimerService实现，而查看SimpleTimerService的源码会发现它只是对InternalTimerService的简单代理。
InternalTimerService的实例通过getInternalTimerService()方法获取，这是个定义在AbstractStreamOperator中的方法。

KeyedProcessOperator.processElement()方法调用用户自定义函数的processElement()方法，并传递了定时器的触发时间戳、TimestampedCollector
的实例以及上下文实例ContextImpl，所以用户可以通过这个上下文实例获得TimerService并注册Timer。Timer被触发时，实际上是根据时间特征调用事件时
间或处理时间的处理函数也就是onEventTime()/onProcessingTime()方法()，并触发用户函数里面的onTimer()函数回调逻辑。