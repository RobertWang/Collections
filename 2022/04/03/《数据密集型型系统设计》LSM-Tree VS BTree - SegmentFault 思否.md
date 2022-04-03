> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [segmentfault.com](https://segmentfault.com/a/1190000041652003)

> 本文为《数据密集型应用系统设计》的读书笔记第一部分第三章的笔记整理，也是个人认为的这本书第一部分最重要的内容。

引言
==

本文为《数据密集型应用系统设计》的读书笔记第一部分第三章的笔记整理，也是个人认为的这本书第一部分最重要的内容。本文将会针对目前数据库系统两个主要阵营进行展开，分别是采用日志型存储结构高速读写的 LSM-Tree 和面向 OLTP 的事务数据库 BTree 两种数据结构对比。

主要内容
====

本文的主要内容介绍如下：

*   最简单 key/value 数据库考量和拓展，从零开始了解日志型存储结构。
*   索引对于数据库的重要性，哈希索引如何优化 key/value 数据库。
*   Btree 数据结构的简单介绍，数据结构和特点等。
*   LSM-Tree 日志存储结构 VS Btree 存储引擎，两大阵营的优劣对比。
*   OLTP 到 OLAP 的对比，行存储到列式存储结构演进，以及数据仓库出现和数据分析的演进。

数据库底层设计考量
=========

*   底层存储形式：记录存储的基本设计格式，虽然格式会有不同的设计，但是最终都是以文件的形式存储。
*   查询 / 新增 / 读取 / 修改方式：在外部来看也就是在数据库概念中的 DML 操作，这种操作面对的是客户端，属于对外接口的部分。而从内部来看则是存储数据结构，操作数据结构和并发性能的考量。
*   操作日志：出于持久性的考量，操作日志是不可或缺的因素，也是意外之后数据修复的保证。
*   数据类型：比如 Key/value 存储还是针对行列的存储结构。

key/value 存储结构和哈希索引
-------------------

关键：#key/value 存储结构处理 #哈希索引优化

从零开始设计一个数据库的存储形式，可以从下面的几个点考虑，从存储结构出发我们看日志存储结构数据库是如何出现的。

首先是 key/value 数据库数据结构设计第一版，从最简单的 k/v 存储数据库开始了解由此引申出哈希索引的结构：

![](https://segmentfault.com/img/remote/1460000041652005)

上面结构有如下的特点：

**存储形式**：主要以 key/value 存储形式，key/value 可以是任何内容。底层使用**纯文本**的形式存储，使用追加方式更新数据，相同 key 使用最后一次读到的 key 为基准。

**读写方式**：`db_set xx`设置数据，`db_get xx`读取数据，修改一个 key 通过最后一行追加形式，意味着更新和删除操作没有任何的开销，无需关注并发的问题。

**查询性能**：查找数据的开销为 O(n)，新增和修改的性能都是 O(1)。

> 追加式处理优点
> 
> *   顺序写比随机写好很多
> *   段文件是追加不可变的，意味着并发访问和崩溃恢复比较容易
> *   压缩和合并分段可以防止数据文件碎片化问题

最简单的 k/v 形式的数据库形成有哪些缺点？

*   追加对于存储空间的浪费，虽然追加对于更新和新增十分方面并且维护成本较低，但是有个明显的问题是存储空间的浪费，同时我们发现其实并不需要存储原始文本的形式，同时数据本身可以通过压缩更加紧凑。
*   查询效率随着数据的膨胀而降低，所以需要对于查询速度进行优化，对于查询最简单的方式是引入索引，而对于 key/value 存储形式设计索引最为常见的是哈希索引

对于 Key/value 存储引擎来说哈希是常用的索引类型，哈希索引使用内存中的哈希表进行实现，键值对的键存储数据需要索引的数值，而值存储偏移量，偏移量通过计算获取存储位置，在原始数据中直接找到相关位置的数据直接读取。

下面是哈希索引对于 key/value 结构数据进行索引优化。

![](https://segmentfault.com/img/remote/1460000041652006)

纯文本存储数据膨胀如何防止性能变差？

*   分段数据：当追加到一定程度之后则写入一个新的文件。
*   压缩段：将最新的数据进行压缩存储，由于使用追加新增方式，可以直接丢弃旧数据。
*   段压缩和多段合并：压缩与合并的过程没有特定规划，取决于数据结构和存储结构的抉择。

如何防止性能变差：

哈希表和段进行绑定，一个段对应一个哈希表，同时执行段压缩和多端合并，保证脏数据及时清理，最后一定在内存中引入哈希表进行维护。

了解了大致思路之后，如何进行具体优化？

1.  压缩合并存储：为了让存储数据更加紧凑同时没有浪费，定期对于追加数据进行合并压缩是必要的，同时数据分段也可以提高线程并发读写性能。
2.  数据分段：注意数据分段是结合压缩合并一同处理的，当压缩合并存储的数据到达一定量的时候需要对于数据进行分段处理，目的是防止单文件过大同时可以提高索引的搜索效率。
3.  哈希表：引入哈希表结构，在数据行上加一层索引目录，可以加快查询性能，索引的 key 存储的是键需要保证唯一，而 value 则存储了**行记录的指针**，这适用于分段的数据结构找到数据存储的位置，通过一次遍历分段直接通过偏移指针查指定数据是否符合要求即可。

上面讨论的存储结构其实存在实际的实现方式，此数据库存储引擎便是：**Bitcast**。

**哈希索引**：

哈希索引设计特点

*   文件格式：并没有使用纯文本存储而是使用二进制，使用字节来记录字符串长度，后面存储真实数据。（二进制也有利于压缩）
    
    *   崩溃恢复：最大的问题是重启之后**哈希表会被释放**，如果需要重新建立哈希表需要重新读取段，所以最大的性能开销在扫描段上，有一种优化方式是将哈希表的快照存储磁盘上直接读取。
    *   部分写入记录：使用 SHA 盐值和操作日志对于部分写入记录进行恢复，操作日志相对于数据存储日志更为简单，只需要做增量操作，因为目的仅仅用于存储引擎崩溃之后可以保证数据的持久性。
    *   并发控制：Bitcast 实际上是一个写线程和多个读线程，数据使用追加方式可以保证多个线程读取，但是只能保证一个线程写入，但是由于数据分段，可以多线程同时改写不同数据段。

通过上面哈希结构的介绍，我们可以总结出**哈希索引**的几个特点：

*   哈希索引适用于**查询多，增删少**的情况，虽然压缩和分段合并可以解决数据存储效率低的问题，但是对于频繁的增删需要额外的开销重新维护改动哈希表。
*   哈希表**需要在内存**中进行使用，所以受限于内存的大小，当然并不是说磁盘无法存储哈希表，而是哈希表在磁盘中难以维护和存储。

哈希的索引形式也存在**局限性**：

*   虽然哈希表不一定必须放入内存，理论可以在磁盘上维护哈希表，但是这样做需要大量的 IO，同时哈希冲突需要更复杂的处理逻辑。
*   区间查询效率不高，对于范围查询的处理能力较弱，此时时间复杂度会退化为 O(n)。

以上是哈希结构对于 key/value 存储的结构的优化。

哈希索引通常的适用场景：

*   点击数：对于数据的准确性要求并不是十分高，但是对于效率要求十分高
*   少量数据的唯一记录查找，注意是少量数据，因为哈希表空间有限。

**小结：**

我们可以看到，从一个最简单的 k/v 数据库到哈希索引的结构引入，数据的存储和读取结构逐渐变复杂，可以看到哈希索引非常适合 key/value 的存储引擎，但是显然它存在比较明显的缺陷，比如只能维护哈希表到内存，并且频繁的更新插入 key/value 对于索引的维护开销也不小，最后最为致命的是范围查询对于哈希索引是硬伤。

> 总结：索引特点
> 
> *   加快原始数据的查询速度
> *   空间换时间，需要更多的存储空间以及更长写数据时间
> *   索引很多时候被视为和原始数据分开的结构

通过上面的设计有缺点，针对哈希索引有了第一层进化，那就是 SSTable 和 LSM-Tree。

SSTable 和 LSM-Tree 结构
=====================

SSTable #LSM-Tree
=================

概念
--

在具体的内容介绍之前，我们需要弄清楚 SSTable 和 LSM-Tree 有什么区别，简单来说 LSM-Tree 是对 SSTable 概念和思想的一个继承。另外需要注意这个结构特点正好解决了**哈希索引局限性**问题，同时 SSTable 并没有抛弃 key/value 这样的存储结构，而是在索引结构上进一步升级。

SSTable 概念
----------

SSTable 起源自谷歌在 2006 年发布的一篇轰动世界的论文，里面的 BigTable 就是 SSTable 和 LSM-Tree 的前身：**Bigtable: A Distributed Storage System for Structured Data**。如果觉得论文难以理解，可以参考这篇文章理解：

[https://blog.csdn.net/OpenNai...](https://link.segmentfault.com/?enc=K8ndNFoIYn4FUIqJUidp0w%3D%3D.GrTxf8KAvsna9biE6OUmu3yVgK5X5SeKU05fbFqLzsHwjpVQYMoQeM0pq3Kynp0zH17TQakE%2BJFJk4zQb3oNvw%3D%3D)

这里挑了其中一些和 SSTable 有关的内容，可惜的是这篇论文并没有详细的介绍 SSTable 的内部数据结构，在论文第六个小节中介绍了 SSTable 的作用：

![](https://segmentfault.com/img/remote/1460000041652007)

BigTable 和 GFS 是什么关系呢？在论文中我们可以看到一个类似树的结构，其中根节点为主服务器，主服务器负责接受请求，通过管理分片服务器将请求分片到不同的片服务器中，所以从外层看最终干活的是片服务器，实际上片服务器本身只是负责管理自己分片 SSTable，通过特殊索引知道数据在那个 SSTable 分片中，然后从 GFS 中读取 SSTable 文件的数据，而 GFS 则可能要从多个 chuncker server 里面搜索数据。

而图中的 metatable 原数据表可以看作是和 SSTable 绑定的类似索引的关系，元数据表的数据是不能被外界访问的，外界访问的是元数据对应的 SSTable 分片，这和后面介绍的 LSM-Tree 有着十分熟悉的契合关系。

![](https://segmentfault.com/img/remote/1460000041652008)

改进与对比
-----

关键点： 数据存储方式，索引查找方式改进

### SSTable 通常如何工作？

*   写入的时候不写入磁盘而是先写入内存表的数据结构，同时在内存将数据进行排序。
*   当数据结构内存占用超过一定的阈值就可以直接写入到磁盘文件由于已经是排好序的状态，所以可以对于旧结构覆盖，写入效率比较高。并且写入和数据结构改动可以同时进行。
*   读写顺序按照 内存 -> 磁盘 -> 上一次写入文件 -> 未找到这样的顺序进行查找和搜索。
*   后台定时线程定时合并和压缩排序分段，将废弃值给覆盖或者丢弃。

### SStable 的改进点

下面是 SSTable 相对于哈希结构的特点：

*   高效合并：合并段的过程更加高效，每一个段都是按照特定顺序排序，当出现多个重复数值的时候可以合并到最新的段，对于旧数据则可以直接舍弃前面的内容。
*   **范围索引优化**：内存中哈希表也是有序存储，可以将多个 kv 对应的数据条目一同压缩存储，这样索引条目只需要开头部分的键值即可，因为后续所有的记录都是有序的。
*   索引优化：比起哈希较为松散随意的结构，这样的处理牺牲一定的读写开销换来更加高效的存储和索引结构，并且可以支持范围索引扫描。
*   **顺序读写**：数据顺序存储的好处是可以顺序读写，避免磁盘的随机读写可以大幅度提升读写性能

下面是改进过后的压缩过程图：

最大的改动点：压缩合并的基础上对于 SSTable 基础内容进行合并操作。

![](https://segmentfault.com/img/remote/1460000041652009)

> 如何维护 sstable?  
> 首先是数据如何在内存中排序，可以使用红黑树和 AVL 树的结构也可以是任意结构，重点是在内存中完成数据压缩合并和排序的操作。
> 
> 为什么数据集远远大于内存依然可以高效？  
> 因为使用排序以及分段合并压缩分段的数据，所以一次加在到内存的数据不需要太多，其实只需要把内存索引表查找某个区间段的数据，然后进行顺序查找，由于是按照排序的方式顺序存储的，在段上查找数据集通常可以根据键直接偏移或者按照特定规则二分查找的方式搜索。

### SSTable 和哈希索引的对比：

可以看到 SSTable 在哈希索引的基础上进行优化，使用排序的手段虽然会损耗一定的写入消耗，但是换来的是更高效率的范围查询以及更高效率的压缩存储形式。

哈希索引：

*   索引查询效率十分高
*   内存中维护，磁盘 IO 开销很小
*   非常适用于 Key 频繁更新的场景

SSTable：

*   利于磁盘维护索引和顺序读写，
*   优化范围查询。
*   将范围搜索的查询效率优化至 O(logN) 的水平

实际案例和应用
-------

全文索引：

全文索引虽然比 key/value 复杂很多，但是本质都是类似的，某些数据维护依然基于 key/value 方式存储，比如词条的映射关系使用的 SS Table 进行存储，在内存中排好序存储，并且需要的时候进行合并。

LevelDB 和 RockDB：

LevelDB 和 RockDB 是最为典型的 LSM-Tree 实践案例，尤其是 LSM-Tree 作者刚好又是谷歌的工程师，深入了解这两款经典 LSM-Tree 实现案例可以对于 SSTable 的设计和应用有更深的了解。

> LSM-Tree 的代码非常简单易懂，难懂的地方作者也给了注释，对于我这种 JAVA 开发者也能了解大概。

SSTable 的弊端
-----------

*   最大的隐患是在压缩合并分段的时候不能进行数据的读写，否则数据一致性会存在问题，这对于吞吐量要求高的系统很难接受。
*   压缩另一个问题是对于带宽的占用非常高，压缩数据量越大，带宽消耗越高，容易阻塞大量的读写请求。
*   如果压缩速度跟不上写入数据，那么就会出现大量未压缩数据堆积情况，长期累计容易造成磁盘空间不足，但是 sstable 的数据结构决定了追加写入是不受控制的，需要外部力量监控。

Btree 存储结构
==========

简介
--

Btree 数据结构自 1970 年被提出之后，被广大的数据库使用者接受 BTree 目前不太可能被新技术淘汰，而日志索引结构的 Lsm-Tree 未来前景十分可观。

Btree 特点
--------

和 SSTable 仅仅只有一点类似：B-tree 保留按键排序的 key/value 对，这样可以实现高效的 key-value 查找和区间查询，除此之外没有其他的相似点。

下面是 Btree 的特点：

*   和日志存储结构不同的是，Btree 将数据分为固定的数据页，每一个数据页固定大小，通常为 4KB，InnoDB 一个数据页固定设置为 16KB。
*   更新数据在原结构上进行更新，也就是使用新数据页覆盖旧的数据页，所以更新的代价相对于日志结构要大很多的。
*   数据页存在和磁盘对应的唯一标识，同时数据页之间通过链表指针串联，但是指针存储的不是内存上的地址，而是磁盘的地址，
*   新增数据如果数据页内容不足需要进行页分裂，同时父节点需要包含新分裂的页，而页分裂很容易造成磁盘碎片和数据的排序，同时因为内部需要维持数据排序开销不小。

> 为什么数据页要固定大小？  
> 这和磁盘的读写有关，传统机械硬盘为 512b 最小单位，而 SSD 通常为 4KB 最小单位读写，所以数据页设计为和磁盘兼容的形式存储有利于**顺序读写**，同时可以最大化利用磁盘空间。

![](https://segmentfault.com/img/remote/1460000041652010)

Btree 优化和可改进点
-------------

*   压缩存储：使用缩略的键信息，因为对于树中间一个数据行来说只要提供开始和结束的位置即可，这样可以在有限的数据页存储更多的数据。
*   数据页排序：虽然数据页可以存储在不同的磁盘空间，但是对于某些需要范围查询的情况下需要对于磁盘进行顺序查找，而此时数据页的随机查询就会出现问题
*   BTree 到 B + 树的优化，优化的方式也十分简单易懂，在各层都加链表加快索引查找，这种甚至不能叫优化只能说是改进。
*   使用**写时复制**替代覆盖页的方式，在修改页的时候不对原数据改动，将写入新的父页并且更新旧引用。

Btree 和 LSM-Tree 对比
-------------------

Btree 的读写速度快于 LSM-Tree，因为一次 IO 读写可以索引到大量的数据页，而 LSM-Tree 需要跨越多个压缩段才可能找到数据。但是 LSM-Tree 的读写速度要快于 Btree，同时存储效率要比 Btree 要高，因为压缩和合并分段之后数据间隙之间基本不存在数据间隙碎片。

所以 LSM-Tree 适用于读写多的场景，而 Btree 因为需要高效查询设计上要复杂非常多所以为了服务查询性能可以容忍写入和删除的额外开销。

单纯对比数据结构可能比较枯燥，这里从老外的网站上找了一份 **Mysql** 和 **LevelDB** 的对比，这两款基本可以代言 Btree 和 LSM-Tree 这两种数据结构了。可以看到 LevelDB 要比 Mysql 的出现晚上不少，这和 BigTable 有着密切关系，也和网络时代的发展和互联网的用户量指数上升有关系。

<table><thead><tr><th>Name</th><th>LevelDB <a href="https://link.segmentfault.com/?enc=SAuYxcjDm%2FHoYXPjoPkQIw%3D%3D.pu768f1whdJNwe9dg5w9F%2BjTAN%2Bm9IV8CL6Sh%2FVvduMCpMZc%2FkYZZxIq%2BPDdWaYE" rel="nofollow" target="_blank">X</a></th><th>MySQL <a href="https://link.segmentfault.com/?enc=i8s8rVHfO%2FgW3nDI4NKxcw%3D%3D.DZQWRKj%2BHPVd0RIiK4n7X1U3paJvOfROTD6RdP499kFMfVgYSp9MtQ5z0Yo6BoRD" rel="nofollow" target="_blank">X</a></th></tr></thead><tbody><tr><td>Description</td><td>Embeddable fast <a href="https://link.segmentfault.com/?enc=71isKejVh%2FBhz1RmSIIxqQ%3D%3D.CP6iVONZ7ERwZE7UJmVbeL9CiSKLBm6hJaRLdxbY%2FA3o3dzQpEpKG3V5Bfnut7ikTpV0vZBNEJjaxZ%2FVH9rhzw%3D%3D" rel="nofollow" target="_blank">key-value storage</a> library that provides an ordered mapping from string keys to string values</td><td>Widely used open source <a href="https://link.segmentfault.com/?enc=wMDY4OTktFAStdSp%2FaytBQ%3D%3D.IxaWWVay1C0%2FV1hR5oru3Xbm1cHXZVrATvQ1fYW%2BB%2F8ke9jT5bB67GAl0vxS%2BhvR" rel="nofollow" target="_blank">RDBMS</a></td></tr><tr><td>Primary database model</td><td><a href="https://link.segmentfault.com/?enc=aBM4dtu9MPmIrlc3GN0zFw%3D%3D.wfzpTZClGZKuQgxlWWgfi7m%2B0NrxQ%2FbqVWQb2PN59uR09SumTOy1VbStGQlh8%2BN62lWOWoCO5nLiPkNWaECigQ%3D%3D" rel="nofollow" target="_blank">Key-value store</a></td><td><a href="https://link.segmentfault.com/?enc=jc6OrEnn1207J%2BX69likNg%3D%3D.emeIibhRlJTyhAgXoHbUFtbwE6keXXG7XJWcrYZRiz1z0tJISC4KhVxJYRBDb5uy" rel="nofollow" target="_blank">Relational DBMS</a> <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td></tr><tr><td>Secondary database models</td><td></td><td><a href="https://link.segmentfault.com/?enc=fxmOPY91Yf6ITmOwnmViDQ%3D%3D.X6EMT1u0hpMBh5naTvhf6s6zptRfrxXTf21uSValhuS8BLJ6abofDLt5XCFueOCIHhCaFW5%2FDvGoGlAlosCqvA%3D%3D" rel="nofollow" target="_blank">Document store</a> <a href="https://link.segmentfault.com/?enc=Wf1ZXm5M5TJ9s2EScNFqRg%3D%3D.4IP44WSRBU2rIWHbrQAUGqtRfUfGjMF%2Fk1ySQc08IVEcg7hu9eADH%2BXi04QJfQvE" rel="nofollow" target="_blank">Spatial DBMS</a></td></tr><tr><td><a href="https://link.segmentfault.com/?enc=BAqAgYFLjCN%2B9Nkb4DFU8A%3D%3D.7kk%2FHHbJDwQLPVbT6ubJCh6WKZZUPlXHePFNE%2FisdFacr8QMOsOHp1EdxkeuFiGq" rel="nofollow" target="_blank">DB-Engines Ranking</a> <img class="" src="https://segmentfault.com/img/remote/1460000041652011"><a href="https://link.segmentfault.com/?enc=uQh443EGeZiV6B3gVaRhmQ%3D%3D.KE4LxRvgkpJ%2FaIF2dNw%2BoKW%2BZOBq%2FeVzap856Yp0XX1b%2F%2BdSYmdOpRC12MMc9HXPyz428N0JZFvmo%2BHAbj1PYw%3D%3D" rel="nofollow" target="_blank"><img class="" src="https://segmentfault.com/img/remote/1460000041652012"></a><a href="https://link.segmentfault.com/?enc=6HHTIrr%2B8KKedfX9avOhsg%3D%3D.TiyVbxTYvtDM1eh%2Fk7ju13pRlf%2Bk9SNCVI1IDfJ1RdmPR1Py2I9uSa%2BxJZdTiFEyNYGgXB4v7ybka8C6%2BCjEWg%3D%3D" rel="nofollow" target="_blank">Trend Chart</a></td><td>Score3.19Rank#97 <a href="https://link.segmentfault.com/?enc=s1OFH7p2LeVvzW%2FZc2JgTg%3D%3D.cGW4BUOcv6G%2BRjBlOnSuUXRdHZmyCosCzeHYntpJFedi6e5HaGQuGCF9pFk7TFed" rel="nofollow" target="_blank">Overall</a>#18 <a href="https://link.segmentfault.com/?enc=g4OXabDP7YLayrr%2BMlKpcQ%3D%3D.4w%2BHhgZNXdam0YboRGulwm0p9hmwV1BWRl7NLvSKCx4Gw89wcmRkp59EzOaQZAKmqOvV9Kv5B3ctb%2FwC04lwuA%3D%3D" rel="nofollow" target="_blank">Key-value stores</a></td><td>Score1204.16Rank#2 <a href="https://link.segmentfault.com/?enc=JKQ0eqHHHvV03vGzXReqMg%3D%3D.9lUeYClxNV%2F6KYOg0cf9U5NDq4xtBvmxv%2BnMyry4aK2h%2F6YAo2Af9rDf%2F%2Byh%2FNbX" rel="nofollow" target="_blank">Overall</a>#2 <a href="https://link.segmentfault.com/?enc=JDOOHV6YBWOEtD%2Fqzpn0XA%3D%3D.yAq8cTKiMOvAldFtXKmYc7E7YuZcPTDR1gaFODqf4%2B0Zf3min%2BdU9tvemgM5wRN9MRkHTrXkcjatVQD6br70dg%3D%3D" rel="nofollow" target="_blank">Relational DBMS</a></td></tr><tr><td>Website</td><td><a href="https://link.segmentfault.com/?enc=%2F9Tf8VxO%2FKad9f3jdW7h9g%3D%3D.8VNYUE7qAeOEPUjDr%2BCk7xq%2B8Rt8Ls4b6SKwbXTHYyMTYe%2FfOuhSbRXOilwf0KKM" rel="nofollow" target="_blank">github.com/­google/­leveldb</a></td><td><a href="https://link.segmentfault.com/?enc=IyZBlGTi8nlTrdiEA7VemA%3D%3D.7avUUaaN%2Fav7LpjTCzftrdmibI7pUcsIYpm%2FTOUp2BU%3D" rel="nofollow" target="_blank">www.mysql.com</a></td></tr><tr><td>Technical documentation</td><td><a href="https://link.segmentfault.com/?enc=%2B80%2Fe8ul1U1V3rmv7zPGbg%3D%3D.uPduaLnecgE8e8IJMR3LVCHaD9BRhonGKx6BxYSXMJhEz21o4h2lvTWSVVdsBfbkyyjg9zeoenA5wxom3dtzWA%3D%3D" rel="nofollow" target="_blank">github.com/­google/­leveldb/­blob/­master/­doc/­index.md</a></td><td><a href="https://link.segmentfault.com/?enc=GIY%2FiWA1yv6VCrCokcs5pQ%3D%3D.a3m4bXTZWEQ%2BUcVisrt0F%2F1vZv3ycZiLGN5Pvh%2F8Cn4%3D" rel="nofollow" target="_blank">dev.mysql.com/­doc</a></td></tr><tr><td>Developer</td><td>Google</td><td>Oracle <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td></tr><tr><td>Initial release</td><td>2011</td><td>1995</td></tr><tr><td>Current release</td><td>1.23, February 2021</td><td>8.0.28, January 2022</td></tr><tr><td>License <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td><td>Open Source <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td><td>Open Source <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td></tr><tr><td>Cloud-based only <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td><td>no</td><td>no</td></tr><tr><td>DBaaS offerings (sponsored links) <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td><td></td><td><a href="https://link.segmentfault.com/?enc=rHqsO%2BWPWVm8Xi5WlK1YqA%3D%3D.7U8ibg7Fh%2B0R8HRm8WuM2amaQY5qxrS1vhL9ztBDMSlV14%2B%2Bc9Gh1hcfaF9386IYu%2BJhorGXUQRZfkapO02vtMZZXYzRXlmTLtr%2BbDx07uPWg3KHy5pSz%2FhBfre4HWf%2Bg%2BViGhNPuE0SsO%2BLQGEQ7m5KuO6f4Fmyy5zpra9blVtbzFDzwhpfdeyvw4nOzAKIpYkqSYum%2B%2FdldYCZlNwbqm%2BAidcz%2BO8xAsBVSk4ZuVRLbbcFG0aErTHYAaAtLNXJWKs7ya%2BelU43fXs%2FUFM6%2BS%2FBkYD93g8plir5HHZOJ1s%3D" rel="nofollow" target="_blank">ScaleGrid for MySQL</a>: Fully managed MySQL hosting on AWS, Azure and DigitalOcean with high availability and SSH access on the #1 multi-cloud DBaaS.</td></tr><tr><td>Implementation language</td><td>C++</td><td>C and C++</td></tr><tr><td>Server operating systems</td><td>Illumos Linux OS X Windows</td><td>FreeBSD Linux OS X Solaris Windows</td></tr><tr><td>Data scheme</td><td>schema-free</td><td>yes</td></tr><tr><td>Typing <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td><td>no</td><td>yes</td></tr><tr><td>XML support <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td><td>no</td><td>yes</td></tr><tr><td>Secondary indexes</td><td>no</td><td>yes</td></tr><tr><td>SQL <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td><td>no</td><td>yes <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td></tr><tr><td>APIs and other access methods</td><td></td><td>ADO.NET JDBC ODBC Proprietary native API</td></tr><tr><td>Supported programming languages</td><td>C++ Go Java <img class="" src="https://segmentfault.com/img/remote/1460000041652011"> JavaScript (Node.js) <img class="" src="https://segmentfault.com/img/remote/1460000041652011"> Python <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td><td>Ada C C# C++ D Delphi Eiffel Erlang Haskell Java JavaScript (Node.js) Objective-C OCaml Perl PHP Python Ruby Scheme Tcl</td></tr><tr><td>Server-side scripts <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td><td>no</td><td>yes <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td></tr><tr><td>Triggers</td><td>no</td><td>yes</td></tr><tr><td>Partitioning methods <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td><td>none</td><td>horizontal partitioning, sharding with MySQL Cluster or MySQL Fabric</td></tr><tr><td>Replication methods <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td><td>none</td><td>Multi-source replication Source-replica replication</td></tr><tr><td>MapReduce <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td><td>no</td><td>no</td></tr><tr><td>Consistency concepts <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td><td>Immediate Consistency</td><td>Immediate Consistency</td></tr><tr><td>Foreign keys <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td><td>no</td><td>yes <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td></tr><tr><td>Transaction concepts <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td><td>no</td><td>ACID <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td></tr><tr><td>Concurrency <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td><td>yes</td><td>yes <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td></tr><tr><td>Durability <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td><td>yes <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td><td>yes</td></tr><tr><td>In-memory capabilities <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td><td></td><td>yes</td></tr><tr><td>User concepts <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td><td>no</td><td>Users with fine-grained authorization concept <img class="" src="https://segmentfault.com/img/remote/1460000041652011"></td></tr></tbody></table>

OLTP 和 OLAP
===========

特征
--

单从名字的区分，这两个类型分别针对**事务**和针对**分析**，由此引发了不同的完全不同的数据结构和存储形式，同时侧重点和服务范围不同，注意这两者并不能说谁优势谁劣势，因为 OLAP 说白了是在 OLTP 上演化而出现的。

![](https://segmentfault.com/img/remote/1460000041652013)  
如果看不清，可以看看甲骨文提供的一个表格：  
![](https://segmentfault.com/img/remote/1460000041652014)

OLTP：在线**事务**处理。用于处理数据量较小的事务为主的业务，要求的是高吞吐高响应，一般为 web 应用程序，主要面向广大用户群体，适用于解决生活中 80% 左右的业务和系统问题。

OLAP：在线**分析**处理。主要基于大数据量的数据搜索和汇总，随着时间变化而进行数据分析进行数据支撑。一般出现在中大型公司的较大型项目中。

数据仓库
====

事务型号数据库这里不再赘述了，相信大家也很熟悉，这里说说数据仓库是什么？

数据仓库是上世纪 90 年代被提出的放弃基于传统事务 OLTP 结构的数据库分析，转为使用单个数据仓库进行数据分析的方式，简单理解就是讲数据和业务剥离，只保留数据部分进行存储，通常为了方便分析会使用 [[《数据密集型系统设计》列式存储]] 引擎和前面提到的 [[《数据密集型型系统设计》SSTable 和 LSM-Tree]] 数据结构。

优势
--

数据仓库有下面的优势。

*   面向主题：数据仓库可以高效分析关于特定主题或职能领域（例如销售）的数据。
*   集成：数据仓库可在不同来源的不同数据类型之间建立一致性。
*   相对稳定：进入数据仓库后，数据将保持稳定，不会发生改变。
*   反映历史变化：数据仓库分析着眼于反映历史变化。

存储形式
----

数据仓库一般不存在于小公司，因为根本没有那个资金支撑也没有，而是面对较大公司多达 10 几个系统数据规模的存在形式，所以对于很多人来说可能就是围观看造火箭，看懂了好像也没啥价值。

结构图

下面是整个数仓系统的结构

进化
--

数据仓库到了现在又出现了进一步的转变，大数据也被分为了很多个方向，这些都是战未来的东西，这里简单提炼一下概念：

*   数据挖掘
*   预测和统计
*   人工智能和机器学习

数据湖
---

注意数据库并不是指比数据仓库大很多倍的数据仓库或者数据库集群，他是完全不同的概念，数据仓库是已经被整理  
数据湖主要用于存储大量迥然不同并且没有进行分类筛选的数据，数据湖对于分析人员来说是最为自由的一种，可以形象看作垃圾池掏金子，收益和代价并存，更多时候是把数据湖转为数据仓库。

应用
--

*   数据库 比较流行的有：MySQL, Oracle, SqlServer 等。
*   数据仓库 比较流行的有：AWS Redshift, Greenplum, Hive 等。

分析模式
----

对于数据仓库的分析模型，现今通常有两种方式：星型模型和雪花模型。

### 星型模型

星型模型也别叫做纬度模型，这个模型针对的是非强依赖的数据关系分析，比较典型的如电商系统和购物网站，虽然商品，订单，库存，广告等等看似没有什么关系，然而这些数据就像是星星一样分散，有着千丝万缕的直接依赖或者间接依赖关系。  
星型模型的数据更像是星星和月亮的关系，核心通常位于中间， 而四周散布关系模型。

### 雪花模型

雪花模型要比星型模型设计上规范很多，也是对于维度模型的进一步拆分，比如对于商品表引入更加细分的品牌和分类进行更进一步的细分，也类似超市的商品分类。

**小结**

这一节点从 OLAP 讲到数据仓库，再讲到两种主流的分析方式，星型模型和雪花模型。  
当然这些内容都是简单介绍，读者可以根据自己感兴趣的点进行深入和熟悉。

列式存储
====

介绍
--

对于传统的业务数据库并且用户量较小的公司来说，通常使用关系型数据库，而关系型数据库基本以 Btree 数据结构作为核心，这些数据库基本都是行存储，行存储比较符合理解思维，然而实际上对于数据分析而言，行数据往往会造成长时间的时间等待，并且如果需要对于数据进行实施分析作为业务决策，使用行存储分析显然系统开销是无法接受的，由此引入了 [[OLTP 和 OLAP]]。

之前提到的内容都是行式存储，有些情况下列存储更利于数据管理，特别是对于事务关系数据库行存储结构不利于单一列分析，比如我们想要存储某一分类的价格趋势，行存储需要扫描所有行求和，而列存储直接把一整列拿出来求和即可，两者效率自然不用多说。

列存储结构特点
-------

*   所有列值紧凑存储在一起
*   通常比行存储更有利于理解
*   可用于非关系型数据库，当然也可以用于关系型数据库。（反过来行存储就不适用这条规则了）

列压缩
---

列式存储意味着把同类数据压缩到不同的段，这对于存储结构来说是有利的，通常列压缩在数据仓库使用位图的形式存储。

因为列存储的都是同一个数据类型的数据，所以可以针对不同的数据类型进行优化存储，这样也意味着比行存储更少的空间压缩更多的数据，比如对于一些整型数据在压缩存储的时候可以使用位模式存储。也就是直接存数字的二进制位编码，列查询的时候进行位操作即可。

下面是一个典型的列存储和压缩的例子：  
![](https://segmentfault.com/img/remote/1460000041652015)

存储位图可能人不是很好理解和计算，但是计算器来说就不一样了，位元算操作远远高于数学运算。

![](https://segmentfault.com/img/remote/1460000041652016)

另外除开列压缩以外，列的存储还以一个**列族**的概念，列族存在于 Cassandra 和 HBase 这两个数据库，而列族这个概念继承自 BigTable。  
但是我们之前介绍 [[《数据密集型型系统设计》SSTable 和 LSM-Tree]] 讲述基本还是行存储方式和实现。

列族：其实指的是**把一行中的所有列和行主键保存到一起**，并且不使用列压缩的形式存储。其实这种用行转列基本就可以实现，所以列族严格意义上依然是行存储的变体，和真正的列存储还是存在差异的。

列排序
---

相对于行存储，列的排序其实并没有多重要，因为他不关心数据归属而是数据的整理，和 [[SSTable]] 一样，列排序最好的方式也是通过追加压缩合并的方式，然后利用索引和一些天然的有序数据结构完成存储。

注意列排序一般不会针对单列进行排序，因为没有多少意义，至于原因这里依然强调单列没有办法知道数据的归属。

列排序的第一个优势是可以对列的重复值进行压缩，比如性别通常只有男和女，这时候重复的列存储是没有必要的。

列排序的另一个优点是多列排序可以快速的定位某一列的极值情况和方便快速的分组或者过滤查询。

C-Store 最早引入了 一 种改进想蓓，井被商业数据仓库 Vertica。

最后，面相列存储通常会具备多个排序顺序，但是多列排序很难维护，所以更常见的情况是引入二级索引维护，和行存储的索引维护不同，行存储的二级索引通常指向数据行（或者像 InnoDB 一样指向主键，不过这种处理比较特殊），**列存储二级索引通常指向值**。

视图
--

列存储中一个十分重要的优化特性是引入试图对于临时查询进行加速，数据仓库中的 SQL 中经常会碰到聚合函数的查询通常会想到使用缓存进行存储。  
列存储的缓存被叫做标准视图，注意这个视图并不是逻辑临时结构，而是真实物理视图，列存储相比行存储对于物理存储的数据有效和可用性高很多，OLTP 系统不适用物理视图主要原因是缓存失效很快并且需要应对低延迟高响应的查询，而数据仓库由于主要做数据分析数据静态情况更多。  
视图的特殊情况叫做数据立方体，数据立方体的概念类似于列存储的 “行聚合” 和“列聚合”交叉查询的方式：  
![](https://segmentfault.com/img/remote/1460000041652017)

从上面的图可以看到，数据分为多个维度进行分析和查询，通过多维度角度可以把多个分类模型的数据放到一起进行交叉统计，这对于数据分析人员来说无疑省去大量的准备工作，将这种数据立方物化之后的查询效率也十分高。

物化数据毫无疑问让数据查询更快，因为数据已经预先准备，但是数据立方的缺点和缓存的缺点一样是不能频繁改动，否则失去其本身意义。

总结
==

主要的内容集中在 Key/value 的数据库发展和哈希索引，以及后续的 Btree 和 LSM-Tree 上，可以看到 BTree 出现最早但是经历了 30 多年依然长盛不衰，一时半会应该还没有更优秀的数据结构替代，而 LSM-Tree 则比较符合未来和现代的发展潮流，因为他存储形式更自由，并且非常适合用于列式存储和列压缩以及数据分析，总之这篇笔记更多的是让读者了解更广阔的数据视野。

写在最后
====

这篇除开书本的内容之外，个人对于其他内容做了一些补充，如果有不同的看法欢迎讨论，如果文中有错误欢迎批评指正。

参考资料：
=====

*   [https://blog.csdn.net/OpenNai...](https://link.segmentfault.com/?enc=OoMmG5ETm0jNpfvCNxlPhQ%3D%3D.LC7pPS%2FtWpP6RHZbA4ceJLlLcqvPa1VAlyd47WCvl56oM5crvfq%2BHCskaZvJ0m6DJjKkG2jrtfPSECkiZWDOYQ%3D%3D) BigTable 翻译和解读，讲的不错
*   [https://db-engines.com/en/sys...](https://link.segmentfault.com/?enc=mXsdD%2Bb3OFhT%2Bt7YP84QxQ%3D%3D.DyKop%2F86nvskOZGVftaxdSVEbweDwFh7fBcc89Su6jr6y1VKriWnpHqcirxT9AWjThZ8LHOfb3KHMUh6I4AG%2BA%3D%3D) 文中图表来源