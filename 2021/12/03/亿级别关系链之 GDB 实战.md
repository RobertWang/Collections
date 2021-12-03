> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7036946890851614728)*   我关注的人是否赞过这条内容。
    
*   我关注的人在一定时间之内关注的其他人。
    

我们分别通过 MySQL 方案和 GDB 方案来处理，然后对比一下其中优劣，就能得出本小节的答案。下面采用图的方式来描述示例数据。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/026e3277d2ee4d04b740187dcaef95f5~tplv-k3u1fbpfcp-watermark.awebp?)

**我关注的人是否赞过这条内容**

**一般 MySQL 方案**：

*   查到所有关注用户（现有 set 缓存）:select followUserId from user_follow where userId = {$userId}
*   查询这些用户是否有点赞过某些动态: select * from content_light where userId in {followUserId} and contentId in {contentId}

*   *   需要注意，in 查询，在数组较大的情况，索引经常会失效，原因在于 MySQL 索引策略在判断时如果发现需要查询太多次的索引，可能还不如直接扫表来得快。这也正是 DBA 同学建议我们 in 查询不要超过 200 个的原因。

**GDB 方案**：

*   根据图例写出节点与边的结构即可：match (u1:User) -[:Attention]->(u2:User)-[:Light]->(c:Content) where u1.userId = {userId} and c.contentId in {contentIds} return u2.userId , c.contentId;

**方案对比：**

在 MySQL 方案中，当「我关注的用户」较多时（在得物，关注的人大于 1000 的大有人在），存在慢查风险。在 GDB 方案中，查询语句简洁明了，并且查询效率也较高，所谓无图无证据，下图为实践数据（无 Redis 缓存）：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/86eb3da30ed5420a821658d6eff9a0ee~tplv-k3u1fbpfcp-watermark.awebp?)

**我关注的人在 24h 之内关注的其他人**

**一般 MySQL 方案**：

*   查到所有关注用户（现有 set 缓存）:select followUserId from user_follow where userId = {$userId}
*   user_follow_xx 分表查询这些关注用户又关注的其他人

**GDB 方案**：

*   根据图例写出节点与边的结构即可：match (u1:User)-[:Attention]->(u2:User)-[:24HAttention]->(u3:User) return u3;

**方案对比：**

在 MySQL 方案中，由于关注数据达到上亿级别，在 MySQL 中做分表，是必要的处理，但是正是因为这个处理，使得这个业务场景的查询更加麻烦，需要拆开分别调用，复杂度可想而知。在 GDB 方案中，查询语句简洁明了，查询效率如下图：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a768018c6914e488cb264845a158813~tplv-k3u1fbpfcp-watermark.awebp?)

**如何使用 GDB**
============

**语法的学习**

*    Cypher：[neo4j.com/docs/cypher…](https://link.juejin.cn?target=https%3A%2F%2Fneo4j.com%2Fdocs%2Fcypher-manual%2F3.5%2F "https://neo4j.com/docs/cypher-manual/3.5/") 更类似于 SQL，更形象，便于理解，学习成本较低
    
*   gremlin：[tinkerpop-gremlin.cn/#traversal](https://link.juejin.cn?target=http%3A%2F%2Ftinkerpop-gremlin.cn%2F%23traversal "http://tinkerpop-gremlin.cn/#traversal") 更类似于 ORM，封装了许许多多链式方法，学习成本较高 
    

**遇到的问题 & 解决**

*   1.  唯一索引问题：为了确保数据的唯一性，我们通常会设置唯一性约束

创建索引语法：Create CONSTRAINT on (a:User) ASSERT a.userId IS UNIQUE - 在实践中阿里云的 GDB 索引可能会失效，需要采用特殊语法来更新数据： - - 类似于 MySQL 的 on duplicate key update，来实现存在即更新，不存在即插入：merge (n:User{userId: userId}) on create set n.isAllowLike = isAllowLike on match set n.isAllowLike = $isAllowLike return n.userId as userId

*   2.  二级查询效率问题：

尽量不使用二级查询的属性进行排序，可以根据业务将二级查询数量级降低，在结果代码中排序  
**Badcase 示例**：match (u:User)-[:Attention]->(c:User)-[a:Attention]->(t:User) where u.userId=userId and a.createTime > {TTFtime} return c.userId as cUserId, t.userId as tUserId order by a.createTime desc  
**Goodcase 实例**：match (u:User)-[:Attention]->(c:User)-[a:TTFAttention]->(t:User) where u.userId=$userId return c.userId as cUserId, t.userId as tUserId  

**未来应用**
========

下面通过一些示例，来扩展下应用场景。

**部署依赖图**

我们在版本发布的时候，通常会整理发布清单，其中最重要的一环，就是厘清依赖关系，有时候项目众多，依赖关系复杂通过表格等方式往往难以表达清楚。我们通过 GDB 将依赖关系表达得非常清晰。

下图是某版本部署图，有几点特征：

*   部署图变成了一个「森林」
*   「森林」中不同的「树」是可以独立发布的

*   通过「有向边」可以知道「树」的依赖关系
*   必须从「根节点」开始发布。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ef937ca8f2134071827bf6eddf03e728~tplv-k3u1fbpfcp-watermark.awebp?) **用户画像**

给自己弄了个用户画像，通过不同用户的画像相似程度，可以找到志同道合的朋友。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b36cd4d1a05841d1979d82e83cb2fd29~tplv-k3u1fbpfcp-watermark.awebp?)

**知识图谱**

想知道《权利的游戏》中各大家族的关系么？这类知识图谱可以非常方便的查到某人的家族关系。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42f17285d09d4b33af4f99b75a628abe~tplv-k3u1fbpfcp-watermark.awebp?)

**总结**
======

目前社区 GDB 服务，支持了上亿点和边的关系数据，上百的 QPS，平均 RT 在 28ms 左右，较好的支持了这部分业务场景。当然 GDB 肯定也不是万能的，相信这个世界上没有最好的技术，只有最适合当前应用场景的技术。

限于篇幅，这篇文档并没有与大家进行更深入的讨论，欢迎线下探讨。最后，希望这篇文章能给你带来一些灵感与思路，感谢阅读。

如果有帮助，可以留言，也可以关注 “得物技术微信” 公众号！