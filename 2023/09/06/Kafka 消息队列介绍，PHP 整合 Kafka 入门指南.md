> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/q013Xn9LAcjPGXU3VmHSoQ)

> Kafka 消息队列介绍，PHP 整合 Kafka 入门指南什么是消息队列消息（Message）是指在应用之间传送的

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQejTaqxeT74vD5TDzlbXXe41fXobq0G4JBINZroGAWOnuFMibkYWq9Loc3eauia1soXfDsDXQAeP8Gw/640?wx_fmt=png) Kafka 消息队列介绍，PHP 整合 Kafka 入门指南
===========================================================================================================================================================================

什么是消息队列
=======

消息（Message）是指在应用之间传送的数据，消息可以非常简单，比如只包含文本字符串，也可以更复杂，可能包含嵌入对象。消息队列（Message Queue）是一种应用间的通信方式，消息发送后可以立即返回，有消息系统来确保信息的可靠专递，**消息发布者只管把消息发布到 MQ 中而不管谁来取，消息使用者只管从 MQ 中取消息而不管谁发布的**，这样发布者和使用者都不用知道对方的存在。

消息队列的应用场景
=========

消息队列主要有以下应用场景：

*   • **异步处理：**多应用对消息队列中同一消息进行处理，应用间并发处理消息，相比串行处理，可以减少处理时间，如短信通知、文件下载、订单推送等。
    
*   • **应用耦合**：多应用间通过消息队列对同一消息进行处理，**避免调用接口失败导致整个过程失败，**如人脸识别，文字翻译等不能做到业务系统里面，业务系统负责生产消息，人脸识别系统消费消息
    
*   • **限流削峰：**广泛应用于秒杀或抢购活动中，使用消息队列避免流量过大导致应用系统挂掉的情况，1. 网关在接受到请求后，就把请求放入到消息队列里面 2. 后端的服务从消息队列里面获取到请求，完成后续的秒杀处理流程。然后再给用户返回结果。优点：控制了流量 缺点：会让流程变慢
    
*   • **消息驱动的系统：**系统分为消息队列、消息生产者、消息消费者，生产者负责产生消息，消费者 (可能有多个) 负责对消息进行处理；如大量的物联网系统中，平台和硬件端的通讯需要消息队列
    
*   • **日志聚合：**- 可以使用 Kafka 收集来自不同系统的日志并将其存储在集中式系统中以供进一步处理。
    

什么是发布 / 订阅模式
============

发布 / 订阅模式下包括三个角色：

*   • 角色主题（Topic）
    
*   • 发布者 (Publisher)
    
*   • 订阅者 (Subscriber)
    

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQejTaqxeT74vD5TDzlbXXe4twfibSafoamyxrOicTozbp54QV0SKett7zS5rqGpdXgQZ2IqlOxvViakw/640?wx_fmt=png) 发布 / 订阅模式

  
发布者将消息发送到 Topic，系统将这些消息传递给多个订阅者。  
发布 / 订阅模式特点：

*   • 每个消息可以有多个订阅者；
    
*   • 发布者和订阅者之间有时间上的依赖性。针对某个主题（Topic）的订阅者，它必须创建一个订阅者之后，才能消费发布者的消息。
    
*   • 为了消费消息，订阅者需要提前订阅该角色主题，并保持在线运行；
    

常用的消息队列工具有哪些
============

1) RabbitMQ
-----------

RabbitMQ 2007 年发布，是一个在 AMQP(高级消息队列协议) 基础上完成的，可复用的企业消息系统，是当前最主流的消息中间件之一。

2) RocketMQ
-----------

RocketMQ 出自 阿里公司的开源产品，用 Java 语言实现，在设计时参考了 Kafka，并做出了自己的一些改进，消息可靠性上比 Kafka 更好。RocketMQ 在阿里集团被广泛应用在订单，交易，充值，流计算，消息推送，日志流式处理等。

3) ActiveMQ
-----------

ActiveMQ 是由 Apache 出品，ActiveMQ 是一个完全支持 JMS1.1 和 J2EE 1.4 规范的 JMS Provider 实现。它非常快速，支持多种语言的客户端和协议，而且可以非常容易的嵌入到企业的应用环境中，并有许多高级功能。

4) Pulsar
---------

Apahce Pulasr 是一个企业级的发布 - 订阅消息系统，最初是由雅虎开发，是下一代云原生分布式消息流平台，集消息、存储、轻量化函数式计算为一体，采用计算与存储分离架构设计，支持多租户、持久化存储、多机房跨区域数据复制，具有强一致性、高吞吐、低延时及高可扩展性等流数据存储特性。  
Pulsar 非常灵活：它既可以应用于像 Kafka 这样的分布式日志应用场景，也可以应用于像 RabbitMQ 这样的纯消息传递系统场景。它支持多种类型的订阅、多种交付保证、保留策略以及处理模式演变的方法，以及其他诸多特性。

5) Kafka
--------

Apache Kafka 是一个分布式消息发布订阅系统。它最初由 LinkedIn 公司基于独特的设计实现为一个分布式的提交日志系统 (a distributed commit log)，，之后成为 Apache 项目的一部分。Kafka 系统快速、可扩展并且可持久化。它的分区特性，可复制和可容错都是其不错的特性。

Kafka 工作原理
==========

Kafka 是一个分布式系统，**由通过高性能 TCP 网络协议进行通信的服务器端和客户端组成**。它可以部署在企业内部和云环境中的裸机硬件、虚拟机和容器上。并通过**消息**进行通讯

**服务器**：Kafka 以一个或多个服务器集群的形式运行，可以跨越多个数据中心或云区域。其中一些服务器构成存储层，称为 "经纪人"。其他服务器运行 Kafka Connect，以事件流的形式持续导入和导出数据，从而将 Kafka 与关系数据库等现有系统以及其他 Kafka 集群集成。为了让您实现关键任务用例，Kafka 集群具有高度可扩展性和容错性：如果其中任何一台服务器出现故障，其他服务器将接替它们的工作，以确保持续运行而不会丢失任何数据。

**客户端**：它们允许我们编写分布式应用程序和微服务，并行、大规模地读取、写入和处理事件流，即使在网络问题或机器故障的情况下也能容错。Kafka 随附了一些这样的客户端，并由 Kafka 社区提供的几十种客户端对其进行了扩充：有 Java 和 Scala 客户端（包括更高级的 Kafka Streams 库）、Go、Python、**PHP**、C/C++ 和许多其他编程语言的客户端以及 REST API  
**事件：**当我们向 Kafka 读取或写入数据时，是以事件的形式执行此操作。从概念上讲，事件具有键、值、时间戳和可选的元数据标头。一个消息示例如下：  
Event key: "Alice"，  
Event value: "Made a payment of $200 to Bob"，  
Event timestamp: "Jun. 25, 2020 at 2:06 p.m."  
**生产者和消费者：**生产者是将事件发布（写入）到 Kafka 的客户端应用程序，而消费者是订阅（读取和处理）这些事件的客户端应用程序。在 Kafka 中，生产者和消费者彼此完全解耦且互不可知，这是实现 Kafka 闻名的高可扩展性的关键设计元素。例如，生产者永远不需要等待消费者。 Kafka 提供了各种保证，例如一次性处理事件的能力。  
**主题（topic）：**事件被组织并持久存储在**主题**中。可以这样理解，主题类似于文件系统中的文件夹，事件是该文件夹中的文件。示例主题名称可以是 “付款”。 Kafka 中的主题始终是多生产者和多订阅者：一个主题可以有零个、一个或多个向其写入事件的生产者，以及零个、一个或多个订阅这些事件的消费者。主题中的事件可以根据需要随时读取——与传统消息传递系统不同，事件在使用后不会被删除。相反，您可以通过每个主题的配置设置来定义 Kafka 应保留事件的时间，之后旧事件将被丢弃。 Kafka 的性能在数据大小方面实际上是恒定的，因此长时间存储数据是完全可以的。  
**主题分区：**一个主题分布在位于不同 Kafka 代理上的多个 buckets 中。这种数据的分布式放置对于可扩展性非常重要，因为它允许客户端应用程序**同时从多个代理读取数据或向多个代理写入数据**。一个主题可以有多个分区，当新事件发布到主题时，它实际上会附加到主题的分区之一。具有相同事件键（例如，客户或车辆 ID）的事件被写入同一分区，并且 Kafka 保证给定主题分区的任何消费者将始终按照与写入的顺序完全相同的顺序读取该分区的事件（先进先出）。  

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQejTaqxeT74vD5TDzlbXXe4eVALOPeO1eVyeCgKGQnbcucwamIuofz1wKxE8DVfy78hfFBzFibibiaFw/640?wx_fmt=png)主题分区

  
如上图，此示例主题有四个分区 P1–P4。两个不同的生产者客户端通过网络将事件写入主题的分区，相互独立地向主题发布新事件。具有相同键（在图中用颜色表示）的事件将写入同一分区。请注意，如果合适，两个生产者都可以写入同一分区。

安装 Kafka
========

首先安装 jdk

```
yum search java | grep jdk
yum install -y java-11-openjdk*

```

下载 Kafka

```
wget https://dlcdn.apache.org/kafka/3.5.0/kafka_2.13-3.5.0.tgz
tar -xzf kafka_2.13-3.5.0.tgz
cd kafka_2.13-3.5.0

```

启动，进入目录后，打开两个 terminal, 分别执行如下

```
bin/zookeeper-server-start.sh config/zookeeper.properties

```

```
bin/kafka-server-start.sh config/server.properties

```

这样就完成了 kafka 的启动了

使用 Terminal 操作 Kafka
====================

首先要现在下载 Kafka，下载后进入 Kafka  
创建一个主题

```
bin/kafka-topics.sh --create --topic ld-events --bootstrap-server 服务器Ip:9092
Created topic ld-events.

```

生成者将事件写入主题

```
bin/kafka-console-producer.sh --topic ld-events --bootstrap-server 服务器Ip:9092
//回车后
>this is my first event
>this is the second event

```

消费者读取事件

```
bin/kafka-console-consumer.sh --topic ld-events --from-beginning --bootstrap-server bin/kafka-console-producer.sh --topic ld-events --bootstrap-server 服务器Ip:9092

```

效果如下  

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQejTaqxeT74vD5TDzlbXXe4bKVialhR3HJkWWiakyeIDRt8p5H0cWXJNK4Da3uiaJTbo6Oicq0q635IicA/640?wx_fmt=png)image.png

PHP 对接 Kafka
============

PHP 要对接 Kafka，需要先安装 rdkafka php extension[1]，  
首先安装预构建包

```
//for ubuntu
apt install librdkafka-dev
//for contos
yum install librdkafka-devel

```

安装 rdkafka 扩展  
直接通过 pecl 安装

```
sudo pecl install rdkafka 

```

然后配置 php.ini

```
extension=rdkafka.so

```

检查 rdkafka 是否安装

```
//cli环境下运行：
php -m | grep rdkafka
//php-fpm环境下就运行phpinfo，然后网页上查看模块

```

编写生产者端
======

我们首先需要创建一个生产者，并向其添加代理

```
$conf = new RdKafka\Conf();
$conf->set('log_level', (string) LOG_DEBUG);
$conf->set('debug', 'all');
$rk = new RdKafka\Producer($conf);
$rk->addBrokers("127.0.0.1:9092");

```

创建主题并生成消息

```
$topic = $rk->newTopic("test");
$topic->produce(RD_KAFKA_PARTITION_UA, 0, "我是生产者1");
$topic->produce(RD_KAFKA_PARTITION_UA, 0, "我是生产者2");
$topic->produce(RD_KAFKA_PARTITION_UA, 0, "我是生产者3");
$topic->produce(RD_KAFKA_PARTITION_UA, 0, "我是生产者4");

```

> 这里我只是用简单文本消息而已，实际更多的是用 json

编写消费者端逻辑
========

我们首先需要创建一个的消费者，因 Low-level consumer 即将弃用，官方建议直接 High-level consumer[2]，代码如下

```
<?php

$conf = new RdKafka\Conf();

// Configure the group.id. All consumer with the same group.id will consume
// 设置分组
$conf->set('group.id', 'ldGroup');

// 设置代理
$conf->set('metadata.broker.list', '127.0.0.1');

// Set where to start consuming messages when there is no initial offset in
// offset store or the desired offset is out of range.
// 'earliest': start from the beginning
$conf->set('auto.offset.reset', 'earliest');

// Emit EOF event when reaching the end of a partition
$conf->set('enable.partition.eof', 'true');

$consumer = new RdKafka\KafkaConsumer($conf);

// 订阅主题
$consumer->subscribe(['test']);

echo "等待消息中。。。\n";
//业务逻辑
while (true) {
    $message = $consumer->consume(120*1000);
    switch ($message->err) {
        case RD_KAFKA_RESP_ERR_NO_ERROR:
            echo '收到消息：'.$message->payload."\n";
            break;
        case RD_KAFKA_RESP_ERR__PARTITION_EOF:
            echo "No more messages; will wait for more\n";
            break;
        case RD_KAFKA_RESP_ERR__TIMED_OUT:
            echo "Timed out\n";
            break;
        default:
            throw new \Exception($message->errstr(), $message->err);
            break;
    }
}


```

测试一下，我们运行消费者代码 php consumer.php，然后打开新的 Terminal，再运行 php producer.php  
最终效果如下：  

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQejTaqxeT74vD5TDzlbXXe4A3QstKpewXqticoUM2qlV6UXT1CNdjbmZ22JgZVAFun2ITwPT1qJFyg/640?wx_fmt=png)image.png

在 Laravel 中使用 Kafka
===================

如果我们的开发框架是 Laravel，我们可以直接使用 Laravel-kafka[3]，当然也这个也要求要先配置 rdkafka。  
通过 composer 安装

```
composer require mateusjunges/laravel-kafka

```

发布配置

```
php artisan vendor:publish --tag=laravel-kafka-config

```

默认配置如下

```
<?php

return [
      //kafka地址
    'brokers' => env('KAFKA_BROKERS', 'localhost:9092'),

    /*
     | Kafka consumers belonging to the same consumer group share a group id.
     | The consumers in a group then divides the topic partitions as fairly amongst themselves as possible by
     | establishing that each partition is only consumed by a single consumer from the group.
     | This config defines the consumer group id you want to use for your project.
     */
    'consumer_group_id' => env('KAFKA_CONSUMER_GROUP_ID', 'group'),

    /*
     | After the consumer receives its assignment from the coordinator,
     | it must determine the initial position for each assigned partition.
     | When the group is first created, before any messages have been consumed, the position is set according to a configurable
     | offset reset policy (auto.offset.reset). Typically, consumption starts either at the earliest offset or the latest offset.
     | You can choose between "latest", "earliest" or "none".
     */
    'offset_reset' => env('KAFKA_OFFSET_RESET', 'latest'),

    /*
     | If you set enable.auto.commit (which is the default), then the consumer will automatically commit offsets periodically at the
     | interval set by auto.commit.interval.ms.
     */
    'auto_commit' => env('KAFKA_AUTO_COMMIT', true),

    'sleep_on_error' => env('KAFKA_ERROR_SLEEP', 5),

    'partition' => env('KAFKA_PARTITION', 0),

    /*
     | Kafka supports 4 compression codecs: none , gzip , lz4 and snappy
     */
    'compression' => env('KAFKA_COMPRESSION_TYPE', 'snappy'),

    /*
     | Choose if debug is enabled or not.
     */
    'debug' => env('KAFKA_DEBUG', false),
    
     /*
     | Repository for batching messages together
     | Implement BatchRepositoryInterface to save batches in different storage
     */
    'batch_repository' => env('KAFKA_BATCH_REPOSITORY', \Junges\Kafka\BatchRepositories\InMemoryBatchRepository::class),

    /*
     | The sleep time in milliseconds that will be used when retrying flush
     */
    'flush_retry_sleep_in_ms' => 100,

    /*
     | The cache driver that will be used
     */
    'cache_driver' => env('KAFKA_CACHE_DRIVER', env('CACHE_DRIVER', 'file')),
];

```

关于如何使用，请查看 Laravel-kafka 使用文档 [4]

在 Swoole 中使用 Kafka
==================

对于 Swoole 应用，官方推荐使用 phpkafka[5]，由于是基于 swoole，所以 phpkafka 不需要 rdkafka 扩展的支持即可直接使用  
安装：

```
composer require longlang/phpkafka

```

生产者示例：

```
use longlang\phpkafka\Producer\Producer;
use longlang\phpkafka\Producer\ProducerConfig;
use longlang\phpkafka\Protocol\RecordBatch\RecordHeader;

$config = new ProducerConfig();
$config->setBootstrapServer('127.0.0.1:9092');
$config->setUpdateBrokers(true);
$config->setAcks(-1);
$producer = new Producer($config);
$topic = 'test';
$value = (string) microtime(true);
$key = uniqid('', true);
$producer->send('test', $value, $key);

// 指定 headers
// key-value或使用 RecordHeader 对象，都可以
$headers = [
    'key1' => 'value1',
    (new RecordHeader())->setHeaderKey('key2')->setValue('value2'),
];
$producer->send('test', $value, $key, $headers);

```

消费者示例：

```
use longlang\phpkafka\Consumer\ConsumeMessage;
use longlang\phpkafka\Consumer\Consumer;
use longlang\phpkafka\Consumer\ConsumerConfig;

function consume(ConsumeMessage $message)
{
    var_dump($message->getKey() . ':' . $message->getValue());
    // $consumer->ack($message); // autoCommit设为false时，手动提交
}
$config = new ConsumerConfig();
$config->setBroker('127.0.0.1:9092');
$config->setTopic('test'); // 主题名称
$config->setGroupId('testGroup'); // 分组ID
$config->setClientId('test'); // 客户端ID，不同的消费者进程请使用不同的设置
$config->setGroupInstanceId('test'); // 分组实例ID，不同的消费者进程请使用不同的设置
$config->setInterval(0.1);
$consumer = new Consumer($config, 'consume');
$consumer->start();

```

Kafka 和 MQTT 区别
===============

两者虽然都是从传统的 Pub/Sub 消息系统演化出来的，但是进化的方向不一样，Kafka 和 MQTT **是两种不同的消息传递协议和系统**，具有不同的设计目标和应用场景，他们的区别如下：

设计目标不同
------

Kafka 是一个高吞吐量、持久化的分布式消息系统，主要用于高效的数据发布、订阅和存储和构建大规模数据流和实时管道，而 MQTT 是一种**轻量级**的消息传输协议，主要用于在网络中传输小型数据包。它的设计目标是实现简单、可靠的设备间通信，**特别适用于物联网设备和低带宽、不稳定网络环境**。

通讯模式不同
------

Kafka 和 MQTT 虽然都使用了发布 / 订阅模式，但 Kafka 每个消息被处理后都仍然保留在 Kafka 服务器中，从而支持数据的持久化和离线消费。

消息可靠性保证
-------

Kafka 提供持久化的消息存储，确保消息在被处理之后仍然可供订阅者消费。**它具有至少一次交付保证**，可以通过适当的配置和副本设置来实现更高的可靠性。而 MQTT 提供了不同的消息传递服务质量（Quality of Service，QoS）级别，包括至多一次（at most once）、至少一次（at least once）和恰好一次（exactly once）交付。不同的 QoS 级别提供了不同的可靠性和传输效率

#### 引用链接

`[1]` rdkafka php extension: _https://github.com/confluentinc/librdkafka#installation_  
`[2]` igh-level consumer: _https://arnaud.le-blanc.net/php-rdkafka-doc/phpdoc/rdkafka.examples-high-level-consumer.html_  
`[3]` Laravel-kafka: _https://github.com/mateusjunges/laravel-kafka_  
`[4]` Laravel-kafka 使用文档: _https://junges.dev/documentation/laravel-kafka/v1.13/1-introduction_  
`[5]` phpkafka: _https://github.com/swoole/phpkafka_