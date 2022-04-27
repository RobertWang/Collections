> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5ODYwMjI2MA==&mid=2649770438&idx=1&sn=a23ed1876fea4973df76c560c040fbb9&chksm=beccdabd89bb53ab2ef8e592e7b7b4d9078c34162518ca16f915fe4807e1947c705d5be1f35d&scene=21#wechat_redirect)

### 一、背景

#### 1 后台系统由集中式发展为分布式

#### 2 分布式系统架构引入的新问题

（1）由于多节点甚至多地部署，节点之间的数据一致性如何保证？

（2）在并发场景下如何保证任务只被执行一次？

（3）一个节点挂掉不能提供服务时如何被集群知晓并由其他节点接替任务？

（4）存在资源共享时，资源的安全性和互斥性如何保证？

以上列举了分布式系统中面临的一些挑战，需要一个协调机制来解决分布式集群中的问题，使得开发者更专注于应用本身的逻辑而不是关注分布式系统处理。

#### 3 分布式协调组件

为解决分布式系统中面临的这些问题，开发者们通过工程实践创造了很多非常优秀的分布式系统协调组件，这些组件可以在分布式环境下，保证分布式系统的数据一致性和容错性等。其中为我们熟知的有：ZooKeeper、ETCD、Consul 等。ZooKeeper 作为 Apache 的顶级开源项目，基于 Google Chubby 开源实现，在 Hadoop，Hbase，Kafka 等技术中充当核心组件的角色。虽然历史悠久，但就像陈酿一样，其设计思想和实现不论何时还是值得仔细学习和品味。

### 二、ZooKeeper

#### 1 ZooKeeper 是什么

从理论概念角度解释：ZooKeeper 是一个分布式的，开源的分布式应用程序协调服务，它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。

从数据读写角度解释：ZooKeeper 是一个分布式的开源协调服务，用于分布式系统。ZooKeeper 允许你读取、写入数据和发现数据更新。数据按层次结构组织在文件系统中，并复制到 ensemble（一个 ZooKeeper 服务器的集合）中所有的 ZooKeeper 服务器。对数据的所有操作都是原子的和顺序一致的。ZooKeeper 通过 Zab 一致性协议在 ensemble 的所有服务器之间复制一个状态机来确保这个特性。

#### 2 ZooKeeper 的安装与使用

**“纸上得来终觉浅，绝知此事要躬行”**，学习一个新的组件，我们先通过安装使用，对配置、API 等有一个直观的认识，也为后面动手实现一些功能部署好开发环境基础。

##### 2.1 ZooKeeper 下载与安装

（1）ZooKeeper 使用 JAVA 语言开发，使用前需要先安装 JDK(读者自行安装)，安装 JDK 后可在终端命令行中使用 java -version 命令查看版本（**注意：本文均在 Linux 环境下指导演示**）。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiarFEVW6Au5sVDDxvO2OKnvyvibrvnXADibmJHnRL4qpXnNDFwUgyOmiaerQ/640?wx_fmt=jpeg)

（2）ZooKeeper 下载：https://zookeeper.apache.org/releases.html 

在下载页面分为最新的 Release 版本和最近的稳定 Release 版本，这里生产环境使用推荐稳定版本，点击下载并上传 apache-zookeeper-3.7.0-bin.tar.gz 到 Linux 服务器上。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiarnossasJmzCTEqp7DgEFuWCQkyN2lJ5fkrEAGKSZd0V5MRsR7xf9IBw/640?wx_fmt=jpeg)

（3）ZooKeeper 安装：ZooKeeper 安装分为集群安装和单机安装，生产环境一般为集群安装。此处作为演示，使用一台服务器来做模拟集群，也称伪集群安装 (通过三个不同的文件夹 zk1/zk2/zk3，模拟真实环境中的三台服务器实例)。

1.  本篇中我们将要在本地开发机上安装三个 zk 实例（可以认为在生产集群模式中，这是三台不同的服务器），其安装位置分别如下：
    
    ```
    /Users/newboy/ZooKeeper/zk1/Users/newboy/ZooKeeper/zk2/Users/newboy/ZooKeeper/zk3
    ```
    
2.  将上文中下载的 ZooKeeper 安装包 apache-zookeeper-3.7.0-bin.tar.gz 上传到第一个实例 zk1 文件夹下，并使用如下命令进行解压：
    
    ```
    tar -xzvf apache-zookeeper-3.7.0-bin.tar.gz
    ```
    
3.  解压完成后在 zk1 文件夹下创建 data 和 log 目录，分别用于存储当前 zk 实例数据和日志：
    
    ```
    mkdir data logs
    ```
    
    此时 zk1 文件夹目录结构如下所示：
    
    ![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiarKjrnkaLqn31aEMXvm0oJ2ibNaze0W6baGyltxeuGBZwg50Kw2Eu0E9Q/640?wx_fmt=jpeg)
    

4.  创建 myid 文件 在 zk1 的 data 目录下，**创建 myid 文件**，此文件记录节点 id，每个 zookeeper 节点都需要一个 myid 文件来记录节点在集群中的 id，此文件中只能有一个数字，这里 zk1 实例 myid 中写入一个 1 即可：
    
    ```
    echo "1" >> /Users/newboy/ZooKeeper/zk1/data/myid  // 实例zk1的myid赋值为1echo "2" >> /Users/newboy/ZooKeeper/zk2/data/myid  // 实例zk2的myid赋值为2echo "3" >> /Users/newboy/ZooKeeper/zk3/data/myid  // 实例zk3的myid赋值为3
    ```
    
5.  进入 zk1 文件夹下 apache-zookeeper-3.7.0-bin/conf / 目录，将配置文件 zoo_sample.cfg 重命名为 zoo.cfg，打开 zoo.cfg 进行配置，具体配置如下:
    
    ```
    tickTime=2000  # 单位时间，其他时间都是以这个倍数来表示initLimit=10   # 节点初始化时间，10倍单位时间(即十倍tickTime)syncLimit=5    # 心跳最大延迟周期dataDir=/Users/newboy/ZooKeeper/zk1/data     # 该实例对应的数据目录（上文步骤3创建）dataLogDir=/Users/newboy/ZooKeeper/zk1/logs  # 该实例对应的日志目录（上文步骤3创建）clientPort=2181                              # 端口（每个实例不同）// 集群配置server.1=127.0.0.1:8881:7771                 # server.id=host:port:portserver.2=127.0.0.1:8882:7772                 # server.id=host:port:portserver.3=127.0.0.1:8883:7773                 # server.id=host:port:port
    ```
    
    集群配置中模版为 server.id=host:port:port，id 是上面 myid 文件中配置的 id；ip 是节点的 ip，第一个 port 是节点之间通信的端口，第二个 port 用于选举 leader 节点（在真正的集群模式下，不同服务器可以共用同一个 port，这里单机上演示为了避免端口冲突，选择不同的端口）。
    

6.  zk2 和 zk3 的实例配置与 zk1 类似，为了方便我们可以直接拷贝 zk1 的配置到 zk2 和 zk3 文件夹，然后修改各自的 zoo.cfg 和 data 目录下的 myid 即可。拷贝命令：
    
    ```
    cp -R zk1 zk2cp -R zk1 zk3
    ```
    
    zk2 对应的 zoo.cfg：
    
    ```
    tickTime=2000  # 单位时间，其他时间都是以这个倍数来表示initLimit=10   # 节点初始化时间，10倍单位时间(即十倍tickTime)syncLimit=5    # 心跳最大延迟周期dataDir=/Users/newboy/ZooKeeper/zk2/data     # 该实例对应的数据目录（上文步骤3创建）dataLogDir=/Users/newboy/ZooKeeper/zk2/logs  # 该实例对应的日志目录（上文步骤3创建）clientPort=2182                              # 端口（每个实例不同）// 集群配置server.1=127.0.0.1:8881:7771                 # server.id=host:port:portserver.2=127.0.0.1:8882:7772                 # server.id=host:port:portserver.3=127.0.0.1:8883:7773                 # server.id=host:port:port
    ```
    
    zk3 对应的 zoo.cfg：
    
    ```
    tickTime=2000  # 单位时间，其他时间都是以这个倍数来表示initLimit=10   # 节点初始化时间，10倍单位时间(即十倍tickTime)syncLimit=5    # 心跳最大延迟周期dataDir=/Users/newboy/ZooKeeper/zk3/data     # 该实例对应的数据目录（上文步骤3创建）dataLogDir=/Users/newboy/ZooKeeper/zk3/logs  # 该实例对应的日志目录（上文步骤3创建）clientPort=2183                              # 端口（每个实例不同）// 集群配置server.1=127.0.0.1:8881:7771                 # server.id=host:port:portserver.2=127.0.0.1:8882:7772                 # server.id=host:port:portserver.3=127.0.0.1:8883:7773                 # server.id=host:port:port
    ```
    
    至此 zk 伪集群模式的安装配置已经完成，整体目录结构纵览如下：
    
    ```
    .├── zk1│   ├── data│   │     └── myid│   ├── logs│   └── apache-zookeeper-3.7.0-bin├── zk2│   ├── data│   │     └── myid│   ├── logs│   └── apache-zookeeper-3.7.0-bin└── zk3│   ├── data│   │     └── myid    ├── logs    └── apache-zookeeper-3.7.0-bin
    ```
    
    （4）ZooKeeper 实例启动及使用客户端交互：
    

1.  启动刚刚创建的三个 zk 实例 (1) 启动 zk1 实例，命令行运行下面命令：
    
    ```
    // 启动命令/Users/newboy/ZooKeeper/zk1/apache-zookeeper-3.7.0-bin/bin/zkServer.sh start// 启动结果ZooKeeper JMX enabled by defaultUsing config: /Users/newboy/ZooKeeper/zk1/apache-zookeeper-3.7.0-bin/bin/../conf/zoo.cfgStarting zookeeper ... STARTED
    ```
    
    (2) 同样启动 zk2 和 zk3 实例，命令行运行下面命令：
    
    ```
    // 启动zk2命令/Users/newboy/ZooKeeper/zk2/apache-zookeeper-3.7.0-bin/bin/zkServer.sh start// 启动zk3命令/Users/newboy/ZooKeeper/zk3/apache-zookeeper-3.7.0-bin/bin/zkServer.sh start
    ```
    

2.  连接实例 所有实例全部启动过后，选择任一实例进行连接，这里选择实例 zk2，命令行输入如下命令：
    
    ```
    /Users/newboy/ZooKeeper/zk2/apache-zookeeper-3.7.0-bin/bin/zkCli.sh -server 127.0.0.1:2182
    ```
    

3.  创建节点 连接之后，可以在当前实例上创建节点，类似于创建一个 kv 值或者文件夹（ZK 的命令和可选参数读者可以自行查看用户手册）
    
    ```
    // 创建节点 create表示创建命令，/zk-demo为节点名称 123为节点值[zk: 127.0.0.1:2181(CONNECTED) 1] create /zk-demo 123Created /zk-demo// 获取节点值 get表示获取 /zk-demo为需要获取的节点名称[zk: 127.0.0.1:2181(CONNECTED) 2] get /zk-demo123
    ```
    

4.  在其他实例上获取 zk2 实例创建的节点 由于 zk 会将节点写入的值同步到集群中每个节点，从而保证数据的一致性，那么其他节点理论上也可以访问到刚刚 zk2 创建的值。下面连接 zk1 来验证下:
    
    ```
    // 连接zk1/Users/newboy/ZooKeeper/zk1/apache-zookeeper-3.7.0-bin/bin/zkCli.sh -server 127.0.0.1:2181// 获取zk2上创建的节点/zk-demo[zk: 127.0.0.1:2183(CONNECTED) 0] get /zk-demo123
    ```
    
    可以看到，我们成功的在实例 zk1 上获取到了实例 zk2 创建的节点，说明数据写入 zk2 后，在各个节点间同步并实现了一致，zk 的下载、安装和基本命令操作也就讲完了。
    

#### 3 ZooKeeper 能做什么

前文中，我们了解了 ZooKeeper 出现的背景，它是分布式系统中非常重要的中间件，分布式应用程序可以基于 ZooKeeper 实现：

*   数据的发布和订阅
    
*   服务注册与发现
    
*   分布式配置中心
    
*   命名服务
    
*   分布式锁
    
*   Master 选举
    
*   负载均衡
    
*   分布式队列
    

可以看到 ZooKeeper 可以实现非常多的功能，之所以能够实现各种不同的能力，源于 ZooKeeper 底层的数据结构和数据模型。

#### 4 ZooKeeper 的数据结构和数据模型

##### 1 Znode 数据节点

ZooKeeper 的数据节点可以视为树状结构，树中的各节点被称为 Znode（即 ZooKeeper node），一个 Znode 可以有多个子节点，ZooKeeper 中的所有存储的数据是由 znode 组成，并以 key/value 形式存储数据。整体结构类似于 linux 文件系统的模式以树形结构存储，其中根路径以 / 开头：

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiarhU6ph7yibBJ9hCKFiabOC01nHEpdkDJRicfjfRX3RiceEDoS52gQB0QSFQ/640?wx_fmt=jpeg)

如上图所示，在根目录下我们创建 Dog 和 Cat 两个不同的数据节点，Cat 节点下有 TomCat 这个数据存储节点，整个 ZooKeeper 的树形存储结构就是这样的 Znode 构成，并存储在内存中。

命令行下使用 ZooKeeper 客户端工具创建节点的过程如下：首先连接一个 zk 实例：

创建节点：

```
// 连接zk1/Users/newboy/ZooKeeper/zk1/apache-zookeeper-3.7.0-bin/bin/zkCli.sh -server 127.0.0.1:2181
```

使用 ls 命令查看各个目录下的节点数据：

```
[zk: 127.0.0.1:2181(CONNECTED) 5] create /DogCreated /Dog[zk: 127.0.0.1:2181(CONNECTED) 6] create /CatCreated /Cat[zk: 127.0.0.1:2181(CONNECTED) 7] create /Cat/TomCatCreated /Cat/TomCat
```

Znode 节点类似于 Unix 文件系统，但也有自己的特性：

（1）**Znode 兼具文件和目录特点** 既像文件一样维护着数据、信息、时间戳等数据，又像目录一样可以作为路径标识的一部分，并可以具有子 Znode。用户对 Znode 具有增、删、改、查等操作；

（2）**Znode 具有原子性操作** 读操作将获取与节点相关的所有数据，写操作也将替换掉节点的所有数据；

（3） **Znode 存储数据大小有限制** 每个 Znode 的数据大小至多 1M，但是常规使用中应该远小于此值；

（4）**Znode 通过路径引用** 如同 Unix 中的文件路径。路径必须是绝对的，因此他们必须由斜杠字符来开头。除此以外，他们必须是唯一的，也就是说每一个路径只有一个表示，因此这些路径不能改变。

##### 2 Znode 节点类型

Znode 有两种，分别为临时节点和永久节点，节点的类型在创建时即被确定，并且不能改变。

临时节点：该节点的生命周期依赖于创建它们的会话。一旦会话结束，临时节点将被自动删除，当然可以也可以手动删除。临时节点不允许拥有子节点。

永久节点：该节点的生命周期不依赖于会话，并且只有在客户端显式执行删除操作的时候，才能被删除。

Znode 还有一个序列化的特性，如果创建的时候指定的话，该 Znode 的名字后面会自动追加一个递增的序列号。序列号对于此节点的父节点来说是唯一的，这样便会记录每个子节点创建的先后顺序。因此组合之后，Znode 有四种节点类型：

*   PERSISTENT：永久节点
    
*   EPHEMERAL：临时节点
    
*   PERSISTENT_SEQUENTIAL：永久顺序节点
    
*   EPHEMERAL_SEQUENTIAL：临时顺序节点
    

为了对节点类型有更清楚的认识，在命令行下来模拟创建一个临时节点：（1）首先连接 zk1 实例：

```
[zk: 127.0.0.1:2181(CONNECTED) 8] ls /[Cat, Dog, zk-demo, zookeeper][zk: 127.0.0.1:2181(CONNECTED) 10] ls /Cat[TomCat]
```

（2）创建一个临时节点：

```
// 连接zk1/Users/newboy/ZooKeeper/zk1/apache-zookeeper-3.7.0-bin/bin/zkCli.sh -server 127.0.0.1:2181
```

（3）连接 zk2 实例，查看该临时节点是否同步：

```
// -e 表示该节点为临时节点[zk: 127.0.0.1:2181(CONNECTED) 12] create -e /Dog/Puppy 123Created /Dog/Puppy
```

（4）断开 zk1 实例的会话

```
// 连接zk2/Users/newboy/ZooKeeper/zk2/apache-zookeeper-3.7.0-bin/bin/zkCli.sh -server 127.0.0.1:2182// 查询/Dog/Puppy节点值[zk: 127.0.0.1:2182(CONNECTED) 2] get /Dog/Puppy123
```

（5）在 zk2 上查看该节点

```
[zk: 127.0.0.1:2181(CONNECTED) 16] quitWATCHER::WatchedEvent state:Closed type:None path:null2022-03-15 15:39:55,807 [myid:] - INFO  [main:ZooKeeper@1232] - Session: 0x1000c3279ae0000 closed2022-03-15 15:39:55,807 [myid:] - INFO  [main-EventThread:ClientCnxn$EventThread@570] - EventThread shut down for session: 0x1000c3279ae00002022-03-15 15:39:55,810 [myid:] - ERROR [main:ServiceUtils@42] - Exiting JVM with code 0
```

可以看到 / Dog/Puppy 临时节点随着 zk1 实例会话的退出消失了，这就是临时节点的特性，zk1 创建的临时节点会随着 zk1 实例连接的退出而消失，永久节点则只能通过 delete /Dog(节点名) 删除才会消失。

##### 3 ZooKeeper 的 Znode Watcher 机制

ZooKeeper 可以用来做数据的发布和订阅，一个典型的发布 / 订阅模型系统定义了一种一对多的订阅关系，能够让多个订阅者同时监听某一个主题对象，当这个主题对象自身状态变化时，会通知所有订阅者，使它们能够做出相应的处理。在 ZooKeeper 中，引入了 **Watcher 机制**来**实现这种分布式的通知功能**。

ZooKeeper 允许 ZK 客户端向服务端注册一个 Watcher 监听，当服务端的一些指定事件触发了这个 Watcher，那么就会向指定客户端发送一个事件通知。例如 ZK 客户端监听临时节点 / Cat, 当该临时节点消失时，则会由服务端触发调用客户端 WatchManager，客户端从 WatchManager 中取出对应的 Watcher 对象来进行处理逻辑。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiarIKiagcs4wSgGPbHqgkEAEWoVR7T4TpHWjVFQicmIOeun8Nua25L6MYng/640?wx_fmt=jpeg)

（1）客户端首先将 Watcher 注册到服务端，同时将 Watcher 对象保存到客户端的 Watch 管理器中；（2）当 ZooKeeper 服务端监听的数据状态发生变化时，服务端会主动通知客户端；（3) 接着客户端的 Watch 管理器会触发相关 Watcher 来回调相应处理逻辑，从而完成整体的数据发布 / 订阅流程。

##### 4 经典案例：基于 Znode 临时顺序节点 + Watcher 机制实现公平分布式锁

1、临时顺序节点：在介绍 Znode 节点时，我们提到过 Znode 节点有 “临时节点” 这个类型，它会随着客户端连接的断开而消失，同时节点类型可以选择顺序性，组合起来就是“临时顺序节点”，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiarcgBfJy6Yl2KKHkDb79EBcHiaqBXQN6SA83aibXJsy2yvCFQ5Jgm42CfQ/640?wx_fmt=jpeg)

在根目录 “/” 下创建分布式锁 “/Lock” 节点目录，/Lock 节点本身可以是永久节点，用于存放客户端抢占创建的临时顺序节点。此时假设有两个 ZK 客户端 A 和 B 同时调用 Create 函数，在 "/Lock" 节点下创建临时顺序节点，A 比 B 网络延时更小，先创建，ZK 分配节点名称为 "/Lock/Seq0001", B 晚于 A 创建成功，ZK 分配节点名为 "/Lock/Seq0002"，ZK 负责维护这个递增的顺序节点名。

2、分布式锁实现的具体流程 （1）如下图，客户端 A、B 同时在 "/Lock" 节点下创建临时顺序子节点，可以理解为同时抢占分布式锁，A 先于 B 创建成功，此时分配的节点为 “/Lock/seq-0000001”，由于 A 创建成功，并且临时顺序节点的顺序值序号最小，代表它是最先获取到该锁，此时加锁成功。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiarFfz0LSibicnibbkXGEIQ75ShibWDm4pGiaTuOfWsQFIIc6Zdgu9qaguiaFkA/640?wx_fmt=jpeg)

（2）如下图，（红色虚线）客户端 B 晚于 A 创建临时顺序节点，此时 ZK 分配的节点顺序值为 “/Lock/seq-0000002”，B 创建成功之后，它的顺序值大于 A 的顺序值，不是最小顺序值，此时说明 A 已经抢占到分布式锁，这个时候 B 就使用 Watcher 监听机制，监听次小于自己的临时顺序节点 A 的状态变化。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiaroEpITK5kBrSfD3ibDgeYwwmKicgTjvQDdagq6YmxquXCCZxHbtAYDs0A/640?wx_fmt=jpeg)

（3）如下图，当 A 客户端因宕机或者完成处理逻辑而断开链接时，A 创建的临时顺序节点会随之消失，此时由于客户端 B 已经监听了 A 临时顺序节点的状态变化，当消失事件发生时，Watcher 监听器逻辑会回调客户端 B，B 重新开始获取锁。注意此时不是 B 再次创建节点，而是获取 "/Lock" 下的临时顺序节点，发现自己的顺序值最小，那么就加锁成功。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiarkicxiaF1q9rCM4vTIrUC6JErBnBFIsibP0IZrD2HRHlaWPNNqibRGFicW7g/640?wx_fmt=jpeg)

如果有 C、D 甚至更多的客户端同时抢占，原理都是一致的，他们会依次排队，监听自己之前 (节点顺序值次小于自己) 的节点，等待他们的状态发生变化时，再去重新获取锁。

这里使用临时顺序节点和 Watcher 机制实现了一个公平分布式锁，还有很多其他用法，如只使用临时节点实现非公平分布式锁，篇幅所限，读者可以自行探索。

### 三、深入 ZooKeeper 一致性协议原理

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiar0NymRc9gbnJ4oyAicicqBtdBp6y6OEvJBqRtNw745aN8geib2B7P1IHPw/640?wx_fmt=jpeg)

上图是 ZooKeeper 的整体架构，ZooKeeper Service 是服务端集群，也是整个组件的核心，客户端的读写请求都是它来处理。ZK 下载安装章节模拟的 zk1/zk2/zk3 就可以认为是一个 ZK 服务端集群，我们在 zk2 中写入的节点值，在 zk1 和 zk3 实例中也能读到这个节点值，zk2 会话退出后临时节点在其他服务器上也同样消失了，ZK 服务端是通过什么机制实现数据在各个节点之间的同步，从而保证一致性？当有节点出现故障时又是如何保证正常提供对外服务？这就涉及到 ZooKeeper 的核心 - 分布式一致性原理。

#### 1 ZooKeeper 服务端角色

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiarerxgWybr0QcGpuNEwhou2a1qpuIB9fqFJnovmhFosic0RicicXt1SWJDQ/640?wx_fmt=jpeg)

*   **Leader** 一个 ZooKeeper 集群同一时间只会有一个实际工作的 Leader，它会发起并维护与各 Follwer 及 Observer 间的心跳。所有的写操作必须要通过 Leader 完成再由 Leader 将写操作广播给其它服务器。
    
*   **Follower** 一个 ZooKeeper 集群可能同时存在多个 Follower，它会响应 Leader 的心跳。Follower 可直接处理并返回客户端的读请求，同时会将写请求转发给 Leader 处理，并且负责在 Leader 处理写请求时对请求进行投票。
    
*   **Observer** 角色与 Follower 类似，但是无投票权。
    

1、早期的 ZooKeeper 集群服务运行过程中，只有 Leader 服务器和 Follow 服务器 2、随着集群规模扩大，follower 变多，ZK 在创建节点和选主等事务性请求时，需要一半以上节点 AC，所以导致性能下降写入操作越来越耗时，follower 之间通信越来越耗时 3、为了解决这个问题，就引入了观察者，可以处理读，但是不参与投票。既保证了集群的扩展性，又避免过多服务器参与投票导致的集群处理请求能力下降 `

#### 2 一致性协议 - ZAB

ZooKeeper 为了保证集群中各个节点读写数据的一致性和可用性，设计并实现了 ZAB 协议，ZAB 全称是 ZooKeeper Atomic Broadcast，也就是 ZooKeeper 原子广播协议。这种协议支持崩溃恢复，并基于主从模式，同一时刻只有一个 Leader，所有的写操作都由 Leader 节点主导完成，而读操作可通过任意节点完成，因此 ZooKeeper 读性能远好于写性能，更适合读多写少的场景。

一旦 Leader 节点宕机，ZAB 协议的崩溃恢复机制能自动从 Follower 节点中重新选出一个合适的替代者，即新的 Leader，该过程即为领导选举。领导选举过程，是 ZAB 协议中最为重要和复杂的过程。

#### 3. ZAB 协议读写流程

##### 3.1 ZAB 写流程

###### 3.1.1 写 Leader

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiarHdPQKg7ckrTY4kZ2hrMVtaHDJRfibHROdiaMtsfWNOBJzVlTT1dM1s6w/640?wx_fmt=jpeg)

由上图可见，通过 Leader 进行写操作，主要分为五步：

1.  客户端向 Leader 发起写请求
    
2.  Leader 将写请求以 Proposal 的形式发给所有 Follower 并等待 ACK
    
3.  Follower 收到 Leader 的 Proposal 后返回 ACK
    
4.  Leader 得到过半数的 ACK（Leader 对自己默认有一个 ACK）后向所有的 Follower 和 Observer 发送 Commmit
    
5.  Leader 将处理结果返回给客户端
    

**注意** Leader 并不需要得到 Observer 的 ACK，即 Observer 无投票权

Leader 不需要得到所有 Follower 的 ACK，只要收到过半的 ACK 即可，同时 Leader 本身对自己有一个 ACK。上图中有 4 个 Follower，只需其中两个返回 ACK 即可，因为 (2+1) / (4+1) > 1/2

Observer 虽然无投票权，但仍须同步 Leader 的数据从而在处理读请求时可以返回尽可能新的数据 `

###### 3.1.2 写 Follower

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiarvzPFtoxmicqDHSnL1X9bTM2OvnicLf4ib7xy5neF4ZLgyLTMRG7mT9GhQ/640?wx_fmt=jpeg)

从上图可见：

*   Follower 可接受写请求，但不能直接处理，而需要将写请求转发给 Leader 处理
    
*   Observer 与 Follower 写流程相同
    
*   除了多了一步请求转发，其它流程与直接写 Leader 无任何区别
    

##### 3.2 ZAB 读流程

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiarDBIJPFrbl2fIvZy9bxO0cfdDc9ibCsHUz8ewibr67nybMGCoQwxdqLag/640?wx_fmt=jpeg)

Leader/Follower/Observer 都可直接处理读请求，从本地内存中读取数据并返回给客户端即可。由于处理读请求不需要服务器之间的交互，Follower/Observer 越多，整体可处理的读请求量越大，也即读性能越好。`ZooKeeper官方文档数据，Client数量1000时，读写性能比10:1`

#### 4 ZooKeeper Leader 选举算法

##### 4.1 选举算法

ZooKeeper 中默认的并建议使用的 Leader 选举算法是：基于 TCP 的 FastLeaderElection，其他选举算法被废弃。集群模式下 zoo.cfg 配置文件中有参数可配选举算法：

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiarxKkEX6ticQmCU7eWy6TocBGn0u4zRJvLicnvWicrXVG3KiasU9urdEPkFQ/640?wx_fmt=jpeg)

##### 4.2 FastLeaderElection 选举参数解析

（1）选举算法参数 **myid**：每个 ZooKeeper 服务器，都需要在数据文件夹下创建一个名为 myid 的文件，该文件包含整个 ZooKeeper 集群唯一的 ID（整数）。例如，我们第二章中部署的 zk1/zk2/zk3 三个实例，其 myid 分别为 1、2 和 3，在配置文件中其 ID 与 hostname 必须一一对应，如下所示。在该配置文件中，server. 后面的 id 即为 myid。该参数在选举时如果无法通过其他判断条件选择 Leader，那么将该 ID 的大小来确定优先级。

```
[zk: 127.0.0.1:2182(CONNECTED) 3] get /Dog/Puppyorg.apache.zookeeper.KeeperException$NoNodeException: KeeperErrorCode = NoNode for /Dog/Puppy
```

**zxid**：用于标识一次更新操作的 ID。为了保证顺序性，该 zxid 必须单调递增，因此 ZooKeeper 使用一个 64 位的数来表示，高 32 位是 Leader 的 epoch，从 1 开始，每次选出新的 Leader，epoch 加一。低 32 位为该 epoch 内的序号，每次有写操作低 32 位加一，每次 epoch 变化，都将低 32 位的序号重置。这样保证了 zxid 的全局递增性。之前看到过有博主使用中国古代的年号来解释这个字段，非常形象：万历十五年，万历是 epoch，十五年是序号**选票数据结构**，每个服务器在进行选举时，发送的选票包含如下关键信息：

```
// 集群配置server.1=127.0.0.1:8881:7771                 # server.id=host:port:portserver.2=127.0.0.1:8882:7772                 # server.id=host:port:portserver.3=127.0.0.1:8883:7773                 # server.id=host:port:port
```

**节点服务器状态**，每个服务器所处的状态时下面状态中的一种：

*   **LOOKING** 不确定 Leader 状态。该状态下的服务器认为当前集群中没有 Leader，会发起 Leader 选举。
    
*   **FOLLOWING** 跟随者状态。表明当前服务器角色是 Follower，并且它知道 Leader 是谁。
    
*   **LEADING** 领导者状态。表明当前服务器角色是 Leader，它会维护与 Follower 间的心跳。
    
*   **OBSERVING** 观察者状态。表明当前服务器角色是 Observer，与 Folower 唯一的不同在于不参与选举，也不参与集群写操作时的投票。
    

##### 4.3 选举投票流程

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiarBeuhw2bia2nD7szZwZzjUtTOZ9LppmFgqy8FvXibl5XCadyJsTUSgdKA/640?wx_fmt=jpeg)

每个服务器的一次选举流程：

1.  **自增选举轮次**：即 logicClock 加一，ZooKeeper 规定所有有效的投票都必须在同一轮次中。每个服务器在开始新一轮投票时，会先对自己维护的 logicClock 进行自增操作。
    
2.  **初始化选票**：每个服务器在开始进行新一轮的投票之前，会将自己的投票箱清空，然后初始化自己的选票。在初始化阶段，每台服务器都会将自己推选为 Leader，也就是将票都投给自己。例如：服务器 1、2、3 都投票给自己 (1->1), (2->2),(3->3)。
    
3.  **发送初始化选票**：每个服务器通过广播将初始化投给自己的票广播出去，让其他服务器接收。
    
4.  **接收外部投票**：服务器会尝试从其它服务器获取投票，并记入自己的投票箱内。如果无法获取任何外部投票，则会确认自己是否与集群中其它服务器保持着有效连接。如果是，则再次发送自己的投票；如果否，则马上与之建立连接。
    
5.  **判断选举轮次**：收到外部投票后，首先会根据投票信息中所包含的 logicClock 来进行不同处理。（1）如果大于当前服务的选票中的选举次数，那么则会更新当前服务的 logicClock，并且清空所有收到的选票，再次拿选票和外部投票进行选票的比较，确定是否真的要更改自身的选票，然后重新发送选票信息；（2）如果外部选票的选举次数小于当前服务实例的选举次数，那么直接无视掉这个选票信息，并且继续发送自身的选票出去；（3）如果外部选票和自身服务实例的选举次数一致，那么就需要进入选票之间的比较操作。
    
6.  **选票 PK**：选票 PK 是基于 (self_myid, self_zxid) 与(vote_myid, vote_zxid)的对比。
    
    （1）外部投票的 logicClock 大于自己的 logicClock，则将自己的 logicClock 及自己的选票的 logicClock 变更为收到的 logicClock；
    
    （2）若 logicClock 一致，则对比二者的 vote_zxid，若外部投票的 vote_zxid 比较大，则将自己的票中的 vote_zxid 与 vote_myid 更新为收到的票中的 vote_zxid 与 vote_myid 并广播出去，另外将收到的票及自己更新后的票放入自己的票箱。如果票箱内已存在 (self_myid, self_zxid) 相同的选票，则直接覆盖；
    
    （3）若二者 vote_zxid 一致，则比较二者的 vote_myid，若外部投票的 vote_myid 比较大，则将自己的票中的 vote_myid 更新为收到的票中的 vote_myid 并广播出去，另外将收到的票及自己更新后的票放入自己的票箱。
    
7.  **统计选票**：如果已经确定有过半服务器认可了自己的投票（可能是更新后的投票），则终止投票。否则继续接收其它服务器的投票。
    
8.  **更新服务器状态**：投票终止后，服务器开始更新自身状态。若过半的票投给了自己，则将自己的服务器状态更新为 LEADING，否则将自己的状态更新为 FOLLOWING。
    

**同时还需要注意的一点是，即使选票超过半数了，选出 Leader 服务实例了，也不是立刻结束，而是等待 200ms，确保没有丢失其他服务的更优的选票。**

#### 5 ZooKeeper 集群启动选举流程图解

##### 5.1 集群启动领导选举

1、**各自推选自己**：ZooKeeper 集群刚启动时，所有服务器的 logicClock 都为 1，zxid 都为 0。各服务器初始化后，先把第一票投给自己并将它存入自己的票箱，同时广播给其他服务器。此时各自的票箱中只有自己投给自己的一票，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiarRYK1WxV6wlicoQvpZ4h9KK1pE0MOYqZzPib82X8L3oQpcJlpaqanTj2w/640?wx_fmt=jpeg)

2、**更新选票**：第一步中各个服务器先投票给自己，并把投给自己的结果广播给集群中的其他服务器，这一步其他服务器接收到广播后开始更新选票操作（如果对此规则不熟悉，可以对照 **4.3 选举投票流程**小节），以 Server1 为例流程如下：

（1）Server1 收到 Server2 和 Server3 的广播选票后，由于 logicClock 和 zxid 都相等，此时就比较 myid；

（2）Server1 收到的两张选票中 Server3 的 myid 最大，此时 Server1 判断应该遵从 Server3 的投票决定，将自己的票改投给 Server3。接下来 Server1 先清空自己的票箱 (票箱中有第一步中投给自己的选票)，然后将自己的新投票(1->3) 和接收到的 Server3 的 (3->3) 投票一起存入自己的票箱，再把自己的新投票决定 (1->3) 广播出去, 此时 Server1 的票箱中有两票：(1->3),(3->3)；

（3）同理，Server2 收到 Server3 的选票后也将自己的选票更新为（2->3）并存入票箱然后广播。此时 Server2 票箱内的选票为 (2->3)，(3->3)；

（4）Server3 根据上述规则，无须更新选票，自身的票箱内选票仍为（3->3）；

（5）Server1 与 Server2 重新投给 Server3 的选票广播出去后，由于三个服务器最新选票都相同，最后三者的票箱内都包含三张投给服务器 3 的选票。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiar7WxTKHhnhSYgibSHUXqSVqMvkFEXYFhyK4Kf95CEBMCcbaYRTLraCnA/640?wx_fmt=jpeg)

3、**根据选票确定角色**：根据上述选票，三个服务器一致认为此时 Server3 应该是 Leader。因此 Server1 和 Server2 都进入 FOLLOWING 状态，而 Server3 进入 LEADING 状态。之后 Leader 发起并维护与 Follower 间的心跳。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiarehCboUPfBFoOmY3dN9FNZ1lt8BK5yRt9zCtWbFBy9bHgGW0QF8qFMA/640?wx_fmt=jpeg)

##### 5.2 Follower 重启选举

本节讨论 Follower 节点发生故障重启或网络产生分区恢复后如何进行选举。1、**Follower 重启投票给自己**：Follower 重启，或者发生网络分区后找不到 Leader，会进入 LOOKING 状态并发起新的一轮投票。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiarwQtp3CkQRVMDNh3QPHmib8BEopWtQOKLvcvUCQOtC5YUsVkjPY06aCg/640?wx_fmt=jpeg)

2、**发现已有 Leader 后成为 Follower**：Server3 收到 Server1 的投票后，将自己的状态 LEADING 以及选票返回给 Server1。Server2 收到 Server1 的投票后，将自己的状态 FOLLOWING 及选票返回给 Server1。此时 Server1 知道 Server3 是 Leader，并且通过 Server2 与 Server3 的选票可以确定 Server3 确实得到了超过半数的选票。因此服务器 1 进入 FOLLOWING 状态。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiarYDIfC3wxAuyA1ZsTxEtFKmfTVIR3CDAV2EX4faV55Atq663slexTNg/640?wx_fmt=jpeg)

##### 5.3 Leader 宕机重启选举

1、**Follower 发起新投票**：Leader（Server3）宕机后，Follower（Server1 和 2）发现 Leader 不工作了，因此进入 LOOKING 状态并发起新的一轮投票，并且都将票投给自己，同时将投票结果广播给对方。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiar6ibqX6y2etK80p7k9bwMic47syZpkMsveslIWrsYwZOj0ACvLmzP52QQ/640?wx_fmt=jpeg)

2、**更新选票**：（1）Server1 和 2 根据外部投票确定是否要更新自身的选票，这里跟之前的选票 PK 流程一样，比较的优先级为：logicLock > zxid > myid，这里 Server1 的参数 (L=3, M=1, Z=11) 和 Server2 的参数 (L=3, M=2, Z=10)，logicLock 相等，zxid 服务器 1 大于服务器 2，因此服务器 2 就清空已有票箱，将(1->1) 和(2->1)两票存入票箱，同时将自己的新投票广播出去 （2）服务器 1 收到 2 的投票后，也将自己的票箱更新。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiareCOrtrQHFGTzHiaC4eoVmk3hEjibLicODg49iasE63ibgM2cp7iaY5JYC3xw/640?wx_fmt=jpeg)

3、**重新选出 Leader**：此时由于只剩两台服务器，服务器 1 投票给自己，服务器 2 投票给 1，所以 1 当选为新 Leader。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiarvsOyXsfB5wC5sS4cMXibE26G8bZlquia0EWU9WKcX38YRcXHX2nGlEVw/640?wx_fmt=jpeg)

4、**旧 Leader 恢复发起选举**：之前宕机的旧 Leader 恢复正常后，进入 LOOKING 状态并发起新一轮领导选举，并将选票投给自己。此时服务器 1 会将自己的 LEADING 状态及选票返回给服务器 3，而服务器 2 将自己的 FOLLOWING 状态及选票返回给服务器 3。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiaricU9XoKZtPBrC9ibggFRJ9P3LZ8I4Qg6B7sianZzcPerJfNR7WA1kNHbg/640?wx_fmt=jpeg)

5、**旧 Leader 成为 Follower**：服务器 3 了解到 Leader 为服务器 1，且根据选票了解到服务器 1 确实得到过半服务器的选票，因此自己进入 FOLLOWING 状态。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvauGUvf9s99QG3kOmDstWPiarWcS1ESFHcuuNbtHm6xiakk1icTN2jMILK9ErDTRwO6kKCq0GZ5mAh2eg/640?wx_fmt=jpeg)

#### 6 commit 过的数据不丢失

ZK 的数据写入都是通过 Leader，一条数据写入过程中，ZK 服务集群中只有超过一半的服务器返回给 Leader ACK 后，Leader 服务器才会 Commit 这条消息，同步到每一个节点。已经被过 Leader commit，也就是被过半节点同步过的消息，在 Leader 宕机之后，重新选举出 Leader 这个消息也不会丢失。但是未被 commit 也就是未被过半节点复制到的消息则会丢失。

### 四、参考文献 && 鸣谢

*   https://forthe77.github.io/2019/03/25/one-zookeeper-deploy/
    
*   https://blog.nowcoder.net/n/16f13a7d72b2496c8ff4da080f777a5a
    
*   https://blog.csdn.net/Weixiaohuai/article/details/112788171
    
*   https://www.cnblogs.com/IcanFixIt/p/7818592.html
    
*   https://www.runoob.com/w3cnote/zookeeper-znode-data-model.html
    
*   https://www.cnblogs.com/reycg-blog/p/10208585.html
    
*   https://dbaplus.cn/news-141-1875-1.html
    
*   https://www.infoq.cn/article/us5gjqqz8bmbeha25io0 [http://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/24%E8%AE%B2%E5%90%83%E9%80%8F%E5%88%86%E5%B8%83%E5%BC%8F%E6%95%B0%E6%8D%AE%E5%BA%93-%E5%AE%8C/20%20%20%E5%85%B1%E8%AF%86%E7%AE%97%E6%B3%95%EF%BC%9A%E4%B8%80%E6%AC%A1%E6%80%A7%E8%AF%B4%E6%B8%85%E6%A5%9A%20Paxos%E3%80%81Raft%20%E7%AD%89%E7%AE%97%E6%B3%95%E7%9A%84%E5%8C%BA%E5%88%AB.md](http://learn.lianglianglee.com / 专栏 / 24 讲吃透分布式数据库 - 完 / 20 共识算法：一次性说清楚 Paxos、Raft 等算法的区别. md)
    
*   https://juejin.cn/post/6907151199141625870
    
*   https://lotabout.me/2019/QQA-What-is-Sequential-Consistency/
    
*   [http://fishleap.top/pages/a958bc/#_6-%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95](http://fishleap.top/pages/a958bc/#_6-%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95)
    
*   https://developer.aliyun.com/article/768655 https://www.cnblogs.com/aspirant/p/8994227.html
    
*   https://blog.xiaohansong.com/lamport-logic-clock.html http://icyfenix.cn/distribution/consensus/
    
*   https://dbaplus.cn/news-141-2053-1.html https://www.infoq.cn/article/dvaaj71f4fbqsxmgvdce
    
*   https://segmentfault.com/a/1190000039760185[https://blog.csdn.net/chenshijie2011/article/details/118075170?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1.pc_relevant_default&utm_relevant_index=2](https://blog.csdn.net/chenshijie2011/article/details/118075170?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1.pc_relevant_default&utm_relevant_index=2)
    
*   [https://javaedge.blog.csdn.net/article/details/110585930?spm=1001.2101.3001.6650.4&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-4.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-4.pc_relevant_default&utm_relevant_index=7](https://javaedge.blog.csdn.net/article/details/110585930?spm=1001.2101.3001.6650.4&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-4.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-4.pc_relevant_default&utm_relevant_index=7)