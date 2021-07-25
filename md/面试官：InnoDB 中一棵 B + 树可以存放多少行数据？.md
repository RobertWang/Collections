> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU0MzQ5MDA0Mw==&mid=2247493716&idx=1&sn=f144fe0d9efa68a71b370d3684135342&chksm=fb0802c0cc7f8bd6f9f694446eedd688a0f2c527a563fc21a72f6ddbf139edc143354f3f7a6c&scene=21#wechat_redirect)

InnoDB 一棵 B + 树可以存放多少行数据？这个问题的简单回答是：约 2 千万。为什么是这么多呢？因为这是可以算出来的，要搞清楚这个问题，我们先从 InnoDB 索引数据结构、数据组织方式说起。

我们都知道计算机在存储数据的时候，有最小存储单元，这就好比我们今天进行现金的流通最小单位是一毛。在计算机中磁盘存储数据最小单元是扇区，一个扇区的大小是 512 字节，而文件系统（例如 XFS/EXT4）他的最小单元是块，一个块的大小是 4k，而对于我们的 InnoDB 存储引擎也有自己的最小储存单元——页（Page），一个页的大小是 16K。

下面几张图可以帮你理解最小存储单元：

文件系统中一个文件大小只有 1 个字节，但不得不占磁盘上 4KB 的空间。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbIS6a5cOLmibwSVlPfHdKknpZmHH5AlRHRgc0H2or0fx6kjicHTZxmia6Puro7Zjnuw83tqgOBIftaZA/640?wx_fmt=png)

innodb 的所有数据文件（后缀为 ibd 的文件），他的大小始终都是 16384（16k）的整数倍。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbIS6a5cOLmibwSVlPfHdKknpHrWVEqgLrRicxeVXJpVOQMmzQwaTbystUxntE1Pw4ia9QsibuJNeYxyibg/640?wx_fmt=png)

磁盘扇区、文件系统、InnoDB 存储引擎都有各自的最小存储单元。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbIS6a5cOLmibwSVlPfHdKknpCgWSlPQZJHjrBXH6W4KMgZFHKTj6g4sqXOtUkve3lREwoqxC4ClGCg/640?wx_fmt=png)

在 MySQL 中我们的 InnoDB 页的大小默认是 16k，当然也可以通过参数设置：

```
mysql> show variables like 'innodb_page_size';

+------------------+-------+

| Variable_name   | Value |

+------------------+-------+

| innodb_page_size | 16384 |

+------------------+-------+

1 row in set (0.00 sec)
```

数据表中的数据都是存储在页中的，所以一个页中能存储多少行数据呢？假设一行数据的大小是 1k，那么一个页可以存放 16 行这样的数据。

如果数据库只按这样的方式存储，那么如何查找数据就成为一个问题，因为我们不知道要查找的数据存在哪个页中，也不可能把所有的页遍历一遍，那样太慢了。所以人们想了一个办法，用 B + 树的方式组织这些数据。如图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbIS6a5cOLmibwSVlPfHdKknpFBWZBYJWnTBT5USFtsdC5a4baBVbxBHwWNkFQ8xuWYHCKtBKeLxVNQ/640?wx_fmt=png)

我们先将数据记录按主键进行排序，分别存放在不同的页中（为了便于理解我们这里一个页中只存放 3 条记录，实际情况可以存放很多），除了存放数据的页以外，还有存放键值 + 指针的页，如图中 page number=3 的页，该页存放键值和指向数据页的指针，这样的页由 N 个键值 + 指针组成。当然它也是排好序的。这样的数据组织形式，我们称为索引组织表。现在来看下，要查找一条数据，怎么查？

如`select * from user where id=5;`

这里 id 是主键, 我们通过这棵 B + 树来查找，首先找到根页，你怎么知道 user 表的根页在哪呢？其实每张表的根页位置在表空间文件中是固定的，即 page number=3 的页（这点我们下文还会进一步证明），找到根页后通过二分查找法，定位到 id=5 的数据应该在指针 P5 指向的页中，那么进一步去 page number=5 的页中查找，同样通过二分查询法即可找到 id=5 的记录：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbIS6a5cOLmibwSVlPfHdKknpEKqMM1ZKBg9Nj644AS3rl8Wuytrk4elt1ljeZGicWpKSoRl6h59t3Ig/640?wx_fmt=png)

现在我们清楚了 InnoDB 中主键索引 B + 树是如何组织数据、查询数据的，我们总结一下：

1.  InnoDB 存储引擎的最小存储单元是页，页可以用于存放数据也可以用于存放键值 + 指针，在 B + 树中叶子节点存放数据，非叶子节点存放键值 + 指针。
    
2.  索引组织表通过非叶子节点的二分查找法以及指针确定数据在哪个页中，进而在去数据页中查找到需要的数据；
    

那么回到我们开始的问题，通常一棵 B + 树可以存放多少行数据？

这里我们先假设 B + 树高为 2，即存在一个根节点和若干个叶子节点，那么这棵 B + 树的存放总记录数为：根节点指针数 * 单个叶子节点记录行数。

上文我们已经说明单个叶子节点（页）中的记录数 = 16K/1K=16。（这里假设一行记录的数据大小为 1k，实际上现在很多互联网业务数据记录大小通常就是 1K 左右）。

那么现在我们需要计算出非叶子节点能存放多少指针，其实这也很好算，我们假设主键 ID 为 bigint 类型，长度为 8 字节，而指针大小在 InnoDB 源码中设置为 6 字节，这样一共 14 字节，我们一个页中能存放多少这样的单元，其实就代表有多少指针，即 16384/14=1170。那么可以算出一棵高度为 2 的 B + 树，能存放 1170*16=18720 条这样的数据记录。

根据同样的原理我们可以算出一个高度为 3 的 B + 树可以存放：1170_1170_16=21902400 条这样的记录。所以在 InnoDB 中 B + 树高度一般为 1-3 层，它就能满足千万级的数据存储。在查找数据时一次页的查找代表一次 IO，所以通过主键索引查询通常只需要 1-3 次 IO 操作即可查找到数据。

### 怎么得到 InnoDB 主键索引 B + 树的高度？

上面我们通过推断得出 B + 树的高度通常是 1-3，下面我们从另外一个侧面证明这个结论。在 InnoDB 的表空间文件中，约定 page number 为 3 的代表主键索引的根页，而在根页偏移量为 64 的地方存放了该 B + 树的 page level。如果 page level 为 1，树高为 2，page level 为 2，则树高为 3。即 B + 树的高度 = page level+1；下面我们将从实际环境中尝试找到这个 page level。

在实际操作之前，你可以通过 InnoDB 元数据表确认主键索引根页的 page number 为 3，你也可以从《InnoDB 存储引擎》这本书中得到确认。

```
SELECT
b.name, a.name, index_id, type, a.space, a.PAGE_NO
FROM
information_schema.INNODB_SYS_INDEXES a,
information_schema.INNODB_SYS_TABLES b
WHERE
a.table_id = b.table_id AND a.space <> 0;
```

执行结果：

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/HV4yTI6PjbIS6a5cOLmibwSVlPfHdKknpVOPHVu6icNDD5eCTn8XFX6IBscDnL9NrV9Q37OogODTtLhEy560eXIQ/640?wx_fmt=jpeg)

可以看出数据库 dbt3 下的 customer 表、lineitem 表主键索引根页的 page number 均为 3，而其他的二级索引 page number 为 4。关于二级索引与主键索引的区别请参考 MySQL 相关书籍，本文不在此介绍。

下面我们对数据库表空间文件做相关的解析：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbIS6a5cOLmibwSVlPfHdKknpo0ESS7s0g0yemayicTQHKoXFB154fGxq2OMR5Vu3rc0TrMmlbiadCm0Q/640?wx_fmt=png)

因为主键索引 B + 树的根页在整个表空间文件中的第 3 个页开始，所以可以算出它在文件中的偏移量：16384*3=49152（16384 为页大小）。

另外根据《InnoDB 存储引擎》中描述在根页的 64 偏移量位置前 2 个字节，保存了 page level 的值，因此我们想要的 page level 的值在整个文件中的偏移量为：16384*3+64=49152+64=49216，前 2 个字节中。

接下来我们用 hexdump 工具，查看表空间文件指定偏移量上的数据：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbIS6a5cOLmibwSVlPfHdKknpBBpO1UB8xMGHPvIiaNAZQRLX9gb4kgKtd2OChukqNpzX40oeZLEq7mQ/640?wx_fmt=png)

linetem 表的 page level 为 2，B + 树高度为 page level+1=3；

region 表的 page level 为 0，B + 树高度为 page level+1=1；

customer 表的 page level 为 2，B + 树高度为 page level+1=3；

这三张表的数据量如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbIS6a5cOLmibwSVlPfHdKknpnSXzNicHEqoIoBVPMmibNvXAyJTdAuGEVZWT73XoZZxttB5EkmY25qsQ/640?wx_fmt=png)

总结：

lineitem 表的数据行数为 600 多万，B + 树高度为 3，customer 表数据行数只有 15 万，B + 树高度也为 3。可以看出尽管数据量差异较大，这两个表树的高度都是 3，换句话说这两个表通过索引查询效率并没有太大差异，因为都只需要做 3 次 IO。那么如果有一张表行数是一千万，那么他的 B + 树高度依旧是 3，查询效率仍然不会相差太大。

region 表只有 5 行数据，当然他的 B + 树高度为 1。

### 最后回顾一道面试题

有一道 MySQL 的面试题，为什么 MySQL 的索引要使用 B + 树而不是其它树形结构? 比如 B 树？

现在这个问题的复杂版本可以参考本文；

他的简单版本回答是：

因为 B 树不管叶子节点还是非叶子节点，都会保存数据，这样导致在非叶子节点中能保存的指针数量变少（有些资料也称为扇出），指针少的情况下要保存大量数据，只能增加树的高度，导致 IO 操作变多，查询性能变低；

### 小结

本文从一个问题出发，逐步介绍了 InnoDB 索引组织表的原理、查询方式，并结合已有知识，回答该问题，结合实践来证明。当然为了表述简单易懂，文中忽略了一些细枝末节，比如一个页中不可能所有空间都用于存放数据，它还会存放一些少量的其他字段比如 page level，index number 等等，另外还有页的填充因子也导致一个页不可能全部用于保存数据。关于二级索引数据存取方式可以参考 MySQL 相关书籍，他的要点是结合主键索引进行回表查询。