> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653558912&idx=1&sn=2f7fb59bc8ea933d1d27f3e3c2300c5f&chksm=81398918b64e000e77b419b9cfae00bd4a24f3e2e7fd79ad4e579131103c11af08e544cb8fc7&scene=21#wechat_redirect)

1 Redis 主从复制原理简介
----------------

*   **全量同步**
    

当主库收到从库的同步请求时，如果从库的复制历史与主库不一致，或者未能在复制积压区中找到从库请求的同步点时，则会与从库进行全量同步。主库通过 fork 子进程产生内存快照，然后将数据序列化为 RDB 格式同步到从库，使从库的数据与主库某一时刻的数据一致。

*   **命令传播**
    

2 Redis 复制缓存区相关问题分析
-------------------

### 2.1 多从库时主库内存占用过多

图 1 Redis 主从复制

如上图所示，对于 Redis 主库，当用户的写请求到达时，主库会将变更命令分别写入所有从库的缓存区（OutputBuffer)，以及复制积压区 (ReplicationBacklog)。需要指出的是，全量同步时依然会执行该逻辑，所以在全量同步阶段经常会触发 client-output-buffer-limit，主库断开与从库的连接，导致主从同步失败，甚至出现循环持续失败的情况。

该实现一个明显的问题是内存占用过多，所有从库的连接在主库上是独立的，也就是说每个从库 OutputBuffer 占用的内存空间也是独立的，那么主从复制消耗的内存就是所有从库缓冲区内存大小之和。如果我们设定从库的 client-output-buffer-limit 为 1GB，如果有三个从库，则在主库上可能会消耗 3GB 的内存用于主从复制。另外，真实环境中从库的数量不是确定的，这也导致 Redis 实例的内存消耗不可控。

### 2.2 OutputBuffer 拷贝和释放的堵塞问题

Redis 为了提升多从库全量复制的效率和减少 fork 产生 RDB 的次数，会尽可能的让多个从库共用一个 RDB，如下代码所示，当已经有一个从库触发 RDB BGSAVE 时，后续需要全量同步的从库会共享这次 BGSAVE 的 RDB，为了从库复制数据的完整性，会将之前从库的 OutputBuffer 拷贝到请求全量同步从库的 OutputBuffer 中。

```
/* To attach this slave, we check that it has at least all the
 * capabilities of the slave that triggered the current BGSAVE. */
if (ln && ((c->slave_capa & slave->slave_capa) == slave->slave_capa)) {
    /* Perfect, the server is already registering differences for
     * another slave. Set the right state, and copy the buffer.
     * We don't copy buffer if clients don't want. */
    if (!(c->flags & CLIENT_REPL_RDBONLY)) copyClientOutputBuffer(c,slave);
    replicationSetupSlaveForFullResync(c,slave->psync_initial_offset);
    serverLog(LL_NOTICE,"Waiting for end of BGSAVE for SYNC");
} else if ( ... )


```

> 注：为避免无效的复制数据积累和拷贝，我实现了 rdb-only 复制 [1] 以更好地支持远程全量备份。

`copyClientOutputBuffer` 看似只是一个简单的 buffer 拷贝，但可能存在堵塞问题，因为 OutputBuffer 链表上的数据可达数百 MB 甚至数 GB 之多，对其拷贝可能使用百毫秒甚至秒级的时间，而且该堵塞问题没法通过日志或者 latency 观察到，但影响却很大。

同样地，当 OutputBuffer 大小触发 limit 限制时，Redis 就是关闭该从库链接，而在释放 OutputBuffer 时，也需要释放数百 MB 甚至数 GB 的数据，其耗时对 Redis 而言也很长。

### 2.3 ReplicationBacklog 的限制

我们知道复制积压缓冲区 ReplicationBacklog 是 Redis 实现部分重同步的基础，如果从库可以进行增量同步，则主库会从 ReplicationBacklog 中拷贝从库缺失的数据到其 OutputBuffer。拷贝的数据最多是 ReplicationBacklog 的大小，为了避免拷贝数据过多的问题，我们通常不会让该值过大，一般百兆左右。但在大容量实例中，为了避免由于主从网络中断导致的全量同步，我们又希望该值大一些，这就存在矛盾了。

此外，我们在重新设置 ReplicationBacklog 大小时，会导致 ReplicationBacklog 中的内容全部清空，所以如果在变更该配置期间发生主从断链重连，则很有可能导致全量同步。

> 注: Redis 存在重复拷贝 ReplicationBacklog 的问题， 避免 ReplicationBacklog 拷贝的 PR[2] 已修复。

3 共享复制缓存区的设计与实现
---------------

### 3.1 方案简述

```
/* Similar with 'clientReplyBlock', it is used for shared buffers between
 * all replica clients and ReplicationBacklog. */
typedef struct replBufBlock {
    int refcount;           /* Number of replicas or repl backlog using. */
    long long id;           /* The unique incremental number. */
    long long repl_offset;  /* Start replication offset of the block. */
    size_t size, used;
    char buf[];
} replBufBlock;


```

*   `refcount`：block 的引用计数
    
*   `id`：block 的唯一标识，单调递增的数值
    
*   `repl_offset`：block 开始的复制偏移
    

![](https://mmbiz.qpic.cn/mmbiz_png/cJMXkxTQicjueTI1ZSGdZ9WAOibVhBQqic0c6e42jXdDuzs7GHfCKOLHJ9enBlicDAQGPzzS5kvmsiayepMer9tEruQ/640?wx_fmt=png)  

图 2 ReplicationBacklog 和 从库对 ReplicationBuffer 的引用描述

**2.1 提到的多从库消耗内存过多的问题通过共享复制缓存区方案得到了解决，对于 2.2 和 2.3 提出的问题是否解决了呢？**

```
typedef struct client {
    ...
    listNode *ref_repl_buf_node; /* Referenced node of replication buffer blocks,
                                  * see the definition of replBufBlock. */
    size_t ref_block_pos;        /* Access position of referenced buffer block,
                                  * i.e. the next offset to send. */
    ...
} client;


```

首先来看 2.2 中的问题， **OutputBuffer 拷贝**问题，当前从库的 OutputBuffer 的描述只有对共享 ReplicationBuffer 的引用信息，如上代码所示，所以之前的数据深拷贝变成了更新引用信息，即对正在使用的 replBufBlock refcount 加 1，这仅仅是一条简单的赋值操作，非常轻量。**OutputBuffer 释放**问题呢？在当前的方案中释放从库 OutputBuffer 就变成了对其正在使用的 replBufBlock refcount 减 1，也是一条赋值操作，不会有任何阻塞。

```
/* Replication backlog is not separate memory, it just is one consumer of
 * the global replication buffer. This structure records the reference of
 * replication buffers. Since the replication buffer block list may be very long,
 * it would cost much time to search replication offset on partial resync, so
 * we use one rax tree to index some blocks every REPL_BACKLOG_INDEX_PER_BLOCKS
 * to make searching offset from replication buffer blocks list faster. */
typedef struct replBacklog {
    listNode *ref_repl_buf_node; /* Referenced node of replication buffer blocks,
                                  * see the definition of replBufBlock. */
    size_t unindexed_count;      /* The count from last creating index block. */
    rax *blocks_index;           /* The index of reocrded blocks of replication
                                  * buffer for quickly searching replication
                                  * offset on partial resynchronization. */
    long long histlen;           /* Backlog actual data length */
    long long offset;            /* Replication "master offset" of first
                                  * byte in the ReplicationBacklog buffer.*/
} replBacklog;


```

### 3.2 关键问题

*   **ReplicationBuffer 的裁剪和释放**
    

*   当从库使用完当前的 replBufBlock 会对其 refcount 减 1
    
*   当从库断开链接时会对正在引用的 replBufBlock refcount 减 1，无论是因为超过 client-output-buffer-limit 导致的断开还是网络原因导致的断开
    
*   当 ReplicationBacklog 引用的 replBufBlock 数据量超过设置的该值大小时，会对正在引用的 replBufBlock refcount 减 1，以尝试释放内存
    

> > 注：一般而言，设置的从库 client-output-buffer-limit 会远大于 ReplicationBacklog 大小，所以如果有一个慢从库节点，则其引用的 ReplicationBuffer 将大于 ReplicationBacklog，但是为了尽可能的进行部分重同步（PSYNC)，这种情况下 ReplicationBacklog 不会尝试释放 replBufBlock，从而隐式地将 ReplicationBacklog 变大，以便尽可能支持部分重同步。

*   **ReplicatonBacklog 和从库只对正在使用的 replBufBlock 增减 refcount 的原因**
    

首先 replBufBlock 通过链表维护，天然具有有序性，也保证了 ReplicationBuffer 的正确性，其次，与我们上述提到的释放策略有关，我们只释放链表**第一个** refcount 为 0 的 replBufBlock，若不为 0 则**终止**对 replBufBlock 链表的遍历从而避免对中间 refcount 为 0 的 replBufBlock 的释放。

*   **ReplicatonBacklog 使用 ReplicationBuffer 的问题和解决方案**
    

当从库尝试与主库进行增量重同步时，会发送自己的 repl_offset, 若从库可满足增量重同步条件，则主库会从 ReplicationBacklog 中拷贝从库缺失的数据到从库 OutputBuffer，虽然现在的拷贝变成了仅仅是对特定 replBufBlock 引用计数的改变，在每个 replBufBlock 中记录了该其第一个字节对应的 repl_offset，但如何高效地从数万个 replBufBlock 的链表中找到特定的那个, 仍然一件棘手的事情。

![](https://mmbiz.qpic.cn/mmbiz_png/cJMXkxTQicjueTI1ZSGdZ9WAOibVhBQqic00uhVuogU23OPpjWnIAUa3Bh9C14iaKyFYo34ibpDFKAgeEAvd1MkEh8Q/640?wx_fmt=png)

图 3 查找包含特定 repl_offset 的 replBufBlock

直接从头到位遍历链表查找对应的 replBufBlock 必然会耗费较多时间而堵塞服务。起初，我额外使用了一个链表用于索引固定区间间隔的 replBufBlock，每 1000 个 replBufBlock 记录一个索引信息，当查找 repl_offset 时，会先从索引链表中查起，然后再查找 replBufBlock 链表，类似于跳表的查找实现，如上图所示。但这个方案在极端场景下可能会查找超过千次，有 10 毫秒以上的延迟，与 Redis 的维护者 Oran Agra 讨论后，我们对此仍不满足，遂放弃。

最终使用 rax 树实现了对 replBufBlock 固定区间间隔的索引，每 64 个记录一个索引点。一方面，rax 索引占用的内存较少；另一方面，查询效率也是非常高，理论上查找比较次数不会超过 100，耗时在 1 毫秒以内。

### 3.3 其他相关问题

*   **mem_total_replication_buffers**
    

在 Redis 的 INFO 命令输出中增加了 `mem_total_replication_buffers` 字段，用以展示全局复制缓存区 ReplicationBuffer 的大小。因共享复制缓冲区，只有所有从库引用的复制数据超过 ReplicationBacklog 时，我们才认为超过的内存是从库消耗的内存，即 `mem_clients_slaves` 字段的数值，否则 `mem_clients_slaves` 为 0。

*   **Key 驱逐**
    

在 Redis 进行 key 驱逐时, 从库的 OutputBuffer 内存是不算入 Redis 的内存消耗，因共享复制缓冲区机制，我们认为只有超出 ReplicationBacklog 大小的部分是从库额外的内存消耗，也就是说 ReplicationBacklog 消耗的内存仍然会导致 key 的驱逐。

*   **IO 多线程**
    

由于所有从库和 ReplicationBacklog 使用全局复制缓冲区，如果启用 IO 多线程，为了保证数据访问线程安全，我们必须让主线程处理发送数据到所有从库的工作。但在此之前，其他 IO 线程可以处理从库的发送 OutputBuffer 工作，无差别对待普通客户端和从库客户端。

4 总结
----

本文主要分析了 Redis 主从复制缓存区存在的几个问题：内存占用过多，OutputBuffer 拷贝和释放造成的不易察觉的堵塞问题，以及 ReplicaitonBacklog 的不足，然后提出了共享复制缓存区的方案，并分析了该方案的几个关键问题。希望该方案能够减少大家在运维 Redis 过程中遇到的主从复制问题，降低内存消耗，避免线上稳定性问题，Cheers!

事实上，本文主要是对 PR: Replication backlog and replicas use one global shared replication buffer[3] 动机和实现的中文描述。也感谢 sundb 和 zhaozhao.zz 对这个 PR 的 BugFix[4] 和 索引优化 [5]。如有问题，欢迎大家指正！

当前，笔者在百度负责 Redis 的研发工作，是 Redis 社区的 Member，持续跟进 Redis 的优化工作，也会跟大家分享 Redis 中一些有趣的设计。同时本人也是[磁盘 KV 存储服务 Kvrocks](http://mp.weixin.qq.com/s?__biz=MzUxNTg5NzM1Nw==&mid=2247483681&idx=1&sn=e05dbf64f71a44977b38ba64a5bb9e8f&chksm=f9aee043ced969555aa489067c612c44c2ce4ac47b03587afb516fc5ed7f6f57cf921036749d&scene=21#wechat_redirect) 的维护者，该项目在快速发展阶段，欢迎大家关注。

参考
--

[1]

Implement rdb-only replication: https://github.com/redis/redis/pull/8303

[2]

Remove unnecessary replication backlog copy: https://github.com/redis/redis/pull/9157

[3]

Replication backlog and replicas use one global shared replication buffer: https://github.com/redis/redis/pull/9166

[4]

Fix not updating backlog histlen: https://github.com/redis/redis/pull/9713

[5]

Rebuild replication backlog index: https://github.com/redis/redis/pull/9720