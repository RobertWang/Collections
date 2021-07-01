> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247518712&idx=3&sn=4c9f588fb29eb45a44867c2d8ccdf878&chksm=ce0e3e7bf979b76d5527dc6ca641d287ee8ef828641b2231176bfcdc74776967095e8b4fa726&mpshare=1&scene=1&srcid=0701slRfAzUiQh309or4twPe&sharer_sharetime=1625124062702&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

来源 | https://www.oschina.net/translate/10-common-mistakes-java-developers-make-when-writing-sql

*   技能（任何人都能容易学会命令式编程）
    
*   模式（有些人用 “模式 - 模式”, 举个例子, 模式可以应用到任何地方，而且都可以归为某一类模式）
    
*   心境（首先，要写个好的面向对象程序是比命令式程序难的多，你得花费一些功夫）
    

*   好好的训练你自己。当你写 SQL 时要不停得想到 NULL 的用法：
    
*   这个 NULL 完整性约束条件是正确的？
    
*   NULL 是否影响到结果？
    

但是一些 SQL 数据库支持先进的（而且是 SQL 标准支持的）OLAP 特性, 这一特性表现更好而且写起来也更加方便。一个（并不怎么标准的）例子就 是 Oracle 超棒的 MODEL 分句。只让数据库来做处理然后只把结果带到 Java 内存中吧。因为毕竟所有非常聪明的家伙已经对这些昂贵的产品进行了优 化。因此实际上, 通过将 OLAP 移到数据库，你将获得一下两项好处：

*   便利性。这比在 Java 中编写正确的 SQL 可能更加的容易。
    
*   性能表现。数据库应该比你的算法处理起来更加快. 而且更加重要的是, 你不必再去传递数百万条记录了。
    

**解决方法：**

每次你使用 Java 实现一个以数据为中心的算法时，问问自己：有没有一种方法可以让数据库代替为我做这种麻烦事。Spring Boot 学习笔记，这个分享给你学习下。

**3、使用 UNION 代替 UNION ALL**

*   UNION ALL（允许重复）
    
*   UNION （去除了重复）
    

移除重复行不仅很少需要（有时甚至是错的），而且对于带很多行的大数据集合会相当慢，因为两个子 select 需要排序，而且每个元组也需要和它的子序列元组比较。

注意即使 SQL 标准规定了 INTERSECT ALL 和 EXCEPT ALL，很少数据库会实现这些没用的集合操作符。

**解决方法：**

每次写 UNION 语句时，考虑实际上是否需要 UNION ALL 语句。

**4、通过 JDBC 分页技术给大量的结果进行分页操作**

大部分的数据库都会支持一些分页命令实现分页效果，譬如 LIMIT..OFFSET,TOP..START AT,OFFSET..FETCH 语句等。即使没有支持这些语句的数据库，仍有可能对 ROWNUM（Oracle）或者是 ROW NUMBER()、OVER() 过滤（DB2、SQL Server2008 等），这些比在内存中实现分页更快速。在处理大量数据中，效果尤其明显。

**解决方法：**

仅仅使用这些语句，那么一个工具（例如 JOOQ）就可以模拟这些语句的操作。

**5、在 Java 内存中加入数据**

从 SQL 的初期开始，当在 SQL 中使用 JOIN 语句时，一些开发者仍旧有不安的感觉。这是源自对加入 JOIN 后会变慢的固有恐惧。

假如基于成本的 优化选择去实现嵌套循环，在创建一张连接表源前，可能加载所有的表在数据库内存中，这可能是真的。但是这事发生的概率太低了。通过合适的预测，约束和索 引，合并连接和哈希连接的操作都是相当的快。这完全是是关于正确元数据（在这里我不能够引用 Tom Kyte 的太多）。而且，可能仍然有不少的 Java 开发人员加载两张表通过分开查询到一个映射中，并且在某种程度上把他们加到了内存当中。

**解决方法：**

假如你在各个步骤中有从各种表的查询操作，好好想想是否可以表达你的查询操作在单条语句中。

**6、在一个临时的笛卡尔积集合中使用 DISTINCT 或 UNION 消除重复项**

通过复杂的连接，人们可能会对 SQL 语句中扮演关键角色的所有关系失去概念。特别的，如果这涉及到多列外键关系的话，很有可能会忘记在 JOIN .. ON 子句中增加相关的判断。这会导致重复的记录，但或许只是在特殊的情况下。有些开发者因此可能选择 DISTINCT 来消除这些重复记录。从三个方面来说 这是错误的：

*   它（也许）解决了表面症状但并没有解决问题。它也有可能无法解决极端情况下的症状。
    
*   对具有很多列的庞大的结果集合来说它很慢。DISTINCT 要执行 ORDER BY 操作来消除重复。
    
*   对庞大的笛卡尔积集合来说它很慢，还是需要加载很多的数据到内存中。
    

**解决方法：**

根据经验，如果你获得了不需要的重复记录，还是检查你的 JOIN 判断吧。可能在某个地方有一个很难觉察的笛卡尔积集合。

**7、不使用 MERGE 语句**

这并不是一个过失，但是可能是缺少知识或者对于强悍的 MERGE 语句信心不足。一些数据库理解其它形式的更新插入（UPSERT）语句， 如 MYSQL 的重复主键更新语句，但是 MERGE 在数据库中确是很强大，很重要，以至于大肆扩展 SQL 标准，例如 SQL SERVER。

**解决方法：**

如果你使用像联合 INSERT 和 UPDATE 或者联合 SELECT .. FOR UPDATE 然后在 INSERT 或 UPDATE 等更新插入时，请三思。你完全可以使用一个更简单的 MERGE 语句来远离冒险竞争条件。2021 最新 Java 面试题出炉！

**8、使用聚合函数代替窗口函数（window functions）**

在介绍窗口函数之前，在 SQL 中聚合数据意味着使用 GROUP BY 语句与聚合函数相映射。在很多情形下都工作得很好，如聚合数据需要浓缩常规数据，那么就在 join 子查询中使用 group 查询。

但是在 SQL2003 中定义了窗口函数，这个在很多主流数据库都实现了它。窗口函数能够在结果集上聚合数据，但是却没有分组。事实上，每个窗口函数都有自己的、独立的 PARTITION BY 语句，这个工具对于显示报告太好了。

使用窗口函数：

*   使 SQL 更易读（但在子查询中没有 GROUP BY 语句专业）
    
*   提升性能，像关系数据库管理系统能够更容易优化窗口函数
    

**解决方法：**

当你在子查询中使用 GROUP BY 语句时，请再三考虑是否可以使用窗口函数完成。

**9、使用内存间接排序**

SQL 的 ORDER BY 语句支持很多类型的表达式，包括 CASE 语句，对于间接排序十分有用。你可能重来不会在 Java 内存中排序数据，因为你会想：

*   SQL 排序很慢
    
*   SQL 排序办不到
    

**解决方法：**

如果你在内存中排序任何 SQL 数据，请再三考虑，是否不能在数据库中排序。这对于数据库分页数据十分有用。

**10、一条一条地插入大量记录**

JDBC“懂” 批处理（batch），你应该不会忘了它。不要使用 INSERT 语句来一条一条的出入成千上万的记录，（因为）每次都会创建一个新 的 PreparedStatement 对象。如果你的所有记录都插入到同一个表时，那么就创建一个带有一条 SQL 语句以及附带很多值集合的插入批处理语 句。你可能需要在达到一定量的插入记录后才提交来保证 UNDO 日志瘦小，这依赖于你的数据库和数据库设置。

**解决方法：**

总是使用批处理插入大量数据。

推荐阅读

_1._ [GitHub 上有什么好玩的项目？](http://mp.weixin.qq.com/s?__biz=MzUxNjg4NDEzNA==&mid=2247498662&idx=1&sn=0087c4f3b79ba3420e917e9b42d45eda&chksm=f9a2286fced5a1794eb9a73d0be7c2e16eaceabf3a0420647c40cb4202bd116d9a15dd57c008&scene=21#wechat_redirect)

2. [Linux 运维必备 150 个命令汇总](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247518355&idx=2&sn=23c5bc3477a0185b517840b8318fb166&chksm=ce0e3f10f979b606a9915aea543e34da632e6af092d65f0a071845e2c079cf6fe7077bf2315c&scene=21#wechat_redirect)

_3._ [SpringSecurity + JWT 实现单点登录](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247516734&idx=1&sn=966d166d3052f10b9310ff4014733669&chksm=ce0e35bdf979bcabbb75366e1bad32bdbf8f41d1dc80b8be6e39225bd708913ce2c93472f2fc&scene=21#wechat_redirect)

_4._ [100 道 Linux 常见面试题](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247516664&idx=1&sn=0dae1a8cf99c4bde8fd8c9c197769009&chksm=ce0e367bf979bf6dec769decc0194bb36de4caeae7947671aba1852fe6bac8e5dab7d20916a5&scene=21#wechat_redirect)