大家好，我是张师傅。为了能帮助到更多对源码感兴趣、想学会看源码、提升自己后端技术能力的同学。我倾力持续组织了一年每周大家一起学习 Netty 源码共读活动，感兴趣的可以 点此扫码加我微信 zhangya_no1 参与。

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

### Netty 是如何启动服务的

- Netty 使用的是那种 I/O 模型
- 如何创建 selector
- 在哪里做 bind、listen
- backlog 参数有什么作用
- boss thread 和 worker thread 的职责是什么

关注：
`io.netty.channel.nio.NioEventLoop#openSelector`，
`io.netty.bootstrap.AbstractBootstrap#bind(int)` 等

### Netty 的连接创建是如何进行的

- 什么是事件循环
- 什么是 Reactor 模型
- Netty 事件循环采用的是什么模式，与 Nginx、redis、muduo 等框架是一样的吗
- accept 事件是在哪里注册的


关注：`io.netty.channel.nio.NioEventLoop#run`

### Netty 读取、发送数据流程

- Netty 读数据时，使用了哪些 Allocator
- Netty 在 linux 上采用的是边缘触发还是水平触发
- tcp 发送缓冲区满、接收缓存区满时会发生什么

关注：`io.netty.channel.ServerChannelRecvByteBufAllocator`、`io.netty.channel.AdaptiveRecvByteBufAllocator` 等

### Netty Idle 检测是如何实现的

- 什么是 tcp 的 keep-alive
- 有了 TCP 层面的 keep-alive 为什么还需要应用层 keepalive ?
- Netty 的 Idle 检测是如何实现的，是用 HashedWheelTimer 时间轮吗？





