> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/37447670)

TIDB 和 MySQL 事务隔离级别对比
---------------------

1 事务 ACID 概念
------------

*   A(Atomicity) : 原子性，事务中的一系列操作要么全部完成，要么全部不完成，不能做了一半不做了。这个最好理解，比如转账不能扣完 A 的钱，不给 B 加钱。
*   I(Isolation) : 隔离性，多个事务之间相互隔离的特性。首先不同业务对事务的隔离等级要求不一样，有的严格要求隔离，有的并不是那么严格。因此数据库系统都会实现多种隔离级别，从技术角度讲，每种隔离级别都需要不同的技术手段来保证，通常来说涉及各种锁和 MVCC 机制。
*   D(Durability) : 持久性，事务一旦提交，所修改的数据就会被持久化，后面即使发生任何异常都不会出现数据丢失。这个容易理解，要么数据直接落盘，要么数据操作日志落盘。但是通常情况下数据库系统也一般会根据数据重要性提供多种持久化策略供客户端选择使用，比如对于重要数据，就会要求数据同步落盘之后才能算事务完成，这是最严格的持久化策略；而对于部分不重要数据，可能只会要求数据异步落盘就算事务完成。
*   C(Consistency) : 一致性，要求事务必须始终保持系统处于一致的状态。比如 A 转账给 B，转账前账户总额和转账后账户总和需要保持一致。

2 隔离性简述
-------

*   数据操作过程中利用数据库的锁机制或者多版本并发控制机制获取更高的隔离等级。但是，随着数据库隔离级别的提高，数据的并发能力也会有所下降。所以，如何在并发性和隔离性之间做一个很好的权衡就成了一个至关重要的问题。
*   数据库事务的隔离级别有 4 个，由低到高依次为 Read uncommitted、Read committed、Repeatable read、Serializable，这四个级别可以逐个解决脏读、不可重复读、幻读这几类问题。
*   我们讨论隔离级别的场景，主要是在多个事务并发的情况下。

3.TIDB 的事务隔离级别 (官方文档)
---------------------

*   根据官方文档：TIDB 实现了读已提交和可重复读。
*   TiDB 使用 [percolator 事务模型](https://link.zhihu.com/?target=https%3A//research.google.com/pubs/pub36726.html)，当事务启动时会获取全局读时间戳，事务提交时也会获取全局提交时间戳，并以此确定事务的执行顺序，如果想了解 TiDB 事务模型的实现可以详细阅读以下两篇文章：[TiKV 的 MVCC（Multi-Version Concurrency Control）机制](https://link.zhihu.com/?target=https%3A//pingcap.com/blog-cn/mvcc-in-tikv/)，[Percolator 和 TiDB 事务算法](https://link.zhihu.com/?target=https%3A//pingcap.com/blog-cn/percolator-and-txn/)。

3.1TIDB 的可重复读
-------------

*   可重复读是 TiDB 的默认隔离级别，当事务隔离级别为可重复读时，只能读到该事务启动时已经提交的其他事务修改的数据，未提交的数据或在事务启动后其他事务提交的数据是不可见的。对于本事务而言，事务语句可以看到之前的语句做出的修改。
*   对于运行于不同节点的事务而言，不同事务启动和提交的顺序取决于从 PD 获取时间戳的顺序。
*   处于可重复读隔离级别的事务不能并发的更新同一行，当时事务提交时发现该行在该事务启动后，已经被另一个已提交的事务更新过，那么该事务会回滚并启动自动重试。

3.2 与 ANSI 可重复读隔离级别的区别
----------------------

*   尽管名称是可重复读隔离级别，但是 TiDB 中可重复读隔离级别和 ANSI 可重复隔离级别是不同的，按照 [A Critique of ANSI SQL Isolation Levels](https://link.zhihu.com/?target=https%3A//www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tr-95-51.pdf) 论文中的标准，TiDB 实现的是论文中的 snapshot 隔离级别，该隔离级别不会出现幻读，但是会出现写偏斜，而 ANSI 可重复读隔离级别不会出现写偏斜，会出现幻读。

3.3 与 MySQL 可重复读隔离级别的区别
-----------------------

*   MySQL 可重复读隔离级别在更新时并不检验当前版本是否可见，也就是说，即使该行在事务启动后被更新过，同样可以继续更新。这种情况在 TiDB 会导致事务回滚并后台重试，重试最终可能会失败，导致事务最终失败，而 MySQL 是可以更新成功的。 MySQL 的可重复读隔离级别并非 snapshot 隔离级别，MySQL 可重复读隔离级别的一致性要弱于 snapshot 隔离级别，也弱于 TiDB 的可重复读隔离级别。

3.4 读已提交
--------

*   读已提交隔离级别和可重复读隔离级别不同，它仅仅保证不能读到未提交事务的数据，需要注意的是，事务提交是一个动态的过程，因此读已提交隔离级别可能读到某个事务部分提交的数据。
*   不推荐在有严格一致要求的数据库中使用读已提交隔离级别。

3.5 事务重试
--------

*   对于 insert/delete/update 操作，如果事务执行失败，并且系统判断该错误为可重试，会在系统内部自动重试事务。

4.Mysql 和 TIDB 的隔离级别对比
----------------------

4.1Read uncommitted
-------------------

*   读未提交，顾名思义，就是一个事务可以读取另一个未提交事务的数据。
*   未提交读的数据库锁情况:  
    

*   事务在读数据的时候并未对数据加锁。
*   事务在修改数据的时候只对数据增加行级共享锁。

*   表现  
    

*   事务 1 读取某行记录时，事务 2 也能对这行记录进行读取、更新；当事务 2 对该记录进行更新时，事务 1 再次读取该记录，能读到事务 2 对该记录的修改版本，即使该修改尚未被提交。
*   事务 1 更新某行记录时，事务 2 不能对这行记录做更新，直到事务 1 结束。

![](https://pic2.zhimg.com/v2-1bef2f6c0361d8ef4a75b7e5bbd0dbc9_b.jpg)

*   刚开始，1 号事务和 2 号事务看到的 A 都是 A0，接着 1 号事务将 A0 更新为 A1，再接着 2 号事务就读到 A1 新值，1 号事务将 A1 回滚回了 A0。
*   对比（set session transaction isolation level read uncommitted;）  
    

*   MySQL

![](https://pic4.zhimg.com/v2-eeaedee4d1a2dafd100d09982cd86f2b_b.jpg)

*   TIDB（这个图片最后面有点问题，TIDB 的结果应该是 20，手误）

![](https://pic3.zhimg.com/v2-21f367ad45bb7a91fdb87c5acacdd94e_b.jpg)

4.2 read committed
------------------

*   提交读，顾名思义，就是一个事务要等另一个事务提交后才能读取数据。
*   提交读的数据库锁情况:  
    

*   事务对当前被读取的数据加 行级共享锁（当读到时才加锁），一旦读完该行，立即释放该行级共享锁；
*   事务在更新某数据的瞬间（就是发生更新的瞬间），必须先对其加 行级排他锁，直到事务结束才释放。

*   很显然，Read Committed 是与 Read Uncommitted 是相对的，意思是说 1 号事务可以在 2 号事务提交之后看到 2 号事务修改的数据。这种隔离级别可以避免脏读，但是又引入了一个新的问题：不可重复读，如下图所示：

![](https://pic3.zhimg.com/v2-7ff77ef2476d27d29a9a5d4f3565c1fe_b.jpg)

*   但是从上面的例子中我们也看到，事务一两次读取的结果并不一致，所以提交读不能解决不可重复读的读现象。
*   简而言之，提交读这种隔离级别保证了读到的任何数据都是提交的数据，避免了脏读。但是不保证事务重新读的时候能读到相同的数据，因为在每次数据读完之后其他事务可以修改刚才读到的数据。
*   对比（set session transaction isolation level read committed;）  
    

*   MySQL:

![](https://pic1.zhimg.com/v2-773d09742d880abf9632cfb3adc178c4_b.jpg)

*   TIDB:

![](https://pic2.zhimg.com/v2-fd89d5f97cd5ff68220bb06a11e15eb9_b.jpg)

4.3.repeatable read
-------------------

*   重复读，就是在开始读取数据（事务开启）时，不再允许修改操作 。
*   重复读可以解决不可重复读问题。不可重复读对应的是修改，即 UPDATE 操作。但是可能还会有幻读问题。因为幻读问题对应的是插入 INSERT 操作，而不是 UPDATE 操作。
*   可重复读的数据库锁情况

*   事务在读取某数据的瞬间（就是开始读取的瞬间），必须先对其加 行级共享锁，直到事务结束才释放。
*   事务在更新某数据的瞬间（就是发生更新的瞬间），必须先对其加 行级排他锁，直到事务结束才释放。

*   从字面意思来看这种隔离级别修复了不可重复读这样的问题，表现如下图所示：

![](https://pic3.zhimg.com/v2-8eddaa2a2d9e748a45c22ab182c08852_b.jpg)

*   可以看出，无论 1 号事务如何更新 A，2 号事务在随后的进程中看到的 A 值都是事务开始第一次看到的 A 值（A0）。虽然解决了不可重复读的问题，但是还有一个问题－幻读：

![](https://pic4.zhimg.com/v2-fb4c41c3a5595cdb13679077094bd113_b.jpg)

*   上图中 1 号事务在事务过程中插入了一个大于 B0 的新值 B2，2 号事务在插入操作前后读取 B > 0 的时候读到的值却不同。
*   对比 1（set session transaction isolation level repeatable read;）  
    

*   MySQL 和 TIDB 行为一致：

![](https://pic1.zhimg.com/v2-31ea5e3c01d7c7046716d8eb8bbff5e8_b.jpg)

*   对比 2（set session transaction isolation level repeatable read;）  
    

*   MySQL:

![](https://pic3.zhimg.com/v2-4f2d5f5226e0beda02ddde56be3b042e_b.jpg)

*   TIDB:

![](https://pic3.zhimg.com/v2-0b6b675832726a15d4e3bda5f768eb1e_b.jpg)

*   对比 3（set session transaction isolation level repeatable read;）  
    

*   MySQL：

![](https://pic3.zhimg.com/v2-a9240c37bfa2688f232898f32bf9d052_b.jpg)

*   TIDB：

![](https://pic1.zhimg.com/v2-73bd8bd36c0216b013b6544580dc46ac_b.jpg)

*   对比 4（set session transaction isolation level repeatable read;）  
    

*   MySQL:

![](https://pic1.zhimg.com/v2-c51c34b6a82085c68ec85e07a8b52c0c_b.jpg)

*   TIDB:

![](https://pic1.zhimg.com/v2-6e19bf44309a194b4722e273bddb228c_b.jpg)

*   对比 5（set session transaction isolation level repeatable read;）  
    

*   MySQL:

![](https://pic4.zhimg.com/v2-50e8328e2fcacf7a7827343fc0343943_b.jpg)

*   TIDB:

4.4Serializable
---------------

*   串性化是隔离最严格的一种形式，要求有读写冲突的事务必须严格串行执行。如下图所示，2 号事务要读取 1 号事务修改的记录 A，这就导致 2 号事务必须等待 1 号事务提交之后才能开启执行。通过这种形式可以避免之前所提到脏读、不可重复读和幻读。虽说如此，几乎所有数据库业务都不会开启这种隔离级别，因为这会带来严重的锁冲突。
*   可序列化的数据库锁情况  
    

*   事务在读取数据时，必须先对其加 表级共享锁 ，直到事务结束才释放
*   事务在更新数据时，必须先对其加 表级排他锁 ，直到事务结束才释放。

![](https://pic4.zhimg.com/v2-3adecfd215dc77a51e43a4eb0d63a163_b.jpg)

参考文献
----

*   [TiDB 事务隔离级别](https://link.zhihu.com/?target=https%3A//www.pingcap.com/docs-cn/sql/transaction-isolation/)