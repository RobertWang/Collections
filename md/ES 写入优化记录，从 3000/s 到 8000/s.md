> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/mZRDVzIbmWdwLVLRktGyRQ)

**大家好，我是磊哥。**

优化前，写入速度平均`3000条/s`，一遇到压测，写入速度骤降，甚至 es 直接频率 gc、oom 等；优化后，写入速度平均`8000条/s`，遇到压测，能在压测结束后 30 分钟内消化完数据，各项指标回归正常。  

### 生产配置

这里我先把自己优化的结果贴出来，后面有参数的详解：

elasticsearch.yml 中增加如下设置

```
indices.memory.index_buffer_size: 20%
indices.memory.min_index_buffer_size: 96mb

# Search pool
thread_pool.search.size: 5
thread_pool.search.queue_size: 100
# 这个参数慎用！强制修改cpu核数，以突破写线程数限制
# processors: 16
# Bulk pool
#thread_pool.bulk.size: 16
thread_pool.bulk.queue_size: 300
# Index pool
#thread_pool.index.size: 16
thread_pool.index.queue_size: 300

indices.fielddata.cache.size: 40%

discovery.zen.fd.ping_timeout: 120s
discovery.zen.fd.ping_retries: 6
discovery.zen.fd.ping_interval: 30s


```

索引优化配置：

```
PUT /_template/elk
{
      "order": 6,
      "template": "logstash-*",    #这里配置模板匹配的Index名称
      "settings": {
        "number_of_replicas" : 0,    #副本数为0,需要查询性能高可以设置为1
        "number_of_shards" :   6,    #分片数为6, 副本为1时可以设置成5
         "refresh_interval": "30s",
         "index.translog.durability": "async",
        "index.translog.sync_interval": "30s"

      }
}


```

### 优化参数详解

**精细设置全文域：** string 类型字段默认会分词，不仅会额外占用资源，而且会影响创建索引的速度。所以，把不需要分词的字段设置为`not_analyzed`

**禁用_all 字段:** 对于日志和 apm 数据，目前没有场景会使用到

**使用 es 自动生成 id：** es 对于自动生成的 id 有优化，避免了版本查找。因为其生成的 id 是唯一的

**设置 index.refresh_interval：** 索引刷新间隔，默认为 1s。因为不需要如此高的实时性，我们修改为 30s – 扩展学习：刷新索引到底要做什么事情

**设置段合并的线程数量：**

```
curl -XPUT 'your-es-host:9200/nginx_log-2018-03-20/_settings' -d '{ 
   "index.merge.scheduler.max_thread_count" : 1
}'


```

段合并的计算量庞大，而且还要吃掉大量磁盘 I/O。合并在后台定期操作，因为他们可能要很长时间才能完成，尤其是比较大的段

机械磁盘在并发 I/O 支持方面比较差，所以我们需要降低每个索引并发访问磁盘的线程数。这个设置允许`max_thread_count + 2`个线程同时进行磁盘操作，也就是设置为 1 允许三个线程

> 扩展学习：什么是段 (segment)？如何合并段？为什么要合并段？（what、how、why）

1. 设置异步刷盘事务日志文件：

```
"index.translog.durability": "async",
"index.translog.sync_interval": "30s"


```

对于日志场景，能够接受部分数据丢失。同时有全量可靠日志存储在 hadoop，丢失了也可以从 hadoop 恢复回来

2.elasticsearch.yml 中增加如下设置：

```
indices.memory.index_buffer_size: 20%
indices.memory.min_index_buffer_size: 96mb


```

已经索引好的文档会先存放在内存缓存中，等待被写到到段 (segment) 中。缓存满的时候会触发段刷盘(吃 i/o 和 cpu 的操作)。默认最小缓存大小为 48m，不太够，最大为堆内存的 10%。对于大量写入的场景也显得有点小。

> 扩展学习：数据写入流程是怎么样的 (具体到如何构建索引)？

1. 设置 index、merge、bulk、search 的线程数和队列数。例如以下 elasticsearch.yml 设置：

```
# Search pool
thread_pool.search.size: 5
thread_pool.search.queue_size: 100
# 这个参数慎用！强制修改cpu核数，以突破写线程数限制
# processors: 16
# Bulk pool
thread_pool.bulk.size: 16
thread_pool.bulk.queue_size: 300
# Index pool
thread_pool.index.size: 16
thread_pool.index.queue_size: 300


```

2. 设置 filedata cache 大小，例如以下 elasticsearch.yml 配置：

```
indices.fielddata.cache.size: 15%


```

filedata cache 的使用场景是一些聚合操作 (包括排序), 构建 filedata cache 是个相对昂贵的操作。所以尽量能让他保留在内存中

然后日志场景聚合操作比较少，绝大多数也集中在半夜，所以限制了这个值的大小，默认是不受限制的，很可能占用过多的堆内存

> 扩展学习：什么是 filedata？构建流程是怎样的？为什么要用 filedata？（what、how、why）

1. 设置节点之间的故障检测配置，例如以下 elasticsearch.yml 配置：

```
discovery.zen.fd.ping_timeout: 120s
discovery.zen.fd.ping_retries: 6
discovery.zen.fd.ping_interval: 30s


```

大数量写入的场景，会占用大量的网络带宽，很可能使节点之间的心跳超时。并且默认的心跳间隔也相对过于频繁（1s 检测一次）

此项配置将大大缓解节点间的超时问题

### 后记

这里仅仅是记录对我们实际写入有提升的一些配置项，没有针对个别配置项做深入研究。

扩展学习后续填坑。基本都遵循（what、how、why）原则去学习。