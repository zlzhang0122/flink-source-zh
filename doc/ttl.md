### 状态生成时间(State TTL)

为了避免状态数据无限增长引发的OOM问题，必须有一个机制能够给状态进行瘦身，从而清除掉无效状态给系统带来的影响，Flink 从1.6开始引入了TTL(状态的生存
时间)机制，它支持KeyedState的自动过期，并自动清除掉过期的State。

从StateTtlConfig类来分析TTL机制的底层实现，该类中有5个成员属性，它们也是用户需要指定的参数配置。包括：
  * ttl表示用户设定的状态生存时间;
  * updateType表示状态时间戳的更新方式，它的取值类型是一个枚举;
  * stateVisibility表示已过期的状态数据的可见性，取值也是一个枚举;
  * ttlTimeCharacteristic表示对应的时间特征，取值同样是一个枚举，但实际上现阶段只支持一种时间特征，也就是处理时间;

CleanupStrategies内部类用于规定过期状态的清理策略，可以在构造StateTtlConfig时通过调用其中的方法来指定，常见的方法如下：
  * cleanupFullSnapshot()：当对状态做全量快照时清理过期数据，这对于开启增量快照的RocksDB状态后端没有作用，它对应于源码中的EmptyCleanupStrategy
  策略。由于它只能保证状态后端持久化时不包含过期数据，而对于TM本身的过期状态则不作任何清理，因此无法从根本上解决状态过大的问题;
  * cleanupIncrementally()：增量清理过期数据，默认是在每次访问状态时进行清理，如果设置第二个参数runCleanupForEveryRecord为true则也会在每
  次写入/删除时清理。第一个参数cleanupSize指定每次触发清理时检查的状态数量，它仅对基于堆的状态后端生效，对应于源码中的IncrementalCleanupStrategy
  策略;
  * cleanupInRocksdbCompactFilter()：通过FLink定制的过滤器过滤掉过期状态数据，它有一个参数queryTimeAfterNumEntries表示在写入多少条状态
  数据后，通过状态时间戳来判断是否过期。仅对于RocksDB状态后端生效，对应于源码中的RocksdbCompactFilterCleanupStrategy策略;
  
getOrCreateKeyedState()方法用于创建并记录状态实例，它定义在所有Keyed State状态后端的抽象基类AbstractKeyedStateBackend中。
 