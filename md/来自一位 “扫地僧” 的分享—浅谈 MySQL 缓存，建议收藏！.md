> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/-nsJJeeYEHvo2TZlB0GB0Q)

作者 | 杨洋

MySQL 一次查询步骤  

---------------

在聊缓存之前，我们先聊一聊一次 SQL 查询大概经过的一些步骤。一次查询由客户端发起 SQL 查询，再提交到 MySQL 服务，经过 MySQL 层面的处理，再流转到数据库的存储引擎中。如下图所示：

MySQL 查询缓存
----------

### 查询缓存特点

*   当一条 SQL 包含不确定结果查询，则会直接跳过查询缓存。比如：NOW() 时间函数，这些 SQL 执行不是幂等的就不会启用查询缓存。
    
*   当一条 SQL 有任意的变化包含标点符号空格等变化，之前的缓存都会失效，因为查询缓存在 SQL 优化之前，即使你多了个空格对查询结果不影响，但是对于 MySQL 来说就是另外一条查询。
    
*   开启了查询缓存不一定会提示 MySQL 的性能：每次查询都会维护查询缓存内容，每次写操作也需要失效对应表的查询缓存。
    
*   在某张表的某次事务过程中的任何查询，MySQL 是不会对其进行缓存的。
    
*   查询缓存非常适用于涉及到的数据量巨大，且结果非常小的场景，比如说一些统计类的查询，聚合函数查询等。
    
*   写大于读的场景下不建议开启查询缓存，因为会给写带来额外的开销。
    

InnoDB Buffer Pool
------------------

### 名词解释

#### Buffer Pool Instance

#### Buffer chunks

#### 链表

*   Free List：存放未曾使用的空闲 page，InnoDB 初始化后，Buffer Chunks 中的所有数据页都被加入到 Free List，表示所有节点都可用。
    
*   LRU List：存放所有使用过的 page，所有新读取进来的数据页都被放在上面。
    
*   Flush List：存放所有被修改过的 page，也就是所有的未被刷新到磁盘的脏页。
    

#### 互斥锁

为了保证各个链表访问的互斥，比如说需要更新 LRU 链表数据，则需要对 LRU 加上 Mutex。以免其他线程修改 LRU 数据，造成数据不一致。

### Buffer Pool Instance

![](https://mmbiz.qpic.cn/mmbiz_png/ibm2sb53lRhz0fPjM22b6fdPcK1CP41icbpffWx6rFkw2gjyibD5mfgkqkUrFjRVfHw2eHONnOCT4zaYLzWOHA9gA/640?wx_fmt=png)

上图展示的是一个 instance 所包含的内容，下面介绍一些核心数据结构。

![](https://mmbiz.qpic.cn/mmbiz_png/ibm2sb53lRhz0fPjM22b6fdPcK1CP41icb3ZPK4eLic5E5LgtgRo6OxZHBHxnoVTjSYNmvlQkNoKJ2clFuZBrV6Ng/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ibm2sb53lRhz0fPjM22b6fdPcK1CP41icbVl7afRLywE5FEASTOneIHNYtrze0eaMpayWxJpGkdlkaTegrMdPkcQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ibm2sb53lRhz0fPjM22b6fdPcK1CP41icbEN6uOpcNOrKQ7Cc5IGeaFPcADXYic6bGa0rx1GLBeWZxpld4VkHLQXw/640?wx_fmt=png)

`buf_page_state`的几种重要状态：

*   `BUF_BLOCK_NOT_USED`：未被使用的 Page，存在于 Free List 中。是一种长时间存在的状态。
    
*   `BUF_BLOCK_READY_FOR_USE`：准备被使用的 Page，不存在任何链表中，从 Free List 中刚刚拿出来的状态，是一种很短暂的状态。
    
*   `BUF_BLOCK_FILE_PAGE`：正常的被查询后缓存的 Page，存储在 LRU List 中，是一种长时间存在的状态。
    
*   `BUF_BLOCK_MEMORY`：存储着系统的数据，比如说 InnoDB 行锁，自适应哈希索引等，该数据不存在任何链表中。是一种长时间存在的状态。
    
*   `BUF_BLOCK_REMOVE_HASH`：当加入 Free List 之前，需要先把 page hash 移除。因此这种状态就表示此页面 page hash 已经被移除，但是还没被加入到 Free List 中，是一个比较短暂的状态。
    

### LRU List

LRU 顾名思义就是最少使用原则，就是把最少使用的慢慢放到链表尾端。新鲜的数据放到链表表头。如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/ibm2sb53lRhz0fPjM22b6fdPcK1CP41icb4XbTj1xkWbvBZDZibCOf9uIDkNiaWSmgz5NylEvj1CR5XlibG1hicAEa4A/640?wx_fmt=png)

那么这种情况下，如果我进行一次查询需要全表扫描的时候（比如说：不走索引然后 like），InnoDB 有预读特性会把可能跟当前查询有关的数据都缓存起来。就导致了最终 LRU List 里缓存的数据并不是我想要的热点数据。这种情况怎么办呢，我们给 LRU 简单改造一下如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/ibm2sb53lRhz0fPjM22b6fdPcK1CP41icbn4gDy54WFegx0qB184WMNYL9IXMxUrEVZBXNicpTZLcTv3BL7Q3uWMA/640?wx_fmt=png)

可以看到我们把链表分成了两部分，第一部分我们称之为 young 区，第二部分我们称之为 old 区。当有新的 page 要加入链表的时候，优先加入到 old 区。当然需要排掉数据的时候也是把 old 尾端排掉。这时候可以调整 young 区，或者直接插入新数据到 old 区。当 old 区数据页内数据被真正访问的时候（非预读）就把这条数据插入到 young 区的头结点，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/ibm2sb53lRhz0fPjM22b6fdPcK1CP41icbpEPWxuBNvTOZCIFzRNMW7JicmYAiaeaISnibOrpT931aGw38exDSV28Lw/640?wx_fmt=png)

那么这种情况是不是就完美了呢，然而并不是，当我们做一次大范围查询的时候，最终结果集非常大，这样就造成了 LRU List 中的热点数据被刷走。所以为了避免这种情况 InnoDB 做了如下改造：

![](https://mmbiz.qpic.cn/mmbiz_png/ibm2sb53lRhz0fPjM22b6fdPcK1CP41icbmKLkmjDc4e6HmJ4ib84nTG4uJdkjNSuuYS1QeW6vFzOaiavicXMa7S9oQ/640?wx_fmt=png)

这样给定一个参数，如果在规定时间内 old 区数据被再一次访问，则会把该数据插入到 young 区。当然 young 区的数据如果被再次访问，则有可能会调整到链表的头部。为什么说有可能呢，因为 InnoDB 规定只有 young 区的后 3/4 区域的数据被再一次访问才会加到头部。

### 获取 Free Page 流程

![](https://mmbiz.qpic.cn/mmbiz_png/ibm2sb53lRhz0fPjM22b6fdPcK1CP41icbQQeT32VJZNxmDsvwwOfKwNLa41icjIV0TkWT5loSIglCibpdSEgVl13w/640?wx_fmt=png)

以上是简单化版的 Get Free Page 流程，当然实际上的步骤要复杂的多，而且情况也繁杂的多。

全文完

以下文章您可能也会感兴趣：

*   [数据介绍一个 MySQL 自动化运维利器 - Inception](http://mp.weixin.qq.com/s?__biz=MzUxOTE5MTY4MQ==&mid=2247484036&idx=1&sn=375e7aa38c55e064ac5964d6d32a41cf&chksm=f9fc2f6ace8ba67c69f91ab42d9d06abb8ff8efda94d6876a815cd6eea9153525b6f3789a8d2&scene=21#wechat_redirect)  
    
*   [小程序中 Redux 的使用](http://mp.weixin.qq.com/s?__biz=MzUxOTE5MTY4MQ==&mid=2247484028&idx=1&sn=0a662c070100648371fc147eeadebf85&chksm=f9fc2f92ce8ba684db5026e2db94bd1e726d63d8cba0d238f4a0cf9f367be1d10bad8eaff7ca&scene=21#wechat_redirect)
    
*   [数据可视化过程不完全指南](http://mp.weixin.qq.com/s?__biz=MzUxOTE5MTY4MQ==&mid=2247484022&idx=1&sn=112aaff32972bd01e67495b54d8586d9&chksm=f9fc2f98ce8ba68ed9de2215c8ed67e28f087295fb8d76a16d4e9fd8e9d56693af1bc4aec676&scene=21#wechat_redirect)
    
*   [小型大写字母的用武之处](http://mp.weixin.qq.com/s?__biz=MzUxOTE5MTY4MQ==&mid=2247484018&idx=1&sn=573f6c6a2bf485ebd5a9b99ac5d4acd7&chksm=f9fc2f9cce8ba68a4fb497c42bf64af65246e00803e6762042352e2e1e5013608bf9c61a1c4a&scene=21#wechat_redirect)
    
*   [React Native 项目整合 CodePush 完全指南](http://mp.weixin.qq.com/s?__biz=MzUxOTE5MTY4MQ==&mid=2247484014&idx=1&sn=8e491a67f4aa1b5880b2fc76d161e237&chksm=f9fc2f80ce8ba696c7814afef6d6f261d4a0ee99d29f55f8c8ad6d63aecfba64347c009ea456&scene=21#wechat_redirect)  
    
*   [震惊！JavaScript 竟然可以类型推断！](http://mp.weixin.qq.com/s?__biz=MzUxOTE5MTY4MQ==&mid=2247484011&idx=1&sn=d68ea0d4d4312507e8af3abe1669e2bb&chksm=f9fc2f85ce8ba693fe6f7ec0e716bbedaecdb9e1930fea26c18fc7c891b9e54681fdccf87551&scene=21#wechat_redirect)
    
*   [OpenResty 不完全指南](http://mp.weixin.qq.com/s?__biz=MzUxOTE5MTY4MQ==&mid=2247484003&idx=1&sn=d881cdc62ee2aa6a398a0af3ef28da32&chksm=f9fc2f8dce8ba69bb3ac5e39af331b2809fe3cb18f93b290664ef6315dd28d237fb688679ba8&scene=21#wechat_redirect)
    
*   [如何进行系统分析与设计](http://mp.weixin.qq.com/s?__biz=MzUxOTE5MTY4MQ==&mid=2247484000&idx=1&sn=d57cc07291d0201a972041d3864b90a0&chksm=f9fc2f8ece8ba6980fdc2b8f4b073c9c096ef6404d18a5cecbee69b4975fa137f6cb912054cc&scene=21#wechat_redirect)  
    
*   [ConcurrentHashMap 的 size 方法原理分析](http://mp.weixin.qq.com/s?__biz=MzUxOTE5MTY4MQ==&mid=2247483997&idx=1&sn=a0b5fbf479353504868cbbdf1fc9c5bb&chksm=f9fc2fb3ce8ba6a543b419e243914c63ee6a18a22945f763c3683ac846f14fa765d8efe10db9&scene=21#wechat_redirect)  
    
*   [从 ThreadLocal 的实现看散列算法](http://mp.weixin.qq.com/s?__biz=MzUxOTE5MTY4MQ==&mid=2247483992&idx=1&sn=33087e68ab0dca6fce798c014b2a42c6&chksm=f9fc2fb6ce8ba6a02e88d015796ce399ed107a57f3e4a9e6c9f04ed70a54088161346545115c&scene=21#wechat_redirect)  
    
*   [理清 Promise 的状态及使用](http://mp.weixin.qq.com/s?__biz=MzUxOTE5MTY4MQ==&mid=2247483989&idx=1&sn=fa3c58c8e1d58e8306bb3a819c075172&chksm=f9fc2fbbce8ba6adad7cee4d2f185151a0565c9f8b3fe66c66da3660c2dc0d48c2af89cb0629&scene=21#wechat_redirect)
    
*   [逻辑思维：理清思路，表达自己的技巧](http://mp.weixin.qq.com/s?__biz=MzUxOTE5MTY4MQ==&mid=2247483985&idx=1&sn=09d8233703c998557cf8279485dfce36&chksm=f9fc2fbfce8ba6a99ae2a7d6cc86d5804ded43399b7b5118da2893b6efc446799017f2271f39&scene=21#wechat_redirect)
    
*   [Actor 模型及 Akka 简介](http://mp.weixin.qq.com/s?__biz=MzUxOTE5MTY4MQ==&mid=2247483981&idx=1&sn=9aa4b6b15a72b0d319d9e36fb2114796&chksm=f9fc2fa3ce8ba6b515f8644af28d7c5b9d847d3c4d9ff6995d903255071c14c372dad80fe53e&scene=21#wechat_redirect)