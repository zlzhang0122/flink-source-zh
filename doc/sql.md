### Flink SQL

Flink SQL是Flink内部最高级的API，使用者可以通过SQL语句执行流批任务，屏蔽了底层DataStream/DataSet的细节，从而降低了使用门槛。
那么一条Flink SQL语句究竟是如何转化为可执行的任务的呢？就让我们来深入的看一看吧。

当然，在此之前有些前置知识需要先介绍一下，这就是Apache Calcite和Blink Planner。

