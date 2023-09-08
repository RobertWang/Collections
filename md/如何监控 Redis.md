> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Y6ygx8ewZOqIlapGrVhD8w)

> 本篇文章介绍了 redis 监控需要关注的指标。并通过 redis-exporter 的方式，将 redis 的监控纳入到 prometheus 体系中来

### 简述

上篇文章介绍了如何搭建 prometheus 监控体系，监控 linux 服务器，这篇文章跟大家介绍如何监控 redis，以及我们要关注的指标都有哪些

### 监控 redis 需要关注什么指标

在[《聊聊监控》](http://mp.weixin.qq.com/s?__biz=MzU2NzkzOTQzNQ==&mid=2247484193&idx=1&sn=fd11411644476f5769c96af5b03ec8c6&chksm=fc94db25cbe35233e942227df4d7e6cd78f5318b904209aae73832b67bb1040388a432d26103&scene=21#wechat_redirect)这篇文章，介绍了 google 提出的监控四个黄金指标（没看过的朋友可以看看这篇文章），下面我们就分别通过`延迟`、`流量`、`错误`、`饱和度`四方面，来看看对应到 redis 中，我们要监控哪些数据指标（metrics）

#### 延迟

redis-cli 提供了`--latency`命令，可以很方面的让我们获取到 redis 执行命令的延迟，其原理是用 redis-cli 连接到 redis-server 上，然后不断发送`ping`命令，统计`ping`命令的耗时

```
> redis-cli --latency -h 127.0.0.1 -p 6379
min: 0, max: 1, avg: 0.13 (412 samples)


```

可以看到这里的延迟是`0.13ms`，因为我是在 redis-server 所在机器执行的`--latency`命令，下面看看我在另外一台机器执行`--latnecy`命令的结果

```
> redis-cli -h 192.168.57.140 -p 6379 --latency
min: 0, max: 3, avg: 1.21 (199 samples)


```

可以看到，现在的延迟为`1.21ms`，证明有大概`1.08ms`花费在了网络 I/O 上

到这里可能有些人会说，ping 命令很简单，是不是不能反馈出真实的命令执行延迟呢？其实，我们都知道，redis 是单线程模型的，如果有一条命令执行的慢，那么其后面的命令都得等着，所以我们是可以使用 ping 命令的执行耗时来作为 redis 命令执行耗时的指标的

`--latency`命令只能知道 redis 在什么时间点延迟比较高，并不知道延迟高是什么原因造成的，或者说不知道是哪条命令执行比较耗时，导致 redis 延迟高。跟 mysql 一样，redis 也提供了慢查询的功能，使用`slowlog get [count]`可以查看最近执行的慢查询命令（慢查询时间通过`slowlog-log-slower-than`配置指定）

```
127.0.0.1:6379> SLOWLOG get 1
1) 1) (integer) 47
   2) (integer) 1668743666
   3) (integer) 13168
   4) 1) "hset"
      2) "/idents/Default"
      3) "tt-fc-dev01.nj"
      4) "1668743666"
   5) "127.0.0.1:43172"
   6) ""


```

#### 流量

在 redis 的流量监控中，我们一般关注的是 redis 每秒的请求数（即执行了多少次操作）、每秒接受跟返回的数据量。这些指标在都可以通过`info all`命令获取

```
> redis-cli -h 127.0.0.1 -p 6379 info all | grep instantaneous
instantaneous_ops_per_sec:0
instantaneous_input_kbps:0.00
instantaneous_output_kbps:0.00


```

*   `instantaneous_ops_per_sec`: 每秒执行了多少次操作
    
*   `instantaneous_input_kbps`: 每秒接受多少 KiB 的数据
    
*   `instantaneous_output_kbps`: 每秒返回多少 KiB 的数据
    

如果将 redis 作为缓存使用的话，还要关注缓存的命中率，同样的，可以使用`info all`命令查询

```
> redis-cli -h 127.0.0.1 -p 6379 info all | grep keyspace
keyspace_hits:0
keyspace_misses:1


```

*   `keyspace_hits`: 自 redis 启动以来，查询命令的命中数量
    
*   `keyspace_misses`: 自 redis 启动以来，未命中的数量
    

有了这两个指标，就可以通过`keyspace_hits / (keyspace_hits + keyspace_misses)`计算出缓存的命中率

#### 错误

因为 redis 都是内存操作，基本不会出现什么错误，有错误的话一般是命令写错导致的，这一般在开发的时候就解决了，所以不用对错误做什么特殊的监控

#### 饱和度

饱和度指的是 redis 有多 “满”，在 redis 中有两个数据可以反映出 redis 究竟有多 “满”，一个是`内存使用率`，另外一个是`内存的碎片率`

`内存使用率`可以通过`info memory`命令查看

```
> info memory 
# Memory
used_memory:1227384
used_memory_human:1.17M
used_memory_rss:4308992
used_memory_rss_human:4.11M
...
maxmemory:134217728
maxmemory_human:128.00M
...
mem_fragmentation_ratio:3.51
...


```

*   `used_memory`: 使用了多少内存
    
*   `used_memory_rss`: 操作系统分配了多少内存给 redis
    
*   `mem_fragmentation_ratio`: 即内存碎片率，根据`use_memory_rss/use_memory`计算得出，正常来讲，操作系统在分配内存的时候，有最小分配单位的限制（不同操作系统不一样，有 8byte、16byte 等），所以内存碎片率稍大于 1 是正常的，如果内存碎片率过高，可能就需要考虑对内存碎片进行清理了
    

### redis-exporter 安装使用

redis 本身不通过 prometheus 协议暴露自身的各种数据指标，与`node-exporter`一样，我们可以运行通过`redis-exporter`，将 redis 的指标暴露给 pormetheus

`redis-exporter`下载地址：https://github.com/oliver006/redis_exporter/releases，目前最新的版本是 1.52.0

```
$ wget https://github.com/oliver006/redis_exporter/releases/download/v1.52.0/redis_exporter-v1.52.0.linux-amd64.tar.gz
$ tar -zxvf redis_exporter-v1.52.0.linux-amd64.tar.gz
$ mv redis_exporter-v1.52.0
$ cd redis_exporter-v1.52.0
$ ./redis_exporter &


```

`redis-exporter`暴露的端口是`9121`，可以通过访问 9121 查看采集的所有指标![](https://mmbiz.qpic.cn/mmbiz_png/clfPRsQo4DShiaYiaia0PvBKtAxSna1fU1ooXAAiawDicqSpnib7p0HiaMquWy8HKPUFEUCfZL53kRASiauyUdSFLWjuAA/640?wx_fmt=png)

### prometheus 配置

在 prometheus 配置文件中加入如下配置

```
- job_name: 'redis-exporter'
    static_configs:
      - targets: ['localhost:9121']


```

向 prometheus 发送 HUP 信号，让 prometheus 重新读取配置文件

```
$ kill -HUP `pidof prometheus`


```

> prometheus 与 grafana 的安装，在我上篇文章有讲，还不清楚怎么搭建的同学可以翻阅我上篇文章——[《如何搭建 Linux 服务器监控系统》](http://mp.weixin.qq.com/s?__biz=MzU2NzkzOTQzNQ==&mid=2247484208&idx=1&sn=d4f308638b791535ad3b882f9d51a02f&chksm=fc94db34cbe35222b0e4e1c5c48d5854cfdbc7d9f00b482cd5b170750f2db817ffb96a8be2ab&scene=21#wechat_redirect)

### grafana 配置

redis 控制面板，我这里用的是`11835`这个面板，一样通过 dashboard ID 的方式导入![](https://mmbiz.qpic.cn/mmbiz_png/clfPRsQo4DShiaYiaia0PvBKtAxSna1fU1ooslL2ZayyVUnIv81P7LGPWicVa7MYW7lJ0hia9jUMeDAL7JVoOQgI4tg/640?wx_fmt=png)

监控面板如下![](https://mmbiz.qpic.cn/mmbiz_png/clfPRsQo4DShiaYiaia0PvBKtAxSna1fU1oJdFzAX0dFWtI7MF9J7HO2cEI3f42TJtibaiaWNoNWQncmpXRibwmJwic3g/640?wx_fmt=png)

可以看到，面板除了展示了我们上面所讲到的指标外（如内存使用率、缓存命中数等），还展示了客户端连接数、redis 正常运行时间等

另外需要注意的是：如果你像下面一样不展示内存使用率的话

![](https://mmbiz.qpic.cn/mmbiz_png/clfPRsQo4DShiaYiaia0PvBKtAxSna1fU1oJSIPyDexMZ8cWeUjXeFnicWIIzR7eRo4aMSVpkPyzFM3cNwKn3ZTRbw/640?wx_fmt=png)

可能是读取不到`redis_memory_max_bytes`指标，那是因为没配置 redis 的最大内存，可以在 redis 配置文件中添加`maxmemory`配置，或者使用`config rewrite`命令进行修改

```
127.0.0.1:6379> config set maxmemory 128mb
OK
127.0.0.1:6379> config rewrite
8110:M 07 Aug 2023 09:21:53.983 # CONFIG REWRITE executed with success.
OK


```

### 总结

本篇文章讲了 redis 监控需要关注的指标。并通过`redis-exporter`的方式，将 redis 的监控纳入到 prometheus 体系中来，如果觉得我的文章对你有帮助的话，可以点个关注或者在看哦，你的支持是我写作的动力