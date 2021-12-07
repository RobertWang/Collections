> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA5OTAyNzQ2OA==&mid=2649745100&idx=2&sn=6c59819efac97d529a1eb16125ed5139&chksm=8893d9afbfe450b98969667819941aa34f52705db01679d70b30fa15cc46d203b32b73668806&mpshare=1&scene=1&srcid=1207MTEwdcs6DUcknHQRfoKm&sharer_sharetime=1638847536928&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

MySQL 在我们的开发中基本每天都要面对的，作为开发中的数据的来源，MySQL 承担者存储数据和读写数据的职责。因为学习和了解 MySQL 是至关重要的，那么当我们在客户端发起一个 SQL 到出现详细的查询数据，这其中究竟经历了什么样的过程？MySQL 服务端是如何处理请求的，又是如何执行 SQL 语句的？本篇博客将来探讨这些问题。

MySQL 执行过程

MySQL 整体的执行过程如下图所示：

  

![](https://mmbiz.qpic.cn/mmbiz_png/A1HKVXsfHNn3icELYmTSxVGj0DeuafSq8tgkTVw6d3AUzffxV9PBmTLyR16SZeZQNLBFbKGNdicS2YgfYq471cXQ/640?wx_fmt=png)

  

**连接器**

  

连接器的主要职责就是：

  

1、负责与客户端的通信，是半双工模式，这就意味着某一固定时刻只能由客户端向服务器请求或者服务器向客户端发送数据，而不能同时进行，其中 MySQL 在与客户端连接 TC/IP 的。

  

2、验证请求用户的账户和密码是否正确，如果账户和密码错误，会报错：Access denied for user 'root'@'localhost' (using password: YES)

  

3、如果用户的账户和密码验证通过，会在 MySQL 自带的权限表中查询当前用户的权限。

  

MySQL 中存在 4 个控制权限的表，分别为 user 表，db 表，tables_priv 表，columns_priv 表：

  

*   user 表：存放用户账户信息以及全局级别（所有数据库）权限，决定了来自哪些主机的哪些用户可以访问数据库实例
    
*   db 表：存放数据库级别的权限，决定了来自哪些主机的哪些用户可以访问此数据库
    
*   tables_priv 表：存放表级别的权限，决定了来自哪些主机的哪些用户可以访问数据库的这个表
    
*   columns_priv 表：存放列级别的权限，决定了来自哪些主机的哪些用户可以访问数据库表的这个字段
    

  

MySQL 权限表的验证过程为：

  

1、 先从 user 表中的 Host，User，Password 这 3 个字段中判断连接的 IP、用户名、密码是否存在，存在则通过验证。

  

2、通过身份认证后，进行权限分配，按照 user，db，tables_priv，columns_priv 的顺序进行验证。即先检查全局权限表 user，如果 user 中对应的权限为 Y，则此用户对所有数据库的权限都为 Y，将不再检查 db，tables_priv，columns_priv；如果为 N，则到 db 表中检查此用户对应的具体数据库，并得到 db 中为 Y 的权限；如果 db 中为 N，则检查 tables_priv 中此数据库对应的具体表，取得表中的权限 Y，以此类推。

  

3、如果在任何一个过程中权限验证不通过，都会报错。

  

**缓存**

  

MySQL 的缓存主要的作用是为了提升查询的效率，缓存以 key 和 value 的哈希表形式存储，key 是具体的 SQL 语句，value 是结果的集合。如果无法命中缓存，就继续走到分析器的这一步，如果命中缓存就直接返回给客户端。不过需要注意的是在 MySQL 的 8.0 版本以后，缓存被官方删除掉了。之所以删除掉，是因为查询缓存的失效非常频繁，如果在一个写多读少的环境中，缓存会频繁的新增和失效。对于某些更新压力大的数据库来说，查询缓存的命中率会非常低，MySQL 为了维护缓存可能会出现一定的伸缩性的问题，目前在 5.6 的版本中已经默认关闭了，比较推荐的一种做法是将缓存放在客户端，性能大概会提升 5 倍左右。

  

**分析器**

  

分析器的主要作用是将客户端发过来的 SQL 语句进行分析，这将包括预处理与解析过程，在这个阶段会解析 SQL 语句的语义，并进行关键词和非关键词进行提取、解析，并组成一个解析树。具体的关键词包括不限定于以下：select/update/delete/or/in/where/group by/having/count/limit 等。如果分析到语法错误，会直接给客户端抛出异常：“ERROR:You have an error in your SQL syntax.”。

  

比如：select * from user where userId =1234;

  

在分析器中就通过语义规则器将 select from where 这些关键词提取和匹配出来，MySQL 会自动判断关键词和非关键词，将用户的匹配字段和自定义语句识别出来。这个阶段也会做一些校验：比如校验当前数据库是否存在 user 表，同时假如 user 表中不存在 userId 这个字段同样会报错：“unknown column in field list.”。

  

**优化器**

  

能够进入到优化器阶段表示 SQL 是符合 MySQL 的标准语义规则的并且可以执行的，此阶段主要是进行 SQL 语句的优化，会根据执行计划进行最优的选择，匹配合适的索引，选择最佳的执行方案。比如一个典型的例子是这样的：

  

表 T，对 A、B、C 列建立联合索引，在进行查询的时候，当 SQL 查询到的结果是：select xx where B=x and A=x and C=x，很多人会以为是用不到索引的，但其实会用到，虽然索引必须符合最左原则才能使用，但是本质上，优化器会自动将这条 SQL 优化为：where A=x and B=x and C=X，这种优化会为了底层能够匹配到索引，同时在这个阶段是自动按照执行计划进行预处理，MySQL 会计算各个执行方法的最佳时间，最终确定一条执行的 SQL 交给最后的执行器。

  

**执行器**

  

在执行器的阶段，此时会调用存储引擎的 API，API 会调用存储引擎，主要有一下存储的引擎，不过常用的还是 myisam 和 innodb：

  

![](https://mmbiz.qpic.cn/mmbiz_png/A1HKVXsfHNn3icELYmTSxVGj0DeuafSq8f9G5eNa5YrhGJuWHzPhfLgcRPB8ZicdlCGldicG9QafbjV4uicTIF6v3w/640?wx_fmt=png)

  

引擎以前的名字叫做：表处理器（其实这个名字我觉得更能表达它存在的意义）负责对具体的数据文件进行操作，对 SQL 的语义比如 select 或者 update 进行分析，执行具体的操作。在执行完以后会将具体的操作记录到 binlog 中，需要注意的一点是：select 不会记录到 binlog 中，只有 update/delete/insert 才会记录到 binlog 中。而 update 会采用两阶段提交的方式，记录都 redolog 中。

执行的状态

可以通过命令：show full processlist，展示所有的处理进程，主要包含了以下的状态，表示服务器处理客户端的状态，状态包含了从客户端发起请求到后台服务器处理的过程，包括加锁的过程、统计存储引擎的信息，排序数据、搜索中间表、发送数据等。囊括了所有的 MySQL 的所有状态, 其中具体的含义如下图：

  

![](https://mmbiz.qpic.cn/mmbiz_png/A1HKVXsfHNn3icELYmTSxVGj0DeuafSq8kb4PuZclXkzdnw3N4rhQjVyTO6WMb0rWg9pRiaQPoxyzmw3JfQDGHTw/640?wx_fmt=png)

SQL 的执行顺序

事实上，SQL 并不是按照我们的书写顺序来从前往后、左往右依次执行的，它是按照固定的顺序解析的，主要的作用就是从上一个阶段的执行返回结果来提供给下一阶段使用，SQL 在执行的过程中会有不同的临时中间表，一般是按照如下顺序：

  

![](https://mmbiz.qpic.cn/mmbiz_png/A1HKVXsfHNn3icELYmTSxVGj0DeuafSq8KCcLOmAestSyHFTFsduow7WOgSMlTl7ZJHUSia8mUibibN0oWKNH0ovNw/640?wx_fmt=png)

  

例子：select distinct s.id from T t join S s on t.id=s.id where t. group by t.mobile having count(*)>2 order by s.create_time limit 5;

  

**from**

  

第一步就是选择出 from 关键词后面跟的表，这也是 SQL 执行的第一步：表示要从数据库中执行哪张表。

  

实例说明：在这个例子中就是首先从数据库中找到表 T。

  

**join on**

  

join 是表示要关联的表，on 是连接的条件。通过 from 和 join on 选择出需要执行的数据库表 T 和 S，产生笛卡尔积，生成 T 和 S 合并的临时中间表 Temp1。on：确定表的绑定关系，通过 on 产生临时中间表 Temp2。

  

实例说明：找到表 S，生成临时中间表 Temp1，然后找到表 T 的 id 和 S 的 id 相同的部分组成成表 Temp2，Temp2 里面包含着 T 和 Sid 相等的所有数据。

  

**where**

  

where 表示筛选，根据 where 后面的条件进行过滤，按照指定的字段的值（如果有 and 连接符会进行联合筛选）从临时中间表 Temp2 中筛选需要的数据，注意如果在此阶段找不到数据，会直接返回客户端，不会往下进行。这个过程会生成一个临时中间表 Temp3。注意：在 where 中不可以使用聚合函数，聚合函数主要是（min\max\count\sum 等函数）。

  

实例说明：在 temp2 临时表集合中找到 T 表的 的数据，找到数据后会成临时中间表 Temp3，T。emp3 里包含 name 列为 "Yrion" 的所有表数据。

  

**group by**

  

group by 是进行分组，对 where 条件过滤后的临时表 Temp3 按照固定的字段进行分组，产生临时中间表 Temp4，这个过程只是数据的顺序发生改变，而数据总量不会变化，表中的数据以组的形式存在。

  

实例说明：在 Temp3 表数据中对 mobile 进行分组，查找出 mobile 一样的数据，然后放到一起，产生 Temp4 临时表。

  

**Having**

  

对临时中间表 Temp4 进行聚合，这里可以为 count 等计数，然后产生中间表 Temp5，在此阶段可以使用 select 中的别名。

  

实例说明：在 Temp4 临时表中找出条数大于 2 的数据，如果小于 2 直接被舍弃掉，然后生成临时中间表 Temp5。

  

**select**

  

对分组聚合完的表挑选出需要查询的数据，如果为 * 会解析为所有数据，此时会产生中间表 Temp6。

  

实例说明：在此阶段就是对 Temp5 临时聚合表中 S 表中的 id 进行筛选产生 Temp6，此时 Temp6 就只包含有 s 表的 id 列数据，并且 ，通过 mobile 分组数量大于 2 的数据。

  

**Distinct**

  

Distinct 对所有的数据进行去重，此时如果有 min、max 函数会执行字段函数计算，然后产生临时表 Temp7。

  

实例说明：此阶段对 Temp5 中的数据进行去重，引擎 API 会调用去重函数进行数据的过滤，最终只保留 id 第一次出现的那条数据，然后产生临时中间表 Temp7。

  

**order by**

  

会根据 Temp7 进行顺序排列或者逆序排列，然后插入临时中间表 Temp8，这个过程比较耗费资源。

  

实例说明：这段会将所有 Temp7 临时表中的数据按照创建时间（create_time）进行排序，这个过程也不会有列或者行损失。

  

**limit**

  

limit 对中间表 Temp8 进行分页，产生临时中间表 Temp9，返回给客户端。

实例说明：在 Temp7 中排好序的数据，然后取前五条插入到 Temp9 这个临时表中，最终返回给客户端。

  

PS：实际上这个过程也并不是绝对这样的，中间 MySQL 会有部分的优化以达到最佳的优化效果，比如在 select 筛选出找到的数据集。

总结

本篇博客总结了 MySQL 的执行过程，以及 SQL 的执行顺序，理解这些有助于我们对 SQL 语句进行优化，以及明白 MySQL 中的 SQL 语句从写出来到最终执行的轨迹，有助于我们对 SQL 有比较深入和细致的理解，提高我们的数据库理解能力。同时，对于复杂 SQL 的执行过程、编写都会有一定程度的意义。

  

原文链接：https://www.cnblogs.com/wyq178/p/11576065.html