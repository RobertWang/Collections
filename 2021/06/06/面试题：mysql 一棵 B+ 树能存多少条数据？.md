> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU0OTE4MzYzMw==&mid=2247512901&idx=2&sn=69c2ad6174ceaafe5fe0d2462fbf575d&chksm=fbb13abbccc6b3ad9c8b4b9bc5f627087d3da78472f0fd2e9919cdf50655e93faea92d52f2cb&mpshare=1&scene=1&srcid=0605FOybmv21HMXlorgTRPR6&sharer_sharetime=1622886418832&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

大家好，我是 Tom 哥~

> 今日寄语：充满活力的新人，能让身边的人都重回初心，真是不可思议。

mysql 的 InnoDB 存储引擎 一棵 B + 树可以存放多少行数据?

（答案在文章中！！）

要搞清楚这个问题，首先要从 InnoDB 索引数据结构、数据组织方式说起。

我们都知道计算机有五大组成部分：控制器，运算器，存储器，输入设备，输出设备。

其中很重要的，也跟今天这个题目有关系的是存储器。

我们知道万事万物都有自己的单元体系，若干个小单体组成一个个大的个体。就像拼乐高一样，可以自由组合。所以说，如果能熟悉最小单元，就意味着我们抓住了事物的本事，再复杂的问题也会迎刃而解。

存储单元

  

**存储器范围比较大，但是数据具体怎么存储，有自己的最小存储单元。**

1、数据持久化存储磁盘里，磁盘的最小单元是扇区，`一个扇区的大小是 512个字节`

2、文件系统的最小单元是块，`一个块的大小是 4K`

3、InnoDB 存储引擎，有自己的最小单元，称之为页，`一个页的大小是16K`

**扇区、块、页这三者的存储关系？**

InnoDB 引擎  

```
mysql -u root -p
```

查看 InnoDB 的页大小

```
show variables like 'innodb_page_size';
```

![](https://mmbiz.qpic.cn/mmbiz_png/2KTof9YshwfxA7iaBRvMqhOtRNlUOketzEFG2ibmpdO3d6tPLUeKoXj30aAaJKib31DbJrsucRC8RGAffyxIcNO6w/640?wx_fmt=png)

mysql 数据库中，table 表中的记录都是存储在页中，那么一页可以存多少行数据？假如一行数据的大小约为 1K 字节，那么按 `16K / 1K = 16`，可以计算出一页大约能存放 16 条数据。

mysql 的最小存储单元叫做 “页”，这么多的页是如何构建一个庞大的数据组织，我们又如何知道数据存储在哪一个页中？

如果逐条遍历，性能肯定很差。为了提升查找速度，我们引入了`B+树`，先来看下`B+树`的存储结构

![](https://mmbiz.qpic.cn/mmbiz_png/2KTof9YshwfxA7iaBRvMqhOtRNlUOketzQoknttTWIrdxibtddSIiaXkNRwaa7nbLNhzAZic8jOx7ExBGFkDT5hZQg/640?wx_fmt=png)

页除了可以存放`数据`（叶子节点），还可以存放`健值和指针`（非叶子节点），当然他们是有序的。这样的数据组织形式，我们称为索引组织表。

> 如：上图中 page number=3 的页，该页存放键值和指向数据页的指针，这样的页由 N 个键值 + 指针组成

**B+ 树是如何检索记录？**

*   首先找到根页，你怎么知道一张表的根页在哪呢？
    
*   其实每张表的根页位置在表空间文件中是固定的，即 page number=3 的页
    
*   找到根页后通过二分查找法，定位到 id=5 的数据应该在指针 P5 指向的页中
    
*   然后再去 page number=5 的页中查找，同样通过二分查询法即可找到 id=5 的记录
    

  

  

如何计算 B + 树的高度？

在`InnoDB` 的表空间文件中，约定`page number = 3`表示主键索引的根页

```
SELECT
b.name, a.name, index_id, type, a.space, a.PAGE_NO
FROM
information_schema.INNODB_SYS_INDEXES a,
information_schema.INNODB_SYS_TABLES b
WHERE
a.table_id = b.table_id AND a.space <> 0
and b.name like '%sp_job_log';
```

![](https://mmbiz.qpic.cn/mmbiz_png/2KTof9YshwfxA7iaBRvMqhOtRNlUOketzDYSy4C3FBQGVAicTia8eWaE0ibSbmR1nR0fQrxvPzpH314j8wwD7BQzJQ/640?wx_fmt=png)

> 从图中可以看出，每个表的主键索引的根页的 page number 都是 3，而其他的二级索引 page number 为 4

在根页偏移量为`64`的地方存放了该 B + 树的`page level`。主键索引 B + 树的根页在整个表空间文件中的第 3 个页开始，所以算出它在文件中的偏移量：`16384*3 + 64 = 49152 + 64 =49216`，前 2 个字节中。

首先，找到 MySql 数据库物理文件存放位置：

```
show global variables like "%datadir%" ;
```

![](https://mmbiz.qpic.cn/mmbiz_png/2KTof9YshwfxA7iaBRvMqhOtRNlUOketznDFboCv74nXKOaM99gicGfGPWOX4iaV47GdCokFqSrlUv32h8z5mS6wA/640?wx_fmt=png)

hexdump 工具，查看表空间文件指定偏移量上的数据：

```
hexdump -s 49216 -n 10  sp_job_log.ibd
```

![](https://mmbiz.qpic.cn/mmbiz_png/2KTof9YshwfxA7iaBRvMqhOtRNlUOketzuZepNfX9ZfrDShVg5coaEoKmmZEe5jAxad1Te2Q5y8v5IcI09mzzpw/640?wx_fmt=png)

page_level 值是 1，那么 B + 树高度为 `page level + 1 = 2`

**特别说明：**

*   查询数据库时，不论读一行，还是读多行，都是将这些行所在的整页数据加载，然后在内存中匹配过滤出最终结果。
    
*   表的检索速度跟树的深度有直接关系，毕竟一次页加载就是一次 IO，而磁盘 IO 又是比较费时间。`对于一张千万级条数B+树高度为3的表与几十万级B+树高度也为3的表，其实查询效率相差不大。`
    

一棵树可以存放多少行数据？

**假设 B + 树的深度为 2**

这棵 B + 树的存储总记录数 = `根节点指针数 * 单个叶子节点记录条数`

**那么指针数如何计算？**

假设主键 ID 为`bigint`类型，长度为`8字节`，而指针大小在 InnoDB 源码中设置为`6字节`，这样一共`14字节`。

那么一个页中能存放多少这样的组合，就代表有多少指针，即 `16384 / 14 = 1170`。那么可以算出一棵高度为 2 的 B + 树，能存放 `1170 * 16 = 18720` 条这样的数据记录。

同理：

高度为 3 的 B + 树可以存放的行数 =  `1170 * 1170 * 16 = 21902400`

> 千万级的数据存储只需要约 3 层 B + 树，查询数据时，每加载一页（page）代表一次 IO。所以说，根据主键 id 索引查询约 3 次 IO 便可以找到目标结果。

**对于一些复杂的查询，可能需要走二级索引，那么通过二级索引查找记录最多需要花费多少次 IO 呢？**

![](https://mmbiz.qpic.cn/mmbiz_png/2KTof9YshwfxA7iaBRvMqhOtRNlUOketz38TgsCyJTkFxDq7psbdsdsbYqoL9le40CKiaeaiaObAliaFWnOaGs48aA/640?wx_fmt=png)

首先，从二级索引 B + 树中，根据`name` 找到对应的主键 id  

![](https://mmbiz.qpic.cn/mmbiz_png/2KTof9YshwfxA7iaBRvMqhOtRNlUOketzk8Vt860QGV3uwh5GjeEgfwmRcYFpADH8hZNwV2ic5eTutzYPGhVEttQ/640?wx_fmt=png)

然后，再根据主键 id 从 聚簇索引查找到对应的记录。如上图所示，二级索引有 3 层，聚簇索引有 3 层，那么最多花费的 IO 次数是：3+3 = 6

聚簇索引默认是主键，如果表中没有定义主键，InnoDB 会选择一个唯一的非空索引代替。如果没有这样的索引，InnoDB 会隐式定义一个主键来作为聚簇索引。

> 这也是为什么 InnoDB 表必须有主键，并且推荐使用整型的自增主键！！！

> InnoDB 使用的是聚簇索引，将主键组织到一棵 B + 树中，而行数据就储存在叶子节点上

**举例说明：**

1、若使用`"where id = 14"`这样的条件查找记录，则按照 B + 树的检索算法即可查找到对应的叶节点，之后获得行数据。

2、若对 Name 列进行条件搜索，则需要两个步骤：

*   第一步在辅助索引 B + 树中检索 Name，到达其叶子节点获取对应的主键值。
    
*   第二步使用主键值在主索引 B + 树中再执行一次 B + 树检索操作，最终到达叶子节点即可获取整行数据。（重点在于通过其他键需要建立辅助索引）
    
      
    
      
    

实战演示

实际项目中，每个表的结构设计都不一样，占用的存储空间大小也各不相等。如何计算不同的 B + 树深度下，一个表可以存储的记录条数？

我们以业务日志表 `sp_job_log` 为例，讲解详细的计算过程：

1、查看表的状态信息

```
show table status like 'sp_job_log'\G
```

![](https://mmbiz.qpic.cn/mmbiz_png/2KTof9YshwfxA7iaBRvMqhOtRNlUOketzPSO6icytqwXDU2rF2yy2CJKyc3V4462NyPbvuVroIicBhoz7Bk5TILDQ/640?wx_fmt=png)

图中看到`sp_job_log`表的行平均大小为`153`个字节

2、查看表结构

```
desc sp_job_log;
```

![](https://mmbiz.qpic.cn/mmbiz_png/2KTof9YshwfxA7iaBRvMqhOtRNlUOketzlKOpFialibXWIiaZOIY26AZjhr2lTWicGTfOxAdq7xeZTzCZS0ib5ZQgVicQ/640?wx_fmt=png)

3、计算 B + 树的行数

*   单个叶子节点（页）中的记录数 = 16K / 153 = 105
    
*   非叶子节点能存放多少指针， 16384 / 14 = 1170
    
*   如果树的高度为 3，可以存放的记录行数 =  1170 * 1170 * 105 = 143,734,500
    

  

  

最后加餐

普通索引和唯一索引在查询效率上有什么不同？

唯一索引就是在普通索引上增加了约束性，也就是关键字唯一，找到了关键字就停止检索。而普通索引，可能会存在用户记录中的关键字相同的情况，根据页结构的原理，当我们读取一条记录的时候，不是单独将这条记录从磁盘中读出去，而是将这个记录所在的页全部加载到内存中进行读取。InnoDB 存储引擎的页大小为 16KB，在一个页中可能存储着上千个记录，因此在普通索引的字段上进行查找也就是在内存中多几次`判断下一条记录`的操作，对于 CPU 来说，这些操作所消耗的时间是可以忽略不计的。所以对一个索引字段进行检索，采用普通索引还是唯一索引在检索效率上基本上没有差别。