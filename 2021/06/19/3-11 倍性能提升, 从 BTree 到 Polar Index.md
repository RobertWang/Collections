> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653555932&idx=1&sn=fb9366ca0a628c88d3dc44351be0d051&chksm=81399d44b64e145235034bdf7bab907cfaaf13eeda7e6c4fa8be9044aaa215f691b06d02e76e&mpshare=1&scene=1&srcid=06199NxfeSETuxURdNlRXfZN&sharer_sharetime=1624092982721&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

首先再来回顾一下 InnoDB SMO 的加锁流程（简化起见，假设本次 SMO 只需分裂 leaf page）：

1.  对全局 index->lock 加 SX 锁
    
2.  从 root page 开始，以不加锁的方式向下遍历 non-leaf pages 至 level 2
    
3.  对 level 1 的 non-leaf page 加 X 锁
    
4.  对 level 0 的 leaf page 及其 left、right page 加 X 锁，完成 leaf page 的 SMO
    
5.  从 root page 开始，以不加锁的方式向下遍历至 leaf page 的 parent
    
6.  向 parent page 插入 SMO 中对应指向 new page 的 nodeptr
    
7.  释放所有锁，SMO 结束
    

1.  对于单个 SMO 来说，参与 SMO 的 leaf pages 及其 parent page 的 X 锁会从一开始加着直到 SMO 结束，这样的加锁粒度有些大，其实 SMO 也是分层、从下到上依次操作的，如上面流程中：
    
    步骤 4 先在 level 0 对 leaf page 做分裂，然后再在步骤 5 向 parent page 插入指向 new page 的 nodeptr，但其实在做步骤 4 的时候没必要先加着 parent page X 锁，同样再步骤 5 中也没必要还占着 leaf pages X 锁，这个问题在级联 SMO 场景（leaf page 分裂引发其路径上多个 non-leaf pages 分裂）更为明显，这样在读写混合场景下，SMO 路径上的读性能会受影响
    
2.  虽然 SMO 对 index->lock 加了 SX 锁，可以允许其他非 SMO 操作并发进来，但 SX 之间还是互斥的，也就是说多个 SMO 并不能并发，即使它们之间完全没有 page 交集，这样在高并发大写入压力下（剧烈触发 SMO）性能不理想
    

原先 BTree 之所以需要持有 Index latch 的原因是正常的搜索顺序是保证严格的自上而下, 自左向右, 但是 SMO 操作由于需要保持对 BTree 修改的原子性, 不能让其他线程访问到 BTree 的中间状态, 因此需要持有叶子加点去加父节点的 latch, 因此 SMO 操作出现了自下而上的加锁操作, 在编程实现中, 一旦出现了多个线程无法遵守同一严格的加锁顺序, 那么死锁就无法避免, 为了避免这样的冲突 InnoDB 通过将整个 BTree index latch, 从而 SMO 的时候, 不会有搜索操作进行.

Polar Index 的核心想法是把 SMO 操作分成了两个阶段.

在 Polar Index 中每一个 node 包含有一个 link page 指针, 指向他的 node.  以及 fence key 记录的是 link page 的最小值.

比如 split 阶段

阶段 1: 将一个 page 进行 split 操作,  然后建立一个 link 连接在两个 page 之间. 下图 Polar Index 就是这样的状态

阶段 2: 给父节点添加一个指针, 从父节点指向新创建的 page.

当然还可以有一个阶段 3 将两个 page 之间的 link 指针去掉.

在 Polar Index 中, 阶段 1 和阶段 2 的中间状态我们也认为是合理状态, 如果这个阶段实例 crash, 那么在 crash recovery 阶段可以识别当前 page 有 Link page, 那么会将 SMO 的下一个阶段继续完成, 从而保证 BTree 的完整性.

这样带来的优点是在 SMO 的过程中, 由于允许中间状态是合法状态, 那么就不需要为了防止出现中间状态的出现而需要持有叶子节点加父节点 latch 的过程. 因此就避免的自下而上的加锁操作, 从而就不需要 Index latch.

如下图对比 BTree 和 Polar Index.

![](https://mmbiz.qpic.cn/mmbiz_png/y7l9KJ42n2zlezGv3JUb0rjU4SLcHchWjFG9JU7GrNjUE4sPmt7jP2BW6n6tBN9BMtH8onVHQyTxGvJibg5ENfA/640?wx_fmt=png)

在去掉 Index latch 之后, 通过 latch coupling 从而保证每一次的修改都只需要在 btree 的某一层加 latch, 从而最大的减少了 latch 的粒度.

如下是具体执行 right split 的过程:

![](https://mmbiz.qpic.cn/mmbiz_png/y7l9KJ42n2zlezGv3JUb0rjU4SLcHchW7JTk8loKzfUWicydLdaOGnUP4TvdoGSX6X0ia69XNrPED6jNjRibNDlOw/640?wx_fmt=png)

**带来的收益是:**

1.  降低 SMO 的 page 加锁粒度，当前修改哪一层，就只对这一层相关的 page 加 X 锁，并且修改完之后立刻放锁再去修改其他层，这样读写并发就上来了。这样的做法要解决的问题就是：
    
    > 对 leaf page 做完分裂之后，放锁放锁去修改 parent，那么已经迁移到 new page 上的数据怎么被其他线程访问到呢？
    
    这里 Polar Index 采用了类似 Blink tree 的做法，给分裂的 leaf page 设置一个 high key，这个值为 new page 上最小的 rec，这样如果 leaf page 放 X 锁之后，从 parent 下来的其他读操作检测到这个 high key 之后，就知道如果要查找的目标 rec 在当前 leaf page 没找到并且大于等于 high key 的话，就去 next page（也就是 new page）上查找。
    
2.  去掉全局 index->lock，正常的读写及 SMO 不对 index->lock 加任何锁，这样写并发就能上来了。不过在具体实现中，不是简单的删掉代码那么容易，要解决去掉它之后各种各样的问题：
    
    > 遍历 BTree 的加锁方式
    
    InnoDB 在普通读、写操作时遍历 BTree 的方式：是从 root page 开始，将路径上所有 non-leaf pages 加 S 锁，然后占着 S 锁去加目标 leaf page 的 X 锁，加到之后释放 non-leaf pages 的 S 锁；在 SMO 是遍历 BTree 的方式是前面流程中的步骤 2。当我们去掉 index->lock，允许多个 SMO 并发起来，显然 SMO 的遍历方式是有问题的，因为在第一遍以无锁方式遍历 BTree 找到所有需要加 X 锁的 page 到第二遍遍历真正对这些 page 加锁之间，可能其他 SMO 已经修改了 BTree 结构。所以我们将遍历方式统一改成 lock coupling，同时最多占 2 层 page 锁，这样做的好处是不管是普通读、写还是 SMO 操作，在遍历 BTree 时对 non-leaf pages 的加锁区间都很小，进一步提高并发
    

除此之外，在具体实现中，还要解决大量问题，比如：

> 1.  多个 SMO 之间有重叠的 pages，如何解决冲突，避免死锁
>     
> 2.  对于左分裂、左合并这种右 -> 左的加锁，如何避免死锁
>     
> 3.  对于 non-leaf page 删除 leftmost rec 而触发其 parent 的级联删除如何处理
>     
> 4.  … …
>     

**总结**

在 InnoDB 里面, 依然有一个全局的 Index latch, 由于全局的 Index latch 存在会导致同一时刻在 Btree 中只有一个 SMO 能够发生, 从而导致性能无法提升.

Polar Index 通过将 SMO 操作分成两个阶段, 并保证中间状态的合理性, 从而避免了 Index latch. 从而保证任意时刻在 BTree 中只会持有一层 latch, 从而实现性能极大提升.

**参考阅读**  

*   [Kvrocks: 一款开源的企业级磁盘 KV 存储服务](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653555924&idx=1&sn=e018ed7e8dabc2a99bcdc322f9598211&chksm=81399d4cb64e145aa758742083ef7d4180e86d14cd6802875a0bd7f64f4ccec831e5419fd0a9&scene=21#wechat_redirect)  
    
*   [异地多活 paxos 实现：Multi-Master-Paxos-3](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653555895&idx=1&sn=6e1237d48487b36adebecde28704db61&chksm=81399d2fb64e14396d52aa71cd13a7e6b7bf02de218508905f61bc7d6234443fa2a081803baf&scene=21#wechat_redirect)  
    
*   [百度大规模 Service Mesh 落地实践](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653555866&idx=1&sn=8a8734e8d1d9b4093ee38206105189b7&chksm=81399d02b64e1414a83c3673f72be7e268e13f518bd5e82db3c5c870c0e9e2cf10fb7f69910f&scene=21#wechat_redirect)  
    
*   [春节红包活动如何应对 10 亿级流量？看大佬复盘总结](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653555824&idx=1&sn=48b13160ac74e851e654c95a9f9d8bfc&chksm=81399de8b64e14fe1bce230c88f0624bb57c6f2dbd1e43b60216526e49bbff9b21615f7de14d&scene=21#wechat_redirect)  
    
*   [微服务拆分之道](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653555796&idx=1&sn=03dd951bba1977ca7fc04e5e202568c2&chksm=81399dccb64e14da2fe29de49861a71ce84ec4b0cf4b98defffacb18d70e5e7788ac8c5bbaad&scene=21#wechat_redirect)