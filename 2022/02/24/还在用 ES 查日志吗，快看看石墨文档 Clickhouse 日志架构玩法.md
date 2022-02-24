> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653558910&idx=1&sn=9bec04afef9a2a4d39f0d6a7379bf82d&chksm=813989e6b64e00f054c741beb190d62485f8ff121040ae9f6811ea709f0a5916c34c9d314a2d&scene=21#wechat_redirect)

1 背景
----

*   成本问题：
    

*   SLS 个人觉得是一个非常优秀的产品，速度快，交互方便，但是 SLS 索引成本比较贵
    
*   我们想减少 SLS 索引成本的时候，发现云厂商并不支持分析单个索引的成本，导致我们无法知道是哪些索引构建的不够合理
    

*   ES 使用的存储非常多，并且耗费大量的内存
    

*   通用问题：
    

*   如果业务是混合云架构，或者业务形态有 SAAS 和私有化两种方式，那么 SLS 并不能通用
    
*   日志和链路，需要用两套云产品，不是很方便
    

*   精确度问题：SLS 存储的精度只能到秒，但我们实际日志精度到毫秒，如果日志里面有 traceid，SLS 中无法通过根据 traceid 信息，将日志根据毫秒时间做排序，不利于排查错误
    

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOMWWU8ZTZO7g79A0N2xvpdkZBwibyvClDdvhzYSEJT6cRYRZe7vyMtMmakJsGKnCkHv5VBPRh7peiaw/640?wx_fmt=png)

2 架构原理图
-------

*   日志采集：LogCollector 采用 Daemonset 方式部署，将宿主机日志目录挂载到 LogCollector 的容器内，LogCollector 通过挂载的目录能够采集到应用日志、系统日志、K8S 审计日志等
    
*   日志传输：通过不同 Logstore 映射到 Kafka 中不同的 Topic，将不同数据结构的日志做了分离
    

*   日志存储：使用 Clickhouse 中的两种引擎数据表和物化视图
    
*   日志管理：开源的 Mogo 系统，能够查询日志，设置日志索引，设置 LogCollector 配置，设置 Clickhouse 表，设置报警等
    

以下我们按照这四大部分，阐述其中的架构原理

3 日志采集
------

### **3.1 采集方式**

Kubernetes 容器内日志收集的方式通常有以下三种方案

*   DaemonSet 方式采集：在每个 node 节点上部署 LogCollector，并将宿主机的目录挂载为容器的日志目录，LogCollector 读取日志内容，采集到日志中心。
    
*   网络方式采集：通过应用的日志 SDK，直接将日志内容采集到日志中心 。
    

*   SideCar 方式采集：在每个 pod 内部署 LogCollector，LogCollector 只读取这个 pod 内的日志内容，采集到日志中心。
    

以下是三种采集方式的优缺点：

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOMWWU8ZTZO7g79A0N2xvpdkfaY5RO1OWfzwuJ2e8261DlBRZLuLpvSPSPKmPfMJkVEib8kPVQWibdGA/640?wx_fmt=png)

我们主要采用 DaemonSet 方式和网络方式采集日志。DaemonSet 方式用于 ingress、应用日志的采集，网络方式用于大数据日志的采集。以下我们主要介绍下 DeamonSet 方式的采集方式。

### **3.2 日志输出**

从上面的介绍中可以看到，我们的 DaemonSet 会有两种方式采集日志类型，一种是标准输出，一种是文件。

引用元乙的描述：虽然使用 Stdout 打印日志是 Docker 官方推荐的方式，但大家需要注意：这个推荐是基于容器只作为简单应用的场景，实际的业务场景中我们还是建议大家尽可能使用文件的方式，主要的原因有以下几点：

*   Stdout 性能问题，从应用输出 stdout 到服务端，中间会经过好几个流程（例如普遍使用的 JSONLogDriver）：应用 stdout -> DockerEngine -> LogDriver -> 序列化成 JSON -> 保存到文件 -> Agent 采集文件 -> 解析 JSON -> 上传服务端。整个流程相比文件的额外开销要多很多，在压测时，每秒 10 万行日志输出就会额外占用 DockerEngine 1 个 CPU 核；
    
*   Stdout 不支持分类，即所有的输出都混在一个流中，无法像文件一样分类输出，通常一个应用中有 AccessLog、ErrorLog、InterfaceLog（调用外部接口的日志）、TraceLog 等，而这些日志的格式、用途不一，如果混在同一个流中将很难采集和分析；
    

*   Stdout 只支持容器的主程序输出，如果是 daemon/fork 方式运行的程序将无法使用 stdout；
    
*   文件的 Dump 方式支持各种策略，例如同步 / 异步写入、缓存大小、文件轮转策略、压缩策略、清除策略等，相对更加灵活。
    

### **3.3 日志目录**

以下列举了日志目录的基本情况

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOMWWU8ZTZO7g79A0N2xvpdk3epwTxZs4QxiaVM4mAPFs2ibEepmk4O7OOaNLQYCaaPib2jVVmg16WHVw/640?wx_fmt=png)

#### **3.3.1 标准输出日志目录**

我们参照 K8S 日志的规范：/var/log/containers/%{DATA:pod_name}_%{DATA:namespace}_%{GREEDYDATA:container_name}-%{DATA:container_id}.log。可以将 nginx-ingress 日志解析为：

*   pod_name：nginx-ingress-controller-mt2w
    
*   namespace：kube-system
    

*   container_name：nginx-ingress-controller
    
*   container_id：be3741043eca1621ec4415fd87546b1beb29480ac74ab1cdd9f52003cf4abf0a
    

通过以上的日志解析信息，我们的 LogCollector 就可以很方便的追加 pod、namespace、container_name、container_id 的信息。

#### **3.3.2 容器信息目录**

应用的容器信息存储在 / var/lib/docker/containers 目录下，目录下的每一个文件夹为容器 ID，我们可以通过 cat config.v2.json 获取应用的 docker 基本信息。

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOMWWU8ZTZO7g79A0N2xvpdkicmn3dTxIC5ibSMJaqqCiaibrQWuw13hpn0NHPwILeUwxz9czVQXgGZ22A/640?wx_fmt=png)

### **3.4 LogCollector 采集日志**

#### **3.4.1 配置**

我们 LogCollector 采用的是 fluent-bit，该工具是 cncf 旗下的，能够更好的与云原生相结合。通过 Mogo 系统可以选择 Kubernetes 集群，很方便的设置 fluent-bit configmap 的配置规则。

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOMWWU8ZTZO7g79A0N2xvpdkC4OJeBFrw6A3MOFOkrQ61HI5LpnxEJibqAPeMaEj2ibHU9g1iaTwjGnmQ/640?wx_fmt=png)

#### **3.4.2 数据结构**

fluent-bit 的默认采集数据结构

*   @timestamp 字段：string or float，用于记录采集日志的时间
    
*   log 字段：string，用于记录日志的完整内容
    

Clickhouse 如果使用 @timestamp 的时候，因为里面有 @特殊字符，会处理的有问题。所以我们在处理 fluent-bit 的采集数据结构，会做一些映射关系，并且规定双下划线为 Mogo 系统日志索引，避免和业务日志的索引冲突。

*   _time_字段：string or float，用于记录采集日志的时间
    
*   _log_字段：string，用于记录日志的完整内容
    

例如你的日志记录的是 {"id":1}，那么实际 fluent-bit 采集的日志会是 {"_time_":"2022-01-15...","_log_":"{\"id\":1}" 该日志结构会直接写入到 kafka 中，Mogo 系统会根据这两个字段_time_、_log_设置 clickhouse 中的数据表。

#### **3.4.3 采集**

如果我们要采集 ingress 日志，我们需要在 input 配置里，设置 ingress 的日志目录，fluent-bit 会把 ingress 日志采集到内存里。

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOMWWU8ZTZO7g79A0N2xvpdkYZpmgZLoLj7ZX7gA3qdriamoOibNedjTiao4J9DXnib1obctuM7A51gNng/640?wx_fmt=png)

然后我们在 filter 配置里，将 log 改写为_log_

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOMWWU8ZTZO7g79A0N2xvpdkVFzgd5ia3xNIFicE5YPdsnicFQvwY0ib4pYXSicNrWeS3yxSEaBoOMKPiaPg/640?wx_fmt=png)

然后我们在 ouput 配置里，将追加的日志采集时间设置为_time_，设置好日志写入的 kafka borkers 和 kafka topics，那么 fluent-bit 里内存的日志就会写入到 kafka 中

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOMWWU8ZTZO7g79A0N2xvpdkyfY4HSiaUSrAp3KH85lanDSq2zePxlIeZ808DZmvTI5DYfO3u0XLfyA/640?wx_fmt=png)

日志写入到 Kafka 中_log_需要为 json，如果你的应用写入的日志不是 json，那么你就需要根据 fluent-bit 的 parser 文档，调整你的日志写入的数据结构：https://docs.fluentbit.io/manual/pipeline/filters/parser

4 日志传输
------

Kafka 主要用于日志传输。上文说到我们使用 fluent-bit 采集日志的默认数据结构，在下图 kafka 工具中我们可以看到日志采集的内容。

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOMWWU8ZTZO7g79A0N2xvpdkT46icVqAsvxjC6SWL8d6eaeP3H8jdAoNHv0jaKy1Kdsg4OKsI7UgD1Q/640?wx_fmt=png)

在日志采集过程中，会由于不用业务日志字段不一致，解析方式是不一样的。所以我们在日志传输阶段，需要将不同数据结构的日志，创建不同的 Clickhouse 表，映射到 Kafka 不同的 Topic。这里以 ingress 为例，那么我们在 Clickhouse 中需要创建一个 ingress_stdout_stream 的 Kafka 引擎表，然后映射到 Kafka 的 ingress-stdout Topic 里。

5 日志存储
------

我们会使用三种表，用于存储一种业务类型的日志。

*   Kafka 引擎表：将数据从 Kafka 采集到 Clickhouse 的 ingress_stdout_stream 数据表中
    

```
create table logger.ingress_stdout_stream
(
	_source_ String,
	_pod_name_ String,
	_namespace_ String,
	_node_name_ String,
	_container_name_ String,
	_cluster_ String,
	_log_agent_ String,
	_node_ip_ String,
	_time_ Float64,
	_log_ String
)
engine = Kafka SETTINGS kafka_broker_list = 'kafka:9092', kafka_topic_list = 'ingress-stdout', kafka_group_name = 'logger_ingress_stdout', kafka_format = 'JSONEachRow', kafka_num_consumers = 1;

```

*   物化视图：将数据从 ingress_stdout_stream 数据表读取出来，_log_根据 Mogo 配置的索引，提取字段在写入到 ingress_stdout 结果表里
    

```
CREATE MATERIALIZED VIEW logger.ingress_stdout_view TO logger.ingress_stdout AS
SELECT
    toDateTime(toInt64(_time_)) AS _time_second_,
fromUnixTimestamp64Nano(toInt64(_time_*1000000000),'Asia/Shanghai') AS _time_nanosecond_,
	_pod_name_,
	_namespace_,
	_node_name_,
	_container_name_,
	_cluster_,
	_log_agent_,
	_node_ip_,
	_source_,
	_log_ AS _raw_log_,JSONExtractInt(_log_, 'status') AS status,JSONExtractString(_log_, 'url') AS url
	FROM logger.ingress_stdout_stream where 1=1;

```

*   结果表：存储最终的数据
    

```
create table logger.ingress_stdout
(
	_time_second_ DateTime,
	_time_nanosecond_ DateTime64(9, 'Asia/Shanghai'),
	_source_ String,
	_cluster_ String,
	_log_agent_ String,
	_namespace_ String,
	_node_name_ String,
	_node_ip_ String,
	_container_name_ String,
	_pod_name_ String,
	_raw_log_ String,
	status Nullable(Int64),
	url Nullable(String),
)
engine = MergeTree PARTITION BY toYYYYMMDD(_time_second_)
ORDER BY _time_second_
TTL toDateTime(_time_second_) + INTERVAL 7 DAY
SETTINGS index_granularity = 8192;

```

6 总结流程
------

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOMWWU8ZTZO7g79A0N2xvpdkWYJ5E88z5XjCbpsQeIcibtAgPGeLlL1WOlicJTZp7ODZt9PvSx6SSTHQ/640?wx_fmt=png)

*   日志会通过 fluent-bit 的规则采集到 kafka，在这里我们会将日志采集到两个字段里
    

*   _time_字段用于存储 fluent-bit 采集的时间
    
*   _log_字段用于存放原始日志
    

*   通过 mogo，在 clickhouse 里设置了三个表
    

*   app_stdout_stream：将数据从 Kafka 采集到 Clickhouse 的 Kafka 引擎表
    
*   app_stdout_view：视图表用于存放 mogo 设置的索引规则
    

*   app_stdout：根据 app_stdout_view 索引解析规则，消费 app_stdout_stream 里的数据，存放于 app_stdout 结果表中
    

*   最后 mogo 的 UI 界面，根据 app_stdout 的数据，查询日志信息
    

7 Mogo 界面展示
-----------

查询日志界面

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOMWWU8ZTZO7g79A0N2xvpdk8yZTBHx0FTNqSZ9ub5wZBVK0g9G9Wlas5UNibBRjJibMxiadPBWtYVJ3A/640?wx_fmt=png)

设置日志采集配置界面

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOMWWU8ZTZO7g79A0N2xvpdkyuzaKekCzoApdLx6iaDOmDj89icaC6pdqQjR2ZZmn6LOb12GficSiaMAvg/640?wx_fmt=png)

以上文档描述是针对石墨 Kubernetes 的日志采集，想了解物理机采集日志方案的，可以在下文中找到《Mogo 使用文档》的链接，运行 docker-compose 体验 Mogo 全部流程，查询 Clickhouse 日志。限于篇幅有限，Mogo 的日志报警功能，下次再讲解。

8 资料
----

*   github 地址: https://github.com/shimohq/mogo
    
*   Mogo 文档：https://mogo.shimo.im
    

*   Mogo 使用文档：https://mogo.shimo.im/doc/AV62KU4AABMRQ
    
*   fluent-bit 文档：https://docs.fluentbit.io/
    

*   K8S 日志
    

*   6 个 K8S 日志系统建设中的典型问题，你遇到过几个：https://developer.aliyun.com/article/718735
    
*   一文看懂 K8S 日志系统设计和实践：https://developer.aliyun.com/article/727594
    

*   9 个技巧，解决 K8S 中的日志输出问题：https://developer.aliyun.com/article/747821
    
*   直击痛点，详解 K8S 日志采集最佳实践：https://developer.aliyun.com/article/749468?spm=a2c6h.14164896.0.0.24031164UoPfIX
    

*   Clickhouse
    

*   Clickhouse 官方文档：https://clickhouse.com/
    
*   Clickhouse 作为 Kubernetes 日志管理解决方案中的存储：http://dockone.io/article/9356
    

*   Uber 如何使用 ClickHouse 建立快速可靠且与模式无关的日志分析平台？：https://www.infoq.cn/article/l4thjgnr7hxpkgpmw6dz
    
*   干货 | 携程 ClickHouse 日志分析实践：[https://mp.weixin.qq.com/s/IjOWAPOJXANRQqRAMWXmaw](https://mp.weixin.qq.com/s?__biz=MjM5MDI3MjA5MQ==&mid=2697269343&idx=1&sn=d10eba09aa33d0b8756ee259910f06e2&scene=21#wechat_redirect)
    

*   为什么我们要从 ES 迁移到 ClickHouse：[https://mp.weixin.qq.com/s/l4RgNQPxvdNIqx52LEgBnQ](https://mp.weixin.qq.com/s?__biz=MjM5ODI5Njc2MA==&mid=2655836923&idx=1&sn=284ac51a6e74c3ed1784d7c8d27acd14&scene=21#wechat_redirect)
    
*   ClickHouse 在日志存储与分析方面作为 ElasticSearch 和 MySQL 的替代方案：[https://mp.weixin.qq.com/s/nJXorcgi0QfXPCKr_HdUZg](https://mp.weixin.qq.com/s?__biz=MzIxMTE0ODU5NQ==&mid=2650244709&idx=1&sn=333d74e765861036510be2e57d2c3681&scene=21#wechat_redirect)
    

*   快手、携程等公司转战到 ClickHouse，ES 难道不行了？：[https://mp.weixin.qq.com/s/hP0ocT-cBCeIl9n1wL_HBg](https://mp.weixin.qq.com/s?__biz=MzIzMzgxOTQ5NA==&mid=2247549780&idx=2&sn=c8cc0841104373a26a5163f469f62381&scene=21#wechat_redirect)
    
*   日志分析下 ES/ClickHouse/Loki 比较与思考：[https://mp.weixin.qq.com/s/n2I94X6tz2jOABzl1djxYg](https://mp.weixin.qq.com/s?__biz=MzI0MjAxNjk0OA==&mid=2247485948&idx=3&sn=7c02fad7d064b9aac88817715abb581d&scene=21#wechat_redirect)