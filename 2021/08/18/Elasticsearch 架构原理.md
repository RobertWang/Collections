> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA3OTc0MzY1Mg==&mid=2247514876&idx=3&sn=faba4915c817c6d6ecdeaf0374baccfa&chksm=9fac2537a8dbac213cb0e26709005689c6ea99e78c9743448daca64af2c809e52d91b1a852f2&mpshare=1&scene=1&srcid=0817fPldGuX4N41fBcFGHRlS&sharer_sharetime=1629168655740&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

[来源：《ELKstack 权威指南》](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)

*   **架构原理**
    
*   **带着问题学习**
    
*   **segment、buffer 和 translog 对实时性的影响**
    

*   **动态更新的 Lucene 索引**
    
*   **translog 提供的磁盘同步控制**
    
*   **Elasticsearch 分布式索引**
    

*   **segment merge 对写入性能的影响**
    

*   **归并线程配置**
    
*   **归并策略**
    
*   **forcemerge 接口**
    

*   **routing 和 replica 的读写过程**
    

*   **路由计算**
    
*   **副本一致性**
    

*   **shard 的 allocate 控制**
    

*   **reroute 接口**
    
*   **分配失败原因**
    
*   **节点下线**
    
*   **冷热数据的读写分离**
    

*   **集群自动发现**
    

* * *

**架构原理**
--------

本书作为 Elastic Stack 指南，关注于 Elasticsearch 在日志和数据分析场景的应用，并不打算对底层的 Lucene 原理或者 Java 编程做详细的介绍，但是 Elasticsearch 层面上的一些架构设计，对我们做性能调优，故障处理，具有非常重要的影响。

所以，作为 ES 部分的起始章节，先从数据流向和分布的层面，介绍一下 ES 的工作原理，以及相关的可控项。各位读者可以跳过这节先行阅读后面的运维操作部分，但作为性能调优的基础知识，依然建议大家抽时间返回来了解。

> 推荐下自己做的 Spring Boot 的实战项目：
> 
> https://github.com/YunaiV/ruoyi-vue-pro

**带着问题学习**
----------

1.  写入的数据是如何变成 elasticsearch 里可以被检索和聚合的索引内容的？
    
2.  lucene 如何实现准实时索引？
    
3.  什么是 segment？
    
4.  什么是 commit？
    
5.  segment 的数据来自哪里？
    
6.  segment 在写入磁盘前就可以被检索，是因为利用了什么？
    
7.  elasticsearch 中的 refresh 操作是什么？配置项是哪个？设置的命令是什么？
    
8.  refresh 只是写到了文件系统缓存，那么实际写入磁盘是由什么控制呢？，如果这期间发生错误和故障，数据会不会丢失？
    
9.  什么是 translog 日志？什么时候会被清空？什么是 flush 操作？配置项是什么？怎么配置？
    
10.  什么是段合并？为什么要段合并？段合并线程配置项？段合并策略？怎么 forcemerge(optimize)？
    
11.  routing 的规则是什么样的？replica 读写过程？wait_for_active_shards 参数 timeout 参数 ？
    
12.  reroute 接口？
    
13.  两种 自动发现方式？
    

> 推荐下自己做的 Spring Cloud 的实战项目：
> 
> https://github.com/YunaiV/onemall

**segment、buffer 和 translog 对实时性的影响**
-------------------------------------

既然介绍数据流向，首先第一步就是：写入的数据是如何变成 Elasticsearch 里可以被检索和聚合的索引内容的？

以单文件的静态层面看，每个全文索引都是一个词元的倒排索引，具体涉及到全文索引的通用知识，这里不单独介绍，有兴趣的读者可以阅读《Lucene in Action》等书籍详细了解。

### **动态更新的 Lucene 索引**

以在线动态服务的层面看，要做到实时更新条件下数据的可用和可靠，就需要在倒排索引的基础上，再做一系列更高级的处理。

其实总结一下 Lucene 的处理办法，很简单，就是一句话：**新收到的数据写到新的索引文件里** 。

Lucene 把每次生成的倒排索引，叫做一个段 (segment)。然后另外使用一个 commit 文件，记录索引内所有的 segment。而生成 segment 的数据来源，则是内存中的 buffer。也就是说，动态更新过程如下：

1.  当前索引有 3 个 segment 可用。索引状态如图 2-1；![](https://mmbiz.qpic.cn/mmbiz_png/DMP9YVibia1dRJwpYwoHxuN1XJhwIJEAyEfDU55iclWcmsFPc1icA8O1mayX7ET5hliarc53NuHlAuMZRtqiaS3SklfQ/640?wx_fmt=png) 
    
2.  新接收的数据进入内存 buffer。索引状态如图 2-2；![](https://mmbiz.qpic.cn/mmbiz_png/DMP9YVibia1dRJwpYwoHxuN1XJhwIJEAyEFSC3WicZrfUBa34FG1VRM5217vewzHXcTsGlLssiaucKWz8xeN7S05lA/640?wx_fmt=png) 
    
3.  内存 buffer 刷到磁盘，生成一个新的 segment，commit 文件同步更新。索引状态如图 2-3。![](https://mmbiz.qpic.cn/mmbiz_png/DMP9YVibia1dRJwpYwoHxuN1XJhwIJEAyE2l9O93VGfDPOxW19JazmaAdUDO0EDDL3LJiaL3cgQ3LrVB3ia5t1N71w/640?wx_fmt=png) 
    

**利用磁盘缓存实现的准实时检索**

既然涉及到磁盘，那么一个不可避免的问题就来了：磁盘太慢了！对我们要求实时性很高的服务来说，这种处理还不够。所以，在第 3 步的处理中，还有一个中间状态：

1.  内存 buffer 生成一个新的 segment，刷到文件系统缓存中，Lucene 即可检索这个新 segment。索引状态如图 2-4。![](https://mmbiz.qpic.cn/mmbiz_png/DMP9YVibia1dRJwpYwoHxuN1XJhwIJEAyE5uJ7eDBNMkuQy1an8GzXC8t0ACIYUarmuvYQOqzlawSTkYyUah3gMQ/640?wx_fmt=png)  
    
2.  文件系统缓存真正同步到磁盘上，commit 文件更新。达到图 2-3 中的状态。
    

这一步刷到文件系统缓存的步骤，在 Elasticsearch 中，是默认设置为 1 秒间隔的，对于大多数应用来说，几乎就相当于是实时可搜索了。Elasticsearch 也提供了单独的 `/_refresh` 接口，用户如果对 1 秒间隔还不满意的，可以主动调用该接口来保证搜索可见。

_注：5.0 中还提供了一个新的请求参数：`?refresh=wait_for`，可以在写入数据后不强制刷新但一直等到刷新才返回。_

不过对于 Elastic Stack 的日志场景来说，恰恰相反，我们并不需要如此高的实时性，而是需要更快的写入性能。所以，一般来说，我们反而会通过 `/_settings` 接口或者定制 template 的方式，加大 `refresh_interval` 参数：

```
# curl -XPOST http://127.0.0.1:9200/logstash-2015.06.21/_settings -d'
{ "refresh_interval": "10s" }
'
```

如果是导入历史数据的场合，那甚至可以先完全关闭掉：

```
# curl -XPUT http://127.0.0.1:9200/logstash-2015.05.01 -d'
{
  "settings" : {
    "refresh_interval": "-1"
  }
}'
```

在导入完成以后，修改回来或者手动调用一次即可：

```
# curl -XPOST http://127.0.0.1:9200/logstash-2015.05.01/_refresh
```

### **translog 提供的磁盘同步控制**

既然 refresh 只是写到文件系统缓存，那么第 4 步写到实际磁盘又是有什么来控制的？如果这期间发生主机错误、硬件故障等异常情况，数据会不会丢失？

这里，其实有另一个机制来控制。Elasticsearch 在把数据写入到内存 buffer 的同时，其实还另外记录了一个 translog 日志。也就是说，第 2 步并不是图 2-2 的状态，而是像图 2-5 这样：

![](https://mmbiz.qpic.cn/mmbiz_png/DMP9YVibia1dRJwpYwoHxuN1XJhwIJEAyETkhjOOnw8Wm89ITgXeA0Mhxg716JcVj00v70x59NzSo7Y9Z4KkFXSA/640?wx_fmt=png)  

在第 3 和第 4 步，refresh 发生的时候，translog 日志文件依然保持原样，如图 2-6：

![](https://mmbiz.qpic.cn/mmbiz_png/DMP9YVibia1dRJwpYwoHxuN1XJhwIJEAyEakC9bJVBFccTGrQ4STGK0Zqn3jcTeW9FDaE5xbEAicAkaIwTFhdUcsA/640?wx_fmt=png)  

也就是说，如果在这期间发生异常，Elasticsearch 会从 commit 位置开始，恢复整个 translog 文件中的记录，保证数据一致性。

等到真正把 segment 刷到磁盘，且 commit 文件进行更新的时候， translog 文件才清空。这一步，叫做 flush。同样，Elasticsearch 也提供了 `/_flush` 接口。

对于 flush 操作，Elasticsearch 默认设置为：每 30 分钟主动进行一次 flush，或者当 translog 文件大小大于 512MB (老版本是 200MB) 时，主动进行一次 flush。这两个行为，可以分别通过 `index.translog.flush_threshold_period` 和 `index.translog.flush_threshold_size` 参数修改。

如果对这两种控制方式都不满意，Elasticsearch 还可以通过 `index.translog.flush_threshold_ops` 参数，控制每收到多少条数据后 flush 一次。

#### **translog 的一致性**

索引数据的一致性通过 translog 保证。那么 translog 文件自己呢？

默认情况下，Elasticsearch 每 5 秒，或每次请求操作结束前，会强制刷新 translog 日志到磁盘上。

后者是 Elasticsearch 2.0 新加入的特性。为了保证不丢数据，每次 index、bulk、delete、update 完成的时候，一定触发刷新 translog 到磁盘上，才给请求返回 200 OK。这个改变在提高数据安全性的同时当然也降低了一点性能。

如果你不在意这点可能性，还是希望性能优先，可以在 index template 里设置如下参数：

```
{    "index.translog.durability": "async"}
```

### **Elasticsearch 分布式索引**

大家可能注意到了，前面一段内容，一直写的是 "Lucene 索引"。这个区别在于，Elasticsearch 为了完成分布式系统，对一些名词概念作了变动。_索引_成为了整个集群级别的命名，而在单个主机上的 _Lucene 索引_，则被命名为_分片 (shard)_。至于数据是怎么识别到自己应该在哪个分片，请阅读稍后有关 routing 的章节。

**segment merge 对写入性能的影响**
--------------------------

通过上节内容，我们知道了数据怎么进入 ES 并且如何才能让数据更快的被检索使用。其中用一句话概括了 Lucene 的设计思路就是 "开新文件"。从另一个方面看，开新文件也会给服务器带来负载压力。因为默认每 1 秒，都会有一个新文件产生，每个文件都需要有文件句柄，内存，CPU 使用等各种资源。一天有 86400 秒，设想一下，每次请求要扫描一遍 86400 个文件，这个响应性能绝对好不了！

为了解决这个问题，ES 会不断在后台运行任务，主动将这些零散的 segment 做数据归并，尽量让索引内只保有少量的，每个都比较大的，segment 文件。这个过程是有独立的线程来进行的，并不影响新 segment 的产生。归并过程中，索引状态如图 2-7，尚未完成的较大的 segment 是被排除在检索可见范围之外的：

![](https://mmbiz.qpic.cn/mmbiz_png/DMP9YVibia1dRJwpYwoHxuN1XJhwIJEAyEjqrjC6aQccnUW0SsSC5bTiaic0mOvAW9WF9jcDdouvib5aEP8Axjy1leA/640?wx_fmt=png)  

当归并完成，较大的这个 segment 刷到磁盘后，commit 文件做出相应变更，删除之前几个小 segment，改成新的大 segment。等检索请求都从小 segment 转到大 segment 上以后，删除没用的小 segment。这时候，索引里 segment 数量就下降了，状态如图 2-8 所示：

![](https://mmbiz.qpic.cn/mmbiz_png/DMP9YVibia1dRJwpYwoHxuN1XJhwIJEAyE4VUxx2Y3mg7k0FfFl7CVicYCc4zCjVK7jr5LC11ibpw0YuvlCnC1LJog/640?wx_fmt=png)  

### **归并线程配置**

segment 归并的过程，需要先读取 segment，归并计算，再写一遍 segment，最后还要保证刷到磁盘。可以说，这是一个非常消耗磁盘 IO 和 CPU 的任务。所以，ES 提供了对归并线程的限速机制，确保这个任务不会过分影响到其他任务。

在 5.0 之前，归并线程的限速配置 `indices.store.throttle.max_bytes_per_sec` 是 20MB。对于写入量较大，磁盘转速较高，甚至使用 SSD 盘的服务器来说，这个限速是明显过低的。对于 Elastic Stack 应用，社区广泛的建议是可以适当调大到 100MB 或者更高。

```
# curl -XPUT http://127.0.0.1:9200/_cluster/settings -d'
{
    "persistent" : {
        "indices.store.throttle.max_bytes_per_sec" : "100mb"
    }
}'
```

5.0 开始，ES 对此作了大幅度改进，使用了 Lucene 的 CMS(ConcurrentMergeScheduler) 的 auto throttle 机制，正常情况下已经不再需要手动配置 `indices.store.throttle.max_bytes_per_sec` 了。官方文档中都已经删除了相关介绍，不过从源码中还是可以看到，这个值目前的默认设置是 10240 MB。

归并线程的数目，ES 也是有所控制的。默认数目的计算公式是：`Math.min(3, Runtime.getRuntime().availableProcessors() / 2)`。即服务器 CPU 核数的一半大于 3 时，启动 3 个归并线程；否则启动跟 CPU 核数的一半相等的线程数。相信一般做 Elastic Stack 的服务器 CPU 合数都会在 6 个以上。所以一般来说就是 3 个归并线程。如果你确定自己磁盘性能跟不上，可以降低 `index.merge.scheduler.max_thread_count` 配置，免得 IO 情况更加恶化。

### **归并策略**

归并线程是按照一定的运行策略来挑选 segment 进行归并的。主要有以下几条：

*   index.merge.policy.floor_segment 默认 2MB，小于这个大小的 segment，优先被归并。
    
*   index.merge.policy.max_merge_at_once 默认一次最多归并 10 个 segment
    
*   index.merge.policy.max_merge_at_once_explicit 默认 forcemerge 时一次最多归并 30 个 segment。
    
*   index.merge.policy.max_merged_segment 默认 5 GB，大于这个大小的 segment，不用参与归并。forcemerge 除外。
    

根据这段策略，其实我们也可以从另一个角度考虑如何减少 segment 归并的消耗以及提高响应的办法：加大 flush 间隔，尽量让每次新生成的 segment 本身大小就比较大。

### **forcemerge 接口**

既然默认的最大 segment 大小是 5GB。那么一个比较庞大的数据索引，就必然会有为数不少的 segment 永远存在，这对文件句柄，内存等资源都是极大的浪费。但是由于归并任务太消耗资源，所以一般不太选择加大 `index.merge.policy.max_merged_segment` 配置，而是在负载较低的时间段，通过 forcemerge 接口，强制归并 segment。

```
# curl -XPOST http://127.0.0.1:9200/logstash-2015-06.10/_forcemerge?max_num_segments=1
```

由于 forcemerge 线程对资源的消耗比普通的归并线程大得多，所以，绝对不建议对还在写入数据的热索引执行这个操作。这个问题对于 Elastic Stack 来说非常好办，一般索引都是按天分割的。更合适的任务定义方式，请阅读本书稍后的 curator 章节。

**routing 和 replica 的读写过程**
---------------------------

之前两节，完整介绍了在单个 Lucene 索引，即 ES 分片内的数据写入流程。现在彻底回到 ES 的分布式层面上来，当一个 ES 节点收到一条数据的写入请求时，它是如何确认这个数据应该存储在哪个节点的哪个分片上的？

### **路由计算**

作为一个没有额外依赖的简单的分布式方案，ES 在这个问题上同样选择了一个非常简洁的处理方式，对任一条数据计算其对应分片的方式如下：

> shard = hash(routing) % number_of_primary_shards

每个数据都有一个 routing 参数，默认情况下，就使用其 `_id` 值。将其 `_id` 值计算哈希后，对索引的主分片数取余，就是数据实际应该存储到的分片 ID。

由于取余这个计算，完全依赖于分母，所以导致 ES 索引有一个限制，索引的主分片数，不可以随意修改。因为一旦主分片数不一样，所以数据的存储位置计算结果都会发生改变，索引数据就完全不可读了。

### **副本一致性**

作为分布式系统，数据副本可算是一个标配。ES 数据写入流程，自然也涉及到副本。在有副本配置的情况下，数据从发向 ES 节点，到接到 ES 节点响应返回，流向如下 (附图 2-9)：

1.  客户端请求发送给 Node 1 节点，注意图中 Node 1 是 Master 节点，实际完全可以不是。
    
2.  Node 1 用数据的 `_id` 取余计算得到应该讲数据存储到 shard 0 上。通过 cluster state 信息发现 shard 0 的主分片已经分配到了 Node 3 上。Node 1 转发请求数据给 Node 3。
    
3.  Node 3 完成请求数据的索引过程，存入主分片 0。然后并行转发数据给分配有 shard 0 的副本分片的 Node 1 和 Node 2。当收到任一节点汇报副本分片数据写入成功，Node 3 即返回给初始的接收节点 Node 1，宣布数据写入成功。Node 1 返回成功响应给客户端。
    
4.    
    

![](https://mmbiz.qpic.cn/mmbiz_png/DMP9YVibia1dRJwpYwoHxuN1XJhwIJEAyEF6JFGekkMDNqIxM7KGS3jXHgvND3XQot9hIdIr55yxwN7YGvMVmu9g/640?wx_fmt=png)

图片

这个过程中，有几个参数可以用来控制或变更其行为：

*   wait_for_active_shards 上面示例中，2 个副本分片只要有 1 个成功，就可以返回给客户端了。这点也是有配置项的。其默认值的计算来源如下：
    

> int((primary + number_of_replicas) / 2 ) + 1

根据需要，也可以将参数设置为 one，表示仅写完主分片就返回，等同于 async；还可以设置为 all，表示等所有副本分片都写完才能返回。

*   timeout 如果集群出现异常，有些分片当前不可用，ES 默认会等待 1 分钟看分片能否恢复。可以使用 `?timeout=30s` 参数来缩短这个等待时间。
    

副本配置和分片配置不一样，是可以随时调整的。有些较大的索引，甚至可以在做 forcemerge 前，先把副本全部取消掉，等 optimize 完后，再重新开启副本，节约单个 segment 的重复归并消耗。

```
# curl -XPUT http://127.0.0.1:9200/logstash-mweibo-2015.05.02/_settings -d '{
"index": { "number_of_replicas" : 0 }
}'
```

**shard 的 allocate 控制**
-----------------------

某个 shard 分配在哪个节点上，一般来说，是由 ES 自动决定的。以下几种情况会触发分配动作：

1.  新索引生成
    
2.  索引的删除
    
3.  新增副本分片
    
4.  节点增减引发的数据均衡
    

ES 提供了一系列参数详细控制这部分逻辑：

*   cluster.routing.allocation.enable 该参数用来控制允许分配哪种分片。默认是 `all`。可选项还包括 `primaries` 和 `new_primaries`。`none` 则彻底拒绝分片。该参数的作用，本书稍后集群升级章节会有说明。
    
*   cluster.routing.allocation.allow_rebalance 该参数用来控制什么时候允许数据均衡。默认是 `indices_all_active`，即要求所有分片都正常启动成功以后，才可以进行数据均衡操作，否则的话，在集群重启阶段，会浪费太多流量了。
    
*   cluster.routing.allocation.cluster_concurrent_rebalance 该参数用来控制**集群内** 同时运行的数据均衡任务个数。默认是 2 个。如果有节点增减，且集群负载压力不高的时候，可以适当加大。
    
*   cluster.routing.allocation.node_initial_primaries_recoveries 该参数用来控制**节点** 重启时，允许同时恢复几个主分片。默认是 4 个。如果节点是多磁盘，且 IO 压力不大，可以适当加大。
    
*   cluster.routing.allocation.node_concurrent_recoveries 该参数用来控制**节点** 除了主分片重启恢复以外其他情况下，允许同时运行的数据恢复任务。默认是 2 个。所以，节点重启时，可以看到主分片迅速恢复完成，副本分片的恢复却很慢。除了副本分片本身数据要通过网络复制以外，并发线程本身也减少了一半。当然，这种设置也是有道理的——主分片一定是本地恢复，副本分片却需要走网络，带宽是有限的。从 ES 1.6 开始，冷索引的副本分片可以本地恢复，这个参数也就是可以适当加大了。
    
*   indices.recovery.concurrent_streams 该参数用来控制**节点** 从网络复制恢复副本分片时的数据流个数。默认是 3 个。可以配合上一条配置一起加大。
    
*   indices.recovery.max_bytes_per_sec 该参数用来控制**节点** 恢复时的速率。默认是 40MB。显然是比较小的，建议加大。
    

此外，ES 还有一些其他的分片分配控制策略。比如以 `tag` 和 `rack_id` 作为区分等。一般来说，Elastic Stack 场景中使用不多。运维人员可能比较常见的策略有两种：

1.  磁盘限额 为了保护节点数据安全，ES 会定时 (`cluster.info.update.interval`，默认 30 秒) 检查一下各节点的数据目录磁盘使用情况。在达到 `cluster.routing.allocation.disk.watermark.low` (默认 85%) 的时候，新索引分片就不会再分配到这个节点上了。在达到 `cluster.routing.allocation.disk.watermark.high` (默认 90%) 的时候，就会触发该节点现存分片的数据均衡，把数据挪到其他节点上去。这两个值不但可以写百分比，还可以写具体的字节数。有些公司可能出于成本考虑，对磁盘使用率有一定的要求，需要适当抬高这个配置：
    

```
# curl -XPUT localhost:9200/_cluster/settings -d '{
"transient" : {
"cluster.routing.allocation.disk.watermark.low" : "85%",
"cluster.routing.allocation.disk.watermark.high" : "10gb",
"cluster.info.update.interval" : "1m"
}
}'
```

2.  热索引分片不均 默认情况下，ES 集群的数据均衡策略是以各节点的分片总数 (_indices_all_active_) 作为基准的。这对于搜索服务来说无疑是均衡搜索压力提高性能的好办法。但是对于 Elastic Stack 场景，一般压力集中在新索引的数据写入方面。正常运行的时候，也没有问题。但是当集群扩容时，新加入集群的节点，分片总数远远低于其他节点。这时候如果有新索引创建，ES 的默认策略会导致新索引的所有主分片几乎全分配在这台新节点上。整个集群的写入压力，压在一个节点上，结果很可能是这个节点直接被压死，集群出现异常。所以，对于 Elastic Stack 场景，强烈建议大家预先计算好索引的分片数后，配置好单节点分片的限额。比如，一个 5 节点的集群，索引主分片 10 个，副本 1 份。则平均下来每个节点应该有 4 个分片，那么就配置：
    

```
# curl -s -XPUT http://127.0.0.1:9200/logstash-2015.05.08/_settings -d '{
"index": { "routing.allocation.total_shards_per_node" : "5" }
}'
```

注意，这里配置的是 5 而不是 4。因为我们需要预防有机器故障，分片发生迁移的情况。如果写的是 4，那么分片迁移会失败。

此外，另一种方式则更加玄妙，Elasticsearch 中有一系列参数，相互影响，最终联合决定分片分配：

*   cluster.routing.allocation.balance.shard 节点上分配分片的权重，默认为 0.45。数值越大越倾向于在节点层面均衡分片。
    
*   cluster.routing.allocation.balance.index 每个索引往单个节点上分配分片的权重，默认为 0.55。数值越大越倾向于在索引层面均衡分片。
    
*   cluster.routing.allocation.balance.threshold 大于阈值则触发均衡操作。默认为 1。
    

Elasticsearch 中的计算方法是：

(indexBalance _(node.numShards(index) – avgShardsPerNode(index)) + shardBalance_ (node.numShards() – avgShardsPerNode)) <=> weightthreshold

所以，也可以采取加大 `cluster.routing.allocation.balance.index`，甚至设置 `cluster.routing.allocation.balance.shard` 为 0 来尽量采用索引内的节点均衡。

### reroute 接口

上面说的各种配置，都是从策略层面，控制分片分配的选择。在必要的时候，还可以通过 ES 的 reroute 接口，手动完成对分片的分配选择的控制。

reroute 接口支持五种指令：`allocate_replica`, `allocate_stale_primary`, `allocate_empty_primary`，`move` 和 `cancel`。常用的一般是 allocate 和 move：

*   `allocate_*` 指令
    

因为负载过高等原因，有时候个别分片可能长期处于 UNASSIGNED 状态，我们就可以手动分配分片到指定节点上。默认情况下只允许手动分配副本分片 (即使用 `allocate_replica`)，所以如果要分配主分片，需要单独加一个 `accept_data_loss` 选项：

```
# curl -XPOST 127.0.0.1:9200/_cluster/reroute -d '{
"commands" : [ {
"allocate_stale_primary" :
{
"index" : "logstash-2015.05.27", "shard" : 61, "node" : "10.19.0.77", "accept_data_loss" : true
}
}
]
}'
```

注意，`allocate_stale_primary` 表示准备分配到的节点上可能有老版本的历史数据，运行时请提前确认一下是哪个节点上保留有这个分片的实际目录，且目录大小最大。然后手动分配到这个节点上。以此减少数据丢失。

*   move 指令
    

因为负载过高，磁盘利用率过高，服务器下线，更换磁盘等原因，可以会需要从节点上移走部分分片：

```
curl -XPOST 127.0.0.1:9200/_cluster/reroute -d '{
"commands" : [ {
"move" :
{
"index" : "logstash-2015.05.22", "shard" : 0, "from_node" : "10.19.0.81", "to_node" : "10.19.0.104"
}
}
]
}'
```

### 分配失败原因

如果是自己手工 reroute 失败，Elasticsearch 返回的响应中会带上失败的原因。不过格式非常难看，一堆 YES，NO。从 5.0 版本开始，Elasticsearch 新增了一个 allocation explain 接口，专门用来解释指定分片的具体失败理由：

```
curl -XGET 'http://localhost:9200/_cluster/allocation/explain' -d'{
"index": "logstash-2016.10.31",
"shard": 0,
"primary": false

}'
```

得到的响应如下：

```
{
  "shard" : {
    "index" : "myindex",
    "index_uuid" : "KnW0-zELRs6PK84l0r38ZA",
    "id" : 0,
    "primary" : false
  },
  "assigned" : false,
  "shard_state_fetch_pending": false,
  "unassigned_info" : {
    "reason" : "INDEX_CREATED",
    "at" : "2016-03-22T20:04:23.620Z"
  },
  "allocation_delay_ms" : 0,
  "remaining_delay_ms" : 0,
  "nodes" : {
    "V-Spi0AyRZ6ZvKbaI3691w" : {
      "node_name" : "H5dfFeA",
      "node_attributes" : {
        "bar" : "baz"
      },
      "store" : {
        "shard_copy" : "NONE"
      },
      "final_decision" : "NO",
      "final_explanation" : "the shard cannot be assigned because one or more allocation decider returns a 'NO' decision",
      "weight" : 0.06666675,
      "decisions" : [ {
        "decider" : "filter",
        "decision" : "NO",
        "explanation" : "node does not match index include filters [foo:\"bar\"]"
      }  ]
    },
    "Qc6VL8c5RWaw1qXZ0Rg57g" : {
      ...
```

这会是很长一串 JSON，把集群里所有的节点都列上来，挨个解释为什么不能分配到这个节点。

### 节点下线

集群中个别节点出现故障预警等情况，需要下线，也是 Elasticsearch 运维工作中常见的情况。如果已经稳定运行过一段时间的集群，每个节点上都会保存有数量不少的分片。这种时候通过 reroute 接口手动转移，就显得太过麻烦了。这个时候，有另一种方式：

```
curl -XPUT 127.0.0.1:9200/_cluster/settings -d '{
"transient" :{
"cluster.routing.allocation.exclude._ip" : "10.0.0.1"
}
}'
```

Elasticsearch 集群就会自动把这个 IP 上的所有分片，都自动转移到其他节点上。等到转移完成，这个空节点就可以毫无影响的下线了。

和 `_ip` 类似的参数还有 `_host`, `_name` 等。此外，这类参数不单是 cluster 级别，也可以是 index 级别。下一小节就是 index 级别的用例。

### 冷热数据的读写分离

Elasticsearch 集群一个比较突出的问题是: 用户做一次大的查询的时候, 非常大量的读 IO 以及聚合计算导致机器 Load 升高, CPU 使用率上升, 会影响阻塞到新数据的写入, 这个过程甚至会持续几分钟。所以，可能需要仿照 MySQL 集群一样，做读写分离。

#### 实施方案

1.  N 台机器做热数据的存储, 上面只放当天的数据。这 N 台热数据节点上面的 elasticsearc.yml 中配置 `node.tag: hot`
    
2.  之前的数据放在另外的 M 台机器上。这 M 台冷数据节点中配置 `node.tag: stale`
    
3.  模板中控制对新建索引添加 hot 标签：
    
    ```
     {
     "order" : 0,
     "template" : "*",
     "settings" : {
       "index.routing.allocation.require.tag" : "hot"
       }
     }
    ```
    
4.  每天计划任务更新索引的配置, 将 tag 更改为 stale, 索引会自动迁移到 M 台冷数据节点
    
5.    
    

```
# curl -XPUT http://127.0.0.1:9200/indexname/_settings -d'
{
"index": {
   "routing": {
      "allocation": {
         "require": {
            "tag": "stale"
         }
      }
  }
}
}'
```

6.  这样，写操作集中在 N 台热数据节点上，大范围的读操作集中在 M 台冷数据节点上。避免了堵塞影响。
    

该方案运用的，是 Elasticsearch 中的 allocation filter 功能，详细说明见：https://www.elastic.co/guide/en/elasticsearch/reference/master/shard-allocation-filtering.html

集群自动发现
------

ES 是一个 P2P 类型 (使用 gossip 协议) 的分布式系统，除了集群状态管理以外，其他所有的请求都可以发送到集群内任意一台节点上，这个节点可以自己找到需要转发给哪些节点，并且直接跟这些节点通信。

所以，从网络架构及服务配置上来说，构建集群所需要的配置极其简单。在 Elasticsearch 2.0 之前，无阻碍的网络下，所有配置了相同 `cluster.name` 的节点都自动归属到一个集群中。

2.0 版本之后，基于安全的考虑，Elasticsearch 稍作了调整，避免开发环境过于随便造成的麻烦。

### unicast 方式

ES 从 2.0 版本开始，默认的自动发现方式改为了单播 (unicast) 方式。配置里提供几台节点的地址，ES 将其视作 gossip router 角色，借以完成集群的发现。由于这只是 ES 内一个很小的功能，所以 gossip router 角色并不需要单独配置，每个 ES 节点都可以担任。所以，采用单播方式的集群，各节点都配置相同的几个节点列表作为 router 即可。

此外，考虑到节点有时候因为高负载，慢 GC 等原因可能会有偶尔没及时响应 ping 包的可能，一般建议稍微加大 Fault Detection 的超时时间。

同样基于安全考虑做的变更还有监听的主机名。现在默认只监听本地 lo 网卡上。所以正式环境上需要修改配置为监听具体的网卡。

```
network.host: "192.168.0.2"
discovery.zen.minimum_master_nodes: 3
discovery.zen.ping_timeout: 100s
discovery.zen.fd.ping_timeout: 100s
discovery.zen.ping.unicast.hosts: ["10.19.0.97","10.19.0.98","10.19.0.99","10.19.0.100"]
```

上面的配置中，两个 timeout 可能会让人有所迷惑。这里的 **fd** 是 fault detection 的缩写。也就是说：

*   discovery.zen.ping_timeout 参数仅在加入或者选举 master 主节点的时候才起作用；
    
*   discovery.zen.fd.ping_timeout 参数则在稳定运行的集群中，master 检测所有节点，以及节点检测 master 是否畅通时长期有用。
    

既然是长期有用，自然还有运行间隔和重试的配置，也可以根据实际情况调整：

```
discovery.zen.fd.ping_interval: 10s
discovery.zen.fd.ping_retries: 10
```