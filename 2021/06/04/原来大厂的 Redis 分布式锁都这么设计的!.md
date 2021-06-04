> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA5ODM5MDU3MA==&mid=2650871191&idx=1&sn=b416fad0325198fe8f184d25cee8e8c2&chksm=8b67f2d2bc107bc452275b6440b41ff4b50876dc12d61b1f3fe0cf59f1b42972b4cc5a2c2ac8&scene=21#wechat_redirect)

1 本地锁
=====

常用的即 synchronize 或 Lock 等 JDK 自带的锁，只能锁住当前进程，仅适用于单体架构服务。而在分布式多服务实例场景下必须使用分布式锁。

2 分布式锁
======

2.1 分布式锁的原理
-----------

### 厕所占坑理论

可同时去一个地方 “占坑”：

*   占到，就执行逻辑
    
*   否则等待，直到释放锁
    

可通过自旋方式自旋

“占坑” 可以去 Redis、DB、任何所有服务都能访问的地方。

2.2 分布式锁演进
----------

### 一阶段

```
// 占分布式锁，去redis占坑
Boolean lock = redisTemplate.opsForValue().setIfAbsent("lock", "111");
if(lock) {
 //加锁成功... 执行业务
 Map<String, List<Catelog2Vo>> dataFromDb = getDataFromDb();
 redisTemplate . delete( key: "lock");//fHßti
 return dataF romDb ;
} else {
 // 加锁失败，重试。synchronized()
 // 休眠100ms重试
 // 自旋
 return getCatalogJsonFromDbwithRedisLock();
}



```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/DmibiaFiaAI4B3GR7EuRhXxLnwp24K9ncdamhkfHzTB8qeSqvgNcM9XDicUpAEZnKLce4D8zDkfmnlcmJHu151ASmw/640?wx_fmt=png)

#### 问题场景

*   setnx 占好了坑，但是业务代码异常或程序在执行过程中宕机，即没有执行成功删除锁逻辑，导致死锁
    

解决方案：设置锁的自动过期，即使没有删除，会自动删除。

### 阶段二

![](https://mmbiz.qpic.cn/sz_mmbiz_png/DmibiaFiaAI4B3GR7EuRhXxLnwp24K9ncdaT9RyLqicmGyp7wvxGInIqmlXpeoyUHdtLghS4icxfFpNL4JQV4yawFLQ/640?wx_fmt=png)

```
// 1. 占分布式锁，去redis占坑
Boolean lock = redisTemplate.opsForValue().setIfAbsent( "lock", "110")
if(lock) {
 // 加锁成功...执行业务
 
 // 突然断电
 
 // 2. 设置过期时间
 redisTemplate.expire("lock", timeout: 30, TimeUnit.SECONDS) ;
 Map<String, List<Catelog2Vo>> dataFromDb = getDataFromDb();
 //删除锁
 redisTemplate. delete( key; "lock");
 return dataFromDb;
} else {
 // 加锁失败...重试。synchronized ()
 // 休眠100ms重试
 // 自旋的方式
 return getCatalogJsonF romDbWithRedisLock();
}



```

#### 问题场景

*   setnx 设置好，正要去设置过期时间，宕机，又死锁
    

解决方案：设置过期时间和占位必须是原子操作。redis 支持使用 setNxEx 命令

### 阶段三

![](https://mmbiz.qpic.cn/sz_mmbiz_png/DmibiaFiaAI4B3GR7EuRhXxLnwp24K9ncdaXOgZN2HWHaW3MU077QDtC5AK17ce58ROsReEjEkFcc6BFqvejEFAGw/640?wx_fmt=png)

```
// 1. 分布式锁占坑
Boolean lock = redisTemplate.opsForValue().setIfAbsent("lock", "110", 300, TimeUnit.SECONDS);
if(lock)(
 // 加锁成功，执行业务
 
 // 2. 设置过期时间，必须和加锁一起作为原子性操作
 // redisTemplate. expire( "lock", з0, TimeUnit.SECONDS);
 Map<String, List<Catelog2Vo>> dataFromDb = getDataFromDb();
 // 删除锁
 redisTemplate.delete( key: "lock")
 return dataFromDb;
else {
 // 加锁失败，重试
 // 休眠100ms重试
 // 自旋
 return getCatalogJsonFromDbithRedislock()
}



```

### 阶段四

![](https://mmbiz.qpic.cn/sz_mmbiz_png/DmibiaFiaAI4B3GR7EuRhXxLnwp24K9ncdal6WGw5BP5zo4xibrrUlnicjQ4FqF5icdDCWsdXPibxZJPvL8v0BjMWkAyA/640?wx_fmt=png)

已经拿到了 lockvalue ，有了 UUID，但是过期了现在！其他人拿到所锁设置了新值，于是 if 后将别人的锁删了！！也就是删除锁不是原子操作。

```
Map<String, List<Catelog2Vo>> dataFromDb = getDataFromDb();
String lockValue = redisTemplate.opsForValue().get("lock");
if(uuid.equals(lockValue)) {
 // 删除我自己的锁
 redisTemplate.delete("lock");
}


```

#### 问题场景

*   如果正好判断是当前值，正要删除锁时，锁已过期，别人已设置成功新值。那删除的就是别人的锁.
    
*   解决方案
    

删除锁必须保证原子性。使用 redis+Lua 脚本。

### 阶段五

*   确保加锁 / 解锁都是原子操作 ![](https://mmbiz.qpic.cn/sz_mmbiz_png/DmibiaFiaAI4B3GR7EuRhXxLnwp24K9ncdaWLw38APeMmxQccLlicablqG1IUFxVSJLWx0vb0VibH195L971Gd1OWnw/640?wx_fmt=png)
    

```
String script = 
 "if redis.call('get', KEYS[1]) == ARGV[1] 
  then return redis.call('del', KEYS[1]) 
 else 
  return 0 
 end";


```

保证加锁【占位 + 过期时间】和删除锁【判断 + 删除】的原子性。更难的事情，锁的自动续期。

总结
==

其实更麻烦的事情，还有锁的自动续期。所以不管是大厂还是中小型公司，我们都是直接选择解决了这些问题的 Redisson！不重复造轮子，但也要知道该框架到底解决了哪些问题，方便我们遇到问题时也能快速排查定位。

- EOF -

推荐阅读  点击标题可跳转

1、[几率大的 Redis 面试题（含答案）](http://mp.weixin.qq.com/s?__biz=MzA5ODM5MDU3MA==&mid=2650869443&idx=2&sn=f14d9d264b5283b724a298d92b27a477&chksm=8b67e986bc106090a832357deefadcc83565f4f2ae2dc97ddbd8a206e40ec8199a0f2c58c4b1&scene=21#wechat_redirect)

2、[Redis 秒杀实战](http://mp.weixin.qq.com/s?__biz=MzA5ODM5MDU3MA==&mid=2650868046&idx=1&sn=e8a2519905475776b061f98d12815e5e&chksm=8b67ee0bbc10671dbec55063670fc4a500bd12f992c6095edbcd7aa0ce9796850fd523a4616e&scene=21#wechat_redirect)

3、[](http://mp.weixin.qq.com/s?__biz=MzA5ODM5MDU3MA==&mid=2650868046&idx=1&sn=e8a2519905475776b061f98d12815e5e&chksm=8b67ee0bbc10671dbec55063670fc4a500bd12f992c6095edbcd7aa0ce9796850fd523a4616e&scene=21#wechat_redirect)[QPS 过万，Redis 大量连接超时怎么解决？](http://mp.weixin.qq.com/s?__biz=MzA5ODM5MDU3MA==&mid=2650868459&idx=1&sn=e8e3a0926f9a378eb1b0d538d58b6d5b&chksm=8b67edaebc1064b8f37a8c0d58a377f48604dae95f98ef23deed97c6d12fc8fca863555d2f8c&scene=21#wechat_redirect)

看完本文有收获？请转发分享给更多人  

点赞和在看就是最大的支持❤️