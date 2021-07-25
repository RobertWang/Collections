> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIzODIzNzE0NQ==&mid=2654440471&idx=1&sn=9743266edd29c114487044de7442a4de&chksm=f2ff48a1c588c1b7f790e6a3aa6c7df169c247cb3f9bcd9a5154663d866f1fb205838acfa365&scene=132#wechat_redirect)

1.  解耦合。
    
2.  异步处理 例如电商平台，秒杀活动。一般流程会分为：1: 风险控制、2：库存锁定、3：生成订单、4：短信通知、5：更新数据。
    
3.  通过消息系统将秒杀活动业务拆分开，将不急需处理的业务放在后面慢慢处理；流程改为：1：风险控制、2：库存锁定、3: 消息系统、4: 生成订单、5：短信通知、6：更新数据。
    
4.  流量的控制 1. 网关在接受到请求后，就把请求放入到消息队列里面 2. 后端的服务从消息队列里面获取到请求，完成后续的秒杀处理流程。然后再给用户返回结果。优点：控制了流量 缺点：会让流程变慢。
    

**-     Kafka 核心概念    -** 

**-     集群架构    -** 

比如，有一个主题 topicA 程序 A 去消费了这个 topicA，那么程序 B 就不能再去消费 topicA（程序 A 和程序 B 属于一个消费组） 再比如程序 A 已经消费了 topicA 里面的数据，现在还是重新再次消费 topicA 的数据，是不可以的，但是重新指定一个 group id 号以后，可以消费。不同消费组之间没有影响。

消费组需自定义，消费者名称程序自动生成（独一无二）。Controller：Kafka 节点里面的一个主节点。

kafka 写数据：顺序写，往磁盘上写数据时，就是追加数据，没有随机写的操作。经验: 如果一个服务器磁盘达到一定的个数，磁盘也达到一定转数，往磁盘里面顺序写（追加写）数据的速度和写内存的速度差不多生产者生产消息，经过 kafka 服务先写到 os cache 内存中，然后经过 sync 顺序写到磁盘上。

**-     零拷贝数据高性能    -** 

消费者读取数据流程：

1.  消费者发送请求给 kafka 服务；
    
2.  kafka 服务去 os cache 缓存读取数据（缓存没有就去磁盘读取数据）；
    
3.  从磁盘读取了数据到 os cache 缓存中；
    
4.  os cache 复制数据到 kafka 应用程序中；
    
5.  kafka 将数据（复制）发送到 socket cache 中；
    
6.  socket cache 通过网卡传输给消费者。
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/2UHIhrbfNBe9IzpCl25bP5KncIjblSQvs7WQY8teLxG6yescF6cmVo4NQUv3UR59uO5GbWsxgOYYCuusYjCxHQ/640?wx_fmt=png)
    

1. 消费者发送请求给 kafka 服务 ；

2.kafka 服务去 os cache 缓存读取数据（缓存没有就去磁盘读取数据） ；

3. 从磁盘读取了数据到 os cache 缓存中 ；

4.os cache 直接将数据发送给网卡 ；

5. 通过网卡将数据传输给消费者。

![](https://mmbiz.qpic.cn/mmbiz_png/2UHIhrbfNBe9IzpCl25bP5KncIjblSQvo9A1L5TZnV3ib1a2ChweawicAZLrpmJYYF6oc0F9Td8Lia8ibUicjhOkxoA/640?wx_fmt=png)

**-     Kafka 日志分段保存    -** 

Kafka 中一个主题，一般会设置分区；比如创建了一个 topic_a，然后创建的时候指定了这个主题有三个分区。其实在三台服务器上，会创建三个目录。服务器 1（kafka1）创建目录 topic_a-0:。

目录下面是我们文件（存储数据），kafka 数据就是 message，数据存储在 log 文件里。.log 结尾的就是日志文件，在 kafka 中把数据文件就叫做日志文件 。一个分区下面默认有 n 多个日志文件（分段存储），一个日志文件默认 1G。

![](https://mmbiz.qpic.cn/mmbiz_png/2UHIhrbfNBe9IzpCl25bP5KncIjblSQvDibcRANHTTAhZyAaGiaDOdubE2icpcorBZIh0V1T7W4M4cLqouU86IYvg/640?wx_fmt=png)

服务器 2（kafka2）：创建目录 topic_a-1: 服务器 3（kafka3）：创建目录 topic_a-2。

**-     二分查找定位数据    -** 

Kafka 里面每一条消息，都有自己的 offset（相对偏移量），存在物理磁盘上面，在 position Position：物理位置（磁盘上面哪个地方）也就是说一条消息就有两个位置：offset：相对偏移量（相对位置）position：磁盘物理位置稀疏索引：         Kafka 中采用了稀疏索引的方式读取索引，kafka 每当写入了 4k 大小的日志（.log），就往 index 里写入一个记录索引。其中会采用二分查找：  

![](https://mmbiz.qpic.cn/mmbiz_png/2UHIhrbfNBe9IzpCl25bP5KncIjblSQvAIXbQPRaeia7UVgEu0gg27TbVuqLtAdYUDicIsH7kayLmbEeroyun6IQ/640?wx_fmt=png)

**-     高并发网络设计 NIO    -** 

网络设计部分是 kafka 中设计最好的一个部分，这也是保证 Kafka 高并发、高性能的原因，对 kafka 进行调优，就得对 kafka 原理比较了解，尤其是网络设计部分。

Reactor 网络设计模式 1：

![](https://mmbiz.qpic.cn/mmbiz_png/2UHIhrbfNBe9IzpCl25bP5KncIjblSQvmeBsSCRn0UwaDzRYG8oG9ITkZrcib9usIoPFjpO7qKN0JIN7L1RfZTQ/640?wx_fmt=png)

Reactor 网络设计模式 2：

![](https://mmbiz.qpic.cn/mmbiz_png/2UHIhrbfNBe9IzpCl25bP5KncIjblSQvjbiac3YVN02IY3OqTIqLicu7R3ribs71zkaDeiciaR0X0HUqXhqqSUeBUgw/640?wx_fmt=png)

Reactor 网络设计模式 3：

![](https://mmbiz.qpic.cn/mmbiz_png/2UHIhrbfNBe9IzpCl25bP5KncIjblSQvibT40CldTzmwpXtib2WCrCdFs27qa8v8AbF4ictLkOkKIu4yhuEWnJ5cg/640?wx_fmt=png)

Kafka 超高并发网络设计：

![](https://mmbiz.qpic.cn/mmbiz_png/2UHIhrbfNBe9IzpCl25bP5KncIjblSQvwwmAbH8cr3cCicCFLParhbibwicXlNKToN65LuTs7I0x0t4icUSIpHPryA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/2UHIhrbfNBe9IzpCl25bP5KncIjblSQvicMVicZm3gKl6cGUziaF5hDYOTfuQCXj84bUuNwt11tNUXOGpVX8AcZmw/640?wx_fmt=png)

**-     Kafka 冗余副本保证高可用    -** 

在 kafka 里面分区是有副本的，注：0.8 以前是没有副本机制的。创建主题时，可以指定分区，也可以指定副本个数。副本是有角色的：leader partition：1、写数据、读数据操作都是从 leader partition 去操作的。

  
它会维护一个 ISR（in-sync- replica ）列表，但是会根据一定的规则删除 ISR 列表里面的值 生产者发送来一个消息，消息首先要写入到 leader partition 中 写完了以后，还要把消息写入到 ISR 列表里面的其它分区，写完后才算这个消息提交 follower partition：从 leader partition 同步数据。

**-     优秀架构思考    -** 

Kafka — 高并发、高可用、高性能 高可用：多副本机制 高并发：网络架构设计 三层架构：多 selector -> 多线程 -> 队列的设计（NIO） 高性能：写数据：

1.  把数据先写入到 OS Cache
    
2.  写到磁盘上面是顺序写，性能很高
    

读数据：

1.  根据稀疏索引，快速定位到要消费的数据
    
2.  零拷贝机制 减少数据的拷贝 减少了应用程序与操作系统上下文切换
    

**-     Kafka 生产环境搭建    -** 

### **需求场景分析**

> 电商平台，需要每天 10 亿请求都要发送到 Kafka 集群上面。二八反正，一般评估出来问题都不大。10 亿请求 -> 24 过来的，一般情况下，每天的 12:00 到早上 8:00 这段时间其实是没有多大的数据量的。80% 的请求是用的另外 16 小时的处理的。16 个小时处理 -> 8 亿的请求。16 * 0.2 = 3 个小时 处理了 8 亿请求的 80% 的数据。

也就是说 6 亿的数据是靠 3 个小时处理完的。我们简单的算一下高峰期时候的 qps6 亿 / 3 小时 =5.5 万 / s qps=5.5 万。

> 10 亿请求 * 50kb = 46T 每天需要存储 46T 的数据。

一般情况下，我们都会设置两个副本 46T * 2 = 92T  Kafka 里面的数据是有保留的时间周期，保留最近 3 天的数据。92T * 3 天 = 276T 我这儿说的是 50kb 不是说一条消息就是 50kb 不是（把日志合并了，多条日志合并在一起），通常情况下，一条消息就几 b，也有可能就是几百字节。

**-     物理机数量评估    -** 

（1）首先分析一下是需要虚拟机还是物理机 像 Kafka mysql hadoop 这些集群搭建的时候，我们生产里面都是使用物理机。

（2）高峰期需要处理的请求总的请求每秒 5.5 万个，其实一两台物理机绝对是可以抗住的。一般情况下，我们评估机器的时候，是按照高峰期的 4 倍的去评估。如果是 4 倍的话，大概我们集群的能力要准备到 20 万 qps。这样子的集群才是比较安全的集群。大概就需要 5 台物理机。每台承受 4 万请求。

场景总结：搞定 10 亿请求，高峰期 5.5 万的 qps,276T 的数据，需要 5 台物理机。

**-     磁盘选择    -** 

搞定 10 亿请求，高峰期 5.5 万的 qps,276T 的数据，需要 5 台物理机。

（1）SSD 固态硬盘，还是需要普通的机械硬盘 SSD 硬盘：性能比较好，但是价格贵 SAS 盘：某方面性能不是很好，但是比较便宜。SSD 硬盘性能比较好，指的是它随机读写的性能比较好。适合 MySQL 这样集群。但是其实他的顺序写的性能跟 SAS 盘差不多。

kafka 的理解：就是用的顺序写。所以我们就用普通的【机械硬盘】就可以了。

（2）需要我们评估每台服务器需要多少块磁盘 5 台服务器，一共需要 276T ，大约每台服务器 需要存储 60T 的数据。我们公司里面服务器的配置用的是 11 块硬盘，每个硬盘 7T。11 * 7T = 77T。

> 77T * 5 台服务器 = 385T。

**场景总结：**

搞定 10 亿请求，需要 5 台物理机，11（SAS） * 7T。

**-     内存评估    -** 

搞定 10 亿请求，需要 5 台物理机，11（SAS） * 7T。

我们发现 kafka 读写数据的流程 都是基于 os cache, 换句话说假设咱们的 os cashe 无限大那么整个 kafka 是不是相当于就是基于内存去操作，如果是基于内存去操作，性能肯定很好。内存是有限的。

（1）尽可能多的内存资源要给 os cache。

（2）Kafka 的代码用 核心的代码用的是 scala 写的，客户端的代码 java 写的。都是基于 jvm。所以我们还要给一部分的内存给 jvm。Kafka 的设计，没有把很多数据结构都放在 jvm 里面。所以我们的这个 jvm 不需要太大的内存。根据经验，给个 10G 就可以了。

> NameNode: jvm 里面还放了元数据（几十 G），JVM 一定要给得很大。比如给个 100G。

假设我们这个 10 请求的这个项目，一共会有 100 个 topic。100 topic * 5 partition * 2 = 1000 partition 一个 partition 其实就是物理机上面的一个目录，这个目录下面会有很多个. log 的文件。

.log 就是存储数据文件，默认情况下一个. log 文件的大小是 1G。我们如果要保证 1000 个 partition 的最新的. log 文件的数据 如果都在内存里面，这个时候性能就是最好。1000 * 1G = 1000G 内存. 我们只需要把当前最新的这个 log 保证里面的 25% 的最新的数据在内存里面。250M * 1000 = 0.25 G* 1000 =250G 的内存。

250 内存 / 5 = 50G 内存 50G+10G = 60G 内存。

64G 的内存，另外的 4G，操作系统本生是不是也需要内存。其实 Kafka 的 jvm 也可以不用给到 10G 这么多。评估出来 64G 是可以的。当然如果能给到 128G 的内存的服务器，那就最好。

我刚刚评估的时候用的都是一个 topic 是 5 个 partition，但是如果是数据量比较大的 topic，可能会有 10 个 partition。

总结：搞定 10 亿请求，需要 5 台物理机，11（SAS） * 7T ，需要 64G 的内存（128G 更好）

**-     CPU 压力评估    -** 

评估一下每台服务器需要多少 cpu core(资源很有限)。

我们评估需要多少个 cpu ，依据就是看我们的服务里面有多少线程去跑。线程就是依托 cpu 去运行的。如果我们的线程比较多，但是 cpu core 比较少，这样的话，我们的机器负载就会很高，性能不就不好。

评估一下，kafka 的一台服务器 启动以后会有多少线程？

Acceptor 线程 1 processor 线程 3 6~9 个线程 处理请求线程 8 个 32 个线程 定时清理的线程，拉取数据的线程，定时检查 ISR 列表的机制 等等。所以大概一个 Kafka 的服务启动起来以后，会有一百多个线程。

cpu core = 4 个，一遍来说，几十个线程，就肯定把 cpu 打满了。cpu core = 8 个，应该很轻松的能支持几十个线程。如果我们的线程是 100 多个，或者差不多 200 个，那么 8 个 cpu core 是搞不定的。所以我们这儿建议：CPU core = 16 个。如果可以的话，能有 32 个 cpu core 那就最好。

结论：kafka 集群，最低也要给 16 个 cpu core，如果能给到 32 cpu core 那就更好。2cpu * 8 =16 cpu core 4cpu * 8 = 32 cpu core。

总结：搞定 10 亿请求，需要 5 台物理机，11（SAS） * 7T ，需要 64G 的内存（128G 更好），需要 16 个 cpu core（32 个更好）。

**-     网络需求评估    -** 

评估我们需要什么样网卡？一般要么是千兆的网卡（1G/s），还有的就是万兆的网卡（10G/s）。

```
高峰期的时候 每秒会有5.5万的请求涌入，5.5/5 = 大约是每台服务器会有1万个请求涌入。
我们之前说的，
10000 * 50kb = 488M  也就是每条服务器，每秒要接受488M的数据。数据还要有副本，副本之间的同步
也是走的网络的请求。488 * 2 = 976m/s
说明一下：
   很多公司的数据，一个请求里面是没有50kb这么大的，我们公司是因为主机在生产端封装了数据
   然后把多条数据合并在一起了，所以我们的一个请求才会有这么大。
   
说明一下：
   一般情况下，网卡的带宽是达不到极限的，如果是千兆的网卡，我们能用的一般就是700M左右。
   但是如果最好的情况，我们还是使用万兆的网卡。
   如果使用的是万兆的，那就是很轻松。
```

```
-     集群规划    - 
```

请求量规划物理机的个数 分析磁盘的个数，选择使用什么样的磁盘 内存 cpu core 网卡就是告诉大家，**以后要是公司里面有什么需求，进行资源的评估，服务器的评估，大家按照我的思路去评估**：

一条消息的大小 50kb -> 1kb 500byte 1Mip 主机名 192.168.0.100 hadoop1 192.168.0.101 hadoop2 192.168.0.102 hadoop3。

主机的规划：kafka 集群架构的时候：主从式的架构：controller -> 通过 zk 集群来管理整个集群的元数据。

1.  zookeeper 集群 hadoop1 hadoop2 hadoop3；
    
2.  kafka 集群 理论上来讲，我们不应该把 kafka 的服务于 zk 的服务安装在一起。但是我们这儿服务器有限。所以我们 kafka 集群也是安装在 hadoop1 haadoop2 hadoop3。
    

**-     Kafka 运维工具与命令    -** 

KafkaManager — 页面管理工具。

**场景一：topic 数据量太大，要增加 topic 数。**

一开始创建主题的时候，数据量不大，给的分区数不多。

```
kafka-topics.sh --create --zookeeper hadoop1:2181,hadoop2:2181,hadoop3:2181 --replication-factor 1 --partitions 1 --topic test6
kafka-topics.sh --alter --zookeeper hadoop1:2181,hadoop2:2181,ha
```

broker id：

hadoop1:0 hadoop2:1 hadoop3:2 假设一个 partition 有三个副本：partition0：a,b,c

a：leader partition b，c:follower partition

ISR:{a,b,c} 如果一个 follower 分区 超过 10 秒 没有向 leader partition 去拉取数据，那么这个分区就从 ISR 列表里面移除。

**场景二：核心 topic 增加副本因子**

如果对核心业务数据需要增加副本因子 vim test.json 脚本，将下面一行 json 脚本保存：

```
{“version”:1,“partitions”:[{“topic”:“test6”,“partition”:0,“replicas”:[0,1,2]},{“topic”:“test6”,“partition”:1,“replicas”:[0,1,2]},{“topic”:“test6”,“partition”:2,“replicas”:[0,1,2]}]}
```

执行上面 json 脚本：

```
kafka-reassign-partitions.sh --zookeeper hadoop1:2181,hadoop2:2181,hadoop3:2181 --reassignment-json-file test.json --execute
```

**场景三：负载不均衡的 topic，手动迁移 vi topics-to-move.json**

```
{“topics”: [{“topic”: “test01”}, {“topic”: “test02”}], “version”: 1} // 把你所有的topic都写在这里
```

```
kafka-reassgin-partitions.sh --zookeeper hadoop1:2181,hadoop2:2181,hadoop3:2181 --topics-to-move-json-file topics-to-move.json --broker-list “5,6” --generate
```

把你所有的包括新加入的 broker 机器都写在这里，就会说是把所有的 partition 均匀的分散在各个 broker 上，包括新进来的 broker 此时会生成一个迁移方案，可以保存到一个文件里去：expand-cluster-reassignment.json

```
kafka-reassign-partitions.sh --zookeeper hadoop01:2181,hadoop02:2181,hadoop03:2181 --reassignment-json-file expand-cluster-reassignment.json --execute

kafka-reassign-partitions.sh --zookeeper hadoop01:2181,hadoop02:2181,hadoop03:2181 --reassignment-json-file expand-cluster-reassignment.json --verify
```

这种数据迁移操作一定要在晚上低峰的时候来做，因为他会在机器之间迁移数据，非常的占用带宽资源–generate: 根据给予的 Topic 列表和 Broker 列表生成迁移计划。generate 并不会真正进行消息迁移，而是将消息迁移计划计算出来，供 execute 命令使用。–execute: 根据给予的消息迁移计划进行迁移。–verify: 检查消息是否已经迁移完成。

**场景四：如果某个 broker leader partition 过多**

正常情况下，我们的 leader partition 在服务器之间是负载均衡。hadoop1 4 hadoop2 1 hadoop3 1。

现在各个业务方可以自行申请创建 topic，分区数量都是自动分配和后续动态调整的， kafka 本身会自动把 leader partition 均匀分散在各个机器上，这样可以保证每台机器的读写吞吐量都是均匀的，但是也有例外。

那就是如果某些 broker 宕机，会导致 leader partition 过于集中在其他少部分几台 broker 上， 这会导致少数几台 broker 的读写请求压力过高，其他宕机的 broker 重启之后都是 folloer partition，读写请求很低，造成集群负载不均衡有一个参数，auto.leader.rebalance.enable。

默认是 true，每隔 300 秒（leader.imbalance.check.interval.seconds）检查 leader 负载是否平衡 如果一台 broker 上的不均衡的 leader 超过了 10%，leader.imbalance.per.broker.percentage， 就会对这个 broker 进行选举 配置参数：auto.leader.rebalance.enable 默认是 true leader.imbalance.per.broker.percentage: 每个 broker 允许的不平衡的 leader 的比率。如果每个 broker 超过了这个值，控制器会触发 leader 的平衡。

这个值表示百分比。10% leader.imbalance.check.interval.seconds：默认值 300 秒。

**-     Kafka 生产者发消息原理    -** 

![](https://mmbiz.qpic.cn/mmbiz_png/2UHIhrbfNBe9IzpCl25bP5KncIjblSQvvb23Qy7eiclIV932hlSLxhJicvUtTicWqp1BI3nmQR2xLcEvS9tpCPibyg/640?wx_fmt=png)

### 生产者发送消息原理—基础案例演示：

![](https://mmbiz.qpic.cn/mmbiz_png/2UHIhrbfNBe9IzpCl25bP5KncIjblSQvlsibfKUHDXtNrtHKk7AasBibrFTqcaOiblxPRDZxHwhCdgicJia3TaYL2jQ/640?wx_fmt=png)

**-     如何提升吞吐量    -** 

如何提升吞吐量：参数一：buffer.memory：设置发送消息的缓冲区，默认值是 33554432，就是 32MB 参数二：compression.type：默认是 none，不压缩，但是也可以使用 lz4 压缩，效率还是不错的，压缩之后可以减小数据量，提升吞吐量，但是会加大 producer 端的 cpu 开销 参数三：batch.size：设置 batch 的大小，如果 batch 太小，会导致频繁网络请求，吞吐量下降。

如果 batch 太大，会导致一条消息需要等待很久才能被发送出去，而且会让内存缓冲区有很大压力，过多数据缓冲在内存里，默认值是：16384，就是 16kb，也就是一个 batch 满了 16kb 就发送出去，一般在实际生产环境，这个 batch 的值可以增大一些来提升吞吐量，如果一个批次设置大了，会有延迟。

一般根据一条消息大小来设置。如果我们消息比较少。配合使用的参数 linger.ms，这个值默认是 0，意思就是消息必须立即被发送，但是这是不对的，一般设置一个 100 毫秒之类的，这样的话就是说，这个消息被发送出去后进入一个 batch，如果 100 毫秒内，这个 batch 满了 16kb，自然就会发送出去。

**-     如何处理异常    -** 

1.  LeaderNotAvailableException：这个就是如果某台机器挂了，此时 leader 副本不可用，会导致你写入失败，要等待其他 follower 副本切换为 leader 副本之后，才能继续写入，此时可以重试发送即可；如果说你平时重启 kafka 的 broker 进程，肯定会导致 leader 切换，一定会导致你写入报错，是 LeaderNotAvailableException。
    
2.  NotControllerException：这个也是同理，如果说 Controller 所在 Broker 挂了，那么此时会有问题，需要等待 Controller 重新选举，此时也是一样就是重试即可。
    
3.  NetworkException：网络异常 timeout a. 配置 retries 参数，他会自动重试的 b. 但是如果重试几次之后还是不行，就会提供 Exception 给我们来处理了, 我们获取到异常以后，再对这个消息进行单独处理。我们会有备用的链路。发送不成功的消息发送到 Redis 或者写到文件系统中，甚至是丢弃。
    

**-     重试机制    -** 

重试会带来一些问题：

1.  消息会重复有的时候一些 leader 切换之类的问题，需要进行重试，设置 retries 即可，但是消息重试会导致, 重复发送的问题，比如说网络抖动一下导致他以为没成功，就重试了，其实人家都成功了。
    
2.  消息乱序消息重试是可能导致消息的乱序的，因为可能排在你后面的消息都发送出去了。所以可以使用 "max.in.flight.requests.per.connection" 参数设置为 1， 这样可以保证 producer 同一时间只能发送一条消息。两次重试的间隔默认是 100 毫秒，用 "retry.backoff.ms" 来进行设置 基本上在开发过程中，靠重试机制基本就可以搞定 95% 的异常问题。
    

**-     ACK 参数详情    -** 

producer 端设置的 request.required.acks=0；只要请求已发送出去，就算是发送完了，不关心有没有写成功。性能很好，如果是对一些日志进行分析，可以承受丢数据的情况，用这个参数，性能会很好。request.required.acks=1；发送一条消息，当 leader partition 写入成功以后，才算写入成功。

  
不过这种方式也有丢数据的可能。request.required.acks=-1；需要 ISR 列表里面，所有副本都写完以后，这条消息才算写入成功。ISR：1 个副本。1 leader partition 1 follower partition kafka 服务端：min.insync.replicas：1， 如果我们不设置的话，默认这个值是 1 一个 leader partition 会维护一个 ISR 列表，这个值就是限制 ISR 列表里面 至少得有几个副本，比如这个值是 2，那么当 ISR 列表里面只有一个副本的时候。

往这个分区插入数据的时候会报错。设计一个不丢数据的方案：数据不丢失的方案：1) 分区副本 >=2 2)acks = -1 3)min.insync.replicas >=2 还有可能就是发送有异常：对异常进行处理。

**-     自定义分区    -** 

分区：1、没有设置 key 我们的消息就会被轮训的发送到不同的分区。2、设置了 keykafka 自带的分区器，会根据 key 计算出来一个 hash 值，这个 hash 值会对应某一个分区。如果 key 相同的，那么 hash 值必然相同，key 相同的值，必然是会被发送到同一个分区。但是有些比较特殊的时候，我们就需要自定义分区了：

```
public class HotDataPartitioner implements Partitioner {
private Random random;
@Override
public void configure(Map<String, ?> configs) {
random = new Random();
}
@Override
public int partition(String topic, Object keyObj, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
String key = (String)keyObj;
List partitionInfoList = cluster.availablePartitionsForTopic(topic);
//获取到分区的个数 0,1，2
int partitionCount = partitionInfoList.size();
//最后一个分区
int hotDataPartition = partitionCount - 1;
return !key.contains(“hot_data”) ? random.nextInt(partitionCount - 1) : hotDataPartition;
}
}
```

如何使用：配置上这个类即可：props.put(”partitioner.class”, “com.zhss.HotDataPartitioner”);

**-     综合案例演示    -** 

消费组概念：groupid 相同就属于同一个消费组。

（1）每个 consumer 都要属于一个 consumer.group，就是一个消费组，topic 的一个分区只会分配给 一个消费组下的一个 consumer 来处理，每个 consumer 可能会分配多个分区，也有可能某个 consumer 没有分配到任何分区。

（2）如果想要实现一个广播的效果，那只需要使用不同的 group id 去消费就可以。topicA: partition0、partition1 groupA：consumer1: 消费 partition0 consuemr2: 消费 partition1 consuemr3: 消费不到数据 groupB: consuemr3: 消费到 partition0 和 partition1 3）如果 consumer group 中某个消费者挂了，此时会自动把分配给他的分区交给其他的消费者，如果他又重启了，那么又会把一些分区重新交还给他。

**-     Kafka 消费组概念    -** 

groupid 相同就属于同一个消费组。

（1）每个 consumer 都要属于一个 consumer.group，就是一个消费组，topic 的一个分区只会分配给 一个消费组下的一个 consumer 来处理，每个 consumer 可能会分配多个分区，也有可能某个 consumer 没有分配到任何分区。

（2）如果想要实现一个广播的效果，那只需要使用不同的 group id 去消费就可以。topicA: partition0、partition1 groupA：consumer1: 消费 partition0 consuemr2: 消费 partition1 consuemr3: 消费不到数据 groupB: consuemr3: 消费到 partition0 和 partition1 3）如果 consumer group 中某个消费者挂了，此时会自动把分配给他的分区交给其他的消费者，如果他又重启了，那么又会把一些分区重新交还给他。

基础案例演示：

![](https://mmbiz.qpic.cn/mmbiz_png/2UHIhrbfNBe9IzpCl25bP5KncIjblSQvAVoslQARw7xrhKSSB82hB8jwGicrPojBkHUIJ3b8MroWVTbtteIQVLw/640?wx_fmt=png)

**-     偏移量管理    -** 

1.  每个 consumer 内存里数据结构保存对每个 topic 的每个分区的消费 offset，定期会提交 offset，老版本是写入 zk，但是那样高并发请求 zk 是不合理的架构设计，zk 是做分布式系统的协调的，轻量级的元数据存储，不能负责高并发读写，作为数据存储。
    
2.  现在新的版本提交 offset 发送给 kafka 内部 topic：__consumer_offsets，提交过去的时候， key 是 group.id+topic + 分区号，value 就是当前 offset 的值，每隔一段时间，kafka 内部会对这个 topic 进行 compact(合并)，也就是每个 group.id+topic + 分区号就保留最新数据。
    
3.  __consumer_offsets 可能会接收高并发的请求，所以默认分区 50 个 (leader partitiron -> 50 kafka)，这样如果你的 kafka 部署了一个大的集群，比如有 50 台机器，就可以用 50 台机器来抗 offset 提交的请求压力. 消费者 -> broker 端的数据 message -> 磁盘 -> offset 顺序递增 从哪儿开始消费？-> offset 消费者（offset）。
    

**-     偏移量监控工具介绍    -** 

1.  web 页面管理的一个管理软件 (kafka Manager) 修改 bin/kafka-run-class.sh 脚本，第一行增加 JMX_PORT=9988 重启 kafka 进程。
    
2.  另一个软件：主要监控的 consumer 的偏移量。就是一个 jar 包 java -cp KafkaOffsetMonitor-assembly-0.3.0-SNAPSHOT.jar com.quantifind.kafka.offsetapp.OffsetGetterWeb –offsetStorage kafka \（根据版本：偏移量存在 kafka 就填 kafka，存在 zookeeper 就填 zookeeper） –zk hadoop1:2181 –port 9004 –refresh 15.seconds –retain 2.days。
    

**-     消费异常感知    -** 

heartbeat.interval.ms：consumer 心跳时间间隔，必须得与 coordinator 保持心跳才能知道 consumer 是否故障了， 然后如果故障之后，就会通过心跳下发 rebalance 的指令给其他的 consumer 通知他们进行 rebalance 的操作 session.timeout.ms：kafka 多长时间感知不到一个 consumer 就认为他故障了，默认是 10 秒 max.poll.interval.ms：如果在两次 poll 操作之间，超过了这个时间，那么就会认为这个 consume 处理能力太弱了，会被踢出消费组，分区分配给别人去消费，一般来说结合业务处理的性能来设置就可以了。

fetch.max.bytes：获取一条消息最大的字节数，一般建议设置大一些，默认是 1M 其实我们在之前多个地方都见到过这个类似的参数，意思就是说一条信息最大能多大？

1.  Producer 发送的数据，一条消息最大多大， -> 10M。
    
2.  Broker 存储数据，一条消息最大能接受多大 -> 10M。
    
3.  Consumer max.poll.records: 一次 poll 返回消息的最大条数，默认是 500 条 connection.max.idle.ms：consumer 跟 broker 的 socket 连接如果空闲超过了一定的时间，此时就会自动回收连接，但是下次消费就要重新建立 socket 连接，这个建议设置为 - 1，不要去回收 enable.auto.commit: 开启自动提交偏移量 auto.commit.interval.ms: 每隔多久提交一次偏移量，默认值 5000 毫秒 _consumer_offset auto.offset.reset：earliest 当各分区下有已提交的 offset 时，从提交的 offset 开始消费；无提交的 offset 时，从头开始消费 topica -> partition0:1000 partitino1:2000 latest 当各分区下有已提交的 offset 时，从提交的 offset 开始消费；无提交的 offset 时，消费新产生的该分区下的数据 none topic 各分区都存在已提交的 offset 时，从 offset 后开始消费；只要有一个分区不存在已提交的 offset，则抛出异常。
    

  

**-     综合案例演示    -** 

引入案例：二手电商平台（欢乐送），根据用户消费的金额，对用户星星进行累计。订单系统（生产者） -> Kafka 集群里面发送了消息。会员系统（消费者） -> Kafak 集群里面消费消息，对消息进行处理。

### **group coordinator 原理**

面试题：消费者是如何实现 rebalance 的？— 根据 coordinator 实现：

1.  什么是 coordinator 每个 consumer group 都会选择一个 broker 作为自己的 coordinator，他是负责监控这个消费组里的各个消费者的心跳，以及判断是否宕机，然后开启 rebalance 的。
    
2.  如何选择 coordinator 机器 首先对 groupId 进行 hash（数字），接着对__consumer_offsets 的分区数量取模，默认是 50，_consumer_offsets 的分区数可以通过 offsets.topic.num.partitions 来设置，找到分区以后，这个分区所在的 broker 机器就是 coordinator 机器。比如说：groupId，“myconsumer_group” -> hash 值（数字）-> 对 50 取模 -> 8 __consumer_offsets 这个主题的 8 号分区在哪台 broker 上面，那一台就是 coordinator 就知道这个 consumer group 下的所有的消费者提交 offset 的时候是往哪个分区去提交 offset。
    
3.  运行流程（1）每个 consumer 都发送 JoinGroup 请求到 Coordinator，（ 2）然后 Coordinator 从一个 consumer group 中选择一个 consumer 作为 leader，（3）把 consumer group 情况发送给这个 leader，（4）接着这个 leader 会负责制定消费方案，（5）通过 SyncGroup 发给 Coordinator 6）接着 Coordinator 就把消费方案下发给各个 consumer，他们会从指定的分区的 leader broker 开始进行 socket 连接以及消费消息。
    

![](https://mmbiz.qpic.cn/mmbiz_png/2UHIhrbfNBe9IzpCl25bP5KncIjblSQvQgOYtlhdqCibsKCaQI3vZSznLVhpEHvbxBe6NldnAjaZAMwaYf8K3NQ/640?wx_fmt=png)

**-     Rebalance 策略    -** 

consumer group 靠 coordinator 实现了 Rebalance。

这里有三种 rebalance 的策略：range、round-robin、sticky。

比如我们消费的一个主题有 12 个分区：p0,p1,p2,p3,p4,p5,p6,p7,p8,p9,p10,p11 假设我们的消费者组里面有三个消费者：

1.  range 策略 range 策略就是按照 partiton 的序号范围 p0~3 consumer1 p4~7 consumer2 p8~11 consumer3 默认就是这个策略；
    
2.  round-robin 策略 就是轮询分配 consumer1:0,3,6,9 consumer2:1,4,7,10 consumer3:2,5,8,11 但是前面的这两个方案有个问题：12 -> 2 每个消费者会消费 6 个分区。假设 consuemr1 挂了: p0-5 分配给 consumer2,p6-11 分配给 consumer3 这样的话，原本在 consumer2 上的的 p6,p7 分区就被分配到了 consumer3 上。
    
3.  sticky 策略 最新的一个 sticky 策略，就是说尽可能保证在 rebalance 的时候，让原本属于这个 consumer 的分区还是属于他们，然后把多余的分区再均匀分配过去，这样尽可能维持原来的分区分配的策略。
    

consumer1：0-3 consumer2: 4-7 consumer3: 8-11 假设 consumer3 挂了 consumer1：0-3，+8,9 consumer2: 4-7，+10,11。

**-     Broker 管理    -** 

Leo、hw 含义：

1.  Kafka 的核心原理
    
2.  如何去评估一个集群资源
    
3.  搭建了一套 kafka 集群 -》 介绍了简单的一些运维管理的操作。
    
4.  生产者（使用，核心的参数）
    
5.  消费者（原理，使用的，核心参数）
    
6.  broker 内部的一些原理
    

核心的概念：LEO，HW LEO：是跟 offset 偏移量有关系。

LEO：在 kafka 里面，无论 leader partition 还是 follower partition 统一都称作副本（replica）。

> 每次 partition 接收到一条消息，都会更新自己的 LEO，也就是 log end offset，LEO 其实就是最新的 offset + 1

HW：高水位 LEO 有一个很重要的功能就是更新 HW，如果 follower 和 leader 的 LEO 同步了，此时 HW 就可以更新 HW 之前的数据对消费者是可见，消息属于 commit 状态。HW 之后的消息消费者消费不到。

### Leo 更新：

![](https://mmbiz.qpic.cn/mmbiz_png/2UHIhrbfNBe9IzpCl25bP5KncIjblSQvicZe9Y5iaa1miaawXR0ohY4aXA8l4BBuRQSF8N9ocCdmnNkDpgiaVqLiaibQ/640?wx_fmt=png)

### hw 更新：

![](https://mmbiz.qpic.cn/mmbiz_png/2UHIhrbfNBe9IzpCl25bP5KncIjblSQvx8ibCBKr0pTZhXwH58VaVAbsS22eNy7oMSjVRTfrAgzevM9NHZO3yUw/640?wx_fmt=png)

### **controller 如何管理整个集群**

1: 竞争 controller 的 /controller/id 2：controller 服务监听的目录：/broker/ids/ 用来感知 broker 上下线 /broker/topics/ 创建主题，我们当时创建主题命令，提供的参数，ZK 地址。/admin/reassign_partitions 分区重分配……

  
[![](https://mmbiz.qpic.cn/mmbiz_png/2UHIhrbfNBe9IzpCl25bP5KncIjblSQvlG5CaVST7aMWPkFibokRNVfzU3ntkojd4I8MfrNTQ0zcLgpKQmWvN5w/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=MzIxMTE0ODU5NQ==&mid=2650245169&idx=2&sn=54ffaab18e546e714519880a141627c0&chksm=8f5ae26db82d6b7bf2b51ea15938ce8623f4593cf228e801c0920b1dda4a01b29da0b1956118&scene=21#wechat_redirect)

### **延时任务**

kafka 的延迟调度机制（扩展知识） 我们先看一下 kafka 里面哪些地方需要有任务要进行延迟调度。第一类延时的任务：比如说 producer 的 acks=-1，必须等待 leader 和 follower 都写完才能返回响应。

有一个超时时间，默认是 30 秒（request.timeout.ms）。所以需要在写入一条数据到 leader 磁盘之后，就必须有一个延时任务，到期时间是 30 秒延时任务 放到 DelayedOperationPurgatory（延时管理器）中。

假如在 30 秒之前如果所有 follower 都写入副本到本地磁盘了，那么这个任务就会被自动触发苏醒，就可以返回响应结果给客户端了， 否则的话，这个延时任务自己指定了最多是 30 秒到期，如果到了超时时间都没等到，就直接超时返回异常。

第二类延时的任务：follower 往 leader 拉取消息的时候，如果发现是空的，此时会创建一个延时拉取任务 延时时间到了之后（比如到了 100ms），就给 follower 返回一个空的数据，然后 follower 再次发送请求读取消息， 但是如果延时的过程中 (还没到 100ms)，leader 写入了消息，这个任务就会自动苏醒，自动执行拉取任务。

海量的延时任务，需要去调度。

**-     时间轮机制    -** 

1.  什么会有要设计时间轮？Kafka 内部有很多延时任务，没有基于 JDK Timer 来实现，那个插入和删除任务的时间复杂度是 O(nlogn)， 而是基于了自己写的时间轮来实现的，时间复杂度是 O(1)，依靠时间轮机制，延时任务插入和删除，O(1)。
    
2.  时间轮是什么？其实时间轮说白其实就是一个数组。tickMs: 时间轮间隔 1ms wheelSize：时间轮大小 20 interval：timckMS * whellSize，一个时间轮的总的时间跨度。20ms currentTime：当时时间的指针。a: 因为时间轮是一个数组，所以要获取里面数据的时候，靠的是 index，时间复杂度是 O(1) b: 数组某个位置上对应的任务，用的是双向链表存储的，往双向链表里面插入，删除任务，时间复杂度也是 O（1） 举例：插入一个 8ms 以后要执行的任务 19ms 3. 多层级的时间轮 比如：要插入一个 110 毫秒以后运行的任务。tickMs: 时间轮间隔 20ms wheelSize：时间轮大小 20 interval：timckMS * whellSize，一个时间轮的总的时间跨度。20ms currentTime：当时时间的指针。第一层时间轮：1ms * 20 第二层时间轮：20ms * 20 第三层时间轮：400ms * 20。
    

  
![](https://mmbiz.qpic.cn/mmbiz_png/2UHIhrbfNBe9IzpCl25bP5KncIjblSQvXIYveZRuQ8FpAicyBG8LlMEbIpSLFcia4ty6URWb4eBDy4KuN7hNRVOA/640?wx_fmt=png)

> 作者：erainm
> 
> 来源：
> 
> https://blog.csdn.net/eraining/article/details/115860664