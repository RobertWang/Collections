> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MDc5NzQ2MQ==&mid=2247501741&idx=1&sn=ebf82e894614e3c7f3a51f5182b8222b&chksm=e917d5d7de605cc1b2333455edace221610a22567a15ac84b6206d6bdd39a52bbc3856b57a3f&scene=21#wechat_redirect)

OLAP

作者：樊思国

京东科技 IoT 团队原创，转载请获得授权

![](https://mmbiz.qpic.cn/mmbiz_png/Lic3WIjno4gwNvRibyWtbEOdv79jdqD9ib5gjssNMkpQpbPVK5FuWLR54V3pJza3gAwnd8iasmQyPqY9vQZ3sj7pmQ/640?wx_fmt=png)

**二、技术选型**  

### **三、ClickHouse 的应用**

![](https://mmbiz.qpic.cn/mmbiz_png/Lic3WIjno4gwNvRibyWtbEOdv79jdqD9ib5cgZiaZMzWHeCxsfjIXc3HBTG5kIN47BiaaJGJ3PQrpBJPfv0nbBgqmWw/640?wx_fmt=png)

**说明：**

*   **物管平台：**对设备的管理，管理物模型及设备状态、采集设备数据。
    
*   **消息总线：**kafka 消息队列，利用 JSON 格式数据实现物管平台和能平台的数据交互。
    
*   **差分器：**对每次上报的累计值同上一次上报的累计值做差值计算，得到可累加指标。
    
*   **异常规则链：**提供一个异常规则集，用于差分器判定上报数据是否异常，如异常则进行记录，数据不作处理。
    
*   **OLAP 引擎：**基于 ClickHouse 实现的 OLAP 引擎。
    
*   **多维分析服务：**提供通用的数据多维分析查询服务，能够通过统一的 API 实现各种维度和指标的组合查询。
    
*   **政府和企业界面：**政企客户的 WEB 界面。
    

**2、ClickHouse 应用**

通过上面的架构图可以看出，能源平台采用 ClickHouse 作为 OLAP 引擎提供多维查询服务。下面重点从数据的接入、存储以及通用化接口设计方面谈一谈

ClickHouse 的应用：

*   **数据接入**
    

ClickHouse 基于 kafka 引擎表的数据接入可以看做是一个典型的 ETL 过程，数据的抽取（Extract）是通过建立一张 kafka 引擎表，产生消费端订阅 kafka topic 实现；数据的转换（Transform）通过物化视图实现；数据最终加载（Load）进 MergeTree 表，实现实际数据存储。

![](https://mmbiz.qpic.cn/mmbiz_png/Lic3WIjno4gwNvRibyWtbEOdv79jdqD9ib5FojrXibWc7sUygwsEoc9DhsaT9lAIUNgl2BMTCjWbasbSUmrVyHQIxg/640?wx_fmt=png)

创建 Kafka 表示例：

<左右滑动以查看完整代码>

*   kafka_broker_list: kafka broker 地址。
    
*   kafka_topic_list：消费的 topic。
    
*   kafka_group_name：消费 groupId。
    
*   kafka_format：数据格式 JSONEachRow 表示消息体为 JSON 格式。
    
*   kafka_skip_broken_messages：表示忽略的 kafka 异常消息条数，默认为 0。
    
*   kafka_num_consumers：消费者个数，默认值为 1，建议同 kafka 分区数对应。
    

创建物化视图示例：

```
1CREATE MATERIALIZED VIEW statistics_view ON CLUSTER '{cluster}' TO statistics_replica AS
2SELECT timestamp,
3       level,
4       message
5FROM statistics_kafka;
```

<左右滑动以查看完整代码>

创建 MergeTree 引擎表示例：

```
1CREATE TABLE statistics_replica ON CLUSTER '{cluster}'{
2    timestamp UInt64,
3    dt String,
4    deviceId String,
5    level String,
6     message String
7} ENGINE = ReplicatedMergeTree('/clickhouse/tables/{shard}/statistics_replica','{replica}')
8PARTITION BY dt
9ORDER BY (dt,deviceId,level);
```

<左右滑动以查看完整代码>

*   **存储**
    

1.  **ClickHouse 表类型**
    
    本地表：实际数据存储的表，如上示例表 statistics_replica。
    
    分布式表：一个逻辑上的表, 可以理解为数据库中的视图, 一般查询都查询分布式表. 分布式表引擎会将我们的查询请求路由本地表进行查询, 然后进行汇总最终返回给用户。创建分布式表示例：
    
    ```
    1CREATE TABLE statistics ON CLUSTER '{cluster}' AS statistics_replica
    2ENGINE = Distributed(ck_cluster_1,test,events_local,rand());
    ```
    
    <左右滑动以查看完整代码>
    
2.  **Replication 和 Sharding**
    
    Replication 是 ClickHouse 提供的副本机制，对于 Replicated MergeTree 系列复制表，可以设置每个表有多份完全一样的数据存放在不同的计算节点上，每一份数据都是完整的，并且称为一个副本。
    
    Shard：将表中的数据按照一定的规则拆分为多个部分，每个部分的数据均存储在不同的计算节点上，每个计算节点上的数据称为一个分片。
    

![](https://mmbiz.qpic.cn/mmbiz_png/Lic3WIjno4gwNvRibyWtbEOdv79jdqD9ib5Z6hAW6ic7s3icYegxKb0XQdeB3BwaPJU30MUDxK34z5BhicmrCh7IKE2Q/640?wx_fmt=png)

ClickHouse 基于 Replicated MergeTree 引擎与 Zookeeper 实现了复制表机制，在创建表时，可以决定表是否高可用。上一节的 statistics_replica 表，其中 / clickhouse/tables/{shard}/statistics_replica 表示 Zookeeper 中对应副本表的 node。当数据写入 ReplicatedMergeTree 表时，过程如下：

![](https://mmbiz.qpic.cn/mmbiz_png/Lic3WIjno4gwNvRibyWtbEOdv79jdqD9ib5DY7zcRmyWzbUPVyEXXr8NIvzXZgicJv0OtKEG4M5FmjbTgYt9rOyBZQ/640?wx_fmt=png)

1.  某一个 ClickHouse 节点接收到数据写入请求。
    
2.  通过 interserver HTTP port 端口同步到其他实例。
    
3.  更新 Zookeeper 集群上的 node 信息。
    

**3、OLAP 通用接口设计**

ClickHouse 提供标准的 SQL 查询引擎，通过 JDBC 引用程序可以实现多 ClickHouse 的基本操作。OLAP 的常规操作如上卷、下钻和切片会涉及到多种维度自由组合、多种指标交叉剖析的过程，如果服务端采用 Mybatis 或 JPA 等常规 ORM 操作，工程师很容易根据不同的查询场景要求设计出对应的接口，亦或是根据大量的分支操作设计出复杂的判定性接口，鉴于此，作者从 mdx 思想获得启示，设计一套对 OLAP 优化的通用多维服务查询接口。

首先，一个典型的分析类 SQL 语句如下：

```
1SELECT day_str,
 2       factory_name,
 3       workshop_name,
 4       prodline_name,
 5       device_id,
 6       SUM(w_total) AS total
 7FROM statistics
 8WHERE day_str BETWEEN '2020-10-01' AND '2020-12-31'
 9GROUP BY day_str,factory_name,workshop_name,prodline_name,device_id
10ORDER BY day_str ASC;
```

如上语句，我们翻译成业务语言为『分别查询 2020 年 4 季度全厂所有设备的耗电量』，从这里我们可以清楚的知道这里的维度是指『设备名称』，指标为『耗电量』，基于此，可以进一步归类，维度通常出现在 SQL 语句的 SELECT、WHERE、GROUP BY 和 ORDER BY 后面，指标则通常出现在 SELECT 后面，也就是可以总结如下模式：

```
1SELECT {维度},{指标}
2FROM table_name
3WHERE {维度}='xxx'
4GROUP BY {维度}
5ORDER BY {维度};
```

因此，我们可以设计如下通用接口方法：

<左右滑动以查看完整代码>

**四、总结**

本文重点介绍了京东综合能源管理平台多维数据分析引擎的架构和设计，从数据接入、存储和多维分析服务设计的角度，阐述了 ClickHouse 的一种典型应用场景。希望通过本文让读者在应对大数据实时 OLAP 领域，提供一种思路和方法。当然，限于篇幅和本人水平有限，没有进一步展开阐述更多的可能性方案，随着我们对于业务的深入，系统的迭代升级，适宜于将来更优方案势必会步步推出，也请期待。