> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU3NDgxMzI0Mw==&mid=2247498391&idx=3&sn=3e487fe4252fa569800cd81eaaa44e97&chksm=fd2e1fc3ca5996d567a15f60022cdb7aca3c398dc1e03cef532d837f1c2dd79c1bd6ecb56e91&mpshare=1&scene=1&srcid=0708L4yFY5nmjJCCuWcaHxIp&sharer_sharetime=1625733762040&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

Redis 的 VM(虚拟内存)机制就是暂时把不经常访问的数据 (冷数据) 从内存交换到磁盘中，从而腾出宝贵的内存空间用于其它需要访问的数据(热数据)。通过 VM 功能可以实现冷热数据分离，使热数据仍在内存中、冷数据保存到磁盘。这样就可以避免因为内存不足而造成访问速度下降的问题。

Redis 提高数据库容量的办法有两种：一种是可以将数据分割到多个 Redis Server 上；另一种是使用虚拟内存把那些不经常访问的数据交换到磁盘上。需要特别注意的是 Redis 并没有使用 OS 提供的 Swap，而是自己实现。

Redis 为了保证查找的速度，只会将 value 交换出去，而在内存中保留所有的 Key。所以它非常适合 Key 很小，Value 很大的存储结构。如果 Key 很大，value 很小，那么 vm 可能还是无法满足需求。

VM 相关配置
=======

通过在 redis 的 redis.conf 文件里，设置 VM 的相关参数来实现数据在内存和磁盘之间 换入和 换出操作。

相关配置如下：

```
#开启vm功能
vm-enabled yes
#交换出来的value保存的文件路径
vm-swap-file /tmp/redis.swap
#设置当内存消耗达到上限时开始将value交换出来
vm-max-memory 1000000
#设置单个页面的大小，单位是字节
vm-page-size 32
#设置最多能交换保存多少个页到磁盘
vm-pages 13417728
#设置完成交换动作的工作线程数，设置为0表示不使用工作线程而使用主线程,这会以阻塞的方式来运行。建议设置成CPU核个数
vm-max-threads 4


```

redis 规定同一个数据页面只能[保存一个对象](http://mp.weixin.qq.com/s?__biz=MzU2MTI4MjI0MQ==&mid=2247504688&idx=2&sn=8f682c3311dd35377f5ec3329392dd9e&chksm=fc79be9ecb0e3788ca70d201967df7a60d231d70b366310f83f4d7e0c27dd6ecb7b42eda858f&scene=21#wechat_redirect)，但一个对象可以保存在多个数据页面中。在 redis 使用的内存没超过 vm-max-memory 时，是不会交换任何 value 到磁盘上的。当超过最大内存限制后，redis 会选择较老的对象 (如果两个对象一样老会优先交换比较大的对象) 将它从内存中移除，这样会更加节约内存。

对于 Redis 来说，一个数据页面只会保存一个对象，也就是一个 Value 值，所以应该将 vm-page-size 设置成大多数 value 可以保存进去。如果设置太小，一个 value 对象就会占用几个数据页面，如果设置太大，就会造成页面空闲空间浪费。

VM 的工作机制
========

redis 的 VM 的工作机制分为两种：一种是`vm-max-threads=0`，一种是`vm-max-threads>0`。

第一种：`vm-max-threads = 0`

**数据换出：** 主线程定期检查使用的内存大小，如果发现内存超出最大上限，会直接以阻塞的方式，将选中的对象 换出 到磁盘上 (保存到文件中)，并释放对象占用的内存，此过程会一直重复直到下面条件满足任意一条才结束：

1.  内存使用降到最大限制以下。
    
2.  设置的交换文件数量达到上限。
    
3.  几乎全部的对象都被交换到磁盘了。
    

**数据换入：** 当有 client 请求 key 对应的 value 已被换出到磁盘中时，主线程会以阻塞的方式从换出文件中加载对应的 value 对象，加载时此时会阻塞所有 client，然后再处理 client 的请求。这种方式会阻塞所有的 client。

第二种：`vm-max-threads > 0`

**数据换出：** 当主线程检测到使用内存超过最大上限，会将选中的要交换的数据放到一个队列中交由工作线程后台处理，主线程会继续处理 client 请求。

**数据换入：** 当有 client 请求 key 的对应的 value 已被换出到磁盘中时，主线程先阻塞当前 client，然后将加载对象的信息放到一个队列中，让工作线程去加载，此时进主线程继续处理其他 client 请求。加载完毕后工作线程通知主线程，主线程再执行被阻塞的 client 的命令。这种方式只阻塞单个 client。

总结
==

Redis 直接自己构建了 VM 机制 ，不会像一般的系统会调用系统函数处理，会浪费一定的时间去 移动 和 请求，而 Redis 不存在。这也是 Redis 能够那么快的一个原因。

_（感谢阅读，希望对你所有帮助)_

_来源：www.codenong.com/cs106843764_