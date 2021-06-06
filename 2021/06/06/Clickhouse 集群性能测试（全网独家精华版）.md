> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/qa-freeroad/p/14394135.html)

背景
==

公司使用 clickhouse 作为其时序分析数据库，在上线前需要对 Clickhouse 集群做一个性能基准测试，用于数据评估。这里我搭建了三节点的集群，集群采用三分片单副本的模式（即数据分别存储在三个 Clickhouse 节点上，每个 Clickhouse 节点都有一个单独的副本，如下图：

![](https://img2020.cnblogs.com/blog/371763/202102/371763-20210209202225695-904203289.png) 

具体的搭建方式参考：[Clickhouse 集群搭建](https://www.cnblogs.com/qa-freeroad/p/14394135.html)

性能测试说明
======

性能关注指标
------

*   clickhouse-server 写性能
*   clickhouse-server 读性能
*   clickhouse-server 的 CPU 和内存占用情况

测试环境说明
------

1）虚拟机列表

<table><colgroup><col><col><col><col><col></colgroup><tbody><tr><td data-cell-id="4358-1577155300040-cell-0-0">机器名</td><td data-cell-id="4358-1577155300040-cell-0-1">IP</td><td data-cell-id="4358-1577155300040-cell-0-2">配置</td><td data-cell-id="4358-1577155300040-cell-0-3">部署的服务</td><td data-cell-id="4358-1577155300040-cell-0-4">备注</td></tr><tr><td data-cell-id="4358-1577155300040-cell-1-0">server01</td><td data-cell-id="4358-1577155300040-cell-1-1">192.168.21.21</td><td data-cell-id="4358-1577155300040-cell-1-2">8c8g</td><td data-cell-id="4358-1577155300040-cell-1-3">clickhouserver（cs01-01）和 clickhouserver（cs01-02）</td><td data-cell-id="4358-1577155300040-cell-1-4">clickhouse01-01: 实例 1, 端口: tcp 9000, http 8123, 同步端口 9009, 类型: 分片 1, 副本 1 clickhouse01-02: 实例 2, 端口: tcp 9001, http 8124, 同步端口 9010, 类型: 分片 2, 副本 2 (clickhouse2 的副本)</td></tr><tr><td data-cell-id="4358-1577155300040-cell-2-0">server02</td><td data-cell-id="4358-1577155300040-cell-2-1">192.168.21.69</td><td data-cell-id="4358-1577155300040-cell-2-2">8c8g</td><td data-cell-id="4358-1577155300040-cell-2-3">clickhouserver（cs02-01）和 clickhouserver（cs02-02）</td><td data-cell-id="4358-1577155300040-cell-2-4">clickhouse02-01: 实例 1, 端口: tcp 9000, http 8123, 同步端口 9009, 类型: 分片 2, 副本 1 clickhouse02-02: 实例 2, 端口: tcp 9001, http 8124, 同步端口 9010, 类型: 分片 3, 副本 2 (clickhouse3 的副本)</td></tr><tr><td data-cell-id="4358-1577155300040-cell-3-0">server03</td><td data-cell-id="4358-1577155300040-cell-3-1">192.168.21.6</td><td data-cell-id="4358-1577155300040-cell-3-2">8c8g</td><td data-cell-id="4358-1577155300040-cell-3-3">clickhouserver（cs03-01）和 clickhouserver（cs03-02）</td><td data-cell-id="4358-1577155300040-cell-3-4">clickhouse03-01: 实例 1, 端口: tcp 9000, http 8123, 同步端口 9009, 类型: 分片 3, 副本 1 clickhouse03-02: 实例 2, 端口: tcp 9001, http 8124, 同步端口 9010, 类型: 分片 1, 副本 2 (clickhouse1 的副本)</td></tr><tr><td data-cell-id="4358-1577155300040-cell-4-0">发压机器</td><td data-cell-id="4358-1577155300040-cell-4-1">192.168.21.3</td><td data-cell-id="4358-1577155300040-cell-4-2">16c2g</td><td data-cell-id="4358-1577155300040-cell-4-3">压测机器</td><td data-cell-id="4358-1577155300040-cell-4-4">用于测试 clickhouse-server 的性能</td></tr></tbody></table>

2）测试数据表说明

server02 上的 cs02-01 中数据表使用如下 sql 创建写测试表：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
create database test_ck;

#创建本地复制表用于写入（使用ReplicatedMergeTree复制表）
CREATE TABLE test_ck.device_thing_data (
                time                     UInt64,
                user_id                 String,
                device_id                 String,
                source_id                 String,
                thing_id                   String,
                identifier                String,
                value_int32                Int32,
                value_float                Float32,
                value_double            Float64,
                value_string            String,
                value_enum              Enum8('0'=0,'1'=1,'2'=2,'3'=3,'4'=4,'5'=5,'6'=6,'7'=7,'8'=8),
                value_string_ex         String,
                value_array_string         Array(String),
                value_array_int32         Array(Int32),
                value_array_float         Array(Float32),
                value_array_double         Array(Float64),
                action_date                Date,
                action_time             DateTime
            ) Engine= ReplicatedMergeTree('/clickhouse/tables/01-02/device_thing_data','cluster01-02-1') PARTITION BY toYYYYMM(action_date) ORDER BY (user_id,device_id,thing_id,identifier,time,intHash64(time)) SAMPLE BY intHash64(time) SETTINGS index_granularity=8192
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

创建分布式表用于查询，在三台每个机器上均执行如下 sql：

```
CREATE TABLE device_thing_data_all AS test_ck.device_thing_data ENGINE = Distributed(cluster_3s_1r, test_ck, device_thing_data, rand())
```

测试数据说明
------

1）测试原始数据从开发联调环境的 clickhouse 导出，保存到本地的 csv 文件

2）写数据测试往 192.168.21.69 的 9000 端口（cs02-01）的 test_ck.device_thing_data 写入，使用的 sql 类似如下：

```
self.client.execute('INSERT INTO test_ck.device_thing_data (time,user_id,device_id,source_id,thing_id,identifier,value_int32,value_float,value_double,value_string,value_enum,value_string_ex,value_array_string,value_array_int32,value_array_float,value_array_double,action_date,action_time) VALUES', data,types_check=True)
```

3）读数据测试和写数据 clickhouse-server 实例一致，表使用 device_thing_data_all，使用的 sql 类似如下：

```
self.client.execute('select count(1) from (select time,user_id,device_id,source_id,thing_id,identifier,value_int32,value_float,value_double,value_string,value_enum,value_string_ex,value_array_string,value_array_int32,value_array_float,value_array_double,action_date,action_time from device_thing_data_all limit %d) t1' % self.bulksize)
```

测试工具
----

测试工具使用 python 和 shell 编写，python 使用 clickhouse 的客户端，shell 使用 parallel 实现多进程

测试场景与性能数据
---------

 1）写入测试，对集群的 (cs02-01) 的复制表的写入测试

<table><colgroup><col><col><col><col><col><col><col><col></colgroup><tbody><tr><td data-cell-id="2244-1577155151891-cell-0-0">每次批量数据条数</td><td data-cell-id="2244-1577155151891-cell-0-1">客户端连接数</td><td data-cell-id="2244-1577155151891-cell-0-2">耗时（秒）</td><td data-cell-id="2244-1577155151891-cell-0-3">插入总行数</td><td data-cell-id="2244-1577155151891-cell-0-4">TPS（records/sec)</td><td data-cell-id="2244-1577155151891-cell-0-5">clickhouse 的 CPU 占用</td><td data-cell-id="2244-1577155151891-cell-0-6">clickhouse 内存占用 (m)</td><td data-cell-id="2244-1577155151891-cell-0-7">备注</td></tr><tr><td data-cell-id="2244-1577155151891-cell-1-0">10</td><td data-cell-id="2244-1577155151891-cell-1-1">1</td><td data-cell-id="2244-1577155151891-cell-1-2">12.319155</td><td data-cell-id="2244-1577155151891-cell-1-3">10000</td><td data-cell-id="2244-1577155151891-cell-1-4">811.744020</td><td data-cell-id="2244-1577155151891-cell-1-5">43%</td><td data-cell-id="2244-1577155151891-cell-1-6">1.8%（约 160M)</td><td data-cell-id="2244-1577155151891-cell-1-7">/bin/bash start_clickhouse_perf.sh --mode=1 --clientnum=1 --bulksize=10 --times=1000</td></tr><tr><td data-cell-id="2244-1577155151891-cell-2-0">100</td><td data-cell-id="2244-1577155151891-cell-2-1">3</td><td data-cell-id="2244-1577155151891-cell-2-2">25.015171</td><td data-cell-id="2244-1577155151891-cell-2-3">300000</td><td data-cell-id="2244-1577155151891-cell-2-4">12026.095374</td><td data-cell-id="2244-1577155151891-cell-2-5">72%</td><td data-cell-id="2244-1577155151891-cell-2-6">1.8%（约 160M)</td><td data-cell-id="2244-1577155151891-cell-2-7">/bin/bash start_clickhouse_perf.sh --mode=1 --clientnum=3 --bulksize=100 --times=1000</td></tr><tr><td data-cell-id="2244-1577155151891-cell-3-0">1000</td><td data-cell-id="2244-1577155151891-cell-3-1">3</td><td data-cell-id="2244-1577155151891-cell-3-2">61.579590</td><td data-cell-id="2244-1577155151891-cell-3-3">1500000</td><td data-cell-id="2244-1577155151891-cell-3-4">24496.428544</td><td data-cell-id="2244-1577155151891-cell-3-5">18.3%</td><td data-cell-id="2244-1577155151891-cell-3-6">1.9%（约 160M)</td><td data-cell-id="2244-1577155151891-cell-3-7">/bin/bash start_clickhouse_perf.sh --mode=1 --clientnum=3 --bulksize=1000 --times=500</td></tr><tr><td data-cell-id="2244-1577155151891-cell-4-0">1000</td><td data-cell-id="2244-1577155151891-cell-4-1">6</td><td data-cell-id="2244-1577155151891-cell-4-2">64.323068</td><td data-cell-id="2244-1577155151891-cell-4-3">3000000</td><td data-cell-id="2244-1577155151891-cell-4-4">47051.112386</td><td data-cell-id="2244-1577155151891-cell-4-5">35.2%</td><td data-cell-id="2244-1577155151891-cell-4-6">1.9%（约 160M)</td><td data-cell-id="2244-1577155151891-cell-4-7">/bin/bash start_clickhouse_perf.sh --mode=1 --clientnum=6 --bulksize=1000 --times=500</td></tr><tr><td data-cell-id="2244-1577155151891-cell-5-0">10000</td><td data-cell-id="2244-1577155151891-cell-5-1">6</td><td data-cell-id="2244-1577155151891-cell-5-2">222.632641</td><td data-cell-id="2244-1577155151891-cell-5-3">12000000</td><td data-cell-id="2244-1577155151891-cell-5-4">54542.892502</td><td data-cell-id="2244-1577155151891-cell-5-5">9.3%</td><td data-cell-id="2244-1577155151891-cell-5-6">2.4%（约 160M)</td><td data-cell-id="2244-1577155151891-cell-5-7">/bin/bash start_clickhouse_perf.sh --mode=1 --clientnum=6 --bulksize=1000 --times=500</td></tr></tbody></table>

2）读取测试，对集群的 (cs02-01) 的分布式表的读取测试

<table><colgroup><col><col><col><col><col><col><col><col></colgroup><tbody><tr><td data-cell-id="7615-1577155151891-cell-0-0">每次批量数据条数</td><td data-cell-id="7615-1577155151891-cell-0-1">客户端连接数</td><td data-cell-id="7615-1577155151891-cell-0-2">耗时（秒）</td><td data-cell-id="7615-1577155151891-cell-0-3">插入总行数</td><td data-cell-id="7615-1577155151891-cell-0-4">TPS（records/sec)</td><td data-cell-id="7615-1577155151891-cell-0-5">clickhouse 的 CPU 占用</td><td data-cell-id="7615-1577155151891-cell-0-6">clickhouse 内存占用 (m)</td><td data-cell-id="7615-1577155151891-cell-0-7" class="">备注</td></tr><tr><td data-cell-id="7615-1577155151891-cell-1-0">1000</td><td data-cell-id="7615-1577155151891-cell-1-1">1</td><td data-cell-id="7615-1577155151891-cell-1-2">11.610356</td><td data-cell-id="7615-1577155151891-cell-1-3">1000000</td><td data-cell-id="7615-1577155151891-cell-1-4">86130.004332</td><td data-cell-id="7615-1577155151891-cell-1-5">69.4%</td><td data-cell-id="7615-1577155151891-cell-1-6">2.1%（约 160M)</td><td data-cell-id="7615-1577155151891-cell-1-7" class="">/bin/bash start_clickhouse_perf.sh --mode=0 --clientnum=1 --bulksize=1000 --times=1000</td></tr><tr><td data-cell-id="7615-1577155151891-cell-2-0">1000</td><td data-cell-id="7615-1577155151891-cell-2-1">3</td><td data-cell-id="7615-1577155151891-cell-2-2">12.897658</td><td data-cell-id="7615-1577155151891-cell-2-3">3000000</td><td data-cell-id="7615-1577155151891-cell-2-4">233129.085885</td><td data-cell-id="7615-1577155151891-cell-2-5">200.1%</td><td data-cell-id="7615-1577155151891-cell-2-6">2.1%（约 160M)</td><td data-cell-id="7615-1577155151891-cell-2-7">/bin/bash start_clickhouse_perf.sh --mode=0 --clientnum=3 --bulksize=1000 --times=1000</td></tr><tr><td data-cell-id="7615-1577155151891-cell-3-0">10000</td><td data-cell-id="7615-1577155151891-cell-3-1">3</td><td data-cell-id="7615-1577155151891-cell-3-2">12.971161</td><td data-cell-id="7615-1577155151891-cell-3-3">30000000</td><td data-cell-id="7615-1577155151891-cell-3-4">2322824.513353</td><td data-cell-id="7615-1577155151891-cell-3-5">207%</td><td data-cell-id="7615-1577155151891-cell-3-6">2.1%（约 160M)</td><td data-cell-id="7615-1577155151891-cell-3-7" class="">/bin/bash start_clickhouse_perf.sh --mode=0 --clientnum=3 --bulksize=10000 --times=1000</td></tr><tr><td data-cell-id="7615-1577155151891-cell-4-0">10000</td><td data-cell-id="7615-1577155151891-cell-4-1">6</td><td data-cell-id="7615-1577155151891-cell-4-2">16.298867</td><td data-cell-id="7615-1577155151891-cell-4-3">60000000</td><td data-cell-id="7615-1577155151891-cell-4-4">3705072.680627</td><td data-cell-id="7615-1577155151891-cell-4-5">353.5%</td><td data-cell-id="7615-1577155151891-cell-4-6">2.1%（约 160M)</td><td data-cell-id="7615-1577155151891-cell-4-7">/bin/bash start_clickhouse_perf.sh --mode=0 --clientnum=6 --bulksize=10000 --times=1000</td></tr><tr><td data-cell-id="7615-1577155151891-cell-5-0">100000</td><td data-cell-id="7615-1577155151891-cell-5-1">6</td><td data-cell-id="7615-1577155151891-cell-5-2">19.740923</td><td data-cell-id="7615-1577155151891-cell-5-3">600000000</td><td data-cell-id="7615-1577155151891-cell-5-4">30605253.774755</td><td data-cell-id="7615-1577155151891-cell-5-5">461%</td><td data-cell-id="7615-1577155151891-cell-5-6">2.2%（约 160M)</td><td data-cell-id="7615-1577155151891-cell-5-7">/bin/bash start_clickhouse_perf.sh --mode=0 --clientnum=6 --bulksize=100000 --times=1000</td></tr></tbody></table>

3）写入数据量测试

写入 1 亿条记录到 clickhouse 单实例中，最后硬盘上的数据大小约为 450M 左右。

最后
--

可以看出，Clickhouse 的单批次读写的记录越多，性能越好；尽量使用多线程进行读写，这样能够最大化利用 Clickhouse 的性能。

> 博主：测试生财（一个不为 996 而 996 的测开码农）
> 
> 座右铭：专注测试开发与自动化运维，努力读书思考写作，为内卷的人生奠定财务自由。
> 
> 内容范畴：技术提升，职场杂谈，事业发展，阅读写作，投资理财，健康人生。
> 
> csdn：[https://blog.csdn.net/ccgshigao](https://blog.csdn.net/ccgshigao)
> 
> 博客园：[https://www.cnblogs.com/qa-freeroad/](https://www.cnblogs.com/qa-freeroad/)
> 
> 51cto：[https://blog.51cto.com/14900374](https://blog.51cto.com/14900374)
> 
> 微信公众号：**测试生财**（定期分享独家内容和资源）