> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [my.oschina.net](https://my.oschina.net/u/4526289/blog/10114608)

> 本文分享自华为云社区 《华为云 GES 持久化图数据库复合索引介绍》，作者：村头树下。

本文分享自华为云社区 [《华为云 GES 持久化图数据库复合索引介绍》](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fbbs.huaweicloud.com%2Fblogs%2F411666%3Futm_source%3Doschina%26utm_medium%3Dbbs-ex%26utm_campaign%3Dother%26utm_content%3Dcontent)，作者：村头树下。

本文章主要介绍索引的作用，以及如何实现这种功能，希望可以帮助理解索引的作用以及如何使用索引

1. 什么是复合索引
----------

复合索引是用户手动建立的用于加速查询的一类额外数据。详细参数可以参考规格文档

[https://support.huaweicloud.com/api-ges/ges_03_0454.html](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fsupport.huaweicloud.com%2Fapi-ges%2Fges_03_0454.html)

2. 复合索引能做什么
-----------

复合索引有两类。一是 label 索引，用于加速 label 的扫描。二是属性索引，用于加速属性过滤。

这里列举了一些常用接口（语句）与索引的关系

<table><tbody><tr><th><p><strong>api 接口</strong></p></th><th><p><strong>索引加速方式</strong></p></th></tr></tbody><tbody><tr><td><p>summary</p></td><td><p>扫描 label 索引，统计各 label 点边数目</p></td></tr><tr><td><p>match (n:user) return count(*)</p></td><td><p>扫描点 label 索引，统计 label 为 user 的点数目</p></td></tr><tr><td><p>match ()-[r:label]-() return count®</p></td><td><p>扫描边 label 索引，统计指定 label 点数目</p></td></tr><tr><td><p>match (n:user) return n limit 1</p></td><td><p>通过点 label 索引快速寻找 label 为 user 的点</p></td></tr><tr><td><p>match (n:user) where n.age &gt; 10 return n limit 1</p></td><td><p>仅有 label 索引时扫描 label 索引，寻找 user 的点，然后进行属性过滤。当存在 age 属性索引时直接使用属性索引定位到目标点</p></td></tr><tr><td><p>match (n:user) where n.age in [1, 10] return n limit 1</p></td><td><p>同上</p></td></tr></tbody></table>

3. 无索引时如何查询
-----------

首先了解无索引的情况下，查询的逻辑，才可以理解索引在此基础上做了什么使得查询能够加速。查询逻辑主要与两个方面有关：数据结构，以及数据访问方式，以及查询场景。

### a) 原始点结构

持久化版本所有数据都是以 KV（键值对）的方式存储在分布式 KV 数据库中，在没有建立索引的时候，数据库中仅有原始点边 KV。以点数据结构为例：

Key: ![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtybbs/519/984/817/2850086000519984817.20230927102532.59034676660642229201211385752181:50001231000000:2800:E900A4FBBB3D3F9F6E97CFC3D06C74E3E1A9CDCEF610F408CBE35CF042A02EB3.png)

Value:![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtybbs/519/984/817/2850086000519984817.20230927102532.09951834891756932510363651849529:50001231000000:2800:635CA164693DB173830EC496DD2FF4EC716BB1565F80C73A33B6BC995BE350CA.png)

key 的开始部分为 kVType，这是所有数据都会存在的固定前缀，用以区分不同类型的数据。然后是 Vid 是全局唯一点 id。Labelid 是标识 label 的内置编码。Value 则是属性的数据。

### b) 数据访问方式

所有的图数据的查询最终都是依托于 KV 数据库的访问。常用的访问 KV 数据的方式有两种：

1.  精确查询接口，指定完整的 key 查询 value
2.  前缀查询接口，仅指定 key 的前缀部分，查询所有 key 的前缀匹配的 KV 数据对。前缀查相对来说会更加频繁的使用。一个场景可能会需要多次前缀查，而前缀查的次数越多，结果越多，相应的此场景响应速度就越慢。前缀查结果大小直接与前缀的长度有关，前缀越长或者越精确，那么前缀查的结果越少。需要的计算量也越少。相应速度就会越快。

### c) 查询场景：

常见查询场景的对应的 kv 层接口调用：

<table><tbody><tr><th><p><strong>场景</strong></p></th><th><p><strong>KV 接口及调用次数</strong></p></th><th><p><strong>查询速度</strong></p></th><th><p><strong>对应 Cypher 语句</strong></p></th></tr></tbody><tbody><tr><td><p>指定 id 过滤</p></td><td><p>前缀查 * 1</p></td><td><p>快，由于 KVType 和 Vid 已知，可以拼出前缀，同时一个 id 一般不会有太多 label，前缀查的结果不会特别多。</p></td><td><p>match（n） where id(n)=‘0’ return n</p></td></tr><tr><td><p>指定 label 过滤</p></td><td><p>前缀查</p><p>n + 过滤</p>m</td><td><p>慢， 由于不知道 Vid，所以只能先拼出只有 KVType 的前缀，然后前缀查出所有点，再逐个过滤 Label, 点数据较多时，会有多次前缀查，分批获取再过滤。</p></td><td><p>match（n:Label） return n</p></td></tr><tr><td><p>指定 label + 属性过滤</p></td><td><p>前缀查</p><p>n + 过滤</p>m</td><td><p>慢， 查询前缀为 KvType, 遍历全图点，先进行 Label 过滤，再进行属性过滤</p></td><td><p>match (n:Label) where n.prop=‘xx’ return n</p></td></tr><tr><td><p>指定属性过滤</p></td><td><p>前缀查</p><p>n + 过滤</p>m</td><td><p>非常慢， 查询前缀为 KvType, 遍历全图点，全部进行属性过滤</p></td><td><p>match (n) where n.prop=‘xx’ return n</p></td></tr></tbody></table>

可见，除了指定 id 的查询，其他所有查询均非常慢。这些查询都需要进行全图点扫描加过滤的方式来获取结果。这与查询出来的结果数目无关。对于较大的图来说，这样的查询代价是十分巨大的。

4. 复合索引如何加速
-----------

查询慢的场景无外乎两种场景，label 查询或者属性查询。在没有索引的情况下，这两种查询都是建立在全局点扫描的基础上，进行过滤。当有效数据占比越低（例如全局点 1w, 目标点仅有 1 个），这种扫描方式就越显得不划算。

对于这两种场景，我们可以建立对应的索引。索引本身也是 KV 数据。所以其 key 的布局就决定了其功能。

1. 对于 label 过滤场景，索引的 key 的格式为：

![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtybbs/519/984/817/2850086000519984817.20230927102532.79665385416397739426054876098597:50001231000000:2800:A2CE99DE5BD4B61F5F56D26220A0FF10DC970A0B97E7B3BBD343B82E13682481.png)

对于每一个点，都会有一条对应的 Label 索引 KV。

当需要过滤特定 Label 时，可以拼出 KVType+Label 的前缀，利用 kv 数据底座的前缀查接口，就能直接将所有符合条件的点过滤出来。

2. 对于属性过滤的场景，索引的 key 格式为：

![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtybbs/519/984/817/2850086000519984817.20230927102532.89575332567975621558954633422059:50001231000000:2800:721EE988DF62FDDA4506951B526B38089F1F94BC427E246FF092B9A3B510B7B1.png)

属性索引只针对个别过滤较为频繁的属性而建立。所以也只会对包含此属性的点才会生成属性索引 kv。相比于 Label 索引这里只是多了一个 property 字段。此字段填的是 Vid 对应点的属性的值。需要注意的是，property 字段并不包含全部的点属性，仅仅是待过滤属性的值。

当进行属性查询时，由于知道目标值（例如 where n.prop=1, 目标值就是 1）。直接拼出 KVTypr+Label+Property，调用前缀查询接口。即可查出所有符合条件的点。

当利用索引查出匹配的索引 KV 之后，就可以很方便的拿到对应的 VId。然后根据此 Vid，就能快速查询到这个点的属性，或者邻居等信息。

5. 索引建立的若干建议
------------

索引并不是没有代价的，虽然它能加速查询，但是会降低写操作的性能，以及耗费更多的磁盘空间。所以建立索引之前需要考虑是不是必要的。这可以从数据区分度，数据大小，以及访问频率三个方面来评估。

*   数据区分度：对于属性索引建议在过滤性好的属性上建立。值分布较为分散，比较适合建立。例如身份证号，手机号。但是对于性别这种属性，就不建议为此建立。对于 label 索引，如果图里面只有一个 label，那么建 label 索引其实也是没有什么必要的，但是大部分情况，label 索引都是必要的。
*   数据大小：这主要是针对属性索引来说的，在已经有 Label 索引的前提下，如果某个 label 下的点边数目很少，即使扫描所有 label 代价也不高，这时候没有必要再为其建立属性索引。
*   访问频率：这一点很好理解，只对频繁在 where 子句中出现的属性建立索引。

[**点击关注，第一时间了解华为云新鲜技术～**](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fbbs.huaweicloud.com%2Fblogs%3Futm_source%3Doschina%26utm_medium%3Dbbs-ex%26utm_campaign%3Dother%26utm_content%3Dcontent)