> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/MUhaZaWa_qgQs5V3xFySrA)

### 背景

目前笔者所在团队正常研发一款流程编排引擎，其中有多个功能特性需要 MQ 的延迟消息 / 消费者重试等特性。经过多方面的考量，我们最终决定采用计算存储分离的架构，在分布式 KV 存储的基础上，研发一款定制化的 MQ。目前，其具备了 MQ 的主要特性。本文所描述的是基于分布式 KV 的基础上研发 MQ 的核心思路。

### 术语

*   **「Message：」** 消息
    
*   **「Topic：」** 消息的逻辑分类
    
*   **「Partition：」** 分区，一个 Topic 中包含多个分区，每个消息最终发送到 Topic 的某个分区中
    
*   **「Partition Offset：」** 每当消息被发送到某个 Partition 中，这个 Partition 的 Offset+1
    
*   **「Producer：」** 生产者，发送消息到某个 Topic 下的某个分区
    
*   **「Consumer Group：」** 消费者组。一个 Topic 可以有多个不同的 Consumer Group 进行消费，每个 Consumer Group 可以消费到 Topic 中的全量消息。
    
*   **「Consumer：」** 消费者，消费 Topic 中的消息。如果一个 Consumer Group 下只有一个 Consumer，则其会消费到全部消息；如果有多个 Consumer，则每个 Consumer 只消费部分消息。
    
*   **「Consumer Group Offset：」** 记录消费者组，针对某个 Topic 下所有 Partition 当前消费到位置。针对每个 Partition，不会超过 Partition 最大 Offset。
    
*   **「Broker：」** 消息代理，Producer 将消息发送给 Broker，由 broker 发送到 topic 下某个分区。Consumer 连接 Broker，从 Broker 消费消息。
    
*   **「Broker Cluster：」** Broker 集群，为了实现高可用，每个 Broker 集群下包含多个 Broke 实例
    
*   **「Rebalance：」** 再均衡。传统的实现中，一个 Consumer Group 下的 Consumer 数量不能超过这个 Topic 下 Partition 的数量，一个 Partition 最多只能分配给一个消费者，超出 Partition 数量的消费者无法消费到消息。在本文实现中，不存在 Rebalance 概念。消费者数量，不受 Partition 限制。
    
*   **「Delay Message：」** 延迟消息。通常情况下，一个消息被投递到 Topic 中，就会被立即消费。延迟消息的意思是，延迟指定时间后，才可以被消费。
    
*   **「Retry Message：」** 重试消息。当一条消息消费失败后，需要被重试，即按照一定的重试策略，重新让消费者来消费这条消息。
    

### 整体架构

![](https://mmbiz.qpic.cn/mmbiz_png/fIGsxicVhVyaHB3m7EecHxZUicib645vQRqj9Qg5zzdvNkb5uTW5Qsm5WpzbB9nW1MYta760gluBXcGv0QsQtxXpw/640?wx_fmt=png)说明：

**「KV」** **「存储」**

本文以 KV Storage 是 Redis Cluster 为例进行讲解，Broker Cluster 中的所有 Broker 实例共享这个 Redis Cluster，其他任意支持按 key 排序 scan 的 KV 存储均可。

从持久化的角度来说，使用内存模式 Redis 并不合适，这里意图在于说明基于分布式 KV 实现的 MQ 的核心原理。事实上，在公司内部，我们使用的是基于 RocksDB 基础上研发的分布式 KV，在网络通信上兼容 redis 协议。

为了简化，本文中不讨论 Redis Cluster 扩 / 缩容, Slot 迁移的情况。但足以掌握基于分布式 KV 研发一款消息中间件的核心原理。

**「网络通信」**

Producer 和 Consumer 与 Broker 的通信，在笔者的项目中，使用的是 Grpc。在开源的通信框架中，Grpc 可以说是最流行的方案，Apacha RocketMQ 5.x 版本也采用了 Grpc。在本文中，并不会对 Grpc 进行介绍。

### 详细设计

#### Broker 集群元数据

每个 Broker 启动时，可以将自身信息注册到 Redis 中，以便 producer/consumer 进行服务发现。例如通过 hash 结构维护：

Key

```
[cluster]$cluster_name
```

Value

```
filed         value

broker1       ip:port

broker2       ip:port
```

#### Topic 元数据

Topic 元数据主要是维护 Topic 下有多少 Partition，这些 Partition 在 Redis Cluster 中是如何分布的。用户在创建 Topic 时，指定分区数量。

Redis Cluster 有 16384 个槽，每个 Redis 分片负责其中部分槽。当创建一个 Topic 时，例如指定 10 个分区，可以按照一定策略把这个 10 个分区映射到不同的槽上，相当于间接的把分区分配到了不同的 redis 分片上。

当创建好一个 Topic 之后，将 Topic 下的分区分配给不同的 Broker。例如 10 个分区，10 个 Broker，则每个 Broker 负责一个分区。如果只有 5 个分区，那么需要分配给其中 5 个 broker。

例如通过 hash 结构维护维护这个映射关系

key

```
[topic_metadata]$topic_name
```

value

```
filed         value

partition1    broker1

partition2    broker2
```

#### 消息

消息使用 protobuf 进行定义：

```
message Message{

  google.protobuf.Struct metadata = 1;  //消息的元数据

  string partition = 2;  //消息所属的分区

  int64 offset = 3;      //消息的offset

  string msgId = 4;      //消息的唯一id

  string topic = 5;      //消息的topic

  string key = 6;        //消息key，用于路由

  bytes body = 7;        //消息体

  google.protobuf.Timestamp born_time = 8;  //消息生成时间 

  google.protobuf.Timestamp expireTime = 9; //消息截止时间，用于延迟消息

}
```

生产者在发送消息时，最简单的情况下，只需要指定消息的 topic、body。当有其他特殊需求时，可以指定以下字段：

*   key：具有相同 key 的消息，经过 hash 算法，写入到相同的分区。
    
*   partition：直接指定分区，不根据 key 计算。
    
*   expireTime：延迟消息。消息不希望被立即消费，而是到指定时间后，才会被消费。
    

#### 消息发送

**「从 Producer 的角度来说：」**

*   **「重试：」** 发送一条消息到 broker 可能失败，所以需要重试。重试需要设置一定的次数和超时时间，在超时时间内进行重试。
    
*   **「分区选择：」** 选择分区应该在 producer 端确定，确定分区后，消息发送到分区所属的 broker。
    
*   **「聚合：」** 为了减少网络 io，应该聚合批次进行发送，注意聚合是按照分区进行聚合
    
*   **「broker 选择：」** 对于无序消息，选择 broker 可以有一定的策略，例如某个 broker 失败率比较高，或者延迟比较高，则应该优先选择其他的 broker。
    

**「从 broker 角度来说：」**

接收到一条消息时，offset 信息维护。每次发送，在确定消息需要发送到的分区后，broker 需要将对应 partition 的 offset+1。在笔者的项目中，使用了 hash 结构存储每个分区的最大的 offset：

key：

```
[topic_offset]{$topic}
```

value

```
field             value

partition 1       offset1

partition 2       offset2
```

为了提升 offset 维护的效率，不需要每次都调用 HINCRBY，而是在 broker 启动时，将自己维护的分区 offset 信息加载到内存中，之后发送消息时，内存中增加，定期保存到 KV 中。

此外，需要有一个修正 offset 的逻辑，避免 broker 异常宕机的情况下，offset 没有成功保存到 redis 中。在 broker 启动时，可以从当前维护的最大 offset 开始往后扫描，如果发现了新消息，则说明 offset 需要修正 (参考如下消息存储部分)。

#### 消息存储

当消息被写入到 redis 中，key 满足以下格式：

```
[topic]{$topic_$partition}$offset
```

其中：

*   [topic]：是固定前缀
    
*   {$topic_$partition}：topic 是 topic 名称，partition 表示分区。这里利用了 redis hash tag 能力。
    
*   $offset：表示当前这个分区的 offset
    

#### 消息拉取

拉取通过 redis scan 操作进行，将 scan 到的消息交由消费者处理。

在拉取消息时，依赖于一个 consumer offset，其维护了某个 consumer group 消费某个 topic 的进度信息。拉取时，从这个位置开始。这里可以考虑使用 hash 数据结构：

key：

```
[consumer_offset]$group_$topic
```

value

```
field             value

partition 1       offset1

partition 2       offset2
```

当有消费者连接上某个 broker 时，broker 查询到自己负责的分区的 parititon offset，从这个位置开始拉取消息。

#### 延迟消息

对于所有延迟消息，会首先发送到一个特殊的 delay topic 中，相当于暂存这个消息。消息到期后，投递到目标 topic 中。

*   在发送到延迟 topic 之前，会记录消息的原始 topic、partition 到 metadata 的`origin_topic` **「、」** `origin_partition`字段中。之后发送到 delay topic 中。key 格式与普通消息不同，以时间戳排序：
    

```
[delay]$broker_id}$expireTime
```

*   会有一个延迟消息转发器，不断的扫描 abase，当发现有消息到期时，修改延迟消息中的目标 topic 为`origin_topic`、`origin_partition`字段，之后从发送逻辑，投递到目标 topic 中。
    
*   同时，会记录当前扫描到的位置。
    

#### 消费重试

当消费者消费一条消息失败时，默认也是会走延迟消息逻辑，到期后，投递给目标消费者，重新消费。重试消息，也是基于延迟消息的基础上开发的。

在逻辑上不同的是：

*   延迟消息直接投递给了目标 topic
    
*   而重试消息不能投递给你目标 topic，因为一个 topic 有多个 consumer group，如果只是某个 consumer group 失败需要重试，那么其他 consumer group 应该不受影响。因此每个 consumer_group，应该有独立的重试 topic。如：
    

```
[topic]retry_$consumer_group
```

这个 Topic 下应该也有包含分区，策略与之前所述 Topic 元数据维护相同。

#### 死信队列

当消息重试到最大重试次数后，依然失败，可以放入死信队列。如：

```
[topic]dead_$consumer_group
```

#### 消息 TTL

为了避免已经被消费的消息，占用大量的存储空间，消息会被清理。我们的策略是：

*   对于普通消息：3 天后会自动清理，意味着一条消息 3 天内没被消费，将会被删除。
    
*   对于延迟消息：在截止时间的基础上 + 3 天。
    

### 总结

本文的目的是介绍如何基于分布式 KV 研发一款 MQ 的核心思路，很多容灾 / 高可用 / 性能优化等方面的主题并没有讨论。仅仅是提供一种核心思路，如果希望在生产环境使用，需要进行大量的改进与优化。

点击**阅读原文**，了解更多技术干货~