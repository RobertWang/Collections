> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Axpd3KWU4J-ePGfURqTP9Q)

**1.  背景**
----------

业务每到凌晨时间，经常告警反馈 Redis（本文提到的 Redis 版本为 6.0） CPU 打满，并且持续时间大概一分钟左右。在持续期间中， C 端相关请求延迟耗时会有所增高，影响整体 RT。

**2. 分析过程**
-----------

### **2.1 现象分析**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/yiaMkmW2kMZHzTWaz2k7zS5FCgWOo3VCT8UQ3vt15cIe1vTxb25K4pEP5AADArrwsRgtbib0KJichOWicSYuuT4q6w/640?wx_fmt=png&from=appmsg)

命令执行次数

![](https://mmbiz.qpic.cn/sz_mmbiz_png/yiaMkmW2kMZHzTWaz2k7zS5FCgWOo3VCT1koTWQyCGFNFndeK79S008TuYKgjHzxRLjaNsLH27JXQiczDicvFKMOg/640?wx_fmt=png&from=appmsg)

高峰期从延迟

### **2.2 原因分析**

明确是过期导致的问题后，对代码进行排查，发现面向 C 端项目中有几个业务模块将 Redis key 的过期时间设置为在凌晨 00:00 准点过期。

#### **2.2.1 Redis 请求阻塞**

> active_expire_effort(1~10) 默认值 1，数值越大则消耗 CPU 资源越多。

Redis 是采用被动删除与主动删除相结合的方式对过期进行删除 key，被动删除几乎对性能没有影响。Redis 的主动删除方式又分为慢周期（ACTIVE_EXPIRE_CYCLE_SLOW）、快周期（ACTIVE_EXPIRE_CYCLE_FAST）。  

● 快周期

![](https://mmbiz.qpic.cn/sz_mmbiz_png/yiaMkmW2kMZHzTWaz2k7zS5FCgWOo3VCTFTnlaeNgjokrzFFM7IIuJ6mzwfAp16uOvw2benybAQ01EgEeicicezng/640?wx_fmt=png&from=appmsg)

快周期调用链路

1000 + 1000/4*effort(active_expire_effort 的配置值 - 1)，默认值为 1000 微秒，最多不超过 3250 微秒。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/yiaMkmW2kMZHzTWaz2k7zS5FCgWOo3VCT4F3BRHXx8Uia2Zo6ribcLDykjMTeXG2aFJhhbKwuonR5icIrl5sDW6KKg/640?wx_fmt=png&from=appmsg)

快周期执行时长

此外，快周期是周期性执行的，进入时会依赖上一次快周期时间片是否执行完和脏 key 百分比是否超过 config_cycle_acceptable_stale 值来判断进入快周期。如下图，config_cycle_acceptable_stale = 10-effort(active_expire_effort 的配置值 - 1)，默认配置下容忍 10% 的过期 key 在内存中未正式删除。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/yiaMkmW2kMZHzTWaz2k7zS5FCgWOo3VCTkMGKjmdUqLATy3PgCZhutA8vWIqPJ5Jj4SbFdGGqWWickZWf9hzAcXA/640?wx_fmt=png&from=appmsg)

快周期进入条件

通常情况下，快周期删除对命令的阻塞并不长，但需要关注 active_expire_effort 配置为 10 的情况下最大会阻塞 3250 微秒。

● 慢周期

![](https://mmbiz.qpic.cn/sz_mmbiz_png/yiaMkmW2kMZHzTWaz2k7zS5FCgWOo3VCTveicl4u5ypYtkPs0gfQf5J45hZGxSBiaFDaT2Aq4oMswQibXmnJONZseA/640?wx_fmt=png&from=appmsg)

慢周期调用链路

如上图 databasesCron() 方法由 redis 的 ae 框架触发调度，时间间隔由 1000/server.hz（默认值 10，每隔 100ms 执行一次。数值越大越耗费 CPU）控制。慢周期的执行时间由如下公式确定：

config_cycle_slow_time_perc= 25 +2*effort(active_expire_effort 的配置值 - 1)，最小百分比 25%，最大 43%。默认情况下，timelimit = 25 * 1000000/10/100，25000 微秒，即 25 毫秒。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/yiaMkmW2kMZHzTWaz2k7zS5FCgWOo3VCTjkgGst6sBYibfPtjCjnTIKChuUkib4RR1qGibD1yjDPnF5Xe4Sqhs5hqA/640?wx_fmt=png&from=appmsg)

快周期执行时间时间公式

确定完时间后就会开始对每个 db 执行 scan ，进行 key 采样抽取，对已经过期的 key 执行删除操作。大致流程如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/yiaMkmW2kMZHzTWaz2k7zS5FCgWOo3VCTOyGgicWxAbGiasykbIaVY3rob1PlBDt5xra2u4N4bvGtVfic79Tss8xkg/640?wx_fmt=png&from=appmsg)

慢周期删除流程

Redis 使用 ae 框架在单线程内先执行 call before/after sleep 事件，文件事件（网络读写 / 执行），再执行时间事件（定时任务）。慢周期则在时间事件中执行。如上文所分析的，如果大量 key 同时过期，可能会出现一段时间内周期性阻塞 25ms （默认配置）的情况。此时 Redis 对外的性能就会较大下降。

#### **2.2.2 请求穿透到数据库**

Redis 中大量的数据同时过期，其结果就是业务请求时间会随之增高。业务判断缓存层查询超时，自动降级到数据库层。请求降级到数据库层后，导致数据库层压力增加。进而导致前端用户请求开始重试，加剧了 Redis CPU 增高。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/yiaMkmW2kMZHzTWaz2k7zS5FCgWOo3VCTjiazC2jsUqicr16QpXFuMXl2icqG84UNrMRZC1V769Qc9N23u6iaEfyP1g/640?wx_fmt=png&from=appmsg)

降级查询流程

**3.  解决思路**
------------

找到根因后，对业务中凌晨 00:00 过期的缓存进行改造，避免集中过期对业务的影响。本次改造的重点是避免在某一时间段出现大量过期 Key 导致 Redis 处理性能的降低，而不是预防缓存雪崩问题。所以只需要考虑如何在不影响原本业务的前提下，将缓存过期的时间从凌晨十二点调整到其他时间段，利用空间换时间。

Key 过期对业务而言又可以分为两类：

1.  Key 过期时间与业务不是强相关的，比如缓存记录一些热点数据，这类数据的缓存过期时间对业务的影响不大。
    
2.  Key 过期时间与业务强相关，比如缓存记录用户当日的抽奖次数，这类数据的缓存过期时间业务上不允许将缓存过期时间提前或延后。
    

### **3.1 解决方案**

本文主要针对以上提到的第二种情况进行缓存过期时间的改造。有以下两种解决方案：

1.  缓存过期时间设置为随机，缓存值中存储真实的过期时间
    
2.  为缓存 Key 添加一个时间后缀
    

#### **3.1.1 缓存值中存储真实过期时间**

为 key 设置一个随机的过期时间（原过期时间 + 随机值）。同时，在设置缓存值时，新增一个用于记录真实过期时间的字段。当业务代码在访问该缓存时，需要额外通过缓存值中的真实过期时间来判断当前缓存是否失效。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/yiaMkmW2kMZHzTWaz2k7zS5FCgWOo3VCT5YEYic2fpCl9bKVTZmGn9ID0JPpG3ZXRMhOgztBUCEicROw0OSF7BGKA/640?wx_fmt=png&from=appmsg)

value 保存过期时间

该方案的优点在于，可以将过期的缓存分散到不同的时间点，有效避免了缓存集中在同一个时间段过期。但该方案的缺点也很明显，就是对原有业务代码的改动过大。不仅需要改变原来缓存值的数据结构，同时还需要在业务代码上加入当前缓存是否失效的判断语句。

**3.1.2 缓存 Key 添加时间后缀**

为 key 设置一个随机的过期时间（原过期时间 + 随机值）。同时为 Key 添加一个时间后缀。假设缓存 Key 需要零点过期，那么我们就为缓存 Key 添加一个当天日期的时间后缀。

当第一天用户访问该缓存时，会携带第一天的日期作为 Key 的后缀。第二天用户访问该缓存时，会带上第二天的日期作为 Key 的后缀，此时前一天的缓存 Key 在逻辑上其实已经是处于过期状态了（访问不到以第一天日期作为后缀的 Key，而且该 key 也设置了过期时间，到期时 Redis 会自动删除该 Key）。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/yiaMkmW2kMZHzTWaz2k7zS5FCgWOo3VCTp6bX0qAib0vrKld8BpUoYib7uXibJL5p2icL0MuyhrurK5cs8seQSZDzaQ/640?wx_fmt=png&from=appmsg)

key 保存过期时间

该方案与方案 1 的优点相同，可以有效避免缓存集中在同一个时间段过期。同时，相比于方案 1，该方法的代码改动量小，且不需要改变原来缓存值的数据结构。

但该方案也有缺点：

首先，需要精准的授时服务保证分布式应用时钟偏差在较小范围内。

其次，在某些时间段内，部分缓存数据会占用双倍的内存空间。如果对 Redis 的内存空间敏感，该方法可能不太适用。（如上图所示，用户 A 在 [2023-10-16 10:38:37,2023-10-16 13:11:56] 这段时间内，将会占用双倍的内存空间）

**4.  总结**
----------

业务通过上文提到的两个方案改造上线后，相比于之前 Redis 的性能表现有了明显的提升。在零点高峰阶段，已经不存在 CPU 100% 的情况了。

改造前

![](https://mmbiz.qpic.cn/sz_mmbiz_png/yiaMkmW2kMZHzTWaz2k7zS5FCgWOo3VCTNHjBNme1ezYg8mw1hj60micEbDZOm3MG7KgvwUSGchiaMtRAWUMibt87Q/640?wx_fmt=png&from=appmsg)

优化前 CPU 占用情况

改造后

![](https://mmbiz.qpic.cn/sz_mmbiz_png/yiaMkmW2kMZHzTWaz2k7zS5FCgWOo3VCTFX8W93bibSvHB3HicicIia2bFW37gVFQzXdgaGUMVyCams7YibuEnh6zU8g/640?wx_fmt=png&from=appmsg)

优化后 CPU 占用情况

综上，在线上发生告警时需要熟练掌握组件原理，并对告警的上下文进行合理分析。找到具体引发的根因才能结合业务找到合适的解决方案。

我们在日常编码的时候，也需要考虑到数据体量的整体增加对性能影响。在项目上线初期，数据量较少时不会触发相关故障，当数据量到达一定程度就要预先做好相关的代码优化、容量评估工作。

**联系我们**

朴朴科技 - 用户增长组，主要负责拉新、 LTV 、触达、活动等相关功能的落地与实践，团队内成员是 Apache Dubbo、Spring Cloud、etcd 等众多优秀开源社区的贡献者。

如果你期待参与业务飞速增长的旅程，亲自推动业务、技术、价值的落地，陪伴影响力深远的技术团队茁壮成长。请加入我们。

戳原文链接了解加入信息