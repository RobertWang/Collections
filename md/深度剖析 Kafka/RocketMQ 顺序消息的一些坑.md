> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NzM0MjcyMQ==&mid=2650119955&idx=4&sn=d792422783692ef613572a698311df3a&chksm=beda447d89adcd6b10b317e6a749e0256cab336b16b88dad29ee80d60be0a983f62da476ec74&mpshare=1&scene=1&srcid=0615ITJu4fBnqlMAPCjGuBLd&sharer_sharetime=1623766923439&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

我觉得这个问题问得很频繁，而且非常经典，在这里我就以 Kafka 为例子，说说我对 Kafka 顺序消息的一些理解吧，如有理解不对的地方麻烦留言指点一下。

我们都知道无论是 Kafka 还是 RocketMQ，每个主题下面都有若干分区（RocketMQ 叫队列），如果消息被分配到不同的分区中，那么 Kafka 是不能保证消息的消费顺序的，因为每个分区都分配到一个消费者，此时无法保证消费者的消费先后，因此如果需要进行消息具有消费顺序性，可以在生产端指定这一类消息的 key，这类消息都用相同的 key 进行消息发送，kafka 就会根据 key 哈希取模选取其中一个分区进行存储，由于一个分区只能由一个消费者进行监听消费，因此这时候消息就具有消息消费的顺序性了。

**生产端**
-------

2、当 Broker 宕机出现问题，此时生产端有可能会把顺序消息发送到不同的分区，这时会发生短暂消息顺序不一致的现象。

针对以上两点，生产端必须保证单线程同步发送，这还好解决，针对第二点，想要做到严格的消息顺序，就要保证当集群出现故障后集群立马不可用，或者主题做成单分区，但这么做大大牺牲了集群的高可用，单分区也会另集群性能大大降低。

针对以上第二点，下面盘点一下 Kafka 集群中有哪些意外情况会打乱消息的顺序。

1、分区变更的情况

假设有集群中有两个分区的主题 A，生产端需要往分区 1 发送 3 条顺序消息，我们都知道生产端是根据消息 Key 取模计算决定消息发往哪个分区的，如果此时生产端发送第三条消息前，主题 A 增加了一个分区，生产端根据 Key 取模得出的分区号就不一样了，第三条消息路由到其它分区，结果就是这三条顺序消息就不在同一个分区了，此时就不能保证这三条消息的消费顺序了。

2、分区不变更

1.1、分区单副本

假设此时集群有两个分区的主题 A，副本因子为 1，生产端需要往分区 1 发送 3 条顺序消息，前两条消息已成功发送到分区 1，此时分区 1 所在的 broker 挂了（由于副本因子只有 1，因此会导致分区 1 不可用），当生产端发送第三条消息时发现分区 1 不可用，就会导致发送失败，然后尝试进行重试发送，如果此时分区 1 还未恢复可用，这时生产端会将消息路由到其它分区，导致了这三条消息不在同一个分区。

1.2、分区多副本

针对分区单副本情况，我们自然会想到将分区设置为多副本不就可以避免这种情况发生吗？多副本情况下，发送端同步发送，acks = all，即保证消息都同步到全部副本后，才返回发送成功，保证了所有副本都处在 ISR 列表中，如果此时其中一个 broker 宕机了，也不会导致分区不可用的情况，看起来确实避免了分区单副本分区不可用导致消息路由到其它分区的情况发生。

但我想说的是，还有一种极端的现象会发生，当某个 broker 宕机了，处在这个 broker 上的 leader 副本就不可用了，此时 controller 会进行该分区的 leader 选举，在选举过程中分区 leader 不可用，生产端会短暂报 no leader 警告，这时生产端也会出现消息被路由到其它分区的可能。

**消费端**
-------

当然，还有一个读者是这么问的：

![](https://mmbiz.qpic.cn/mmbiz_png/dkwuWwLoRKibpAXY94Z3F1nq3taSkIVNPicwNtrfD1ibcEHotWhQ3pzicNMv6A0ovibIjLQ0CULicE9yKgicScEFa9hRg/640?wx_fmt=png)

以下分析假设生产端已经将顺序消息成功发送到同一个分区。

### Kafka

kafka 的消费类 KafkaConsumer 是非线程安全的，因此用户无法在多线程中共享一个 KafkaConsumer 实例，且 KafkaConsumer 本身并没有实现多线程消费逻辑，如需多线程消费，还需要用户自行实现，在这里我会讲到 Kafka 两种多线程消费模型。

1、每个线程维护一个 KafkaConsumer

这样相当于一个进程内拥有多个消费者，也可以说消费组内成员是有多个线程内的 KafkaConsumer 组成的。

![](https://mmbiz.qpic.cn/mmbiz_png/dkwuWwLoRKibpAXY94Z3F1nq3taSkIVNP7H8bUick3KlIkhAfDGkYFXIKgFXWzZpOE3gKdiaPocicYiaJ7xx0agGP9A/640?wx_fmt=png)

但其实这个消费模型是存在很大问题的，从消费消费模型可看出每个 KafkaConsumer 会负责固定的分区，因此无法提升单个分区的消费能力，如果一个主题分区数量很多，只能通过增加 KafkaConsumer 实例提高消费能力，这样一来线程数量过多，导致项目 Socket 连接开销巨大，项目中一般不用该线程模型去消费。

2、单 KafkaConsumer 实例 + 多 worker 线程

针对第一个线程模型的缺点，我们可采取 KafkaConsumer 实例与消息消费逻辑解耦，把消息消费逻辑放入单独的线程中去处理，线程模型如下：

![](https://mmbiz.qpic.cn/mmbiz_png/dkwuWwLoRKibpAXY94Z3F1nq3taSkIVNPq6Guwk9KzicialffB2F9jJgqRKB30xlWEzWicuVhY7mM5C9omk9zHxkPw/640?wx_fmt=png)

从消费线程模型可看出，当 KafkaConsumer 实例与消息消费逻辑解耦后，我们不需要创建多个 KafkaConsumer 实例就可进行多线程消费，还可根据消费的负载情况动态调整 worker 线程，具有很强的独立扩展性，在公司内部使用的多线程消费模型就是用的单 KafkaConsumer 实例 + 多 worker 线程模型。

但这个消费模型由于消费逻辑是利用多线程进行消费的，因此并不能保证其消息的消费顺序，在这里我们可以引入阻塞队列的模型，一个 woker 线程对应一个阻塞队列，线程不断轮训从阻塞队列中获取消息进行消费，对具有相同 key 的消息进行取模，并放入相同的队列中，实现顺序消费， 消费模型如下：

![](https://mmbiz.qpic.cn/mmbiz_png/dkwuWwLoRKibpAXY94Z3F1nq3taSkIVNPiccfXzL06rzpvmhQadD655XTSoRic8kUCPVHUUgn8UZllCusCeuibWfrA/640?wx_fmt=png)

但是以上两个消费线程模型，存在一个问题：

在消费过程中，如果 Kafka 消费组发生重平衡，此时的分区被分配给其它消费组了，如果拉取回来的消息没有被消费，虽然 Kakfa 可以实现 ConsumerRebalanceListener 接口，在新一轮重平衡前主动提交消费偏移量，但这貌似解决不了未消费的消息被打乱顺序的可能性？

因此在消费前，还需要主动进行判断此分区是否被分配给其它消费者处理，并且还需要锁定该分区在消费当中不能被分配到其它消费者中（但 kafka 目前做不到这一点）。

参考 RocketMQ 的做法：

在消费前主动调用 ProcessQueue#isDropped 方法判断队列是否已过期，并且对该队列进行加锁处理（向 broker 端请求该队列加锁）。

### RocketMQ

RocketMQ 不像 Kafka 那么 “原生”，RocketMQ 早已为你准备好了你的需求，它本身的消费模型就是单 consumer 实例 + 多 worker 线程模型，有兴趣的小伙伴可以从以下方法观摩 RocketMQ 的消费逻辑：

```
org.apache.rocketmq.client.impl.consumer.PullMessageService#run
```

RocketMQ 会为每个队列分配一个 PullRequest，并将其放入 pullRequestQueue，PullMessageService 线程会不断轮询从 pullRequestQueue 中取出 PullRequest 去拉取消息，接着将拉取到的消息给到 ConsumeMessageService 处理，ConsumeMessageService 有两个子接口：

```
// 并发消息消费逻辑实现类
org.apache.rocketmq.client.impl.consumer.ConsumeMessageConcurrentlyService;
// 顺序消息消费逻辑实现类
org.apache.rocketmq.client.impl.consumer.ConsumeMessageOrderlyService;
```

其中，ConsumeMessageConcurrentlyService 内部有一个线程池，用于并发消费，同样地，如果需要顺序消费，那么 RocketMQ 提供了 ConsumeMessageOrderlyService 类进行顺序消息消费处理。

经过对 Kafka 消费线程模型的思考之后，从 ConsumeMessageOrderlyService 源码中能够看出 RocketMQ 能够实现局部消费顺序，我认为主要有以下两点：

1）RocketMQ 会为每个消息队列建一个对象锁，这样只要线程池中有该消息队列在处理，则需等待处理完才能进行下一次消费，保证在当前 Consumer 内，同一队列的消息进行串行消费。

2）向 Broker 端请求锁定当前顺序消费的队列，防止在消费过程中被分配给其它消费者处理从而打乱消费顺序。

**总结**
------

经过这篇文章的分析后，尝试回答读者的问题：

1、生产端：

1）生产端必须保证单线程同步发送，将顺序消息发送到同一个分区（当然如果发生了文中所描述的 Kafka 集群中意外情况，还是有可能会打乱消息的顺序，因此无论是 Kafka 还是 RocketMQ 都无法保证严格的顺序消息）；

2、消费端：

2）多分区的情况下：

如果想要保证 Kafka 在消费时要保证消费的顺序性，可以使用每个线程维护一个 KafkaConsumer 实例，并且是一条一条地去拉取消息并进行消费（防止重平衡时有可能打乱消费顺序）；对于能容忍消息短暂乱序的业务（话说回来， Kafka 集群也不能保证严格的消息顺序），可以使用单 KafkaConsumer 实例 + 多 worker 线程 + 一条线程对应一个阻塞队列消费线程模型。

3）单分区的情况下：

由于单分区不存在重平衡问题，以上两个线程模型的都可以保证消费的顺序性。

另外如果是 RocketMQ，使用 MessageListenerOrderly 监听消费可保证消息消费顺序。

很多人也有这个疑问：既然 Kafka 和 RocketMQ 都不能保证严格的顺序消息，那么顺序消费还有意义吗？

一般来说普通的的顺序消息能够满足大部分业务场景，如果业务能够容忍集群异常状态下消息短暂不一致的情况，则不需要严格的顺序消息。

本文来源：https://my.oschina.net/objcoding/blog/4270405