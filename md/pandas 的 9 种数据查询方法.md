> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NzU0MzU0Nw==&mid=2651398637&idx=4&sn=ecfd627e9da23fdcd7ac037ceee16837&chksm=bd25eaf98a5263ef96c0452154130a29726093a40110696aa0281ec6a0aeaff9f06fb13eb6d3&mpshare=1&scene=1&srcid=0724oh9lR3HcZH8MeV7F8WbI&sharer_sharetime=1627116120748&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> Pandas 之于日常数据分析工作的重要地位不言而喻，而灵活的数据访问则是其中的一个重要环节。本文旨在讲清 P

Pandas 之于日常数据分析工作的重要地位不言而喻，而灵活的数据访问则是其中的一个重要环节。**本文旨在讲清 Pandas 中的 9 种数据访问方式，包括范围读取和条件查询等。**

![](https://mmbiz.qpic.cn/mmbiz_png/p3q2hn6de2O4OypGPXpQNPG0cxJyVwWq4hBkeRJqMzck1eqczRCrmR2PNjqcH32jkYcusvP7wespb7Vibmo76Sg/640?wx_fmt=png)

  

Pandas 中的核心数据结构是 DataFrame，所以在讲解数据访问前有必要充分认清和深刻理解 DataFrame 这种数据结构。以下面经典的 titanic 数据集为例，可以从两个方面特性来认识 DataFrame：  

![](https://mmbiz.qpic.cn/mmbiz_png/p3q2hn6de2O4OypGPXpQNPG0cxJyVwWqsc3Hh050FwDNL50hL9Zu6awxaID5yLRo0iap4ysWrVHkW7ANdlIIRhA/640?wx_fmt=png)

**1、DataFrame 是一个行列均由多个 Series 组成的二维数据表框，其中 Series 可看做是一个一维向量。**理解这一点很重要，因为如果把 DataFrame 看做是一个集合类型的话，那么这个集合的元素泛型即为 Series；

  

  

**2、DataFrame 可看做是一个二维嵌套的 dict，其中第一层 dict 的 key 是各个列名；**而每个 dict 内部则是一个以各行索引为 key 的子 dict。当然，这里只是将其 "看做" 而非等价，是因为其与一个严格的 dict 还是有很大区别的，一个很重要的形式上区别在于：DataFrame 的列名是可以重复的，而 dict 的 key 则是不可重复的。

  

认识了这两点，那么就很容易理解 DataFrame 中数据访问的若干方法，比如：

  

**1. [ ]，这是一种最常用的数据访问方式，某种意义上沿袭了 Python 中的语法糖特色。**通常情况下，[] 常用于在 DataFrame 中获取单列、多列或多行信息。具体而言：

*   当在 [] 中提供单值或多值（多个列名组成的列表）访问时按列进行查询，单值访问不存在列名歧义时还可直接用属性符号 "." 访问
    
*   切片形式访问时按行进行查询，又区分数字切片和标签切片两种情况：当输入数字索引切片时，类似于普通列表切片；当输入标签切片时，执行**范围查询**（即无需切片首末值存在于标签列中），包含两端标签结果，无匹配行时返回为空，但要求标签切片类型与索引类型一致。例如，当标签列类型（可通过 df.index.dtype 查看）为时间类型时，若使用无法隐式转换为时间的字符串作为索引切片，则引发报错
    

![](https://mmbiz.qpic.cn/mmbiz_png/p3q2hn6de2NJiav932Bh1ibpk9iaKvdDxhdy0wVnTjfC811vNSgUUDuvtLBpU5paN0Ulqh7KPxhmyaeNCrXbnPcsQ/640?wx_fmt=png)

切片形式返回行查询，且为范围查询  

  

![](https://mmbiz.qpic.cn/mmbiz_png/p3q2hn6de2NJiav932Bh1ibpk9iaKvdDxhdaK8gJd2C0rMqjHVqYV227KJb1RbSjVjtXd5hnANibaXhoj9w8Pt1Z7w/640?wx_fmt=png)

切片类型与索引列类型不一致时，引发报错  

  

**2. loc/iloc**，可能是除 [] 之外最为常用的两种数据访问方法，**其中 loc 按标签值（列名和行索引取值）访问；iloc 按数字索引访问，均支持单值访问或切片查询。**与 [ ] 访问类似，loc 按标签访问时也是执行范围查询，包含两端结果。

  

**3. at/iat，**其实是可看分别做为 loc 和 iloc 的一种特殊形式，只不过不支持切片访问，仅可用于单值提取，即指定单个标签值或单个索引值进行访问，一般返回标量结果，除非标签值存在重复。

**4. isin****，条件范围查询，**一般是对某一列判断其取值是否在某个可迭代的集合中。即根据特定列值是否存在于指定列表返回相应的结果。

**5. where，**妥妥的 Pandas 仿照 SQL 中实现的算子命名。不过这个命名其实是非常直观且好用的，如果熟悉 Spark 则会自然联想到在 Spark 中其实数据过滤主要就是用给的 where 算子。**这里仍然是执行条件查询，但与直观不大相符的是这里会返回全部结果，只是将不满足匹配条件的结果赋值为 NaN 或其他指定值，可用于筛选或屏蔽值**

![](https://mmbiz.qpic.cn/mmbiz_png/p3q2hn6de2NJiav932Bh1ibpk9iaKvdDxhdvILm3mpYg3JQemSfXSS1hR5gAOqOIGq3lnosiclaLWicibt7O4obk0Pkg/640?wx_fmt=png)

  

**6. query，**提到 query，还得多说两句。前面受 where 容易使人联想到 SQL，其实提到 query 让人想到的仍然是 SQL，因为 SQL=Structed Query Language，所以 query 用在 DataFrame 中其实是提供了一种以类 SQL 语法执行数据访问的方式，这对熟悉 SQL 的使用者来说非常有帮助！当然，这种用法一般都可用常规的条件查询替代。

![](https://mmbiz.qpic.cn/mmbiz_png/p3q2hn6de2NJiav932Bh1ibpk9iaKvdDxhdALj2KFSQBfz30MewMOKhahwp2NicFhQibS2rqHcVPek5w8qzlRzVGMmg/640?wx_fmt=png)

  

**7. filter，**说完 where 和 query，其实还有一个表面上很类似的查询功能，那就是 filter。在 Spark 中，filter 是 where 的别名算子，即二者实现相同功能；但在 pandas 的 DataFrame 中却远非如此。**在 DataFrame 中，filter 是用来读取特定的行或列，并支持三种形式的筛选：固定列名 (items)、正则表达式(regex) 以及模糊查询(like)，并通过 axis 参数来控制是行方向或列方向的查询。**这里给出其文档简介，很容易理解其功能：  

![](https://mmbiz.qpic.cn/mmbiz_png/p3q2hn6de2O4OypGPXpQNPG0cxJyVwWqRRwtx7wg84ibzR0L6jwgWe9Mgvu3WcnyhHDY5JHVjgtGuBmrP4cVpgQ/640?wx_fmt=png)

  

**8. get。**由于 DataFrame 可看做是嵌套 dict 结构，所以也提供了类似字典中的 get() 方法，主要适用于不确定数据结构中是否包含该标签时，与字典的 get 方法非常类似:

![](https://mmbiz.qpic.cn/mmbiz_png/p3q2hn6de2NJiav932Bh1ibpk9iaKvdDxhdPwRHv7taXeUQhSCR4NvlJAOJibwEyEQwnXwrYYtnfEsFUkhOhQZSWkw/640?wx_fmt=png)

  

**9. lookup。**如果说提到 query 自然联想到 SQL，那么提到 lookup 自然会想到的就是 Excel 了。实际上，DataFrame 中的 lookup 执行的功能与 Excel 中的 lookup 函数差距还是挺大的，初学之时颇有一种挂羊头卖狗肉的感觉。**实际上，这里的 lookup 可看做是 loc 的一种特殊形式，即分别传入一组行标签和列标签，lookup 解析成一组行列坐标，返回相应结果：**  

![](https://mmbiz.qpic.cn/mmbiz_png/p3q2hn6de2NJiav932Bh1ibpk9iaKvdDxhdyPC2ZOnTP1t51uYlNpgNcib7Bydrd4ma0E9Aqgg0ic70FOaUjq2uqYHQ/640?wx_fmt=png)

  

最后，pandas 中提供了非常灵活多样的数据访问形式，可以说是兼顾了嵌套 Series 和嵌套 dict 的双重特性，但最为常用的其实还是 []、loc 和 iloc 这几种方法，而对于 where、query、isin 等在某些情况下也会非常高效，但对于 filter、get、lookup 以及 at/iat 等其实则并不常用。

* * *