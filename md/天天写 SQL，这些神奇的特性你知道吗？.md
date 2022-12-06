> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NzM0MjcyMQ==&mid=2650160967&idx=4&sn=56e767bf653bff069fdd7317c4795eb3&chksm=bed9e42989ae6d3fe4711849bda0d2701a598cccabd75d508d9e773310887cc5d852e067dec9&mpshare=1&scene=1&srcid=0918sbWeSGd8LJ36lbF4I7sB&sharer_sharetime=1663475428594&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

作者 | 华为云开发者联盟 - 龙哥手记

**一 SQL 的第一个神奇特性**
------------------

日常开发我们经常会对表进行聚合查询操作，但只能在 SELECT 子句中写下面 3 种内容：通过 GROUP BY 子句指定的聚合键、聚合函数（SUM 、AVG 等）、常量，不懂没关系我们来看个例子

### 听我解释

有学生班级表（tbl_student_class） 以及数据如下

```
DROP TABLE IF EXISTS tbl_student_class;
CREATE TABLE tbl_student_class (
  id int(8) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增主键',
 sno varchar(12) NOT NULL COMMENT '学号',
 cno varchar(5) NOT NULL COMMENT '班级号',
 cname varchar(20) NOT NULL COMMENT '班级名',
 PRIMARY KEY (id)
) COMMENT='学生班级表';
-- ----------------------------
-- Records of tbl_student_class
-- ----------------------------
INSERT INTO tbl_student_class VALUES ('1', '20190607001', '0607', '影视7班');
INSERT INTO tbl_student_class VALUES ('2', '20190607002', '0607', '影视7班');
INSERT INTO tbl_student_class VALUES ('3', '20190608003', '0608', '影视8班');
INSERT INTO tbl_student_class VALUES ('4', '20190608004', '0608', '影视8班');
INSERT INTO tbl_student_class VALUES ('5', '20190609005', '0609', '影视9班');
INSERT INTO tbl_student_class VALUES ('6', '20190609006', '0609', '影视9班');

```

我想统计各个班（班级号、班级名）一个有多少人、以及最大的学号，我们该怎么写这个查询 SQL？我想大家用脚都写得出来

```
SELECT cno,cname,count(sno),MAX(sno) 
FROM tbl_student_class
GROUP BY cno,cname;

```

可是有人会想了，cno 和 cname 本来就是一对一，cno 一旦确定，cname 也就确定了吗，那 SQL 咱们是不是可以这么写？

```
SELECT cno,cname,count(sno),MAX(sno) 
FROM tbl_student_class
GROUP BY cno;

```

执行报错了

```
[Err] 1055 - Expression #2 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'test.tbl_student_class.cname' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
提示信息：SELECT 列表中的第二个表达式（cname）不在 GROUP BY 的子句中，同时它也不是**聚合函数**；这与 sql 模式：ONLY_FULL_GROUP_BY 不相容的哈

```

那为什么 GROUP BY 之后不能直接引用原表（不在 GROUP BY 子句）中的列 ？莫急，我们慢慢往下看就明白了

### 1.0 SQL 模式

MySQL 服务器可以在不同的 SQL 模式下运行，并且可以针对不同的客户端以不同的方式应用这些模式，具体取决于 sql_mode 系统变量的值。DBA 可以设置全局 SQL 模式以匹配站点服务器操作要求，并且每个应用程序可以将其会话 SQL 模式设置为其自己的要求。

模式会影响 MySQL 支持的 SQL 语法以及它执行的数据验证检查，这使得在不同环境中使用 MySQL 以及将 MySQL 与其他数据库服务器一起使用变得更加容易。更多详情请查官网自己找：Server SQL Modes

MySQL 版本不同，内容会略有不同（包括默认值），查阅的时候注意与自身的 MySQL 版本保持一致哈

SQL 模式主要分两类：语法支持类和数据检查类，常用的如下

### 语法支持类

*   ONLY_FULL_GROUP_BY  
    对于 GROUP BY 聚合操作，如果在 SELECT 中的列、HAVING 或者 ORDER BY 子句的列，没有在 GROUP BY 中出现，那么这个 SQL 是不合法的
    
*   ANSI_QUOTES  
    启用 ANSI_QUOTES 后，不能用双引号来引用字符串，因为它被解释为识别符。设置它以后，update t set f1="" …，会报 Unknown column ‘’ in field list 这样的语法错误
    
*   PIPES_AS_CONCAT  
    把 || 视为字符串的连接操作符而非 或 运算符，这种和 Oracle 数据库是一样的哈，也和字符串的拼接函数 CONCAT () 有点类似
    
*   NO_TABLE_OPTIONS  
    使用 SHOW CREATE TABLE 时不会输出 MySQL 特有的语法部分，如 ENGINE，这个在使用 mysqldump 跨 DB 种类迁移的时候需要考虑
    
*   NO_AUTO_CREATE_USER  
    字面意思不自动创建用户，在给 MySQL 用户授权时，我们习惯用 GRANT … ON … TO dbuser，顺道一起创建用户。设置该选项后就与 oracle 操作类似，授权之前必须先建立好用户
    

### 1.1 数据检查类

*   NO_ZERO_DATE
    

认为日期‘0000-00-00’非法，与是否设置后面的严格模式有关系

1、如果设置了严格模式，则 NO_ZERO_DATE 自然满足。但如果是 INSERT IGNORE 或 UPDATE IGNORE，’0000-00-00’依然允许且只显示 warning；

2、如果在非严格模式下，设置了 NO_ZERO_DATE，效果与上面一样，’0000-00-00’ 允许但显示 warning；如果没有设置 NO_ZERO_DATE，no warning，当做完全合法的值；

3、NO_ZERO_IN_DATE 情况与上面类似，不同的是控制日期和天，是否可为 0 ，即 2010-01-00 是否合法；

*   NO_ENGINE_SUBSTITUTION
    

使用 ALTER TABLE 或 CREATE TABLE 指定 ENGINE 时，需要的存储引擎被禁用或未编译，该如何处理。启用 NO_ENGINE_SUBSTITUTION 时，那么直接抛出错误；不设置此值时，CREATE 用默认的存储引擎替代，ATLER 不进行更改，并抛出一个 warning

*   STRICT_TRANS_TABLES
    

设置它，表示启用严格模式。注意 STRICT_TRANS_TABLES 不是几种策略的组合，单独指 INSERT、UPDATE 出现少值或无效值该如何处理：

1、前面提到的把 ‘’ 传给 int，严格模式下非法，若启用非严格模式则变成 0，产生一个 warning；

2、Out Of Range，变成插入最大边界值；

3、当要插入的新行中，不包含其定义中没有显式 DEFAULT 子句的非 NULL 列的值时，该列缺少值

### 1.2 默认模式

当我们没有修改配置文件的情况下，MySQL 是有自己的默认模式的；版本不同，默认模式也不同

```
-- 查看 MySQL 版本
SELECT VERSION();
-- 查看 sql_mode
SELECT @@sql_mode;

```

![](https://mmbiz.qpic.cn/mmbiz_gif/dkwuWwLoRK9mWPlj4ia8tk5ctiaPYfia5KWjwkJku14yicqhIBolZqicoVdKn1IwVn8UxZTGJe0TZCKzo8fWdghVhAQ/640?wx_fmt=gif)

我们可以看到，5.7.21 的默认模式包含

```
ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

```

而第一个：ONLY_FULL_GROUP_BY 就会约束：当我们进行聚合查询的时候，SELECT 的列不能直接包含非 GROUP BY 子句中的列。那如果我们去掉该模式（从 “严格模式” 到 “宽松模式”）呢？

![](https://mmbiz.qpic.cn/mmbiz_gif/dkwuWwLoRK9mWPlj4ia8tk5ctiaPYfia5KWpPYTwf1Xl3Ap04pt7eOtl5LE4PKkfPJfTLiaiaaDNdicD1Cbiaia7jRsPtw/640?wx_fmt=gif)

我们发现，上述报错的 SQL

```
-- 宽松模式下 可以执行
SELECT cno,cname,count(sno),MAX(sno) 
FROM tbl_student_class
GROUP BY cno;

```

能正常执行了，但是一般情况下不推荐这样配置，线上环境往往是 “严格模式”，而不是 “宽松模式”；虽然案例中，无论是 “严格模式”，还是 “宽松模式”，结果都是对的，那是因为 cno 与 cname 唯一对应的，如果 cno 与 cname 不是唯一对应，那么在 “宽松模式下” cname 的值是随机的，这就会造成难以排查的问题，有兴趣的可以去试下；

**二 SQL 的第二个神奇特性**
------------------

### 2.1 问题描述下

今天我想比较两个数据集

表 A 一共 50,000,000 行，其中有一列叫「ID」，表 B 也有一列叫「ID」。我想查的是有 A 表里有多少 ID 在 B 表里面，数据库用的是 snowflake，它是一种一种多租户、事务性、安全、高度可扩展的弹性数据库，或者叫它实施数仓也行，具备完整的 SQL 支持和 schema-less 数据模式，支持 ACID 的事务，也提供用于遍历、展平和嵌套半结构化数据的内置函数和 SQL 扩展，并支持 JSON 和 Avro 等流行格式；

用 query:

```
with A as
(
 select distinct(id) as id from Table_A
),
B as 
(
 select distinct(id) as id from Table_B 
),
result as 
(
 select * from A where id in (select id from B)
)
select count(*) from result

```

返回结果是 26,000,000

也就是说，A 应有 24,000,000 行不在 B 里面，对吧

可是我把第 11 行的 in 改成 not in 后，情况有点出乎我的意料

```
with A as
(
 select distinct(id) as id from Table_A
),
B as 
(
 select distinct(id) as id from Table_B 
),
result as 
(
 select * from A where id not in (select id from B)
)
select count(*) from result

```

返回结果竟然是 0，而不是 24,000,000

于是我在 snowflake 论坛搜了下，发现 5 年前在这帖子下面有人回复到：

> If you use NOT IN (subquery), it compares every returned value and in case of NULL on any side of comparison it stops immediately with non defined result if you use NOT IN (subquery), it compares every returned value and in case of NULL on any side of comparison it stops immediately with non defined result

就是说，当你用 not in，subquery（例如上面第 11 行的 select id from B）里如果有 Null, 那么它就会立刻停止，返回未定义的结果，所以最后结果是 0；

该如何解决？很简单

### 2.2 去掉 null 值

在第 7 行加了限定 where id is not null 后，结果正常了

```
with A as
(
 select distinct(id) as id from Table_A
),
B as 
(
 select distinct(id) as id from Table_B where id is not null
),
result as 
(
 select * from A where id not in (select id from B)
)
select count(*) from result

```

最终返回结果为 24,000,000，这样就对了啊

### 2.3 用 not exists 代替 not in

注意第 11 行，用了 not exists 代替 not in

```
with A as
(
 select distinct(id) as id from Table_A
),
B as 
(
 select distinct(id) as id from Table_B where id is not null
),
result as 
(
 select * from A where not exists (select * from B where A.id=B.id)
)
select count(*) from result

```

返回结果也为 24,000,000

当然，这肯定不是 bug 哈，而是特性，不然不会这么多年都留着，是我懂得太少了，得恶补下 SQL 哭泣。

不过不知道这个特性设计当初目的是什么，如果 subquery 返回了 undefined，你好歹给我报个错啊。这个「特性」不仅仅在 snowflake 上面出现，看 StackOverflow 上讨论，貌似 Oracle 也是有这个「特性」的哈

**三 SQL 的第三个神奇特性**
------------------

像 Web 服务这样需要快速响应的应用场景中，SQL 的性能直接决定了系统是否可以使用；特别在一些中小型应用中，SQL 的性能更是决定服务能否快速响应的唯一标准

严格地优化查询性能时，必须要了解所使用数据库的功能特点，此外，查询速度慢并不只是因为 SQL 语句本身，还有可能内存分配不佳、文件结构不合理、刷脏页等其他原因啊；

因此下面介绍些 SQL 神奇特性，但不能解决所有的性能问题，但是却能处理很多因 SQL 写法不合理而产生的性能问题

所以下面尽量介绍一些不依赖具体数据库实现，使 SQL 执行速度更快、消耗内存更少的优化技巧，只需调整 SQL 语句就能实现的通用的优化 Tip

### **3.1 环境准备**

下文所讲的内容是从 SQL 层面展开的哈，而不是针对某种特性的数据库，也就是说，下文的内容基本上适用于任何关系型数据库都可以；

但是，关系型数据库那么多，逐一来演示示例了，显然不太现实；我们以常用的 MySQL 来进行就行啦

MySQL 版本：5.7.30-log ，存储引擎：InnoDB

准备两张表：tbl_customer 和 tbl_recharge_record

![](https://mmbiz.qpic.cn/mmbiz_jpg/dkwuWwLoRK9mWPlj4ia8tk5ctiaPYfia5KWqU9ic3ebJzViaia3lcDzlu3doFE5ZicjMaSpPib7ic5HLB9VrYJF5JjRmsuQ/640?wx_fmt=jpeg)

### **3.2 使用高效的查询**

针对某一个查询，有时候会有多种 SQL 实现，例如 IN、EXISTS、连接之间的互相转换

从理论上来讲，得到相同结果的不同 SQL 语句应该有相同的性能的哈，但遗憾的是，查询优化器生成的执行计划很大程度上要受到外部数据结构的影响

所以，要想优化查询性能，必须知道如何写 SQL 语句才能使优化器生成更高效的执行计划

### **3.3 使用 EXISTS 代替 IN**

关于 IN，相信大家都比较熟悉，使用方便，也容易理解；虽说 IN 使用方便，但它却存在性能瓶颈

在大多时候， [NOT]IN 和 [NOT]EXISTS 返回的结果是相同的，但是两者用于子查询时，EXISTS 的速度会更快一些

相信大家第一时间想到的是 IN

IN 使用起来确实简单，也非常好理解；我们来看下它的执行计划

![](https://mmbiz.qpic.cn/mmbiz_jpg/dkwuWwLoRK9mWPlj4ia8tk5ctiaPYfia5KWUD5xq7LGTOibzR6Qhx8xSFmdEnmKayO9sElTaeDzEKTUunXHjcdDdpw/640?wx_fmt=jpeg)

我们再来看看 EXISTS 的执行计划：

![](https://mmbiz.qpic.cn/mmbiz_jpg/dkwuWwLoRK9mWPlj4ia8tk5ctiaPYfia5KW0BcjOZFVtGCjNMSLt6TkaYggTwXYpN78gNicpgEC6r3miaEJDeC9nH4A/640?wx_fmt=jpeg)

可以看到的是，IN 的执行计划中新产生了一张临时表：<subquery2> ，这会导致效率变慢

所以通常来讲，EXISTS 比 IN 更快的原因有两个

*   1、如果连接列（customer_id）上建立了索引，那么查询 tbl_recharge_record 时可以通过索引查询，而不是全表查询
    
*   2、使用 EXISTS，一旦查到一行数据满足条件就会终止查询，不用像使用 IN 时一样进行扫描全表（NOT EXISTS 也一样）
    

如果当 IN 的参数是子查询时，数据库首先会执行子查询，然后将结果存储在一张临时表里（内联视图），然后扫描整个视图，很多情况下这种做法非常耗费资源

使用 EXISTS 的话，数据库不会生成临时表

但是从代码的可读性上来看，IN 要比 EXISTS 好，使用 IN 时的代码看起来更加一目了然，易于理解

因此，如果确信使用 IN 也能快速获取结果，就没有必要非得改成 EXISTS 了

其实有很多数据库也尝试着改善了 IN 的性能

Oracle 数据库中，如果我们在有索引的列上使用 IN， 也会先扫描索引

PostgreSQL 从版 本 7.4 起也改善了使用子查询作为 IN 谓词参数时的查询速度

说不定在未来的某一天，无论在哪个关系型数据库上，IN 都能具备与 EXISTS 一样的性能

### **3.4 使用连接代替 IN**

其实在平时工作当中，更多的是用连接代替 IN 来改善查询性能，而非 EXISTS，不是说连接更好，而是 EXISTS 很难掌握

回到问题：查询有充值记录的顾客信息，如果用连接来实现，SQL 改如何写？

![](https://mmbiz.qpic.cn/mmbiz_jpg/dkwuWwLoRK9mWPlj4ia8tk5ctiaPYfia5KWm36TQf9b0T38bNZBBbJTzWWIVGBHwicbDibTHiaeAHJyqXfYMibUSHDJnw/640?wx_fmt=jpeg)

这种写法能充分利用索引；而且，因为没有了子查询，所以数据库也不会生成中间表；所以，查询效率是不错的

至于 JOIN 与 EXISTS 相比哪个性能更好，这不太好说；如果没有索引，可能 EXISTS 会略胜一筹，有索引的话，两者都差不多

### **3.5 避免排序**

说到 SQL 的排序，我们第一时间想到的肯定是：ORDERBY，通过它，我们可以按指定的某些列来顺序输出结果

但是，除了 ORDERBY 显示的排序，数据库内部还有很多运算在暗中进行排序；会进行排序的代表性的运算有下面这些

![](https://mmbiz.qpic.cn/mmbiz_jpg/dkwuWwLoRK9mWPlj4ia8tk5ctiaPYfia5KW0MmiaKicFRjEFu5wycI89yCl014p9RGpwQT2t7SA5EtLNb47NPKV6s9g/640?wx_fmt=jpeg)

如果只在内存中进行排序，那么还好；但是如果因内存不足而需要在硬盘上排序，那么性能就会急剧下降

所以，要尽量避免（或减少）无谓的排序，能够大大提高查询效率

灵活使用集合运算符的 ALL 可选项

SQL 中有 UNION 、 INTERSECT 、 EXCEPT 三个集合运算符，分表代表这集合运算的 并集、交集、差集

默认情况下，这些运算符会为了排除掉重复数据而进行排序

![](https://mmbiz.qpic.cn/mmbiz_jpg/dkwuWwLoRK9mWPlj4ia8tk5ctiaPYfia5KWu0fGPBjibSQ2JTh4k9RGp2K4ibvnkypYWzicfRZ7rvzIWQGexUOhASCqg/640?wx_fmt=jpeg)

Using temporary 表示进行了排序或者分组，显然这个 SQL 并没有进行分组，而是进行了排序运算

所以如果我们不在乎结果中是否有重复数据，或者事先知道不会有重复数据，可以使用 UNIONALL 代替 UNION 试下，可以看到，执行计划中没有排序运算了

![](https://mmbiz.qpic.cn/mmbiz_jpg/dkwuWwLoRK9mWPlj4ia8tk5ctiaPYfia5KWxaOrcdBcgyj2F15iaZEhqEdqUdOJMLR0qpQAib3xwmFaWRicicehM0DIkQ/640?wx_fmt=jpeg)

对于 INTERSECT 和 EXCEPT 也是一样的，加上 ALL 可选项后就不会进行排序了

加上 ALL 可选项是一个非常有效的优化手段，但各个数据库对它的实现情况却是参差不齐，如下图所示

![](https://mmbiz.qpic.cn/mmbiz_jpg/dkwuWwLoRK9mWPlj4ia8tk5ctiaPYfia5KWcAEhfDl1xBsrbhvDM9j85UJk4qmibI1ZT7UWLf9GibPiaIsTtVbd0hicJA/640?wx_fmt=jpeg)

注意：Oracle 使用 MINUS 代替 EXCEPT；MySQL 压根就没有实现 INTERSECT 和 EXCEPT 运算

### **3.6 使用 EXISTS 来代替 DISTINCT**

为了排除重复数据， DISTINCT 也会进行排序

还记得用连接代替 IN 的案例吗，如果不用 DISTINCT

```
SQL：SELECT tc.*FROM tbl_recharge_record trr LEFTJOIN tbl_customer tc on trr.customer_id = tc.id

```

那么查出来的结果会有很多重复记录，所以我们必须改进 SQL

```
SELECTDISTINCT tc.*FROM tbl_recharge_record trr LEFTJOIN tbl_customer tc on trr.customer_id = tc.id

```

会发现执行计划中有个 Using temporary，它表示用到了排序运算

![](https://mmbiz.qpic.cn/mmbiz_jpg/dkwuWwLoRK9mWPlj4ia8tk5ctiaPYfia5KWWKfKVJI19mretZYrJ76NHPNs4CFPzPMd5iagzI0ia2oywaGARLxtxnmw/640?wx_fmt=jpeg)

我们使用 EXISTS 来进行优化

![](https://mmbiz.qpic.cn/mmbiz_jpg/dkwuWwLoRK9mWPlj4ia8tk5ctiaPYfia5KWHb2wM2PPjAZB601zkfZgI3SsKSiapiargqPtTRueRvQ2IME7I1a02AAw/640?wx_fmt=jpeg)

可以看到，已经规避了排序运算

### **3.7 在极值函数中使用索引**

SQL 语言里有两个极值函数：MAX 和 MIN，使用这两个函数时都会进行排序

例如：SELECTMAX (recharge_amount) FROM tbl_recharge_record

会进行全表扫描，并会进行隐式的排序，找出单笔充值最大的金额

但是如果参数字段上建有索引，则只需扫描索引，但不需要扫描整张表

例如：SELECTMAX (customer_id) FROM tbl_recharge_record;

会通过索引：idx_c_id 进行扫描，找出充值记录中最大的顾客 ID

但是这种方法并不是去掉了排序这一过程，而是优化了排序前的查找速度，从而减弱排序对整体性能的影响

能写在 WHERE 子句里的条件千万不要写在 HAVING 子句里

我们来看两个 SQL 以及其执行结果

![](https://mmbiz.qpic.cn/mmbiz_jpg/dkwuWwLoRK9mWPlj4ia8tk5ctiaPYfia5KWtEuiakZktkk0RwH9Me7ku4gyDic88MpX4qFqTGmyp8Q7m4lQcXkYu17g/640?wx_fmt=jpeg)

你就明白了

从结果上来看，两条 SQ 一样；但是从性能上来看，第二条语句写法效率更高，原因有两个:

1）减少排序的数据量

GROUP BY 子句聚合时会进行排序，如果事先通过 WHERE 子句筛选出一部分行，就能够减轻排序的负担了

2）有效利用索引

### **3.8 WHERE 子句的条件里可以使用索引**

HAVING 子句是针对聚合后生成的视图进行筛选的，但是很多时候聚合后的视图都没有继承原表的索引结构

关于 HAVING，更多详情可查看：神奇的 SQL 之 HAVING→ 容易被轻视的主角

在 GROUP BY 子句和 ORDER BY 子句中使用索引

一般来说，GROUP BY 子句和 ORDER BY 子句都会进行排序

如果 GROUP BY 和 ORDER BY 的列有索引，那么可以提高查询效率

特别是在一些数据库中，如果列上建立的是唯一索引，那么排序过程本身都会被省略掉

*   使用索引
    

使用索引是最常用的 SQL 优化手段，这个大家都知道，怕就怕大家不知道：明明有索引，为什么查询还是这么慢（为什么索引没用上）

关于索引未用到的情况，可查看：神奇的 SQL 之擦肩而过 → 真的用到索引了吗，本文就不做过多阐述了

总之就是：查询尽量往索引上靠，规避索引未用上的情况

*   减少临时表
    

在 SQL 中，子查询的结果会被看成一张新表（临时表），这张新表与原始表一样，可以通过 SQL 进行操作

但是，频繁使用临时表会带来两个问题

*   1、临时表相当于原表数据的一份备份，会耗费内存资源
    
*   2、很多时候（特别是聚合时），临时表没有继承原表的索引结构
    

因此，尽量减少临时表的使用也是提升性能的一个重要方法

*   灵活使用 HAVING 子句
    

对聚合结果指定筛选条件时，使用 HAVING 子句是基本原则

但是如果对 HAVING 不熟，我们往往找出替代它的方式来实现，就像这样

![](https://mmbiz.qpic.cn/mmbiz_jpg/dkwuWwLoRK9mWPlj4ia8tk5ctiaPYfia5KWiaibDP82wOk1A8jlvRsfnx9BV0g6lkpP6UG79oyQt1iceSWHyWuFpfE1w/640?wx_fmt=jpeg)

然而，对聚合结果指定筛选条件时不需要专门生成中间表，像下面这样使用 HAVING 子句就可以

![](https://mmbiz.qpic.cn/mmbiz_jpg/dkwuWwLoRK9mWPlj4ia8tk5ctiaPYfia5KWAvH4LRL9SKibwj7mn1GJINfITUCWDHr0iaRJRmNhcIibPCORNaI9SibJUg/640?wx_fmt=jpeg)

HAVING 子句和聚合操作都是同时执行的，所以比起生成临时表后再执行 WHERE 子句，效率会更高一些，而且代码看起来也更简洁

需要对多个字段使用 IN 谓词时，让它们汇总到一处

SQL-92 中加入了行与行比较的功能，这样一来，比较谓词 = 、<、> 和 IN 谓词的参数就不再只是标量值了，而应是值列表了

我们来看一个示例，多个字段使用 IN 谓词

![](https://mmbiz.qpic.cn/mmbiz_jpg/dkwuWwLoRK9mWPlj4ia8tk5ctiaPYfia5KWpmFXSQmOLe59A1z4xzEzuFyaDeOmIrKee7t3UrOhibIJzfDrWS7LzPA/640?wx_fmt=jpeg)

这段代码中用到了两个子查询，我们可以进行列汇总优化，把逻辑写在一起

![](https://mmbiz.qpic.cn/mmbiz_jpg/dkwuWwLoRK9mWPlj4ia8tk5ctiaPYfia5KWV551FTwAY57DKKjOqaeKHOmkAh4APbR2IUMLWbXJY36jic2lfUtcOkg/640?wx_fmt=jpeg)

这样一来，子查询不用考虑关联性，而且只执行一次就可以

还可以进一步简化，在 IN 中写多个字段的组合

![](https://mmbiz.qpic.cn/mmbiz_jpg/dkwuWwLoRK9mWPlj4ia8tk5ctiaPYfia5KWqsDSGW6XWHADCvbmHDFCG8rlZFNHOiabm4HcPsgpmW51uuP2lyHQNOw/640?wx_fmt=jpeg)

简化后，不用担心连接字段时出现的类型转换问题，也不会对字段进行加工，因此可以使用索引

先进行连接再进行聚合

连接和聚合同时使用时，先进行连接操作可以避免产生中间表

合理地使用视图

视图是非常方便的工具，我们在日常工作中经常用到

但是，如果没有经过深入思考就定义复杂的视图，可能会带来巨大的性能问题

特别是视图的定义语句中包含以下运算的时候，SQL 会非常低效，执行速度也会变得非常慢

![](https://mmbiz.qpic.cn/mmbiz_jpg/dkwuWwLoRK9mWPlj4ia8tk5ctiaPYfia5KWFyJdjlnia3fEOE2BZEqZYcNnjQHl2PGfsMWibhAWV1FSFh98vLaedURA/640?wx_fmt=jpeg)

### **小结下**

文中虽然列举了几个要点，但其实优化的核心思想只有一个，那就是找出性能瓶颈所在，然后解决它；

其实不只是数据库和 SQL，计算机世界里容易成为性能瓶颈的也是对硬盘，也就是文件系统的访问（因此可以通过增加内存，或者使用访问速度更快的硬盘等方法来提升性能）

不管是减少排序还是使用索引，亦或是避免临时表的使用，其本质都是为了减少对硬盘的访问！

**四 高斯数据库特性为啥优异**
-----------------

### 首先，能释放 CPU 多核心的计算资源

众所周知，软件计算能力的提升一方面得益于 CPU 硬件能力的增强，另一方面也得益于软件设计层面能够充分利用 CPU 的计算资源。当前处理器普遍采用多核设计，GaussDB (for MySQL) 单个节点最多可以支持 64 核的 CPU。单线程查询的方式至多能用满一个核的 CPU 资源，性能提升程度有限，远远无法满足企业大数据量查询场景下对降低时延的要求。因此，复杂的查询分析型计算过程必须考虑充分利用 CPU 的多核计算资源，让多个核参与到并行计算任务中才能大幅度提升查询计算的处理效率；

![](https://mmbiz.qpic.cn/mmbiz_jpg/dkwuWwLoRK9mWPlj4ia8tk5ctiaPYfia5KWzP7WE0nt9d3iaNw5wiaiavtKbfkib0Ys9PfYovWfLmdpCjCfdRyhjcQyRA/640?wx_fmt=jpeg)

下图是使用 CPU 多核资源并行计算一个表的 count (_) 过程的例子：表数据进行切块后分发给多个核进行并行计算，** 每个核计算部分数据得到一个中间 count (_) 结果 **，并在最后阶段将所有中间结果进行聚合得到最终结果

![](https://mmbiz.qpic.cn/mmbiz_jpg/dkwuWwLoRK9mWPlj4ia8tk5ctiaPYfia5KW3hbdQzVbegffEty6sH04D4nfLBoPcnpykK42umI5fPNFO40ibqhvibVQ/640?wx_fmt=jpeg)

### 然后，是并行查询

GaussDB (for MySQL) 支持并行执行的查询方式，用于降低分析型查询场景的处理时间，满足企业级应用对查询低时延的要求。如前面所述，并行查询的基本实现原理是将查询任务进行切分并分发到多个 CPU 核上进行计算，充分利用 CPU 的多核计算资源来缩短查询时间。并行查询的性能提升倍数，理论上与 CPU 的核数正相关，就是说并行度越高能够使用的 CPU 核数就越多，性能提升的倍数也就越高；

下图展示的是：在 GaussDB (for MySQL) 的 64U 实例上查询 100G 数据量的 COUNT (*) 查询耗时，不同的查询并发度分别对应不同耗时，并发度越高对应的查询耗时越短

![](https://mmbiz.qpic.cn/mmbiz_jpg/dkwuWwLoRK9mWPlj4ia8tk5ctiaPYfia5KWWwNBzRaUtWYQoIpvdNOiaHH62iay9ygoYxsmb2jM7SfIQDjy2PqNauMA/640?wx_fmt=jpeg)

它支持多种类型的并行查询算子，以满足客户各种不同复杂查询场景。当前最新版本（2021-9）已经支持的并行查询场景包括：

*   主键查询、二级索引查询
    
*   主键扫描、索引扫描、范围扫描、索引等值查询，索引逆向查询
    
*   并行条件过滤（where/having）、投影计算
    
*   并行多表 JOIN（包括 HashJoin、NestLoopJoin、SemiJoin 等）查询
    
*   并行聚合函数运算，包括 SUM/AVG/COUNT/BIT_AND/BIT_OR/BIT_XOR 等
    
*   并行表达式运算，包括算术运算、逻辑运算、一般函数运算及混合运算等
    
*   并行分组 group by、排序 order by、limit/offset、distinct 运算
    
*   并行 UNION、子查询、视图查询
    
*   并行分区表查询
    
*   并行查询支持的数据类型包括：整型、字符型、时间类型、浮点型等等
    
*   其他查询
    

下图是 GaussDB (for MySQL) 并行查询针对 TPC-H 的 22 条查询场景所做的性能测试结果，测试数据量为 100G，并发线程数据是 32。下图展示了并行查询相比传统 MySQL 单线程查询的性能提升情况：32 并行执行下，单表复杂查询最高提升 26 倍性能，普遍提升 20 + 倍性能。多表 JOIN 复杂查询最高提升近 27 倍性能，普遍提升 10 + 倍性能，子查询性能也有较大提升；

![](https://mmbiz.qpic.cn/mmbiz_jpg/dkwuWwLoRK9mWPlj4ia8tk5ctiaPYfia5KWZkic1xics4wdw3A2ZNyrbpc57DtlhmiahUR3ThC1a3Pao5RUlzhXAgeIA/640?wx_fmt=jpeg)

### **总而言之**

GaussDB (for MySQL) 并行查询充分调用了 CPU 的多核计算资源，极大降低了分析型查询场景的处理时间，大幅度提升了数据库性能，可以很好的满足客户多种复杂查询场景的低时延要求。

END  

* * *