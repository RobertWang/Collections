> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/vr-joS1-5VI-b19zbp72aA)

额，MySQL 加的锁，和**事务隔离级别**相关，又和**索引**相关，尝试花 2 分钟讲讲看。  

_画外音：这 2 分钟需要的辅助知识，都已经附带了链接，贴心吧！_

**第一类，普通 select 加什么锁？**

（1）在读未提交 (Read Uncommitted)，读提交 (Read Committed, RC)，可重复读 (Repeated Read, RR) 这三种事务隔离级别下，普通 select 使用**快照读** (snpashot read)，不加锁，并发非常高；

（2）在串行化 (Serializable) 这种事务的隔离级别下，普通 select 会升级为 select ... in share mode;

**【快照读】**辅助阅读：

《[InnoDB，并发如此之高的原因](http://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651967047&idx=1&sn=b605fe50e6dd74ecad659c464a0e29ee&chksm=bd2d7b9b8a5af28d35c13e469e8e2c6a7082f00e608fd52d999fc0f7a6aec016faf2a8d748fa&scene=21#wechat_redirect)》

**【事务隔离级别】**辅助阅读：

《[InnoDB，巧妙的实现四种事务的隔离级别](http://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651967707&idx=1&sn=733da08d6c24a19a05bf3ba2ada96364&chksm=bd2d65078a5aec11ee28557355c736359ef7a1dc42629a22cc3274c44b4affd6a47859d903a6&scene=21#wechat_redirect)》  

****第二类，加锁 select 加什么锁？****

加锁 select 主要是指：

 - select ... for update

 - select ... in share mode

（1）如果，在唯一索引 (unique index) 上使用唯一的查询条件 (unique search condition)，会使用记录锁 (record lock)，而不会封锁记录之间的间隔，即不会使用间隙锁 (gap lock) 与临键锁 (next-key lock)；

**【记录锁，间隙锁，临键锁】**辅助阅读：

《[InnoDB 里的七种锁](http://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651967369&idx=1&sn=d639abf6772a72c25cc2537749258163&chksm=bd2d7a558a5af3436e0a435a3e7cd9f72f67c1269370be936a82dc0af483f47aefa48bd58d69&scene=21#wechat_redirect)》

举个栗子，假设有 InnoDB 表：

t(id PK, name);

表中有三条记录：

1, shenjian

2, zhangsan

3, lisi

SQL 语句：

select * from t where id=1 for update;

只会封锁记录，而不会封锁区间。

（2）其他的查询条件和索引条件，InnoDB 会封锁被扫描的索引范围，并使用间隙锁与临键锁，避免索引范围区间插入记录；

****第三类，**update 与 delete 加什么锁？**

（1）和加锁 select 类似，如果在唯一索引上使用唯一的查询条件来 update/delete，例如：

update t set name=xxx where id=1;

同时，会在插入区间加插入意向锁 (insert intention lock)，但这个并不会真正封锁区间，也不会阻止相同区间的不同 KEY 插入。

**【插入意向锁】**辅助阅读：

《[InnoDB 里的七种锁](http://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651967369&idx=1&sn=d639abf6772a72c25cc2537749258163&chksm=bd2d7a558a5af3436e0a435a3e7cd9f72f67c1269370be936a82dc0af483f47aefa48bd58d69&scene=21#wechat_redirect)》

了解不同 SQL 语句的加锁，对于分析多个事务之间的并发与互斥，以及事务死锁，非常有帮助。

_画外音：文章的参考资料为 MySQL 官网，以及楼主对 MySQL 的理解，版本基于 5.6，欢迎探讨。_

希望这 2 分钟，大家有收获。

公众号

**架构师之路** - 分享可落地的架构文章

**辅助阅读**：

《[InnoDB，并发如此之高的原因](http://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651967047&idx=1&sn=b605fe50e6dd74ecad659c464a0e29ee&chksm=bd2d7b9b8a5af28d35c13e469e8e2c6a7082f00e608fd52d999fc0f7a6aec016faf2a8d748fa&scene=21#wechat_redirect)》

《[InnoDB，巧妙的实现四种事务的隔离级别](http://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651967707&idx=1&sn=733da08d6c24a19a05bf3ba2ada96364&chksm=bd2d65078a5aec11ee28557355c736359ef7a1dc42629a22cc3274c44b4affd6a47859d903a6&scene=21#wechat_redirect)》

《[InnoDB 里的七种锁](http://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651967369&idx=1&sn=d639abf6772a72c25cc2537749258163&chksm=bd2d7a558a5af3436e0a435a3e7cd9f72f67c1269370be936a82dc0af483f47aefa48bd58d69&scene=21#wechat_redirect)》

《[索引，底层是如何实现的？](http://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651967146&idx=1&sn=e11b8613a0d18bab4bd2abcb63175ad6&chksm=bd2d7b768a5af26073e275fd6556f35b48098576bb69304f04d4b28066e0bc7eb10a948a79ee&scene=21#wechat_redirect)》

《[InnoDB，聚集索引与普通索引有什么不同？](http://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651968648&idx=1&sn=93dd534d59123ea27ba7f0ab1150ca7a&chksm=bd2d61548a5ae84240a8364fa184e91145aab9b43c94d5fbd2baba73fb782bff75bb840ccd6d&scene=21#wechat_redirect)》

谢转。