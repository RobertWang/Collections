> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247518355&idx=3&sn=390a81de37237a3cbc440372b9b29a20&chksm=ce0e3f10f979b6061db184b46bff0d35a7e0b89aeff3ed1f19731f8984017e23bc6d36f72837&mpshare=1&scene=1&srcid=0622ioqcYMrGPIWwNOpk0gwR&sharer_sharetime=1624349969673&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

### **引言**  

小 A 正在 balabala 写代码呢，DBA 小 B 突然发来了一条消息，“快看看你的用户特定信息表 T，里面的主键，也就是自增 id，都到 16 亿了，这才多久，在这样下去过不了多久主键就要超出范围了，插入就会失败，balabala......”  

我记得没有这么多，最多 1k 多万，count 了下，果然是 1100 万。原来运维是通过 auto_increment 那个值看的，就是说，表中有大量的删除插入操作，但是我大部分情况都是更新的，怎么会这样？

### **问题排查**

这张表是一个简单的接口服务在使用，每天大数据会统计一大批信息，然后推送给小 A，小 A 将信息更新到数据库中，如果是新数据就插入，旧数据就更新之前的数据，对外接口就只有查询了。  

很快，小 A 就排查了一遍自己的代码，没有删除的地方，也没有主动插入、更新 id 的地方，怎么会这样呢？难道是小 B 的原因，也不太可能，DBA 那边儿管理很多表，有问题的话早爆出来了，但问题在我这里哪里也没头绪。

小 A 又仔细观察了这 1000 多万已有的数据，将插入时间、id 作为主要观察字段，很快，发现了个问题，每天第一条插入的数据总是比前一天多 1000 多万，有时候递增的多，有时候递增的少，小 A 又将矛头指向了 DBA 小 B，将问题又给小 B 描述了一遍。

小 B 问了小 A，“你是是不是用了 REPLACE INTO... 语句”，这是怎么回事呢，原来 REPLACE INTO... 会对主键有影响。

##### **“REPLACE INTO ...” 对主键的影响**

假设有一张表 t1：

![](https://mmbiz.qpic.cn/mmbiz_png/icbViakEeV5qHeWkrhOGzc2QwAwPQFC8nA4UGDXgUr75Y2ND8VI9AWKKSo6SQlyfGJjPA8qGUlzgrXhKg6AM50oA/640?wx_fmt=png)

如果新建这张表，执行下面的语句，最后的数据记录如何呢？

![](https://mmbiz.qpic.cn/mmbiz_png/icbViakEeV5qHeWkrhOGzc2QwAwPQFC8nAGZYA8kJudKqic6W2nP6CHlmdyhRxhgic1MqSOv4icDibhvgjUwHp4b3jQQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/aVp1YC8UV0cJiaudg0RbFbO0p9iaupg5tYspQNn3ZmVyObQnYAxoj5ia6Z3tXfAsiaT3icWBtpawuKy4RE4mugMVU9w/640?wx_fmt=png)

原来， REPLACE INTO... 每次插入的时候如果唯一索引对应的数据已经存在，会删除原数据，然后重新插入新的数据，这也就导致 id 会增大，但实际预期可能是更新那条数据。

小 A 说：“我知道 replace 是这样，所有既没有用它”，但还是又排查了一遍，确实不是自己的问题，没有使用 REPLACE INTO...。

小 A 又双叒叕仔细的排查了一遍，还是没发现问题，就让小 B 查下 binlog 日志，看看是不是有什么奇怪的地方，查了之后还是没发现问题，确实存在跳跃的情况，但并没有实质性的问题。

下图中 @1 的值对应的是自增主键 id，用 (@2,@3) 作为唯一索引：

![](https://mmbiz.qpic.cn/mmbiz_png/aVp1YC8UV0cJiaudg0RbFbO0p9iaupg5tYwS9GqBq0sh1ibeutFujuV9UDr2f5cGSQuVmZY3gCkqeO50TCBhgjcRQ/640?wx_fmt=jpeg)

后来过了很久，小 B 给小 A 指了个方向，小 A 开始怀疑自己的插入更新语句 

INSERT...ON DUPLICATE KEY UPDATE... 了，查了许久，果然是这里除了问题。

##### **“INSERT ... ON DUPLICATE KEY UPDATE ...” 对主键的影响**

这个语句跟 REPLACE INTO... 类似，不过他并不会变更该条记录的主键，还是上面 t1 这张表，我们执行下面的语句，执行完结果是什么呢？

![](https://mmbiz.qpic.cn/mmbiz_png/icbViakEeV5qHeWkrhOGzc2QwAwPQFC8nAD6N9hcJzetzmIGDkBBKGib30MSkmhicr5vtKqxXHrAxbCpJEUmsh1CUg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/aVp1YC8UV0cJiaudg0RbFbO0p9iaupg5tYZ0q8qzHyXicBQgKdiacKIebXuibcPOfd8xBic2BY1amP5qeu9NfJYkH40A/640?wx_fmt=png)  

没错，跟小 A 预想的一样，主键并没有增加，而且 name 字段已经更新为想要的了，但是执行结果有条提示，引起了小 A 的注意：

> No errors; 2 rows affected, taking 10.7ms

明明更新了一条数据，为什么这里的影响记录条数是 2 呢？小 A，又看了下目前表中的 auto_increment：

![](https://mmbiz.qpic.cn/mmbiz_png/icbViakEeV5qHeWkrhOGzc2QwAwPQFC8nAUmGRdxHJQyrMKnYuoM9I7bibdcN2ibvWyxgXpoiakqA47ibCM9JTPGRLdw/640?wx_fmt=png)

竟然是 5`，这里本应该是 4 的。

也就是说，上面的语句，会跟 REPLACE INTO... 类似的会将自增 ID 加 1，但实际记录没有加，这是为什么呢?

查了资料之后，小 A 得知，原来，mysql 主键自增有个参数 innodb_autoinc_lock_mode，他有三种可能只 0, 1, 2，mysql5.1 之后加入的，默认值是 1，之前的版本可以看做都是 0。

可以使用下面的语句看当前是哪种模式：

1.  `select @@innodb_autoinc_lock_mode;`
    

小 A 使用的数据库默认值也是 1，当做简单插入（可以确定插入行数）的时候，直接将 auto_increment 加 1，而不会去锁表，这也就提高了性能。当插入的语句类似 insert into select ... 这种复杂语句的时候，提前不知道插入的行数，这个时候就要要锁表（一个名为 AUTO_INC 的特殊表锁）了，这样 auto_increment 才是准确的，等待语句结束的时候才释放锁。

还有一种称为 Mixed-mode inserts 的插入，比如 INSERT INTO t1 (c1,c2) VALUES (1,'a'), (NULL,'b'), (5,'c'), (NULL,'d')，其中一部分明确指定了自增主键值，一部分未指定，还有我们这里讨论的 INSERT ... ON DUPLICATE KEY UPDATE ... 也属于这种，这个时候会分析语句，然后按尽可能多的情况去分配 auto_incrementid，这个要怎么理解呢，我看下面这个例子：

![](https://mmbiz.qpic.cn/mmbiz_png/icbViakEeV5qHeWkrhOGzc2QwAwPQFC8nAgRVG2QRt6BiaWMiaDreuWOn9YvAjuqUMsLQv6HicAtdB9iapibsVwAqibYGg/640?wx_fmt=png)

此时数据表下一个自增 id 是 7：

```
deletefrom t1 where id in (2,3,4);

```

此时数据表只剩 1，5，6 了，自增 id 还是 7：

![](https://mmbiz.qpic.cn/mmbiz_png/icbViakEeV5qHeWkrhOGzc2QwAwPQFC8nAzoTiccaMj7J2NHwjdJaYgzBBjStsdbOxAib0Vc8qUwY61KfAHC0zFR1A/640?wx_fmt=png)

这里的自增 id 是多少呢？

上面的例子执行完之后表的下一个自增 id 是 10，你理解对了吗，因为最后一条执行的是一个 Mixed-mode inserts 语句，innoDB 会分析语句，然后分配三个 id，此时下一个 id 就是 10 了，但分配的三个 id 并不一定都使用。此处 * @总是迟到 [zongshichidao] * 多谢指出，看官方文档理解错了。

模式 0 的话就是不管什么情况都是加上表锁，等语句执行完成的时候在释放，如果真的添加了记录，将 auto_increment 加 1。

至于模式 2，什么情况都不加 AUTO_INC 锁，存在安全问题，当 binlog 格式设置为 Statement 模式的时候，从库同步的时候，执行结果可能跟主库不一致，问题很大。因为可能有一个复杂插入，还在执行呢，另外一个插入就来了，恢复的时候是一条条来执行的，就不能重现这种并发问题，导致记录 id 可能对不上。

至此，id 跳跃的问题算是分析完了，由于 innodb_autoinc_lock_mode 值是 1， INSERT...ON DUPLICATE KEY UPDATE... 是简单的语句，预先就可以计算出影响的行数，所以不管是否更新，这里都将 auto_increment 加 1（多行的话大于 1）。

如果将 innodb_autoinc_lock_mode 值改为 0，再次执行 INSERT...ON DUPLICATE KEY UPDATE... 的话，你会发现 auto_increment 并没有增加，因为这种模式直接加了 AUTO_INC 锁，执行完语句的时候释放，发现没有增加行数的话，不会增加自增 id 的。

##### **“INSERT ... ON DUPLICATE KEY UPDATE ...” 影响的行数是 1 为什么返回 2？**

为什么会这样呢，按理说影响行数就是 1 啊，看看官方文档的说明：

> With ON DUPLICATE KEY UPDATE, the affected-rows value per row is 1 if the row is inserted as a new row, 2 if an existing row is updated, and 0 if an existing row is set to its current values

官方明确说明了，插入影响 1 行，更新影响 2 行，0 的话就是存在且更新前后值一样。是不是很不好理解？

其实，你要这样想就好了，这是为了区分到底是插入了还是更新了，返回 1 表示插入成功，2 表示更新成功。

**解决方案**  

将 innodb_autoinc_lock_mode 设置为 0 肯定可以解决问题，但这样的话，插入的并发性可能会受很大影响，因此小 A 自己想着 DBA 也不会同意。经过考虑，目前准备了两种较为可能的解决方案：  

##### **修改业务逻辑**

修改业务逻辑，将 INSERT...ON DUPLICATE KEY UPDATE... 语句拆开，先去查询，然后去更新，这样就可以保证主键不会不受控制的增大，但增加了复杂性，原来的一次请求可能变为两次，先查询有没有，然后去更新。

##### **删除表的自增主键**

删除自增主键，让唯一索引来做主键，这样子基本不用做什么变动，只要确定目前的自增主键没有实际的用处即可，这样的话，插入删除的时候可能会影响效率，但对于查询多的情况来说，小 A 比较两种之后更愿意选择后者。

**结语**  

其实 INSERT...ON DUPLICATE KEY UPDATE... 这个影响行数是 2 的，小 A 很早就发现了，只是没有保持好奇心，不以为然罢了，没有深究其中的问题，这深究就起来会带出来一大串新知识，挺好，看来小 A 还是要对外界保持好奇心，保持敏感，这样才会有进步。

#### 作者：燕南飞 Liam

来自：https://segmentfault.com/a/1190000017268633

推荐阅读

_1._ [GitHub 上有什么好玩的项目？](http://mp.weixin.qq.com/s?__biz=MzUxNjg4NDEzNA==&mid=2247498662&idx=1&sn=0087c4f3b79ba3420e917e9b42d45eda&chksm=f9a2286fced5a1794eb9a73d0be7c2e16eaceabf3a0420647c40cb4202bd116d9a15dd57c008&scene=21#wechat_redirect)

_2._ [这个牛逼哄哄的数据库开源了](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247516775&idx=1&sn=43b3c7460718c40539512f56e4dd87fb&chksm=ce0e35e4f979bcf2c7c985dbabfe32f2a5f563e191d12a64325355e1c586853a4ab682077137&scene=21#wechat_redirect)

_3._ [SpringSecurity + JWT 实现单点登录](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247516734&idx=1&sn=966d166d3052f10b9310ff4014733669&chksm=ce0e35bdf979bcabbb75366e1bad32bdbf8f41d1dc80b8be6e39225bd708913ce2c93472f2fc&scene=21#wechat_redirect)

_4._ [100 道 Linux 常见面试题](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247516664&idx=1&sn=0dae1a8cf99c4bde8fd8c9c197769009&chksm=ce0e367bf979bf6dec769decc0194bb36de4caeae7947671aba1852fe6bac8e5dab7d20916a5&scene=21#wechat_redirect)