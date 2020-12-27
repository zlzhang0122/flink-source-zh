### 调度器

调度器是Flink作业执行的核心组件，它负责管理作业执行的所有相关过程，是作业执行、异常处理等的核心，它至少应该包含以下几个能力：
  * 作业生命周期的管理：如作业开始调度、挂起、取消等;
  * 作业执行资源的管理：如作业执行资源的申请、分配、释放等;
  * 作业状态的管理：如作业发布过程中的状态变化和作业异常时的Failover等;
  * 作业信息的管理：如对外提供作业的详细信息等;
  
目前，Flink中有两种调度器的实现，分别是DefaultScheduler和LegacyScheduler。其中，LegacyScheduler属于遗留的调度器，从
Flink1.12开始，已经将该调度器从代码中移除，因此对该调度器不再赘述。而DefaultScheduler使用SchedulingStrategy来实现调度。

SchedulingStrategy是一个接口，其中定义了调度的行为，一共包含四种行为：
  * startScheduling()：调度的入口，触发调度器的调度行为;
  * restartTasks()：执行失败的Task的重启，一般都是Task执行异常导致的;
  * onExecutionStateChange()：当Execution的状态发生改变时触发的行为;
  * onPartitionConsumable()：当IntermediateResultPartition中的数据可以被消费时触发;
  
目前，Flink中实现了三种类型的调度，分别是：Eager调度、分阶段调度、分阶段Slot重用调度。Eager调度(EagerSchedulingStrategy)
主要用于流式计算，它会一次性申请作业运行所需要的所有资源，如果资源无法被满足，则调度失败。分阶段调度(LazyFromSourcesSchedulingStrategy)
主要适用于批处理，它从Source Task开始进行分阶段调度，在申请资源时，只会申请本阶段所需要的所有资源。在上游Task执行完毕后开始执行
下游Task，读取上游的结果数据，开始执行本阶段的计算任务，在本阶段执行完成后再调度后一个阶段的Task，依次进行调度直到作业完成。分阶段
Slot重用调度(PipelinedRegionSchedulingStrategy)，它也主要用于批处理作业，它与分阶段调度的不同之处在于它可以在资源不足的情况
下执行作业，但需要保证本阶段作业的执行中没有Shuffle行为。

执行模式指定了程序在数据交换方面的执行方式，Flink中主要有两类执行模式，分别为：Pipelined模式和Batch模式，它们在实现上又细分成了
四类具体的执行模式：
  * Pipelined：以流水线方式执行作业，如果可能会出现数据交换的死锁，则将数据交换以Batch方式执行(当数据流被多个下游分支处理，且处理
  的结果再进行JOIN时就可能出现数据交换死锁);
  * Pipelined_forced：与Pipelined模式类似，但不同之处在于即使可能会出现数据交换死锁，也不会切换为Batch方式;
  * Batch：对于所有的Shuffle和Broadcast以Batch方式执行，仅对于本地数据交换使用Pipelined方式;
  * Batch_forced：与Batch模式类似，不同之处在于即使本地数据交换也使用Batch方式;
  
