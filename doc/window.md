### 窗口Window

Window可以理解为Flink中流数据的一种分组方式，它其中只定义了一个方法maxTimestamp()，其意义为该window时间跨度所能包含的最大时间点(用时间戳表示)。
Window类有两个子类，分别是GlobalWindow和TimeWindow，前者是全局窗口，而后者是具有起止时间的时间窗口。