> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI5MjQ2MzM0Ng==&mid=2247494531&idx=1&sn=fe15294a397c67dc4fa13486a1ce0bd6&chksm=ec025c1edb75d508339bbc1f378416bc3e4ae95d033b0fdc9951537a099315f786611017b398&mpshare=1&scene=1&srcid=0219oMR0u0MzEvELROZx8Bp2&sharer_sharetime=1645254473641&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> 本文内容节选自 InfoQ：
> 
> https://www.infoq.cn/article/1UaxFKWUhUKTY1t_5gPq

1.  消息消费——如何发送和消费消息；
    
2.  消息确认（ack）——如何确认消息；
    
3.  消息保存——消息保留多长时间，触发消息删除的原因以及怎样删除；
    

消息消费模型
------

流式（Stream）模型

相比之下，流模型要求消息的消费严格排序或独占消息消费。对于一个管道，使用流式模型，始终只会有一个消费者使用和消费消息。消费者按照消息写入管道的确切顺序接收从管道发送的消息。

流模型通常与有状态应用程序相关联。有状态的应用程序更加关注消息的顺序及其状态。消息的消费顺序决定了有状态应用程序的状态。消息的顺序将影响应用程序处理逻辑的正确性。

在面向微服务或事件驱动的体系结构中，队列模型和流模型都是必需的。

### Pulsar 的消息消费模型

*   消费者被组合在一起以消费消息，每个消费组是一个订阅。
    
*   每个 Topic 可以有不同的消费组。
    
*   每组消费者都是对主题的一个订阅。
    
*   每组消费者可以拥有自己不同的消费方式：独占（Exclusive），故障切换（Failover）或共享（Shared）。
    

#### 独占订阅（Stream 流模型）

![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZubDbBye0zFfFLQo9ZYlls3BS916cmn2CGP6t6kxbxEria6eEqP2G2WsWxIaHDAbhgU2xiaIu9Lzx7icvsHRuzEGA/640?wx_fmt=png)

#### 故障切换（Stream 流模型）

使用故障切换订阅，多个消费者（Consumer）可以附加到同一订阅。但是，一个订阅中的所有消费者，只会有一个消费者被选为该订阅的主消费者。其他消费者将被指定为故障转移消费者。

当主消费者断开连接时，分区将被重新分配给其中一个故障转移消费者，而新分配的消费者将成为新的主消费者。发生这种情况时，所有未确认（ack）的消息都将传递给新的主消费者。这类似于 Apache Kafka 中的 Consumer partition rebalance。

下图是故障切换订阅的示例。消费者 B-0 和 B-1 通过订阅 B 订阅消费消息。B-0 是主消费者并接收所有消息。B-1 是故障转移消费者，如果消费者 B-0 出现故障，它将接管消费。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZubDbBye0zFfFLQo9ZYlls3BS916cmn26TErboiaR9dbpXIPSbHOSJehXtdXrFiaJwcCoFzIMBqlHywZomPqtMKw/640?wx_fmt=png)

#### 共享订阅（Queue 队列模型）

使用共享订阅，在同一个订阅背后，用户按照应用的需求挂载任意多的消费者。订阅中的所有消息以循环分发形式发送给订阅背后的多个消费者，并且一个消息仅传递给一个消费者。

当消费者断开连接时，所有传递给它但是未被确认（ack）的消息将被重新分配和组织，以便发送给该订阅上剩余的剩余消费者。

下图是共享订阅的示例。消费者 C-1，C-2 和 C-3 都在同一主题上消费消息。每个消费者接收大约所有消息的 1/3。

如果想提高消费的速度，用户不需要不增加分区数量，只需要在同一个订阅中添加更多的消费者。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZubDbBye0zFfFLQo9ZYlls3BS916cmn2YvKDEPGUsF6OR8mRECsrCTVticxznIiapAeic3WdVsSuv1WuJoI21ibjBg/640?wx_fmt=png)

#### 三种订阅模式的选择

独占和故障切换订阅，仅允许一个消费者来使用和消费每个对主题的订阅。这两种模式都按主题分区顺序使用消息。它们最适用于需要严格消息顺序的流（Stream）用例。

共享订阅允许每个主题分区有多个消费者。同一订阅中的每个消费者仅接收主题分区的一部分消息。共享订阅最适用于不需要保证消息顺序的队列（Queue）的使用模式，并且可以按照需要任意扩展消费者的数量。

Pulsar 中的订阅实际上与 Apache Kafka 中的 Consumer Group 的概念类似。创建订阅的操作很轻量化，而且具有高度可扩展性，用户可以根据应用的需要创建任意数量的订阅。

对同一主题的不同订阅，也可以采用不同的订阅类型。比如用户可以在同一主题上可以提供一个包含 3 个消费者的故障切换订阅，同时也提供一个包含 20 个消费者的共享订阅，并且可以在不改变分区数量的情况下，向共享订阅添加更多的消费者。

下图描绘了一个包含 3 个订阅 A，B 和 C 的主题，并说明了消息如何从生产者流向消费者。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZubDbBye0zFfFLQo9ZYlls3BS916cmn2bQickfAgEicEZOYFjBkGrfu7gZdEhMgsiaRO08eficQ9MOmW9S1z1kxA4Q/640?wx_fmt=png)

除了统一消息 API 之外，由于 Pulsar 主题分区实际上是存储在 Apache BookKeeper 中，它还提供了一个读取 API（Reader），类似于消费者 API（但 Reader 没有游标管理），以便用户完全控制如何使用 Topic 中的消息。

### Pulsar 的消息确认（ACK）

由于分布式系统的特性，当使用分布式消息系统时，可能会发生故障。比如在消费者从消息系统中的主题消费消息的过程中，消费消息的消费者和服务于主题分区的消息代理（Broker）都可能发生错误。消息确认（ACK）的目的就是保证当发生这样的故障后，消费者能够从上一次停止的地方恢复消费，保证既不会丢失消息，也不会重复处理已经确认（ACK）的消息。

在 Apache Kafka 中，恢复点通常称为 Offset，更新恢复点的过程称为消息确认或提交 Offset。

在 Apache Pulsar 中，每个订阅中都使用一个专门的数据结构–游标（Cursor）来跟踪订阅中的每条消息的确认（ACK）状态。每当消费者在主题分区上确认消息时，游标都会更新。更新游标可确保消费者不会再次收到消息。

Apache Pulsar 提供两种消息确认方法，单条确认（Individual Ack）和累积确认（Cumulative Ack）。通过累积确认，消费者只需要确认它收到的最后一条消息。主题分区中的所有消息（包括）提供消息 ID 将被标记为已确认，并且不会再次传递给消费者。累积确认与 Apache Kafka 中的 Offset 更新类似。

Apache Pulsar 可以支持消息的单条确认，也就是选择性确认。消费者可以单独确认一条消息。被确认后的消息将不会被重新传递。下图说明了单条确认和累积确认的差异（灰色框中的消息被确认并且不会被重新传递）。在图的上半部分，它显示了累计确认的一个例子，M12 之前的消息被标记为 acked。在图的下半部分，它显示了单独进行 acking 的示例。仅确认消息 M7 和 M12 - 在消费者失败的情况下，除了 M7 和 M12 之外，其他所有消息将被重新传送。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZubDbBye0zFfFLQo9ZYlls3BS916cmn2ovpQM0ZEfJzAwJ8xLYCkORETkiaY2F8F3LpAFVlZRXDhTdEAS0qjCCw/640?wx_fmt=png)

独占订阅或故障切换订阅的消费者能够对消息进行单条确认和累积确认；共享订阅的消费者只允许对消息进行单条确认。单条确认消息的能力为处理消费者故障提供了更好的体验。对于某些应用来说，处理一条消息可能需要很长时间或者非常昂贵，防止重新传送已经确认的消息非常重要。

这个管理 Ack 的专门的数据结构–游标（Cursor），由 Broker 来管理，利用 BookKeeper 的 Ledger 提供存储，在后面的文章中我们会介绍更多的关于游标（Cursor）的细节。

Apache Pulsar 提供了灵活的消息消费订阅类型和消息确认方法，通过简单的统一的 API，就可以支持各种消息和流的使用场景。

### Pulsar 的消息保留（Retention）

在消息被确认后，Pulsar 的 Broker 会更新对应的游标。当 Topic 里面中的一条消息，被所有的订阅都确认 ack 后，才能删除这条消息。Pulsar 还允许通过设置保留时间，将消息保留更长时间，即使所有订阅已经确认消费了它们。

下图说明了如何在有 2 个订阅的主题中保留消息。订阅 A 在 M6 和订阅 B 已经消耗了 M10 之前的所有消息之前已经消耗了所有消息。这意味着 M6 之前的所有消息（灰色框中）都可以安全删除。订阅 A 仍未使用 M6 和 M9 之间的消息，无法删除它们。如果主题配置了消息保留期，则消息 M0 到 M5 将在配置的时间段内保持不变，即使 A 和 B 已经确认消费了它们。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZubDbBye0zFfFLQo9ZYlls3BS916cmn29aHrmQdE03jibsfRMhkibJegshCrCe7HPqaHd1wpyXQZqKPiaeyZxicjPQ/640?wx_fmt=png)

在消息保留策略中，Pulsar 还支持消息生存时间（TTL）。如果消息未在配置的 TTL 时间段内被任何消费者使用，则消息将自动标记为已确认。消息保留期消息 TTL 之间的区别在于：消息保留期作用于标记为已确认并设置为已删除的消息，而 TTL 作用于未 ack 的消息。上面的图例中说明了 Pulsar 中的 TTL。例如，如果订阅 B 没有活动消费者，则在配置的 TTL 时间段过后，消息 M10 将自动标记为已确认，即使没有消费者实际读取该消息。

### Pulsar VS. Kafka

通过以上几个方面，我们对 Pulsar 和 Kafka 在消息模型方面的不同点进行一个总结。

#### 模型概念

Kafka：Producer - topic - consumer group - consumer；

Pulsar：Producer - topic - subscription - consumer。

#### 消费模式

Kafka： 主要集中在流（Stream）模式，对单个 partition 是独占消费，没有共享（Queue）的消费模式；

Pulsar：提供了统一的消息模型和 API。流（Stream）模式 – 独占和故障切换订阅方式；队列（Queue）模式 – 共享订阅的方式。

#### 消息确认（Ack）

Kafka：使用偏移 Offset；

Pulsar：使用专门的 Cursor 管理。累积确认和 Kafka 效果一样；提供单条或选择性确认。

#### 消息保留

Kafka：根据设置的保留期来删除消息。有可能消息没被消费，过期后被删除。不支持 TTL。

Pulsar：消息只有被所有订阅消费后才会删除，不会丢失数据。也允许设置保留期，保留被消费的数据。支持 TTL。

#### 对比总结

Apache Pulsar 将高性能的流（Apache Kafka 所追求的）和灵活的传统队列（RabbitMQ 所追求的）结合到一个统一的消息模型和 API 中。Pulsar 使用统一的 API 为用户提供一个支持流和队列的系统，且具有同样的高性能。 应用程序可以将此统一的 API 用于高性能队列和流式传输，而无需维护两套系统：RabbitMQ 进行队列处理，Kafka 进行流式处理。

本文内容节选自 InfoQ：https://www.infoq.cn/article/1UaxFKWUhUKTY1t_5gPq

**关于 StreamNative**  

StreamNative 是一家开源基础软件公司，由 Apache 软件基金会顶级项目 Apache Pulsar 创始团队组建而成，围绕 Pulsar 打造下一代云原生批流融合数据平台。StreamNative 作为 Apache Pulsar 商业化公司，专注于开源生态和社区构建，致力于前沿技术领域的创新，创始团队成员曾就职于 Yahoo、Twitter、Splunk、EMC 等知名大公司。

- EOF -