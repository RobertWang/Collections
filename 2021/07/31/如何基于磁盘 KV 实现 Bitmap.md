> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653556448&idx=1&sn=bf9966923270b667cca235d95b869103&chksm=81399f78b64e166e73a448f3f4ab85a79194f5a886c1a0d0c6e7c2090a42622d3628e3134f40&mpshare=1&scene=1&srcid=0727jEjutPb8eSt4lxEU8bTX&sharer_sharetime=1627373521718&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

大部分开发对 Bitmap 应该都不陌生，除了作为 Bloom Filter 实现的存储之外，许多数据库也有提供 Bitmap 类型的索引。对于内存型的存储来说，Bitmap 只是一个特殊类型 (bit) 的稀疏数组，操作内存不会带来读写放大问题(指的是物理读写的数据量远大于逻辑的数据量), Redis 就是在字符串类型上支持 bit 的相关操作，而对于 Kvrocks 这种基于磁盘 KV 实现的存储则会是比较大挑战，本篇文章主要讨论的其实是「**基于磁盘 KV 实现 Bitmap 要如何减少磁盘读写放大**」

**为什么会产生读写放大**
--------------

读写放大的主要是来源于两方面:

*   硬件层面的最小读写单元
    
*   软件层面存储组织方式
    

硬件层面一般是由于最小读写单元带来的读写放大，以 SSD 为例，读写的最小单位是页 (一般是 4KiB/8KiB/16KiB)。即使应用层只写入一个字节，在磁盘上实际会写入一个页，这也就是我们所说的写放大，反之读也是一样。另外，SSD 修改数据不是在页内位置原地修改而是 Read-Modify-Write 的方式，修改时需要将原来的数据读出来，修改之后再写到新页，老的磁盘页由 GC 进行回收。所以，即使对同一页的一小块数据反复修改也会由于硬件本身机制而造成写放大。类似于如下:

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapONsKF2yqcsTpmmZgSiajbNs3hebleXfmsW4z2NJLp9Ux44Q9GH1QaIgxvaNEp8poFlwmsMqkXKX1iag/640?wx_fmt=png)

由此可见，大量随机小 io 读写对于 SSD 磁盘来说是很不友好的，除了在性能方面有比较大的影响之外，频繁擦写也会严重导致 SSD 的寿命 (随机读写对 HDD 同样不友好，需要不断寻道和寻址)。LSM-Tree 就是通过将随机写入变成顺序批量写入来缓解这类问题。

软件层面的读写放大主要来自于数据组织方式，不同的存储组织方式所带来的读写放大程度也会有很大的差异。这里以 RocksDB 为例，RocksDB 是 Facebook 基于 Google LevelDB 之上实现了多线程，Backup 以及 Compaction 等诸多很实用的功能。RocksDB 的数据组织方式是 LSM-Tree，在解决磁盘写入方法问题，本身的数据存储也带来了一些空间放大问题。下面可以简单看一下 LSM-Tree 是如何组织数据:

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapONsKF2yqcsTpmmZgSiajbNs3FU7ZicGXjJtTaQaNvOicpmR5ZgVrEYk3eibvQ6ic69XHw7IutVm79v5GQw/640?wx_fmt=png)

LSM-Tree 每次写入都会产生一条记录，比如上图 x 先后写了 4 次，分别是 0，1，2，3。如果单看 x 这个变量，这里相当于有 4 倍的空间放大，这些重复的记录会在 compaction 的时候进行回收。同样，删除也是通过插入一条 value 为空的记录来实现。LSM-Tree 每一层空间大小是逐层递增，当容量大小当层最大时会触发 compaction 合并到下一层，以此类推。假设 Level 0 最大存储大小是 M Bytes，逐层按照 10 倍增长且最大 7 层，理论上空间放大的大约是 1.111111 倍。计算公式如下:

```
空间放大率 = (1 + 10 + 100 +1000 + 10000 + 100000 + 1000000) * M / (1000000 * M)
```

但在实际场景中，由于最后一层一般无法达到最大值，所以放大空间率比这个理论值大不少，具体在 RocksDB 的文档里面也有提过，具体见: https://rocksdb.org/blog/2015/07/23/dynamic-level.html

另外，由于 RocksDB 读写都是以 KV 为单位，Value 越大带来的读写放大就可能越大。举个例子，假设有一个 Value 为 10 MiB 的 JSON，如果要修改这个 key 中的一个字段，那么需要把整个 JSON 读出来，修改后再重新写回去，就会导致巨大的读写放大。有一篇 paper「WiscKey: Separating Keys from Values in SSD-conscious Storage」就是通过 Key/Value 分离的方式来优化 LSM-Tree 大 KV 的来减少 Compaction 时带来写放大的问题。TiKV 里面的 titan 就是基于 Wiskey 论文优化 RocksDB 在大 KV 场景的写放大问题，RocksDB 也在社区版本里面实现这个功能，不过还是实验性的阶段。

**基于磁盘 KV 实现 Bitmap**
---------------------

Kvrocks 是基于 RocksDB 之上实现的兼容 Redis 协议的磁盘存储， 需要支持 Bitmap 功能，所以就需要在磁盘 KV 之上实现 Bitmap 的功能。而大部分使用 Bitmap 的场景都是作为稀疏数组来用，意味着第一次写入的 offset 为 1，下次的 offset 可能就是 1000000000 甚至更大，所以在实现 Bitmap 就会面临上述读写和空间放大问题。

一种最简单的实现方式是仍然把整个 Bitmap 作为一个 Value，读写时将 Value 读取到内存中再回写。这种实现虽然很简单，但一不小心可能导致 value 巨大，单个 Value 大小上 GiB 都是可能的。除了存在有效空间利用率问题之外，可能会直接导致整个服务不可用 (需要读写整个 Value)。Pika 里面的 Bitmap 就是这种实现，但限制最大的 Value 为 128 KiB，限制 Value 大小虽然避免上述的极端情况，但会大大限制 Bitmap 的使用场景，甚至是无法使用。

既然知道核心问题是由于单个 KV 过大导致， 那么最直接的方式就是将 Bitmap 拆分成多个 KV，然后控制单个 KV 大小在合理范围之内， 那么读写带来的放大也是相对可控。在当前 Kvrocks 的实现里面是按照每个 KV 为 1 KiB 来划分，相当于每个 value 可以存放 8192 bits。算法示意图如下:

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapONsKF2yqcsTpmmZgSiajbNs36J7xIWBfYwicUCWucCSuGyUaXafw19ExHwmPMFSz5kLAibQRBkDheytA/640?wx_fmt=png)

以 setbit foo 8192002 1 为例，实现的步骤如下:

1.  计算 8192002 这个 offset 对应所在的 key, 因为 Kvrocks 是按照 1 KiB 一个 value，那么所在 key 的编号就是 8192002/(1024*8) = 1000，所以就可以知道这个 offset 应该写到 "foo" + 1000 这个 key 对应的 value 里面
    
2.  接着从 RocksDB 里面去获取这个 key 对应的 value
    
3.  计算这个 offset 在分段里面的偏移，8192002%8291 等于 2，然后把 value 中偏移为 2 的 bit 位设置为 1
    
4.  最后将 value 回写到 RocksDB
    

这种实现比较关键的一个特点是 Bitmap 对应的 KV 只在有写入的时候才会真正写到 RocksDB 里面。假设我们只执行过两次 setbit ，分别是 setbit foo 1 1 和 setbit foo 8192002 1 ，那么 RocksDB 里面只会有 foo:0 和 foo:1000 这两个 key，实际的写入 KV 总共也只有 2 KiB。刚好也可以完美适应 Bitmap 这种稀疏数组的场景，不会因为稀疏写入而带来空间放大的问题。

**这个想法也和 Linux 的虚拟内存 / 物理内存映射策略类似，比如我们 malloc 申请了 1GiB 的内存，操作系统也只是分配一片虚拟内存地址空间，只有在真正写入的时候才会触发缺页中断去分配物理内存 (目前正常内存页大小是 4KiB)。也就是如果内存页没有被写过，只读也不会产生物理内存分配。**

GetBit 也是类似，先计算 offset 所在的 key，然后从 RocksDB 读取这个 key, 如果不存在则说明这段没有被写过，直接返回 0。如果存在则读取 Value，返回对应 bit 的值。另外，在实现上也单个 KV 实际存储大小也是由目前写入最大的 offset 决定，并不是有写入就会分配 1024 KiB，这样也可以一定程度优化单个 KV 内的读写放大问题。实现可参考: https://github.com/KvrocksLabs/kvrocks/blob/unstable/src/redis_bitmap.cc

**总结**
------

可以看到基于内存和磁盘之上去实现同一个功能，除了不同类型存储介质本身的速度差异之外，问题和挑战是完全不一样的。对于磁盘类型的服务，需要不断去优化随机读写和空间放大问题，除了对于软件本身熟悉之外，同样需要了解硬件设备。

另外，Kvrocks 作为基于磁盘 KV 之上兼容 Redis 协议存储服务，最经常被问到是跟其他功能类似的服务有什么区别？简单来说，最大的差异在于不同项目维护者在功能设计上的差异，不同设计会让功能看似一样的服务在表现上完全不一样。所以，最好的方式就是通过代码去了解项目的设计和问题。

**参考连接：**

[1]  https://rocksdb.org/blog/2015/07/23/dynamic-level.html

[2] https://www.usenix.org/system/files/conference/fast16/fast16-papers-lu.pdf

[3] https://github.com/KvrocksLabs/kvrocks

[4] https://github.com/facebook/rocksdb

[5] https://github.com/tikv/titan