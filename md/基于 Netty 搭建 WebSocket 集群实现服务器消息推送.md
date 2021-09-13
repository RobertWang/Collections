> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247522658&idx=1&sn=0bdcdf85cac29b9bad5be8be6349ba52&chksm=ce0e2ee1f979a7f72315fbd4a377dc3792c06ef0a6a42dac757080808f7aeba919ea0409ff06&mpshare=1&scene=1&srcid=0913vVsAfDqmln4Hr6272R8S&sharer_sharetime=1631530353640&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

本文内容是构建高性能、高可用消息推送的经典案例，在微服务 Spring Cloud 环境下基于 Netty 搭建 websocket 集群实现的。目录为：

1、背景

2、websocket

3、netty

   3.1 socket

   3.2 Java IO 模型

   3.3 netty

      3.3.1 概念：

      3.3.2 三大特点：

      3.3.3 主从 Reactor 架构图

      3.3.4 应用场景

4、springboot 环境下使用 netty 搭建 websocket

     4.1 系统设计架构图

     4.2 架构中存在的六大经典问题

     4.3 引入 pom 依赖和 yml 配置

     4.4 Websocket 初始化器

     4.5 websocket 通道初始化器

     4.6 websocket 入站处理器

     4.7 channel 中任务队列的线程任务

     4.8 websocket 启动程序

     4.9 问题六解决方案

     4.10 前端代码

     4.11 wesocket 在 nginx 中配置

     4.12 效果图

5、总结      

**（1）短轮询**：前端老师利用 ajax 定期向服务器发起 http 请求，无论数据是否更新立马返回数据。这样存在的缺点就是，一方面如果后端数据木有更新，那么这一次 http 请求就是无用的，另一方面高并发情况下，短链接的频繁创建销毁，以及客户端数量过大造成过多无用的 http 请求，都会对服务器和带宽造成压力，短轮询只适用于客户端连接少，并发量不高的场景；

**（2）长轮询**：利用 comet 不断向服务器发起请求，服务器将请求暂时挂起，直到有新的数据的时候才返回，相对短轮询减少了请求次数得到了一定的优化，但是在高并发的场景下依然不适用；

**（3）SSE**：服务端推送（Server Send Event），在客户端发起一次请求后会保持该连接，服务器端基于该连接持续向客户端发送数据，从 HTML5 开始加入。

**（4）Websocket**：这是也是一种保持长连接的技术，并且是双向的，从 HTML5 开始加入，并非完全基于 HTTP，适合于频繁和较大流量的双向通讯场景，是服务器推送消息功能的最佳实践。而实现 websocket 的最佳方式，就是 **netty**。

网上的很多 netty 搭建 websocket 的博文都不够全面，有很多问题都木有解决方式，我通过实际工作中的经验，把常遇到的问题总结了方法，下文会说到！

**什么是 websocekt 呢？**

websocket 是一种在单个 TCP 连接上进行全双工通信的协议。也就是一种保持长连接的技术，并且是双向的。

websocket 协议本身是构建在 http 协议之上的升级协议，客户端首先向服务器端去建立连接，这个连接本身就是 http 协议只是在头信息中包含了一些 websocket 协议的相关信息，一旦 http 连接建立之后，服务器端读到这些 websocket 协议的相关信息就将此协议升级成 websocket 协议。WebSocket 使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在 WebSocket API 中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。

**简单理解，就是一种通讯协议，重点是 websocket 的实现方式–netty。**

**什么是 netty 呢?**

3.1 socket
----------

（1）**网络编程本质**就是说两个设备之间信息的发送与接收，通过操作相应 API 调度计算机硬件资源，并且利用管道（网线）进行数据交互的过程。相关技术点像 ISO 七层模型，TCP 三次握手 / 四次挥手等网络编程的基础不再赘述。

（2）**而 socket 是**对 TCP/IP 协议的封装，Socket 本身并不是协议，而是一个调用接口（API），通过 Socket 发起系统调用操作系统内核，我们才能使用 TCP/IP 协议。

（3）**我们经常说的 I/O**，在计算机中指 Input/Output, 即输入输出，实质上 IO 分为两种，一种是磁盘 IO，磁盘上的数据读取到用户空间，那么这次数据转移操作其实就是一次 I/O 操作，也就是一次文件 I/O。一种是网络 IO，当一个客户端和服务端之间相互通信, 交互我们称之为网络 io(网络通讯)

3.2 Java IO 模型
--------------

有 BIO（同步阻塞 IO）、NIO（同步非阻塞 IO）、AIO（异步 IO），netty 就是一个 NIO 的高性能的框架。

（１）BIO：同步阻塞 IO 模型，适用于连接数目比较少且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，服务器实现模式为一个连接一个线程，即客户端有连 接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造 成不必要的线程开销，可以通过线程池机制改善 (实现多个客户连接服务器)。  

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsiaUGQfNQRQoVrV2or4HmfiaSmrFZ1saGWK91sVfDicOmYfgF5dpHlERlPJYwMrmXCnI5azmKxvtVmJQ/640)

（２）NIO：同步非阻塞 IO 模型，适用于连接数据多且连接比较短的架构，如聊天服务器，弹幕系统，服务器间通讯等；服务器实现模式为一个线程处理多个连接（一个请求一个线程），包含 Selector 、Channel 、Buffer 三大组件。  

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsiaUGQfNQRQoVrV2or4HmfiaSylD3kfic73yibzrPschjdWZiaJYQkvZP21hLmKgUptwNkMU1y5ibFTGY5A/640)

①**selector 选择器**：用一个线程处理多个客户端的连接，就会使用到 selector 选择器，selector 用于监听多个通道上是否有事件发生（比如连接请求，数据到达等），如果有事件发生，便获取事件然后针对于每个事件进行相应的处理，因此可以使用单个线程就可以监听多个客户端通道。

②**channel 通道**：Channel 管道和 Java IO 中的 Stream(流) 是差不多一个等级的。只不过 Stream 是单向的，譬如：InputStream, OutputStream. 而 Channel 是双向的，同时进行读写数据，而流只能读或者写。可以实现异步读写数据，可以从缓冲区读数据，也可以写数据到缓冲区。

③**buffer 缓冲区**：Buffer 本质上是一个可以读写的内存块，可以理解成容器对象，底层是有一个数组，通过 buffer 实现非阻塞机制，该对象提供了一组方法，可以轻松的使用内存块，缓冲区对象内置了一些机制，能够跟踪和记录缓冲区的状态变化情况。Channel 提供了从文件，网络读取数据的通道，但是读取和写入的数据必须经过 buffer。

（３）AIO：异步 IO 模型，使用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分 调用 OS 参与并发操作，编程比较复杂。在 Linux 底层用 epoll(一种轮询模型)，aio 多包了一层封装，aio 的 api 更好用。Windows 上的 aio 是自己实现的，不是轮询模型是事件模型，完成端口实现的，要比 linxu 上的 aio 效率高。

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsiaUGQfNQRQoVrV2or4HmfiaSTUtchhTmnI8ibbwc8DkKzJ7oVrZZaGqcicxQGgmNibPw9CFfEBfznNu0w/640)

3.3 netty
---------

### 3.3.1 概念：

### netty 是一个开源异步的事件驱动的网络应用程序框架，用于快速开发可维护的高性能协议服务器和客户端。

### 3.3.2 三大特点：

①高并发：Netty 是一款基于 NIO（Nonblocking IO，非阻塞 IO）开发异步事件驱动的高性能网络通信框架，nio 使用了 select 模型（多路复用器技术），从而使得系统在单线程的情况下可以同时处理多个客户端请求。Netty 使用了 Reactor 模型，Reactor 模型有三种多线程模型，netty 是在主从 Reactor 多线程模型上做了一定的改进。

Netty 有两个线程组，一个作为 bossGroup 线程组, 负责客户端接收, 一个 workerGroup 线程组负责工作线程的工作 (与客户端的 IO 操作和任务操作等等)，Netty 的所有 IO 操作都是异步非阻塞的，通过 Future-Listener 机制，用户可以方便的主动获取或者通过通知机制获得 IO 操作结果。他的并发性能得到了很大提高。

②传输快：Netty 的传输依赖于零拷贝特性，实现了更高效率的传输。零拷贝要求内核（kernel）直接将数据从磁盘文件拷贝到 Socket 缓冲区（套接字），而无须通过应用程序。零拷贝减少不必要的内存拷贝，不仅提高了应用程序的性能，而且减少了内核态和用户态上下文切换。

③封装好：Netty 封装了 NIO 操作的很多细节，提供了易于使用调用接口。

### 3.3.3 主从 Reactor 架构图

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsiaUGQfNQRQoVrV2or4HmfiaSfF8icpEGk0L0l8C8NBZLZtcBt7BpebcXCMRK8rruDTXvsUibfM9ZricCQ/640)

  
说明：①Reactor 响应式编程（事件驱动模型）：一般有一个主循环和一个任务队列，所有事件只管往队列里塞，主循环则从队列里取出并处理。

如果不依赖于多路复用处理多个任务就会需要多线程 (与连接数对等) , 但是依赖于多路复用，这个循环就可以在单线程的情况下处理多个连接。无论是哪个连接发生了什么事件，都会被主循环从队列取出并处理 (可能用回调函数处理等) , 也就是说程序的走向由事件驱动.

②mainReactor：主 Reactor 负责 单线程就可以接受所有客户端连接

③subReactor：子 Reactor 负责 多线程处理客户端的读写 IO 事件

④ThreadPool：线程池负责 处理业务耗时的操作

### 3.3.4 应用场景

①现在物联网的应用无处不在，大量的项目都牵涉到应用传感器和服务器端的数据通信，Netty 作为基础通信组件进行网络编程。

②现在互联网系统讲究的都是高并发、分布式、微服务，各类消息满天飞，Netty 在这类架构里面的应用可谓是如鱼得水，如果你对当前的各种应用服务器不爽，那么完全可以基于 Netty 来实现自己的 HTTP 服务器，FTP 服务器，UDP 服务器，RPC 服务器，WebSocket 服务器，Redis 的 Proxy 服务器，MySQL 的 Proxy 服务器等等。

现在非常多的开源软件都是基于 netty 开发的，例如阿里分布式服务框架 Dubbo 的 RPC 框架，淘宝的消息中间件 RocketMQ；

③游戏行业：无论是手游服务端还是大型的网络游戏，Java 语言得到了越来越广泛的应用。Netty 作为高性能的基础通信组件，它本身提供了 TCP/UDP 和 HTTP 协议栈。地图服务器之间可以方便的通过 Netty 进行高性能的通信。

④大数据：开源集群运算框架 Spark；分布式计算框架 Storm；

4.1 系统设计架构图
-----------

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsiaUGQfNQRQoVrV2or4HmfiaSbqXI4tPNPiaym2wmqKiaRKtLjbH7gvibJORGrsUrRAPicpZBM0KBORdGxw/640)

4.2 架构中存在的六大经典问题
----------------

**第一个问题**：客户端和服务端单独通信，怎么实现？

**第二个问题**：单机中 websocekt 主动向所有客户端推送消息如何实现？在集群中如何实现？

**第三个问题**：单机如何统计同时在线的客户数量？websocket 集群如何统计在线的客户数量呢？

**第四个问题**：由于客户端和 websocket 服务器集群中的某个节点建立长连接是随机的，如何解决服务端向某个或某些部分客户端推送消息？

**第五个问题**：websocket 服务端周期性向客户端推送消息，单机或集群中如何实现？

**第六个问题**：websocket 集群中一个客户端向其他客户端主动发送消息，如何实现？  
福利来啦！！以上所有问题，已在代码中全部解决并实践！！！

4.3 引入 pom 依赖和 yml 配置
---------------------

（1）pom 依赖
---------

```
<dependencies>
    <dependency>
      <groupId>io.netty</groupId>
      <artifactId>netty-all</artifactId>
      <version>4.1.36.Final</version>
    </dependency>
  </dependencies>
```

（2）yml 配置

```
websocket:
  port: 7000 #端口
  url: /msg #访问url
```

(3) 客户端和服务端交互的消息体

```
package com.wander.netty.websocket.yeelight;
import lombok.Data;
import java.io.Serializable;

@Data
public class MessageRequest implements Serializable {

    private Long unionId;

    private Integer current = 1;

    private Integer size = 10;
}
```

4.4 Websocket 初始化器
------------------

```
/**
 * @Author WDYin
 * @Date 2021/6/10
 * @Description websocket初始化器
 **/
@Slf4j
@Component
public class WebsocketInitialization {

    @Resource
    private WebsocketChannelInitializer websocketChannelInitializer;

    @Value("${websocket.port}")
    private Integer port;

    @Async
    public void init() throws InterruptedException {

        //bossGroup连接线程组，主要负责接受客户端连接，一般一个线程足矣
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        //workerGroup工作线程组，主要负责网络IO读写
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {

            //启动辅助类
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            //bootstrap绑定两个线程组
            serverBootstrap.group(bossGroup, workerGroup);
            //设置通道为NioChannel
            serverBootstrap.channel(NioServerSocketChannel.class);
            //可以对入站\出站事件进行日志记录，从而方便我们进行问题排查。
            serverBootstrap.handler(new LoggingHandler(LogLevel.INFO));
            //设置自定义的通道初始化器，用于入站操作
            serverBootstrap.childHandler(websocketChannelInitializer);
            //启动服务器,本质是Java程序发起系统调用，然后内核底层起了一个处于监听状态的服务，生成一个文件描述符FD
            ChannelFuture channelFuture = serverBootstrap.bind(port).sync();
            //异步
            channelFuture.channel().closeFuture().sync();

        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

}
```

4.5 websocket 通道初始化器
--------------------

```
/**
 * @Author WDYin
 * @Date 2021/6/10
 * @Description websocket通道初始化器
 **/
@Component
public class WebsocketChannelInitializer extends ChannelInitializer<SocketChannel> {

    @Autowired
    private WebSocketHandler webSocketHandler;

    @Value("${websocket.url}")
    private String websocketUrl;

    @Override
    protected void initChannel(SocketChannel socketChannel) throws Exception {

        //获取pipeline通道
        ChannelPipeline pipeline = socketChannel.pipeline();
        //因为基于http协议，使用http的编码和解码器
        pipeline.addLast(new HttpServerCodec());
        //是以块方式写，添加ChunkedWriteHandler处理器
        pipeline.addLast(new ChunkedWriteHandler());
        /*
          说明
          1. http数据在传输过程中是分段, HttpObjectAggregator ，就是可以将多个段聚合
          2. 这就就是为什么，当浏览器发送大量数据时，就会发出多次http请求
        */
        pipeline.addLast(new HttpObjectAggregator(8192));
        /* 说明
          1. 对应websocket ，它的数据是以 帧(frame) 形式传递
          2. 可以看到WebSocketFrame 下面有六个子类
          3. 浏览器请求时 ws://localhost:7000/msg 表示请求的uri
          4. WebSocketServerProtocolHandler 核心功能是将 http协议升级为 ws协议 , 保持长连接
          5. 是通过一个 状态码 101
        */
        pipeline.addLast(new WebSocketServerProtocolHandler(websocketUrl));
        //自定义的handler ，处理业务逻辑
        pipeline.addLast(webSocketHandler);
    }
}
```

4.6 websocket 入站处理器
-------------------

```
/**
 * @Author WDYin
 * @Date 2021/6/10
 * @Description websocket处理器
 **/
@Slf4j
@Component
@ChannelHandler.Sharable//保证处理器，在整个生命周期中就是以单例的形式存在，方便统计客户端的在线数量
public class WebSocketHandler extends SimpleChannelInboundHandler<TextWebSocketFrame> {

    //通道map，存储channel，用于群发消息，以及统计客户端的在线数量，解决问题问题三，如果是集群环境使用redis的hash数据类型存储即可
    private static Map<String, Channel> channelMap = new ConcurrentHashMap<>();
    //任务map，存储future，用于停止队列任务
    private static Map<String, Future> futureMap = new ConcurrentHashMap<>();
    //存储channel的id和用户主键的映射，客户端保证用户主键传入的是唯一值，解决问题四，如果是集群中需要换成redis的hash数据类型存储即可
    private static Map<String, Long> clientMap = new ConcurrentHashMap<>();

    /**
     * 客户端发送给服务端的消息
     *
     * @param ctx
     * @param msg
     * @throws Exception
     */
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, TextWebSocketFrame msg) throws Exception {

        try {
            //接受客户端发送的消息
            MessageRequest messageRequest = JSON.parseObject(msg.text(), MessageRequest.class);

            //每个channel都有id，asLongText是全局channel唯一id
            String key = ctx.channel().id().asLongText();
            //存储channel的id和用户的主键
            clientMap.put(key, messageRequest.getUnionId());
            log.info("接受客户端的消息......" + ctx.channel().remoteAddress() + "-参数[" + messageRequest.getUnionId() + "]");

            if (!channelMap.containsKey(key)) {
                //使用channel中的任务队列，做周期循环推送客户端消息，解决问题二和问题五
                Future future = ctx.channel().eventLoop().scheduleAtFixedRate(new WebsocketRunnable(ctx, messageRequest), 0, 10, TimeUnit.SECONDS);
                //存储客户端和服务的通信的Chanel
                channelMap.put(key, ctx.channel());
                //存储每个channel中的future，保证每个channel中有一个定时任务在执行
                futureMap.put(key, future);
            } else {
                //每次客户端和服务的主动通信，和服务端周期向客户端推送消息互不影响 解决问题一
                ctx.channel().writeAndFlush(new TextWebSocketFrame(Thread.currentThread().getName() + "服务器时间" + LocalDateTime.now() + "wdy"));
            }

        } catch (Exception e) {

            log.error("websocket服务器推送消息发生错误：", e);

        }
    }

    /**
     * 客户端连接时候的操作
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        log.info("一个客户端连接......" + ctx.channel().remoteAddress() + Thread.currentThread().getName());
    }

    /**
     * 客户端掉线时的操作
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {

        String key = ctx.channel().id().asLongText();
        //移除通信过的channel
        channelMap.remove(key);
        //移除和用户绑定的channel
        clientMap.remove(key);
        //关闭掉线客户端的future
        Optional.ofNullable(futureMap.get(key)).ifPresent(future -> {
            future.cancel(true);
            futureMap.remove(key);
        });
        log.info("一个客户端移除......" + ctx.channel().remoteAddress());
        ctx.close(); //关闭连接
    }

    /**
     * 发生异常时执行的操作
     *
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        String key = ctx.channel().id().asLongText();
        //移除通信过的channel
        channelMap.remove(key);
        //移除和用户绑定的channel
        clientMap.remove(key);
        //移除定时任务
        Optional.ofNullable(futureMap.get(key)).ifPresent(future -> {
            future.cancel(true);
            futureMap.remove(key);
        });
        //关闭长连接
        ctx.close();
        log.info("异常发生 " + cause.getMessage());
    }

    public static Map<String, Channel> getChannelMap() {
        return channelMap;
    }

    public static Map<String, Future> getFutureMap() {
        return futureMap;
    }

    public static Map<String, Long> getClientMap() {
        return clientMap;
    }
}
```

4.7 channel 中任务队列的线程任务
----------------------

```
/**
 * @Author WDYin
 * @Date 2021/8/10
 * @Description websocket程序
 **/
@Slf4j
@Component
public class WebsocketApplication {

    @Resource
    private WebsocketInitialization websocketInitialization;

    @PostConstruct
    public void start() {
        try {
            log.info(Thread.currentThread().getName() + ":websocket启动中......");
            websocketInitialization.init();
            log.info(Thread.currentThread().getName() + ":websocket启动成功！！！");
        } catch (Exception e) {
            log.error("websocket发生错误：",e);
        }
    }
}
```

4.8 websocket 启动程序
------------------

```
/**
 * @Author WDYin
 * @Date 2021/8/10
 * @Description websocket程序
 **/
@Slf4j
@Component
public class WebsocketApplication {

    @Resource
    private WebsocketInitialization websocketInitialization;

    @PostConstruct
    public void start() {
        try {
            log.info(Thread.currentThread().getName() + ":websocket启动中......");
            websocketInitialization.init();
            log.info(Thread.currentThread().getName() + ":websocket启动成功！！！");
        } catch (Exception e) {
            log.error("websocket发生错误：",e);
        }
    }
}
```

4.9 问题六解决方案
-----------

```
/**
 * @Author WDYin
 * @Date 2021/9/12
 * @Description
 **/
@RequestMapping("index")
@Controller
public class WebsocketController {

    /**
     *
     * @param id 用户主键
     * @param idList 要把消息发送给其他用户的主键
     */
    @RequestMapping("hello1")
    private void hello(Long id, List<Long> idList){
        //获取所有连接的客户端,如果是集群环境使用redis的hash数据类型存储即可
        Map<String, Channel> channelMap = WebSocketHandler.getChannelMap();
        //获取与用户主键绑定的channel,如果是集群环境使用redis的hash数据类型存储即可
        Map<String, Long> clientMap = WebSocketHandler.getClientMap();
        //解决问题六,websocket集群中一个客户端向其他客户端主动发送消息，如何实现？
        clientMap.forEach((k,v)->{
            if (idList.contains(v)){
                Channel channel = channelMap.get(k);
                channel.eventLoop().execute(() -> channel.writeAndFlush(new TextWebSocketFrame(Thread.currentThread().getName()+"服务器时间" + LocalDateTime.now() + "wdy")));
            }
        });
    }
}
```

4.10 前端代码
---------

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<script>
    var socket;
    //判断当前浏览器是否支持websocket
    if(window.WebSocket) {
        socket = new WebSocket("ws://localhost:7000/msg");
        //相当于channelReado, ev 收到服务器端回送的消息
        socket.onmessage = function (ev) {
            var rt = document.getElementById("responseText");
            rt.value = rt.value + "\n" + ev.data;
        }

        //相当于连接开启(感知到连接开启)
        socket.onopen = function (ev) {
            var rt = document.getElementById("responseText");
            rt.value = "连接开启了.."
        }

        //相当于连接关闭(感知到连接关闭)
        socket.onclose = function (ev) {

            var rt = document.getElementById("responseText");
            rt.value = rt.value + "\n" + "连接关闭了.."
        }
    } else {
        alert("当前浏览器不支持websocket")
    }

    //发送消息到服务器
    function send(websocketMessage) {
        if(!window.socket) { //先判断socket是否创建好
            return;
        }
        if(socket.readyState == WebSocket.OPEN) {
            //通过socket 发送消息
            socket.send(websocketMessage)
        } else {
            alert("连接没有开启");
        }
    }
</script>
    <form onsubmit="return false">
        <textarea ></textarea>
        <input type="button" value="发生消息" onclick="send(this.form.websocketMessage.value)">
        <textarea id="responseText" style="height: 300px; width: 300px"></textarea>
        <input type="button" value="清空内容" onclick="document.getElementById('responseText').value=''">
    </form>
</body>
</html>
```

4.11 wesocket 在 nginx 中配置
-------------------------

nginx.conf 中的配置

```
#第一步：
upstream websocket-router {
        server 192.168.1.31:7000 max_fails=10 weight=1 fail_timeout=5s;
        keepalive 1000;
}
#第二步：
server {
        listen      80; #监听80端口
        server_name websocket.wdy.com; #域名配置

        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;

        location / {
            client_max_body_size 100M;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header Upgrade $http_upgrade; #支持wss
            proxy_set_header Connection "upgrade"; #支持wssi
            proxy_pass http://websocket-router; #代理路由
            root   html;
            index  index.html index.htm;
        }
    }
```

4.12 效果图  

-----------

springboot 启动后，用浏览器打开前端页面：  

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsiaUGQfNQRQoVrV2or4HmfiaSdI9dt3BLibqctGFc8T1OdRJYDSwvS31NrRPSxico3Ezxgq4ujsgs1l9w/640)

本文主要讲述了网络编程相关的 IO 模型以及 NIO 框架 netty 如何搭建 websocket。本文作者：王德印，欢迎复制下方链接关注博主动态。

```
链接：https://blog.csdn.net/qq_41889508/article/details/105953114
```