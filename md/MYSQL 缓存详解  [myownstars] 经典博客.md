> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/zengkefu/p/5606342.html)

  
http://blog.itpub.net/15480802/viewspace-755582/

在服务器级别只提供了 query cache，而在存储引擎级别，MyISAM 和 InnoDB 分别引入了 key cache 和 buffer pool

**什么是 query cache**

Mysql 没有 shared_pool 缓存执行计划，但是提供了 query cache 缓存 sql 执行结果和文本，如果在生命周期内完全相同的 sql 再次运行，则连 sql 解析都免去了；

所谓完全相同，包含如下条件

Sql 的大小写必须完全一样；

发起 sql 的客户端必须使用同样的字符集和通信协议；

sql 查询同一数据库下的同一个表 (不同数据库可能有同名表)；

Sql 查询结果必须确定，即不能带有 now() 等函数；

当查询表发生 DML 或 DDL，其缓存即失效；

针对 mysql/information_schema/performance_schema 的查询不缓存；

使用临时表的 sql 也不能缓存；

开启缓存后，每个 select 先检查是否有可用缓存 (必须对这些表有 select 权限)，而每个写入操作先执行查询语句并使相关缓存失效；

5.5 起可缓存基于视图的查询

Mysql 维护一个 hash 表用来查找缓存，其 key 为 sql text，数据库名以及客户端协议的版本等

相应参数

Have_query_cache：服务器是否支持查询缓存

Query_cache_type：0(OFF) 不缓存；1(ON) 缓存查询但不包括使用 SQL_NO_CACHE 的 sql；2(DEMAND) 只缓存使用 SQL_CACHE 的 sql

Query_cache_size：字节为单位，即使 query_cache_type=0 也会为分配该内存，所以应该一并设置为 0

Query_cache_limit：允许缓存的最大结果集，大于此的 sql 不予缓存

Query_cache_min_res_limit：用于限定块的最小尺寸，默认 4K；

缓存的 metadata 占有 40K 内存，其可分为大小不等的多个子块，各块之间使用双向链表链接；根据其功能分别存储查询结果，基表和 sql text 等；

每个 sql 至少用到两个块：分别存储 sql 文本和查询结果，查询引用到的表各占一个块；

为了减少响应时间，每产生 1 行数据就发送给客户端；

数据库启动时调用 malloc() 分配查询缓存

查询缓存拥有一个全局锁，一旦有会话获取就会阻塞其他访问缓存的会话，因此当缓存大量 sql 时，缓存 invalidation 可能会消耗较长时间；

Innodb 也可以使用查询缓存，每个表在数据字典中都有一个事务 ID 计数器，ID 小于此值的事务不可使用缓存；表如果有锁 (任何锁) 则也不可使用查询缓存；

 **状态变量**

有关 query cache 的状态变量都以 Qcache 打头

mysql> **SHOW STATUS LIKE 'Qcache%';**

+-------------------------+--------+

| Variable_name           | Value  |

+-------------------------+--------+

| Qcache_free_blocks      | 36     |

| Qcache_free_memory      | 138488 |

| Qcache_hits             | 79570  |

| Qcache_inserts          | 27087  |

| Qcache_lowmem_prunes    | 3114   |

| Qcache_not_cached       | 22989  |

| Qcache_queries_in_cache | 415    |

| Qcache_total_blocks     | 912    |

+-------------------------+--------+

Qcache_inserts—被加到缓存中 query 数目

Qcache_queries_in_cache—注册到缓存中的 query 数目

缓存每被命中一次，Qcache_hits 就加 1；

计算缓存 query 的平均大小 =(query_cache_size-Qcache_free_memory)/Qcache_queries_in_cache

Com_select = Qcache_not_cached + Qcache_inserts + queries with errors found during the column-privileges check

Select = Qcache_hits + queries with errors found by parser

**Buffer pool**

innodb 即缓存表又缓存索引，还有设置多个缓冲池以增加并发，很像 oracle

采用 LRU 算法：

所有 buffer 块位于同一列表，其中后 3/8 为 old，每当新读入一个数据块时，先从队尾移除同等块数然后插入到 old 子列的头部，如再次访问该块则将其移至 new 子列的头部

Innodb_buffer_pool_size:  buffer pool 大小

Innodb_buffer_pool_instances: 子 buffer pool 数量，buffer pool 至少为 1G 时才能生效

Innodb_old_blocks_pct: 范围 5 – 95， 默认为 37 即 3/8，指定 old 子列的比重

Innodb_old_blocks_time: 以 ms 为单位，新插入 old 子列的 buffer 块必须等待指定时间后才能移入 new 列，适用于 one-time scan 频繁的操作，以避免经常访问的数据块被剔出 buffer pool

可通过状态变量获知当前 buffer pool 的运行信息

Innodb_buffer_pool_pages_total：缓存池总页数

Innodb_buffer_pool_bytes_data：当前 buffer pool 缓存的数据大小，包括脏数据

Innodb_buffer_pool_pages_data：缓存数据的页数量

Innodb_buffer_pool_bytes_dirty：缓存的脏数据大小

Innodb_buffer_pool_pages_diry：缓存脏数据页数量

Innodb_buffer_pool_pages_flush：刷新页请求数量

Innodb_buffer_pool_pages_free：空闲页数量

Innodb_buffer_pool_pages_latched：缓存中被 latch 的页数量，这些页此刻正在被读或写；然而计算此变量比较消耗资源，只有在 UNIV_DEBUG 被定义了才可用

相关源代码如下

#ifdef UNIV_DEBUG  
  {"buffer_pool_pages_latched",  
  (char*) &export_vars.innodb_buffer_pool_pages_latched,  SHOW_LONG},  
#endif /* UNIV_DEBUG */

Innodb_buffer_pool_pages_misc：用于维护诸如行级锁或自适应 hash 索引的内存页 = 总页数 - 空闲页 - 使用的页数量

Innodb_buffer_pool_read_ahead：预读入缓存的页数量

Innodb_buffer_pool_read_ahead_evicted：预读入但是 1 次都没用就被剔出缓存的页

Innodb_buffer_pool_read_requests：InnoDB 的逻辑读请求次数

Innodb_buffer_pool_reads：直接从磁盘读取数据的逻辑读次数

Innodb_buffer_pool_wait_free：缓存中没有空闲页满足当前请求，必须等待部分页回收或刷新，记录等待次数

Innodb_buffer_pool_write_requests：向缓存的写数量

可使用 innodb standard monitor 监控 buffer pool 的使用情况，主要有如下指标：

Old database pages: old 子列中的页数

Pages made young, not young: 从 old 子列移到 new 子列的页数，old 子列中没有被再次访问的页数

Youngs/s  non-youngs/s: 访问 old 并导致其移到 new 列的次数

**Key cache**

5.5 仅支持一个结构化变量，即 key cache，其包含 4 个部件

Key_buffer_size

Key_cache_block_size：单个块大小，默认 1k

Key_cache_division_limit：warm 子列的百分比 (默认 100)，key cache buffer 列表的分隔点，用于分隔 host 和 warm 子列表

Key_cache_age_threshold：页在 hot 子列中的生命周期，值越小则越快的移至 warm 列表

MyISAM 只缓存索引，

可创建多个 key buffer—set global hot_cache.key_buffer_size=128*1024

索引指定 key buffer—cache index t1 in hot_cache

可在数据库启动时 load index into key_buffer 提前加载缓存，也可通过配置文件自动把索引映射到 key cache

key_buffer_size = 4G

hot_cache.key_buffer_size = 2G

cold_cache.key_buffer_size = 2G

init_file=/**_path_**/**_to_**/**_data-directory_**/mysqld_init.sql

mysqld_init.sql 内容如下

CACHE INDEX db1.t1, db1.t2, db2.t3 IN hot_cache

CACHE INDEX db1.t4, db2.t5, db2.t6 IN cold_cache

默认采用 LRU 算法，也支持名为中间点插入机制 midpoint insertion strategy

索引页刚读入 key cache 时，被放在 warm 列的尾部，被访问 3 次后则移到 hot 列尾并循环移动，如果在 hot 列头闲置连续 N 次都没访问到，则会被移到 warm 列头，成为被剔出 cache 的首选；

N= block no* key_cache_age_threshold/100