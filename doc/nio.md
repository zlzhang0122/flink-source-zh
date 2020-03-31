### IO模型及NIO

我们知道在Unix网络编程中的五种IO模型，它们分别是：
  * 阻塞IO(BIO)

  * 非阻塞IO(NIO)

  * IO多路复用(IO multiplexing)

  * 信号驱动IO(signal driven IO)

  * 异步IO(asynchronous IO)

但是实际上除了AIO外，其余的都是同步IO(同步IO不一定是阻塞IO)，之所以称它们是同步IO，是因为在读写事件就绪后它们都需要自己负责进行读写，也就是
说整个读写过程是阻塞的。