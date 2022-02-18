> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/sPO-6ULwIfUexLY3V4acBg)

> 本文介绍了基于 MySQL 的 SQL 优化分析实战。

一、背景

随着业务不断迭代，系统中出现了较多的 SQL 慢查。慢查虽不致命，但会让商家感知到系统较慢，影响使用体验。在进行慢查优化过程中，我们积累了一些经验。本文将基于我们的实战经历，讲解工作中比较常见的慢查原因，以及如何去优化。

本文讲解基于 MySQL 5.7。

二、慢查优化

本节主要针对常见的慢查进行分类，讲解怎么去优化。

**2.1** **建立索引的****正确姿势**

数据量较大的时候，如果没有索引，那么数据库只能全表一行一行的遍历判断数据，因此优化 SQL 的时候，第一步要做的就是确定有没有合适的可用的索引。在业务本身比较复杂的情况下，一个表会涉及各种各样的查询方式，因此我们需要建立各种各样的索引去提高查询。然而索引数量过多又会影响增删改的效率，并且也会占用更多额外的空间去存储索引，因此我们要懂得怎么去正确的建立索引，避免索引滥用。

**2.1.1 索引不要包含选择性过低字段**

选择性过低，即通过该字段只能过滤掉少部分的数据，是没必要建立索引的，因为如果该数据只是占小部分，即使没有索引直接查询数据表也不用过多的遍历即可找到目标数据，没有必要基于索引查询。

SQL：

```
select * from my_table where col_a=1 and col_b=1
```

索引：  
index (col_a,col_b)

col_b 为逻辑删除字段，包含 0 未删除和 1 已删除，数据库中的值只有很少量部分是逻辑删除的。但是在业务中我们一般都只查未删除的，那么这种情况 col_b 是完全不必要在索引中的，可以把 col_b 从组合索引中去掉。

**2.1.2 选择性高的字段前置或者单独建立索引**

SQL：

```
select * from my_table where col_a=1 and col_b=1 and col_c=1
```

索引：  
index(col_a,col_b,col_c)

假设选择性 col_c>col_b>col_a，抛开业务本身需要，组合索引建立的顺序尽可能建为 index(col_c,col_b,col_a)。

原因是组合索引底层的存储先按照第一个进行排序，第一个字段相同再按照第二字段排序，如果选择性低的字段放在前面，因此选择性高的字段放前面相对而言 IO 的次数可能会减少一些。比如基于第一个字段过滤完会有 10000 条数据，基于第二个字段过滤完只有 100 条数据，如果先查第一个字段则需要在 10000 条数据的基础上再进行过滤查询，而基于第二字段过滤完只需要基于 100 条数据进行过滤查询即可。

而如果 col_c 选择性特别高，经过 col_c 过滤后只剩下极少的数据甚至一条数据，单独建立 col_c 索引就可以。

需要注意的是这个规则特别提到了抛开业务本身需要，比如如果 col_a 选择性比 col_b 高一点相差不大，但是 col_b 在业务场景中通用性更强，那么 col_b 放在前面更合适, 可以减少创建的索引。

**2.1.3 尽量使用覆盖索引**

SQL：

```
SELECT sum(col_c) FROM my_table where col_a=1 and col_b=1
```

索引：  
index(col_a,col_b)

如果 col_a 和 col_b 过滤完后还有大量数据，那么建议建一个 index(col_a,col_b,col_c) 索引，否则 MySQL 需要通过大量回表去查询 col_c 的数据再去求和。

**2.1.4 小结**

1、选择性低的字段不用建立索引。

2、具有唯一性或者高选择性的字段无需与其他字段建立组合索引。

3、除了业务需求上的考虑，尽量将选择性高的索引字段前置。

4、在经过索引过滤后数据量依旧很大的情况下可以考虑通过覆盖索引优化。

**2.2 使用索引的正确姿势**

除了 SQL 本身没有适用的索引，有了相关的索引但是对应的索引没有生效是比较常见的情况，以下列举一些常见的失效场景，在日常的开发中，我们要尽量避免。

  
需要注意的是，索引失效这里指的是没有利用到索引的二分查询进行数据过滤。因为存在 ICP，所以会存在触发了失效场景执行计划还是显示使用了索引的情况。

**2.2.1 最左匹配截断**

SQL：

```
select * from my_table where col_b=1  
select * from my_table order by col_b
```

索引:  
index(col_a,col_b)

组合索引的匹配规则是从左往右匹配，无论是作为过滤条件还是排序条件都要遵循这个原则。如果要使用 col_b 字段走索引，查询条件则必须要携带 col_a 字段。

补充说明:  
1、col_b 作为排序字段如果要走索引，只要保证组合索引中 col_b 前面的字段都可以包含在过滤条件或者排序条件中即可，也不需要保证 col_b 作为组合索引中的最后一个字段。  

比如：

```
select * from my_table order by col_a,col_b
```

col_a 和 col_b 都可以走索引。

2、如果 col_b 是作为过滤条件，则 col_b 前面的字段都应该在过滤条件中。  
比如：

```
select * from my_table where col_b=1 order by col_a
```

col_a 和 col_b 都走不了索引，因为 col_a 在组合索引左边，但是 col_a 不在查询条件中。

**2.2.2 隐式转换**

字段类型：  
col_a(VARCHAR)  
col_b(DATETIME)

索引：  
index1(col_a)  
index2(col_b)

SQL：

```
select * from my_table where col_a=1  
select * from my_table where col_b=1603296000000
```

失效原因  
字段类型和查询数据的值类型不一致，会导致字段上的索引失效。

*   col_a 是字符类型，使用了数字类型进行查询。
    
*   col_b 是 datetime 类型，针对 datetime/date/time 类型，MySQL 增删查改都要基于字符串形式日期去处理，否则 MySQL 就需要额外进行转换。（虽然底层储存的是数字类型，但是并不是存储时间戳，底层是处理是统一将外部传入的字符串进行转换，比如是 date 类型通过将 “2021-12-01” 字符串转数字 20211201 这种形式去存储）。
    

**2.2.3 in** **+ order by 导致排序失效**

索引：  
index(col_a,col_b)

SQL：

```
select * from my_table where col_a in (1,2) order by col_b
```

解决方式：

*   如果 col_a 的过滤性不高，在组合索引中可以通过将 col_b 字段前置，将 col_a 移动到组合索引后面，只用于避免或减少回表。
    
*   如果 col_a 的过滤性高，过滤后的数据相对较少，则维持当前的索引即可，剩余不多的数据通过 filesort 进行排序。
    
*   如果存在大量数据，并且经过 col_b 过滤后还是存在大量数据，建议基于别的数据存储实现，比如 Elasticsearch。
    

另外 SQL 建议调整为只查询 id（或者其他已经在索引中的字段），再根据 id 去查对应的数据。可以促使 SQL 走覆盖索引进一步优化、也可以促使 MySQL 底层在进行 filesort 使用更优的排序算法。

**2.2.4 范围查询阻断组合索引**

索引：  
index(col_a,col_b)

SQL：

```
select * from table where col_a >'2021-12-01' and col_b=10
```

解决方式：  
可以调整下索引顺序，col_a 放在最后面。index(col_b,col_a)

**2.2.5 后缀匹配不能走索引**

索引:  
index(col_a,col_b)

SQL：

```
select * from table where col_a=1 and col_b like '%name%'
```

以上 SQL 会导致索引失效。前缀匹配比如 name% 是可以走索引的，但是后缀匹配比如 %name 会导致没办法基于索引树进行二分查找。如果需要进行后缀匹配，数据量较大建议基于 Elasticsearch 实现。

**2.2.6 or 查询导致失效**

索引：  
index(col_a,col_b)

SQL：

```
select * from table where col_a=1 or col_b=''
```

or 查询会导致索引失效，可以将 col_a 和 col_b 分别建立索引，利用 Mysql 的 index merge(索引合并) 进行优化。本质上是分别两个字段分别走各自索引查出对应的数据，再将数据进行合并。

**2.2.7 使用函数查询或运算**

索引：  
index(col_a,col_b)

SQL：

```
select * from table where col_a=1 and DATE_SUB(CURDATE(), INTERVAL 7 DAY) <= date(col_b);  
select * from table where col_a=1 and col_b+1=10
```

**2.2.8 不等于、不包含 (只用到 ICP)**

索引：  
index(col_a,col_b,col_c)

SQL：

```
select * from table where col_a=1 and col_b not in (1,2)  
select * from table where col_a=1 and col_b != 1
```

**2.2.9 选择性过低，直接走全表**

选择性过低会导致索引失效。由于通过二级索引查询后还有回表查询的开销，如果通过该字段只能过滤少量的数据，整体上还不如直接查询数据表的性能，则 MySQL 会放弃这个索引，直接使用全表扫描。底层会根据表大小、IO 块大小、行数等信息进行评估决定。  

索引：  

index(col_a)

SQL：

```
select * from table where col_a>'2017-10-22'
```

**2.2.10 asc 和 desc 混用**

索引：  
index(col_a,col_b,col_c)

SQL：  

```
select * from my_table where col_a=1 order by col_b desc,col_c asc
```

desc 和 asc 混用时会导致索引失效，不建议混用。

**2.2.11 小结**

根据以上例子，总结几个索引失效的场景：

1.  组合索引左匹配原则
    
2.  发生隐式转换
    
3.  组合索引，in + order by in 会阻断排序用索引
    
4.  范围查询会阻断组合索引, 索引涉及到范围查询的索引字段要放在组合索引的最后面。
    
5.  前模糊匹配导致索引失效
    
6.  or 查询，查询条件部分有索引，部分无索引，导致索引失效。
    
7.  查询条件使用了函数运算、四则运算等。
    
8.  使用了!=、not in
    
9.  选择性过低
    
10.  asc 和 desc 混用
    

2.3 编写 SQL 的正确姿势

懂得怎么建立索引，也懂得了怎么避免索引失效，但是有些场景即使索引走对了，也会慢查，这时我们要考虑优化我们 SQL 写法。

**2.3.1 深分页**

索引：

index(col_c)

SQL：

```
select * from my_table where col_c=1 limit 1000,10
```

为什么深分页会效率变差，首先我们要了解一下分页的原理。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PfMGv3PxR7ibEyRbjV8KD53yksfBszYJbnRUkrcQia7f13FyYuctCbxCI0OuicOiaQJVKMf8pzrqFYbj6FeD4TH6UA/640?wx_fmt=png)

MySQL limit 不会传递到引擎层，只是在服务层进行数据过滤。查询数据时，先由引擎层通过索引过滤出一批数据（索引过滤），然后服务层进行二次过滤（非索引过滤）。

  
引擎层过滤后会将获取的数据暂存，服务层一条一条数据获取，获取时引擎层回表获得完成数据交给服务层，服务层判断是否匹配查询条件（非索引过滤），如果匹配会继续判断是否满足 limit 限制的数据范围，符合并且范围内的数据都查完了才返回。

所以如果深分页，会导致大量的无效回表（前 1000 条进行了回表，实际上只需要 1000-1010 的数据），因此优化的方式就是避免深分页带来的额外回表。  

解决方式：

```
# 避免深分页
select * from my_table where id>上次查询的数据id值 limit 100
# 延迟关联 避免大量回表
SELECT * FROM my_table t1,(select id from my_table where col_c=1 limit 1000,100) t2 where t1.id=t2.id
```

避免深分页： 我们可以改成 id 过滤，每次都只查询大于上次查询的数据 id。这样每次只查询 100 条，回表也只需要回表 100 条。  

覆盖索引： 如果业务需要的字段比较少，可以通过保证 SQL 查询的字段和查询条件都在索引上，避免回表。  

延迟关联： 通过延迟关联，通过编写完全基于索引查询数据的 SQL，再根据 id 查询详细的字段数据。

**2.3.2 order by id**

索引：  
index(col_a)

SQL：

```
select * from table where col_a=1 and col_b=2 order by id
```

MySQL INNODB 二级索引最终叶子结点引用的都是主键 id，因此我们可以利用这个点去使用 id 排序。  

但是在本场景中，col_b 截断了索引，导致 SQL 没法利用 id 进行索引排序。而主键索引的权重会比较高，可能会导致 MySQL 没有正确选择索引，从而选择了可能性能更差的主键索引去做排序，查询条件通过遍历扫描数据。

  
因此在不能保证 id 排序可以走索引的情况下，建议改用其他字段排序。如果查询结果集确定会比较少排序字段可以不在索引上，如果结果集较大还是要把排序字段加到索引中。

三、慢查分析

在掌握了 SQL 优化的理论知识之后，我们怎么验证编写的 SQL 是否有按照预期使用了比较合适的索引？这就需要学会分析 SQL 的执行情况。

执行计划：我们可以通过 explain 关键字查看 SQL 的执行计划，通过执行计划分析 SQL 的执行情况。

执行计划字段描述：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PfMGv3PxR7ibEyRbjV8KD53yksfBszYJbficQkUvbl1ZibgGhslxHMP2UIUfA6xV90ia00BwtoBsiaN7nphYgxHMOog/640?wx_fmt=png)

extra 字段常见值：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PfMGv3PxR7ibEyRbjV8KD53yksfBszYJbhLYQdF8riaEtO1La8OF1Q9SlicWsC8KPq0A79X8VtH4vFDbZADnr7MKQ/640?wx_fmt=png)

using index condition 补充说明：  
using index condition 表示使用了 ICP（Index Condition Pushdown 索引下推），ICP 是 MySQL 5.6 之后新增的特性，部分字段因为某些情况无法走索引查询，则会展示 using where（在 server 层进行数据过滤），ICP 是在存储引擎层进行数据过滤，而不是在服务层过滤，利用索引现有的数据过滤调一部分数据。

using where 和 using index condition 的区别：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PfMGv3PxR7ibEyRbjV8KD53yksfBszYJb6EvIZsRRI5icibxSQKUFfa9ibNnmorzkD3omPh7ebXNREKCNy0bELofBw/640?wx_fmt=png)

为什么需要 ICP 机制：

索引:

index(col_a,col_b)

SQL：  

```
select * from my_table where col_a="value" and col_b like "%value%"
```

如果没有 using index condtion，col_a 会走索引查询，匹配到对应的数据后，回表查出剩余字段信息，再去匹配 col_b。假设 col_a 过滤后还有 10000 条数据，而通过 col_b 过滤后只会剩余 1000 条数据，则有 9000 条的数据是没必要的回表。

本质上索引树上是包含 col_b 字段的，只是 col_b 不能利用索引树二分查找特性（因为使用了前模糊匹配），但是可以利用索引上现有的数据进行遍历，减少无效回表。有了 ICP 后，基于索引就可以过滤 col_a 和 col_b 字段，过滤后只会剩下 1000 条数据，减少了大量的回表操作。

**小结：  
通过执行计划我们可以分析出 SQL 最终使用了什么索引，对索引的使用是处于什么情况，进而可以得出还有没有优化空间。**

四、总结

我们要有质量意识，做好预防而不是做补救，SQL 优化在开发阶段就要考虑清楚，而不是等上线后出现慢查了才去优化。

  
做好 SQL 优化可以记住一个口诀，有用高。SQL 要有索引 (建立正确的索引)，索引要可用（避免索引失效），最后要考虑高效（覆盖索引、索引的选择性）。