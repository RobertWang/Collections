> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/axNxUpwtq3Why0UbuAQF8Q)

**导读：**  

全量采集 Trace 数据 (日增数百 TB 、数千亿条 Span 数据) 并以较低的成本保证数据的实时处理与高效查询，对 Trace2.0 后端整体的可观测性解决方案提出了极高的要求。本文将详细介绍 Trace2.0 背后的架构设计、尾部采样和冷热存储方案，以及我们是如何通过自建存储实现进一步的降本增效(存储成本下降 66%)。

**1. 整体架构设计**
=============

*   **客户端 & 数据采集：**集成并定制 OpenTelemetry 提供的多语言 SDK(Agent)，生成统一格式的可观测数据。
    
*   **控制平面 Control Plane：**统一的配置中心向数据采集侧下发各类动态配置发并实时生效；支持向各采集器下发动态配置并实时生效，支持应用按实例数灰度接入，并提供出入参收集动态开关、性能剖析动态开关、流量染色动态配置、客户端版本管理等。
    
*   **数据收集服务 OTel Server：**数据收集器 OTel Server 兼容 OpenTelemetry Protocol（OTLP) 协议，提供 gRPC 和 HTTP 两种方式接收采集器发送的可观测数据。
    
*   **分析计算 & 存储 OTel Storage：**计算侧除了基础的实时检索能力外，还提供了场景化的数据分析计算主要包括：
    

*   **存储 Trace 数据：**数据分为两段，一段是索引字段，包括 TraceID、ServiceName、SpanName、StatusCode、Duration 和起止时间等基本信息，用于高级检索；另一段是明细数据 (源数据，包含所有的 Span 数据)
    
*   **计算 SpanMetrics 数据：**聚合计算 Service、SpanName、Host、StatusCode、Env、Region 等维度的执行总次数、总耗时、最大耗时、最小耗时、分位线等数据；
    
*   **业务单号关联 Trace：**电商场景下部分研发多以订单号、履约单号、汇金单号作为排障的输入，因此和业务研发约定特殊埋点规则后 -- 在 Span 的 Tag 里添加一个特殊字段 "bizOrderId={实际单号}"-- 便将这个 Tag 作为 ClickHouse 的索引字段；从而实现业务链路到全链路 Trace 形成一个完整的排障链路；
    
*   **Redis 热点数据统计：**在客户端侧扩展调用 Redis 时入参和出参 SpanTag 埋点，以便统 Redis 命中率、大 Key、高频写、慢调用等指标数据；
    
*   **MySQL** **热点数据统计：**按照 SQL 指纹统计调用次数、慢 SQL 次数以及关联的接口名。
    

**2. 尾部采样 & 冷热存储**
==================

全量存储 Trace 数据不仅会造成巨大的成本浪费，还会显著地影响整条数据处理链路的性能以及稳定性。所以，如果我们能够只保存那些有价值、大概率会被用户实际查询的 Trace，就能取得成本与收益的平衡。那什么是有价值的 Trace 呢？根据日常排查经验，我们发现业务研发主要关心以下四类优先级高场景：

1.  在调用链上出现了异常 ERROR；
    
2.  在调用链上出现了大于「200ms」的数据库调用；
    
3.  整个调用链耗时超过「1s」；
    
4.  业务场景的调用链，比如通过订单号关联的调用链。
    

在这个背景下，并结合业界的实践经验，落地 Trace2.0 的过程中设计了尾部采样 & 冷热分层存储方案，方案如下:

*   「3 天」内的 Trace 数据全量保存，定义为热数据。
    
*   基于 Kafka 延迟消费 + Bloom Filter 尾部采样的数据 (错、慢、自定义采样规则、以及默认常规 0.1% 采样数据) 保留「30 天」，定义为冷数据。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74Bs4mwibUGtJcE5zUdvgdsKfKLIrOf3wuYtGaEFF1Hibhr6Qo8kl2esdeHkz7tfV1HL2K0zibPZRuAdg/640?wx_fmt=png)

整体处理流程如下：

*   **OTel Server 数据收集 & 采样规则：**将客户端采集器上报的全量 Trace 数据实时写入 Kafka 中，并把满足采样规则 (上述定义的场景) 的 Span 数据对应的 TraceID 记录到 Bloom Filter 中;
    
*   **OTel Storage 持久化热数据：**实时消费 Kafka 中数据，并全量持久化到 ClickHouse 热集群中；
    
*   **OTel Storage 持久化冷数据：**订阅上游 OTel Server 的 Bloom Filter，**延迟**消费 Kafka 中的数据，将 TraceID 在 Bloom Filter 中可能存在的 Span 数据持久化到 ClickHouse 冷集群中；
    

*   延迟时间配置的 30 分钟，尽量保证一个 Trace 下的 Span 完整保留。
    

*   **TraceID 点查：** Trace2.0 自定义了 TraceID 的生成规则；在生成 TraceID 时，会把当前时间戳秒数的 16 进制编码结果 (占 8 个字节) 作为 TraceID 的一部分。查询时只需要解码 TraceId 中的时间戳，即可知道应该查询热集群还是冷集群。
    

接下来再介绍一下尾部采样中 Bloom Filter 的设计细节，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74Bs4mwibUGtJcE5zUdvgdsKfRKpDeoxUQPTicNJTywnPvZdPhOeglPo6toib3sEnmiamVE0gaqrAcWmvQ/640?wx_fmt=png)

整体处理流程如下：

1.  OTel Server 会将满足采样规则的 Span 数据对应的 TraceID，根据 TraceID 中的时间戳写入到对应时间戳的 Bloom Filter 中；
    
2.  Bloom Filter 会按十分钟粒度 (可根据实际的数据量并结合 BloomFilter 的误算率和样本大小计算内存消耗并调整) 进行分片，十分钟过后将 Bloom Filter 进行序列化并写入到 ClickHouse 存储中；
    
3.  OTel Storage 消费侧拉取 Bloom Filter 数据 (注意：同一个时间窗口，每一个 OTel Server 节点都会生成一个 BloomFilter) 并进行合并 Merge(减少 Bloom Filter 的内存占用并提高 Bloom Filter 的查询效率)。
    

**3. 自建存储 & 降本增效**
==================

**3.1 基于 SLS-Trace 的解决方案**
--------------------------

1.  利用 OpenTelemetry Collector _aliyunlogserverexporter_【2】将 Trace 数据写入到 SLS-Trace Logstore 中；
    
2.  SLS-Trace 通过默认提供的 Scheduled SQL 任务定时聚合 Trace 数据并生成相应的 Span 指标与应用、接口粒度的拓扑指标等数据。
    

随着 Trace2.0 在公司内部全面铺开，SLS 的存储成本压力变得越来越大，为了响应公司 “利用技术手段实现降本提效” 的号召，我们决定自建存储。

**3.2 基于 ClickHouse 的解决方案**
---------------------------

目前业内比较流行的全链路追踪开源项目 (SkyWalking、Pinpoint、Jaeger 等) 采用的存储大都是基于 ES 或者 HBase 实现的。而近几年新兴的开源全链路追踪开源项目(_Uptrace_【3】、_Signoz_【4】等)采用的存储大都是基于 ClickHouse 实现的，同时将 Span 数据清洗出来的指标数据也存储在 ClickHouse 中。且 ClickHouse 的物化视图 (很好用) 也很好地解决了指标数据降采样 (DownSampling) 的问题。最终经过一番调研，我们决定基于 ClickHouse 来自建新的存储解决方案。整体架构图如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74Bs4mwibUGtJcE5zUdvgdsKfuOeH6I4OarTA8ia6OZr16HRdkvNnUnicCeduTuuiaAQdNANtY1LKjicgfg/640?wx_fmt=png)

整体处理流程如下：

*   **Trace 索引 & 明细数据：**OTel Storage 会将基于 Span 原始数据构建的索引数据写入到 SpanIndex 表中，将 Span 原始明细数据写入到 SpanData 表中 (相关表设计可以参考 _Uptrace_【5】)；
    
*   **计算 & 持久化 SpanMetrics 数据：**OTel Storage 会根据 Span 的 Service、SpanName、Host、StatusCode 等属性统计并生成「30 秒」粒度的总调用次数、总耗时、最大耗时、最小耗时、分位线等指标数据，并写入到 SpanMetrics 表；
    

*   **指标** **Down****S****ampling 功能****：**利用 ClickHouse 的物化视图将「秒级」指标聚合成「分钟级」指标，再将「分钟级」指标聚合成「小时级」指标；从而实现多精度的指标以满足不同时间范围的查询需求；
    

*   **元数据** **(****上下游拓扑数据****)****：**OTel Storage 根据 Span 属性中的上下游关系 (需要在客户端埋相关属性)，将拓扑依赖关系写入到图数据库 Nebula 中。
    

*   ### **ClickHouse 写入细节**
    

ClickHouse 使用 Distributed 引擎实现了 Distributed(分布式)表机制，可以在所有分片 (本地表) 上建立视图，实现分布式查询。并且 Distributed 表自身不会存储任何数据，它会通过读取或写入其他远端节点的表来进行数据处理。SpanData 表创建语句如下所示：

```
-- span_data
CREATE TABLE IF NOT EXISTS '{database}'.span_data_local ON CLUSTER '{cluster}'
(
    traceID                   FixedString(32),
    spanID                    FixedString(16),
    startTime                 DateTime64(6 ) Codec (Delta, Default),
    body                      String CODEC (ZSTD(3))
) ENGINE = MergeTree
ORDER BY (traceID,startTime,spanID)
PARTITION BY toStartOfTenMinutes(startTime)
TTL toDate(startTime) + INTERVAL '{TTL}' HOUR;
-- span_data_distributed
CREATE TABLE IF NOT EXISTS '{database}'.span_data_all ON CLUSTER '{cluster}'
as '{database}'.span_data_local
    ENGINE = Distributed('{cluster}', '{database}', span_data_local,
                         xxHash64(concat(traceID,spanID,toString(toDateTime(startTime,6)))));

```

整体写入流程比较简单 (注意：避免使用分布式表)，如下所示：

*   定时获取 ClickHouse 集群节点；
    
*   通过 Hash 函数选择对应的 ClickHouse 节点，然后批量写 ClickHouse 的本地表。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74Bs4mwibUGtJcE5zUdvgdsKf9I9FKbIlk3xZkA4z5pOON4Xz8AmfC2ZxBoLGj1FV1so1tlOfhkrvXA/640?wx_fmt=png)

*   ### **上线效果**
    

全链路追踪是一个典型的写多读少的场景，因此我们采用了 ClickHouse ZSTD 压缩算法对数据进行了压缩，压缩后的压缩比高达 12，效果非常好。目前 ClickHouse 冷热集群各使用数十台 16C64G ESSD 机器，单机写入速度 25w/s(ClickHouse 写入的行数)。相比于初期的阿里云 SLS-Trace 方案，存储成本下降 66%，查询速度也从 800+ms 下降至 490+ms。

*   ### **下一步规划**
    

目前 Trace2.0 将 Span 的原始明细数据也存储在了 ClickHouse 中，导致 ClickHouse 的磁盘使用率会有些偏高，后续考虑将 Span 明细数据先写入 HDFS/OSS 等块存储设备中，ClickHouse 来记录每个 Span 在块存储中的 offset，从而进一步降低 ClickHouse 的存储成本。

**关于我们：**

得物监控团队提供一站式的可观测性平台，负责链路追踪、时序数据库、日志系统，包括自定义大盘、应用大盘、业务监控、智能告警、AIOPS 等排障分析。

欢迎对可观测性 / 监控 / 告警 / AIOPS 等领域感兴趣的同学加入我们。

【2】**SLS-Trace Contrib** 

https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/exporter/alibabacloudlogserviceexporter

【3】**Uptrace** 

https://uptrace.dev/

https://github.com/uptrace/uptrace/tree/v0.2.16/pkg/bunapp/migrations

本篇是**《得物云原生全链路追踪 Trace2.0》**系列开篇，更多内容请关注 “得物技术” 公众号。

*   得物云原生全链路追踪 Trace2.0 架构实践
    

*   得物云原生全链路追踪 Trace2.0 产品篇
    

*   得物云原生全链路追踪 Trace2.0 采集篇
    

*   得物云原生全链路追踪 Trace2.0 数据挖掘篇
    

*** 文** **/ 南风**