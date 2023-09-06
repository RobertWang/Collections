> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ywwBXNR4EMExA7jHME6oSQ)

在我们的生产环境中有一张表：courier_consume_fail_message，是存放消息消费失败的数据的，设计之初，这张表的数据量评估在万级别以下，因此没有建立索引。

但目前发现，该表的数据量已经达到百万级别，原因产生了大量的重试消费，这导致了该表的慢查询。

因此需要清理该表数据。而实际上，使用 DELETE 命令删除数据后，我们发现查询速度并没有显著提高，甚至可能会降低。为什么？

因为 DELETE 命令只是标记该行数据为 “已删除” 状态，并不会立即释放该行数据在磁盘中所占用的存储空间，这样就会导致数据文件中存在大量的碎片，从而影响查询性能。所以，除了删除表记录外，还需要清理磁盘碎片。

在表碎片清理前，我们关注以下四个指标。

*   指标一：表的状态：`SHOW TABLE STATUS LIKE 'courier_consume_fail_message';`
    
*   指标二：表的实际行数：`SELECT count(*) FROM courier_consume_fail_message;`
    
*   指标三：要清理的行数：`SELECT count(*) FROM courier_consume_fail_message where created_at < '2023-04-19 00:00:00';`
    
*   指标四：表查询的执行计划：`EXPLAIN SELECT * FROM courier_consume_fail_message WHERE service='courier-transfer-mq';`
    

```
-- 清理磁盘碎片
OPTIMIZE TABLE courier_consume_fail_message;


```

以下是清理前后的指标对比。

一、清理前
-----

指标一，表的状态：

![](https://mmbiz.qpic.cn/mmbiz_png/3BoNTODkwCamkLFak49vibzuMc10ytzPqRb07EAb7raxsYBwrkxxAVZ3icXtwpmgj1I5FaXvFZNEL6T58vHN3zrw/640?wx_fmt=png)

指标二，表的实际行数：76986

指标三，要清理的行数：76813

指标四，表查询的执行计划：

![](https://mmbiz.qpic.cn/mmbiz_png/3BoNTODkwCamkLFak49vibzuMc10ytzPq8I2sOjjOvU48LYURVxtXn6LcGG4iblXyiahhe2Qa4kohUTgzYkODxiaTw/640?wx_fmt=png)

二、清理数据
------

下面是执行 `DELETE FROM courier_consume_fail_message WHERE created_at < '2023-04-19 00:00:00';` 后的统计。

指标一，表的状态：

![](https://mmbiz.qpic.cn/mmbiz_png/3BoNTODkwCamkLFak49vibzuMc10ytzPqLyk31NAkia7qkPBTYHc6IYCDvB7hxP0I0EAEC1F0fQcRefWP7dYTKvQ/640?wx_fmt=png)

指标二，表的实际行数：173

指标三，要清理的行数：0

指标四，表查询的执行计划：

![](https://mmbiz.qpic.cn/mmbiz_png/3BoNTODkwCamkLFak49vibzuMc10ytzPqonNJ8iaibVOTlM7ZwlTMMR1SO7s2EIy0n3CjicdXhNj1DGwscTGYpFpJQ/640?wx_fmt=png)

通过指标四可以看到，清理表记录后，查询扫描的行数依然没变：8651048。

三、清理碎片
------

下面是执行 `OPTIMIZE TABLE courier_consume_fail_message;` 后的统计。

指标一，表的状态：

![](https://mmbiz.qpic.cn/mmbiz_png/3BoNTODkwCamkLFak49vibzuMc10ytzPql1OamWs66LibklL4dI3WGWenkrUHI3RtDzwxEP8vUpuVZeInJRXYGcw/640?wx_fmt=png)

指标四，表查询的执行计划：

![](https://mmbiz.qpic.cn/mmbiz_png/3BoNTODkwCamkLFak49vibzuMc10ytzPqZFtzYtdHbU7hb2PChmVH7SOXanZyMbPbzgIHtPGOO8CDC2QrY6yNxg/640?wx_fmt=png)

通过指标四可以看到，清理表记录后，查询扫描的行数变成了 100。

小结
--

可以看到，该表的数据行数和数据长度都被清理了，查询语句扫描的行数也减少了。

为了提升 `SELECT * FROM courier_consume_fail_message WHERE service='courier-transfer-mq';` 语句的查询效率，还是应当建立索引。

```
alter` `table` `ec_courier.courier_consume_fail_message ``add` `index` `idx_service(service);


```