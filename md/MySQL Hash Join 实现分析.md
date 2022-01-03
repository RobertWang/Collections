> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/JBs2fJe4JhRXTrkAfI3Hew)

提示：公众号展示代码会自动折行，建议横屏阅读
======================

**「前言」**
========

连接操作是一种数据库中最基本的操作，连接算法的执行效率直接影响到整个数据库的效率、吞吐和资源。通常商业数据库系统一般有三种主流的连接实现：Nested Loop Join、Hash Join 和 Sort Merge Join。本文概述目前主流的 Hash Join 实现方式，以及分析 MySQL 中 Hash Join 的实现方式。

MySQL 8.0.18 版本增加了对 Hash Join 算法的支持，在此之前，连接算法仅支持嵌套循环连接 Nested Loop Join。Nested Loop Join 是一个双重循环的结构，外层循环遍历外表 (outer table，又称驱动表、左表)，对于外表的每一行记录，内层循环遍历内表（inner table，又称被驱动表、右表）来判断是否符合连接条件。假设外表存储在 M 个 page 上有 m 条记录，内表存储在 N 个 page 上有 n 条记录。可以得知，Nested Loop Join 的 IO 代价为 M+(m*N)。

Hash Join 可以通过 Hash 的方式降低复杂度：根据连接条件对外表建 hash 表，对于内表的每一行记录也根据连接条件计算 hash 值，只需要验证对应 hash 值是否能否匹配就完成了连接操作。那么，外表分别读了两遍写了一遍，而内表只遍历了 “一趟”，IO 代价为 3*M+N。这种简单直接的哈希连接称为 Basic Hash Join。

如果外表较大，或者可供 Hash Join 计算使用的内存过小，以至于外表不能全部加载到内存，就需要相对复杂的分批处理。Grace Hash Join 利用多层 Hash，使得切分后的分片能够存储于内存中。On-disk Hash Join 不断迭代左表，每当填充满内存时暂停迭代左表，接下来探测完所有右表数据。清空内存后继续迭代左表，反复上述流程直到处理完所有左表中的数据。我们结合 MySQL 的实现，具体介绍这几种哈希连接的实现方式。

**「第一部分 几种 Hash Join 概述」**
==========================

参考 MySQL 介绍 Hash Join 的示例，例如 SQL：

<table><colgroup><col><col></colgroup><tbody><tr><td><p>1<br>2<br>3<br>4</p></td><td><p>SELECT<br>&nbsp;&nbsp;given_name,country_name<br>FROM<br>&nbsp;&nbsp;persons JOIN countries ON persons.country_id= countries.country_id;</p></td></tr></tbody></table>

**1.1 Basic H****ash Join（In-memory Hash Join）**
------------------------------------------------

内存能够存储外表时，可以直接依赖内存执行 Basic Hash Join，所以又被称为 In-memory Hash Join。执行 Hash Join 一般包括两个过程，创建 hash 表的 build 过程，和探测 hash 表的 probe 过程。

1). build 过程：遍历外表，以连接条件为 key，查询需要的列作为 value 创建 hash 表。

2). probe 过程：逐行遍历内表，对于内表的每行记录，根据连接条件计算 hash 值，并在 hash 表中查找。如果匹配到外表的记录，则输出，否则跳过，直到遍历完成所有内表的记录。

![](https://mmbiz.qpic.cn/mmbiz_png/ZB8eCu1DibibLIfibmJeJ79Z6TNM9sDIkIZfv3EomBTzqB5pribFw5Ga5FLfnAnXs0rkT1SyTBT7nMAoB5Yxjsa6Jw/640?wx_fmt=png)

如图所示，左侧是 build 过程，对应的 countries 表是外表；右侧是 probe 过程，对应的 persons 表是内表。其中，country_id 是连接条件，两表由此计算 hash 值。

**1.2 On-disk Hash Join**
-------------------------

Basic Hash Join 的限制在于内存需要装载整个外表。所以，一般选择参与 join 的两个表 (经过其他条件过滤后的结果集) 中较小的表作为外表，使得内存更容易存放 hash 表。在 MySQL 中，Join 可以使用的内存通过参数 join_buffer_size 控制。如果维护外表的 hash table 所需的内存超过 join_buffer_size，那么 Basic Hash Join 就需要调整为 On-disk Hash Join。

On-disk Hash Join 为了控制内存占用，将外表分成若干片段执行，使得内存能够容纳单个分片。每当外表填充满 hash 表时就截断 build 过程。然后，针对每个被截断的分片，都执行一遍内表全量数据的 Proble 过程。假设外表分成了 k 片，那么将扫描 k 次内表，总体 IO 代价是 3*M+k*N。

这种 “多趟” 执行的方式导致内表 IO 开销较大，整体性能差。但是，对于上层算子需要尽快返回数据时就不见得性能差。比如，上层存在 Limit 算子，只需要 5 行计算结果，可能第一个分段就能产生所需的 5 行记录，相当于外表只做了部分的 build 工作，内表也在产生 5 行结果以后停止了 probe 过程。

想要避免 “多趟” 操作时，Build 阶段可以用 hash 算法将数据存入磁盘中对应的分区文件中；然后在 probe 阶段，对于内表使用同样的 hash 算法进行分区。由于使用分片 hash 函数相同，那么 key 相同 (join 条件相同) 必然在同一个分片编号的分区文件中。接下来，再对外表和内表中相同分片编号的数据进行 Basic Hash Join 计算，所有分片计算完成后，整个 join 过程结束。这种算法的代价是，外表和内表在 build 阶段进行一次读 IO 和一次写 IO，在 probe 阶段进行了一次读 IO，所以整体 IO 开销是 3*(M+N)。相对于之前需要 k 次扫描内表的方式，当前算法更优。

在 MySQL8.0.22 中，前者和后者两种算法都有涉及，具体实现我们在后面展开。

![](https://mmbiz.qpic.cn/mmbiz_png/ZB8eCu1DibibLIfibmJeJ79Z6TNM9sDIkIZwroqKOWLGdibaJemlWQlZFWRFqYuEnMXjNCQftmC3MhvHsX3mic6LVibA/640?wx_fmt=png)

左上侧图是外表的 build + 分片过程，右上侧图是内表的 Probe + 分片存储过程，最下面的图是对分片进行 probe 过程。

**1.3 Grace Hash Join**
-----------------------

Grace Hash Join 能够解决内存不足下的连接问题，利用分治思想，先将外表和内表按哈希，切分到不同分片。然后在外表和内表对应的分片上做 Basic Hash Join。如果对应分片仍然超过内存大小则对分片继续执行一次 Grace Hash Join，直到可以存入内存。

这个过程与 MySQL 中的 Hash Join 类似，主要的区别在于内存不足时的解决方法。当出现 join_buffer_size 不足时，MySQL 会对外表进行分片，然后再进行 Basic Hash Join。但极端情况下，如果数据严重倾斜，可能哈希后的分片仍然超过可用内存大小。MySQL 的规避方式是参考 On-disk Hash Join 的方式分批处理：读满 hash 表后停止 build 过程，然后执行一趟 probe。处理这批数据后，清空 hash 表，在上次 build 停止的位点继续 build 过程来填充 hash 表，填充满再做一趟内表分片完整的 probe。直到处理完所有 build 数据。

Grace Hash Join 在遇到这种情况时，继续执行一次 Grace Hash Join，直到可以存入内存。

**1.4 Hybrid Hash Join**
------------------------

历史经验告诉我们，磁盘 IO 和重复计算应尽量避免。Basic Hash Join 完全利用内存，避免了磁盘 IO；同时，数据只看一遍，避免了重复计算。Grace Hash Join 解决了内存不足问题，但对于磁盘 IO 优化不足。Hybrid Hash Join 正是继承 Grace Hash Join 的分治思想来解决内存不足问题，又学习 Basic Hash Join 高效利用内存的优势：在 Build 阶段，在内存中尽可能保留一些完整的分片，不能够被保存在内存中的分片才被落盘；在 Probe 时，内存中完整的外表分片可以被直接探测，这部分分片做到了和 Basic Hash Join 一样的效率。参考 Grace Hash Join，其余的不能完整存放在内存中的分片再继续处理。从 Build 阶段最小化磁盘 IO 的角度，On-disk Hash Join 的章节中可以发现 MySQL 中也保留了一份内存大小的 hash 表，避免了这部分数据的 IO。

接下来，我们介绍 MySQL 中 Hash Join 的使用场景及具体实现方法。

**「第二部分 可使用场景（基于规则）」**
======================

等值连接、笛卡尔积、非等值连接（容易产生误解）

**2.1 join on 上的等值与非等值条件**
--------------------------

hash join 将 join on 上条件分为了等值连接条件和非等值连接条件

```
mysql> explain format = tree  select * from tt1 left join tt2 on tt1.c2 = tt2.c2 and tt1.c1 > tt2.c1;
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| EXPLAIN                                                                                                                                                                                                 |
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| -> Left hash join (tt2.c2 = tt1.c2), extra conditions: (tt1.c1 > tt2.c1)  (cost=1.13 rows=6)
    -> Table scan on tt1  (cost=0.45 rows=2)
    -> Hash
        -> Table scan on tt2  (cost=0.28 rows=3)
 |
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

  
hash join 允许用户仅指定非等值连接条件，这时 hash join 的等值连接条件为 no condition，extra conditions（存储非等值连接条件）

```
mysql> explain format = tree  select * from tt1 left join tt2 on tt1.c1 > tt2.c1;
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| EXPLAIN                                                                                                                                                                                              |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| -> Left hash join (no condition), extra conditions: (tt1.c1 > tt2.c1)  (cost=1.13 rows=6)
    -> Table scan on tt1  (cost=0.45 rows=2)
    -> Hash
        -> Table scan on tt2  (cost=0.28 rows=3)
 |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

**2.2 内连接时，非等值连接条件构建成单独的 Filter iterator**
------------------------------------------

```
mysql> explain format = tree  select * from tt1 inner join tt2 on tt1.c2= tt2.c2 and tt1.c1 > tt2.c1;
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| EXPLAIN                                                                                                                                                                                                                              |
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| -> Filter: (tt1.c1 > tt2.c1)  (cost=1.30 rows=2)
    -> Inner hash join (tt2.c2 = tt1.c2)  (cost=1.30 rows=2)
        -> Table scan on tt2  (cost=0.18 rows=3)
        -> Hash
            -> Table scan on tt1  (cost=0.45 rows=2)
 |
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

即使 inner join 仅有一个非等值连接条件  

```
mysql> explain format = tree  select * from tt1 inner join tt2 on tt1.c1 > tt2.c1;
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| EXPLAIN                                                                                                                                                                                                                           |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| -> Filter: (tt1.c1 > tt2.c1)  (cost=1.30 rows=2)
    -> Inner hash join (no condition)  (cost=1.30 rows=2)
        -> Table scan on tt2  (cost=0.18 rows=3)
        -> Hash
            -> Table scan on tt1  (cost=0.45 rows=2)
 |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

对于 OR 谓词连接的 join on 条件，即使都是等值连接条件，都作为整体放入 extra conditions（外连接）或 Filter iterator（内连接），这里实际缺少 union 的改写（or 改为 union 连接的两个 SQL）  

```
mysql> explain format = tree select * from tt1 inner join tt2 on tt1.c2 = tt2.c2 or tt1.c1 = tt2.c1;
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| EXPLAIN                                                                                                                                                                                                                                                  |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| -> Filter: ((tt2.c2 = tt1.c2) or (tt2.c1 = tt1.c1))  (cost=1.30 rows=3)
    -> Inner hash join (no condition)  (cost=1.30 rows=3)
        -> Table scan on tt2  (cost=0.21 rows=3)
        -> Hash
            -> Table scan on tt1  (cost=0.45 rows=2)
 |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
mysql> explain format = tree select * from tt1 left  join tt2 on tt1.c2 = tt2.c2 or tt1.c1 = tt2.c1;
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| EXPLAIN                                                                                                                                                                                                                     |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| -> Left hash join (no condition), extra conditions: ((tt2.c2 = tt1.c2) or (tt2.c1 = tt1.c1))  (cost=1.13 rows=6)
    -> Table scan on tt1  (cost=0.45 rows=2)
    -> Hash
        -> Table scan on tt2  (cost=0.28 rows=3)
 |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

**「第三部分 自动使用的场景」**
==================

目前，MySQL 8.0.22 中采用基于规则的判断来决定使用何种连接算子。

```
/**
For a given slice of the table list, build up the iterator tree corresponding
to the tables in that slice. It handles inner and outer joins, as well as
semijoins (“first match”).
The join tree in MySQL is generally a left-deep tree of inner joins,
so we can start at the left, make an inner join against the next table,
join the result of that against the next table, etc..
**/
AccessPath *ConnectJoins(){   // sql_executor.cc
  // If this is a BNL, we should replace it with hash join. We did decide
    // during create_access_paths that we actually can replace the BNL with a
    // hash join, so we don't bother checking any further that we actually can
    // replace the BNL with a hash join.
    const bool replace_with_hash_join =    //标记是否能够转为hash join
        UseHashJoin(qep_tab) && !QueryMixesOuterBKAAndBNL(qep_tab->join()) &&
        !PushedJoinRejectsHashJoin(qep_tab->join(), left_tables, right_tables,
                                   JoinType::INNER);
  //…………
  //最终贪心生成最优计划
      if (is_bka) {
        path = CreateBKAAccessPath();
      } else if(replace_with_sort_merge_join) {
        // TO DO: case for interesting order
      } else if (replace_with_hash_join) {
        // The numerically lower QEP_TAB is often (if not always) the smaller
        // input, so use that as the build input.
        path = CreateHashJoinAccessPath();
        // Attach any remaining non-equi-join conditions as a filter after the join.
        path = PossiblyAttachFilter();
      } else {
        path = CreateNestedLoopAccessPath();
        SetCostOnNestedLoopAccessPath();
      }  
}
```

MySQL 8.0.22 中已经存在 Hash Join 代价的计算模型，只是非常不完善的计算方式，计算代价只用作返回给上层算子（及 explain 输出）并未实际影响连接算子的选择。

```
double joined_rows = outer->num_output_rows * inner->num_output_rows;
path->num_output_rows = joined_rows * pos_outer->filter_effect;
path->cost = inner->cost + pos_outer->read_cost +
cost_model.row_evaluate_cost(joined_rows)
```

**「第四部分 生成 Hash Join 路径」**
==========================

**4.1 连接表达式**
-------------

连接表达式归类为：

*   等值连接条件（hash join conditions），用作构建 Hash Key，用来构建和探测 Hash Table
    
*   其它条件（extra conditions），包括非等值连接条件和其它不能下压的单表条件
    

**4.2 是否允许 Hash Table 落盘**
--------------------------

左表（build/right input table）构建 Hash Table 时，是否允许落盘直接决定了 Hash Join 返回第一行的时间：

*   不允许落盘时，需要立即执行 Probe 探测当前 Hash Table，从而 “尽快” 返回上层算子执行结果；探测完成一遍后，需要重复填充 Hash Table，探测再重新执行
    
*   允许落盘时，左表 “一次性” 执行完成 build hash table 过程，从而 “较晚” 返回上层算子执行结果，但是探测过程可以 “一口气” 执行完成
    

是否允许 allow_spill_to_disk 主要基于规则：上层算子是否期望所有结果集。具体来说：

*   如果查询中有 Limit，查询优化器将不允许哈希连接溢出到磁盘。这样做的效果是，hash join 将更早地开始生成结果行，从而更快地达到 Limit 的结果。即，Hash Join 不落盘时需要立即处理当前 Hash Table 中的数据，形成连接结果来返回上层，而不是对 build input table（左表，构建 hash table）全部构建好以后才探测。理想情况下，这应该在优化过程中决定。
    
*   但是有两种情况查询优化器总是允许 Hash Join 溢出到磁盘，即查询中有分组或排序时。因为在这些情况下，上层的迭代器（Group By & aggregation）很可能会以任何方式消耗整个结果集。换句话说，上层很可能需要所有 Hash Join 产生的结果集，而不是 limit 那种几行就可能满足的情况。
    
*   如果此表是 pushed join query 的一部分，则必须读取从属子表中的行，同时定位子表所依赖的 pushed 祖先表中的行。所以，不允许中间落盘。
    

**4.3 Filter 算子构建**
-------------------

在 #1.2. 节中已经展示了需要额外构建 Filter iterator 的情况  

**「第五部分 执行流程」**
===============

**5.1 整体流程**
------------

volcano style iterator：init() →build_hash_table() 我们将这个过程标记为①

![](https://mmbiz.qpic.cn/mmbiz_png/ZB8eCu1DibibLIfibmJeJ79Z6TNM9sDIkIZb3bwCiarcL6HHEOIH6w3DCX0SaJcAiaXDobCPI4nooVzkPyFhRe5nHCg/640?wx_fmt=png)

read()，其中 read() 具体实现了一个状态机的转换过程，我们将具体的实现标记为②~⑥

![](https://mmbiz.qpic.cn/mmbiz_png/ZB8eCu1DibibLIfibmJeJ79Z6TNM9sDIkIZ3WOSC91YViahYUl3wB7Yf3sn6xgibRcUcDNaqIrWVdGmSDqGLYD5YBdw/640?wx_fmt=png)

```
int HashJoinIterator::Read() {
for (;;) {
  switch (m_state) {
     case State::READING_ROW_FROM_PROBE_ITERATOR: ReadRowFromProbeIterator ②
     case State::READING_FIRST_ROW_FROM_HASH_TABLE:
     case State::READING_FROM_HASH_TABLE:   ReadNextJoinedRowFromHashTable(); ③
     case State::LOADING_NEXT_CHUNK_PAIR:  ReadNextHashJoinChunk()  ④
     case State::READING_ROW_FROM_PROBE_CHUNK_FILE: ReadRowFromProbeChunkFile() ⑤
     case State::READING_ROW_FROM_PROBE_ROW_SAVING_FILE: ReadRowFromProbeRowSavingFile()  ⑥
}
```

**5.2 具体实现**
------------

为了在内存可控的情况下支持大表连接，Hash Join 实现类似 Grace Hash Join，依赖 join_buffer_size（system variable）参数控制内存中 hash table 大小，依赖 allow_spill_to_disk（查询优化器控制）参数控制是否允许执行过程中落盘，这两个参数直接影响了 hash join 的实现的方式。前者决定了何时需要停止往内存中加载数据，后者决定了 Hash Join 执行方式以 “多趟” 的 block hash join 还是以类似 On-disk Hash Join 方式执行。

### **5.2.1 Build Hash Table**

总计有三个地方会调用 HashJoinIterator::BuildHashTable() 函数，分别在 Hash Table Iterator::Init() 时、HashJoinIterator::ReadRowFromProbeIterator() 时和 HashJoinIterator::ReadRowFromProbeRowSavingFile() 时，后两者在 Probe 阶段执行，我们可以先关注算子 Init 时执行的 build hash table。

![](https://mmbiz.qpic.cn/mmbiz_png/ZB8eCu1DibibLIfibmJeJ79Z6TNM9sDIkIZ3wLib1eib4oxicnQ2LNN5dWYiafzHDZzfAibT4qHh8M2MUVM5CJiby5upbzw/640?wx_fmt=png)

具体来看

```
bool HashJoinIterator::BuildHashTable() {
  //当不允许hash table写盘时（allow_spill_to_disk=false），完整的执行hash join可能会需要“多趟”填充内存的hash table（可以理解为是否需要“多趟”由Limit等上层算子决定，即第一趟填充hash table后，probe阶段如果返回了足够Limit使用的行，则不再需要“多趟”）
  //如果确需“多趟”执行，需要关注build/left input table最后的一行返回值情况：
  // Restore the last row that was inserted into the row buffer. This is
  // necessary if the build input is a nested loop with a filter on the inner
  // side, like this:
  //
  //        +---Hash join---+
  //        |               |
  //  Nested loop          t1
  //  |         |
  //  t3    Filter: (t3.i < t2.i)
  //               |
  //              t2
  //
  // 直观来说，为了正确返回左表的下一行记录，需要还原左表recode0当时的状态，比如t3.i，这时调用左表的read（）时，才能正确找到t2的下一行匹配的位置
  for (;;) {  // Termination condition within loop.  //一直迭代左表，直到完成或当前hash table填满（>join_buffer_size）
    int res = m_build_input->Read();
    if (res == -1) {
      return false;  //没有其他行，完成
    }
    const hash_join_buffer::StoreRowResult store_row_result =
        m_row_buffer.StoreRow(thd(), reject_duplicate_keys,
                              store_rows_with_null_in_join_key);  //构建 hash key，存储<hash key, row> 
    switch (store_row_result) { 
      case hash_join_buffer::StoreRowResult::ROW_STORED:  //继续，不超join_buffer_size阈值
        break;  
      case hash_join_buffer::StoreRowResult::BUFFER_FULL: {  //超阈值
        if (!m_allow_spill_to_disk) {     //判断是否允许
          if (m_join_type != JoinType::INNER) {
            // 需要注意，外连接时，未产生连接的记录仍需要保留，以留给下一趟 hash table探测
            InitWritingToProbeRowSavingFile();
          }
          return false; //开始Probe
        }
        if (InitializeChunkFiles() { //根据估计值构建chunk file，目标使得最大的文件不超过 join_buffer_size大小
          return true;
        }
        //这里有两种方式，作者选的是：将build table 剩余的行 写出到块文件。然后，将在Proble table是否与已写入哈希表的行匹配后也将Probe的记录写入块文件
    //另一种方法将所有行都写到块文件。Probe table也写到磁盘。但作者不想浪费已经存储在内存中的行。
        if (WriteRowsToChunks(thd(), m_build_input.get(), m_build_input_tables,   //这里m_build_input.get() 直接将bulid input指针传给WriteRowsToChunks（），使其直接独立完成剩余所有行的处理
                              m_join_conditions, kChunkPartitioningHashSeed,
                              &m_chunk_files_on_disk,
                              true /* write_to_build_chunks */,
                              false /* write_rows_with_null_in_join_key */,
                              m_tables_to_get_rowid_for,
                              &m_temporary_row_and_join_key_buffer)) {
          return true;
        }
        return false;
      }
    }
  }
}
```

另外，在构建 hash key 和存储当前 row 时，作者使用了重用了同一段缓冲区  

```
static bool WriteRowToChunk() {
  bool null_in_join_key = ConstructJoinKey(thd, join_conditions, tables.tables_bitmap(), join_key_and_row_buffer);//这里的join_key_and_row_buffer根据join_conditions(等值连接条件)提取了拼接的join key
  const uint64_t join_key_hash =
      MY_XXH64(join_key_and_row_buffer->ptr(),
               join_key_and_row_buffer->length(), xxhash_seed);   //构建hash key
  const size_t chunk_index = join_key_hash & (chunks->size() - 1);
  ChunkPair &chunk_pair = (*chunks)[chunk_index];   
  if (write_to_build_chunk) {
    return chunk_pair.build_chunk.WriteRowToChunk(join_key_and_row_buffer,  //虽然传了join_key_and_row_buffer，但是WriteRowToChunk()函数中会重置，并拷贝表缓存中的记录到join_key_and_row_buffer，然后拷贝到对应的文件中
                                                  row_has_match);
  } else {
    return chunk_pair.probe_chunk.WriteRowToChunk(join_key_and_row_buffer,
                                                  row_has_match);
  }
}
```

首先根据等值连接条件构建 Key，然后拼接一整行 row，最后存储当前 <key,row> 到 hash table。8.0.23 前的代码比较容易理解：  

```
StoreRowResult HashJoinRowBuffer::StoreRow(THD *thd, bool reject_duplicate_keys,bool store_rows_with_null_in_condition) {
  m_buffer.length(0);
  for (const HashJoinCondition &hash_join_condition : m_join_conditions) {
    bool null_in_join_condition =
        hash_join_condition.join_condition()->append_join_key_for_hash_join(
            thd, m_tables.tables_bitmap(), hash_join_condition, &m_buffer);  //首先根据所有的等值连接条件构建Key 
  }
  const size_t join_key_size = m_buffer.length();
  uchar *join_key_data = nullptr;
  if (join_key_size > 0) {
    join_key_data = m_mem_root.ArrayAlloc<uchar>(join_key_size);
    memcpy(join_key_data, m_buffer.ptr(), join_key_size);   //为了重用m_buffer这个变量，这里做了一次拷贝，感觉上并不需要（直接存成单独的Key 和 row）
  }
  if (StoreFromTableBuffers(m_tables, &m_buffer)) {  //m_buffer被写进所有列数据， StoreFromTableBuffers函数会首先重置m_buffer，然后处理blob字段、和其它字段，类似于filesort，为了内存开销将列做pack
    return StoreRowResult::FATAL_ERROR;
  }
  const size_t row_size = m_buffer.length();
  uchar *row = nullptr;
  if (row_size > 0) {
    row = m_mem_root.ArrayAlloc<uchar>(row_size);
    memcpy(row, m_buffer.ptr(), row_size); //这里又做了一次拷贝
  }
  m_last_row_stored = m_hash_map->emplace(Key(join_key_data, join_key_size),    //执行插入<Key,row>
                                          BufferRow(row, row_size)); 
  if (m_mem_root.allocated_size() > m_max_mem_available) {  //插入后判断空间
    return StoreRowResult::BUFFER_FULL;
  }
}
```

### 这里有明显的多次拷贝代价，8.0.23 版本中已经做了优化。StoreFromTableBuffers 拆分为两个函数；一个调整字符串缓冲区大小，另一个实际写入值。后者可以被重用，以便直接写入 MEM_ROOT，减少重复拷贝问题。

###   
**5.2.2 Probe Hash Table**  

Probe 阶段入口函数利用状态机来控制具体的调用流程，状态转换如下

![](https://mmbiz.qpic.cn/mmbiz_png/ZB8eCu1DibibLIfibmJeJ79Z6TNM9sDIkIZyOic0XQXsvS6DLpgQP5Mnd769w2I4nfGS87dslDZIrcnehZRWVHibqPw/640?wx_fmt=png)

从流程②开始执行，读到内表 (右表，probe_input) 一行数据后，在 HashTable 中定位当前行匹配的区间，用迭代器的 begin 和 end 表示，并交给流程③。

③中负责迭代 begin 到 end，判断是否符合额外的条件（extraConditions）。这时候决定如何对外输出结果：inner 和 outer 符合条件的行对外输出并保存在 Saving file；semi join 则直接输出一行结果。

Probe 过程中有两点实现比较难理解，一点是 ReadRowFromProbeRowSavingFile，另外一点是外连接的实现。

1.  在做 On-disk Hash Join 时，对于内存仍然保存不下一个 chunk 时
    
2.  在做‘多趟’ Basic Hash Join 时
    

先处理当前已经加载到 Hash Table 中的记录，Probe 对应的 probe.chunk_file（场景 1）或 probe_input_table(场景 2）。当连接类型为 Semi Join 和 Anti Join 并且已经在 Hash Table 中找到了匹配的记录时，即当前 Probe 的记录能明确如何处理，所以我们无需关心未加载到 Hash Table 中的数据即可处理当前记录，即当前记录只需要 “看” 这一遍。除此之外（内连接、外连接、半连接但未匹配），每行记录仍有可能与当前不在 Hash Table 中的记录存在匹配，所以暂时存储在 ReadRowFromProbeRowSavingFile 中。等到 Probe 侧全部探测结束，这时 Probe 中需要 “再看一次” 的记录也已经存储到了 ReadRowFromProbeRowSavingFile 中。接下来，Hash Table 清空，继续从 build.chunk_file 上次填充满的位置(场景 1）或 Build_input_table（场景 2）加载到 Hash Table，这时候只需要探测 ReadRowFromProbeRowSavingFile 而不是整个 Probe.chunk_file 或 Probe_input_table。

外连接的实现：

从 Probe 过程看，有三类 Probe 路径：Probe_input_table(ReadRowFromProbeIterator，即基表）、Probe.chunk_file(READING_ROW_FROM_PROBE_CHUNK_FILE，即 On-disk Hash Join 分片）和 Probe.ReadRowFromProbeRowSavingFile(ReadRowFromProbeRowSavingFile，过程⑥）。

根据是否能立即输出结果，可以分为两类：

1.  前两种，在不存在第三种时（On-disk Hash Join 能够存入内存 或 Basic Hash Join build_input 能一次存入内存），可以直接输出结果，否则外连接需要处理第三种路径才能输出结果
    
2.  第三种，在过程⑥中，每次加入 ProbeRowSavingFile 时，标记了此行记录是否被匹配过，这时才能清楚如何处理每一行记录是否置 NULL 输出
    

这里会看到 m_build_input→SetNullRowFlag，只会出现在 build_input，是因为在查询优化阶段已经调整了连接顺序。例如 T1 left join T2，这时会在 T2 侧构建 Hash Table（即上文 build_input)，T1 探测。而 T1 right join T2，会被转换成 T2 left join T1。所以，置 NULL 操作只需要出现在 build_input 侧。对于 Full Outer Join/ Full Join，MySQL 不支持，所以 Hash Join 也并未实现。

我们以一个例子来说明外连接的执行过程，例如：

<table resolved="" width="187"><thead><tr><th colspan="3" data-darkmode-bgcolor-16412238157883="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16412238157883="#fff|rgb(244, 245, 247)" data-style="border-top-width: 1px; border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; text-align: left; min-width: 8px; background-color: rgb(244, 245, 247);"><span data-darkmode-bgcolor-16412238157883="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16412238157883="#fff|rgb(244, 245, 247)">Table A</span></th></tr></thead><colgroup><col><col><col></colgroup><tbody><tr><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;"><br></td><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;">a</td><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;">b</td></tr><tr><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;" class="">记录 1</td><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;">1</td><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;">1</td></tr><tr><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;">记录 2</td><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;">2</td><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;">2</td></tr></tbody></table><table resolved="" width="187"><thead><tr><th colspan="3" data-darkmode-bgcolor-16412238157883="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16412238157883="#fff|rgb(244, 245, 247)" data-style="border-top-width: 1px; border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; text-align: left; min-width: 8px; background-color: rgb(244, 245, 247);"><span data-darkmode-bgcolor-16412238157883="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16412238157883="#fff|rgb(244, 245, 247)">Table B</span></th></tr></thead><colgroup><col><col><col></colgroup><tbody><tr><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;"><br></td><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;">a</td><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;">b</td></tr><tr><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;">记录 1</td><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;">1</td><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;">1</td></tr><tr><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;">记录 2</td><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;">2</td><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;">2</td></tr></tbody></table>

SQL：SELECT * FROM A LEFT JOIN B ON A.a = B.a;  
假如内存 Hash 表只能存一行记录，那么 Build 过程 Hash 表中存储一行。（这里假设 hash 函数是 %10）

<table resolved="" width="187"><thead><tr><th colspan="2" data-darkmode-bgcolor-16412238157883="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16412238157883="#fff|rgb(244, 245, 247)" data-style="border-top-width: 1px; border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; text-align: left; min-width: 8px; background-color: rgb(244, 245, 247);" class=""><span data-darkmode-bgcolor-16412238157883="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16412238157883="#fff|rgb(244, 245, 247)">HashTable</span></th></tr></thead><colgroup><col><col></colgroup><tbody><tr><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;">hash(a)</td><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;">value(a,b)</td></tr><tr><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;">1</td><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;">(1,1)</td></tr></tbody></table>

Probe Table B 时，  
1）读到 B 的记录 1，已存在连接，不需要存储到 ProbeRowSavingFile  
2）读到 B 的记录 2，需要存储到 ProbeRowSavingFile，因为不确定 Table A 剩余的没存储在 HashTable 中的记录是否还存在 a=2 的值。

Probe 结束后，清空 HashTable 后再填充 HashTable，加入 Table A 中的记录 2：

<table resolved="" width="187"><thead><tr><th colspan="2" data-darkmode-bgcolor-16412238157883="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16412238157883="#fff|rgb(244, 245, 247)" data-style="border-top-width: 1px; border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; text-align: left; min-width: 8px; background-color: rgb(244, 245, 247);"><span data-darkmode-bgcolor-16412238157883="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16412238157883="#fff|rgb(244, 245, 247)">HashTable</span></th></tr></thead><colgroup><col><col></colgroup><tbody><tr><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;">hash(a)</td><td data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;">value(a,b)</td></tr><tr><td colspan="1" data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;">2</td><td colspan="1" data-style="border-color: rgb(193, 199, 208); padding-top: 7px; padding-bottom: 7px; vertical-align: top; min-width: 8px; text-align: left;">(2,2)</td></tr></tbody></table>

接下来探测 ProbeRowSavingFile，而无需重新做一遍整个 Probe input。

**「第六部分 总结」**
=============

从 MySQL 8.0.18 到 8.0.27，Hash Join 不断优化，已经初具雏形，填补了 Nested Loop Join 的不足。对于大表连接、选择率比较低的连接，以及 Nested Loop Join 无法构建 ref 的场景，Hash Join 有明显的性能优势。但是，现有的 Hash Join 仍然比较原始，在 Hash Table 存储、磁盘对应分片的探测、探测计算下推方面还有很多提升的空间。