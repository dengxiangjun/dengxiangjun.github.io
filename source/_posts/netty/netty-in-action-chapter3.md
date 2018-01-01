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

=======
本文将描述使用netty构建简单的客户服务器模型，客户端发送消息给服务端，服务端将收到的消息回送给客户端。这样的模型叫做echo服务，如图。![](/img/netty-in-action/2-1.png)

## Echo 服务端
echo服务端包含由两部分组成
* ChannelHandler，用以处理业务逻辑（返回消息给客户端等）
* Bootstrapping，服务端的配置与启动。

### ChannelHandlers和业务逻辑
在[上一章](/2017/11/09/netty/netty-in-action-chapter1/)中提到了Future和Callback用于实现事件驱动机制，同时也提到了ChannelHandler，我们可以实现ChannelHandler这个接口。
接口以完成对事件通知的响应。在netty应用中，如果需要实现数据处理的业务逻辑，都需要实现ChannelHandler这个接口。
由于Echo服务端需要对收到的客户端消息进行响应，因此它需要实现ChannelInboundHandler接口，它定义了对inbound事件的响应方法。因为server的逻辑比较简单，所以需要响应的事件比较少，使用ChannelInboundHandlerAdapter类即可实现该功能，ChannelInboundHandlerAdapter是ChannelInboundHandler的默认实现。下面我们重写ChannelInboundHandlerAdapter中的三个函数回调实现应用逻辑。
代码如下：
```java
//标示一个ChannelHandler可以被多个 Channel 安全地共享
@Sharable
public class EchoServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ByteBuf in = (ByteBuf) msg;
        //将消息记录到控制台
        System.out.println(
                "Server received: " + in.toString(CharsetUtil.UTF_8));
        //将接收到的消息写给发送者，而不冲刷出站消息
        ctx.write(in);
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx)
            throws Exception {
        //将剩下的消息冲刷到远程节点，并且关闭该 Channel
        ctx.writeAndFlush(Unpooled.EMPTY_BUFFER)
                .addListener(ChannelFutureListener.CLOSE);//关闭channel
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx,
        Throwable cause) {
        //打印异常栈跟踪
        cause.printStackTrace();
        //关闭该Channel
        ctx.close();
    }
}
```
ChannelInboundHandlerAdapter中的方法将会在事件的生命周期的合适的时间点回调。

### 服务器的配置与启动
当实现了EchoServerHandler后，我们需要配置相关的服务器启动相关的信息。包括
* 服务器绑定端口以监听从客户端到来的连接请求
* 配置Channels，从而使得消息到达时去通知EchoServerHandler
代码如下
```java
public class EchoServer {
    private final int port;

    public EchoServer(int port) {
        this.port = port;
    }

    public static void main(String[] args)
        throws Exception {
        if (args.length != 1) {
            System.err.println("Usage: " + EchoServer.class.getSimpleName() +
                " <port>"
            );
            return;
        }
        //设置端口值（如果端口参数的格式不正确，则抛出一个NumberFormatException）
        int port = Integer.parseInt(args[0]);
        //调用服务器的 start()方法
        new EchoServer(port).start();
    }

    public void start() throws Exception {
        final EchoServerHandler serverHandler = new EchoServerHandler();
        //(1) 创建EventLoopGroup
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            //(2) 创建ServerBootstrap
            ServerBootstrap b = new ServerBootstrap();
            b.group(group)
                //(3) 指定所使用的 NIO 传输 Channel
                .channel(NioServerSocketChannel.class)
                //(4) 使用指定的端口设置套接字地址
                .localAddress(new InetSocketAddress(port))
                //(5) 每accept一个连接请求就添加一个EchoServerHandler到该Channel的 ChannelPipeline
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    public void initChannel(SocketChannel ch) throws Exception {
                        //EchoServerHandler 被标注为@Shareable，所以我们可以总是使用同样的实例
                        //这里对于所有的客户端连接来说，都会使用同一个 EchoServerHandler，因为其被标注为@Sharable，
                        ch.pipeline().addLast(serverHandler);
                    }
                });
            //(6) 异步地绑定服务器；调用 sync()方法阻塞等待直到绑定完成
            ChannelFuture f = b.bind().sync();
            System.out.println(EchoServer.class.getName() +
                " started and listening for connections on " + f.channel().localAddress());
            //(7) 获取 Channel 的CloseFuture，并且阻塞当前线程直到它完成
            f.channel().closeFuture().sync();
        } finally {
            //(8) 关闭 EventLoopGroup，释放所有的资源
            group.shutdownGracefully().sync();
        }
    }
}
```

## Echo客户端
客户端需要做的事情有：
1. 连接到服务端
2. 发送消息
3. 对于每发送的消息，等待服务端响应同样的一条消息
4. 关闭连接

### ChannelHandler和业务逻辑
正如服务端一样。客户端同样需要handler去响应事件的到达，我们使用SimpleChannelInboundHandler去处理业务逻辑。

```java
@Sharable
//标记该类的实例可以被多个 Channel 共享
public class EchoClientHandler
    extends SimpleChannelInboundHandler<ByteBuf> {
    @Override
    public void channelActive(ChannelHandlerContext ctx) {
        //当被通知 Channel是活跃的时候，发送一条消息
        ctx.writeAndFlush(Unpooled.copiedBuffer("Netty rocks!",
                CharsetUtil.UTF_8));
    }

    @Override
    public void channelRead0(ChannelHandlerContext ctx, ByteBuf in) {
        //记录已接收消息的转储
        System.out.println(
                "Client received: " + in.toString(CharsetUtil.UTF_8));
    }

    @Override
    //在发生异常时，记录错误并关闭Channel
    public void exceptionCaught(ChannelHandlerContext ctx,
        Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```
**为什么不用ChannelInboundHandler？**
在客户端，读入的消息立马就处理完了（打印操作），channelRead0方法返回时，便会将字节缓冲区in的内存引用释放，而在服务端，channelRead方法中的write操作是异步的，因此在channelRead方法返回时，可能异步操作并未完成，因此，字节缓冲区in的内存引用不能释放。服务端只有在channelReadComplete中ctx.writeAndFlush调用后才会释放消息的引用。

### 启动客户端
代码如下：
```java
public class EchoClient {
    private final String host;
    private final int port;

    public EchoClient(String host, int port) {
        this.host = host;
        this.port = port;
    }

    public void start()
        throws Exception {
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            //创建 Bootstrap
            Bootstrap b = new Bootstrap();
            //指定 EventLoopGroup 以处理客户端事件；需要适用于 NIO 的实现
            b.group(group)
                //适用于 NIO 传输的Channel 类型
                .channel(NioSocketChannel.class)
                //设置服务器的InetSocketAddress
                .remoteAddress(new InetSocketAddress(host, port))
                //在创建Channel时，向 ChannelPipeline中添加一个 EchoClientHandler实例
                .handler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    public void initChannel(SocketChannel ch)
                        throws Exception {
                        ch.pipeline().addLast(
                             new EchoClientHandler());
                    }
                });
            //连接到远程节点，阻塞等待直到连接完成
            ChannelFuture f = b.connect().sync();
            //阻塞，直到Channel 关闭
            f.channel().closeFuture().sync();
        } finally {
            //关闭线程池并且释放所有的资源
            group.shutdownGracefully().sync();
        }
    }

    public static void main(String[] args)
            throws Exception {
        if (args.length != 2) {
            System.err.println("Usage: " + EchoClient.class.getSimpleName() +
                    " <host> <port>"
            );
            return;
        }

        final String host = args[0];
        final int port = Integer.parseInt(args[1]);
        new EchoClient(host, port).start();
    }
}
```
