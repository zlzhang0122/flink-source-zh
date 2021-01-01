### 调度器

调度器是Flink作业执行的核心组件，它负责管理作业执行的所有相关过程，是作业执行、异常处理等的核心，它至少应该包含以下几个能力：
  * 作业生命周期的管理：如作业开始调度、挂起、取消等;
  * 作业执行资源的管理：如作业执行资源的申请、分配、释放等;
  * 作业状态的管理：如作业发布过程中的状态变化和作业异常时的Failover等;
  * 作业信息的管理：如对外提供作业的详细信息等;
  
目前，Flink中有两种调度器的实现，分别是DefaultScheduler和LegacyScheduler。其中，LegacyScheduler属于遗留的调度器，从Flink1.12开始，已经
将该调度器从代码中移除，因此对该调度器不再赘述。而DefaultScheduler使用SchedulingStrategy来实现调度。

SchedulingStrategy是一个接口，其中定义了调度的行为，一共包含四种行为：
  * startScheduling()：调度的入口，触发调度器的调度行为;
  * restartTasks()：执行失败的Task的重启，一般都是Task执行异常导致的;
  * onExecutionStateChange()：当Execution的状态发生改变时触发的行为;
  * onPartitionConsumable()：当IntermediateResultPartition中的数据可以被消费时触发;
  
目前，Flink中实现了三种类型的调度，分别是：Eager调度、分阶段调度、分阶段Slot重用调度。Eager调度(EagerSchedulingStrategy)主要用于流式计算，
它会一次性申请作业运行所需要的所有资源，如果资源无法被满足，则调度失败。分阶段调度(LazyFromSourcesSchedulingStrategy)主要适用于批处理，它从
Source Task开始进行分阶段调度，在申请资源时，只会申请本阶段所需要的所有资源。在上游Task执行完毕后开始执行下游Task，读取上游的结果数据，开始执行
本阶段的计算任务，在本阶段执行完成后再调度后一个阶段的Task，依次进行调度直到作业完成。分阶段Slot重用调度(PipelinedRegionSchedulingStrategy)，
它也主要用于批处理作业，它与分阶段调度的不同之处在于它可以在资源不足的情况下执行作业，但需要保证本阶段作业的执行中没有Shuffle行为。

执行模式指定了程序在数据交换方面的执行方式，Flink中主要有两类执行模式，分别为：Pipelined模式和Batch模式，它们在实现上又细分成了
四类具体的执行模式：
  * Pipelined：以流水线方式执行作业，如果可能会出现数据交换的死锁，则将数据交换以Batch方式执行(当数据流被多个下游分支处理，且处理的结果再进行JOIN
  时就可能出现数据交换死锁);
  * Pipelined_forced：与Pipelined模式类似，但不同之处在于即使可能会出现数据交换死锁，也不会切换为Batch方式;
  * Pipelined_With_Batch_Fallback：首先使用Pipelined启动作业，如果可能出现死锁，则使用Pipelined_Forced启动，当作业异常退出时，使用Batch
  模式重新执行作业(目前未实现);
  * Batch：对于所有的Shuffle和Broadcast以Batch方式执行，仅对于本地数据交换使用Pipelined方式;
  * Batch_forced：与Batch模式类似，不同之处在于即使本地数据交换也使用Batch方式;
  
执行模式的不同也决定了数据交换行为的不同，Flink在ResultPartitionType中定义了四种类型的数据分区模式，这些数据分区模式与执行模式一起完成批流在数
据交换层面的统一，它们分别是：
  * BLOCKING：适用于批处理，它会等待数据完全处理完毕，然后交给下游进行处理，在上游处理完毕之前，不会与下游进行数据交换;
  * BLOCKING_PERSISTENT：类似于BLOCKING，但生命周期由用户指定;
  * PIPELINED：适用于流计算和批处理，数据处理结果只能被一个消费者消费一次，当数据被消费之后即自动销毁，它可以在运行中保留任意数量的数据，如内存无
  法容纳时可以写入磁盘;
  * PIPELINED_BOUNDED：它带有一个有限大小的本地缓冲池;
  
