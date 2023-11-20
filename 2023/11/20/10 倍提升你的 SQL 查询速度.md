> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIwOTU0ODQxNA==&mid=2247484359&idx=1&sn=2bc30c89417a50f92e7d1b37f56e6b53&chksm=977366d6a004efc0a69a920be8fa5271184d99ad7253e92a828077b72bca86a88f05284146a3&scene=132&exptype=timeline_recommend_article_extendread_extendread_for_notrec#wechat_redirect)

* * *

作为一名数据分析师，SQL 是必备技能之一。其优势也比较明显：易于理解，维护和扩展。然而，最大的挑战在于，随着数据量的增加，我们就会遇到延迟的瓶颈，或者说查询太昂贵（耗时）而无法运行。

在这篇文章中我将会给出一些克服瓶颈的经验，这些 **tips** 也许会让延迟减小 10 倍甚至 100 倍。So，让我们一起深入了解吧。

* * *

1. 理解 SQL 的查询顺序
---------------

SQL 就像一个迷你版的编程语言，它按顺序处理数据。

![](https://mmbiz.qpic.cn/mmbiz_png/LDBRqwNJEPg4zibNXMhZibWtUeRJRA8GmU3seNzrWxgicRf8fmNFdrdkLYXLLViaw2tjSh9wCv3Pr1bkoibPIkD10ibQ/640?wx_fmt=png)

使用诸如 “`where`” 和 “`having`” 的过滤子句来减小数据表的大小非常重要，并且这两个子句的执行速度也相对较快。将较小的表传递到以下步骤是一个好主意。SQL 中的 “`group by`”、聚合和窗口函数等子句可能会更耗时。所以，我们应该尽可能在较小的过滤表上运行这些耗时的计算，也不要在大表上执行计算相关的操作。

2. 用星型模式加快查询速度
--------------

在数据库设计中，数据工程师喜欢对数据库进行规范化，减少数据表之间的冗余，从而优化存储、理清数据关系。然而，凡事皆有利弊，与之对应的缺点是查询时需要多个连接和子查询来对数据进行非规范化以提取所需的信息。

![](https://mmbiz.qpic.cn/mmbiz_png/LDBRqwNJEPg4zibNXMhZibWtUeRJRA8GmUQS57k7bfHnIR5k7iaMTf9dDSZoGQGA3PicTXPM9FQaf0ffKKib3MtfJRw/640?wx_fmt=png)  

> 星型模式使用事实表（通常具有较大尺寸）和维度表（较小尺寸）来优化查询性能

为了加快查询速度，建议首先对维度表进行非规范化或联接，因为维度表通常较小并且联接速度更快。之后，如果可能的话，与大型事实表连接。在上述情况下，请尝试在查询的最后一步处理大型销售表。**根据前人的实践经验，遵循这一理念通常可以将查询速度提高 10 倍左右**。

3. 通过了解关键索引将查询速度提高 100 倍
------------------------

在下面的示例中，用户可以按时间或按列遍历 / 查询数据。从视觉上看，按时间（逐行）或按列遍历数据，时间复杂度可能不会有太大差异。

![](https://mmbiz.qpic.cn/mmbiz_png/LDBRqwNJEPg4zibNXMhZibWtUeRJRA8GmUXlO8sIfsk1MqQuIatYgicHGkmYIID8dO4ib3FXWeXlnMAlJHhPBnQyyQ/640?wx_fmt=png)  

然而，实际上，数据并不是以连续的方式存储的。它更像是一个链表数据结构。通过时间查询与通过列查询之间存在巨大差异。

如下图所示，通过在查询中使用时间索引，您可以轻松地将遍历时间或查询时间缩短 10 倍。随着列数量的增加，效率增益甚至更大。国外某小哥亲述在其项目工作中，在处理大型表（数 GB 数据）时，他们将查询时间从 41 天缩短到大约 40 分钟，速度提高了约 100 倍。

![](https://mmbiz.qpic.cn/mmbiz_png/LDBRqwNJEPg4zibNXMhZibWtUeRJRA8GmU5Xibibv95DfrL03IhlhQ2zpZus8kDJlgICVmMgVAMbhhtRw7BcKuian1w/640?wx_fmt=png)  

在这种情况下，基于时间块运行的查询可能比按列运行的查询快 10 到 100 倍，因为数据库是按时间索引的。

此外，您可以要求数据分析师或数据工程师根据您的业务需求重新索引您的数据库。

```
-- two queries to pull large data datable
-- 1) much faster query by using time index
select * 
from your table
where time>start1 and time<end1

select * 
from your table
where time>start2 and time<end2

... ... --rest of timestamps

-- 2) much slow query by pulling column by column
select column1, column2 
from your table

select column3, column4 
from your table

... ... -- rest columns


```

4. 利用 Python 的能力
----------------

在现实项目中，完成上述步骤后，由于 SQL 的带宽或数据库服务器的计算能力瓶颈，你的 SQL 查询仍然不够快。

这个时候就可以使用 Python/Pandas 将中间表缓存到本地驱动器或云驱动器，之后用户就可以使用 Python 执行繁重的表连接或聚合步骤，这样通常会比在数据库中执行类似的步骤快得多。

下面是一个代码示例，通过 Jupyter Notebook 执行 PostgreSQL 查询并将查询结果导出为 dataframe：

```
import pandas as pd
from sqlalchemy import create_engine

# Set up the database connection
engine = create_engine('postgresql://yourusername:yourpassword@yourhostname:yourport/yourdbname')

# Execute a SQL query and store the results in a Pandas DataFrame
df = pd.read_sql_query('SELECT target_columns FROM yourtablename', con=engine)

# Print the DataFrame
print(df)


```

5. 总结
-----

在这篇文章中，我们总结了四种加快你 SQL 查询速度的方式：

*   理解 SQL 的执行顺序，在运行昂贵的计算之前先减小表的大小。
    
*   理解星型模式首先连接小维度表，最后连接大事实表。
    
*   理解索引并根据关键索引来查询大表以提升查询速度。
    
*   最后是利用 Pandas 来提升查询速度。
    

* * *

希望这篇文章对您有用，如果您有更好的技巧或建议，请与我们一同分享。

Thank you for your reading， happy querying！

* * *

* * *

* * *