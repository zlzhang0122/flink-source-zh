###数据传递

本节主要介绍数据在各个节点之间是如何进行传递的,以及如何在数据传递的过程中进行自然的反压.

数据传递流程
----------------
 .. image:: image/shuffle.png

整体实现步骤如下:

 #. 在M1/M2处理完数据后,本地需要ResultPartition RS1/RS2来临时存储数据;
 #. 通知JobManager,上游有新的数据产生;
 #. JobManager通知和调度下游节点可以消费新的数据;
 #. 下游节点向上游请求数据;
 #. 通过Channel在各个TaskManager之间传递数据.

数据在节点之间传递的具体流程如下图:

 .. image:: image/shuffle-data.png

 * 数据在operator处理完成后,先交给RecordWriter,每条记录都要选择一个下游节点,所以要经过ChannelSelector;
 * 在每个channel都有一个serializer,把这条Record序列化为ByteBuffer;
 * 接下来数据被写入ResultPartition下的各个ResultSubPartition里,此时该数据已经存入MemorySegment;
 * 单独的线程控制数据的flush速度,一旦触发flush,则通过Netty的NIO通道向对端写入;
 * 接收端的Netty Client收到数据后,进行decode操作,把数据拷贝到Buffer里,然后通知InputChannel;
 * 当InputChannel中有可用的数据时,下游算子从阻塞醒来,从InputChannel取出Buffer,再反序列化成Record,并将其交给算子执行相应的用户代码

数据传递源码
----------------

首先,将数据流中的数据交给RecordWriter.
然后,选择序列化器并序列化数据,写入到相应的Channel.
当输出缓冲中的字节数超过了高水位值,则Channel.isWritable()会返回false.当输出缓存中的字节数又掉到低水位值以下,则Channel.isWritable()会重新返回true.
核心发送方法中如果channel不可写,则会跳过发送.当channel再次可写后,Netty会调用该Handle的handleWritabilityChanged方法,从而重新出发发送函数.

Flink通过Credit实现网络流控,即下游会向上游发送一条credit message,用以通知其目前可用信用额度,然后上游会根据这个信用消息来决定向下游发送多少数据.当
上游把数据发送给下游时,它就从下游信用上划走相应的额度.

在上游通过Channel发送数据后,下游通过decodeMsg来获取数据.

Flink其实做阻塞和获取数据的方式非常自然，利用了生产者和消费者模型，当获取不到数据时，消费者自然阻塞；当数据被加入队列，消费者被notify。
Flink的背压机制也是借此实现。
至此，Flink数据在节点之间传递的过程便介绍完成。