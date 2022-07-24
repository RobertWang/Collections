> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247517523&idx=2&sn=f37e84dcda87d1c9321789574e858ef5&chksm=e969db5ede1e5248fd839b01c29eb0c70dba2c98524d3b6ec98a966bb2eb6bacf30e240ee66e&mpshare=1&scene=1&srcid=0722GrAXU7mLSzvBJJy5CGA4&sharer_sharetime=1658456598686&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

点击上方 "**Python 人工智能技术** " 关注，星标或者置顶

22 点 24 分准时推送，第一时间送达

> 编辑：乐乐 | 来自：blog.csdn.net/qq_35642036/article/details/82809974

上一篇：[Python 数据可视化的 3 大步骤！](http://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247517479&idx=1&sn=c47a29b1ea81f0990e937ad8c0596380&chksm=e969db2ade1e523cf3ccc290e40de3ddd363f8bde313f5d02396690fe988eea602236ae8bff3&scene=21#wechat_redirect)

**正文**

**大家好，我是 Python 人工智能技术**

在开始介绍如何优化 sql 前，先附上 mysql 内部逻辑图让大家有所了解

![](https://mmbiz.qpic.cn/mmbiz_png/8KKrHK5ic6XCXibIRJoZL2dvEogyX4vR0dsMEsB8E88zlqcEWkDR2BQ4wny7523nRHcaG0HDZ6Vf2AfpvxEkN8AQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**（2）查询缓存：** 优先在缓存中进行查询，如果查到了则直接返回，如果缓存中查询不到，在去数据库中查询。

*   先说下缓存中数据存储格式：key（sql 语句）- value（数据值），所以如果 SQL 语句（key）只要存在一点不同之处就会直接进行数据库查询了；
    
*   由于表中的数据不是一成不变的，大多数是经常变化的，而当数据库中的数据变化了，那么相应的与此表相关的缓存数据就需要移除掉；
    

*   在分析是否走索引查询时，是通过进行动态数据采样统计分析出来；只要是统计分析出来的，那就可能会存在分析错误的情况，所以在 SQL 执行不走索引时，也要考虑到这方面的因素
    

**（5）执行器：** 根据一系列的执行计划去调用存储引擎提供的 API 接口去调用操作数据，完成 SQL 的执行。

### 一、SQL 语句及索引的优化

SQL 语句的优化

##### 1. 尽量避免使用子查询

例：

其子查询在 Mysql5.5 版本里，内部执行计划是这样：先查外表再匹配内表，而不是先查内表 t2，当外表的数据很大时，查询速度会非常慢。

在 MariaDB10/Mysql5.6 版本里，采用 join 关联方式对其进行了优化，这条 SQL 语句会自动转换为：`SELECT t1.* FROM t1 JOIN t2 on t1.id = t2.id`

但请注意的是：优化只针对 SELECT 有效，对`UPDATE/DELETE`子查询无效，固生产环境应避免使用子查询

由于 MySQL 的优化器对于子查询的处理能力比较弱，所以不建议使用子查询，可以改写成`Inner Join`，之所以 join 连接效率更高，是因为 MySQL 不需要在内存中创建临时表

##### 2. 用 IN 来替换 OR

*   低效查询：`SELECT * FROM t WHERE id = 10 OR id = 20 OR id = 30;`
    
*   高效查询：S`ELECT * FROM t WHERE id IN (10,20,30);`
    

##### 3. 读取适当的记录 LIMIT M,N，而不要读多余的记录

```
select id,name from t limit 866613, 20


```

```
select id,name from table_name where id> 866612 limit 20


```

##### 4. 禁止不必要的 Order By 排序

如果我们对结果没有排序的要求，就尽量少用排序；

如果排序字段没有用到索引，也尽量少用排序；

另外，分组统计查询时可以禁止其默认排序

```
SELECT goods_id,count(*) FROM t GROUP BY goods_id;


```

默认情况下，Mysql 会对所有的`GROUP BT col1,col2…`的字段进行排序，也就是说上述会对 goods_id 进行排序，如果想要避免排序结果的消耗，可以指定`ORDER BY NULL`禁止排序：

```
SELECT goods_id,count(*) FROM t GROUP BY goods_id ORDER BY NULL


```

##### 5. 总和查询可以禁止排重用 union all

union 和 union all 的差异主要是前者需要将结果集合并后再进行唯一性过滤操作，这就会涉及到排序，增加大量的 CPU 运算，加大资源消耗及延迟。

当然，`union all`的前提条件是两个结果集没有重复数据。所以一般是我们明确知道不会出现重复数据的时候才建议使用 `union all` 提高速度。

##### 6. 避免随机取记录

```
SELECT * FROM t1 WHERE 1 = 1 ORDER BY RAND() LIMIT 4;
SELECT * FROM t1 WHERE id >= CEIL(RAND()*1000) LIMIT 4;


```

以上两个语句都无法用到索引

##### 7. 将多次插入换成批量 Insert 插入

```
INSERT INTO t(id, name) VALUES(1, 'aaa');
INSERT INTO t(id, name) VALUES(2, 'bbb');
INSERT INTO t(id, name) VALUES(3, 'ccc');
—>
INSERT INTO t(id, name) VALUES(1, 'aaa'),(2, 'bbb'),(3, 'ccc');


```

##### 8. 只返回必要的列，用具体的字段列表代替 select * 语句

SELECT * 会增加很多不必要的消耗（cpu、io、内存、网络带宽）；增加了使用覆盖索引的可能性；当表结构发生改变时，前者也需要经常更新。所以要求直接在 select 后面接上字段名。

MySQL 数据库是按照行的方式存储，而数据存取操作都是以一个页大小进行 IO 操作的，每个 IO 单元中存储了多行，每行都是存储了该行的所有字段。所以无论取一个字段还是多个字段，实际上数据库在表中需要访问的数据量其实是一样的。

但是如果查询的字段都在索引中，也就是覆盖索引，那么可以直接从索引中获取对应的内容直接返回，不需要进行回表，减少 IO 操作。除此之外，当存在 order by 操作的时候，select 子句中的字段多少会在很大程度上影响到我们的排序效率。

##### 9. 区分 in 和 exists

```
select * from 表A where id in (select id from 表B)


```

上面的语句相当于：

```
select * from 表A where exists(select * from 表B where 表B.id=表A.id)


```

区分 in 和 exists 主要是造成了驱动顺序的改变（这是性能变化的关键），如果是 exists，那么以外层表为驱动表，先被访问，如果是 IN，那么先执行子查询。所以 IN 适合于外表大而内表小的情况；EXISTS 适合于外表小而内表大的情况。

##### 10. 优化 Group By 语句

如果`group by`需要统计的数据量不大，尽量只使用内存临时表；也可以通过适当调大`tmp_table_size`参数，来避免用到磁盘临时表；

*   如果数据量实在太大，使用`SQL_BIG_RESULT`这个提示，来告诉优化器直接使用排序算法（直接用磁盘临时表）得到`group by`的结果。
    

使用 where 子句替换 Having 子句：避免使用 having 子句，having 只会在检索出所有记录之后才会对结果集进行过滤，这个处理需要排序分组，如果能通过 where 子句提前过滤查询的数目，就可以减少这方面的开销。

*   低效: `SELECT JOB, AVG(SAL) FROM EMP GROUP by JOB HAVING JOB = ‘PRESIDENT’ OR JOB = ‘MANAGER’`
    
*   高效: `SELECT JOB, AVG(SAL) FROM EMP WHERE JOB = ‘PRESIDENT’ OR JOB = ‘MANAGER’ GROUP by JOB`
    

##### 11. 尽量使用数字型字段

若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能。引擎在处理查询和连接时会逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了。

##### 12. 优化 Join 语句

当我们执行两个表的 Join 的时候，就会有一个比较的过程，逐条比较两个表的语句是比较慢的，因此可以把两个表中数据依次读进一个内存块中，在 Mysql 中执行：`show variables like ‘join_buffer_size’`，可以看到 join 在内存中的缓存池大小，其大小将会影响 join 语句的性能。在执行 join 的时候，数据库会选择一个表把他要返回以及需要进行和其他表进行比较的数据放进`join_buffer`。

*   **left join** 前面的表是驱动表，后面的表是被驱动表
    
*   **right join** 后面的表是驱动表，前面的表是被驱动表
    
*   **inner join / join** 会自动选择表数据比较少的作为驱动表
    
*   **straight_join(≈join)** 直接选择左边的表作为驱动表（语义上与 join 类似，但去除了 join 自动选择小表作为驱动表的特性）
    

假设有表如右边：t1 与 t2 表完全一样，a 字段有索引，b 无索引，t1 有 100 条数据，t2 有 1000 条数据

若被驱动表有索引，那么其执行算法为：`Index Nested-Loop Join（NLJ）`，示例如下：

1. 执行语句：`select * from t1 straight_join t2 on (t1.a=t2.a)；`由于被驱动表 t2.a 是有索引的，其执行逻辑如下：

*   从表 t1 中读入一行数据 R；
    
*   从数据行 R 中，取出 a 字段到表 t2 里去查找；
    
*   取出表 t2 中满足条件的行，跟 R 组成一行，作为结果集的一部分；
    
*   重复执行步骤 1 到 3，直到表 t1 的末尾循环结束。
    
*   如果一条 join 语句的 Extra 字段什么都没写的话，就表示使用的是 NLJ 算法
    

![](https://mmbiz.qpic.cn/mmbiz_png/8KKrHK5ic6XCXibIRJoZL2dvEogyX4vR0deIbRe2m1wiap5e6lJyXXiclpLWVNCqmtTV88QYXVsVsTFMK9LYhN525g/640?wx_fmt=png)

若被驱动表无索引，那么其执行算法为：`Block Nested-Loop Join（BLJ）`（Block 块，每次都会取一块数据到内存以减少 I/O 的开销），示例如下：

2. 执行语句：`select * from t1 straight_join t2 on (t1.a=t2.b)；`由于被驱动表 t2.b 是没有索引的，其执行逻辑如下：

*   把驱动表 t1 的数据读入线程内存`join_buffer`（无序数组）中，由于我们这个语句中写的是`select *`，因此是把整个表 t1 放入了内存；
    
*   顺序遍历表 t2，把表 t2 中的每一行取出来，跟`join_buffer`中的数据做对比，满足 join 条件的，作为结果集的一部分返回。
    

![](https://mmbiz.qpic.cn/mmbiz_png/8KKrHK5ic6XCXibIRJoZL2dvEogyX4vR0drx7Zq8NAKhAib2WCcVkGRgt3H9NAo8Rvic8vyOcxE7mzGss0nlNKYf4Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

3. 另外还有一种算法为`Simple Nested-Loop Join（SLJ）`，其逻辑为：顺序取出驱动表中的每一行数据，到被驱动表去做全表扫描匹配，匹配成功则作为结果集的一部分返回。

另外，Innodb 会为每个数据表分配一个存储在磁盘的 表名. ibd 文件，若关联的表过多，将会导致查询的时候磁盘的磁头移动次数过多，从而影响性能

所以实践中，尽可能减少 Join 语句中的 NestedLoop 的循环次数：“永远用小结果集驱动大的结果集”

*   用小结果集驱动大结果集，将筛选结果小的表（在决定哪个表做驱动表的时候，应该是两个表按照各自的条件过滤，过滤完成之后，计算参与 join 的各个字段的总数据量，数据量小的那个表，就是 “小表”）首先连接，再去连接结果集比较大的表，尽量减少 join 语句中的 Nested Loop 的循环总次数
    
*   优先优化 Nested Loop 的内层循环（也就是最外层的 Join 连接），因为内层循环是循环中执行次数最多的，每次循环提升很小的性能都能在整个循环中提升很大的性能；
    
*   对被驱动表的 join 字段上建立索引；
    
*   当被驱动表的 join 字段上无法建立索引的时候，设置足够的 Join Buffer Size。
    
*   尽量用 inner join(因为其会自动选择小表去驱动大表). 避免 LEFT JOIN (一般我们使用 Left Join 的场景是大表驱动小表) 和 NULL，那么如何优化 Left Join 呢？
    

*   条件中尽量能够过滤一些行将驱动表变得小一点，用小表去驱动大表
    
*   右表的条件列一定要加上索引（主键、唯一索引、前缀索引等），最好能够使 type 达到 range 及以上（ref,eq_ref,const,system）
    

*   适当地在表里面添加冗余信息来减少 join 的次数
    
*   使用更快的固态硬盘
    

性能优化，left join 是由左边决定的，左边一定都有，所以右边是我们的关键点，建立索引要建在右边。当然如果索引是在左边的，我们可以考虑使用右连接，如下

```
select * from atable left join btable on atable.aid=btable.bid;
-- 最好在bid上建索引


```

```
牛逼啊！接私活必备的 N 个开源项目！赶快收藏吧

```

### 索引的优化 / 如何避免索引失效

##### 1. 最佳左前缀法则

##### 2. 不在索引列上做任何操作

2. 函数：`select * from t_user where length(name)=6;` 此语句对字段使用到了函数，会导致索引失效

从 MySQL 8.0 开始，索引特性增加了函数索引，即可以针对函数计算后的值建立一个索引，也就是说该索引的值是函数计算后的值，所以就可以通过扫描索引来查询数据。

```
alter table t_user add key idx_name_length ((length(name)));


```

（自动 / 手动）类型转换

*   （字符串类型必须带''引号才能使索引生效）字段是 varchar，用整型进行查询时，无法走索引，如`select * from user where phone = 13030303030`；
    

Mysql 在执行上述语句时，会把字段转换为数字再进行比较，所以上面那条语句就相当于：`select * from user where CAST(phone AS signed int) = 13030303030`; CAST 函数是作用在了 phone 字段，而 phone 字段是索引，也就是对索引使用了函数！所以索引失效

*   字段是 int，用 string 进行查询时，mysql 会自动转化，可以走索引，如：`select * from user where id = '1'`；
    

MySQL 在遇到字符串和数字比较的时候，会自动把字符串转为数字，然后再进行比较。以上这条语句相当于：`select * from user where id = CAST(“1” AS signed int)`，索引字段并没有用任何函数，CAST 函数是用在了输入参数，因此是可以走索引扫描的。

3. 存储引擎不能使用索引中范围条件右边的列。

如这样的`sql: select * from user where username='123' and age>20 and phone='1390012345'`, 其中 username, age, phone 都有索引，只有 username 和 age 会生效，phone 的索引没有用到。

4. 尽量使用覆盖索引（只访问索引的查询（索引列和查询列一致））

如`select age from user`，减少`select *`

5.mysql 在使用负向查询条件 (!=、<>、not in、not exists、not like) 的时候无法使用索引会导致全表扫描。

7.like 以通配符开头 (`%abc..`) 时，mysql 索引失效会变成全表扫描的操作。

所以最好用右边 like ‘abc%’。如果两边都要用，可以用`select username from user where username like '%abc%'`，其中 username 是必须是索引列，才可让索引生效

假如`index(a,b,c), where a=3 and b like ‘abc%’ and c=4`，a 能用，b 能用，c 不能用，类似于不能使用范围条件右边的列的索引

对于一棵 B + 树索引来讲，如果根节点是字符 def，假如查询条件的通配符在后面，例如 abc%，则其知道应该搜索左子树，假如传入为 efg%，则应该搜索右子树，如果通配符在前面 %abc，则数据库不知道应该走哪一面，就都扫描一遍了。

8. 少用 or，在 WHERE 子句中，如果在 OR 前的条件列是索引列，而在 OR 后的条件列不是索引列，那么索引会失效。

```
select * from t_user where id = 1 or age = 18; 
-- id有索引，name没有，此时没法走索引


```

因为 OR 的含义就是两个只要满足一个即可，因此只有一个条件列是索引列是没有意义的，只要有条件列不是索引列，就会进行全表扫描。

必须要 or 前后的字段都有索引，查询才能使用上索引（分别使用，最后合并结果`type = index_merge`）

![](https://mmbiz.qpic.cn/mmbiz_png/8KKrHK5ic6XCXibIRJoZL2dvEogyX4vR0d5BLs02Kok35wFhQweSibtickz5SFsydvSem6C5ZGbEH7Q5DbVkTfjexQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

9. 在组合 / 联合索引中，将有区分度的索引放在前面

如果没有区分度，例如用性别，相当于把整个大表分成两部分，查找数据还是需要遍历半个表才能找到，使得索引失去了意义。

10. 使用前缀索引

短索引不仅可以提高查询性能而且可以节省磁盘空间和 I/O 操作，减少索引文件的维护开销，但缺点是不能用于 ORDER BY 和 GROUP BY 操作，也不能用于覆盖索引。

比如有一个`varchar(255)`的列，如果该列在前 10 个或 20 个字符内，可以做到既使前缀索引的区分度接近全列索引，那么就不要对整个列进行索引。为了减少`key_len`，可以考虑创建前缀索引，即指定一个前缀长度，可以使用`count(distinct leftIndex(列名, 索引长度))/count(*)` 来计算前缀索引的区分度。

11.SQL 性能优化 explain 中的 type：至少要达到 range 级别，要求是 ref 级别，如果可以是 consts 最好。

*   consts：单表中最多只有一个匹配行（主键或者唯一索引），在优化阶段即可读取到数据。
    
*   ref：使用普通的索引
    
*   range：对索引进行范围检索。
    

当 type=index 时，索引物理文件全扫，速度非常慢。

****你还有什****么想要补充的吗？****  

> 免责声明：本文内容来源于网络，文章版权归原作者所有，意在传播相关技术知识 & 行业趋势，供大家学习交流，若涉及作品版权问题，请联系删除或授权事宜。  

--END--