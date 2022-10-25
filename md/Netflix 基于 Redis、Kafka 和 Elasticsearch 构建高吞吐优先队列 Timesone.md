> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2651144383&idx=4&sn=b94e40f94b65db0e41a3163c6c101311&chksm=bdb8b8ec8acf31fa004b459f9f1103bde0cb409c1ab4dc2505d04ec1b53e204c719b302eaaee&mpshare=1&scene=1&srcid=10233TYXhwqMX6EOL2BK5WcO&sharer_sharetime=1666526354328&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

> 最近，Netflix 公布了它是如何构建 Timestone 的——一个高吞吐、低延迟的优先队列系统。

作者 | Eran Stiller

译者 | 明知山

策划 | 丁晓昀

最近，Netflix 公布了它是如何构建 Timestone 的——一个高吞吐、低延迟的优先队列系统。Netflix 使用 Redis、Apache Kafka、Apache Flink 和 Elasticsearch 等开源组件来构建这个队列系统。Netflix 的工程师们表示，他们之所以要构建 Timestone，是因为他们无法找到满足其所有要求的现成解决方案。

其中一个需求是不需要在消费者端进行任何锁定或协调的情况下将某些工作项标记为不可并行。这一需求意味着在属于同一工作集的前一个项目完成之前，Timestone 不应该发送消息。Timestone 引入了 “独占队列（Exclusive Queue）” 的概念来实现这一目的。

Netflix 的软件工程师 Kostas Christidis 解释了独占队列的工作原理。

> 独占队列被创建后将与用户定义的独占键相关联——例如，“project”。所有发布到该队列的消息都必须在其元数据中携带此键。例如，带有 "project=foo" 的消息将被接收到独占队列中，不包含该键的消息将不会进入独占队列。在这个例子中，与独占键对应的值是 “foo”，也就是消息的独占值。独占队列的约定是，在任何时间点，每个独占值最多只能有一个消费者。因此，如果我们示例中以“project-” 为前缀的独占队列中有两个消息的键值对为“project=foo”，并且其中一个消息已经分配给了一个消费者，那么另一个消息就不能退出队列。

下图描绘了这个示例。

![](https://mmbiz.qpic.cn/mmbiz_jpg/YriaiaJPb26VP5GmOToWsSXf4jwHwAwNRFdPKZtOSTSQtbwENsQXdI9h2mIObPJdIWX5lU2Va1IRDMctbfZglj2A/640?wx_fmt=jpeg)

当 worker_2 发出出队列调用时，会收到 msg_2 而不是 msg_1，即使 msg_1 具有更高的优先级

来源：https://netflixtechblog.com/timestone-netflixs-high-throughput-low-latency-priority-queueing-system-with-built-in-support-1abf249ba95f

另一个需求是，在任何给定的时间，一条消息只能分配给一个消费者。这很重要，因为 Cosmos 种的工作负载往往是资源密集型的，并且可能扇出数千个动作，这个需求的目标之一便是减少资源浪费。这个需求排除了最终一致性解决方案，这意味着 Netflix 的工程师想要的是队列级别的线性一致性。

Netflix 工程师通过为每条消息维护一个消息状态来实现这一需求。当生产者将消息入队时，消息将被设置为 “Pending” 或“Invisible”状态，这取决于消息的超时设置（可选）。当消费者将挂起的消息从队列中取出时，它将获得该消息的独占租约，Timestone 将该消息设置为 “Running” 状态。在这个阶段，生产者可以将消息标记为 “Completed” 或“Cancelled”。每条消息最多可以尝试有限的取出次数，然后 Timestone 将其标记为 “Errored” 状态。下图说明了所有可能的状态转换。

![](https://mmbiz.qpic.cn/mmbiz_jpg/YriaiaJPb26VP5GmOToWsSXf4jwHwAwNRFGAiccg6tfqicj9PvNXo5299G64C6MHje2O9nTOMjUb3uC3nickwuibkwzQ/640?wx_fmt=jpeg)

来源：https://netflixtechblog.com/timestone-netflixs-high-throughput-low-latency-priority-queueing-system-with-built-in-support-1abf249ba95f

Timestone 服务器提供了一个基于 gRPC 的接口。所有 API 操作都在队列作用域内。所有修改状态的 API 操作都是幂等的。记录系统是一个 Redis 集群。在将响应发送回服务器之前，Redis 会将每个写请求持久化到事务日志中。在 Redis 内部使用了一个按优先级排序的排序集代表每个队列。消息和队列配置以散列值的方式存储。

Christidis 提到了 Netflix 工程师如何用 Redis 实现原子性：

几乎所有 Timestone 和 Redis 之间的交互都写在 Lua 脚本中。在大多数 Lua 脚本中，我们倾向于更新大量的数据结构。由于 Redis 保证每个脚本都是原子执行的，所以成功执行脚本意味着可以保证系统处于一致的（在 ACID 意义上）状态。

![](https://mmbiz.qpic.cn/mmbiz_jpg/YriaiaJPb26VP5GmOToWsSXf4jwHwAwNRFk22EDXzjEicFcrYNIatoaHLuicnML4P76vWjzdExA546LJyyFQ0wkPNQ/640?wx_fmt=jpeg)

来源：https://netflixtechblog.com/timestone-netflixs-high-throughput-low-latency-priority-queueing-system-with-built-in-support-1abf249ba95f

为了实现可观察性，Timestone 捕获关于传入消息及其状态间转换的信息，并将其保存在 Elasticsearch 的两个二级索引中。当 Timtstone 服务器从 Redis 获得写入响应时，它将其转换为发送到 Kafka 集群的事件。有两个分别对应 Timestone 两个索引的 Flink 作业，消费来自相应 Kafka 主题的事件，并更新 Elasticsearch 中的索引。

Netflix 创建 Timestone 是为了满足其媒体编码平台 Cosmos 的需求。Timestone 还支持 Conductor——Netflix 的通用工作流编排引擎，作为大规模数据管道的调度器。

**原文链接：**

https://www.infoq.com/news/2022/10/netflix-timestone-priority-queue/?

​