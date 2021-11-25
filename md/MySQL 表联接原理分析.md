> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/6toEF5NHWoBVxHJrDrq-iA)

2. 业务中到底要不要用表联接？

3. 使用表联接会有什么样的问题，怎样避免和优化？

**02**

**准备内容**

这里我们需要用到 3 个表，这 3 个表都有一个主键索引 id 和一个索引 a，字段 b 上无索引。存储过程 idata() 往表 t1 里插入的是 100 行数据，表 t2、t3 里插入了 1000 行数据。建表语句如下：

```
CREATE TABLE `t1` (
    `id` INT ( 11 ) NOT NULL,
    `t1_a` INT ( 11 ) DEFAULT NULL,
    `t1_b` INT ( 11 ) DEFAULT NULL,
PRIMARY KEY ( `id` ),
KEY `idx_a` ( `t1_a` )) ENGINE = INNODB;
CREATE TABLE `t2` (
    `id` INT ( 11 ) NOT NULL,
    `t2_a` INT ( 11 ) DEFAULT NULL,
    `t2_b` INT ( 11 ) DEFAULT NULL,
PRIMARY KEY ( `id` ),
KEY `idx_a` ( `t2_a` )) ENGINE = INNODB;
CREATE TABLE `t3` (
    `id` INT ( 11 ) NOT NULL,
    `t3_a` INT ( 11 ) DEFAULT NULL,
    `t3_b` INT ( 11 ) DEFAULT NULL,
PRIMARY KEY ( `id` ),
KEY `idx_a` ( `t3_a` )) ENGINE = INNODB;
-- 向t1添加100条数据
-- drop procedure idata;
delimiter ;;
create procedure idata()
begin
  declare i int;
  set i=1;
  while(i<=100)do
        insert into t1 values(i, i, i);
    set i=i+1;
  end while;
end;;
delimiter ;
call idata();
-- 向t2添加1000条数据
drop procedure idata;
delimiter ;;
create procedure idata()
begin
  declare i int;
  set i=101;
  while(i<=1100)do
        insert into t2 values(i, i, i);
    set i=i+1;
  end while;
end;;
delimiter ;
call idata();
-- 向t2添加1000条数据，且t3_a列的值为倒叙
drop procedure idata;
delimiter ;;
create procedure idata()
begin
  declare i int;
  set i=101;
  while(i<=1100)do
        insert into t3 values(i, 1101-i, i);
    set i=i+1;
  end while;
end;;
delimiter ;
call idata();
```

**03**

**联接的过程和实现**

首先我们用准备好的`t1`和`t2`两个表做常规联接数据展示，因为我们表中记录有点多，如果全部展示则有 `100*1000=100000` 条数据，太多了，所以我们只取每个表的前 3 条做为示例：

```
mysql root@localhost:test> select * from t1,t2 where t1.id<=3 and t2.id<=103;
+----+------+------+-----+------+------+
| id | t1_a | t1_b | id  | t2_a | t2_b |
+----+------+------+-----+------+------+
| 1  | 1    | 1    | 101 | 101  | 101  |
| 2  | 2    | 2    | 101 | 101  | 101  |
| 3  | 3    | 3    | 101 | 101  | 101  |
| 1  | 1    | 1    | 102 | 102  | 102  |
| 2  | 2    | 2    | 102 | 102  | 102  |
| 3  | 3    | 3    | 102 | 102  | 102  |
| 1  | 1    | 1    | 103 | 103  | 103  |
| 2  | 2    | 2    | 103 | 103  | 103  |
| 3  | 3    | 3    | 103 | 103  | 103  |
+----+------+------+-----+------+------+
9 rows in set
Time: 0.012s
```

**01.** **表联接的实现过程**

由上面的结果可以看出，联接查询的过程就是把`t1`表和`t2`表中的每一条记录相互联接组成更大的集合，而这也是表联接的实现过程：

1. 先取出两个表其中的一个表的一条符合条件的数据; 2. 用上一步取到的数据，跟另一个表的数据去匹配，符合条件的才放入最终的结果集; 3. 如此以上两个步骤，直到第一个表符合条件的数据全部取完。

这个先取数据的表就是`驱动表`，另一个就是`被驱动表`。

**02. 嵌套循环联接**

**（Nested-Loop Join）**

这个过程在形式上，就像我们程序中的嵌套循环类型，用伪代码表示就是：

```
foreach rows in t1 {   #此处表示遍历满足对t1单表查询结果集中的每一条记录
    foreach rows in t2 {   #此处表示对于某条t1表的记录来说，遍历满足对t2单表查询结果集中的每一条记录
      ... #此处表示可能有更多表需要联接
      #如果满足联接条件，发送到客户端
      if row satisfies join conditions, send to client
    }
}
```

所以这种联接执行方式称之为`嵌套循环联接`（`（Simple）Nested-Loop Join`）算法，简称 NLJ， 这是最简单也是最笨拙的一种联接查询算法。

但是有个问题，`嵌套循环联接`的`步骤2`中可能需要访问多次被驱动表，而且如果访问方式都是全表扫描的话查询效率就会非常低， 但是查询`t2`表其实就相当于一次单表扫描，我们可以利用索引来加快查询速度，这种算法又称为`索引嵌套循环联接`（`Index Nested-Loop Join`）。

**03. 块嵌套循环联接**

**（****Block Nested-Loop Join）**

扫描一个表的过程其实是先把这个表从磁盘上加载到内存（`Buffer Pool`）中，然后从内存中比较匹配条件是否满足。现实生活中的表可不像`t1`、`t2`这种只有几条记录，而是几百万、几千万甚至几亿条记录。那么内存里可能就存放不下表中所有的记录，每次访问被驱动表，被驱动表的记录会被加载到内存中，在内存中的每一条记录只会和驱动表结果集的一条记录做匹配，之后就会被从内存中清除掉。然后再从驱动表结果集中拿出另一条记录，再一次把被驱动表的记录加载到内存中一遍，周而复始，驱动表结果集中有多少条记录，就得把被驱动表从磁盘上加载到内存中多少次。所以在两表联接过程中，被驱动表可是要被访问好多次的，那就相当于要从磁盘上读好几次这个表，这个`I/O`代价就非常大了，所以我们得想办法：尽量减少访问被驱动表的次数。

于是提出了一个`join buffer`的概念，就是执行联接查询前申请的一块固定大小的内存，先把若干条驱动表结果集中的记录装在这个`join buffer`中，然后开始扫描被驱动表，每一条被驱动表的记录一次性和`join buffer`中的多条驱动表记录做匹配，因为匹配的过程都是在内存中完成的，所以这样可以显著减少被驱动表的`I/O`代价。使用`join buffer`的过程如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0DgLwlbnWUDHq0aM9hrDJaZlNmIzFYx1uIq0DZ1FO1CE7yb0JTAP5TkpqtS1t3a6RRumHKfBiaDTiazt0082PWyw/640?wx_fmt=png)

最好的情况是`join buffer`足够大，能容纳驱动表结果集中的所有记录，这样只需要访问一次被驱动表就可以完成联接操作了。设计`MySQL`的大叔把这种加入了`join buffer`的嵌套循环联接算法称之为`块嵌套循环联接`（Block Nested-Loop Join）算法。

注意：

1. 驱动表的记录并不是所有列都会被放到`join buffer`中，只有查询列表中的列和过滤条件中的列才会被放到`join buffer`中，所以我们最好不要把`*`作为查询列表，`最好把真实用到的列作为查询列表`，这样还可以在`join buffer`中放置更多记录。

2. 驱动表需要选择联接表中较小的表，或尽量使用筛选条件将表记录变得尽量少，这样可以减少`join buffer`加载驱动表的次数。

块嵌套循环联接的执行过程：

1. 扫描表 t1，顺序读取数据行放入 join_buffer 中，放完第 88 行 join_buffer 满了，继续第 2 步；

2. 扫描表 t2，把 t2 中的每一行取出来，跟 join_buffer 中的数据做对比，满足 join 条件的，作为结果集的一部分返回；

3. 清空 join_buffer；

4. 继续扫描表 t1，顺序读取最后的 12 行数据放入 join_buffer 中，继续执行第 2 步。(图中的步骤 4 和 5，表示清空 join_buffer 再复用。)

**04**

**批量键访问**

**(Batched Key** **Access)**

**01. Multi-Range Read 优化 (MRR)**

虽然有了前面两个算法，但是这两个算法还是有可优化的空间的。在介绍优化方案之前，我们介绍一个知识点，即：`Multi-Range Read 优化 (MRR)`。这个优化的主要目的是尽量使用顺序读盘。

我们来看一条 SQL 语句及其执行计划：

```
mysql root@localhost:test> select * from t3 where t3_a>=100 and t3_a<=200;
+------+------+------+
| id   | t3_a | t3_b |
+------+------+------+
| 1001 | 100  | 1001 |
| 1000 | 101  | 1000 |
| 999  | 102  | 999  |
| 998  | 103  | 998  |
| 997  | 104  | 997  |
| 996  | 105  | 996  |
| 995  | 106  | 995  |
| 994  | 107  | 994  |
| 993  | 108  | 993  |
| 992  | 109  | 992  |
| 991  | 110  | 991  |
| 990  | 111  | 990  |
...
mysql root@localhost:test> explain select * from t3 where t3_a>=100 and t3_a<=200;
+----+-------------+-------+-------+---------------+-------+---------+--------+------+-----------------------+
| id | select_type | table | type  | possible_keys | key   | key_len | ref    | rows | Extra                 |
+----+-------------+-------+-------+---------------+-------+---------+--------+------+-----------------------+
| 1  | SIMPLE      | t3    | range | idx_a         | idx_a | 5       | <null> | 101  | Using index condition |
+----+-------------+-------+-------+---------------+-------+---------+--------+------+-----------------------+
```

上面我们可以看到，查询结果是按照索引`idx_a`的顺序访问得到的结果，此时主键 id 的值为倒叙的，但是现实中按照索引`idx_a`排序之后的主键 id 可能是乱序的。假设这样就会出现随机访问，性能相对较差。有什么解决办法呢？按照主键 id 重新排序就可以了。这就是 MRR 优化的设计思路。想要使用 MRR 优化，需要设置`set optimizer_switch="mrr_cost_based=off"`。

我们看一下我们当前的设置：

```
mysql root@localhost:test> show variables like 'optimizer_switch';
+------------------+-----------------------------------------------------------------------------------------------------------------
| Variable_name    | Value                                                                                                                                                                                                                                                                                                                                             
+------------------+-----------------------------------------------------------------------------------------------------------------
| optimizer_switch | index_merge=on,index_merge_union=on,index_merge_sort_union=on,index_merge_intersection=on,engine_condition_pushdown=on,index_condition_pushdown=on,mrr=on,mrr_cost_based=on,block_nested_loop=on,batched_key_access=off,materialization=on,semijoin=on,loosescan=on,firstmatch=on,subquery_materialization_cost_based=on,use_index_extensions=on
```

当我们开始 MRR 后再查询会得到以下结果：

```
mysql root@localhost:test> select * from t3 where t3_a>=100 and t3_a<=200;
+------+------+------+
| id   | t3_a | t3_b |
+------+------+------+
| 901  | 200  | 901  |
| 902  | 199  | 902  |
| 903  | 198  | 903  |
| 904  | 197  | 904  |
| 905  | 196  | 905  |
| 906  | 195  | 906  |
| 907  | 194  | 907  |
| 908  | 193  | 908  |
| 909  | 192  | 909  |
| 910  | 191  | 910  |
...
mysql root@localhost:test> explain select * from t3 where t3_a>=100 and t3_a<=200;
+----+-------------+-------+-------+---------------+-------+---------+--------+------+----------------------------------+
| id | select_type | table | type  | possible_keys | key   | key_len | ref    | rows | Extra                            |
+----+-------------+-------+-------+---------------+-------+---------+--------+------+----------------------------------+
| 1  | SIMPLE      | t3    | range | idx_a         | idx_a | 5       | <null> | 101  | Using index condition; Using MRR |
+----+-------------+-------+-------+---------------+-------+---------+--------+------+----------------------------------+
```

**02. Batched Key Access**

理解了 MRR 性能提升的原理，我们就能理解 `Batched Key Access(BKA)` 算法了。

这个 BKA 算法，其实就是对 NLJ 算法的优化。

NLJ 算法执行的逻辑是：从驱动表 t1，一行行地取值，再到被驱动表 t2 去做 join。也就是说，对于表 t2 来说，每次都是匹配一个值。这时，MRR 的优势就用不上了。那怎么才能一次性地多传些值给表 t2 呢？方法就是，我们就把表 t1 的数据取出来一部分，先放到一个临时内存。这个临时内存不是别人，就是 join_buffer。通过 q 前面的介绍我们知道 join_buffer 在 BNL 算法里的作用，是暂存驱动表的数据。但是在 NLJ 算法里并没有用。那么，我们刚好就可以复用 join_buffer 到 BKA 算法中。

BKA 算法的流程如下：

![](https://mmbiz.qpic.cn/mmbiz_jpg/0DgLwlbnWUDHq0aM9hrDJaZlNmIzFYx1ibZd4TGvnZtkRSqVUIXdBHlib6vKpHKbYoRUVxYqHuP1q15rribypccvA/640?wx_fmt=jpeg)

注意：

· BKA 算法本质上还是 NLJ 算法，其发生的条件为被驱动表上有索引，并且该索引为非主键，并且联接需要访问被驱动表主键上的索引。

· BKA 算法会调用 MRR 接口，批量的进行索引键的匹配，拿到主键 id 后进行排序，然后回表获取数据，此时随机 IO 变为顺序 IO，以此来提高联接的执行效率。

那么，这个 BKA 算法到底要怎么启用呢？

需要在执行 SQL 语句之前，先设置`set optimizer_switch='mrr=on,mrr_cost_based=off,batched_key_access=``on';`其中，前两个参数的作用是要启用 MRR。

```
mysql root@localhost:test> explain select * from t3 left join t2 on t3.t3_a=t2.t2_a where t2.t2_a>=100 and t2.t2_a<=200;
+----+-------------+-------+-------+---------------+-------+---------+--------------+------+----------------------------------------+
| id | select_type | table | type  | possible_keys | key   | key_len | ref          | rows | Extra                                  |
+----+-------------+-------+-------+---------------+-------+---------+--------------+------+----------------------------------------+
| 1  | SIMPLE      | t2    | range | idx_a         | idx_a | 5       | <null>       | 100  | Using index condition; Using MRR       |
| 1  | SIMPLE      | t3    | ref   | idx_a         | idx_a | 5       | test.t2.t2_a | 1    | Using join buffer (Batched Key Access) |
+----+-------------+-------+-------+---------------+-------+---------+--------------+------+----------------------------------------+
```

说完了 NLJ 算法的优化，我们再来看 BNL 算法的优化。

我们执行语句之前，需要通过理论分析和查看 explain 结果的方式，确认是否要使用 BNL 算法。如果确认优化器会使用 BNL 算法，就需要做优化。

```
mysql root@localhost:test> explain select * from t1 join t2;
+----+-------------+-------+------+---------------+--------+---------+--------+------+---------------------------------------+
| id | select_type | table | type | possible_keys | key    | key_len | ref    | rows | Extra                                 |
+----+-------------+-------+------+---------------+--------+---------+--------+------+---------------------------------------+
| 1  | SIMPLE      | t1    | ALL  | <null>        | <null> | <null>  | <null> | 100  | <null>                                |
| 1  | SIMPLE      | t2    | ALL  | <null>        | <null> | <null>  | <null> | 1000 | Using join buffer (Block Nested Loop) |
+----+-------------+-------+------+---------------+--------+---------+--------+------+---------------------------------------+
```

1.  给被驱动表的 join 字段加上索引，把 BNL 算法转成 BKA 算法，这是最好的情况。
    
2.  如果是低频的 SQL 语句，为此创建一个索引有些浪费，但是不创建被驱动表需要扫描的条数比较多，待判断结果也需要进行很多次判断，非常耗 CPU 字段，这时就需要用到临时表，创建一个临时表 temp_t，并且在临时表中创建所需字段索引，将驱动表与临时表 temp_t 进行联接即可；
    
3.  如果驱动表和被驱动表联接时，需要判断的次数比较多，比如驱动表有数据 1000 条，被驱动表有 10W 条数据，那么一共就需要 1000 * 100W = 10 亿次判断；这时我们可以结合业务端构造 hash 判断，理论上效果要好于临时表的方案。实现思路：
    

**05**

**业务中要不要用表联接**

1.  如果可以使用 Index Nested-Loop Join 算法，也就是说可以用上被驱动表上的索引，其实是没问题的；
    
2.  如果使用 Block Nested-Loop Join 算法，扫描行数就会过多。尤其是在大表上的 join 操作，这样可能要扫描被驱动表很多次，会占用大量的系统资源。所以这种 join 尽量不要用。或者经过修改再使用。
    

联表查询语句：

```
select * from t1 join t2 on t1.a=t2.a;
```

单表查询过程：

1.  执行 select * from t1，查出表 t1 的所有数据，假设这里有 100 行；
    
2.  循环遍历这 100 行数据:
    

*   从每一行 R 取出字段 a 的值 $a；
    
*   执行 select * from t2 where a=$a；
    
*   把返回的结果和 R 构成结果集的一行。
    

可以看到，在这个查询过程，也是扫描了 200 行，但是总共执行了 101 条语句，比直接 join 多了 100 次交互。除此之外，客户端还要自己拼接 SQL 语句和结果。显然，这么做还不如直接 join 好。

**06**

**总结**

经过上面的分析，我们能发现联接查询成本占大头的就是 “驱动表记录数 乘以 单次访问被驱动表的成本”，所以我们的优化重点其实就是下面这两个部分：  

*   尽量减少驱动表的记录数
    
*   对被驱动表的访问成本尽可能降低