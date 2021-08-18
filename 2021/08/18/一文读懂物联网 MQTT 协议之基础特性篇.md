> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU0OTE4MzYzMw==&mid=2247518411&idx=5&sn=b42bee8c9bfb92a37e98dcbe935599da&chksm=fbb10135ccc6882371cf5b31f6155cd9d38e41d0009fa78f751a79e2464cc6e3927f4b974da8&mpshare=1&scene=1&srcid=0817DIabdoV4X9ZEQbVoJUDM&sharer_sharetime=1629168358179&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

一、前言
----

上个月有个读者问我物联网 MQTT 协议实战相关的问题，我说后面会搞，没想到不知不觉一个月了，太忙了，再怎么忙答应的事情还是要给读者一个交代，所以就有了此文。

![](https://mmbiz.qpic.cn/mmbiz_png/80icw67Ot0qJm3aoCj3RslLDZribdCno4dboKuDI11N1icqiatsZjXh81InxiabRklU5rM3sxZibhbUSv1icvWA9Q5EYA/640?wx_fmt=png)

二、MQTT 协议概要
-----------

**2.1 什么是 MQTT 协议**

MQTT（Message Queuing Telemetry Transport，消息队列遥测传输协议），是一种基于发布 / 订阅（publish/subscribe）模式的 “轻量级” 通讯协议，该协议构建于 TCP/IP 协议上，由 IBM 于 1999 年发明。MQTT 协议的主要特征是开放、简单、轻量级和易于实现，这些特征使得它适用于受约束的应用环境，如：

*   `网络受限`：网络带宽较低且传输不可靠
    
*   `终端受限`：协议运行在嵌入式设备上，嵌入式终端的处理器、内存等是受限的
    

通过 MQTT 协议，目前已经扩展出了数十种 MQTT 服务器端程序，可以通过 PHP、Java、Python、C、C# 等语言向 MQTT 发送消息。由于开放源代码、耗电量小等特点，MQTT 非常适用于物联网领域，如传感器与服务器的通信、传感器信息采集等。

**2.2 发布 / 订阅模式**

发布 / 订阅模式并不是 MQTT 协议特有的模式，像我们很多消息中间件都有使用发布 / 订阅模式，这里你是不是想说，这不就是我们所说的观察者模式嘛，还真不是，这两个模式很容易混淆。观察者模式只有 观察者 + 被观察者两个角色，而发布 / 订阅模式还有一个经纪人 Broker；往更深层次的讲观察者和被观察者，是松耦合的关系，而发布者和订阅者，则完全不存在耦合。

在客户端 / 服务器模型中，客户端直接与服务器端点通信。而发布 / 订阅模式 pub/sub 就不一样了，发布 / 订阅模式会将发送消息的发布者 publisher 与接收消息的订阅者 subscribers 进行分离，publisher 与 subscribers 并不会直接通信，他们甚至都不清楚对方是否存在，他们之间的交流由第三方组件 broker 代理。

![](https://mmbiz.qpic.cn/mmbiz_png/80icw67Ot0qJm3aoCj3RslLDZribdCno4dtcgI6O3A2v02zRkGuPXbeupbyMprabSwASltlQ7qwmhRMWq3U66ASg/640?wx_fmt=png)  
pub/sub 最重要的方面是消息的发布者与接收者（订阅者）的解耦。这种解耦有几个维度：

  

*   空间解耦：发布者和订阅者不需要相互了解（例如，不交换 IP 地址和端口）。
    
*   时间解耦：发布者和订阅者不需要同时运行。
    
*   同步解耦：两个组件的操作在发布或接收时不需要中断。
    

总之，发布 / 订阅模式消除了传统客户端 / 服务器之间的直接通信，把通信这个操作交给了 broker 进行代理，并在空间、时间、同步三个维度上进行了解耦。

**2.3 可扩展性**

pub/sub 比传统的客户端 / 服务器模式有了更好的拓展，这是由于 broker 的高度并行化，并且是基于事件驱动的模式。可扩展性还体现在消息的缓存和消息的智能路由，还可以通过集群代理来实现数百万的连接，使用负载均衡器将负载分配到更多的单个服务器上，这就是 MQTT 的深度应用了。

**2.4 消息过滤**

很明显，broker 在 pub/sub 过程中起着举足轻重的作用。但是代理如何过滤所有消息，以便每个订阅者只接收感兴趣的消息？broker 有几个可以过滤的选项：

*   `基于主题的过滤`  
    此过滤基于属于每条消息的主题。接收客户端向代理订阅感兴趣的主题，订阅后，broker 就会确保客户端收到发布到 topic 中的消息。
    
*   `基于内容的过滤`  
    在基于内容的过滤中，broker 会根据特定的内容过滤消息，接受客户端会经过过滤他们感兴趣的内容。这种方法的一个显著的缺点就是必须事先知道消息的内容，不能加密或者轻易修改。
    
*   `基于类型的过滤`  
    当使用面向对象的语言时，基于消息（事件）的类型 / 类进行过滤是一种常见做法。例如，订阅者可以收听所有类型为 Exception 或任何子类型的消息。
    

**2.5 MQTT 与消息队列的区别**

这里你又会说了，既然 MQTT 与主流的消息的队列都采用发布 / 订阅模式，那他们就是一样的。这里老周得再提一嘴，确实和消息队列很多相似的地方，但还有有些差异的，下面就来说道说道：

*   `消息队列存储消息直到消息被消费` 使用消息队列时，每条传入消息都存储在队列中，直到被客户端（通常称为消费者）接收。如果没有客户端接收到消息，消息将保持在队列中并等待被消费。在消息队列中，不会存在消息没有客户端消费的情况，但是在 MQTT 中，却存在 topic 无 subscriber 订阅的情况。
    
*   `一条消息只被一个客户端消费` 另一个很大的区别是，在传统的消息队列中，一条消息只能被一个消费者处理。负载分布在队列的所有消费者之间。在 MQTT 中，行为完全相反：订阅主题的每个订阅者都会收到消息，每个订阅者有相同的负载。
    
*   `队列是命名的，必须显式创建`  队列比主题严格得多。在使用队列之前，必须使用单独的命令显式创建队列。只有在队列命名和创建之后，才可以发布或消费消息。相比之下，MQTT 主题非常灵活，可以即时创建。
    

三、MQTT 重要概念
-----------

**3.1 MQTT Client**

publisher 和 subscriber 都属于 MQTT Client，之所以有发布者和订阅者这个概念，其实是一种相对的概念，就是指当前客户端是在发布消息还是在接收消息，发布和订阅的功能也可以由同一个 MQTT Client 实现。

MQTT 客户端是运行 MQTT 库并通过网络连接到 MQTT 代理的任何设备（从微控制器到成熟的服务器）。例如，MQTT 客户端可以是一个非常小的、资源受限的设备，它通过无线网络进行连接并具有一个最低限度的库。基本上，任何使用 TCP/IP 协议使用 MQTT 设备的都可以称之为 MQTT Client。MQTT 协议的客户端实现非常简单直接，易于实施是 MQTT 非常适合小型设备的原因之一。MQTT 客户端库可用于多种编程语言。例如，Android、Arduino、C、C++、C#、Go、iOS、Java、JavaScript 和 .NET。

**3.2 MQTT Broker**

与 MQTT Client 对应的就是 MQTT Broker，Broker 是任何发布 / 订阅协议的核心，根据实现的不同，代理可以处理多达数百万连接的 MQTT Client。

Broker 负责接收所有消息，过滤消息，确定是哪个 Client 订阅了每条消息，并将消息发送给对应的 Client，Broker 还负责保存会话数据，这些数据包括订阅的和错过的消息。Broker 还负责客户端的身份验证和授权。

**3.3 MQTT Connection**

MQTT 协议基于 TCP/IP。客户端和代理都需要有一个 TCP/IP 协议支持。

![](https://mmbiz.qpic.cn/mmbiz_png/80icw67Ot0qJm3aoCj3RslLDZribdCno4dYM1pWbgMr28SREev4Y4Vd4YCpiaGsIfuys1MAicQTsexRmIdEnDjibSLw/640?wx_fmt=png)  

MQTT 连接始终位于一个客户端和代理之间。客户端从不直接相互连接。要发起连接，客户端向代理发送 CONNECT 消息。代理使用 CONNACK 消息和状态代码进行响应。建立连接后，代理将保持打开状态，直到客户端发送断开连接命令或连接中断。

![](https://mmbiz.qpic.cn/mmbiz_png/80icw67Ot0qJm3aoCj3RslLDZribdCno4dI4BIh0btiacoI7bQibiaXMetGD6YQgU2CpX17DGls2hTFhvibn67ukwUnA/640?wx_fmt=png)

四、消息列表
------

**4.1 CONNECT**

为了创建连接，客户端向代理发送命令消息。如果此 CONNECT 消息格式错误（根据 MQTT 规范）或打开网络套接字和发送连接消息之间的时间过长，代理将关闭连接。

一个 MQTT 客户端发送一条 CONNECT 连接，这条 CONNECT 连接可能会包含下面这些信息：

![](https://mmbiz.qpic.cn/mmbiz_png/80icw67Ot0qJm3aoCj3RslLDZribdCno4d7R5Yyicda1dV6bbjd8ytbRLuJ4ibWgPSZDricE9UibKicOmpVMSnl5EGwbw/640?wx_fmt=png)  
我们将重点关注以下选项：

*   `ClientId：ClientId` 的长度可以是 1-23 个字符，在一个服务器上 ClientId 不能重复。如果超过 23 个字符，则服务器返回 CONNACK 消息中的返回码为 Identifier Rejected。在 MQTT 3.1.1 中，如果您不需要代理持有状态，您可以发送一个空的 ClientId。空的 ClientId 导致连接没有任何状态。在这种情况下，clean session 标志必须设置为 true，否则代理将拒绝连接。
    
*   `Clean Session`：Clean Session 标志告诉代理客户端是否要建立持久会话。在持久会话 (CleanSession = false) 中，代理存储客户端的所有订阅以及以服务质量（QoS）级别 1 或 2 订阅的客户端的所有丢失消息。如果会话不是持久的 (CleanSession = true )，代理不为客户端存储任何内容，并清除任何先前持久会话中的所有信息。
    
*   `Username/Password`：MQTT 可以发送用户名和密码进行客户端认证和授权。但是，如果此信息未加密或散列，则密码将以纯文本形式发送。我们强烈建议将用户名和密码与安全传输一起使用。像 HiveMQ 这样的代理可以使用 SSL 证书对客户端进行身份验证，因此不需要用户名和密码。
    
*   `Will Message`：LastWillxxx 表示的是遗愿，client 在连接 broker 的时候将会设立一个遗愿，这个遗愿会保存在 broker 中，当 client 因为非正常原因断开与 broker 的连接时，broker 会将遗愿发送给订阅了这个 topic（订阅遗愿的 topic）的 client。
    
*   `KeepAlive`：keepAlive 是 client 在连接建立时与 broker 通信的时间间隔，通常以秒为单位。这个时间指的是 client 与 broker 在不发送消息下所能承受的最大时长。
    

**4.2 CONNACK**

当 broker 收到 CONNECT 消息时，它有义务回复 CONNACK 消息进行响应。CONNACK 消息包括两部分内容：

*   `The session present flag`：会话当前标志
    
*   `A connect return code`：连接返回码
    

![](https://mmbiz.qpic.cn/mmbiz_png/80icw67Ot0qJm3aoCj3RslLDZribdCno4dKKbzLBC5t1fevUAjLqGcUqErmjibY8wIdicMIic12B3NpYrMlJgPapBQA/640?wx_fmt=png)

*   `Session Present flag`
    
    会话当前标志，这个标志会告诉 client 当前 broker 是否有一个持久性会话与 client 进行交互。SessionPresent 标志和 CleanSession 标志有关，当 client 在 CleanSession 设置为 true 的情况下连接时，SessionPresent 始终为 false，因为没有持久性会话可以使用。如果 CleanSession 设置为 false，则有两种可能性，如果 ClientId 的会话信息可用，并且 broker 已经存储了会话信息，那么 SessionPresent 为 true，否则如果没有 ClientId 的任何会话信息，那么 SessionPresent 为 false。
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/80icw67Ot0qJm3aoCj3RslLDZribdCno4dRfkJMlrGTdRicjPFC50rzPM4WuWWqpGnFOXrj58dX8dziaj0lXw0PibLQ/640?wx_fmt=png)
    
      
    
*   `Connect return code`
    
    CONNACK 消息中的第二个标志是连接确认标志。这个标志包含一个返回码，告诉客户端连接尝试是否成功。连接确认标志有下面这些选项：
    

![](https://mmbiz.qpic.cn/mmbiz_png/80icw67Ot0qJm3aoCj3RslLDZribdCno4dzhTUPhLPQse6zRibSm0eM22UPHzqYKAicAjTXVakCN55lpFa4CYk3xjw/640?wx_fmt=png)  
**4.3 PUBLISH**

MQTT 客户端可以在连接到 broker 后立即发布消息，MQTT 使用的是基于 topic 主题的过滤。每条消息都必须包含一个主题，broker 可以使用该主题将消息转发给感兴趣的客户端。通常，每条消息都有一个负载（Payload），其中包含要以字节格式传输的数据。MQTT 是数据无关性的，也就是说数据是由发布者 - publisher 决定要发送的是 XML 、JSON 还是二进制数据、文本数据。

MQTT 中的 PUBLISH 消息有几个我们想要详细讨论的属性：

![](https://mmbiz.qpic.cn/mmbiz_png/80icw67Ot0qJm3aoCj3RslLDZribdCno4de81fzUW4d2ua5uqu19Ic6ib9NTtAIqHlEVHicPkVjib78sTebTZjREQOg/640?wx_fmt=png)

*   `Topic Name`：主题名称是一个简单的字符串，它以正斜杠作为分隔符进行分层结构。例如，“我的家 / 客厅 / 温度” 或 “德国 / 慕尼黑 / 十月节 / 人”。
    
*   `QoS`：此数字表示消息的服务质量 (QoS)。有三个级别：0、1 和 2。服务级别决定了消息到达预期接收者（客户端或代理）的保证类型。
    
*   `Retain Flag`：此标志表示 broker 将最近收到的一条 RETAIN 标志位为 true 的消息保存在服务器端（内存或者文件）。
    
*   `Payload`：这个是每条消息的实际内容。MQTT 是数据无关性的。可以发送任何文本、图像、加密数据以及二进制数据。
    
*   `Packet Identifier`：这个 packetId 标识在 client 和 broker 之间唯一的消息标识。packetId 仅与大于零的 QoS 级别相关。
    
*   `DUP flag`：该标志表明该消息是重复的并且由于预期的接收者（客户端或代理）没有确认原始消息而被重新发送。这仅与 QoS 大于 0 相关。
    

当客户端向 MQTT broker 发送消息进行发布时，broker 读取消息、确认消息（根据 QoS 级别）并处理消息。broker 的处理包括确定哪些客户端订阅了主题并将消息发送给他们。

![](https://mmbiz.qpic.cn/mmbiz_png/80icw67Ot0qJm3aoCj3RslLDZribdCno4dsRAYq0O3iaiaYp61rxw52Fib7uia4NtFMM9A3ObFABQY3aoK2JcZtkZBqg/640?wx_fmt=png)  
最初发布消息的客户端只关心将 PUBLISH 消息传递给 broker。一旦 broker 收到 PUBLISH 消息，broker 就有责任将消息传递给所有订阅者。发布客户端不会得到关于是否有人对发布的消息感兴趣或有多少客户端从 broker 收到消息的任何反馈。

**4.4 Subscribe**

client 会向 broker 发送 SUBSCRIBE 消息来接收有关感兴趣的 topic，这个 SUBSCRIBE 消息非常简单，它包含了一个唯一的数据包标识和一个订阅列表。

![](https://mmbiz.qpic.cn/mmbiz_png/80icw67Ot0qJm3aoCj3RslLDZribdCno4dDp1poPICEntQ8RcVDuXb66CsItInwuCf8wzyB5aML6zlqnKwr0ywIQ/640?wx_fmt=png)

  

*   Packet Identifier：这个 PacketId 和上面的 PacketId 一样，都表示消息的唯一标识符。
    
*   List of Subscriptions：一个 SUBSCRIBE 消息可以包含一个客户端的多个订阅。每个订阅由一个主题和一个 QoS 级别组成。订阅消息中的主题可以包含通配符，使订阅主题模式而不是特定主题成为可能。如果一个客户端存在重叠订阅，则代理会传送该主题具有最高 QoS 级别的消息。
    

**4.5 Suback**

为了确认每个订阅，broker 向客户端发送一个 SUBACK 确认消息。该消息包含原始 Subscribe 消息的数据包标识符（以明确标识该消息）和返回码列表。

![](https://mmbiz.qpic.cn/mmbiz_png/80icw67Ot0qJm3aoCj3RslLDZribdCno4djH2Vt0cJGiaxmDdUF65eZTePKqcrkVJk6PHZ3FyPks3VJY3wweqzyAQ/640?wx_fmt=png)

*   `Packet Identifier`：包标识符是用于标识消息的唯一标识符。它与 SUBSCRIBE 消息中的相同。
    
*   `Return Code`：broker 为它在 SUBSCRIBE 消息中收到的每个主题 / QoS 对发送一个返回代码。例如，如果 SUBSCRIBE 消息有五个订阅，则 SUBACK 消息包含五个返回码。返回码确认每个主题并显示 broker 授予的 QoS 级别。如果 broker 拒绝订阅，则 SUBACK 消息包含该特定主题的失败返回代码。例如，如果客户端没有足够的权限订阅主题或主题格式错误。
    
*   ![](https://mmbiz.qpic.cn/mmbiz_png/80icw67Ot0qJm3aoCj3RslLDZribdCno4dvNpm1Kh83jUrBHyIFAphGqvguYuaErqwaCOwR9SIXAX1ruDwVLusicg/640?wx_fmt=png)
    
      
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/80icw67Ot0qJm3aoCj3RslLDZribdCno4dYr4liaFoia5wAhts0Kpia76Or4c26AScsW7x0IUdoNkM45omzxVdibG7dw/640?wx_fmt=png)
    
      
    
      
    客户端成功发送 SUBSCRIBE 消息并收到 SUBACK 消息后，它会获取与 SUBSCRIBE 消息包含的订阅中的主题匹配的每条已发布消息。
    

**4.6 Unsubscribe**

SUBSCRIBE 消息的对应是 UNSUBSCRIBE 消息。此消息删除 broker 上客户端的现有订阅。UNSUBSCRIBE 消息与 SUBSCRIBE 消息类似，具有数据包标识符和主题列表。

![](https://mmbiz.qpic.cn/mmbiz_png/80icw67Ot0qJm3aoCj3RslLDZribdCno4dY337iaQGyvvLaBu8woKmRibtn72j3dtE7cul0iaPibByCz9Hweoy7gvbicw/640?wx_fmt=png)

**4.7 Unsuback**

为了确认取消订阅，broker 向客户端发送一个 UNSUBACK 确认消息。此消息仅包含原始 UNSUBSCRIBE 消息的数据包标识符（以明确标识该消息）。

![](https://mmbiz.qpic.cn/mmbiz_png/80icw67Ot0qJm3aoCj3RslLDZribdCno4dMiabGRZStHp6pIazSH5xvPQ7DXzSgmsz0ia9fAFiaaOKuRiaTQEsNvDSVg/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_png/80icw67Ot0qJm3aoCj3RslLDZribdCno4dic6tLs8eRL3iaUKRsQpOlvRrjqv5pabJpIYyE8ak4mdmFdicaBvb7bj0Q/640?wx_fmt=png)  
客户端收到来自 broker 的 UNSUBACK 后，可以认为 UNSUBSCRIBE 消息中的订阅被删除了。

五、Topics
--------

前面我们说了很多 MQTT 协议的格式以及消息列表，这一节我们来说下 Topics 主题。主题在 MQTT 中很重要，因为我们写代码的时候往往都是需要先确认好 MQTT 的 Topics。

在 MQTT 中，主题一词是指 broker 用于为每个连接的客户端过滤消息的 UTF-8 字符串。主题由一个或多个主题级别组成。每个主题级别由正斜杠（主题级别分隔符）分隔。

![](https://mmbiz.qpic.cn/mmbiz_png/80icw67Ot0qJm3aoCj3RslLDZribdCno4d1zqSQuxg2Rm1tgMNnhaudTwhaziaGBCmROdq2vAwyTgicI25d6QmAhRQ/640?wx_fmt=png)  
与消息队列相比，MQTT 主题非常轻量级。客户端在发布或订阅它之前不需要创建所需的主题。broker 接受每个有效主题而无需任何事先初始化。

**5.1 通配符**

当客户端订阅主题时，它可以订阅已发布消息的确切主题，也可以使用通配符同时订阅多个主题。通配符只能用于订阅主题，不能用于发布消息。有两种不同类型的通配符：单级和多级。

*   `单级：+`
    

    顾名思义，单级通配符替换一个主题级别。加号代表主题中的单级通配符。  
       

![](https://mmbiz.qpic.cn/mmbiz_png/80icw67Ot0qJm3aoCj3RslLDZribdCno4dlS4qo8cbKiacwFF83olXxc0D3Kc9gyevaudNhgG9BvHWSXkJECwv8rg/640?wx_fmt=png) 如果主题包含任意字符串而不是通配符，则任何主题都与具有单级通配符的主       题匹配。例如，订阅 myhome/groundfloor/+/temperature 可以产生以下结果：

![](https://mmbiz.qpic.cn/mmbiz_png/80icw67Ot0qJm3aoCj3RslLDZribdCno4dPvBTetnwGbu0Yj0Yog9JZQE91Sd3EvmdFWiar8X9wpG7IPnibOP4bdNA/640?wx_fmt=png)

*   `多级：#`
    
    多级通配符涵盖多个主题级别。哈希符号代表主题中的多级通配符。为了让代理确定哪些主题匹配，多级通配符必须作为主题中的最后一个字符放置，并以正斜杠开头。  
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/80icw67Ot0qJm3aoCj3RslLDZribdCno4d1lLOGlj8ic1uwKPXMwfF65bf5LXu3xXdbZCKMJFSHLN5fKQFOaMK7qw/640?wx_fmt=png)
    
      
    
      
    当客户端订阅带有多级通配符的主题时，无论主题多长或多深，它都会收到以通配符之前的模式开头的主题的所有消息。如果您仅将多级通配符指定为主题 (#)，您将收到发送到 MQTT 代理的所有消息。如果您期望高吞吐量，单独使用多级通配符订阅是一种反模式（请参阅下面的最佳实践）。
    

**5.2 以 $ 开头的主题**

******通常，您可以根据需要命名 MQTT 主题。但是，有一个例外：以 符号开头的主题具有不同的目的。当您将多级通配符作为主题 (#) 订阅时，这些主题不是订阅的一部分。`$-symbol` 主题保留用于 MQTT 代理的内部统计信息。客户端无法向这些主题发布消息。目前，此类主题尚无官方标准化。通常，`$SYS/`用于所有以下信息，但代理实现各不相同。MQTT GitHub wiki 中提供了对 `$SYS-topics` 的一项建议 。这里有些例子：******

> $SYS/broker/clients/connected  
>  $SYS/broker/clients/disconnected  
>  $SYS/broker/clients/total  
>  $SYS/broker/messages/sent  
>  $SYS/broker/uptime

* * *

可以呀，看到了最后面。授人以鱼不如授人以渔，下面是一个关于 MQTT Version 3.1.1 的介绍，有些协议格式详细的可以前往查看。

https://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html#_Toc398718035

本文分基础特性篇与实战篇来讲，下一篇老周会带你搭建一个 MQTT 服务器，让其他厂商的设备接入进来，尽情期待~