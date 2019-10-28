数据传递
===============
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

Lists can be unnumbered like:

 * Item Foo
 * Item Bar

Or automatically numbered:

 #. Item 1
 #. Item 2