---
title: Netty实战读书笔记(1)
tags:
    - netty 
    - 网络编程
---
netty是一个高性能事件驱动的网络应用框架.

## Java网络库
下面展示了阻塞式IO的服务端代码

``` java
 //创建一个新的 ServerSocket，用以监听指定端口上的连接请求
        ServerSocket serverSocket = new ServerSocket(portNumber);
        //对accept()方法的调用将被阻塞，直到一个连接建立
        Socket clientSocket = serverSocket.accept();
        //这些流对象都派生于该套接字的流对象
        BufferedReader in = new BufferedReader(
                new InputStreamReader(clientSocket.getInputStream()));
        PrintWriter out =
                new PrintWriter(clientSocket.getOutputStream(), true);
        String request, response;
        //处理循环开始
        while ((request = in.readLine()) != null) {
            if ("Done".equals(request)) {
                break;
            }
            //请求被传递给服务器的处理方法
            response = processRequest(request);
            //服务器的响应被发送给了客户端
            out.println(response);
            //继续执行处理循环
        }
```

* 在代码中，accept() 一直阻塞到ServerSocket的连接建立完成，之后返回一个Socket对象以用于客户端与服务端通信。
* 客户端与服务端使用输入流BufferedReader和输出流PrintWriter进行通信。
* readLine()函数是阻塞的
* 为了处理多个并发的客户端，需要为每一个客户分配一个新的线程。如图1.1,显然，为一个客户端分配一个线程将容易触发计算机的瓶颈。
![](/img/netty-in-action/1-1.png)

## Java NIO
Java NIO提供了非阻塞式的网络库，在读写时若没有数据则立即返回，能够有效利用网络资源。
我们能够注册一组非阻塞式的socket，使用事件通知机制如select/poll/epoll来通知socket网卡上的数据就绪。
Java对epoll进行了封装，叫做selector,单个线程监听多个socket的读写操作，当数据就绪时，该线程会去处理。如图1.2。
![](/img/netty-in-action/1-2.png)

## netty
尽管selectors机制能够应付许多应用了，但是它比较笨重，且容易出错，netty是一个更好用性能更高的网络库。


## 异步和事件驱动
### 同步和异步
同步和异步关注的是消息通信机制 (synchronous communication/ asynchronous communication)所谓同步，就是在发出一个*调用*时，在没有得到结果之前，该*调用*就不返回。但是一旦调用返回，就得到返回值了。换句话说，就是由*调用者*主动等待这个*调用*的结果。而异步则是相反，*调用*在发出之后，这个调用就直接返回了，所以没有返回结果。换句话说，当一个异步过程调用发出后，调用者不会立刻得到结果。而是在*调用*发出后，*被调用者*通过状态、通知来通知调用者，或通过回调函数处理这个调用。
### 异步与可扩展性
异步能提高系统的可扩展性
* 异步方法立即返回，并且当被调用者完成时，会通知调用者。
* selector(调用者)允许我们使用更少线程去管理更多的连接。


## netty的组成
Channels，Callbacks，Futures，Events and handlers
### Channels
Channels是一个开放的连接实体如硬件设备、文件、Socket或者是一个能执行IO操作的程序。
### Callbacks
当回调被触发时
事件可以通过接口实现channelhandler处理,下面展示了一个回调的例子
``` java
public class ConnectHandler extends ChannelInboundHandlerAdapter {
        @Override
        //当一个新的连接已经被建立时，channelActive(ChannelHandlerContext)将会被调用
        public void channelActive(ChannelHandlerContext ctx)
                throws Exception {
            System.out.println(
                    "Client " + ctx.channel().remoteAddress() + " connected");
        }
}
```
### Futures
Future对象是异步操作结果的占位符，Java提供的Future接口需要我们手动检查执行任务是否完成，由于netty的事件驱动机制ChannelFutureListener，netty封装的Future对象免去了手动检查的过程。
```java
 //异步地连接到远程节点
        ChannelFuture future = channel.connect(
                new InetSocketAddress("192.168.0.1", 25));
         //注册一个 ChannelFutureListener，以便在操作完成时获得通知
        future.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture future) {
                //检查操作的状态
                if (future.isSuccess()) {
                    //如果操作是成功的，则创建一个 ByteBuf 以持有数据
                    ByteBuf buffer = Unpooled.copiedBuffer(
                            "Hello", Charset.defaultCharset());
                    //将数据异步地发送到远程节点。返回一个 ChannelFuture
                    ChannelFuture wf = future.channel()
                            .writeAndFlush(buffer);
                    // ...
                } else {
                    //如果发生错误，则访问描述原因的 Throwable
                    Throwable cause = future.cause();
                    cause.printStackTrace();
                }
            }
        });
```
### Events and handlers
Netty使用独特的事件来通知我们状态的变化，这样允许我们采用合适的动作去应对事件的触发。比如说日志、数据传输、流量控制、应用逻辑等。


## 总结
netty是一个由future和callbacks组成的异步编程模型，随着事件的分发，使用Handler处理事件的到来。他允许我们在网络环境下只关心应用的逻辑而不用花太多精力在网络操作上。使用回调机制和异步机制能获得底层操作和数据的传输的执行结果。
netty通过firing events对selector进行了抽象，一个EventLoop被分配到一个Channel去处理所有事件，包括注册感兴趣的事件、分发事件到ChannelHandlers、处理进一步的业务逻辑。
