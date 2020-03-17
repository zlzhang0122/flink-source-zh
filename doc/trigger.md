### 窗口触发器Trigger

Trigger用于决定何时触发窗口计算操作。
这个，是不是很奇怪？为何要触发窗口的计算，当时间到了窗口的结束时间时直接进行计算并输出不就行了吗，这样就可以直接由窗口的滑动时间来控制窗口触
发的频率了，为什么非得多此一举的弄出来一个Trigger？实际上，这个想法理论上可以，但是在实际的生产环境中是不行的。由于生产环境的复杂性，可能或
者说一定存在数据的延迟到达的情况，此时我们就必须通过设置一些参数来允许这些数据也能被处理。此时，窗口的结束时间就必须要加上这个允许的滞后时间，
窗口输出的间隔就被大大的拉大了。此时如果我们由想要结果能够尽快的被输出的话，那么就需要用到Trigger窗口触发器。

Trigger定义了何时开始对窗口进行计算，每个窗口都有一个默认的Trigger，当然也可以为窗口指定一个自定义的Trigger。其中有5个方法，允许其对不同
的事件进行处理：
  * onElement()：每个元素进入窗口都会被调用;

  * onProcessingTime()：当使用触发上下文设置的处理时间定时器timer被触发时调用;

  * onEventTime()：当使用触发上下文设置的事件时间定时器timer被触发时调用;

  * onMerge()：当使用WindowAssigner将多个窗口合并为一个窗口时被调用，如会话窗口的合并;

  * clear()：当窗口被清除时清理指定窗口的状态;

前三个方法被调用后会返回TriggerResult，它决定着接下来的处理行为：
  * CONTINUE：不做任何处理;

  * FIRE_AND_PURGE：触发计算之后执行purge操作;

  * FIRE：触发计算;

  * PURGE：清除所有window中的元素，并且该window被废弃;

为了简便使用，Flink中也定义了一些内置的触发器：
  * EventTimeTrigger：根据EventTime和WaterMark机制来判断是否触发计算;

  * ProcessingTimeTrigger：根据ProcessingTime判断是否触发计算;

  * CountTrigger：窗口中的元素超过预先设定的maxCount限制值时触发计算;

  * PurgingTrigger：它是一个trigger的包装类，如果被包装的trigger触发返回FIRE，则PurgingTrigger将返回修改为FIRE_AND_PURGE，其他的
  返回值不做任何处理;

  * DeltaTrigger：根据传入的DeltaFunction和阈值决定是否触发，DeltaFunction的逻辑需要用户自己定义。该函数比较上一次触发计算的元素，和
  目前到来的元素，比较结果为一个double类型阈值。如果阈值超过DeltaTrigger配置的阈值，则触发计算;