> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/RoxQrMVHdIwR4VsppBwNBw)

背景说明：
-----

Mysql 调优，是大家日常常见的调优工作。所以，**Mysql 调优**是一个**非常、非常核心的面试知识点**。在 40 岁老架构师 尼恩的**读者交流群** (50+) 中，其相关面试题是一个非常、非常高频的交流话题。

近段时间，有小伙伴面试京东，说遇到一个去重 调优的面试题：

> 100W 记录去重，是用 distinct 还是 group by，说说理由？

MYsql 设计的时候，如何高性能进行数据去重，也是调优的重点和难点，社群中，还遇到过大概的变种：

> 形式 1：distinct 效率更高还是 group by 效率更高？？
> 
> 形式 2：为啥 group by 会导致 慢 mysql 产生？
> 
> 形式 3：........， 后面的变种很多，都会收入 《尼恩 Java 面试宝典》。

这里尼恩给大家 对数据去重的 调优，做一下系统化、体系化的梳理，使得大家可以充分展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**。

也一并把这个 题目以及参考答案，收入咱们的《[尼恩 Java 面试宝典](http://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247484558&idx=3&sn=3c498b0f8e3897e899acf154ad1ac8ee&chksm=c142be0af635371c06b830243517aae063e23814195df1cc75ab2123c5b3ab9cb1217cbd80e3&scene=21#wechat_redirect)》V73，供后面的小伙伴参考，提升大家的 3 高 架构、设计、开发水平。

> 最新《尼恩 架构笔记》《尼恩高并发三部曲》、《尼恩 Java 面试宝典》 的 PDF 文件，请关注本公众号【技术自由圈】领取，暗号：领电子书

接下来，我们先来看一下 distinct 和 group by 的功能、使用和底层原理。

本文目录
----

**- 背景说明**

**- 首先：来看 distinct 功能和用法**

  - distinct 用法

  - distinct 多列去重

**- 其次：来看 GROUP BY 的使用场景**

**- Group by 底层原理**

**- group by 在去重场景的使用**

  - 单列去重

  - 多列去重

  - group by 多列去重 和 单列去重的区别

**- distinct 和 group by 去重原理分析**

  - Group by 的隐式排序

**- 那么，为啥要优先推荐 group by 呢？**

**- 说在最后**

**- 推荐阅读**

**- 免费领取 11 个技术圣经 PDF**

首先：来看 distinct 功能和用法
--------------------

DISTINCT 是一种用于去除 SELECT 语句返回结果中重复行的关键字。在使用 SELECT 语句查询数据时，如果结果集中包含重复的行，可以使用 SELECT DISTINCT 语句来去除这些重复的行。例如，以下语句返回一个包含重复行的结果集：

```
SELECT name FROM users;


```

可以使用以下语句来去除重复的行：

```
SELECT DISTINCT name FROM users;


```

这样就可以得到一个不包含重复行的结果集。需要注意的是，DISTINCT 关键字会对查询的性能产生一定的影响，因为它需要对结果集进行排序和去重的操作。因此，在使用 DISTINCT 关键字时需要谨慎，尽可能地使用索引来优化查询，以提高查询的性能。

### distinct 用法

```
SELECT DISTINCT columns FROM table_name WHERE where_conditions;


```

例如：

```
mysql> select distinct age from student;
+------+
| age  |
+------+
|   10 |
|   12 |
|   11 |
| NULL |
+------+
4 rows in set (0.01 sec)


```

DISTINCT 关键词用于返回唯一不同的值。放在查询语句中的第一个字段前使用，且**作用于主句所有列**。

注意：DISTINCT 子句将所有 NULL 值视为相同的值

> 如果列具有 NULL 值，并且对该列使用 DISTINCT 子句，MySQL 将保留一个 NULL 值，并删除其它的 NULL 值。

### distinct 多列去重

distinct 多列的去重，则是根据指定的去重的列信息来进行，

即只有**所有指定的列信息都相同**，才会被认为是重复的信息。

```
SELECT DISTINCT column1,column2 FROM table_name WHERE where_conditions;

mysql> select distinct sex,age from student;
+--------+------+
| sex    | age  |
+--------+------+
| male   |   10 |
| female |   12 |
| male   |   11 |
| male   | NULL |
| female |   11 |
+--------+------+
5 rows in set (0.02 sec)


```

  

其次：来看 GROUP BY 的使用场景
--------------------

GROUP BY 主要的使用场景是 在**分组 聚合**。

具体来说，GROUP BY 子句通常用于将查询结果按照一个或多个列进行分组，然后对每个组进行聚合计算。

例如，假设一个表存储了每个人的姓名、年龄和所在城市，可以使用 GROUP BY 子句按照城市对人进行分组，并计算每个城市的平均年龄或人口数量等统计信息。

GROUP BY 子句通常与聚合函数（例如 COUNT、SUM、AVG、MAX 和 MIN）一起使用，以计算每个组的聚合值。例如，可以使用 GROUP BY 子句和 COUNT 函数来计算每个城市中的人数。

但是，除了分组 聚合，GROUP BY 还可以用来 进行数据去重，并且在某些特定场景下，**性能超过 distinct**

需要注意的是：GROUP BY 子句会对结果集进行排序，因此可能会导致使用临时文件排序。如果查询中包含 ORDER BY 子句，使用不当会产生临时文件排序，容易产生慢 SQL 问题。

_具体该怎么优化呢？请持续 跟踪 尼恩 的《_[_尼恩 Java 面试宝典_](http://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247484558&idx=3&sn=3c498b0f8e3897e899acf154ad1ac8ee&chksm=c142be0af635371c06b830243517aae063e23814195df1cc75ab2123c5b3ab9cb1217cbd80e3&scene=21#wechat_redirect)_》，后面结合其他的面试题进行介绍。_

Group by 底层原理
-------------

`group by`语句根据一个或多个列对结果集进行分组。在分组的列上通常配合 COUNT, SUM, AVG 等函数一起使用。

假定有个需求：统计每个城市的用户数量。

对应的 SQL 语句如下：

```
select city ,count(*) as num from user group by city;


```

执行部分结果如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2b2qgJTXeUfWpwRTadicN5LNRCtHIg2CqHNKlEJiaufpBOuus0edcH8Bb7nWlD0dVTS8GbcroLsPQ/640?wx_fmt=png)

先用 **Explain** 查看一下执行计划

```
explain select city ,count(*) as num from user group by city;


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2b2qgJTXeUfWpwRTadicN5m9qnKwzeRJGLqM2FWy2IAsAsYib1xliajEuQS8gIjz55W7uric3WM8hRw/640?wx_fmt=png)

在 Extra 字段里面，我们可以看到以下信息：

*   用到了 **Using temporary**, 表示执行时创建了一个内部临时表。
    
    注意这里的临时表可能是**内存上的临时表，也有可能是硬盘上的临时表**，当然，如果临时表比较小，就是基于内存的，可以肯定的是：**基于内存的临时表的性能高，时间消耗肯定要比基于硬盘的临时表的实际消耗小**。
    
*   用到了 **Using filesort**， 表示执行过程中没有使用索引的排序，而是使用临时文件。
    
    "Using filesort" 是 MySQL 的 EXPLAIN 输出中的一个短语，表示查询需要使用临时文件对结果集进行排序。
    
    这可能发生在查询包括 ORDER BY 子句或 GROUP BY 子句时，而数据库无法使用索引满足排序顺序。
    
    使用临时文件对大型结果集进行排序可能会导致磁盘 I/O 和内存使用方面的昂贵开销，因此最好尽可能避免 “Using filesort”。
    
    一些避免文件排序的策略包括使用适当的索引优化查询，限制结果集的大小或修改查询以使用不同的排序算法。
    

那么`group by`语句为啥会同时用到临时表和临时文件 排序呢？

首先看下整个执行流程：

1.  在执行过程中首先创建内存临时表，表里有`city`, `num`两个字段，`city`为主键。
    
2.  扫描 user 表，依次取出一行数据，数据中`city`字段的值为 c;
    

*   如果临时表中没有主键为 c 的行， 则插入一条新纪录（c ， 1）；
    
*   如果存在，则更新该行为 （c, num + 1)；
    

3.  遍历完后，再根据`city`进行排序，最后将结果集返回给客户端。
    

这个流程的执行示意图如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2b2qgJTXeUfWpwRTadicN5hk18R90hrM9jBuvXYxU114Uaic189ibVBajSqJ5JAiaebn3yhrVDKwSpQ/640?wx_fmt=png)

然后进入到第二阶段，进行内存排序。其中对内存临时表的排序执行步骤，本质上和的`order by` 流程基本一致。

1.  数据从内存临时表中拷贝到`sort buffer`中
    
2.  `sort buffer`进行排序，根据实际情况采用全字段排序或 rowid 排序，
    
3.  排序结果写回到内存临时表中，
    
4.  从内存临时表中返回结果集。
    

流程示意图如下所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2b2qgJTXeUfWpwRTadicN5YhHRVS2ovOsLvgo3DOOdlSndmmCh9dT0BEG0gdlKribCxpXicCSgIpIQ/640?wx_fmt=png)

group by 在去重场景的使用
-----------------

分为两个去重场景进行介绍：

*   单列去重
    
*   多列去重
    

单列去重和多列去重的区别在于去重的依据不同。

单列去重是指针对某一列数据进行去重，即将该列中重复的值只保留一个。例如，如果有一个包含重复数据的姓名列表，可以使用单列去重将重复的姓名去除，只保留一个。

多列去重是指针对多列数据进行去重，即将多列数据中重复的行只保留一行。例如，如果有一个包含姓名、年龄和所在城市的列表，可以使用多列去重将重复的姓名、年龄和城市都相同的行去除，只保留一行。

### 单列去重

GROUP BY 子句，大部分都是用于单列去重。

例如，假设有一个表包含学生姓名和所在城市，可以使用以下 SQL 语句进行单列去重，以获取不同城市的学生数量：

```
SELECT city, COUNT(DISTINCT name)  FROM student  GROUP BY city;


```

在上述语句中，使用 GROUP BY 子句对城市进行分组，并使用 COUNT 和 DISTINCT 函数计算每个城市中不同姓名的数量。这将返回一个结果集，其中每个行包含一个城市和该城市中不同姓名的数量，从而达到了单列去重的目的。

对于单列去重来说，group by 的使用和 distinct 类似:

单列去重语法：

```
SELECT columns FROM table_name WHERE where_conditions GROUP BY columns;



```

执行：

```
mysql> select age from student group by age;
+------+
| age  |
+------+
|   10 |
|   12 |
|   11 |
| NULL |
+------+
4 rows in set (0.02 sec)



```

注意：和 DISTINCT 子句一样，group by 将所有 NULL 值视为相同的值

> 如果列具有 NULL 值，并且对该列使用 group by 子句，MySQL 将保留一个 NULL 值，并删除其它的 NULL 值。

  

### 多列去重

GROUP BY 子句也可以用于多列去重。

例如，假设有一个表包含学生姓名、所在城市和年龄，可以使用以下 SQL 语句进行多列去重，以获取不同城市、不同年龄的学生数量：

```
SELECT city, age, COUNT(DISTINCT name) 
FROM student
GROUP BY city, age;


```

在上述语句中，使用 GROUP BY 子句对城市和年龄进行分组，并使用 COUNT 和 DISTINCT 函数计算每个城市、年龄组合中不同姓名的数量。这将返回一个结果集，其中每个行包含一个城市、一个年龄和该城市、年龄组合中不同姓名的数量，从而达到了多列去重的目的。

多列去重语法：

```
SELECT column1, column2 FROM table_name WHERE where_conditions GROUP BY column1, column2;


```

多列去重例子：

```
mysql> select sex,age from student group by sex,age;
+--------+------+
| sex    | age  |
+--------+------+
| male   |   10 |
| female |   12 |
| male   |   11 |
| male   | NULL |
| female |   11 |
+--------+------+
5 rows in set (0.03 sec)


```

  

### group by 多列去重 和 单列去重的区别

GROUP BY 子句用于将结果集按照指定的列进行分组，并对每个组进行聚合操作。在 GROUP BY 子句中指定的列将成为聚合键，用于将结果集中的行分组。**在聚合操作之后，可以使用 GROUP BY 子句来去重**。

单列去重是指使用 GROUP BY 子句将结果集**按照单个列**进行分组，并对每个组进行聚合操作。例如，以下查询将返回一个包含不同城市名称的列表：

```
SELECT city FROM mytable GROUP BY city;


```

多列去重是指使用 GROUP BY 子句将结果集**按照多个列**进行分组，并对每个组进行聚合操作。例如，以下查询将返回一个包含不同城市和州的列表：

```
SELECT city, state FROM mytable GROUP BY city, state;


```

group by 的原理是先对结果进行分组排序，然后返回**每组中的第一条**数据。所以，区别在于：**单列去重只按照一个列进行分组，而多列去重则按照多个列进行分组**。

distinct 和 group by 去重原理分析
--------------------------

在大多数例子中，DISTINCT 可以被看作是特殊的 GROUP BY，它们的实现都基于分组操作，且都可以通过**松散索引扫描、紧凑索引扫描**来实现。

> 松散索引扫描和紧凑索引扫描都是 MySQL 中的索引扫描方式。
> 
> 松散索引扫描（Loose Index Scan）是指 MySQL 在使用索引进行查询时，如果索引中的数据不连续，MySQL 将会扫描整个索引树，直到找到符合条件的记录。这种扫描方式会增加查询的时间复杂度，因为需要扫描整个索引树。
> 
> 紧凑索引扫描（Tight Index Scan）是指 MySQL 在使用索引进行查询时，如果索引中的数据连续，MySQL 将会按照顺序读取索引数据块，直到找到符合条件的记录。这种扫描方式会减少查询的时间复杂度，因为可以按照顺序读取索引数据块，避免了扫描整个索引树。
> 
> 通常情况下，如果使用的是 InnoDB 存储引擎，MySQL 会自动选择使用紧凑索引扫描。但是，如果使用的是 MyISAM 存储引擎，或者查询条件中包含了不等于（<>）或不包含（NOT IN）操作符，MySQL 将会使用松散索引扫描。
> 
> 总的来说，紧凑索引扫描比松散索引扫描更快，因为它可以避免扫描整个索引树。但是，如果使用的是 MyISAM 存储引擎，或者查询条件中包含了不等于或不包含操作符，MySQL 将会使用松散索引扫描，这时候就需要注意查询的效率。

DISTINCT 和 GROUP BY 都是**可以使用索引进行扫描搜索**的。

例如以下两条 sql，我们对这两条 sql 进行分析，可以看到，在 extra 中，这两条 sql 都使用了紧凑索引扫描`Using index for group-by`。

所以，在一般情况下，对于相同语义的 DISTINCT 和 GROUP BY 语句，我们可以对其使用相同的索引优化手段来进行优化。

```
mysql> explain select int1_index from test_distinct_groupby group by int1_index;
+----+-------------+-----------------------+------------+-------+---------------+---------+---------+------+------+----------+--------------------------+
| id | select_type | table                 | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra                    |
+----+-------------+-----------------------+------------+-------+---------------+---------+---------+------+------+----------+--------------------------+
|  1 | SIMPLE      | test_distinct_groupby | NULL       | range | index_1       | index_1 | 5       | NULL |  955 |   100.00 | Using index for group-by |
+----+-------------+-----------------------+------------+-------+---------------+---------+---------+------+------+----------+--------------------------+
1 row in set (0.05 sec)

mysql> explain select distinct int1_index from test_distinct_groupby;
+----+-------------+-----------------------+------------+-------+---------------+---------+---------+------+------+----------+--------------------------+
| id | select_type | table                 | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra                    |
+----+-------------+-----------------------+------------+-------+---------------+---------+---------+------+------+----------+--------------------------+
|  1 | SIMPLE      | test_distinct_groupby | NULL       | range | index_1       | index_1 | 5       | NULL |  955 |   100.00 | Using index for group-by |
+----+-------------+-----------------------+------------+-------+---------------+---------+---------+------+------+----------+--------------------------+
1 row in set (0.05 sec)


```

但对于 GROUP BY 来说，在 MYSQL8.0 之前，GROUP Y 默认会依据字段进行**隐式排序**。

可以看到，下面这条 sql 语句在使用了临时表的同时，还进行了 filesort。

```
mysql> explain select int6_bigger_random from test_distinct_groupby GROUP BY int6_bigger_random;
+----+-------------+-----------------------+------------+------+---------------+------+---------+------+-------+----------+---------------------------------+
| id | select_type | table                 | partitions | type | possible_keys | key  | key_len | ref  | rows  | filtered | Extra                           |
+----+-------------+-----------------------+------------+------+---------------+------+---------+------+-------+----------+---------------------------------+
|  1 | SIMPLE      | test_distinct_groupby | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 97402 |   100.00 | Using temporary; Using filesort |
+----+-------------+-----------------------+------------+------+---------------+------+---------+------+-------+----------+---------------------------------+
1 row in set (0.04 sec)


```

  

### Group by 的隐式排序

对于隐式排序，我们可以参考 Mysql 官方的解释：

> GROUP BY 默认隐式排序（指在 GROUP BY 列没有 ASC 或 DESC 指示符的情况下也会进行排序）。然而，GROUP BY 进行显式或隐式排序已经过时（deprecated）了，要生成给定的排序顺序，请提供 ORDER BY 子句。
> 
> 参考连接：
> 
> MySQL :: MySQL 5.7 Reference Manual :: 8.2.1.14 ORDER BY Optimization (https://dev.mysql.com/doc/refman/5.7/en/order-by-optimization.html)

所以，在 Mysql8.0 之前, Group by 会**默认**根据作用字段（Group by 的后接字段）对结果进行**排序**。在能利用索引的情况下，Group by 不需要额外进行排序操作；但当无法利用索引排序时，Mysql 优化器就不得不选择通过使用临时表然后再排序的方式来实现 GROUP BY 了。要命的是，**当临时结果集的大小超出系统设置临时表大小时，Mysql 会将临时表数据 copy 到磁盘上面再进行操作**，语句的执行效率会变得极低。这也是 Mysql 选择将此操作（隐式排序）**弃用**的原因。

基于上述原因，Mysql 在 8.0 时，对此进行了优化更新：

> 从前（Mysql5.7 版本之前），Group by 会根据确定的条件进行隐式排序。
> 
> 在 mysql 8.0 中，已经移除了这个功能，所以不再需要通过添加 order by null 来禁止隐式排序了，但是，查询结果可能与以前的 MySQL 版本不同。
> 
> 如果要生成给定顺序的结果，请按通过 ORDER BY 指定需要进行排序的字段。

MySQL :: MySQL 8.0 Reference Manual :: 8.2.1.16 ORDER BY Optimization (https://dev.mysql.com/doc/refman/8.0/en/order-by-optimization.html)

因此，我们的结论也出来了：

*   在语义相同，有索引的情况下：group by 和 distinct 都能使用索引，效率相同。因为 group by 和 distinct 近乎等价，distinct 可以被看做是特殊的 group by。
    
*   在语义相同，无索引的情况下：distinct 效率高于 group by。原因是：distinct 和 group by 都会进行分组操作，但在 Mysql8.0 之前 group by 会进行隐式排序，导致触发 filesort，sql 执行效率低下。从 Mysql8.0 开始，Mysql 就删除了隐式排序，**所以，此时在语义相同，无索引的情况下，group by 和 distinct 的执行效率也是近乎等价的**。
    

总之，从 Mysql8.0 开始， 不管有索引 、还是索引， group by 和 distinct 的执行效率也是近乎等价的

但是， 100W 级数据去重场景，优先推荐使用 group by。

那么，为啥要优先推荐 group by 呢？
----------------------

相比于 distinct 来说，group by 的语义明确。

1.  group by 语义更为清晰
    
2.  group by 可对数据进行更为复杂的一些处理
    
3.  由于 distinct 关键字会对所有字段生效，在进行复合业务处理时，group by 的使用灵活性更高，
    
4.  group by 能根据分组情况，对数据进行更为复杂的处理，例如通过 having 对数据进行过滤，或通过聚合函数对数据进行运算。
    

所以，不论是 100W 级数据去重场景，还是普通数据去重场景，建议优先选用  group by

说在最后
----

本文收录于 《尼恩 Java 面试宝典》 V73 版 PDF，请关注本公众号【技术自由圈】领取，暗号：领电子书

基本上，把尼恩的 《尼恩 Java 面试宝典》吃透，大厂 offer 很容易拿到滴。

最终，**让面试官爱到 “不能自已、口水直流”**。offer， 也就来了。

学习过程中，如果有啥问题，大家可以来 找 40 岁老架构师尼恩交流。