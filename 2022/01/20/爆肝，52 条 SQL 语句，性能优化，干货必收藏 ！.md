> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Dxu4s98INmQBI22P35UHaA)

2，应尽量避免在 where 子句中对字段进行 null 值判断，创建表时 NULL 是默认值，但大多数时候应该使用 NOT NULL，或者使用一个特殊的值，如 0，-1 作为默 认值。

3，应尽量避免在 where 子句中使用!= 或 <> 操作符， MySQL 只有对以下操作符才使用索引：<，<=，=，>，>=，BETWEEN，IN，以及某些时候的 LIKE。

4，应尽量避免在 where 子句中使用 or 来连接条件， 否则将导致引擎放弃使用索引而进行全表扫描， 可以 使用 UNION 合并查询：select id from t where num=10 union all select id from t where num=20

5，in 和 not in 也要慎用，否则会导致全表扫描，对于连续的数值，能用 between 就不要用 in 了：Select id from t where num between 1 and 3

16，使用表的别名 (Alias)：当在 SQL 语句中连接多个表时, 请使用表的别名并把别名前缀于每个 Column 上. 这样一来, 就可以减少解析的时间并减少那些由 Column 歧义引起的语法错误。

17，使用 “临时表” 暂存中间结果

简化 SQL 语句的重要方法就是采用临时表暂存中间结果，但是，临时表的好处远远不止这些，将临时结果暂存在临时表，后面的查询就在 tempdb 中了，这可以避免程序中多次扫描主表，也大大减少了程序执行中 “共享锁” 阻塞“更新锁”，减少了阻塞，提高了并发性能。

18，一些 SQL 查询语句应加上 nolock，读、写是会相互阻塞的，为了提高并发性能，对于一些查询，可以加上 nolock，这样读的时候可以允许写，但缺点是可能读到未提交的脏数据。使用 nolock 有 3 条原则。查询的结果用于 “插、删、改” 的不能加 nolock ！查询的表属于频繁发生页分裂的，慎用 nolock ！使用临时表一样可以保存“数据前影”，起到类似 Oracle 的 undo 表空间的功能，能采用临时表提高并发性能的，不要用 nolock 。

19，常见的简化规则如下：不要有超过 5 个以上的表连接（JOIN），考虑使用临时表或表变量存放中间结果。少用子查询，视图嵌套不要过深, 一般视图嵌套不要超过 2 个为宜。

20，将需要查询的结果预先计算好放在表中，查询的时候再 Select。这在 SQL7.0 以前是最重要的手段。例如医院的住院费计算。

21，用 OR 的字句可以分解成多个查询，并且通过 UNION 连接多个查询。他们的速度只同是否使用索引有关, 如果查询需要用到联合索引，用 UNION all 执行的效率更高. 多个 OR 的字句没有用到索引，改写成 UNION 的形式再试图与索引匹配。一个关键的问题是否用到索引。

22，在 IN 后面值的列表中，将出现最频繁的值放在最前面，出现得最少的放在最后面，减少判断的次数。

23，尽量将数据的处理工作放在服务器上，减少网络的开销，如使用存储过程。存储过程是编译好、优化过、并且被组织到一个执行规划里、且存储在数据库中的 SQL 语句，是控制流语言的集合，速度当然快。反复执行的动态 SQL, 可以使用临时存储过程，该过程（临时表）被放在 Tempdb 中。

24，当服务器的内存够多时，配制线程数量 = 最大连接数 + 5，这样能发挥最大的效率；否则使用 配制线程数量 < 最大连接数启用 SQL SERVER 的线程池来解决, 如果还是数量 = 最大连接数 + 5，严重的损害服务器的性能。

25，查询的关联同写的顺序

select a.personMemberID, * from chineseresume a,personmember b where personMemberID = b.referenceid and a.personMemberID = ‘JCNPRH39681’ （A = B ,B = ‘号码’）

select a.personMemberID, * from chineseresume a,personmember b where a.personMemberID = b.referenceid and a.personMemberID = ‘JCNPRH39681’ and b.referenceid = ‘JCNPRH39681’ （A = B ,B = ‘号码’， A = ‘号码’）

select a.personMemberID, * from chineseresume a,personmember b where b.referenceid = ‘JCNPRH39681’ and a.personMemberID = ‘JCNPRH39681’ （B = ‘号码’， A = ‘号码’）

26，尽量使用 exists 代替 select count(1) 来判断是否存在记录，count 函数只有在统计表中所有行数时使用，而且 count(1) 比 count(*) 更有效率。

27，尽量使用 “>=”，不要使用 “>”。

28，索引的使用规范：索引的创建要与应用结合考虑，建议大的 OLTP 表不要超过 6 个索引；尽可能的使用索引字段作为查询条件，尤其是聚簇索引，必要时可以通过 index index_name 来强制指定索引；避免对大表查询时进行 table scan，必要时考虑新建索引；在使用索引字段作为条件时，如果该索引是联合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使用；要注意索引的维护，周期性重建索引，重新编译存储过程。

29，下列 SQL 条件语句中的列都建有恰当的索引，但执行速度却非常慢：

SELECT * FROM record WHERE substrINg(card_no,1,4)=’5378’ (13 秒)

SELECT * FROM record WHERE amount/30< 1000 （11 秒）

SELECT * FROM record WHERE convert(char(10),date,112)=’19991201’ （10 秒）

分析：

WHERE 子句中对列的任何操作结果都是在 SQL 运行时逐列计算得到的，因此它不得不进行表搜索，而没有使用该列上面的索引；如果这些结果在查询编译时就能得到，那么就可以被 SQL 优化器优化，使用索引，避免表搜索，因此将 SQL 重写成下面这样：

GROUP BY JOB

HAVING JOB =’PRESIDENT’

OR JOB =’MANAGER’

高效:

SELECT JOB , AVG(SAL)

FROM EMP

WHERE JOB =’PRESIDENT’

OR JOB =’MANAGER’

GROUP BY JOB

34，sql 语句用大写，因为 oracle 总是先解析 sql 语句，把小写的字母转换成大写的再执行。

35，别名的使用，别名是大型数据库的应用技巧，就是表名、列名在查询中以一个字母为别名，查询速度要比建连接表快 1.5 倍。

36，避免死锁，在你的存储过程和触发器中访问同一个表时总是以相同的顺序; 事务应经可能地缩短，在一个事务中应尽可能减少涉及到的数据量; 永远不要在事务中等待用户输入。

37，避免使用临时表，除非却有需要，否则应尽量避免使用临时表，相反，可以使用表变量代替; 大多数时候 (99%)，表变量驻扎在内存中，因此速度比临时表更快，临时表驻扎在 TempDb 数据库中，因此临时表上的操作需要跨数据库通信，速度自然慢。

38，最好不要使用触发器，触发一个触发器，执行一个触发器事件本身就是一个耗费资源的过程; 如果能够使用约束实现的，尽量不要使用触发器; 不要为不同的触发事件 (Insert，Update 和 Delete) 使用相同的触发器; 不要在触发器中使用事务型代码。

39，索引创建规则：

表的主键、外键必须有索引；

数据量超过 300 的表应该有索引；

经常与其他表进行连接的表，在连接字段上应该建立索引；

经常出现在 Where 子句中的字段，特别是大表的字段，应该建立索引；

索引应该建在选择性高的字段上；

索引应该建在小字段上，对于大的文本字段甚至超长字段，不要建索引；

复合索引的建立需要进行仔细分析，尽量考虑用单字段索引代替；

正确选择复合索引中的主列字段，一般是选择性较好的字段；

复合索引的几个字段是否经常同时以 AND 方式出现在 Where 子句中？单字段查询是否极少甚至没有？如果是，则可以建立复合索引；否则考虑单字段索引；

如果复合索引中包含的字段经常单独出现在 Where 子句中，则分解为多个单字段索引；

如果复合索引所包含的字段超过 3 个，那么仔细考虑其必要性，考虑减少复合的字段；

如果既有单字段索引，又有这几个字段上的复合索引，一般可以删除复合索引；

频繁进行数据操作的表，不要建立太多的索引；

删除无用的索引，避免对执行计划造成负面影响；

表上建立的每个索引都会增加存储开销，索引对于插入、删除、更新操作也会增加处理上的开销。另外，过多的复合索引，在有单字段索引的情况下，一般都是没有存在价值的；相反，还会降低数据增加删除时的性能，特别是对频繁更新的表来说，负面影响更大。

尽量不要对数据库中某个含有大量重复的值的字段建立索引。

40，mysql 查询优化总结：使用慢查询日志去发现慢查询，使用执行计划去判断查询是否正常运行，总是去测试你的查询看看是否他们运行在最佳状态下。久而久之性能总会变化，避免在整个表上使用 count(*), 它可能锁住整张表，使查询保持一致以便后续相似的查询可以使用查询缓存

，在适当的情形下使用 GROUP BY 而不是 DISTINCT，在 WHERE, GROUP BY 和 ORDER BY 子句中使用有索引的列，保持索引简单, 不在多个索引中包含同一个列，有时候 MySQL 会使用错误的索引, 对于这种情况使用 USE INDEX，检查使用 SQL_MODE=STRICT 的问题，对于记录数小于 5 的索引字段，在 UNION 的时候使用 LIMIT 不是是用 OR。

为了 避免在更新前 SELECT，使用 INSERT ON DUPLICATE KEY 或者 INSERT IGNORE , 不要用 UPDATE 去实现，不要使用 MAX, 使用索引字段和 ORDER BY 子句，LIMIT M，N 实际上可以减缓查询在某些情况下，有节制地使用，在 WHERE 子句中使用 UNION 代替子查询，在重新启动的 MySQL，记得来温暖你的数据库，以确保您的数据在内存和查询速度快，考虑持久连接，而不是多个连接，以减少开销，基准查询，包括使用服务器上的负载，有时一个简单的查询可以影响其他查询，当负载增加您的服务器上，使用 SHOW PROCESSLIST 查看慢的和有问题的查询，在开发环境中产生的镜像数据中 测试的所有可疑的查询。

41，MySQL 备份过程:

从二级复制服务器上进行备份。在进行备份期间停止复制，以避免在数据依赖和外键约束上出现不一致。彻底停止 MySQL，从数据库文件进行备份。

如果使用 MySQL dump 进行备份，请同时备份二进制日志文件 – 确保复制没有中断。不要信任 LVM 快照，这很可能产生数据不一致，将来会给你带来麻烦。为了更容易进行单表恢复，以表为单位导出数据 – 如果数据是与其他表隔离的。

当使用 mysqldump 时请使用 –opt。在备份之前检查和优化表。为了更快的进行导入，在导入时临时禁用外键约束。

为了更快的进行导入，在导入时临时禁用唯一性检测。在每一次备份后计算数据库，表以及索引的尺寸，以便更够监控数据尺寸的增长。

通过自动调度脚本监控复制实例的错误和延迟。定期执行备份。

42，查询缓冲并不自动处理空格，因此，在写 SQL 语句时，应尽量减少空格的使用，尤其是在 SQL 首和尾的空格 (因为，查询缓冲并不自动截取首尾空格)。

43，member 用 mid 做標準進行分表方便查询么？一般的业务需求中基本上都是以 username 为查询依据，正常应当是 username 做 hash 取模来分表吧。分表的话 mysql 的 partition 功能就是干这个的，对代码是透明的；

在代码层面去实现貌似是不合理的。

44，我们应该为数据库里的每张表都设置一个 ID 做为其主键，而且最好的是一个 INT 型的（推荐使用 UNSIGNED），并设置上自动增加的 AUTO_INCREMENT 标志。

45，在所有的存储过程和触发器的开始处设置 SET NOCOUNT ON ，在结束时设置 SET NOCOUNT OFF 。

无需在执行存储过程和触发器的每个语句后向客户端发送 DONE_IN_PROC 消息。

46，MySQL 查询可以启用高速查询缓存。这是提高数据库性能的有效 Mysql 优化方法之一。当同一个查询被执行多次时，从缓存中提取数据和直接从数据库中返回数据快很多。

47，EXPLAIN SELECT 查询用来跟踪查看效果

使用 EXPLAIN 关键字可以让你知道 MySQL 是如何处理你的 SQL 语句的。这可以帮你分析你的查询语句或是表结构的性能瓶颈。EXPLAIN 的查询结果还会告诉你你的索引主键被如何利用的，你的数据表是如何被搜索和排序的…… 等等，等等。

48，当只要一行数据时使用 LIMIT 1

当你查询表的有些时候，你已经知道结果只会有一条结果，但因为你可能需要去 fetch 游标，或是你也许会去检查返回的记录数。在这种情况下，加上 LIMIT 1 可以增加性能。这样一样，MySQL 数据库引擎会在找到一条数据后停止搜索，而不是继续往后查少下一条符合记录的数据。

49, 选择表合适存储引擎：

myisam: 应用时以读和插入操作为主，只有少量的更新和删除，并且对事务的完整性，并发性要求不是很高的。

Innodb：事务处理，以及并发条件下要求数据的一致性。除了插入和查询外，包括很多的更新和删除。（Innodb 有效地降低删除和更新导致的锁定）。对于支持事务的 InnoDB 类型的表来说，影响速度的主要原因是 AUTOCOMMIT 默认设置是打开的，而且程序没有显式调用 BEGIN 开始事务，导致每插入一条都自动提交，严重影响了速度。可以在执行 sql 前调用 begin，多条 sql 形成一个事物（即使 autocommit 打开也可以），将大大提高性能。

50, 优化表的数据类型, 选择合适的数据类型：

原则：更小通常更好，简单就好，所有字段都得有默认值, 尽量避免 null。

例如：数据库表设计时候更小的占磁盘空间尽可能使用更小的整数类型.(mediumint 就比 int 更合适)

比如时间字段：datetime 和 timestamp, datetime 占用 8 个字节，而 timestamp 占用 4 个字节，只用了一半，而 timestamp 表示的范围是 1970—2037 适合做更新时间

MySQL 可以很好的支持大数据量的存取，但是一般说来，数据库中的表越小，在它上面执行的查询也就会越快。

因此，在创建表的时候，为了获得更好的性能，我们可以将表中字段的宽度设得尽可能小。例如，

在定义邮政编码这个字段时，如果将其设置为 CHAR(255), 显然给数据库增加了不必要的空间，

甚至使用 VARCHAR 这种类型也是多余的，因为 CHAR(6) 就可以很好的完成任务了。同样的，如果可以的话，

我们应该使用 MEDIUMINT 而不是 BIGIN 来定义整型字段。

应该尽量把字段设置为 NOT NULL，这样在将来执行查询的时候，数据库不用去比较 NULL 值。

对于某些文本字段，例如 “省份” 或者“性别”，我们可以将它们定义为 ENUM 类型。因为在 MySQL 中，ENUM 类型被当作数值型数据来处理，

而数值型数据被处理起来的速度要比文本类型快得多。这样，我们又可以提高数据库的性能。

51， 字符串数据类型：char，varchar，text 选择区别

52，任何对列的操作都将导致表扫描，它包括数据库函数、计算表达式等等，查询时要尽可能将操作移至等号右边。

> 作者 |  SimpleWu
> 
> 来源 |  cnblogs.com/SimpleWu/p/9929043.html

(完)

**推荐阅读**：

[Spring Boot + EasyExcel 导入导出，好用到爆，可以扔掉 POI 了！](http://mp.weixin.qq.com/s?__biz=Mzg5NTUxNzg0OA==&mid=2247484290&idx=1&sn=992760c5ce6458af886be9488374f537&chksm=c00e5675f779df63b9fb8e43a78dbd066505fae4070e64b790bd3f8e0c8ca375a446199e0c0f&scene=21#wechat_redirect)  

[扔掉工具类，Mybatis 一个简单配置搞定数据加密解密！](http://mp.weixin.qq.com/s?__biz=Mzg5NTUxNzg0OA==&mid=2247484284&idx=1&sn=ec795f44077eb0fc139d848659fc6fb6&chksm=c00e568bf779df9d640552029a2df816c07e7db6937e66160cbf255f0ea0ef79ed520eca1f25&scene=21#wechat_redirect)  

[还在用策略模式解决 if-else？Map + 函数式接口方法才是 YYDS！](http://mp.weixin.qq.com/s?__biz=Mzg5NTUxNzg0OA==&mid=2247484276&idx=1&sn=a0c7db57c9ddb4d514ba7d2ca6089e44&chksm=c00e5683f779df953c92c31645649133b549fc91db07d89e1713b9efe00de4e23cafe091b2d4&scene=21#wechat_redirect)  

[没想到 Spring Boot + Vue 竟如此简单！](http://mp.weixin.qq.com/s?__biz=Mzg5NTUxNzg0OA==&mid=2247484268&idx=1&sn=99f77a2e7a9a74044f3600645d533fa2&chksm=c00e569bf779df8d067a0569bfb07df6479b0c275d9044e17e905646dd1eaeecee90308cb434&scene=21#wechat_redirect)  

---

![](https://mmbiz.qpic.cn/mmbiz_jpg/f4Mzmh8H1V77TRh6OHib5kWbRRttcxEF7ibbkzZ9Y0Yo7gOdPePZAzsYcxVlNgGtCNdic89XKnoZh3aLYOQ0jgOwg/640?wx_fmt=jpeg)

**点赞**、**在看** ![](https://mmbiz.qpic.cn/mmbiz_png/f4Mzmh8H1V77TRh6OHib5kWbRRttcxEF7MdZt8VPuVTzGgxbtYBPCWrJYIkz8N31nLxqos5eaU0dJiaYWicAibDOibg/640?wx_fmt=png)