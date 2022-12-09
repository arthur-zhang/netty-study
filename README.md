大家好，我是张师傅。为了能帮助到更多对源码感兴趣、想学会看源码、提升自己后端技术能力的同学。组织了大家一起学习源码共读活动。

我对各个中间件源码非常感兴趣，过去一段时间阅读了 MySQL、JVM、Nginx、Netty、Spring、Linux 内核相关的源码，也写过很多关于根据源码来定位问题的文章，详见我的掘金博客 https://juejin.cn/user/430664257374270/posts

对于 Java 后端的同学，Netty 的源码是非常经典的学习资料，它不仅包含了丰富的网络编程相关的知识，还在代码中展示了很多 Java 编程的高级技巧，是我们深入学习网络编程、理解事件驱动、高性能编程不可多得的经典。

## 项目介绍

为了不增加额外的复杂度，这个项目采用最简单的 Netty echo server 来做演示。netty 启动以后监听 8888 端口，客户端使用 nc、telent 等可以直接进行连接，发送任意字符会回复任何字符。


![](https://store-g1.seewo.com/imgs/2022_11_21_16690187547907.jpg)

连接空闲 10s 以后会断开连接。

## 克隆项目


```
git clone https://github.com/arthur-zhang/netty-study.git
```

使用 idea 导入项目，使用 debug 模式启动 `me.ya.study.netty.MyServer`，通过调试源码的方式来学习 netty 源码。

## 源码学习

### Netty 服务端启动服务流程

在 `sun.nio.ch.ServerSocketChannelImpl#bind` 上打断点，看调用堆栈。


![](https://store-g1.seewo.com/imgs/2022_11_22_16691080655708.jpg)

> 任务：backlog 参数有什么作用，请仔细研究这个重要参数。



阅读 `io.netty.bootstrap.ServerBootstrap#init` 查看 Netty 配置服务端启动流程

![](https://store-g1.seewo.com/imgs/2022_11_22_16691076983047.jpg)


> 任务：阅读 `ServerBootstrapAcceptor` 类，搞清楚这个类的职责


在 `io.netty.channel.nio.AbstractNioChannel#AbstractNioChannel` 构造函数上打断点，阅读构造函数的调用堆栈


![](https://store-g1.seewo.com/imgs/2022_11_22_16691084651971.jpg)


> 任务：阻塞非阻塞、同步异步的最底层的区别是什么？这里为什么要设置为非阻塞

在 `io.netty.channel.nio.AbstractNioChannel#doBeginRead` 上打断点，查看 Netty 是如何注册 Accept 事件的。

![](https://store-g1.seewo.com/imgs/2022_11_22_16691089771399.jpg)


> 任务：梳理 Netty 启动服务端的所有流程
> 在哪里创建 Channel
> 如何初始化 Channel、注册 handler
> 如何做端口 bind 触发 active 事件，注册 accept 事件，开始准备接收连接


### Netty 的连接创建是如何进行的



在 `io.netty.channel.nio.NioEventLoop#processSelectedKey(java.nio.channels.SelectionKey, io.netty.channel.nio.AbstractNioChannel)` 打断点，然后使用 nc 或者 telnet 连接上 Netty

```
nc localhost 8888
```


![](https://store-g1.seewo.com/imgs/2022_11_22_16691101430154.jpg)

阅读堆栈，溯源调用链路

> 任务：分析 `io.netty.channel.nio.NioEventLoop#run`做了哪些事情


> 额外任务
- 什么是 Reactor 模型
- Netty 事件循环采用的是什么模式，与 Nginx、redis、muduo 等框架是一样的吗



### Netty 读取、发送数据流程

在 `io.netty.channel.nio.AbstractNioByteChannel.NioByteUnsafe#read` 上打断点，在 nc 中发送数据

![](https://store-g1.seewo.com/imgs/2022_11_22_16691104778193.jpg)

> 任务：分析 allocator 变量，学习 Netty 中所有的 Allocator

> 额外任务：Netty 在 linux 上采用的是边缘触发还是水平触发


在 `me.ya.study.netty.handler.MyEchoServerHandler#channelRead` 打断点，跟进 write 方法，同时在 `io.netty.channel.AbstractChannelHandlerContext#write(java.lang.Object, io.netty.channel.ChannelPromise)` 上打断点，分析 Netty 发送数据的全过程。

> 任务：分析 Netty 发送数据的全流程，画时序图


## 额外任务

### Netty Idle 检测是如何实现的

- 什么是 tcp 的 keep-alive
- 有了 TCP 层面的 keep-alive 为什么还需要应用层 keepalive ?
- Netty 的 Idle 检测是如何实现的，是用 HashedWheelTimer 时间轮吗？


# 第二周

阅读 NioEventLoopGroup 的代码

> 任务：NioEventLoopGroup 默认的构造函数会起多少线程，可以通过什么方式修改？这些线程的职责是什么


早期 Java 版本 NIO 存在严重的 epoll 空轮询 bug，请查询相关的文章。


![](https://store-g1.seewo.com/imgs/2022_11_30_16697908751570.jpg)


> 任务 : 阅读 NioEventLoop.java 的代码，尝试分析 Netty 是如何解决 JDK 中的 epoll 空轮询 BUG 的？


Netty 内部有一个核心的类 ByteToMessageDecoder，它定义了两个累加器  MERGE_CUMULATOR、COMPOSITE_CUMULATOR


![](https://store-g1.seewo.com/imgs/2022_11_30_16697913516892.jpg)

> 任务 1：分析这两个累加器有什么不同

> 任务2：写一个 LengthFieldBasedFrameDecoder 定长编码的消息拆包类，实现如下格式消息的解码，并按逗号拼接

| Length | Content |
|---|---|
| 4 字节  |变长  |


额外任务：零拷贝知识

任务 1：了解什么是零拷贝，C/C++中如何实现零拷贝

任务 2：Java 中如何实现零拷贝

任务 3：netty 是如何实现零拷贝的




## 第三周

Netty 的数据读写是以 ByteBuf 为单位的，它的结构如下：


![](https://store-g1.seewo.com/imgs/2022_12_09_16705461322023.jpg)

任务 1：阅读 ByteBuf 的代码，总结它与 Java NIO 中的 ByteBuffer 有什么区别？


ByteBuf 可以通过扩展底层的数组来实现自动扩展。当缓冲区的容量不足以存储新的数据时，ByteBuf 就会自动扩展底层数组的大小，以便容纳更多的数据

任务 2：Netty 中的 ByteBuf 源码是如何实现自动扩展的，请写出伪代码


任务 3：阅读相关代码，ByteBuf 是线程安全的吗？

任务 4：为什么 ByteBuf 读写需要加锁？


ByteBuf 支持多种内存管理模型，包括堆内内存（heap buffer）、堆外内存（direct buffer）和内存池（pooled buffer）。

任务 5：堆外内存、堆外内存、内存池的优缺点有哪些，分别用在哪些场景

任务 6：下面的分配方式分别对应上面的哪种类型

```
ByteBufAllocator allocator = ByteBufAllocator.DEFAULT;
ByteBuf buffer = allocator.heapBuffer();

ByteBufAllocator allocator = ByteBufAllocator.DEFAULT;
ByteBuf buffer = allocator.directBuffer();

ByteBufAllocator allocator = ByteBufAllocator.DEFAULT;
ByteBuf buffer = allocator.pooledBuffer();

```

ByteBuf 的读写操作是非阻塞的，阅读相关代码。

任务 7：非阻塞特性是通过 ByteBuf 的什么原理实现的