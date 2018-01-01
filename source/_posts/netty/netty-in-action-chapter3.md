---
title: Netty实战读书笔记(3) - netty的组成与设计思想
tags:
    - netty 
    - 网络编程
---
netty主要解决了两个问题：
1. 异步和事件驱动，构建在Java NIO之上。
2. 将应用逻辑与网络层解耦。

# Channel, EventLoop, and ChannelFuture
1. channel--socket
2. EventLoop--控制流多线程与并发
3. ChannelFuture--异步通知

# channel接口
channel接口是所有netty的channel基类，共有一下五种子类：
* EmbeddedChannel
* LocalServerChannel
* NioDatagramChannel
* NioSctpChannel
* NioSocketChannel

# EventLoop接口
EventLoop定义了连接生命周期的事件处理的核心抽象，一个EventLoop将会与一个线程进行绑定，这个线程会处理注册到它的那些socket的IO事件，多个EventLoop组成一个EventLoopGroup。如图展示了Channels, EventLoops, Threads, 和 EventLoopGroups之间的关系![](/img/netty-in-action/3-1.png)

# ChannelFuture接口
netty中的所有IO操作都是异步的，所有的操作结果不是立即返回，我们需要一种方法去知道异步调用的结果，netty提供了ChannelFuture接口，它的addListener()方法将会注册一个ChannelFutureListener去接受一个操作完成的通知。

# ChannelHandler接口
在netty应用开发者的角度，netty的主要是由ChannelHandler组成，它是应用的业务逻辑处理到达和发出的数据的容器，ChannelHandler中的方法在事件到达的时候出发；事件的概念是很广泛的，几乎任何动作都可以叫做事件，如把数据由一种格式转换为另一种格式或抛出一个异常。

# ChannelPipeline接口
ChannelPipeline将ChannelHandler组成一个链，事件流可以在链中传播，当channel创建的时候，系统会自动为其分配一个ChannelPipeline。
将ChannelHandler装配到ChannelPipeline的流程如下：
* ChannelInitializer的实现随着ServerBootstrap而注册
* 当ChannelInitializer.initChannel被调用时，ChannelInitializer安装一个用户定制的ChannelHandler
ChannelHandler接收事件，通过他的实现类的复写的方法执行具体的应用逻辑，然后把数据传输到ChannelPipeline中的下一个ChannelHandler。![](/img/netty-in-action/3-2.png)
inboundHandler和outboundHandler能够被安装在同一个ChannelPipeline中，如果一个消息或者是任一个inbound事件被读入，pipeline的头结点启动，消息被传输到第一个ChannelInboundHandler，该Handler处理数据，并把数据传输到链中的下一个ChannelInboundHandler，最终，数据到达尾节点，所有处理过程终止。
outbound动作(如数据被写)开始后，数据流从尾部传播到头部，到达网络层。
当一个ChannelHandler添加到ChannelPipeline后，会为他分配一个ChannelHandlerContext来代表ChannelHandler和ChannelPipeline绑定的上下文，我们可以利用它来写数据。
实际上写数据。

# ChannelHandler的几个子类
## adapter
ChannelHandler负责把事件转发到下一个ChannelHandler，ChannelHandler的几个adapter（ChannelHandlerAdapter、ChannelInboundHandlerAdapter、ChannelOutboundHandlerAdapter、ChannelDuplexHandlerAdapter）提供了ChannelHandler接口中的方法的一些默认实现，这样这些adapter就能够自动转发事件，我们只需要关心我们想要重写的方法和事件。
## Encoders and decoders
netty数据的收发需要格式转换，inbound消息需要解码，即由字节数据转换为其他格式，比如Java POJO对象，如果是outbound消息，则需要编码成字节数据。

