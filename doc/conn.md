### Flink数据通信

从数据源开始分析数据通信的整个过程，SourceFunction接口中的SourceContext内部接口SourceContext的collect()方法用于发射
数据，其实现类NonTimestampContext的collect()方法直接调用了output对象的collect方法，它是Output<StreamRecord<T>>类
型。