> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/339945069)

今天的主角，Clickhouse.

简单介绍下，Clickhouse 是俄罗斯的一款 OLAP 分析引擎。它有两个非常突出的优点，**列式存储和压缩**。

之前写过 S[QL Server 的列式存储和内存压缩](https://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzI2NjMxNzgyMw%3D%3D%26mid%3D2247484155%26idx%3D1%26sn%3Dfd4f23c274f6bb26966922916f66393b%26chksm%3Dea8eb86bddf9317d14f54bd9cc547818b7c6958e18ff3882b7faf6719382e017910c3ecaf4ac%26scene%3D21%23wechat_redirect)，同样也性能惊人。但 clickhouse 更彪悍。花约摸 4 小时，我对比了下 sql server, mysql 和 clickhouse 的查询效率。

结果惊人！Clickhouse 完胜，快于 SQL Server 10 倍，MySQL 近 1000 多倍。

一个超级 IDE，DBeaver；一个极简 SQL 。就是我本次实验所用：

```
select  TOP 100 [DATE],[C-IP],COUNT(ID) AS CNT 
from AuditTrail 
GROUP BY [DATE],[C-IP] 
ORDER BY COUNT(ID)DESC ,[DATE] DESC
```

以上是 SQL Server 的运行语句，耗时 2.479s

![](https://pic3.zhimg.com/v2-e3388e2533eb819ac652b286a8b0706e_b.jpg)

此处当然有截图为证。

以下是 clickhouse 和 MySQL 的语法：

```
select  `DATE`,`C-IP`,COUNT(ID) AS CNT 
from AuditTrail 
GROUP BY `DATE`,`C-IP` 
ORDER BY COUNT(ID) DESC ,`DATE` DESC 
limit 100 ;
```

Clickhouse 用了 275ms.

![](https://pic4.zhimg.com/v2-d29aa96f463db12311cba9689fbd8aaf_b.jpg)

MySQL 用了 336s 还没出结果，放弃执行！

![](https://pic2.zhimg.com/v2-e42cb039fd07dc674a62d88010c15785_b.jpg)

事实上，在导入 Clickhouse 数据的时候，就发现了，同样的 1200 多万数据，导入 MySQL 使用了 90 分钟，而 clickhouse 导入只需 2 分钟。

这份数据来自 sql server adventure works2014 的 AuditTrail 数据表。有兴趣的朋友，可以自己做个实验。

推荐这款神编辑器 DBeaver. 玩数据库的朋友都烦的事，在各种眼花缭乱的 IDE 之间切来切去，一会儿 oracle, MySQL, 一会儿切 SQL Server， 还要到 Hive, HBase 里面取点数。如果有一款编辑器，可以把天下所有数据库都装进来，多好。

DBeaver 就可以！

![](https://pic4.zhimg.com/v2-5878c59d817cefbb27147eb5b4e66d63_b.jpg)

在这次实验中，我从上到下，分别连接了 MySQL, SQL Server 和 Clickhouse，轻松无压力。

**-- 完 --**