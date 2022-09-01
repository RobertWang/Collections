> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7138062440708702222)

当对数字 append 了字符串之后，type 会变为 raw

如果字符串对象中存储的是字符串，并且**字符串长度大于 32 字节，则用 SDS 来保存，类型为 raw**，需要调用 2 次内存分配分别为 redisObject 和 sdshdr，释放的时候也需要释放两次

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f92f79b3dfb043e0aed79509109d3774~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

为了处理字符串在动态新增导致的可能需要重新申请内存并且复制的问题，在申请的时候回适当的多申请一些空间，在扩容的时候也会适当的多扩容一些空间用 free 记录，同时还预留了方法可以清除 free 空间

如果字符串小于等于 32 字节，采用 embtr 编码来保存，embtr 相比 raw 只需要进行一次内存申请即可，也只需一次内存释放，这样有利于 cpu 高速缓存

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c03146c407d7464894f54d38f7dd9634~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

### 列表对象

列表对象的编码方式可以为 ziplist、linkedlist

采用 ziplist 的优势是更加节省内存，因为在 linkedlist 中每个 StringObject 依据上一章说的 string 来说都会各位占据一些字节来

（1）ziplist

列表中存储的所有字符串长度都小于 64 字节，列表对象保存的元素小于 512 个，就会采用 ziplist 进行编码

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a177528b3ddd4d02990a64cc1fb1c87c~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

> list-max-ziplist-value、list-max-ziplist-entries 可以用于修改这 2 个上限值，当后续新增的元素导致不满足 ziplist 条件时候会自动转码为 linkedlist 将数据移入到双端列表中

（2）linkedlist linkedlist 底层采用了双端列表来实现

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2492bcafff224a95b6064b2282018058~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

### hash 对象

哈希对象的编码也可以是 ziplist 或者 hashtable

（1）ziplist

当 hash 对象保存的所有键值对都小于 64 字节、hash 对象键值对数量小于 512 个就会采用 ziplist 进行存储

> hash-max-ziplist-value、hash-max-ziplist-entries 可以用于修改这 2 个上限值，当新增元素后不满足条件就会进行转码为 hashtable 将数据移入 hashtable 中

将 hash 表的 key 添加到值的末尾后再将 hash 表的 value 添加到值末尾，这样 key 和 value 总是挨在一起的

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/973b7ace62b74a86b4e4dd51dc391c9c~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

（2）hashtable

当不满足 ziplist 条件时候就会存储为 hashtable

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bfece167e82c4efeb219f5b3f7b2fb8f~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

### set

set 是不重复的集合列表，编码类型可以是 intset、hashtable

当满足集合对象所有元素都是整数值、元素数量不超过 512 个，就采用 intset 进行存储，可以节省内存，可以通过 set-max-intset-entries 修改 512 这个上限值

当不满足条件的时候采用 hashtable 进行存储，hash 表存储是为了保证元素不重复，值存储 null 值

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/129eff081bcd4965a2a836c7bc70cd42~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

### sorted set

有序集合的编码方式为 ziplist、skiplist

压缩列表的实现方式和之前说的一致，当元素数量小于 128 个、所有元素长度小于 64 字节，满足该条件的就是要 ziplist 编码

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7683df3b7d974a95b41543ed5957a4b3~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/af5f8093929f4f3291c918ef1f76db2d~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

否则使用 skiplist 来实现，skiplist 依赖的底层的数据结构为跳表，跳表具有基于 key 值快速检索，排序，范围检索等诸多特性

高性能
---

保证高性能需要从以下几个点来处理

（1）网络：reactor 模式、epoll

（2）线程池与线程：减少新增和销毁线程的开销，减少线程切换导致的上下文和 CPU 开销

（3）磁盘读写：PageCache、零拷贝、SSD 硬盘

（4）同步与锁：减少锁粒度比如利用读写锁减少互斥粒度、使用无锁提升性能等机制

**redis 目前是单 reactor 机制**

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/939f26504ea344758c2bad950af5136d~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4ff147fe31d439ba44aa1c4207776a0~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

简单的阐述一下工作机制

（1）redis 启动后绑定一个端口后，将 socket 注册到 selector 监听器中，关注连接建立事件

（2）连接事件到达后，三次握手完成 accept() 从内核全连接队列中取出就绪的 socket 进行保存，然后交给连接应答处理器，将其注册到 IO 多路复用器中关注 OP_READ 事件

（3）当产生 OP_READ 事件后，然后将其压入队列，等待命令请求处理器处理

（3）命令请求处理器会执行命令，比如 get 数据完毕后暂存数据，将当前 socket 关注 OP_WRITE 事件，IO 多路复用感知到 OP_WRITE 后将其压入队列等待命令回复处理器处理

（4）命令回复处理器获取到 socket 事件后将数据写入到对应的 socket 缓冲区后，将其再次注册 OP_READ 事件到 IO 多路复用程序中，等待下一次 READ 事件

整体来看单线程是否非常忙碌，但是所有的操作都是纯内存操作，同时没有上下文切换的开销

DDR4 内存读速度大概 50G 每秒，以每个指令读取 300 字节来算，每秒能够读取 1.7 亿次，当然开销不仅仅是内存读，还有写 CPU 刷盘等等开销，但是内存读不是瓶颈，正是因为单线程模式没有办法充分的利用内存性能，所以在 redis6.x 开始支持多 reactor 模式

过期的 key 如何处理
------------

我们最好仅仅把 redis 当做一个缓存系统，所以尽量为每一个 key 都设置过期时间，同时做好没有获取到缓存的兜底工作，那么 key 过期了有哪些删除策略呢

### 定时删除

设置键过期时间同时创建定时器，定时器在键过期时候将其删除

优点：及时释放内存空间

缺点：过期键较多的时候占用 CPU 时间导致在内存充足的情况下，浪费 CPU 执行时间

### 惰性删除

每次 get 的时候如果发现过期了就将其删除

优点：对 CPU 友好

缺点：对内存不友好，如果有些 key 没有被 get 就会浪费大量内存空间，可能导致内存泄露

### 定期删除

系统每隔一段时间，就对数据库执行一次过期键检查，根据算法来批量删除过期键

每隔一段时间执行一次扫描过期 key 和删除操作，并且限制删除占用的时间

### redis 的过期删除策略

redis 采用了惰性删除 + 定期删除策略配合使用

定期删除：每隔 100MS 随机挑选 20（可配置）个 key，删除其中已经过期的 key，如果其中 25% 都过期了则重复执行挑选删除动作，直到删除的 key 小于 25% 才停止

内存写满了怎么办
--------

redis 作为一个缓存系统，是会存在内存写满的情况，这个时候需要根据淘汰策略，淘汰掉一部分不常用的非热点数据

当配置缓存淘汰策略为 noevction 的时候表明，缓存写满了不进行淘汰，直接抛错 OOM

淘汰策略大体上分为 2 大类，分别是

（1）从指定了过期时间的 key 中进行淘汰

（2）从所有的 key 中进行淘汰：如果有依赖 redis 作为持久化存储，那么不能选用这种策略，因为会删除掉不过期的 key

从算法的实现上进行分类 lru、random、lfu

**random 就是**根据所选分类在所有的 key 或者指定了过期时间的 key 进行随机删除

**lru 算法**大家应该熟悉，需要一个链表来维护所有数据，访问了某个 key 就将其移到头部，满了后就从尾部进行淘汰，不过 redis 如果选用这种策略维护所有的 key 为一个链表那么代价太高了，所以进行了改造，随机选取 5 个组成一个集合，删除其中访问时间最小的数据，下次随机选取 5 个 merge 到这个集合中，再次淘汰最小的数据 （**redisObject 中会有一个字段 lru 来维护当前的访问时间**）

**lfu 算法**将 redisObject 中的 lru 进行拆分前 16 位维护时间戳，后 8 位维护访问次数，淘汰的时候优先淘汰访问次数小的，访问次数一致淘汰时间戳小的

> 8 位最多存储到 2^8 - 1 位数也就是 255，所以系统需要根据一定的算法来进行增长，而不是每次访问就 + 1，当某个 key 一致没有被访问的时候，会按照衰减算法降低访问次数

redis3.0 之前默认的淘汰策略为 volatile-lru、3.0 之后默认的淘汰策略为 volatile-lru（从指定了过期时间中选择 lru 淘汰算法）

如何处理客户端请求
---------

在高性能章节中基于下图说明了处理请求的工作原理 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4ff147fe31d439ba44aa1c4207776a0~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?) 每个客户端与 redis 建立好了连接之后，会为当前连接分配接收缓冲区和发送缓冲区，由于是单线程的 reactor 模型，当客户端一次请求需要返回非常大的数据量的时候，客户端又没有办法读取的时候，导致当前缓冲区被写满会阻塞主线程（概率很低）会导致所有的请求阻塞

需要警惕发送缓冲区被写满的情况

在响应的时候只要数据写入 tcp 缓冲区成功就认为返回成功，开始处理下一个指令

master-slave（提高吞吐量、数据容灾备份）
--------------------------

单台 redis 存在单点故障问题，如果是磁盘损坏了还会导致数据丢失，如果是当前机器故障无法重启则需要启用另外一台具有同样的数据 redis 服务器

单台 redis 是会存在读写瓶颈的，可以将读请求 LB 到 Slave 中提升 QPS

读写分离架构：Slave 中只能进行读请求，写请求需要到 Master 执行

### slave 如何同步 master 数据

当启动 slave 节点挂载 master 节点之后，在 master 端会执行 bgsave 生成一份 RDB 文件同步给 slave 节点

同时由于 master 还在不断的响应客户端请求，为了保持 master 和 slave 数据一致，为会每个 slave 建立一个**复制缓冲区**，记录生成 RDB 期间的变更记录，当 RDB 执行完成 RDB 之后会从复制缓冲区中取出数据来执行，后续 master 上的增量变更都会通过复制缓冲区发送给 slave

> 如果复制缓冲区被写满了，则会中断连接释放资源，所以需要合理设置缓冲区大小，也需要合理的设置单个 redis 实例的大小（也就是为什么需要将大小限制在 <= 6GB 或者 <= 10GB 的原因之一）

### slave 网络闪断怎么办

聊网络闪断之前我们先来说说**复制积压缓冲区**，redis 会维护一个复制积压缓冲区，这是一块指定内存大小的区域是一个环装结构，其中会记录每一条写入指令，当写满之后又会冲头开始覆盖写入，也就是只是**记录最新一段时间写入的指令**

如果 slave 网络闪断，连接被释放如何处理，此时 slave 会不断重试与 master 再次建立连接，建立成功，master 端会再次创建复制缓冲区，然后 slave 将当前同步到的最新偏移量发送到 master，master 会去检查**复制积压缓冲区**，如果在其中找到了，那么就会将这个偏移量到最新的数据发送给 slave 完成之后，slave 基于最新的偏移量读取复制缓冲区发送过来的数据就再次回到最新的情况

如果 slave 宕机比较久，连接 master 成功之后在复制积压缓冲区没有找到偏移量，则会重新执行 上面说的 slave 到 master 的同步流程

### slave 挂载太多会有什么问题

当 slave 挂载过多，自然会导致多次 bgsave rdb 持久化，多次数据同步等等，给 master 造成一定的压力，所以需要多 slave 的情况下，大都会选择树形挂载

master 挂载 slave-1、slave-2，slave-1 挂载 slave-1-1、slave-1-2 依次类推

sentinel - 高可用保证
----------------

sentinel 其实就是一个运行在特殊模式下的 redis 服务器

sentinel 会与每个 redis 实例建立连接，通过心跳感知到下线的服务后，然后选举 slave 为 master，同时监测下线服务器，当其再次上线之后挂载到集群作为 slave

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/52b1215cdad04b9b8d619ef4f1d83681~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

sentinel 在启动会向每个被监视的 redis 服务器建立 2 个连接

（1）创建一个用户发送 ping、info 等指令的连接

（2）创建一个用户订阅 sentinel:hello 的连接，用于感知到其它加入的 sentinel 节点

**sentinel 每隔 10S 向被监视的 master redis 服务器发送 info 指令**，用来获取 master 旗下有哪些 redis 以及 redis id 等信息

**sentinel 每隔 10S 想被监视的 slave redis 服务器发送 info 指令**，用于获取 salve 相关元数据信息来对 sentinel 保存的数据结构进行更新

**sentinel 每隔 2S 通过订阅连接向 sentinel:hello 发送一条消息**（sentinel 自身信息、监听的 master 和 slave 信息），其它的 sentinel 会通过订阅收到这条消息后就能感知到新的 sentinel、以及让整个 sentinel 集群保持信息的一致

**sentinel 每隔 1S 向所有的 redis master salve 实例、sentinel 实例发送 ping 请求**来感知对方的服务是否可用

（1）如果对方在 down-after-milliseconds 时间内连续响应了 ping 无效的响应就，当前 sentinel 就将其标记为主观下线（我们最好保证所有节点 down-after-milliseconds 值保持一致，才便于产生一致的行为）

（2）当前 sentinel 询问其它 sentinel 对于当前宕机节点的判断是否为下线，如果得到的结果下线的判定数量达到了 quorum 数量（可配置），就标记当前节点为宕机节点为客观下线，并且在 sentinel 集群中同步下线节点信息

（3）选举一个 sentinel 为 master 节点来负责 slave 选举（根据 raft 算法选举）

```
- 每个发现 redis 节点宕机的 sentinel 都会将自己选举为 leader，并且将选票发送给其它 sentinel
- 由于网络等等情况，总会有 sentinel 先执行，如果 sentinel 还没有发出选票此时收到了对方发来的选票就会选举对方，后面就算自己通过心跳感知到 redis 宕机后也会发起选举自己的行为，这样就能选举出来 leader
- 如果恰好同时多个 sentinel 同时打成一直发出自己成为 leader，此时每个节点会随机休眠一会再次发起选票
- 不伦选举是否成功，每次选举之后 leader_epoch + 1
- 如果某个 sentinel 收到了过半的 ack 就将当前 sentinel 作为 leader 去负责宕机 slave 选举为 leader 的操作
复制代码

```

提升 slave 为 master 过程为，断开所有 slave 与 master 的联系，然后基于一个判断规则来选取提升哪一个 slave 为 master，优先级按照，配置的优先级 -> 同步的最新偏移量 -> id 小的得分高规则来选举，选举成功后，在 salve 中执行 slave of 新 master 节点即可

cluster - 高可用保证
---------------

### cluster 集群构建

redis cluster 由每个 redis 节点组成，不会有额外的服务器节点，redis 节点启动时候会根据 cluster-enabled 配置来确定是否开启集群模式

每个 redis 节点通过向另外一个 redis 节点发送 CLUSTER MEET 指令就可以让对方加入到当前集群中

（1）A 向 B 发送 MEET 消息

（2）B 收到消息后响应 PONG 消息

（3）A 收到 PONG 消息后，A 向 B 发送 PING 消息

（4）B 收到 PING 消息，握手完成，节点 B 加入节点 A 所在集群

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/07f876590fbd4afdae687ed19e76a0e9~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

redis 集群结点的行为与单机节点的行为保持一致，除此之外会通过 clusterCron 函数执行集群模式下的行为比如发送 gossip、检查节点是否下线、是否需要进行故障转移等行为，每次指令都需要压入队列中排队最后由**时间事件处理器来执行**

当通过 MEET 消息所有节点加入集群完成之后，集群任然处于下线状态，此时就会开始 slot 的分配

### slot 分配

redis 的整个数据库槽位一共有 1638 个，每个槽位都分配到对应的节点之后，集群会处于上线状态，默认情况下 slot 信息会均匀分配给每个 redis 节点

slot 分配完毕之后，每个集群中的 redis 之间会互相传播各自负责的 slot 信息用于达成一致每个节点都会记录完整的 slot 与 redis 节点的映射关系

### 处理客户端发送的指令

slot 分配完毕之后，客户端接入 cluster 集群之后，会将 slots 与 redis 之间的映射关系存储在本地

客户端执行 set msg "hello" 后会根据 crc16 算法计算出来 key: msg 对应的 slot 值，然后检查 slot 在哪一个区间，区间对应了哪一个 redis 节点，然后向其发送指令

redis 计算出键对应的槽位在当前服务器则执行执行，如果不在则返回 **MOVED** 异常，指引客户端转向到指定节点，同时会更新客户端映射缓存（在集成好的客户端这些是自动执行的）

### 集群结点扩容

当集群中新加入一台节点之后，需要进行 slot 的重新分配，此时会迁移节点的一部分数据到另外的节点中，如果此时客户端发送请求指令，到达 redis 服务端后，计算 crc16 得到槽位信息，发现在本地，现在本地查找 key 如果找到了就返回，如果没有招到则返回 ASK 异常，指引客户端重定向请求到新的节点查询（需要注意这里是不会更新客户端 slot 映射 redis 缓存的，因为迁移还没有完成处于不稳定状态）

### 故障转移

在 cluster 架构下，每个 master redis 节点可以挂载 slave 节点

当 master 宕机之后可以选举 slave 作为新的 master

集群中主节点之间会互相发送 ping 消息来感知对方是否在线，如果对方在规定的时间内存一直没有返回 pong 消息，则将其标记为**疑似下线**

redis 集群之间会互相发送消息交换彼此的信息，当发现过半节点都将其标记为疑似下线后会将其标记为**已下线**，同时在集群中广播

当一个从节点发现主节点宕机后广播一条群消息要求进行选举，集群 master 会进行响应，master 只会响应给收到的第一个请求 ack

从节点检查自身收到了过半节点的 ack 后成为 master 节点

新选举出来的 master 节点执行 slaveof no one 成为 master 节点，然后将宕机的 redis 的 slot 指派给自己，随后向集群广播 PONG 消息附带自身信息，成功加入集群，开始处理请求

有哪些阻塞点
------

知道了一些读写工作原理和底层数据结构之后，我们来看看存在哪些阻塞点

### AOF 和 RDB

他们在进行重写或者会 fork 子进程这个过程是阻塞的，内存页，内存越大，fork 的内存页表越大，10GB 需要复制 20MB 大小

同时写入需要基于快照来写，过程中如果客户端有写入，会基于写时复制技术，复制数据所在内存页，所以值越大，需要复制的内存越大，如果开启了内存大页（1 个内存页为 2M），再加上大量写入的话，会造成长时间阻塞（因为他们都是由主线程负责的）

### 挂载 slave

每次挂载 slave 都会导致 master 执行 bgsave fork 子进程复制内存页表，也会基于写时复制技术复制变更的数据页，这 2 个点都由主线程执行，都会导致阻塞

### 大 key

大 key 不仅在写时复制的时候导致耗时更长，也会增加读取的时间，写入缓冲区如果数据太大了导致写满了，也会导致阻塞主线程

### 复杂的指令

主线程通过命令执行器如果执行时间复杂度过高的指令，也会导致阻塞

### redis cluster 新增节点数据迁移

当新增 redis 节点之后，slot 重新分配后，数据需要进行迁移，迁移都是同步进行的由 redis 主线程进行执行和客户端请求指令交替执行，这也会造成一定降低 redis 吞吐量，而 coding 是支持异步迁移的

参考：

*   redis 设计与实现