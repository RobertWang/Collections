> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/e0WtvBf-LJkCG4NTxiiStg)

> 比你优秀的人比你还努力，你有什么资格不去奋斗！

前言
==

前几天帮公司解决线上慢 SQL 告警问题，遇到了几个 case。

下面我会结合 case 案例分析自己这段时间在工作上遇到的慢查询谈谈数据库如何优化慢查询。

一般我们遇到的慢 sql 都是索引没有正确使用导致的，所以我先介绍下索引相关知识

索引介绍
====

**索引概念**

排好序的快速查找的数据结构（我们平时说的索引，如果没有特别指明，都是指 B 树，其中聚集索引、次要索引、覆盖索引、复合索引、前缀索引、唯一索引默认使用的都是 B + 树索引，除 B + 树这种类型的索引外还有哈希索引等）

**索引优缺点**

优点：

*   查找 ：提高数据检索效率，降低 IO 成本。
    
*   排序：通过索引对数据进行排序，降低排序成本，降低 cpu 消耗
    

缺点：

*   实际上索引也是一张表，该表保存了主键与索引字段，并指向索引的记录，所以索引列也需要占空间。
    
*   更新表时（insert、update、delete）不仅要保存数据还要更新保存索引文件新添加的索引列。
    

**索引分类**

*   单值索引（单列索引）：一个索引只包含单个列，一个表中可以有多个单列索引。
    
*   唯一索引：索引列必须唯一，但可以允许有空值
    
*   复合索引：一个索引包含多个列
    

**索引结构**

*   BTree 索引
    
*   Hash 索引
    
*   full-text 全文检索
    
*   R-Tree 索引
    

**哪些情况要建索引**

*   主键自动建主键索引
    
*   频繁作为查询条件的字段应该创建索引
    
*   查询中与其他表关联的字段，外键关系建立索引
    
*   在高并发下倾向建立组合索引
    
*   查询中的排序字段，排序字段若通过索引去访问将大大提高排序速度
    
*   查询中统计或者分组的数据
    

**哪些情况不适合建索引**

*   频繁更新的字段
    
*   where 条件用不到的字段不创建索引
    
*   表记录太少
    
*   经常增删改的表
    
*   数据重复太多的字段，为它建索引意义不大（假如一个表有 10 万，有一个字段只有 T 和 F 两种值，每个值的分布概率大约只有 50%，那么对这个字段的建索引一般不会提高查询效率，索引的选择性是指索引列的不同值数据与表中索引记录的比，，如果一个表中有 2000 条记录，表中索引列的不同值记录有 1980 个，这个索引的选择性为`1980/2000=0.99`，如果索引项越接近 1，这个索引效率越高）
    

**explain 字段分析**

explain 是排查慢 sql 的一种最常用的手段

```
mysql> EXPLAIN SELECT 1;


```

id：表示 select 子句或者操作的顺序

*   id 相同：执行顺序自上而下
    
*   id 不同：id 值越大优先级越高，越先被执行
    
*   id 相同不同：id 越大越先执行，相同的自上而下执行
    

select_type：主要是区分普通查询、联合查询、子查询等。

*   SIMPLE：简单的 select 查询，不包含子查询与 union
    
*   PRIMARY：查询中包含复杂的子部分，最外层会被标记为 primary
    
*   SUBQUERY：在 select 或者 where 列表中包含了子查询
    
*   DERIVED：在 from 列表中包含的子查询衍生表
    
*   UNION：若第二个 select 出现在 union 之后，则被标记为 union
    
*   UNION RESESULT：从 union 表获取结果的 select
    

table：这一行数据是哪个表的数据

type：查询中使用了何种类型

**结果值从最好到最坏：system>const>eq_ref>ref>fulltext>ref_or_null>index_merge>unique_subquery>index_subquery>range>index>all**

*   一般来说，得保证查询至少达到 range 级别，最好能到达 ref
    
*   system：表只有一行记录（等于系统表），这是 const 类型的特例，平时不会出现
    
*   const：表示通过索引一次就能够找到
    
*   eq_ref: 唯一性索引扫描，对于每个索引键，表示只有一条记录与之匹配，常见于主键或唯一索引扫描
    
*   ref：非唯一性索引扫描，返回匹配某个单独值的所有行
    
*   range：只检索给定范围的行，使用一个索引来选择行，一般就是在 where 语句中出现了 between、<、>、in 等的查询
    
*   index：index 比 all 快，因为 index 是从索引中读取，all 是从硬盘中读取
    
*   all：遍历全表才能找到
    

possible_key：显示可能应用在这张表中的索引，但实际上不一定用到

key：实际上使用的索引，如果没有则为 null

key_len：表示索引中使用的字节数（可能使用的，不是实际的），可通过该列查询中使用的索引的长度，在不损失精确性的情况下，长度越短越好

ref：显示索引的哪一列被用到，如果可能的话是一个常数，哪些常量被用于查找索引列上的值

rows：大致估算找出所需的记录要读取的行数

Extra：包含不适合在其他列中显示，但十分重要的的额外信息

*   Using filesort 说明 mysql 会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取，mysql 中无法利用索引完成的排序成为文件排序
    
*   Using temporary 使了用临时表保存中间结果，mysql 在对查询结果排序时使用了临时表，常见于排序 order by 和分组查询 group by
    
*   Using index 表示相应的 select 操作中使用了覆盖索引，避免访问了表的数据行，效率高
    
*   Using where 表明使用了 where 进行过滤
    
*   Using join buffer 使用了连接缓存
    
*   impossible where 如果 where 子句的值总是 false，不能用来获取任何元组
    
*   select table optimized away 在没有 group by 子句的情况下，基于索引优化 min/max 操作或者对于 myisam 存储引擎优化`count(*)`操作，不必等到执行阶段再进行计算
    

更详细的内容，请看我之前的文章：

[最完整的 Explain 总结，SQL 优化不再困难](https://mp.weixin.qq.com/s?__biz=MzUyOTg1OTkyMA==&mid=2247484391&idx=1&sn=c478efeed839831a5dc806536029f2b7&scene=21#wechat_redirect)

最完整的 Explain 总结，SQL 优化不再困难

**索引失效**

*   应该尽量全值匹配
    
*   复合最佳左前缀法则（**第一个索引不能掉，中间不能断开**）
    
*   不在索引列上做任何操作（计算、函数、类型转换）会导致索引失效而转向全表扫描
    
*   存储存引擎不能使用索引中范围条件右边的列
    
*   尽量使用覆盖索引（只访问索引的查询（索引列和查询列一致）），减少`select*`
    
*   mysql 在使用不等于 (!= 或者<>) 的时候无法使用索引会导致全表扫描
    
*   is null，is not null 也会无法使用索引
    
*   like 以统配符开头
    
*   字符串不加单引号
    
*   少用 or
    

**order by 优化**

*   避免 filesort，尽量在索引上进行排序，遵照最佳左前缀原则
    

filesort 有两种排序：

*   双路排序：两次磁盘扫描
    

*   单路排序：一次性读取保存在内存中，没拉完的数据再次拉
    
*   单路排序总体好于双路排序
    
*   优化策略：1、增大`sort_buffer_size`参数的设置，2、增大`max_length_for_sort_data`参数的设置，尽可能一次拿到内存
    

Case 分析
=======

案例一
---

**in 中参数太多**

```
select * from goods_info where goods_status = ? and id in(11,22,33......)


```

in 中 id 数据量比较多，导致查询的数据量比较大，这是一个比较常见的慢查询类型，并且往往在业务数据量比较少的时候这条语句不是慢查；

因为参数传进一个 List 集合，当参数比较多的时候，可以采用在业务层把 List 集合拆分为多个长度较小的集合，分多次查询，具体每一次拆长度为多少，可能需要具体根据业务及数据量进行评估

**我的解决办法**：业务代码增加拆分集合操作，`LIMIT_SIZE`设置为 1000

```
List<List<Integer>> partitionGoodsIdList = Lists.partition(goodsIdList, LIMIT_SIZE);


```

当 SQL 的查询参数过多，我觉得可以考虑使用上述拆分的方式

案例二
---

**返回的查询结果过多**

```
select from goods where goods_status = ? and poi_id = ?


```

**解决办法**：将 SQL 修改为分页查询，并在业务代码上修改为分页查询，修改后的 SQL 语句如下：

```
select from goods where goods_status = 1 and poi_id = 11 and goods_id > 22 order by goods_id limit 2000


```

通过分页的方式可以降低数据量，避免慢查询，但是会从而导致一次查询请求，增加为多次查询请求，对于 limit 的大小需要谨慎评估

案例三
---

**order by 慢查询**

```
SELECT * FROM order FORCE INDEX (orderid)  WHERE orderId = 11 AND status IN (0,22) ORDER BY id ASC ;


```

该 SQL 由于强制**指定了使用 orderId 索引，但条件中并没有 orderId**，导致产生全表扫描（type: ALL）；

如下为问题 SQL 的执行计划：

![](https://mmbiz.qpic.cn/mmbiz_png/hC3oNAJqSRzdibjvTzBglAMb05p1pVAbuR0ELdzMwXRIosu3YCk1uykd3t39phxJcgzMJGQ6VFuxHuiaibFs55abQ/640?wx_fmt=png)

直接原因是最终传给 SQL 查询函数的参数，orderId 没有加入 where 子句，但 forceindex 一直生效

案例四
---

**join 慢查询**

```
select * from useract join userinfo order by useracct.id desc limit 11;


```

对 sql 进行 explain 可以发现，因为忘写了 join 的 on 条件，这是扫全表 sql，如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/hC3oNAJqSRzdibjvTzBglAMb05p1pVAbuMW6ubnCozApoCdrGsbfh3VpOeYsBOLD9HJVicbdoChib1vcqkXicdwSOA/640?wx_fmt=png)

我们首先看 type 级别两个表的级别都是 ALL，说明该条语句没有用到索引，做了全表扫描是最差的情况

优化：

![](https://mmbiz.qpic.cn/mmbiz_png/hC3oNAJqSRzdibjvTzBglAMb05p1pVAbuV8Ylia65EULWALLwV6bG0icYc6lo9mEXPZWibGqDZ6BxRzS1JqpfxiaibZg/640?wx_fmt=png)

案例五
---

**不同索引尝试**

```
select id from goods_info where id > ? and activity_id = ? and goods_switch in(?+) limit ?
select id from goods_info where id > 123991510 and activity_id = 0 and goods_switch in (2,3) limit 1000


```

通过执行计划可知，该语句走的是`activity_id`和主键的索引，但是这种命中率比较低，大量的数据被`goods_switch`筛掉

**解决办法**：在不确定最优的索引的情况下，可以在测试环境下，分别添加不同的索引，观察执行计划及语句的执行时间。

尝试强制走主键索引，效果不佳；尝试添加`activity_id_id`的联合索引，效果不佳；尝试添加`activity_id,goods_switch`的联合索引，问题解决！

所以在不确定哪种索引是最优时，可以尝试建立不同的索引，观察语句在不同索引情况下的执行情况进行权衡。

案例六
---

**MySQL 选错索引**

```
select * from goods_info
where goods_source = ? and goods_switch != ? and id > ? order by id limit ?

select * from goods_info  
where goods_source = 2 and goods_switch != 8 and id > 12395070 order by id limit 1000


```

这条语句从语句本身猜测使用的是主键索引，但是查看该语句的执行计划，发现走的索引是`idx_goods_source`，即走了`goods_source`的单列索引！

**解决办法**：修改 SQL 语句，强制走主键索引，查看执行计划，走了主键索引，查询时间大大降低。

正常情况下 MySQL 会选择最优的索引，但是有时候也会选错，MySQL 的优化器会依据扫描行数、是否排序，索引区分度来选择最优的索引，并且扫描行数不一定完成准确，只是 MySQL 的一个预估值

总结
==

**慢查询优化是一个长期的过程，长期有耐心！**