> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651967871&idx=1&sn=5dc5a0ea2f8a0a4ca2c746be0a434b77&chksm=bd2d64a38a5aedb50f647e08f148e87dfb2fdda68ff08957e4b9d7865c86de534adf88666c56&mpshare=1&scene=1&srcid=0604cZqtpTtZUbMRugTcXHXY&sharer_sharetime=1622773143276&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

用户中心是几乎每一个公司必备的基础服务，用户注册、登录、信息查询与修改都离不开用户中心。

当数据量越来越大时，需要多用户中心进行水平切分。最常见的水平切分方式，按照 uid 取模分库：

![](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOyopbPRUaMFgG0H93ibyFhR1XU7XZ5Llv3XIWjerprU7WczECzHSMniaGZG1ibcXL5rzcuKk0Mc3yN2Q/640?wx_fmt=png)

通过 uid 取模，将数据分布到多个数据库实例上去，提高服务实例个数，降低单库数据量，以达到扩容的目的。

水平切分之后：

![](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOyopbPRUaMFgG0H93ibyFhR12NPH6UWxcTGIhDs1traUtibSFRt6QndHx4cSk8zfXbffiaiaExRcEGCqA/640?wx_fmt=png)

uid 属性上的查询可以直接路由到库，如上图，假设访问 uid=124 的数据，取模后能够直接定位 db-user1。

对于 uname 上的查询，就不能这么幸运了：

![](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOyopbPRUaMFgG0H93ibyFhR1DkB3JDYrUoSLJ6b904UPk2towrGLyicKI1ufrH95cL6r3U7UZq0caGQ/640?wx_fmt=png)

uname 上的查询，如上图，假设访问 uname=shenjian 的数据，由于不知道数据落在哪个库上，往往需要遍历所有库（扫全库法），当分库数量多起来，性能会显著降低。

用 uid 分库，如何高效实现上的查询，是本文将要讨论的问题。

**方案一：索引表法**

**思路**：uid 能直接定位到库，uname 不能直接定位到库，如果通过 uname 能查询到 uid，问题解决。

**解决方案**：

（1）建立一个索引表记录 uname 到 uid 的映射关系；

（2）用 uname 来访问时，先通过索引表查询到 uid，再定位相应的库；

（3）索引表属性较少，可以容纳非常多数据，一般不需要分库；

（4）如果数据量过大，可以通过 uname 来分库；

**潜在不足**：多一次数据库查询，性能下降一倍。

**方案二：缓存映射法**

**思路**：访问索引表性能较低，把映射关系放在缓存里性能更佳。

**解决方案**：

（1）uname 查询先到 cache 中查询 uid，再根据 uid 定位数据库；

（2）假设 cache miss，采用扫全库法获取 uname 对应的 uid，放入 cache；

（3）uname 到 uid 的映射关系不会变化，映射关系一旦放入缓存，不会更改，无需淘汰，缓存命中率超高；

（4）如果数据量过大，可以通过 name 进行 cache 水平切分；

**潜在不足**：多一次 cache 查询。

**方案三：uname 生成 uid**

**思路**：不进行远程查询，由 uname 直接得到 uid。  

解决方案：

（1）在用户注册时，设计函数 uname 生成 uid，uid=f(uname)，按 uid 分库插入数据；

（2）用 uname 来访问时，先通过函数计算出 uid，即 uid=f(uname) 再来一遍，由 uid 路由到对应库；

**潜在不足**：该函数设计需要非常讲究技巧，有 uid 生成冲突风险。

**方案四：基因法，uname 基因融入 uid**

**思路**：不能用 uname 生成 uid，可以从 uname 抽取 “基因”，融入 uid 中。

![](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOyopbPRUaMFgG0H93ibyFhR1e0Nuu1db2rIq0Nyt8GRuXrTJ0CP7F9icicHHfTrf00ysRkWcAtHBVz6A/640?wx_fmt=png)

假设分 8 库，采用 uid%8 路由，潜台词是，uid 的最后 3 个 bit 决定这条数据落在哪个库上，这 3 个 bit 就是所谓的 “基因”。

**解决方案**：

（1）在用户注册时，设计函数 uname 生成 3bit 基因，uname_gene=f(uname)，如上图粉色部分；

（2）同时，生成 61bit 的全局唯一 id，作为用户的标识，如上图绿色部分；

（3）接着把 3bit 的 uname_gene 也作为 uid 的一部分，如上图屎黄色部分；

（4）生成 64bit 的 uid，由 id 和 uname_gene 拼装而成，并按照 uid 分库插入数据；

（5）用 uname 来访问时，先通过函数由 uname 再次复原 3bit 基因，uname_gene=f(uname)，通过 uname_gene%8 直接定位到库；

**总结**

**业务场景**：用户中心，数据量大，通过 uid 分库后，通过 uname 路由不到库。

**解决方案**：

（1）扫全库法：遍历所有库；

（2）索引表法：数据库中记录 uname 到 uid 的映射关系；

（3）缓存映射法：缓存中记录 uname 到 uid 的映射关系；

（4）uname 生成 uid；

（5）基因法：uname 基因融入 uid；

公众号

**相关推荐**：  

《[InnoDB 并发如此高，原因竟然在这？](http://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651967047&idx=1&sn=b605fe50e6dd74ecad659c464a0e29ee&chksm=bd2d7b9b8a5af28d35c13e469e8e2c6a7082f00e608fd52d999fc0f7a6aec016faf2a8d748fa&scene=21#wechat_redirect)》

《[InnoDB，七种锁](http://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651967369&idx=1&sn=d639abf6772a72c25cc2537749258163&chksm=bd2d7a558a5af3436e0a435a3e7cd9f72f67c1269370be936a82dc0af483f47aefa48bd58d69&scene=21#wechat_redirect)》

《[InnoDB 索引，终于懂了](http://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651967183&idx=1&sn=d033755bf26244f4866b38750acfbb2b&chksm=bd2d7b138a5af20548229a22db8da0838f85784d6119833e8ff6abfe1068548ea70ca7587da5&scene=21#wechat_redirect)》

《[InnoDB，四种事务的隔离级别实现](http://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651967707&idx=1&sn=733da08d6c24a19a05bf3ba2ada96364&chksm=bd2d65078a5aec11ee28557355c736359ef7a1dc42629a22cc3274c44b4affd6a47859d903a6&scene=21#wechat_redirect)》

《[InnoDB，调试死锁的方法！](http://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651967734&idx=1&sn=817c850062a1deccf84298677f6095c7&chksm=bd2d652a8a5aec3c3edab171519c29e7df9ab023247bde2f324571874e5f0ee3b96e71a6c69a&scene=21#wechat_redirect)》

**调研**：

贵司用的是哪种方案？