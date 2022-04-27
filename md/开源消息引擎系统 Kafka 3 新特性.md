> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5ODYwMjI2MA==&mid=2649770210&idx=1&sn=5fe3cf70e9860583e37e9c93c7c69963&chksm=beccdb9989bb528f49915f3a64dbb1ad824a04718c309116f30af67c4766afa09189d91d725b&scene=21#wechat_redirect)

> kafka3.0 的版本已经试推行去 zk 的 kafka 架构了，如果去掉了 zk，那么在 kafka 新的版本当中使用什么技术来代替了 zk 的位置呢，接下来我们一起来一探究竟，了解 kafka 的内置共识机制和 raft 算法。

### 1、Kafka 简介

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvm9gfDpdpqEFbd7ygNMRsPSL05tbR6pHFnVpp2btQVyYAwPBnP9vMtNQ/640?wx_fmt=jpeg)

#### 1.1、Kafka 核心组件

**2.consumer**：消息消费者，就是从 broker 拉取数据的客户端。

**3.consumer group**：消费者组，由多个消费者 consumer 组成。消费者组内每个消费者负责消费不同的分区，一个分区只能由同一个消费者组内的一个消费者消费；消费者组之间相互独立，互不影响。所有的消费者都属于某个消费者组，即消费者组是一个逻辑上的订阅者。

**4.broker**：一台服务器就是一个 broker，一个集群由多个 broker 组成，一个 broker 可以有多个 topic。

**5.topic**：可以理解为一个队列，所有的生产者和消费者都是面向 topic 的。

**6.partition**：分区，kafka 中的 topic 为了提高拓展性和实现高可用而将它分布到不同的 broker 中，一个 topic 可以分为多个 partition，每个 partition 都是有序的，即消息发送到队列的顺序跟消费时拉取到的顺序是一致的。

**7.replication**：副本。一个 topic 对应的分区 partition 可以有多个副本，多个副本中只有一个为 leader，其余的为 follower。为了保证数据的高可用性，leader 和 follower 会尽量均匀的分布在各个 broker 中，避免了 leader 所在的服务器宕机而导致 topic 不可用的问题。

#### 1.2、kafka2 当中 zk 的作用

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvm3GozhnmiaR4CUpGwLy0yicH4tE6PZ9H0EabFBeUyibnIiazXUxfvXcq2RA/640?wx_fmt=jpeg)

1.  **/admin** ：主要保存 kafka 当中的核心的重要信息，包括类似于已经删除的 topic 就会保存在这个路径下面
    
2.  **/brokers** ：主要用于保存 kafka 集群当中的 broker 信息，以及没被删除的 topic 信息
    
3.  **/cluster** : 主要用于保存 kafka 集群的唯一 id 信息，每个 kafka 集群都会给分配要给唯一 id，以及对应的版本号
    
4.  **/config** : 集群配置信息
    
5.  **/controller** ：kafka 集群当中的控制器信息，控制器组件（Controller），是 Apache Kafka 的核心组件。它的主要作用是在 Apache ZooKeeper 的帮助下管理和协调整个 Kafka 集群。
    
6.  **/controller_epoch** ：主要用于保存记录 controller 的选举的次数
    
7.  **/isr_change_notification** ：isr 列表发生变更时候的通知，在 kafka 当中由于存在 ISR 列表变更的情况发生, 为了保证 ISR 列表更新的及时性，定义了 isr_change_notification 这个节点，主要用于通知 Controller 来及时将 ISR 列表进行变更
    
8.  **/latest_producer_id_block** ：使用`/latest_producer_id_block`节点来保存 PID 块，主要用于能够保证生产者的任意写入请求都能够得到响应。
    
9.  **/log_dir_event_notification** ：主要用于保存当 broker 当中某些 LogDir 出现异常时候, 例如磁盘损坏, 文件读写失败等异常时候, 向 ZK 当中增加一个通知序号，controller 监听到这个节点的变化之后，就会做出对应的处理操作。
    

以上就是 kafka 在 zk 当中保留的所有的所有的相关的元数据信息，这些元数据信息保证了 kafka 集群的正常运行。

### 2、kafka3 的安装配置

在 kafka3 的版本当中已经彻底去掉了对 zk 的依赖，如果没有了 zk 集群，那么 kafka 当中是如何保存元数据信息的呢，这里我们通过 kafka3 的集群来一探究竟。

#### 2.1、kafka 安装配置核心重要参数

Controller 服务器

不管是 kafka2 还是 kafka3 当中，controller 控制器都是必不可少的，通过 controller 控制器来维护 kafka 集群的正常运行，例如 ISR 列表的变更，broker 的上线或者下线，topic 的创建，分区的指定等等各种操作都需要依赖于 Controller，在 kafka2 当中，controller 的选举需要通过 zk 来实现，我们没法控制哪些机器选举成为 Controller, 而在 kafka3 当中, 我们可以通过配置文件来自己指定哪些机器成为 Controller, 这样做的好处就是我们可以指定一些配置比较高的机器作为 Controller 节点, 从而保证 controller 节点的稳健性。

被选中的 controller 节点参与元数据集群的选举，每个 controller 节点要么是 Active 状态，或者就是 standBy 状态。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvm8A7QeGtXeVhUxn0U0TWsM86o7wDmZVVKlntNT0IQYjB2HVibWRxicAUg/640?wx_fmt=jpeg)

##### **2.1.1、Process.Roles**

使用 KRaft 模式来运行 kafka 集群的话，我们有一个配置叫做 Process.Roles 必须配置，这个参数有以下四个值可以进行配置：

*   Process.Roles = Broker, 服务器在 KRaft 模式中充当 Broker。
    
*   Process.Roles = Controller, 服务器在 KRaft 模式下充当 Controller。
    
*   Process.Roles = Broker,Controller，服务器在 KRaft 模式中同时充当 Broker 和 Controller。
    
*   如果 process.roles 没有设置。那么集群就假定是运行在 ZooKeeper 模式下。
    

如果需要从 zookeeper 模式转换成为 KRaft 模式，那么需要进行重新格式化。如果一个节点同时是 Broker 和 Controller 节点, 那么就称之为组合节点。

实际工作当中，如果有条件的话，尽量还是将 Broker 和 Controller 节点进行分离部署。避免由于服务器资源不够的情况导致 OOM 等一系列的问题

##### **2.1.2、Quorum Voters**

通过 controller.quorum.voters 配置来实习哪些节点是 Quorum 的投票节点, 所有想要成为控制器的节点, 都必须放到这个配置里面。

每个 Broker 和每个 Controller 都必须配置 Controller.quorum.voters，该配置当中提供的节点 ID 必须与提供给服务器的节点 ID 保持一直。

每个 Broker 和每个 Controller 都必须设置 **`controller.quorum.voters`**。需要注意的是，controller.quorum.voters 配置中提供的节点 ID 必须与提供给服务器的节点 ID 匹配。

比如在 Controller1 上，node.Id 必须设置为 1，以此类推。注意，控制器 id 不强制要求你从 0 或 1 开始。然而，分配节点 ID 的最简单和最不容易混淆的方法是给每个服务器一个数字 ID，然后从 0 开始。

#### 2.2、下载并解压安装包

bigdata01 下载 kafka 的安装包，并进行解压：

修改 kafka 的配置文件 broker.properties：

```
[hadoop@bigdata01 kraft]$ cd /opt/soft/<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">[hadoop@bigdata01 soft]$ wget http://archive.apache.org/dist/kafka/3.1.0/kafka_2.12-3.1.0.tgz<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">[hadoop@bigdata01 soft]$ tar -zxf kafka_2.12-3.1.0.tgz -C /opt/install/<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">
```

修改编辑内容如下：

```
[hadoop@bigdata01 kafka_2.12-3.1.0]$ cd /opt/install/kafka_2.12-3.1.0/config/kraft/<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">[hadoop@bigdata01 kraft]$ vim broker.properties<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">
```

创建两个文件夹：

```
node.id=1<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">controller.quorum.voters=1@bigdata01:9093<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">listeners=PLAINTEXT://bigdata01:9092<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">advertised.listeners=PLAINTEXT://bigdata01:9092<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">log.dirs=/opt/install/kafka_2.12-3.1.0/kraftlogs<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">
```

同步安装包到其他机器上面去

#### 2.3、服务器集群启动

启动 kafka 服务：

```
[hadoop@bigdata01 kafka_2.12-3.1.0]$ mkdir -p /opt/install/kafka_2.12-3.1.0/kraftlogs<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">[hadoop@bigdata01 kafka_2.12-3.1.0]$ mkdir -p /opt/install/kafka_2.12-3.1.0/topiclogs<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">
```

#### 2.4、创建 kafka 的 topic

集群启动成功之后，就可以来创建 kafka 的 topic 了，使用以下命令来创建 kafka 的 topic

```
[hadoop@bigdata01 kafka_2.12-3.1.0]$  ./bin/kafka-storage.sh random-uuid<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">YkJwr6RESgSJv-sxa1R1mA<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">[hadoop@bigdata01 kafka_2.12-3.1.0]$  ./bin/kafka-storage.sh format -t YkJwr6RESgSJv-sxa1R1mA -c ./config/kraft/server.properties<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">Formatting /opt/install/kafka_2.12-3.1.0/topiclogs<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">[hadoop@bigdata01 kafka_2.12-3.1.0]$ ./bin/kafka-server-start.sh ./config/kraft/server.properties<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">
```

#### 2.5、任意一台机器查看 kafka 的 topic

组成集群之后，任意一台机器就可以通过以下命令来查看到刚才创建的 topic 了

```
./bin/kafka-topics.sh --create --topic kafka_test --partitions 3 --replication-factor 2 --bootstrap-server bigdata01:9092,bigdata02:9092,bigdata03:9092<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">
```

#### 2.6、消息生产与消费

使用命令行来生产以及消费 kafka 当中的消息

```
[hadoop@bigdata03 ~]$ cd /opt/install/kafka_2.12-3.1.0/<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">[hadoop@bigdata03 kafka_2.12-3.1.0]$ bin/kafka-topics.sh  --list --bootstrap-server bigdata01:9092,bigdata02:9092,bigdata03:9092<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">
```

### 3、Kafka 当中 Raft 的介绍

#### 3.1、kafka 强依赖 zk 所引发的问题

前面我们已经看到了 kafka3 集群在没有 zk 集群的依赖下，也可以正常运行，那么 kafka2 在 zk 当中保存的各种重要元数据信息，在 kafka3 当中如何实现保存的呢？

kafka 一直都是使用 zk 来管理集群以及所有的 topic 的元数据，并且使用了 zk 的强一致性来选举集群的 controller，controller 对整个集群的管理至关重要，包括分区的新增，ISR 列表的维护，等等很多功能都需要靠 controller 来实现，然后使用 zk 来维护 kafka 的元数据也存在很多的问题以及存在性能瓶颈。

以下是 kafka 将元数据保存在 zk 当中的诸多问题。

**1、元数据存取困难**

元数据的存取过于困难，每次重新选举的 controller 需要把整个集群的元数据重新 restore，非常的耗时且影响集群的可用性。

**2、元数据更新网络开销大**

整个元数据的更新操作也是以全量推的方式进行，网络的开销也会非常大。

**3、强耦合违背软件设计原则**

Zookeeper 对于运维来说，维护 Zookeeper 也需要一定的开销，并且 kafka 强耦合与 zk 也并不好，还得时刻担心 zk 的宕机问题，违背软件设计的高内聚，低耦合的原则。

**4、网络分区复杂度高**

Zookeeper 本身并不能兼顾到 broker 与 broker 之间通信的状态，这就会导致网络分区的复杂度成几何倍数增长。

**5、zk 本身不适合做消息队列**

zookeeper 不适合做消息队列，因为 zookeeper 有 1M 的消息大小限制 zookeeper 的 children 太多会极大的影响性能 znode 太大也会影响性能 znode 太大会导致重启 zkserver 耗时 10-15 分钟 zookeeper 仅使用内存作为存储，所以不能存储太多东西。

**6、并发访问 zk 问题多**

最好单线程操作 zk 客户端，不要并发，临界、竞态问题太多

基于以上各种问题，所以提出了脱离 zk 的方案，转向自助研发强一致性的元数据解决方案，也就是 KIP-500。

KIP-500 议案提出了在 Kafka 中处理元数据的更好方法。基本思想是 "**Kafka on Kafka**"，将 Kafka 的元数据存储在 Kafka 本身中，无需增加额外的外部存储比如 ZooKeeper 等。

去 zookeeper 之后的 kafka 新的架构

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvm8A7QeGtXeVhUxn0U0TWsM86o7wDmZVVKlntNT0IQYjB2HVibWRxicAUg/640?wx_fmt=jpeg)

在 KIP-500 中，Kafka 控制器会将其元数据存储在 Kafka 分区中，而不是存储在 ZooKeeper 中。但是，由于控制器依赖于该分区，因此分区本身不能依赖控制器来进行领导者选举之类的事情。而是，管理该分区的节点必须实现自我管理的 Raft 仲裁。

在 kafka3.0 的新的版本当中，使用了新的 KRaft 协议，使用该协议来保证在元数据仲裁中准确的复制元数据，这个协议类似于 zk 当中的 zab 协议以及类似于 Raft 协议，但是 KRaft 协议使用的是基于事件驱动的模式，与 ZAB 协议和 Raft 协议还有点不一样

在 kafka3.0 之前的的版本当中，主要是借助于 controller 来进行 leader partition 的选举，而在 3.0 协议当中，使用了 KRaft 来实现自己选择 leader，并最终令所有节点达成共识，这样简化了 controller 的选举过程，效果更加高效。

#### 3.2、kakfa3 Raft

前面我们已经知道了在 kafka3 当中可以不用再依赖于 zk 来保存 kafka 当中的元数据了，转而使用 Kafka Raft 来实现元数据的一致性，简称 **KRaft**，并且将元数据保存在 kafka 自己的服务器当中，大大提高了 kafka 的元数据管理的性能。

KRaft 运行模式的 Kafka 集群，不会将元数据存储在 Apache ZooKeeper 中。即部署新集群的时候，无需部署 ZooKeeper 集群，因为 Kafka 将元数据存储在 Controller 节点的 KRaft Quorum 中。KRaft 可以带来很多好处，比如可以支持更多的分区，更快速的切换 Controller，也可以避免 Controller 缓存的元数据和 Zookeeper 存储的数据不一致带来的一系列问题。

在新的版本当中，控制器 Controller 节点我们可以自己进行指定, 这样最大的好处就是我们可以自己选择一些配置比较好的机器成为 Controller 节点，而不像在之前的版本当中，我们无法指定哪台机器成为 Controller 节点，而且 controller 节点与 broker 节点可以运行在同一台机器上，并且控制器 controller 节点不再向 broker 推送更新消息, 而是让 Broker 从这个 Controller Leader 节点进行拉去元数据的更新。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvmsmSuxkgVjgyPCDJxsZADibiaupTmF6J5UO7vquZAozrjWaETKgzgwOJA/640?wx_fmt=jpeg)

#### 3.3、如何查看 kafka3 当中的元数据信息

在 kafka3 当中，不再使用 zk 来保存元数据信息了，那么在 kafka3 当中如何查看元数据信息呢，我们也可以通过 kafka 自带的命令来进行查看元数据信息，在 KRaft 中，有两个命令常用命令脚本，kafka-dump-log.sh 和 kakfa-metadata-shell.sh 需要我们来进行关注，因为我们可以通过这两个脚本来查看 kafka 当中保存的元数据信息。

##### 3.3.1、**Kafka-dump-log.sh** 脚本来导出元数据信息

KRaft 模式下，所有的元数据信息都保存到了一个内部的 topic 上面，叫做 @metadata，例如 Broker 的信息, Topic 的信息等, 我们都可以去到这个 topic 上面进行查看, 我们可以通过 kafka-dump-log.sh 这个脚本来进行查看该 topic 的信息。

Kafka-dump-log.sh 是一个之前就有的工具，用来查看 Topic 的的文件内容。这工具加了一个参数 --cluster-metadata-decoder 用来，查看元数据日志，如下所示:

```
[hadoop@bigdata01 kafka_2.12-3.1.0]$ bin/kafka-console-producer.sh --bootstrap-server bigdata01:9092,bigdata02:9092,bigdata03:9092 --topic kafka_test<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)"><br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">[hadoop@bigdata02 kafka_2.12-3.1.0]$ bin/kafka-console-consumer.sh --bootstrap-server bigdata01:9092,bigdata02:9092,bigdata03:9092 --topic kafka_test --from-beginning<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">
```

##### 3.3.2、kafka-metadata-shell.sh 直接查看元数据信息

平时我们用 zk 的时候，习惯了用 zk 命令行查看数据，简单快捷。bin 目录下自带了 kafka-metadata-shell.sh 工具，可以允许你像 zk 一样方便的查看数据。

使用 kafka-metadata-shell.sh 脚本进入 kafka 的元数据客户端

```
[hadoop@bigdata01 kafka_2.12-3.1.0]$ cd /opt/install/kafka_2.12-3.1.0<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">[hadoop@bigdata01 kafka_2.12-3.1.0]$ bin/kafka-dump-log.sh  --cluster-metadata-decoder --skip-record-metadata  --files  /opt/install/kafka_2.12-3.1.0/topiclogs/__cluster_metadata-0/00000000000000000000.index,/opt/install/kafka_2.12-3.1.0/topiclogs/__cluster_metadata-0/00000000000000000000.log  >>/opt/metadata.txt<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">
```

### 4、Raft 算法介绍

raft 算法中文版本翻译介绍：https://github.com/maemual/raft-zh_cn/blob/master/raft-zh_cn.md

著名的 **CAP** 原则又称 CAP 定理的提出，真正奠基了分布式系统的诞生，CAP 定理指的是在一个分布式系统中，[一致性]、[可用性]（Availability）、[分区容错性]（Partition tolerance），这三个要素最多只能同时实现两点，不可能三者兼顾 (nosql)。

分布式系统为了提高系统的可靠性，一般都会选择使用多副本的方式来进行实现，例如 hdfs 当中数据的多副本，kafka 集群当中分区的多副本等，但是一旦有了多副本的话，那么久面临副本之间一致性的问题，而一致性算法就是 用于解决分布式环境下多副本的数据一致性的问题。业界最著名的一致性算法就是大名鼎鼎的 Paxos，但是 **Paxos** 比较晦涩难懂，不太容易理解，所以还有一种叫做 Raft 的算法，更加简单容易理解的实现了一致性算法。

#### 4.1、Raft 协议的工作原理

##### 4.1.1、Raft 协议当中的角色分布

Raft 协议将分布式系统当中的角色分为 Leader（领导者），Follower（跟从者）以及 Candidate（候选者）

*   **Leader**：主节点的角色，主要是接收客户端请求，并向 Follower 同步日志，当日志同步到过半及以上节点之后，告诉 follower 进行提交日志
    
*   **Follower**：从节点的角色，接受并持久化 Leader 同步的日志，在 Leader 通知可以提交日志之后，进行提交保存的日志
    
*   **Candidate**：Leader 选举过程中的临时角色。
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvmgeSr1iclMTtPRdAUiaRt4uYfuNH64bmIFXsZ6qoBe8KqIuJnQ0v4lwtw/640?wx_fmt=jpeg)

##### 4.1.2、Raft 协议当中的底层原理

Raft 协议当中会选举出 Leader 节点，Leader 作为主节点，完全负责 replicate log 的管理。Leader 负责接受所有客户端的请求，然后复制到 Follower 节点，如果 leader 故障，那么 follower 会重新选举 leader，Raft 协议的一致性，概括主要可以分为以下三个重要部分

*   Leader 选举
    
*   日志复制
    
*   安全性
    

其中 Leader 选举和日志复制是 Raft 协议当中最为重要的。

Raft 协议要求系统当中，任意一个时刻，只有一个 leader，正常工作期间，只有 Leader 和 Follower 角色，并且 Raft 协议采用了类似网络租期的方式来进行管理维护整个集群，Raft 协议将时间分为一个个的时间段（term），也叫作任期，每一个任期都会选举一个 Leader 来管理维护整个集群，如果这个时间段的 Leader 宕机，那么这一个任期结束，继续重新选举 leader。

**Raft 算法将时间划分成为任意不同长度的任期（term）**。任期用连续的数字进行表示。**每一个任期的开始都是一次选举（election），一个或多个候选人会试图成为领导人**。如果一个候选人赢得了选举，它就会在该任期的剩余时间担任领导人。在某些情况下，选票会被瓜分，有可能没有选出领导人，那么，将会开始另一个任期，并且立刻开始下一次选举。**Raft 算法保证在给定的一个任期最多只有一个领导人**。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvmJoib2KIOKGGaGyMwk93L8qBUcribOeg6iapNET2TwDGJ9z8HdMkXBcgyA/640?wx_fmt=jpeg)

##### 4.1.3、Leader 选举的过程

Raft 使用心跳来进行触发 leader 选举，当服务器启动时，初始化为 follower 角色。leader 向所有 Follower 发送周期性心跳，如果 Follower 在选举超时间内没有收到 Leader 的心跳，就会认为 leader 宕机，稍后发起 leader 的选举。

每个 Follower 都会有一个倒计时时钟，是一个随机的值，表示的是 Follower 等待成为 Leader 的时间，倒计时时钟先跑完，就会当选成为 Leader，这样做得好处就是每一个节点都有机会成为 Leader。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvmbCDqokgAeicKCsicspKFGJXIDzk9t2RnWDIrCXWiaNhQC1L2JWrK26uaQ/640?wx_fmt=jpeg)

当满足以下三个条件之一时，Quorum 中的某个节点就会触发选举：

1.  向 Leader 发送 Fetch 请求后，在超时阈值 quorum.fetch.timeout.ms 之后仍然没有得到 Fetch 响应，表示 Leader 疑似失败；
    
2.  从当前 Leader 收到了 EndQuorumEpoch 请求，表示 Leader 已退位；
    
3.  Candidate 状态下，在超时阈值 quorum.election.timeout.ms 之后仍然没有收到多数票，也没有 Candidate 赢得选举，表示此次选举作废，重新进行选举。
    

具体详细过程实现描述如下：

*   增加节点本地的 current term，切换到 candidate 状态
    
*   自己给自己投一票
    
*   给其他节点发送 RequestVote RPCs，要求其他节点也投自己一票
    
*   等待其他节点的投票回复
    

整个过程中的投票过程可以用下图进行表述：

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvm4kuRb9mYmOcQy9teZaicviaicFkryuuKvd3jyGBAy8H5ZaxsMzamob02w/640?wx_fmt=jpeg)

leader 节点选举的限制

*   每个节点只能投一票，投给自己或者投给别人
    
*   候选人所知道的日志信息，一定不能比自己的更少，即能被选举成为 leader 节点，一定包含了所有已经提交的日志
    
*   先到先得的原则
    

##### 4.1.4、数据一致性保证（日志复制机制）

前面通过选举机制之后，选举出来了 leader 节点，然后 leader 节点对外提供服务，所有的客户端的请求都会发送到 leader 节点，由 leader 节点来调度这些并发请求的处理顺序，保证所有节点的状态一致，**leader 会把请求作为日志条目（Log entries）加入到他的日志当中，然后并行的向其他服务器发起 AppendEntries RPC 复制日志条目。**当这条请求日志被成功复制到大多数服务器上面之后，Leader 将这条日志应用到它的状态机并向客户端返回执行结果。

*   客户端的每个请求都包含被复制状态机执行的指令
    
*   leader 将客户端请求作为一条心得日志添加到日志文件中，然后并行发起 RPC 给其他的服务器，让他们复制这条信息到自己的日志文件中保存。
    
*   如果这条日志被成功复制，也就是大部分的 follower 都保存好了执行指令日志，leader 就应用这条日志到自己的状态机中，并返回给客户端。
    
*   如果 follower 宕机或者运行缓慢或者数据丢失，leader 会不断地进行重试，直至所有在线的 follower 都成功复制了所有的日志条目。
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvmo7b5vcfmfDlcp2xmTFuicjevvqa6ad2p4vib7VILhkibtr42k2WqJ942g/640?wx_fmt=jpeg)

与维护 Consumer offset 的方式类似，脱离 ZK 之后的 Kafka 集群将元数据视为日志，保存在一个内置的 Topic 中，且该 Topic 只有一个 Partition。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvmqHK3CLP8yUhoXichnPkz0fMWut8azic9ZOlv7zAZ1j23Q59E5oX9ee0A/640?wx_fmt=jpeg)

元数据日志的消息格式与普通消息没有太大不同，但必须携带 Leader 的纪元值 (即之前的 Controller epoch)：

```
[hadoop@bigdata01 kafka_2.12-3.1.0]$ bin/kafka-metadata-shell.sh --snapshot /opt/install/kafka_2.12-3.1.0/topiclogs/__cluster_metadata-0/00000000000000000000.log<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">
```

这样，Follower 以拉模式复制 Leader 日志，就相当于以 Consumer 角色消费元数据 Topic，符合 Kafka 原生的语义。

那么在 KRaft 协议中，是如何维护哪些元数据日志已经提交——即已经成功复制到多数的 Follower 节点上的呢？Kafka 仍然借用了原生副本机制中的概念——high watermark(HW，高水位线) 保证日志不会丢失，HW 的示意图如下。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvmGTgpgnoneFSGNTtkh3xIHNQvhuHctHgmCI2erDPGuwCtTmNCFdb3jg/640?wx_fmt=jpeg)

##### **状态机说明**

要让所有节点达成一致性的状态，大部分都是基于复制状态机来实现的（Replicated state machine）

简单来说就是：**初始相同的状态 + 相同的输入过程 = 相同的结束状态**，这个其实也好理解，就类似于一对双胞胎，出生时候就长得一样，然后吃的喝的用的穿的都一样，你自然很难分辨。其中最重要的就是一定要注意中间的相同输入过程，各个不同节点要以相同且确定性的函数来处理输入，而不要引入一个不确定的值。使用 replicated log 来实现每个节点都顺序的写入客户端请求，然后顺序的处理客户端请求，最终就一定能够达到最终一致性。

##### **状态机安全性保证**

在安全性方面，KRaft 与传统 Raft 的选举安全性、领导者只追加、日志匹配和领导者完全性保证都是几乎相同的。下面只简单看看状态机安全性是如何保证的，仍然举论文中的极端例子：

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvmhU1vxgL3emUxu2KXr5UtbALa7N77hN1c2m4IMjwlpdSjZpSdib195WA/640?wx_fmt=jpeg)

1.  在时刻 a，节点 S1 是 Leader，epoch=2 的日志只复制给了 S2 就崩溃了；
    
2.  在时刻 b，S5 被选举为 Leader，epoch=3 的日志还没来得及复制，也崩溃了；
    
3.  在时刻 c，S1 又被选举为 Leader，继续复制日志，将 epoch=2 的日志给了 S3。此时该日志复制给了多数节点，但还未提交；
    
4.  在时刻 d，S1 又崩溃，并且 S5 重新被选举为领导者，将 epoch=3 的日志复制给 S0~S4。
    

此时日志与新 Leader S5 的日志发生了冲突，如果按上图中 d1 的方式处理，消息 2 就会丢失。传统 Raft 协议的处理方式是：在 Leader 任期开始时，立刻提交一条空的日志，所以上图中时刻 c 的情况不会发生，而是如同 d2 一样先提交 epoch=4 的日志，连带提交 epoch=2 的日志。

与传统 Raft 不同，KRaft 附加了一个较强的约束：当新的 Leader 被选举出来，但还没有成功提交属于它的 epoch 的日志时，不会向前推进 HW。也就是说，即使上图中时刻 c 的情况发生了，消息 2 也被视为没有成功提交，所以按照 d1 方式处理是安全的。

##### **日志格式说明**

所有节点持久化保存在本地的日志，大概就是类似于这个样子：

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvmss90x7Qv7S8BvlK3tkZibtnticj731PruZRcfaXF5ic3TIq0jSgLiblP8w/640?wx_fmt=jpeg)

上图显示，共有八条日志数据，其中已经提交了 7 条，提交的日志都将通过状态机持久化到本地磁盘当中，防止宕机。

**日志复制的保证机制**

*   如果两个节点不同的日志文件当中存储着相同的索引和任期号，那么他们所存储的命令是相同的。（原因：leader 最多在一个任期里的一个日志索引位置创建一条日志条目，日志条目所在的日志位置从来不会改变）。
    
*   如果不同日志中两个条目有着相同的索引和任期号，那么他们之前的所有条目都是一样的（原因：每次 RPC 发送附加日志时，leader 会把这条日志前面的日志下标和任期号一起发送给 follower，如果 follower 发现和自己的日志不匹配，那么就拒绝接受这条日志，这个称之为一致性检查）
    

##### 日志的不正常情况

一般情况下，Leader 和 Followers 的日志保持一致，因此 Append Entries 一致性检查通常不会失败。然而，Leader 崩溃可能会导致日志不一致：**旧的 Leader 可能没有完全复制完日志中的所有条目**。

下图阐述了一些 Followers 可能和新的 Leader 日志不同的情况。**一个 Follower 可能会丢失掉 Leader 上的一些条目，也有可能包含一些 Leader 没有的条目，也有可能两者都会发生**。丢失的或者多出来的条目可能会持续多个任期。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvmJUB7bhvjGoTicmThXSusE8Jq9IL4bFJvjKOCsl7EVIs8AM6uPqXYkhg/640?wx_fmt=jpeg)

##### 如何保证日志的正常复制

如果出现了上述 leader 宕机，导致 follower 与 leader 日志不一致的情况，那么就需要进行处理，保证 follower 上的日志与 leader 上的日志保持一致，leader 通过强制 follower 复制它的日志来处理不一致的问题，follower 与 leader 不一致的日志会被强制覆盖。**leader 为了最大程度的保证日志的一致性，且保证日志最大量，leader 会寻找 follower 与他日志一致的地方，然后覆盖 follower 之后的所有日志条目，从而实现日志数据的一致性。**

具体的操作就是：leader 会从后往前不断对比，每次 Append Entries 失败后尝试前一个日志条目，直到成功找到每个 Follower 的日志一致的位置点，然后向该 Follower 所在位置之后的条目进行覆盖。

**详细过程如下：**

*   Leader 维护了每个 Follower 节点下一次要接收的日志的索引，即 nextIndex
    
*   Leader 选举成功后将所有 Follower 的 nextIndex 设置为自己的最后一个日志条目 + 1
    
*   Leader 将数据推送给 Follower，如果 Follower 验证失败（nextIndex 不匹配），则在下一次推送日志时缩小 nextIndex，直到 nextIndex 验证通过
    

总结一下就是：**当 leader 和 follower 日志冲突的时候**，leader 将**校验 follower 最后一条日志是否和 leader 匹配**，如果不匹配，**将递减查询，直到匹配，匹配后，删除冲突的日志**。这样就实现了主从日志的一致性。

#### 4.2、Raft 协议算法代码实现

前面我们已经大致了解了 Raft 协议算法的实现原理，如果我们要自己实现一个 Raft 协议的算法，其实就是将我们讲到的理论知识给翻译成为代码的过程，具体的开发需要考虑的细节比较多，代码量肯定也比较大，好在有人已经实现了 Raft 协议的算法了，我们可以直接拿过来使用

创建 maven 工程并导入 jar 包地址如下：

```
Record => Offset LeaderEpoch ControlType Key Value Timestamp<br data-darkmode-color-16510825349488="rgb(179, 179, 179)" data-darkmode-original-color-16510825349488="#fff|rgb(0,0,0)|rgb(51, 51, 51)" data-darkmode-bgcolor-16510825349488="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16510825349488="#fff|rgb(248, 248, 248)">
```

定义 Server 端代码实现：

```
<dependencies>        <dependency>            <groupId>com.github.wenweihu86.raft</groupId>            <artifactId>raft-java-core</artifactId>            <version>1.8.0</version>        </dependency>        <dependency>            <groupId>com.github.wenweihu86.rpc</groupId>            <artifactId>rpc-java</artifactId>            <version>1.8.0</version>        </dependency>        <dependency>            <groupId>org.rocksdb</groupId>            <artifactId>rocksdbjni</artifactId>            <version>5.1.4</version>        </dependency>    </dependencies>    <build>        <plugins>            <plugin>                <groupId>org.apache.maven.plugins</groupId>                <artifactId>maven-compiler-plugin</artifactId>                <version>3.5.1</version>                <configuration>                    <source>1.8</source>                    <target>1.8</target>                </configuration>            </plugin>        </plugins>    </build>
```

定义客户端代码实现如下：

```
public class Server1 {    public static void main(String[] args) {        // parse args        // peers, format is "host:port:serverId,host2:port2:serverId2"        //localhost:16010:1,localhost:16020:2,localhost:16030:3 localhost:16010:1        String servers = "localhost:16010:1,localhost:16020:2,localhost:16030:3";        // local server        RaftMessage.Server localServer = parseServer("localhost:16010:1");        String[] splitArray = servers.split(",");        List<RaftMessage.Server> serverList = new ArrayList<>();        for (String serverString : splitArray) {            RaftMessage.Server server = parseServer(serverString);            serverList.add(server);        }        // 初始化RPCServer        RPCServer server = new RPCServer(localServer.getEndPoint().getPort());        // 设置Raft选项，比如：        // just for test snapshot        RaftOptions raftOptions = new RaftOptions();      /*  raftOptions.setSnapshotMinLogSize(10 * 1024);        raftOptions.setSnapshotPeriodSeconds(30);        raftOptions.setMaxSegmentFileSize(1024 * 1024);*/        // 应用状态机        ExampleStateMachine stateMachine = new ExampleStateMachine(raftOptions.getDataDir());        // 初始化RaftNode        RaftNode raftNode = new RaftNode(raftOptions, serverList, localServer, stateMachine);        raftNode.getLeaderId();        // 注册Raft节点之间相互调用的服务        RaftConsensusService raftConsensusService = new RaftConsensusServiceImpl(raftNode);        server.registerService(raftConsensusService);        // 注册给Client调用的Raft服务        RaftClientService raftClientService = new RaftClientServiceImpl(raftNode);        server.registerService(raftClientService);        // 注册应用自己提供的服务        ExampleService exampleService = new ExampleServiceImpl(raftNode, stateMachine);        server.registerService(exampleService);        // 启动RPCServer，初始化Raft节点        server.start();        raftNode.init();    }    private static RaftMessage.Server parseServer(String serverString) {        String[] splitServer = serverString.split(":");        String host = splitServer[0];        Integer port = Integer.parseInt(splitServer[1]);        Integer serverId = Integer.parseInt(splitServer[2]);        RaftMessage.EndPoint endPoint = RaftMessage.EndPoint.newBuilder()                .setHost(host).setPort(port).build();        RaftMessage.Server.Builder serverBuilder = RaftMessage.Server.newBuilder();        RaftMessage.Server server = serverBuilder.setServerId(serverId).setEndPoint(endPoint).build();        return server;    }}
```

先启动服务端，然后启动客户端，就可以将实现客户端向服务端发送消息，并且服务端会向三台机器进行保存消息了。

### 5、Kafka 常见问题

#### **1. 消息队列模型知道吗? Kafka 是怎么做到支持这两种模型的？**

对于传统的消息队列系统支持两个模型：

1. 点对点：也就是消息只能被一个消费者消费，消费完后消息删除

2. 发布订阅：相当于广播模式，消息可以被所有消费者消费

kafka 其实就是通过 Consumer Group 同时支持了这两个模型。如果说所有消费者都属于一个 Group，消息只能被同一个 Group 内的一个消费者消费，那就是点对点模式。如果每个消费者都是一个单独的 Group，那么就是发布订阅模式。

#### **2. 说说 Kafka 通信过程原理吗?**

1. 首先 kafka broker 启动的时候，会去向 Zookeeper 注册自己的 ID（创建临时节点），这个 ID 可以配置也可以自动生成，同时会去订阅 Zookeeper 的 brokers/ids 路径，当有新的 broker 加入或者退出时，可以得到当前所有 broker 信。

2. 生产者启动的时候会指定 bootstrap.servers，通过指定的 broker 地址，Kafka 就会和这些 broker 创建 TCP 连接（通常我们不用配置所有的 broker 服务器地址，否则 kafka 会和配置的所有 broker 都建立 TCP 连接）。

3. 随便连接到任何一台 broker 之后，然后再发送请求获取元数据信息（包含有哪些主题、主题都有哪些分区、分区有哪些副本，分区的 Leader 副本等信息）

4. 接着就会创建和所有 broker 的 TCP 连接

5. 之后就是发送消息的过程

6. 消费者和生产者一样，也会指定 bootstrap.servers 属性，然后选择一台 broker 创建 TCP 连接，发送请求找到协调者所在的 broker

7. 然后再和协调者 broker 创建 TCP 连接，获取元数据

8. 根据分区 Leader 节点所在的 broker 节点，和这些 broker 分别创建连接

9. 最后开始消费消息

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvmP9kDkliaKThbQz8hfTz8KanLYxXWlU3FQTRibZUZgybtgug0oI2Pa0Ow/640?wx_fmt=jpeg)

#### **3. 发送消息时如何选择分区的?**

主要有两种方式：

1. 轮询，按照顺序消息依次发送到不同的分区

2. 随机，随机发送到某个分区

如果消息指定 key，那么会根据消息的 key 进行 hash，然后对 partition 分区数量取模，决定落在哪个分区上，所以，对于相同 key 的消息来说，总是会发送到同一个分区上，也是我们常说的消息分区有序性。很常见的场景就是我们希望下单、支付消息有顺序，这样以订单 ID 作为 key 发送消息就达到了分区有序性的目的。如果没有指定 key，会执行默认的轮询负载均衡策略，比如第一条消息落在 P0，第二条消息落在 P1，然后第三条又在 P1。

除此之外，对于一些特定的业务场景和需求，还可以通过实现 Partitioner 接口，重写 configure 和 partition 方法来达到自定义分区的效果。

#### **4. 为什么需要分区? 有什么好处?**

这个问题很简单，如果说不分区的话，我们发消息写数据都只能保存到一个节点上，这样的话就算这个服务器节点性能再好最终也支撑不住。

实际上分布式系统都面临这个问题，要么收到消息之后进行数据切分，要么提前切分，kafka 正是选择了前者，通过分区可以把数据均匀地分布到不同的节点。

分区带来了负载均衡和横向扩展的能力。

发送消息时可以根据分区的数量落在不同的 Kafka 服务器节点上，提升了并发写消息的性能，消费消息的时候又和消费者绑定了关系，可以从不同节点的不同分区消费消息，提高了读消息的能力。

另外一个就是分区又引入了副本，冗余的副本保证了 Kafka 的高可用和高持久性。

#### **5. 详细说说消费者组和消费者重平衡？**

Kafka 中的消费者组订阅 topic 主题的消息，一般来说消费者的数量最好要和所有主题分区的数量保持一致最好（举例子用一个主题，实际上当然是可以订阅多个主题）。

当消费者数量小于分区数量的时候，那么必然会有一个消费者消费多个分区的消息。而消费者数量超过分区的数量的时候，那么必然会有消费者没有分区可以消费。所以，消费者组的好处一方面在上面说到过，可以支持多种消息模型，另外的话根据消费者和分区的消费关系，支撑横向扩容伸缩。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvm2HWVcvmGNXXg2Kswpco1jbjCdic3oQDu5u5eunHiaRF3eBsQYv0VeEhw/640?wx_fmt=jpeg)

当我们知道消费者如何消费分区的时候，就显然会有一个问题出现了，消费者消费的分区是怎么分配的，有先加入的消费者时候怎么办？

旧版本的重平衡过程主要通过 ZK 监听器的方式来触发，每个消费者客户端自己去执行分区分配算法。新版本则是通过协调者来完成，每一次新的消费者加入都会发送请求给协调者去获取分区的分配，这个分区分配的算法逻辑由协调者来完成。

而重平衡 Rebalance 就是指的有新消费者加入的情况，比如刚开始我们只有消费者 A 在消费消息，过了一段时间消费者 B 和 C 加入了，这时候分区就需要重新分配，这就是重平衡，也可以叫做再平衡，但是重平衡的过程和我们的 GC 时候 STW 很像，会导致整个消费群组停止工作，重平衡期间都无法消息消息。另外，发生重平衡并不是只有这一种情况，因为消费者和分区总数是存在绑定关系的，上面也说了，消费者数量最好和所有主题的分区总数一样。

那只要**消费者数量**、**主题数量**（比如用的正则订阅的主题）、**分区数量**任何一个发生改变，都会触发重平衡。

下面说说重平衡的过程。

重平衡的机制依赖消费者和协调者之间的心跳来维持，消费者会有一个独立的线程去定时发送心跳给协调者，这个可以通过参数 heartbeat.interval.ms 来控制发送心跳的间隔时间。

**1. 每个消费者第一次加入组的时候都会向协调者发送 JoinGroup 请求，第一个发送这个请求的消费者会成为 “群主”，协调者会返回组成员列表给群主**

**2. 群主执行分区分配策略，然后把分配结果通过 SyncGroup 请求发送给协调者，协调者收到分区分配结果**

**3. 其他组内成员也向协调者发送 SyncGroup，协调者把每个消费者的分区分配分别响应给他们**

#### **6. 具体讲讲分区分配策略?**

主要有 3 种分配策略：

**Range**

对分区进行排序，排序越靠前的分区能够分配到更多的分区。比如有 3 个分区，消费者 A 排序更靠前，所以能够分配到 P0\P1 两个分区，消费者 B 就只能分配到一个 P2。如果是 4 个分区的话，那么他们会刚好都是分配到 2 个。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvm1nLZAVoPhEcme6uaaKjib832G30ZcWf4ssyyPUlDqdib2t6wsziaQibvDw/640?wx_fmt=jpeg)

但是这个分配策略会有点小问题，他是根据主题进行分配，所以如果消费者组订阅了多个主题，那就有可能导致分区分配不均衡。比如下图中两个主题的 P0\P1 都被分配给了 A，这样 A 有 4 个分区，而 B 只有 2 个，如果这样的主题数量越多，那么不均衡就越严重。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvmfNAEWY8G60RnFAFBc75tZ2afaib1PBmX1Z5VQeeU0ibeaRhOPsLTUeicw/640?wx_fmt=jpeg)

**RoundRobin**

也就是我们常说的轮询了，这个就比较简单了，不画图你也能很容易理解。这个会根据所有的主题进行轮询分配，不会出现 Range 那种主题越多可能导致分区分配不均衡的问题。

P0->A，P1->B，P1->A。。。以此类推

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvmkzq0moRTV5Eamtrx0vK53WVibia9SKLXFJKibGzuHIuJw0YhUoaZa5zibw/640?wx_fmt=jpeg)

**Sticky**

这个从字面看来意思就是粘性策略，大概是这个意思。主要考虑的是在分配均衡的前提下，让分区的分配更小的改动。比如之前 P0\P1 分配给消费者 A，那么下一次尽量还是分配给 A。这样的好处就是连接可以复用，要消费消息总是要和 broker 去连接的，如果能够保持上一次分配的分区的话，那么就不用频繁的销毁创建连接了。

#### **7. 如何保证消息可靠性?**

**生产者发送消息丢失**

kafka 支持 3 种方式发送消息，这也是常规的 3 种方式，发送后不管结果、同步发送、异步发送，基本上所有的消息队列都是这样玩的。

1. 发送并忘记，直接调用发送 send 方法，不管结果，虽然可以开启自动重试，但是肯定会有消息丢失的可能

2. 同步发送，同步发送返回 Future 对象，我们可以知道发送结果，然后进行处理

3. 异步发送，发送消息，同时指定一个回调函数，根据结果进行相应的处理

为了保险起见，一般我们都会使用异步发送带有回调的方式进行发送消息，再设置参数为发送消息失败不停地重试。

acks=all，这个参数有可以配置 0|1|all。

0 表示生产者写入消息不管服务器的响应，可能消息还在网络缓冲区，服务器根本没有收到消息，当然会丢失消息。

1 表示至少有一个副本收到消息才认为成功，一个副本那肯定就是集群的 Leader 副本了，但是如果刚好 Leader 副本所在的节点挂了，Follower 没有同步这条消息，消息仍然丢失了。配置 all 的话表示所有 ISR 都写入成功才算成功，那除非所有 ISR 里的副本全挂了，消息才会丢失。

retries=N，设置一个非常大的值，可以让生产者发送消息失败后不停重试

**Kafka 自身消息丢失**

kafka 因为消息写入是通过 PageCache 异步写入磁盘的，因此仍然存在丢失消息的可能。

因此针对 kafka 自身丢失的可能设置参数：

replication.factor=N，设置一个比较大的值，保证至少有 2 个或者以上的副本。

min.insync.replicas=N，代表消息如何才能被认为是写入成功，设置大于 1 的数，保证至少写入 1 个或者以上的副本才算写入消息成功。

unclean.leader.election.enable=false，这个设置意味着没有完全同步的分区副本不能成为 Leader 副本，如果是 true 的话，那些没有完全同步 Leader 的副本成为 Leader 之后，就会有消息丢失的风险。

**消费者消息丢失**

消费者丢失的可能就比较简单，关闭自动提交位移即可，改为业务处理成功手动提交。因为重平衡发生的时候，消费者会去读取上一次提交的偏移量，自动提交默认是每 5 秒一次，这会导致重复消费或者丢失消息。

enable.auto.commit=false，设置为手动提交。

还有一个参数我们可能也需要考虑进去的：

auto.offset.reset=earliest，这个参数代表没有偏移量可以提交或者 broker 上不存在偏移量的时候，消费者如何处理。earliest 代表从分区的开始位置读取，可能会重复读取消息，但是不会丢失，消费方一般我们肯定要自己保证幂等，另外一种 latest 表示从分区末尾读取，那就会有概率丢失消息。

综合这几个参数设置，我们就能保证消息不会丢失，保证了可靠性。

#### **8. 聊聊副本和它的同步原理吧?**

Kafka 副本的之前提到过，分为 Leader 副本和 Follower 副本，也就是主副本和从副本，和其他的比如 Mysql 不一样的是，Kafka 中只有 Leader 副本会对外提供服务，Follower 副本只是单纯地和 Leader 保持数据同步，作为数据冗余容灾的作用。

在 Kafka 中我们把所有副本的集合统称为 AR（Assigned Replicas），和 Leader 副本保持同步的副本集合称为 ISR（InSyncReplicas）。

ISR 是一个动态的集合，维持这个集合会通过 replica.lag.time.max.ms 参数来控制，这个代表落后 Leader 副本的最长时间，默认值 10 秒，所以只要 Follower 副本没有落后 Leader 副本超过 10 秒以上，就可以认为是和 Leader 同步的（简单可以认为就是同步时间差）。

另外还有两个关键的概念用于副本之间的同步：

**HW（High Watermark）：**高水位，也叫做复制点，表示副本间同步的位置。如下图所示，0~4 绿色表示已经提交的消息，这些消息已经在副本之间进行同步，消费者可以看见这些消息并且进行消费，4~6 黄色的则是表示未提交的消息，可能还没有在副本间同步，这些消息对于消费者是不可见的。

**LEO（Log End Offset）：**下一条待写入消息的位移

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvmGnGiaTMIY8pyhXKqKqKNvzHODxYMSyNVmDdyKibO4biaAGoNlvy05QuBg/640?wx_fmt=jpeg)

副本间同步的过程依赖的就是 HW 和 LEO 的更新，以他们的值变化来演示副本同步消息的过程，绿色表示 Leader 副本，黄色表示 Follower 副本。

首先，生产者不停地向 Leader 写入数据，这时候 Leader 的 LEO 可能已经达到了 10，但是 HW 依然是 0，两个 Follower 向 Leader 请求同步数据，他们的值都是 0。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvmFVSIGGBM45pibdr8lMuLV20K7nhfFS95jHjXVuy9fhcZRcD6yWkrGQA/640?wx_fmt=jpeg)

此时，Follower 再次向 Leader 拉取数据，这时候 Leader 会更新自己的 HW 值，取 Follower 中的最小的 LEO 值来更新。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvmON8tMhJqQA3X5jkfTITqbu1ZVqiaSwLeD7yibe3HJzVnuHQ5bpYDpnsA/640?wx_fmt=jpeg)

之后，Leader 响应自己的 HW 给 Follower，Follower 更新自己的 HW 值，因为又拉取到了消息，所以再次更新 LEO，流程以此类推。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvasdu5HPSTYVr2l1icQNcuSvm69ia4wD7oibCI0UfeEVGcnpgZAocvBvr4eJ2j6z9UjOJhQ7KfYnXGp3g/640?wx_fmt=jpeg)

#### **9.Kafka 为什么快?**

主要是 3 个方面：

**顺序 IO**

kafka 写消息到分区采用追加的方式，也就是顺序写入磁盘，不是随机写入，这个速度比普通的随机 IO 快非常多，几乎可以和网络 IO 的速度相媲美。

**Page Cache 和零拷贝**

kafka 在写入消息数据的时候通过 mmap 内存映射的方式，不是真正立刻写入磁盘，而是利用操作系统的文件缓存 PageCache 异步写入，提高了写入消息的性能，另外在消费消息的时候又通过 sendfile 实现了零拷贝。

**批量处理和压缩**

Kafka 在发送消息的时候不是一条条的发送的，而是会把多条消息合并成一个批次进行处理发送，消费消息也是一个道理，一次拉取一批次的消息进行消费。

并且 Producer、Broker、Consumer 都使用了优化后的压缩算法，发送和消息消息使用压缩节省了网络传输的开销，Broker 存储使用压缩则降低了磁盘存储的空间。

**参考文档**

*   [你一定看得懂学得会记得住的 paxos、raft 相关知识点梳理](https://km.woa.com/articles/show/489381)
    
*   [【MQ Oteam】Apache Kafka 系列文章 - 深入剖析 Kafka Controller (原理、源码与监控)](https://km.woa.com/group/34294/articles/show/475252)
    
*   [总结 Kafka 背后优秀设计](https://km.woa.com/articles/show/505809)
    
*   [2 万字，一文搞懂 Kafka](https://km.woa.com/group/52023/articles/show/501946)
    
*   《深入理解 Kafka：核心设计实践原理》
    
*   [状态机程序设计套路](https://km.woa.com/group/20930/articles/show/368036?kmref=search&from_page=1&no=2)
    
*   [raft 算法源码](https://github.com/wenweihu86/raft-java)
    
*   https://www.bbsmax.com/A/QW5Y3kaBzm/