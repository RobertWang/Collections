> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU0OTE4MzYzMw==&mid=2247518728&idx=5&sn=108a2f81e57e9070f06218d4485ba1fc&chksm=fbb103f6ccc68ae0a8b67a8d32bb123ffbab3ce12f55e7889458374254dace1c5ac1239202ea&mpshare=1&scene=1&srcid=0819WZnx43FLf7BmVNEgYm7f&sharer_sharetime=1629345314025&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

为了系统间解耦，我们通常会引入`MQ`框架，大家各司其职共同完成上下游的业务流程。

![](https://mmbiz.qpic.cn/mmbiz_jpg/2KTof9YshweLJVZ3yKhblYxiaXUBL3CMYwSdHxu9LiadrVdUVT3Lic3HVToiavEGbdEr0qEZfwfiab1qmFiaGd4xR8tA/640?wx_fmt=jpeg)

**大致过程：**

*   生产端，创建一条消息，通过网络发送到 MQ Server
    
*   MQ 将 消息存储在 topic 的**一个分区**里
    
*   消费端，从分区中拉取消息，消费处理
    

**但现实往往不一样！MQ 架构设计要满足高并发、高性能、高可用等指标**

![](https://mmbiz.qpic.cn/mmbiz_jpg/2KTof9YshweLJVZ3yKhblYxiaXUBL3CMYMBwt2sBKy4a5aoXvHAJ5B0ZiaiaGOtxZIkhAib7WB8NoNclGjX56eic9fg/640?wx_fmt=jpeg)

单分区，达不到我们的吞吐量要求，我们考虑采用`多分区`架构设计，正所谓 ” 三个臭皮匠赛过一个诸葛亮 “，多分区可以有效分摊全局压力，提升整体系统性能。

![](https://mmbiz.qpic.cn/mmbiz_jpg/2KTof9YshweLJVZ3yKhblYxiaXUBL3CMYOBHyObVr0FZ3zcZqZVbusiblH1azsU0SIt5IIkep8nHq4CVkaicaSFCA/640?wx_fmt=jpeg)

两台 MQ 机器，组成一个集群，原先一个分区存储`6条消息`，现在分摊到两个分区，每个分区各存储`3条消息`，性能比上面那个提升一倍。

貌似可以满足我们的需求，但任何事情都有两面性！

**我们看看下面业务场景：**

一个用户在电商网站上下订单到交易完成，中间会经历一系列动作，订单的状态也会随之变化，一个订单会产生多条 MQ 消息，`下单`、`付款`、`发货`、`买家确认收货`，消费端需要严格按照业务状态机的顺序处理，否则，就会出现业务问题。

我们发现，消息带上了状态，不再是一个个独立的个体，有了上下文依赖关系！

对于这个问题，突然想到`HTTP协议`，其本身也是无状态的，也就是说前后两次请求没有关联，但有些业务功能有登录要求，那怎么解决？

> 引入 Cookie 机制，每次请求客户端额外传输一些数据，来达到上下文关联。

**回到 MQ 的消息顺序问题，我们要如何解决？**

![](https://mmbiz.qpic.cn/mmbiz_jpg/2KTof9YshweLJVZ3yKhblYxiaXUBL3CMYQZ2OGJWDBPX9OOictn1wveZv7d9vxGK4dKMFc1DKQ60ePhS3q8buyWw/640?wx_fmt=jpeg)

答案：各退一步，保证局部有序。

> 比如上面的电商例子，只要保证一个订单的多条状态消息在同一个分区，便可以满足业务需求，这个方案可以覆盖大部分的业务场景。

这里面只需要有一个`路由策略`组件，由它决定消息该放到哪个分区中！

考虑到市面 MQ 开源框架很多，常见的如：Kafka、Pulsar、RabbitMQ、RocketMQ 等，API 方法略有区别，但设计思路是相通的。

接下来，我们以 `RocketMQ` 为例：

**生产端提供了一个接口** **`MessageQueueSelector`**

```
public interface MessageQueueSelector {
   MessageQueue select(final List<MessageQueue> mqs, final Message msg, final Object arg);
}
```

接口内定义一个 select 方法，具体参数含义：

*   mqs：该 Topic 下所有的队列分片
    
*   msg：待发送的消息
    
*   arg：发送消息时传递的参数
    

关于`MessageQueueSelector`接口，RocketMQ 框架提供了三个默认实现类：

*   1、SelectMessageQueueByHash：
    

> arg 参数的 hashcode 的绝对值，然后对 mqs.size() 取余，得到目标队列在 mqs 的下标

*   2、SelectMessageQueueByRandom：
    

> 对 mqs.size() 值取随机数作为目标队列在 mqs 的下标

*   3、SelectMessageQueueByMachineRoom
    

> 返回 null

**特别注意：**

虽然保证了单个分片的消息有序，但每个分片的消费者只能是单线程处理，因为多线程无法控制消费顺序。这个可能会损失一些性能。

**这里又引出另一个问题，如何保证一个队列只能有一个消费端呢？**

1、

org.apache.rocketmq.client.impl.consumer.RebalanceImpl#updateProcessQueueTableInRebalance

![](https://mmbiz.qpic.cn/mmbiz_jpg/2KTof9YshweLJVZ3yKhblYxiaXUBL3CMYl2pt0ywfYicNtMhvibW69wZttFKtGPWr8WwIKRMU34sefEibh2iapxlLCA/640?wx_fmt=jpeg)

*   遍历一个 topic 下所有的`MessageQueue`
    
*   `isOrder && !this.lock(mq)` 尝试对它加锁，确保一个`MessageQueue`只能被一个消费者处理
    

2、将`PullRequest`对象放入`PullMessageService`的`pullRequestQueue`队列中

```
public void dispatchPullRequest(List<PullRequest> pullRequestList) {
    for (PullRequest pullRequest : pullRequestList) {
        this.defaultMQPushConsumerImpl.executePullRequestImmediately(pullRequest);
        log.info("doRebalance, {}, add a new pull request {}", consumerGroup, pullRequest);
    }
}
```

3、org.apache.rocketmq.client.impl.consumer.PullMessageService#run

![](https://mmbiz.qpic.cn/mmbiz_jpg/2KTof9YshweLJVZ3yKhblYxiaXUBL3CMYNtXfYvjXiaA3pklFgicmE4opBJLcHpXFX2icosQRmaJaibIUZCJXX54OMA/640?wx_fmt=jpeg)

*   `PullMessageService` 是一个`Runnable`线程任务
    
*   无限循环，从队列中拉取、处理消息
    

**另一个问题，如何保证一个队列，只有一个线程在处理消息呢？**

1、 DefaultMQPushConsumerImpl#pullMessage

![](https://mmbiz.qpic.cn/mmbiz_jpg/2KTof9YshweLJVZ3yKhblYxiaXUBL3CMYU9ZoJd7icmutNLRvKn4hpjrmQappjyVuJw6Wp7uuME61Ex5RkdQvCXg/640?wx_fmt=jpeg)

*   `ConsumeMessageService` 中有两个实现类，因为我们有消费顺序要求，会选择`ConsumeMessageOrderlyService`来处理业务
    

2、 ConsumeMessageOrderlyService.ConsumeRequest

![](https://mmbiz.qpic.cn/mmbiz_jpg/2KTof9YshweLJVZ3yKhblYxiaXUBL3CMYibywA90B6mrYy7tQMhou6Pz1ZUiaQOyzf2PerklFzU8oyxy09HYvRX1g/640?wx_fmt=jpeg)

  

*   从`ConcurrentMap`中获取`messageQueue`对应的锁对象
    
*   通过 `synchronized` 关键字，线程来抢占锁，互斥关系，从而保证了一个`MessageQueue`只能有一个线程并发处理
    

**继续往下看，如果扩容了怎么办？**

原来有 6 个分区，`order_id_1`的消息在`MessageQueue6` 中，此时扩容一倍，现在 12 个分区，`order_id_1`订单后面产生的消息可能路由到了`MessageQueue8` 中，同一个订单的消息分布在两个分区中，无法保证顺序。

我们能做的是，先将存量消息处理完，再扩容。如果是在线业务，可以搞个临时 topic，先将消息暂时堆积，待扩容后，按新的路由规则重新发送。

**顺序消息，如果某条失败了怎么办？会不会一直阻塞？**

1、如果失败，不会提交消费位移，系统会自动重试（有重试上限），此时会阻塞后面的消息消费，直到这条消息处理完

2、如果这个消息达到重试上限，依然失败，会进入`死信队列`，可以继续处理后面的消息