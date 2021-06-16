> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI4OTA3NDQ0Nw==&mid=2455551233&idx=2&sn=b1d8253ad78307b04c0db79e7416e864&chksm=fb9ca161cceb28772325136369c027bdd6c9aa814a3510ecae85948e74aea12dc013b2332a3a&mpshare=1&scene=1&srcid=0615lq9yLa2ROeGV9AuIT1Dm&sharer_sharetime=1623767389696&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

来源：22j.co/cs2v

#### AMQP 协议

1.  server：又称 broker，接受客户端连接，实现 AMQP 实体服务。
    
2.  connection：连接和具体 broker 网络连接。
    
3.  channel：网络信道，几乎所有操作都在 channel 中进行，channel 是消息读写的通道。客户端可以建立多个 channel，每个 channel 表示一个会话任务。
    
4.  message：消息，服务器和应用程序之间传递的数据，由 properties 和 body 组成。properties 可以对消息进行修饰，比如消息的优先级，延迟等高级特性；body 是消息实体内容。
    
5.  Virtual host：虚拟主机，用于逻辑隔离，最上层消息的路由。一个 Virtual host 可以若干个 Exchange 和 Queue，同一个 Virtual host 不能有同名的 Exchange 或 Queue。
    
6.  Exchange：交换机，接受消息，根据路由键转发消息到绑定的队列上。
    
7.  banding：Exchange 和 Queue 之间的虚拟连接，binding 中可以包括 routing key
    
8.  routing key：一个路由规则，虚拟机根据他来确定如何路由 一条消息。
    
9.  Queue：消息队列，用来存放消息的队列。
    

##### Exchange

1.  Direct Exchange, 所有发送到 Direct Exchange 的消息被转发到 RouteKey 中指定的 Queue,Direct Exchange 可以使用默认的默认的 Exchange （default Exchange），默认的 Exchange 会绑定所有的队列，所以 Direct 可以直接使用 Queue 名（作为 routing key ）绑定。或者消费者和生产者的 routing key 完全匹配。
    
2.  Toptic Exchange, 是指发送到 Topic Exchange 的消息被转发到所有关心的 Routing key 中指定 topic 的 Queue 上。Exchange 将 routing key 和某 Topic 进行模糊匹配，此时队列需要绑定一个 topic。所谓模糊匹配就是可以使用通配符，“#”可以匹配一个或多个词，“”只匹配一个词比如 “log.#” 可以匹配“log.info.test” "log." 就只能匹配 log.error。
    
3.  Fanout Exchange: 不处理路由键，只需简单的将队列绑定到交换机上。发送到改交换机上的消息都会被发送到与该交换机绑定的队列上。Fanout 转发是最快的。
    

#### 消息如何保证 100％投递

##### 什么是生产端的可靠性投递？

1.  保证消息的成功发出
    
2.  保证 MQ 节点节点的成功接收
    
3.  发送端 MQ 节点（broker）收到消息确认应答
    
4.  完善消息进行补偿机制
    

###### 可靠性投递保障方案

消息落库，对消息进行打标

![](https://mmbiz.qpic.cn/mmbiz_png/XaklVibwUKn7LrlfnxN5ibcIcEUFwn9pNPPfpztc5KODQyynBBUcHlqMT63h2w3tbNDYNWrPbadTys9Ry4zDQ34g/640?wx_fmt=png)

在高并发场景下，每次进行 db 的操作都是每场消耗性能的。我们使用延迟队列来减少一次数据库的操作。  

##### 消息幂等性

> 幂等性是什么？点击[这篇文章](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247488612&idx=1&sn=da9e2759035d14ef1d9656ac9fd95971&chksm=eb539152dc2418445d74b384da03fccfc3ab47c3b0db11abc2f63d8c31692005a9812f7d709d&scene=21#wechat_redirect)看下。

我对一个动作进行操作，我们肯能要执行 100 次 1000 次，对于这 1000 次执行的结果都必须一样的。比如单线程方式下执行 update count-1 的操作执行一千次结果都是一样的，所以这个更新操作就是一个幂等的，如果是在并发不做线程安全的处理的情况下 update 一千次操作结果可能就不是一样的，所以并发情况下的 update 操作就不是一个幂等的操作。对应到消息队列上来，就是我们即使受到了多条一样的消息，也和消费一条消息效果是一样的。

##### 高并发的情况下如何避免消息重复消费

1.  唯一 id + 加指纹码，利用数据库主键去重。
    
    优点：实现简单
    
    缺点：高并发下有数据写入瓶颈。
    
2.  利用 Redis 的原子性来实习。
    
    使用 Redis 进行幂等是需要考虑的问题
    

*   是否进行数据库落库，落库后数据和缓存如何做到保证幂等（Redis   和数据库如何同时成功同时失败）？
    
*   如果不进行落库，都放在 Redis 中如何这是 Redis 和数据库的同步策略？还有放在缓存中就能百分之百的成功吗？
    

##### confirm 确认消息、Return 返回消息

理解 confirm 消息确认机制

*   消息的确认，指生产者收到投递消息后，如果 Broker 收到消息就会给我们  的生产者一个应答，生产者接受应答来确认 broker 是否收到消息。
    

###### 如何实现 confirm 确认消息。

*   在 Channel 上开启确认模式：channel.confirmSelect()
    
*   在 channel 上添加监听：addConfirmListener，监听成功和失败的结果，具体结果对消息进行重新发送或者记录日志。
    

##### return 消息机制

Return 消息机制处理一些不可路由的消息，我们的生产者通过指定一个 Exchange 和 Routinkey，把消息送达到某一个队列中去，然后我们消费者监听队列进行消费处理！

在某些情况下，如果我们在发送消息的时候当 Exchange 不存在或者指定的路由 key 路由找不到，这个时候如果我们需要监听这种不可到达的消息，就要使用 Return Listener！

Mandatory 设置为 true 则会监听器会接受到路由不可达的消息，然后处理。如果设置为 false，broker 将会自动删除该消息。

##### 消费端自定义监听

##### 消费端限流

> 什么是消费端的限流？限流算法[点击这里](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247484716&idx=1&sn=d2fc1055e6f7641aaa2419b4cb503a6a&chksm=eb53801adc24090c5af5383300b5df347e13bfeac06b605f51e4d5907c99b884d4de1b805694&scene=21#wechat_redirect)阅读。

假设我们有个场景，首先，我们有个 rabbitMQ 服务器上有上万条消息未消费，然后我们随便打开一个消费者客户端，会出现：巨量的消息瞬间推送过来，但是我们的消费端无法同时处理这么多数据。

这时就会导致你的服务崩溃。其他情况也会出现问题，比如你的生产者与消费者能力不匹配，在高并发的情况下生产端产生大量消息，消费端无法消费那么多消息。

*   rabbitMQ 提供了一种 qos（服务质量保证）的功能，即非自动确认消息的前提下，如果有一定数目的消息（通过 consumer 或者 Channel 设置 qos）未被确认，不进行新的消费。
    

> void basicQOS(unit prefetchSize,ushort prefetchCount,Boolean global) 方法。

*   prefetchSize:0 单条消息的大小限制。0 就是不限制，一般都是不限制。
    
*   prefetchCount: 设置一个固定的值，告诉 rabbitMQ 不要同时给一个消费者推送多余 N 个消息，即一旦有 N 个消息还没有 ack，则 consumer 将 block 掉，直到有消息 ack
    
*   global：truefalse 是否将上面的设置用于 channel，也是就是说上面设置的限制是用于 channel 级别的还是 consumer 的级别的。
    

##### 消费端 ack 与重回队列

*   消费端进行消费的时候，如果由于业务异常我们可以进行日志的记录，然后进行补偿！（也可以加上最大努力次数的尝试）
    
*   如果由于服务器宕机等严重问题，那我们就需要手动进行 ack 保证消费端的消费成功！
    

###### 消息重回队列

*   重回队列就是为了对没有处理成功的消息，把消息重新投递给 broker！
    
*   实际应用中一般都不开启重回队列。
    

##### TTL 队列 / 消息

> TTL time to live 生存时间。

*   支持消息的过期时间，在消息发送时可以指定。
    
*   支持队列过期时间，在消息入队列开始计算时间，只要超过了队列的超时时间配置，那么消息就会自动的清除。
    

##### 死信队列

> 死信队列：DLX，Dead-Letter-Exchange

利用 DLX，当消息在一个队列中变成死信（dead message，就是没有任何消费者消费）之后，他能被重新 publish 到另一个 Exchange，这个 Exchange 就是 DLX。

消息变为死信的几种情况：

1.  消息被拒绝（basic.reject/basic.nack）同时 requeue=false（不重回队列）
    
2.  TTL 过期
    
3.  队列达到最大长度
    

> DLX 也是一个正常的 Exchange，和一般的 Exchange 没有任何的区别，他能在任何的队列上被指定，实际上就是设置某个队列的属性。
> 
> 当这个队列出现死信的时候，RabbitMQ 就会自动将这条消息重新发布到 Exchange 上去，进而被路由到另一个队列。可以监听这个队列中的消息作相应的处理，这个特性可以弥补 rabbitMQ 以前支持的 immediate 参数的功能。

死信队列的设置

*   设置 Exchange 和 Queue，然后进行绑定
    

Exchange: dlx.exchange(自定义的名字)

queue: dlx.queue（自定义的名字）

routingkey: #（# 表示任何 routingkey 出现死信都会被路由过来）

然后正常的声明交换机、队列、绑定，只是我们在队列上加上一个参数：

arguments.put("x-dead-letter-exchange","dlx.exchange");

#### rabbitMQ 集群模式

1.  主备模式：实现 rabbitMQ 高可用集群，一般在并发量和数据不大的情况下，这种模式好用简单。又称 warren 模式。（区别于主从模式，主从模式主节点提供写操作，从节点提供读操作，主备模式从节点不提供任何读写操作，只做备份）如果主节点宕机备份从节点会自动切换成主节点，提供服务。
    
2.  集群模式：经典方式就是 Mirror 模式，保证 100% 数据不丢失，实现起来也是比较简单。
    

*   镜像队列，是 rabbitMQ 数据高可用的解决方案，主要是实现数据同步，一般来说是由 2-3 节点实现数据同步，（对于 100% 消息可靠性解决方案一般是 3 个节点）
    

![](https://mmbiz.qpic.cn/mmbiz_png/XaklVibwUKn7LrlfnxN5ibcIcEUFwn9pNPic8RRUIcmXZDo1c3n9EUutRlrj5rIj3s6ictOY7E4Bc8MUsLOmDJj64Q/640?wx_fmt=png)

多活模式：这种模式也是实现异地数据复制的主流模式，因为 shovel 模式配置相对复杂，所以一般来说实现异地集群都是使用这种双活，多活的模式，这种模式需要依赖 rabbitMQ 的 federation 插件，可以实现持续可靠的 AMQP 数据。  

rabbitMQ 部署架构采用双中心模式（多中心）在两套（或多套）数据中心个部署一套 rabbitMQ 集群，各中心的 rabbitMQ 服务需要为提供正常的消息业务外，中心之间还需要实现部分队列消息共享。

多活架构如下：

![](https://mmbiz.qpic.cn/mmbiz_png/XaklVibwUKn7LrlfnxN5ibcIcEUFwn9pNP7FLrGaEx9qUg0vR2UuRFVmQ7BtSzmicAHVrFFUNVpAOtzgQ1F3BMCLg/640?wx_fmt=png)

> federation 插件是一个不需要构建 Cluster，而在 Brokers 之间传输消息的高性能插件，federation 可以在 brokers 或者 cluster 之间传输消息，连接的双方可以使用不同的 users 或者 virtual host 双方也可以使用不同版本的 erlang 或者 rabbitMQ 版本。federation 插件可以使用 AMQP 协议作为通讯协议，可以接受不连续的传输。

![](https://mmbiz.qpic.cn/mmbiz_png/XaklVibwUKn7LrlfnxN5ibcIcEUFwn9pNPm9gWxMOB3MSQib6H87QWqu2s4mee8eEFbgVeDvWLHSAAfOvoCXKmaaA/640?wx_fmt=png)

Federation Exchanges, 可以看成 Downstream 从 Upstream 主动拉取消息，但    
并不是拉取所有消息，必须是在 Downstream 上已经明确定义 Bindings 关系的    
Exchange, 也就是有实际的物理 Queue 来接收消息，才会从 Upstream 拉取消息    
到 Downstream。  

使用 AMQP 协议实施代理间通信，Downstream 会将绑定关系组合在一起， 绑定 / 解除绑定命令将发送到 Upstream 交换机。

因此，Federation Exchange 只接收具有订阅的消息。

> HAProxy 是一款提供高可用性、负载均衡以及基于 TCP (第四层) 和 HTTP    
> (第七层) 应用的代理软件, 支持虚拟主机，它是免费、快速并且可靠的一种解决    
> 方案。
> 
> HAProxy 特别适用于那些负载特大的 web 站点，这些站点通常又需要会    
> 话保持或七层处理。HAProxy 运行在时下的硬件上，完全可以支持数以万计的    
> 并发连接。
> 
> 并且它的运行模式使得它可以很简单安全的整合进您当前的架构中    
> 同时可以保护你的 web 服务器不被暴露到网络上。

##### HAProxy 性能为何这么好？

1.  单进程、事件驱动模型显著降低了. 上下文切换的开销及内存占用.
    
2.  在任何可用的情况下，单缓冲 (single buffering) 机制能以不复制任何数据的方式完成读写操作，这会节约大量的 CPU 时钟周期及内存带宽
    
3.  借助于 Linux 2.6 (>= 2.6.27.19). 上的 splice() 系统调用，HAProxy 可以实现零复制转发 (Zero-copy forwarding), 在 Linux 3.5 及以上的 OS 中还可以实现心零复制启动 (zero-starting)
    
4.  内存分配器在固定大小的内存池中可实现即时内存分配，这能够显著减少创建一个会话的时长
    
5.  树型存储: 侧重于使用作者多年前开发的弹性二叉树，实现了以 O(log(N)) 的低开销来保持计时器命令、保持运行队列命令及管理轮询及最少连接队列
    

##### keepAlive

> KeepAlived 软件主要是通过 VRRP 协议实现高可用功能的。VRRP 是    
> Virtual Router RedundancyProtocol(虚拟路由器冗余协议) 的缩写,    
> VRRP 出现的目的就是为了解决静态路由单点故障问题的，它能够保证当    
> 个别节点宕机时，整个网络可以不间断地运行所以，Keepalived - - 方面    
> 具有配置管理 LVS 的功能，同时还具有对 LVS 下面节点进行健康检查的功    
> 能，另一方面也可实现系统网络服务的高可用功能

keepAlive 的作用

1.  管理 LVS 负载均衡软件
    
2.  实现 LVS 集群节点的健康检查中
    
3.  作为系统网络服务的高可用性 (failover)
    

##### Keepalived 如何实现高可用

Keepalived 高可用服务对之间的故障切换转移，是通过 VRRP (Virtual Router    
Redundancy Protocol , 虚拟路由器冗余协议) 来实现的。

在 Keepalived 服务正常工作时，主 Master 节点会不断地向备节点发送 (多播的方式) 心跳消息，用以告诉备 Backup 节点自己还活看，当主 Master 节点发生故障时，就无法发送心跳消息，备节点也就因此无法继续检测到来自主 Master 节点的心跳了，于是调用自身的接管程序，接管主 Master 节点的 IP 资源及服务。

而当主 Master 节点恢复时备 Backup 节点又会释放主节点故障时自身接管的 IP 资源及服务，恢复到原来的备用角色。