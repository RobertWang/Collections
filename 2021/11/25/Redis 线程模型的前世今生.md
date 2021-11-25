> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/MwG9GpnTVbma69PiOHHvgw)

一、概述

众所周知，Redis 是一个高性能的数据存储框架，在高并发的系统设计中，Redis 也是一个比较关键的组件，是我们提升系统性能的一大利器。深入去理解 Redis 高性能的原理显得越发重要，当然 Redis 的高性能设计是一个系统性的工程，涉及到很多内容，本文重点关注 Redis 的 IO 模型，以及基于 IO 模型的线程模型。

我们从 IO 的起源开始，讲述了阻塞 IO、非阻塞 IO、多路复用 IO。基于多路复用 IO，我们也梳理了几种不同的 Reactor 模型，并分析了几种 Reactor 模型的优缺点。基于 Reactor 模型我们开始了 Redis 的 IO 模型和线程模型的分析，并总结出 Redis 线程模型的优点、缺点，以及后续的 Redis 多线程模型方案。本文的重点是对 Redis 线程模型设计思想的梳理，捋顺了设计思想，就是一通百通的事了。

> 注：本文的代码都是伪代码，主要是为了示意，不可用于生产环境。

二、网络 IO 模型发展史

我们常说的网络 IO 模型，主要包含阻塞 IO、非阻塞 IO、多路复用 IO、信号驱动 IO、异步 IO，本文重点关注跟 Redis 相关的内容，所以我们重点分析阻塞 IO、非阻塞 IO、多路复用 IO，帮助大家后续更好的理解 Redis 网络模型。

我们先看下面这张图；

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5Z10x8K04Dm5HVpHeMHDSf9kLvDXtMQx0FjQzZeUO0exqjnWibj3r6bibwOqcQ2aJnjUIke0Ubz0yA/640?wx_fmt=png)

2.1 阻塞 IO

我们经常说的阻塞 IO 其实分为两种，一种是单线程阻塞，一种是多线程阻塞。这里面其实有两个概念，阻塞和线程。

*   阻塞：指调用结果返回之前，当前线程会被挂起，调用线程只有在得到结果之后才会返回；
    
*   线程：系统调用的线程个数。
    

像建立连接、读、写都涉及到系统调用，本身是一个阻塞的操作。

**2.1.1 单线程阻塞**

服务端单线程来处理，当客户端请求来临时，服务端用主线程来处理连接、读取、写入等操作。

以下用代码模拟了单线程的阻塞模式；

```
import java.net.Socket;
public class BioTest {
    public static void main(String[] args) throws IOException {
        ServerSocket server=new ServerSocket(8081);
        while(true) {
            Socket socket=server.accept();
            System.out.println("accept port:"+socket.getPort());
            BufferedReader  in=new BufferedReader(new InputStreamReader(socket.getInputStream()));
            String inData=null;
            try {
                while ((inData = in.readLine()) != null) {
                    System.out.println("client port:"+socket.getPort());
                    System.out.println("input data:"+inData);
                    if("close".equals(inData)) {
                        socket.close();
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                try {
                    socket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }      
        }
    }
}
```

我们准备用两个客户端同时发起连接请求、来模拟单线程阻塞模式的现象。同时发起连接，通过服务端日志，我们发现此时服务端只接受了其中一个连接，主线程被阻塞在上一个连接的 read 方法上。

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5Z10x8K04Dm5HVpHeMHDSfucPTLKPRKkGToJ4HR7C1RbbP6K0tZ2rkibFyUCanHzwUqICRqMdgAQQ/640?wx_fmt=png)

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5Z10x8K04Dm5HVpHeMHDSfFhT3GZcrOGmzvXZ8Hw4FGbicKSHVYdnNc5ArianXDpSgb0V5J08wyfgw/640?wx_fmt=png)

我们尝试关闭第一个连接，看第二个连接的情况，我们希望看到的现象是，主线程返回，新的客户端连接被接受。

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5Z10x8K04Dm5HVpHeMHDSfrwJsFAIic8PFoMNiarJbMM2tQ4F0o0yvv7J2aOoSreuEsYYeO1lVJaoQ/640?wx_fmt=png)

从日志中发现，在第一个连接被关闭后，第二个连接的请求被处理了，也就是说第二个连接请求在排队，直到主线程被唤醒，才能接收下一个请求，符合我们的预期。

**此时不仅要问，为什么呢？**

主要原因在于 accept、read、write 三个函数都是阻塞的，主线程在系统调用的时候，线程是被阻塞的，其他客户端的连接无法被响应。

通过以上流程，我们很容易发现这个过程的缺陷，服务器每次只能处理一个连接请求，CPU 没有得到充分利用，性能比较低。如何充分利用 CPU 的多核特性呢？自然而然的想到了——**多线程逻辑。**

**2.1.2 多线程阻塞**

对工程师而言，代码解释一切，直接上代码。

**BIO 多线程**

```
package net.io.bio;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.ServerSocket;
import java.net.Socket;
public class BioTest {
    public static void main(String[] args) throws IOException {
        final ServerSocket server=new ServerSocket(8081);
        while(true) {
            new Thread(new Runnable() {
                public void run() {
                    Socket socket=null;
                    try {
                        socket = server.accept();
                        System.out.println("accept port:"+socket.getPort());
                        BufferedReader  in=new BufferedReader(new InputStreamReader(socket.getInputStream()));
                        String inData=null;
                        while ((inData = in.readLine()) != null) {
                            System.out.println("client port:"+socket.getPort());
                            System.out.println("input data:"+inData);
                            if("close".equals(inData)) {
                                socket.close();
                            }
                        }
                    } catch (IOException e) {
                        e.printStackTrace();
                    } finally {
                    }
                }
            }).start();
        }
    }
}
```

同样，我们并行发起两个请求；

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5Z10x8K04Dm5HVpHeMHDSfG2LYhdTfGhNicqeqGUDs3l21sxU3QxlvHeB3Azt7WuprBEITCFI3oiaQ/640?wx_fmt=png)

两个请求，都被接受，服务端新增两个线程来处理客户端的连接和后续请求。

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5Z10x8K04Dm5HVpHeMHDSffgXm6RKt5bCyyic0KEteeicQsbzwpEdTefsPvxVIZEibYDqlAKuVYGjFw/640?wx_fmt=png)

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5Z10x8K04Dm5HVpHeMHDSfGeQjjuaqqvKOsWICoKziazOx62ibIF78P1lIia9ANPXHmDBmYTQeWZy5g/640?wx_fmt=png)

我们用多线程解决了，服务器同时只能处理一个请求的问题，但同时又带来了一个问题，如果客户端连接比较多时，服务端会创建大量的线程来处理请求，但线程本身是比较耗资源的，创建、上下文切换都比较耗资源，又如何去解决呢？

2.2 非阻塞

如果我们把所有的 Socket（文件句柄，后续用 Socket 来代替 fd 的概念，尽量减少概念，减轻阅读负担）都放到队列里，只用一个线程来轮训所有的 Socket 的状态，如果准备好了就把它拿出来，是不是就减少了服务端的线程数呢？

一起看下代码，单纯非阻塞模式，我们基本上不用，为了演示逻辑，我们模拟了相关代码如下；

```
package net.io.bio;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.ServerSocket;
import java.net.Socket;
import java.net.SocketTimeoutException;
import java.util.ArrayList;
import java.util.List;
import org.apache.commons.collections4.CollectionUtils;
public class NioTest {
    public static void main(String[] args) throws IOException {
        final ServerSocket server=new ServerSocket(8082);
        server.setSoTimeout(1000);
        List<Socket> sockets=new ArrayList<Socket>();
        while (true) {
            Socket socket = null;
            try {
                socket = server.accept();
                socket.setSoTimeout(500);
                sockets.add(socket);
                System.out.println("accept client port:"+socket.getPort());
            } catch (SocketTimeoutException e) {
                System.out.println("accept timeout");
            }
            //模拟非阻塞：轮询已连接的socket，每个socket等待10MS，有数据就处理，无数据就返回，继续轮询
            if(CollectionUtils.isNotEmpty(sockets)) {
                for(Socket socketTemp:sockets ) {
                    try {
                        BufferedReader  in=new BufferedReader(new InputStreamReader(socketTemp.getInputStream()));
                        String inData=null;
                        while ((inData = in.readLine()) != null) {
                            System.out.println("input data client port:"+socketTemp.getPort());
                            System.out.println("input data client port:"+socketTemp.getPort() +"data:"+inData);
                            if("close".equals(inData)) {
                                socketTemp.close();
                            }
                        }
                    } catch (SocketTimeoutException e) {
                        System.out.println("input client loop"+socketTemp.getPort());
                    }
                }
            }
        }
    }
}
```

系统初始化，等待连接；

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5Z10x8K04Dm5HVpHeMHDSfzkHzqAsRNfNc2ibMbsr1BUic97BOOdwBsh8ib2XANXIVL2K29RmW5pgQQ/640?wx_fmt=png)

发起两个客户端连接，线程开始轮询两个连接中是否有数据。

同时创建两个连接；

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/4g5IMGibSxt5Z10x8K04Dm5HVpHeMHDSf29WuhUvkIIM0FMzE1ibiavNy5HvjaAYp4h2vCCJokyu6YiaDiceGzicSXlw/640?wx_fmt=jpeg)

两个连接无阻塞的被创建；

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5Z10x8K04Dm5HVpHeMHDSfdDCSv1REbKVLicok7vMjShx5LyrPqQ7CicyDFuyMZPC4dlZqnlhbzyXw/640?wx_fmt=png)

当然操作系统的多路复用有好几种实现方式，我们经常使用的 select()，epoll 模式这里不做过多的解释，有兴趣的可以查看相关文档，IO 的发展后面还有异步、事件等模式，我们在这里不过多的赘述，我们更多的是为了解释 Redis 线程模式的发展。

三、NIO 线程模型解释

我们一起来聊了阻塞、非阻塞、IO 多路复用模式，那 Redis 采用的是哪种呢？

Redis 采用的是 IO 多路复用模式，所以我们重点来了解下多路复用这种模式，如何在更好的落地到我们系统中，不可避免的我们要聊下 Reactor 模式。

首先我们做下相关的名词解释；

*   **Reactor**：类似 NIO 编程中的 Selector，负责 I/O 事件的派发；
    
*   **Acceptor**：NIO 中接收到事件后，处理连接的那个分支逻辑；
    
*   **Handler**：消息读写处理等操作类。
    

3.1 单 Reactor 单线程模型

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5Z10x8K04Dm5HVpHeMHDSfUzyic1IUEVriaHcvrDicxMp6s0SP6RSzpNrdXGXOz5lE1xaqibwJgqlYHQ/640?wx_fmt=png)

**处理流程**

*   Reactor 监听连接事件、Socket 事件，当有连接事件过来时交给 Acceptor 处理，当有 Socket 事件过来时交个对应的 Handler 处理。
    

**优点**

*   模型比较简单，所有的处理过程都在一个连接里；
    
*   实现上比较容易，模块功能也比较解耦，Reactor 负责多路复用和事件分发处理，Acceptor 负责连接事件处理，Handler 负责 Scoket 读写事件处理。
    

**缺点**

*   只有一个线程，连接处理和业务处理共用一个线程，无法充分利用 CPU 多核的优势。
    
*   在流量不是特别大、业务处理比较快的时候系统可以有很好的表现，当流量比较大、读写事件比较耗时情况下，容易导致系统出现性能瓶颈。
    

怎么去解决上述问题呢？既然业务处理逻辑可能会影响系统瓶颈，那我们是不是可以把业务处理逻辑单拎出来，交给线程池来处理，一方面减小对主线程的影响，另一方面利用 CPU 多核的优势。这一点希望大家要理解透彻，方便我们后续理解 Redis 由单线程模型到多线程模型的设计的思路。

3.2 单 Reactor 多线程模型

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5Z10x8K04Dm5HVpHeMHDSfUzyic1IUEVriaHcvrDicxMp6s0SP6RSzpNrdXGXOz5lE1xaqibwJgqlYHQ/640?wx_fmt=png)

这种模型相对单 Reactor 单线程模型，只是将业务逻辑的处理逻辑交给了一个线程池来处理。

**处理流程**

*   Reactor 监听连接事件、Socket 事件，当有连接事件过来时交给 Acceptor 处理，当有 Socket 事件过来时交个对应的 Handler 处理。
    
*   Handler 完成读事件后，包装成一个任务对象，交给线程池来处理，把业务处理逻辑交给其他线程来处理。
    

**优点**

*   让主线程专注于通用事件的处理（连接、读、写），从设计上进一步解耦；
    
*   利用 CPU 多核的优势。
    

**缺点**

*   貌似这种模型已经很完美了，我们再思考下，如果客户端很多、流量特别大的时候，通用事件的处理（读、写）也可能会成为主线程的瓶颈，因为每次读、写操作都涉及系统调用。
    

有没有什么好的办法来解决上述问题呢？通过以上的分析，大家有没有发现一个现象，当某一个点成为系统瓶颈点时，想办法把他拿出来，交个其他线程来处理，那这种场景是否适用呢？

3.3 多 Reactor 多线程模型

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5Z10x8K04Dm5HVpHeMHDSfhA0rFoiaX6ChvRiaNlVs4Ps5Te1S8IcD6d1ugjxyGJkj2HqIib2NOIN5g/640?wx_fmt=png)

这种模型相对单 Reactor 多线程模型，只是将 Scoket 的读写处理从 mainReactor 中拎出来，交给 subReactor 线程来处理。

**处理流程**

*   mainReactor 主线程负责连接事件的监听和处理，当 Acceptor 处理完连接过程后，主线程将连接分配给 subReactor；
    
*   subReactor 负责 mainReactor 分配过来的 Socket 的监听和处理，当有 Socket 事件过来时交个对应的 Handler 处理；
    
*   Handler 完成读事件后，包装成一个任务对象，交给线程池来处理，把业务处理逻辑交给其他线程来处理。
    

**优点**

*   让主线程专注于连接事件的处理，子线程专注于读写事件吹，从设计上进一步解耦；
    
*   利用 CPU 多核的优势。
    

**缺点**

*   实现上会比较复杂，在极度追求单机性能的场景中可以考虑使用。
    

四、Redis 的线程模型

4.1 概述

以上我们聊了，IO 网路模型的发展历史，也聊了 IO 多路复用的 reactor 模式。那 Redis 采用的是哪种 reactor 模式呢？在回答这个问题前，我们先梳理几个概念性的问题。

Redis 服务器中有两类事件，文件事件和时间事件。

*   **文件事件**：在这里可以把文件理解为 Socket 相关的事件，比如连接、读、写等；
    
*   **时间时间**：可以理解为定时任务事件，比如一些定期的 RDB 持久化操作。
    

本文重点聊下 Socket 相关的事件。

4.2 模型图

首先我们来看下 Redis 服务的线程模型图；

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5Z10x8K04Dm5HVpHeMHDSfDIxRn7qGNGwQZ5R4MXyzphmvPXuQejvLdAKjicvSQyiaF6AfKShpUibYA/640?wx_fmt=png)

IO 多路复用负责各事件的监听（连接、读、写等），当有事件发生时，将对应事件放入队列中，由事件分发器根据事件类型来进行分发；

如果是连接事件，则分发至连接应答处理器；GET、SET 等 redis 命令分发至命令请求处理器。

命令处理完后产生命令回复事件，再由事件队列，到事件分发器，到命令回复处理器，回复客户端响应。

4.3 一次客户端和服务端的交互流程

**4.3.1 连接流程**

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5Z10x8K04Dm5HVpHeMHDSfutiaxz7KDZRKrx3azrMXzVh3BcIp6yc4CROTxDGFWQUTUXCUPicSQD1w/640?wx_fmt=png)

**连接过程**

*   Redis 服务端主线程监听固定端口，并将连接事件绑定连接应答处理器。
    
*   客户端发起连接后，连接事件被触发，IO 多路复用程序将连接事件包装好后丢人事件队列，然后由事件分发处理器分发给连接应答处理器。
    
*   连接应答处理器创建 client 对象以及 Socket 对象，我们这里关注 Socket 对象，并产生 ae_readable 事件，和命令处理器关联，标识后续该 Socket 对可读事件感兴趣，也就是开始接收客户端的命令操作。
    
*   当前过程都是由一个主线程负责处理。
    

**4.3.2 命令执行流程**

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5Z10x8K04Dm5HVpHeMHDSfsy6q07bvF7JDwicv48esnA3Z8RCoC7RfWtTJQjw9twfdYCVREyxT9GA/640?wx_fmt=png)

**SET 命令执行过程**

*   客户端发起 SET 命令，IO 多路复用程序监听到该事件后（读事件），将数据包装成事件丢到事件队列中（事件在上个流程中绑定了命令请求处理器）；
    
*   事件分发处理器根据事件类型，将事件分发给对应的命令请求处理器；
    
*   命令请求处理器，读取 Socket 中的数据，执行命令，然后产生 ae_writable 事件，并绑定命令回复处理器；
    
*   IO 多路复用程序监听到写事件后，将数据包装成事件丢到事件队列中，事件分发处理器根据事件类型分发至命令回复处理器；
    
*   命令回复处理器，将数据写入 Socket 中返回给客户端。
    

4.4 模型优缺点

以上流程分析我们可以看出 Redis 采用的是单线程 Reactor 模型，我们也分析了这种模式的优缺点，那 Redis 为什么还要采用这种模式呢？

**Redis 本身的特性**

*   命令执行基于内存操作，业务处理逻辑比较快，所以命令处理这一块单线程来做也能维持一个很高的性能。
    

**优点**

*   Reactor 单线程模型的优点，参考上文。
    

**缺点**

*   Reactor 单线程模型的缺点也同样在 Redis 中来体现，唯一不同的地方就在于业务逻辑处理（命令执行）这块不是系统瓶颈点。
    
*   随着流量的上涨，IO 操作的的耗时会越来越明显（read 操作，内核中读数据到应用程序。write 操作，应用程序中的数据到内核），当达到一定阀值时系统的瓶颈就体现出来了。
    

**Redis 又是如何去解的呢？**

哈哈~ 将耗时的点从主线程拎出来呗？那 Redis 的新版本是这么做的吗？我们一起来看下。

4.5 Redis 多线程模式

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5Z10x8K04Dm5HVpHeMHDSfbZJoXg6h9LibZ0KATdUib635GrGibj0y4Ctic5K1icXRuzhYgdnbybBgorQ/640?wx_fmt=png)

Redis 的多线程模型跟” 多 Reactor 多线程模型 “、“单 Reactor 多线程模型有点区别”，但同时用了两种 Reactor 模型的思想，具体如下；

*   Redis 的多线程模型是将 IO 操作多线程化，本身逻辑处理过程（命令执行过程）依旧是单线程，借助了单 Reactor 思想，实现上又有所区分。
    
*   将 IO 操作多线程化，又跟单 Reactor 衍生出多 Reactor 的思想一致，都是将 IO 操作从主线程中拎出来。
    

**命令执行大致流程**

*   客户端发送请求命令，触发读就绪事件，服务端主线程将 Socket（为了简化理解成本，统一用 Socket 来代表连接）放入一个队列，主线程不负责读；
    
*   IO 线程通过 Socket 读取客户端的请求命令，主线程忙轮询，等待所有 I/O 线程完成读取任务，IO 线程只负责读不负责执行命令；
    
*   主线程一次性执行所有命令，执行过程和单线程一样，然后需要返回的连接放入另外一个队列中，有 IO 线程来负责写出（主线程也会写）；
    
*   主线程忙轮询，等待所有 I/O 线程完成写出任务。
    

五、总结

了解一个组件，更多的是要去了解他的设计思路，要去思考为什么要这么设计，做这种技术选型的背景是啥，对后续做系统架构设计有什么参考意义等等。一通百通，希望对大家有参考意义。

END