> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/RcAEOcFiYVnRp2krGBRujw)

> 背景在我们的开发过程中，经常会在一个项目中使用多种数据库系统。在一些特定场景下，我们希望把数据从一种数据库，

在我们的开发过程中，经常会在一个项目中使用多种数据库系统。在一些特定场景下，我们希望把数据从一种数据库，同步到另一种异构的数据库，以便进行数据分析统计、完成实时监控、实时搜索等功能。这个异构数据源同步的过程称为 Change Data Capture（变化数据捕获）。

![](https://mmbiz.qpic.cn/mmbiz_png/0RZG5gyzadLGEv3TeBia2ftPogneV2oKCsic9a2emJUH2TSyxzOhU6iafefDVzc86pGNvwGtHLfz9fBFxEYSpqblA/640?wx_fmt=png)

我们本文讨论的是 Source 为 MySQL、Target 为 ElasticSearch 的场景下，进行增量和全量同步操作过程。众所周知，MySQL 数据库凭借其性能卓越、服务稳定、开放源代码、社区活跃等因素，成为当下最流行的关系型数据库，但是在数据量级较大或涉及到多表操作时，亦或是需要根据地理位置进行查询时，MySQL 通常不能给我们很好的支持。

为了解决 MySQL 查询缓慢、无法查询的问题，我们通常采用 ElasticSearch 等进行配合检索。在传统方式中，将 MySQL 同步数据到 ElasticSearch 通常采用的是双写、MQ 消息等形式，这些形式都存在着耦合高、性能差、丢失数据等风险。

所以我们需要探索出一套对业务无侵入的 MySQL 同步至 ElasticSearch 异构数据库的解决方案。本文将分别从增量同步、全量同步两个层面进行探讨。

架构
--

将 MySQL 数据实时增量同步至 ElasticSearch 中，一般会借助 MySQL 增量日志 binlog 实现。目前比较流行的 binlog 解析获取中间件是由 Alibaba 开源的 Canal，Canal 译意为水道 / 管道 / 沟渠，主要用途是基于 MySQL 数据库增量日志解析，提供增量数据订阅和消费。

所以我们整体的解决方案为：上游通过 Canal 将解析好的 binlog 消息发送到 Kafka 中，下游通过一个 Adapter 进行解析配置、消费消息。

![](https://mmbiz.qpic.cn/mmbiz_png/0RZG5gyzadLGEv3TeBia2ftPogneV2oKCicYthemiaNzKRXdXzhmumCjNySlqMbj11YOTxrNwJoM6a43UqpW1eD8w/640?wx_fmt=png)

参考上面的图片，我们可以知道整体分成了三个步骤：

1.  Canal 通过伪装成 MySQL Slave，模拟 MySQL Slave 的交互协议，向 MySQL Master 发送 dump 协议，MySQL Master 收到 Canal 发送过来的 dump 请求，开始推送 binlog 给 Canal，然后 Canal 解析 binlog；
    
2.  Canal 将解析序列化好的 binlog 信息发送到 Kafka；
    
3.  Adapter 根据用户配置信息，接收 Kafka 中的信息解析处理，将最终数据更新到 ElasticSearch 的操作。
    

Adapter 设计思路
------------

经过调研，Adapter 决定采用通过 SQL 语句配置，系统根据 SQL 进行解析获得所需表及字段映射关系形式。解析这一过程利用开源数据库连接池 Druid 实现。

比如下面的演示，用户配置了一条 SQL 语句，系统自动解析确定 ElasticSearch 的字段信息，并缓存 MySQL 的表和字段与 ElasticSearch 的字段映射关系。

使用者可以通过定义字段的别名设置在 ElasticSearch 中的对应字段名，同名字段则不需要别名。

![](https://mmbiz.qpic.cn/mmbiz_gif/0RZG5gyzadLGEv3TeBia2ftPogneV2oKCFoIyRJYOnGPPmaR6RLjB5ebD63jDf9Xw44IXxE52CAiawvXJ1nf6Fcg/640?wx_fmt=gif)

Adapter 的整体流程可以用下图表示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RZG5gyzadLGEv3TeBia2ftPogneV2oKCa7ne0v9dZnXibww3vIICOqmchwaXCjY2ichiaqibibWEpCL19DAV9RG3fAQ/640?wx_fmt=png)

分场景处理
-----

我们已经明确了整体的架构思路，接下来我们需要考虑需要在编写 Adapter 的过程中，让他具备哪些解析能力。

### 单表场景

单表同步是最简单的同步场景，当数据库中一张表内容发生变化，将需要的字段提取并写入 ElasticSearch 中。

举个例子，我们现在有一张 student 表，我们希望将其中的 id、name、age、birthday 字段信息信息同步到 ElasticSearch，如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/0RZG5gyzadLGEv3TeBia2ftPogneV2oKCibKTibsfwJqcKiawMjXQ4zvKE2Xh6Pf5nQukVeZC5qWiaW39DDJRErxoMg/640?wx_fmt=png)

那么需要配置的语句是：

```
SELECT
    s.id AS _id,
    s.id,
    s.name,
    s.age,
    s.birthday
FROM
    student s;
```

其中_id 表示的是 ElasticSearch 唯一标识；id 是 ElasticSearch 中实际的字段。

我们需要做的也相对简单，只是将原有数据进行过滤，将需要的数据写入 ElasticSearch 中即可。

### 多表简单场景

多表同步是指两张表利用 Left Join 进行关联的场景，一般是左表的一个字段记录了右表的 id 信息，将两张表 Left Join 后即可获取到所有需要的信息。

举个例子，我们希望记录下学生的班级信息，所以在学生表中添加了 class_id 字段，字段对应 class 表的 id 字段；

如下图两张表，我们希望将 student 表和 class 表关联，在 ElasticSearch 中储存其中的 id、name、class_id、class_name 信息。

![](https://mmbiz.qpic.cn/mmbiz_png/0RZG5gyzadLGEv3TeBia2ftPogneV2oKCbw7yZIdmLTkyJDudcn38JbLX7ibm1K508via0JicxoSKr8m85Bic7eo5iaw/640?wx_fmt=png)

那么我们可以配置成下面的形式：

```
SELECT
    s.id AS _id,
    s.id,
    s.name,
    c.class_id,
    c.class_name
FROM
    student s
    LEFT JOIN class c
    ON s.class_id = c.id;
```

首先，我们约定将具有关联性质的字段称为关联字段，比如上面 SQL 的 student 表的 class_id 字段和 class 表的 id 字段都是关联字段。

针对这种场景，Adapter 会同时监听两张表的更新，分别是：student 和 class，并针对更改的字段确定需要解决的方式：

*   左表非关联字段：直接通过 binlog 中的信息更新到 ElasticSearch 中；
    
*   右表非关联字段：搜索 ElasticSearch 确定影响范围后，将修改数据写入 ElasticSearch 中；
    
*   关联字段更新：通过在 SQL 语句后拼接 where 条件查询 MySQL 后更新。
    

### 多表复杂场景

#### 多行变一行

将多行数据变为一行数据也是多表关联的一种形式，一般会将多条数据使用指定的拼接符号进行连接。

比如：一个同学需要学习的多种课程、一个联系人的多个手机号码等。

举个例子，我们现在还有一张学生家长表：parent，我们期望在 ElasticSearch 中储存学生的全部家长姓名信息，包括 student 的 id、name、age 和 parentNames。parentNames 是此学生的所有家长姓名，并用逗号分隔。如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/0RZG5gyzadLGEv3TeBia2ftPogneV2oKCQ5eiciaKCXRruWz1ibLlbHMPu3BF70dIGZ1punz9zJickMj0s59J70q4Ug/640?wx_fmt=png)

那么我们可以通过配置下面的语句实现这种效果：

```
SELECT
    s.id AS _id,
    s.id,
    s.name,
    s.age,
    p.parentNames
FROM
    student s
    LEFT JOIN (
    SELECT
        student_id,
        group_concat( parent_name ORDER BY id DESC SEPARATOR ',' ) AS parentNames
    FROM
        parent
    GROUP BY
    student_id
    ) p
    ON s.id = p.student_id
```

#### 字段子查询

上面多行变一行的例子还可以通过字段子查询来实现，我们可以配置下面的语句：

```
SELECT
    s.id AS _id,
    s.id,
    s.name,
    s.age,
    (
    SELECT
        group_concat(
            p.parent_name
        ORDER BY
            p.id DESC SEPARATOR ','
        ) AS parentNames
    FROM
        parent p
    WHERE
        p.student_id = s.id
    GROUP BY
        p.student_id
    ) parentNames
FROM
    student s
```

Adapter 会在解析配置的时候缓存子查询表和外层数据表的关联关系，并在感知到不同数据表变化时做出不同动作：

*   parent 表更新：获取关联字段 student_id 信息，在整个 SQL 后拼接外层表的字段限定条件，比如 s.id = ××。查询后更新；
    
*   student 表更新：获取主表的 id 字段，在整个 SQL 后拼接限制条件查询后更新。
    

当然，Adapter 也支持更加复杂的形式，比如在子查询中是 Left Join 或外层查询是 Left Join 的场景。

储存到 ElasticSearch
-----------------

从获取 binlog 到储存至 ElasticSearch，我们需要保证一些特性，解决一些问题。

### 关注特性

#### 顺序性

在单表和多表的部分场景，我们并不会回到 MySQL 中再次查询最新的数据，直接将 binlog 中的数据置入 ElasticSearch 意味着我们必须保证整体的顺序性。

如果我们错误的处理了两条 binlog 的顺序，我们很有可能导致写入的数据只是更新过程的一个中间版本而不是最终版本。

顺序性包括了 binlog 本身有序性和 Adapter 处理的顺序性。

MySQL5.6 之前版本通过 prepare_commit_mutex 锁保证 MySQL 数据库上层二进制日志和 Innodb 存储引擎层的事务提交顺序一致。在 5.6 及之后版本通过引入 BLGC（Binary Log Group Commit）, 将二进制日志的提交过程分成三个阶段，Flush stage、Sync stage、Commit stage，实现二进制日志和实际提交顺序一致。

而 Adapter 的有序解析我们可以通过数据库关联分组分 topic 传递实现。那么什么是具有关联性的数据表呢？举个例子，用户配置了以下三条 SQL 语句：

```
SELECT
    a.id AS _id,
    a.student_name,
    a.student_age,
    a.class_id,
    b.class_name,
    b.teacher_name
FROM
    student a
    LEFT JOIN teacher b
    ON a.teacher_id = b.id;

 SELECT
    b.id AS _id,
    b.teacher_name,
    b.teacher_age,
    b.campus_id,
    c.campus_address,
    c.campus_name
FROM
    teacher b
    LEFT JOIN campus c
    ON b.campus_id = c.id;

    SELECT
    d.id AS _id,
    d.class_name,
    d.class_introduce
FROM
    class d;
```

我们可以看到前两条语句都涉及到了 teacher 表，另外还分别涉及 student 表和 campus 表；第三条语句仅涉及到 class 表。由于前两条语句同步时都需要依赖 teacher 的 binlog，所以我们将 student、teacher、campus 设为一组，将 class 设为一组。同组的 binlog 数据需要保证有序性。

由于 Adapter 通过下游的定义很难影响上游 Kafka 的 Partition 分配，所以我们推荐的做法是每组都采用单 topic 单 Partition 进行传递。当然，Adapter 内部也有分组的机制，如果多组混合传递我们也能够完美的进行分组多线程高效解析处理。

#### 可靠性

我们如果期望保证 ElasticSearch 和 MySQL 数据库中的数据一致，我们就需要完整的处理每一条 binlog，不会出现丢消息的场景。所以，我们需要保证可靠性。

可靠性即保证每一条消息都被消费，不会出现丢消息或因重复消费带来的错误，所以我们需要实现消费幂等性。

考虑到 Adapter 同步程序可能面临各种正常或因异常情况出现的退出，所以我们利用 Kafka 的手动 Offset 机制，在确定一条 Message 数据被成功写入 ElasticSearch 后，才 Commit 该条 Message 的 Offset，这样就保证了不会丢消息。

对于不回表查询直接向 ElasticSearch 置入数据且类型是 Update 的场景中，我们会对 ElasticSearch 中数据初始态进行判断，确认和 binlog 数据中 oldData 信息一致时，我们才会进行更新；对于回表场景已经获得最新数据，所以重复消费也不会造成影响。

### 解决问题

#### 批量提交

为了提高速度，ElasticSearch 将采取批量提交的形式提高整体速度，调用添加方法将会自动根据不同 ElasticSearch 服务集群储存进入不同的 BulkRequest 对象，在该组全部处理完成后进行提交。

#### JSON 数据

我们必须承认，在 MySQL 中储存 JSON 并不是一种优雅的问题解决方案，但是在一些场景下我们依旧会在 MySQL 中储存 JSON 字符串。

在我们的 Adapter 中，我们提供了自动识别 JSON 并以嵌套文档的形式进行插入的能力；当然，在使用这项功能之前我们必须保证同一个名称的对应内容都是相同类型。

#### Geo 数据

在 MySQL 中，经纬度通常是两个字段，但是我们在储存到 ElasticSearch 中期望储存为一个 geo_point 类型；亦或是 MySQL 中将一个多边形数据储存为了一个字符串，我们在储存到 ElasticSearch 中期望储存为一个 geo_shape 类型。

这时候仅靠简单的同步更新并不能解决，尤其是地理位置多边形等 geo_shape 场景。我们最终选择通过标记函数的形式进行处理，配置用户通过自定义的 geo_shape 标记函数示意系统需要对字段进行解析，Adapter 解析 SQL 的时候通过用户自定义的函数获取需要被解析处理的字段。

例如下面这条 SQL：

```
SELECT a.id as _id, geo_shape(a.geo_shape) FROM geography;
```

值得一提的是，在 ElasticSearch 中，经常会因为地理坐标图形构建不合法导致无法存储，所以我们在更新地理类的数据类型之前都会利用 spatial4j 提前进行判断，如果存在问题直接上报监控中心实现自动感知和通知。

#### 内容过滤

我们的 Adapter 更希望能够根据用户的定义，灵活选择转储到 ElasticSearch 的数据。其中不乏很常见的使用场景逻辑删除，但是目前开源的 Adapter 对于 where 条件的支持通常都存在问题。

我们结合已有开源 Adapter 的 issue 分析了一下导致 where 删除逻辑失效的原因：①单表更新不回表查询，所以没有机会拼接 where 查询 ②回表查询到符合条件的更新进 ElasticSearch，但是 ElasticSearch 原有符合但现在不符合数据不能及时删除。

为了解决这两个问题且提高效率，我们选定方案为允许用户配置 MVEL 表达式，在提交前解析表达式对即将提交数据进行校验，符合则提交，如果不符合就根据 _id 将 ElasticSearch 中对应数据进行删除，并停止向 ElasticSearch 更新本条数据信息。

小结
--

目前我们已经支持了以上功能，从线上的监控来看，单条消息平均处理时长在 30ms 左右；P99 在 80ms 左右，基本符合线上实际业务需求。

概述
--

全量同步相比于增量同步简单了许多，全量同步的主要功能是将 MySQL 中的数据优雅且完整的置入到 ElasticSearch 中。

这看起来也十分容易，通过 SQL 查询的形式，将查询结果写到 ElasticSearch 中就可以，在这个过程中我们需要提前考虑哪些问题呢？

关注点
---

### 深分页和丢数据

深分页会造成较大负担比较容易理解，深分页可能还会带来漏处理的问题。比如在 id 较小的部分删除一条数据，深分页可能导致分页接缝出现变化，当前处理页和下一页可能出现丢失数据。

比如下图中的例子，我们深分页每次获取 5 条数据，由于第一次获取之后用户删除了第 4 条信息为 “5” 的数据，导致下一次分页直接从 “9” 开始，造成信息为 “8” 的数据被漏处理。我们的解决办法就是采用根据 id 范围进行更新。

![](https://mmbiz.qpic.cn/mmbiz_png/0RZG5gyzadLGEv3TeBia2ftPogneV2oKCLpwVMW9dvQnXFKEoPvM90VlUVW08ZuXcMJDx7CUic4tGib6ZR5zB9UiaA/640?wx_fmt=png)

### 全量增量互相影响

我们来举一个场景的例子来更好的理解这种影响：

1.  程序读取 id 为 1 ~ 1000 的数据到内存准备全量更新
    
2.  MySQL 修改 id = 100 的数据其中 name 由 “张三” 改为“李四”
    
3.  程序正常进行增量更新，校验 ElasticSearch 中_id 为 100 的数据 name 为 “张三”，将其更新为 “李四”
    
4.  程序全量处理完毕，将 1 ~ 1000 条数据写入 ElasticSearch，其中_id 为 100 的数据 name 被写为 “张三”
    
5.  全量继续进行，直至结束
    

在此过程中，数据库中 id 为 100 的数据 name 是 “李四”，但是由于全量更新和增量更新彼此影响导致 ElasticSearch 的数据被误写为 “张三”。

我们可以通过分段锁的形式细化锁的粒度，实现全量同步同时进行增量同步。

异构数据库的同步一直以来都是一个相对复杂的问题，希望能够对通过本文帮助大家对于异构数据库同步有更多了解，同时为大家提供更多的思路和帮助。

齐同学，便利蜂营建和设备技术团队的 2021 届校园招聘应届生。

最后，便利蜂正在寻找优秀的伙伴，每一份简历我们都会认真对待，期待遇见。

*   邮箱地址：tech-hiring@bianlifeng.com
    
*   邮件标题：营建和设备技术团队
    

https://bianlifeng.gllue.me/portal/homepage

点击 “阅读原文” 进入，了解更多职位详情。