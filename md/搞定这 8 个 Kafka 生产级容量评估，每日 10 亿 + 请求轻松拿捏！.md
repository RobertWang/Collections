> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzkwOTIxNDQ3OA==&mid=2247572928&idx=1&sn=be43ccc45d1ac98f5a5bf6c098d3501a&chksm=c13d8ba1f64a02b77ad162005c27ad9558df579ffecb00e282873da2f6f1648321a64ab23b9e&scene=132#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_jpg/ufWcjcomw8Zicoe8pJRtKkBFeL2ib8P4MAibVJdjgN1yibDqLD3HNtSBjwVjdk9hxeT8JVSb3VwBia2ew1WxBdF6OcA/640?wx_fmt=jpeg)

本篇文章通过场景驱动的方式来深度剖析 Kafka 生产级容量评估方案如何分析, 申请和实施。

**一、kafka 容量评估需求场景分析**

**1、集群如何每天 hold 住 10 亿 + 请求**

拿电商平台为例，kafka 集群每天需要承载 10 亿 + 请求流量数据，一天 24 小时，对于平台来说，晚上 12 点到凌晨 8 点这 8 个小时几乎没多少数据涌入的。这里我们使用**「二八法则」**来进行预估，也就是 80% 的数据（8 亿）会在剩余的 16 个小时涌入，且 8 亿中的 80% 的数据（约 6.4 亿）会在这 16 个小时的 20% 时间 （约 3 小时）涌入。

通过上面的场景分析，可以得出如下：

**QPS 计算公式 = 640000000 ÷ (3 * 60 * 60) = 6 万，**也就是说高峰期集群需要扛住每秒 6 万的并发请求。

假设**每条数据平均按 20kb(生产端有数据汇总)** 来算, 那就是 **1000000000 * 20kb = 18T，**一般情况下我们都会设置 **3 个副本，**即 **54T，**另外 kafka 数据是有保留时间周期的,  一般情况是保留**最近 3 天的数据，**即 **54T * 3 = 162T。**

**2、场景总结**

要搞定 **10 亿 +** 请求，高峰期要支撑 **6 万 QPS，**需要大约 **162T** 的存储空间。

**二、kafka 容量评估之物理机数量**

**1、物理机 OR 虚拟机**

 一般对于 Kafka，Mysql，Hadoop 等集群自建的时候，都会使用物理机来进行搭建，性能和稳定性相对虚拟机要强很多。

**2、物理机数量计算**

在**第一步**中我们分析得出**系统高峰期**的时候要支撑 **6 万 QPS，**如果公司资金和资源充足的情况下，我们一般会让高峰期的 QPS 控制在集群总承载 QPS 能力的 **30% 左右，**这样的话可以得出集群能承载的**总 QPS 能力约为 20 万左右，**这样系统才会是安全的。

**3、场景总结**

根据经验可以得出**每台物理机支撑 4 万 QPS** 是没有问题的，从 QPS 角度分析，我们要支撑 **10 亿 +** 请求，大约需要 **5 台物理机，**考虑到消费者请求，需要增加约 1.5 倍机器，即 **7 台物理机。**

**三、kafka 容量评估之磁盘**

**1、机械硬盘 OR 固态硬盘 SSD**

两者主要区别如下：

*   SSD 就是固态硬盘，它的优点是速度快，日常的读写比机械硬盘快几十倍上百倍。缺点是单位成本高，不适合做大容量存储。
    
*   HDD 就是机械硬盘，它的优点是单位成本低，适合做大容量存储，但速度远不如 SSD。
    

首先 SSD 硬盘性能好，主要是指的随机读写能力性能好，非常适合 Mysql 这样的集群，而 **SSD 的顺序读写性能跟机械硬盘的性能是差不多的。**

**Kafka 写磁盘是顺序追加写的，**所以对于 kafka 集群来说，我们使用**普通机械硬盘**就可以了。

**2、每台服务器需要多少块硬盘**

根据**第一二步骤**计算结果，我们需要 **7 台物理机，**一共需要存储 **162T** 数据，大约每台机器需要存储 **23T** 数据，根据以往经验一般服务器配置 **11 块**硬盘，这样每块硬盘大约存储 2T 的数据就可以了，另外为了服务器性能和稳定性，我们一般要保留一部分空间，保守按每块硬盘**最大能存储 3T 数据。**

**3、场景总结**

要搞定 **10 亿 +** 请求，需要 **7 台物理机，**使用**普通机械硬盘**进行存储，每台服务器 **11 块**硬盘，每块硬盘存储 **2T** 数据。

**四、kafka 容量评估之内存**

**1、Kafka 写磁盘流程及内存分析**

![](https://mmbiz.qpic.cn/mmbiz_png/ufWcjcomw8Zicoe8pJRtKkBFeL2ib8P4MAppxmyvEpv43XubfYgBRian0N6mlcylJWT8or72IKYdQJ3EUCTPLhHsw/640?wx_fmt=png)

从上图可以得出 Kafka 读写数据的流程主要都是基于 os cache，所以基本上 Kafka 都是基于内存来进行数据流转的，这样的话要**分配尽可能多的内存资源给 os cache。**  

kafka 的核心源码基本都是用 scala 和 java (客户端) 写的，底层都是基于 JVM 来运行的，所以要分配一定的内存给 JVM 以保证服务的稳定性。对于 Kafka 的设计，并没有把很多的数据结构存储到 JVM 中，**所以根据经验，给 JVM 分配 6~10G 就足够了。**

![](https://mmbiz.qpic.cn/mmbiz_png/ufWcjcomw8Zicoe8pJRtKkBFeL2ib8P4MAu1Ickbq5vCeZL01AejBhfs56p6PLcCZ53AWAibOWicuxbGa4xiapYhMmA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ufWcjcomw8Zicoe8pJRtKkBFeL2ib8P4MAYcGw7ewrYPsiaSRUG72ibHGeibSl7YUtEe4sxs7HNlInagoIePEJjKKRw/640?wx_fmt=png)

从上图可以看出一个 Topic 会对于多个 partition，一个 partition 会对应多个 segment ，一个 segment 会对应磁盘上 4 个 log 文件。假设我们这个平台总共 100 个 Topic ，那么总共有 **100 Topic * 5 partition * 3 副本 = 1500 partition 。**对于 partition 来说实际上就是物理机上一个文件目录， .log 就是存储数据文件的，默认情况下一个**.log 日志文件大小为 1G。**  

如果要保证这 1500 个 partition 的最新的 .log 文件的数据都在内存中，这样性能当然是最好的，需要 **1500  * 1G = 1500 G** 内存，但是我们没有必要所有的数据都驻留到内存中，我们只保证 **25% 左右**的数据在内存中就可以了，这样大概需要 **1500 * 250M = 1500 * 0.25G = 375G 内存，**通过**第二步**分析结果，我们总共需要 **7 台物理机，**这样的话**每台服务器只需要约 54G 内存，外加上面分析的 JVM 的 10G，总共需要 64G 内存。**还要保留一部分内存给操作系统使用，故我们选择 **128G 内存的服务器**是非常够用了。

**2、场景总结**

要搞定 **10 亿 +** 请求，需要 **7 台物理机，**每台物理机内存选择 **128G 内存**为主，这样内存会比较充裕。

**五、kafka 容量评估之 CPU 压力**

**1、CPU Core 分析**

我们评估需要多少个 CPU Core，主要是看 Kafka 进程里会有多少个线程，**线程主要是依托多核 CPU 来执行的，如果线程特别多，但是 CPU 核很少，就会导致 CPU 负载很高，会导致整体工作线程执行的效率不高, 性能也不会好。**所以我们要保证 CPU Core 的充足，来保障系统的稳定性和性能最优。

**2、Kafka 网络架构及线程数计算**

![](https://mmbiz.qpic.cn/mmbiz_png/ufWcjcomw8Zicoe8pJRtKkBFeL2ib8P4MAwn1ctLCrKzUtmJzSjlorE5fQQ7HwbbtNL7BY0LulSPAFsdJZx0fA4A/640?wx_fmt=png)

我们评估下 Kafka 服务器启动后会有多少线程在跑，其实这部分内容跟 kafka 超高并发网络架构密切相关，**上图是 Kafka 超高并发网络架构图，**从图中我们可以分析得出：  

![](https://mmbiz.qpic.cn/mmbiz_png/ufWcjcomw8Zicoe8pJRtKkBFeL2ib8P4MAI7rYrRfEicH2CMyY06abdEEiczuGQyKdJJ0YFPApeUJOnlHeS0e6WicCg/640?wx_fmt=png)

除了上图所列的还有其他一些线程，所以估算下来，一个 kafka 服务启动后，会**有 100 多个线程在跑。**  

![](https://mmbiz.qpic.cn/mmbiz_png/ufWcjcomw8Zicoe8pJRtKkBFeL2ib8P4MAypDCSGsIl6jrKCFDExfWSJE1MztMViaOEibsIzjad5Ry3icf5scNOTmsA/640?wx_fmt=png)

**2、场景总结**

要搞定 **10 亿 +** 请求，需要 **7 台物理机，**每台物理机内存选择 **128G 内存**为主，需要 **16 个 cpu core(32 个性能更好)。**

**六、kafka 容量评估之网卡**

**1、网卡对比分析**

![](https://mmbiz.qpic.cn/mmbiz_png/ufWcjcomw8Zicoe8pJRtKkBFeL2ib8P4MAf8PFFRwLLLvtFG7XPhvX85wCP9icn28icHYJ9bxTBOGwdIv3t3B5icuJQ/640?wx_fmt=png)

通过上图分析可以得出**千兆网卡和万兆网卡的区别最大之处在于网口的传输速率的不同，千兆网卡的传输速率是 1000Mbps，万兆网卡的则是 10Gbps 万兆网卡是千兆网卡传输速率的 10 倍。**性能上讲，万兆网卡的性能肯定比千兆网卡要好。万兆网卡现在主流的是 10G 的，发展趋势正逐步面向 40G、100G 网卡。但还是要根据使用环境和预算来选择投入，毕竟千兆网卡和万兆网卡的性价比区间还是挺大的。  

**2、网卡选择分析**

根据**第一二步**分析结果，高峰期的时候，每秒会有大约 **6 万**请求涌入，即每台机器**约 1 万**请求涌入 **(60000 / 7)，**每秒要接收的数据大小为: **10000 * 20 kb = 184 M/s，**外加上数据副本的同步网络请求，总共需要 **184 * 3 = 552 M/s。**

![](https://mmbiz.qpic.cn/mmbiz_png/ufWcjcomw8Zicoe8pJRtKkBFeL2ib8P4MAT1yNWsETTXWBqka1Axn37ucgR710mCbXgUgXkTO4DFfR0jpmYYXbtA/640?wx_fmt=png)

**一般情况下，网卡带宽是不会达到上限的，对于千兆网卡，我们能用的基本在 700M 左右，**通过上面计算结果，千兆网卡基本可以满足，万兆网卡更好。  

**3、场景总结**

要搞定 **10 亿 +** 请求，需要 **7 台物理机，**每台物理机内存选择 **128G 内存**为主，需要 **16 个 cpu core(32 个性能更好)，千兆网卡基本可以满足，万兆网卡更好。**

**7、kafka 容量评估之核心参数**

![](https://mmbiz.qpic.cn/mmbiz_png/ufWcjcomw8Zicoe8pJRtKkBFeL2ib8P4MA5Qgfvry2FuOZxmBmRrqQzobspsFQsIJt0dUrQwsltuw8AC7Wicg8brw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ufWcjcomw8Zicoe8pJRtKkBFeL2ib8P4MAt3uAjQYTVcSxnvScU0spX11ia8DOhibR3uSrylmicGIOYoe58SK5Bicia3Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ufWcjcomw8Zicoe8pJRtKkBFeL2ib8P4MAw3borzCq50cZicEiaPfhib8W2Y79kTKcevyBVckZmkdIqibPzibz3ktNzAQ/640?wx_fmt=png)

**8、kafka 容量评估之集群规划**

**1、集群部署规划**

这里我采用五台服务器来构建 Kafka 集群，集群依赖 ZooKeeper，所以在部署 Kafka 之前，需要部署好 ZooKeeper 集群。这里我将 Kafka 和 ZooKeeper 部署在了一起，Kafka 集群节点操作系统仍然采用 Centos 7.7 版本，各个主机角色和软件版本如下表所示：

![](https://mmbiz.qpic.cn/mmbiz_png/ufWcjcomw8Zicoe8pJRtKkBFeL2ib8P4MA15taS2wkMzLv9synicGbO72dib7barNeq7fvXGzjiaBVg8OtKhol3IvKA/640?wx_fmt=png)

这里需要注意：Kafka 和 ZooKeeper 的版本，默认 Kafka2.11 版本自带的 ZooKeeper 依赖 jar 包版本为 3.5.7，因此 ZooKeeper 的版本至少在 3.5.7 及以上。  

**2、下载与安装**

Kafka 需要安装 Java 运行环境，你可以点击 Kafka 官网

(https://kafka.apache.org/downloads) 获取 Kafka 安装包，推荐的版本是 kafka_2.11-2.4.1.tgz。将下载下来的安装包直接解压到一个路径下即可完成 Kafka 的安装，这里统一将 Kafka 安装到 /usr/local 目录下，我以在 kafka-zk1 主机为例，基本操作过程如下：

```
[root@kafkazk1~]# tar -zxvf kafka_2.11-2.4.1.tgz  -C /usr/local
[root@kafkazk1~]# mv /usr/local/kafka_2.11-2.4.1  /usr/local/kafka
```

这里我创建了一个 Kafka 用户，用来管理和维护 Kafka 集群，后面所有对 Kafka 的操作都通过此用户来完成，执行如下操作进行创建用户和授权：

```
[root@kafkazk1~]# useradd kafka
[root@kafkazk1~]# chown -R kafka:kafka /usr/local/kafka
```

在 kafka-zk1 节点安装完成 Kafka 后，先进行配置 Kafka，等 Kafka 配置完成，再统一打包复制到其他两个节点上。

```
broker.id=1
listeners=PLAINTEXT://172.16.213.31:9092
log.dirs=/usr/local/kafka/logs
num.partitions=6
log.retention.hours=72
log.segment.bytes=1073741824
zookeeper.connect=172.16.213.31:2181,172.16.213.32:2181,172.16.213.33:2181
auto.create.topics.enable=true
delete.topic.enable=true
num.network.threads=9
num.io.threads=32
message.max.bytes=10485760
log.flush.interval.message=10000
log.flush.interval.ms=1000
replica.lag.time.max.ms=10
```

Kafka 配置文件修改完成后，接着打包 Kafka 安装程序，将程序复制到其他 4 个节点，然后进行解压即可。注意，在其他 4 个节点上，broker.id 务必要修改，Kafka 集群中 broker.id 不能有相同的 (唯一的)。 

**3、启动集群**

五个节点的 Kafka 配置完成后，就可以启动了，但在启动 Kafka 集群前，需要确保 ZooKeeper 集群已经正常启动。接着，依次在 Kafka 各个节点上执行如下命令即可：

```
[root@kafkazk1~]# cd /usr/local/kafka
[root@kafkazk1 kafka]# nohup bin/kafka-server-start.sh  config/server.properties &
[root@kafkazk1 kafka]# jps
21840 Kafka
15593 Jps
15789 QuorumPeerMain
```

这里将 Kafka 放到后台 (deamon) 运行，启动后，会在启动 Kafka 的当前目录下生成一个 nohup.out 文件，可通过此文件查看 Kafka 的启动和运行状态。通过 jps 指令，可以看到有个 Kafka 标识，这是 Kafka 进程成功启动的标志。  

**九、总结**

**整个场景总结：**

要搞定 **10 亿 +** 请求，经过上面深度剖析评估后需要以下资源：

![](https://mmbiz.qpic.cn/mmbiz_png/ufWcjcomw8Zicoe8pJRtKkBFeL2ib8P4MAqc0nVdP3a72E0OMibrNFu2nqc0Uy25NE6ZFxTwXWzUHtNCXYPl6wy0w/640?wx_fmt=png)

作者丨王江华

来源丨公众号：华仔聊技术（ID：gh_97b8de4b5b34）

dbaplus 社群欢迎广大技术人员投稿，投稿邮箱：editor@dbaplus.cn