> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU0OTE4MzYzMw==&mid=2247518045&idx=3&sn=6ca73341b5a6df3bcf4de0f4975203ef&chksm=fbb10ea3ccc687b519494b54dc62956367155ffda0e3c030f8591a227c58e7395ee91454ff3c&scene=132#wechat_redirect)

> 作者：今天你敲代码了吗  
> 链接：https://www.jianshu.com/p/b6953745e341

在分布式系统中，为保证同一时间只有一个客户端可以对共享资源进行操作，需要对共享资源加锁来实现，常见有三种方式：

*   基于数据库实现分布式锁
    
*   基于 Redis 实现分布式锁
    
*   基于 Zookeeper 实现分布式锁
    

高并发下数据库锁性能太差，本文不做探究。仅针对 Redis 和 Zookeeper 实现的分布式锁进行分析。

实现一个分布式锁应该具备的特性：

*   高可用、高性能的获取锁与释放锁
    
*   在分布式系统环境下，一个方法或者变量同一时间只能被一个线程操作
    
*   具备锁失效机制，网络中断或宕机无法释放锁时，锁必须被删除，防止死锁
    
*   具备阻塞锁特性，即没有获取到锁，则继续等待获取锁
    
*   具备非阻塞锁特性，即没有获取到锁，则直接返回获取锁失败
    
*   具备可重入特性，一个线程中可以多次获取同一把锁，比如一个线程在执行一个带锁的方法，该方法中又调用了另一个需要相同锁的方法，则该线程可以直接执行调用的方法，而无需重新获得锁
    

先上结论，Redis 在锁时间限制和缓存一致性存在一定问题，Zookeeper 在可靠性上强于 Redis，只是效率相对较低，开发人员需要根据实际需求进行技术选型。

**单机情况下：**

### 1. Redis 单机实现分布式锁

**1.1 Redis 加锁**

```
//SET resource_name my_random_value NX PX 30000
String result = jedis.set(key, value, "NX", "PX", 30000);
if ("OK".equals(result)) {
       return true; //代表获取到锁
}
return false;
```

加锁就一行代码：jedis.set(String key, String value, String nxxx, String expx, int time)，这个 set() 方法一共有五个形参：

*   第一个为 key，使用 key 来当锁，因为 key 是唯一的。
    
*   第二个为 value，是由客户端生成的一个随机字符串，相当于是客户端持有锁的标志。用于标识加锁和解锁必须是同一个客户端。
    
*   第三个为 nxxx，传的是 NX，意思是 SET IF NOT EXIST，即当 key 不存在时，进行 set 操作；若 key 已经存在，则不做任何操作。
    
*   第四个为 expx，传的是 PX，意思是我们要给这个 key 加一个过期的设置，具体时间由第五个参数决定。
    
*   第五个为 time，与第四个参数相呼应，代表 key 的过期时间，如上 30000 表示这个锁有一个 30 秒的自动过期时间。
    

**1.2 Redis 解锁**

解锁时，为了防止客户端 1 获得的锁，被客户端 2 给释放，需要采用的 Lua 脚本来释放锁：

```
final Long RELEASE_SUCCESS = 1L;
//采用Lua脚本来释放锁
String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
Object result = jedis.eval(script, Collections.singletonList(lockKey), Collections.singletonList(requestId));
if (RELEASE_SUCCESS.equals(result)) {
    return true;
}
return false;
```

在执行这段 Lua 脚本的时候，KEYS[1] 的值为 key，ARGV[1] 的值为 value。原理就是先获取锁对应的 value 值，保证和客户端传进去的 value 值相等，这样就能避免自己的锁被其他人释放。另外，采取 Lua 脚本操作保证了原子性。如果不是原子性操作，则有了下述情况出现：

![](https://mmbiz.qpic.cn/mmbiz_jpg/fEsWkVrSk578p9jibyuOrHgYYCibbxiackrEMNxaRHGhicRCLtZqdnDoysqflIQ6kNA5HFZibeCzibXWHJiaibtTJ9D90w/640?wx_fmt=jpeg)

**1.3 Redis 加锁过期时间设置问题**

理想情况是客户端 Redis 加锁后，完成一系列业务操作，顺利在锁过期时间前释放掉锁，这个分布式锁的设置是有效的。但是如果客户端在操作共享资源的过程中，因为长期阻塞的原因，导致锁过期，那么接下来访问共享资源就变得不再安全。

### 2. Zookeeper 单机实现分布式锁

**2.1 Curator 实现 Zookeeper 加解锁**

使用 Apache 开源的 curator 可实现 Zookeeper 分布式锁。

可以通过调用 InterProcessLock 接口提供的几个方法来实现加锁、解锁。

```
/**
* 获取锁、阻塞等待、可重入
*/
public void acquire() throws Exception;

/**
* 获取锁、阻塞等待、可重入、超时则获取失败
*/
public boolean acquire(long time, TimeUnit unit) throws Exception;

/**
* 释放锁
*/
public void release() throws Exception;

/**
* Returns true if the mutex is acquired by a thread in this JVM
*/
boolean isAcquiredInThisProcess();
```

**2.2 Zookeeper 加锁实现原理**

Zookeeper 的分布式锁原理是利用了临时节点 (EPHEMERAL) 的特性。其实现原理：

*   创建一个锁目录 lock
    
*   线程 A 获取锁会在 lock 目录下，创建临时顺序节点
    
*   获取锁目录下所有的子节点，然后获取比自己小的兄弟节点，如果不存在，则说明当前线程顺序号最小，获得锁
    
*   线程 B 创建临时节点并获取所有兄弟节点，判断自己不是最小节点，设置监听 (watcher) 比自己次小的节点（只关注比自己次小的节点是为了防止发生“羊群效应”）
    
*   线程 A 处理完，删除自己的节点，线程 B 监听到变更事件，判断自己是最小的节点，获得锁
    

由于节点的临时属性，如果创建 znode 的那个客户端崩溃了，那么相应的 znode 会被自动删除。这样就避免了设置过期时间的问题。

**2.3 GC 停顿导致临时节点释放问题**

但是使用临时节点又会存在另一个问题：Zookeeper 如果长时间检测不到客户端的心跳的时候 (Session 时间)，就会认为 Session 过期了，那么这个 Session 所创建的所有的 ephemeral 类型的 znode 节点都会被自动删除。

![](https://mmbiz.qpic.cn/mmbiz_jpg/fEsWkVrSk578p9jibyuOrHgYYCibbxiackrXC7Zs1zfngfibXFib5cZOJ4j5L3mibPwwceSyru1O4KicQdL7czwia2p6lg/640?wx_fmt=jpeg)

如上图所示，客户端 1 发生 GC 停顿的时候，Zookeeper 检测不到心跳，也是有可能出现多个客户端同时操作共享资源的情形。当然，你可以说，我们可以通过 JVM 调优，避免 GC 停顿出现。但是注意了，我们所做的一切，只能尽可能避免多个客户端操作共享资源，无法完全消除。

**集群情况下：**

### 3. Redis 集群下分布式锁存在问题

**3.1 集群 Master 宕机导致锁丢失**

为了 Redis 的高可用，一般都会给 Redis 的节点挂一个 slave, 然后采用哨兵模式进行主备切换。但由于 Redis 的主从复制（replication）是异步的，这可能会出现在数据同步过程中，**master 宕机，slave 来不及同步数据就被选为 master，从而数据丢失**。具体流程如下所示：

*   (1) 客户端 1 从 Master 获取了锁。
    
*   (2)Master 宕机了，存储锁的 key 还没有来得及同步到 Slave 上。
    
*   (3)Slave 升级为 Master。
    
*   (4) 客户端 1 的锁丢失，客户端 2 从新的 Master 获取到了对应同一个资源的锁。
    

**3.2 Redlock 算法**

为了应对这个情形， Redis 作者 antirez 基于分布式环境下提出了一种更高级的分布式锁的实现方式：**Redlock**。

antirez 提出的 redlock 算法大概是这样的：

在 Redis 的分布式环境中，我们假设有 N 个 Redis master。这些节点**完全互相独立，不存在主从复制或者其他集群协调机制**。我们确保将在 N 个实例上使用与在 Redis 单实例下相同方法获取和释放锁。现在我们假设有 5 个 Redis master 节点 (官方文档里将 N 设置成 5，其实大等于 3 就行)，同时我们需要在 5 台服务器上面运行这些 Redis 实例，这样保证他们不会同时都宕掉。

为了取到锁，客户端应该执行以下操作:

*   (1) 获取当前 Unix 时间，以毫秒为单位。
    
*   (2) 依次尝试从 5 个实例，使用相同的 key 和**具有唯一性的 value**（例如 UUID）获取锁。当向 Redis 请求获取锁时，客户端应该设置一个网络连接和响应超时时间，这个超时时间应该小于锁的失效时间。例如你的锁自动失效时间为 10 秒，则超时时间应该在 5-50 毫秒之间。这样可以避免服务器端 Redis 已经挂掉的情况下，客户端还在死死地等待响应结果。如果服务器端没有在规定时间内响应，客户端应该尽快尝试去另外一个 Redis 实例请求获取锁。
    
*   (3) 客户端使用当前时间减去开始获取锁时间（步骤 1 记录的时间）就得到获取锁使用的时间。**当且仅当从大多数**（N/2+1，这里是 3 个节点）**的 Redis 节点都取到锁，并且使用的时间小于锁失效时间时，锁才算获取成功**。
    
*   (4) 如果取到了锁，key 的真正有效时间等于有效时间减去获取锁所使用的时间（步骤 3 计算的结果）。
    
*   (5) 如果因为某些原因，获取锁失败（没有在至少 N/2+1 个 Redis 实例取到锁或者取锁时间已经超过了有效时间），客户端应该在**所有的 Redis 实例上进行解锁**（即便某些 Redis 实例根本就没有加锁成功，防止某些节点获取到锁但是客户端没有得到响应而导致接下来的一段时间不能被重新获取锁）。
    

redisson 已经有对 redlock 算法封装，如下是调用代码示例：

```
Config config = new Config();
config.useSentinelServers().addSentinelAddress("127.0.0.1:6369","127.0.0.1:6379", "127.0.0.1:6389")
        .setMasterName("masterName")
        .setPassword("password").setDatabase(0);
RedissonClient redissonClient = Redisson.create(config);
// 还可以getFairLock(), getReadWriteLock()
RLock redLock = redissonClient.getLock("REDLOCK_KEY");
boolean isLock;
try {
    isLock = redLock.tryLock();
    // 500ms拿不到锁, 就认为获取锁失败。10000ms即10s是锁失效时间。
    isLock = redLock.tryLock(500, 10000, TimeUnit.MILLISECONDS);
    if (isLock) {
        //TODO if get lock success, do something;
    }
} catch (Exception e) {
} finally {
    // 无论如何, 最后都要解锁
    redLock.unlock();
}
```

**3.3 Redlock 未完全解决问题**

Redlock 算法细想一下还存在下面的问题：**节点崩溃重启，会出现多个客户端持有锁** 假设一共有 5 个 Redis 节点：A, B, C, D, E。设想发生了如下的事件序列：

*   (1) 客户端 1 成功锁住了 A, B, C，获取锁成功（但 D 和 E 没有锁住）。
    
*   (2) 节点 C 崩溃重启了，但客户端 1 在 C 上加的锁没有持久化下来，丢失了。
    
*   (3) 节点 C 重启后，客户端 2 锁住了 C, D, E，获取锁成功。
    

这样，客户端 1 和客户端 2 同时获得了锁（针对同一资源）。

为了应对节点重启引发的锁失效问题，redis 的作者 antirez 提出了**延迟重启**的概念，即一个节点崩溃后，先不立即重启它，而是等待一段时间再重启，等待的时间大于锁的有效时间。采用这种方式，这个节点在重启前所参与的锁都会过期，它在重启后就不会对现有的锁造成影响。这其实也是通过人为补偿措施，降低不一致发生的概率。**时间跳跃问题**

*   (1) 假设一共有 5 个 Redis 节点：A, B, C, D, E。设想发生了如下的事件序列：
    
*   (2) 客户端 1 从 Redis 节点 A, B, C 成功获取了锁（多数节点）。由于网络问题，与 D 和 E 通信失败。
    
*   (3) 节点 C 上的时钟发生了向前跳跃，导致它上面维护的锁快速过期。
    
*   (4) 客户端 2 从 Redis 节点 C, D, E 成功获取了同一个资源的锁（多数节点）。
    
*   (5) 客户端 1 和客户端 2 现在都认为自己持有了锁。
    

为了应对始终跳跃引发的锁失效问题，redis 的作者 antirez 提出了应该禁止人为修改系统时间，使用一个不会进行 “跳跃” 式调整系统时钟的 ntpd 程序。这也是通过人为补偿措施，降低不一致发生的概率。**超时导致锁失效问题** RedLock 算法并没有解决，操作共享资源超时，导致锁失效的问题。回忆一下 RedLock 算法的过程，如下图所示

![](https://mmbiz.qpic.cn/mmbiz_jpg/fEsWkVrSk578p9jibyuOrHgYYCibbxiackrRjbH8qFo7icKSvzfAficuVoT0BfjDB9DcJNH9VLlS5qqzGZ2HdzarSXw/640?wx_fmt=jpeg)

如图所示，我们将其分为上下两个部分。对于上半部分框图里的步骤来说，无论因为什么原因发生了延迟，RedLock 算法都能处理，客户端不会拿到一个它认为有效，实际却失效的锁。然而，对于下半部分框图里的步骤来说，如果发生了延迟导致锁失效，都有可能使得客户端 2 拿到锁。因此，RedLock 算法并没有解决该问题。

### 4. Zookeeper 集群下分布式锁可靠性分析

**4.1 Zookeeper 的写数据的原理**

Zookeeper 在集群部署中，Zookeeper 节点数量一般是奇数，且一定大等于 3。下面是 Zookeeper 的写数据的原理：

![](https://mmbiz.qpic.cn/mmbiz_jpg/fEsWkVrSk578p9jibyuOrHgYYCibbxiackr4LRXVR77HZ4SrmOH5UaLbrOOSKibA4u6GlPjBnelazA83sJgcUFAaVg/640?wx_fmt=jpeg)

那么写数据流程步骤如下：

*   (1) 在 Client 向 Follwer 发出一个写的请求
    
*   (2)Follwer 把请求发送给 Leader
    
*   (3)Leader 接收到以后开始发起投票并通知 Follwer 进行投票
    
*   (4)Follwer 把投票结果发送给 Leader，**只要半数以上返回了 ACK 信息，就认为通过**
    
*   (5)Leader 将结果汇总后如果需要写入，则开始写入同时把写入操作通知给 Leader，然后 commit;
    
*   (6)Follwer 把请求结果返回给 Client
    

还有一点，Zookeeper 采取的是**全局串行化操作。**

**4.2 集群模式下 Zookeeper 可靠性分析**

下面列出 Redis 集群下分布式锁可能存在的问题，判断其在 Zookeeper 集群下是否会存在：

**集群同步**

*   client 给 Follwer 写数据，可是 Follwer 却宕机了，会出现数据不一致问题么？不可能，这种时候，client 建立节点失败，根本获取不到锁。
    
*   client 给 Follwer 写数据，Follwer 将请求转发给 Leader，Leader 宕机了，会出现不一致的问题么？不可能，这种时候，Zookeeper 会选取新的 leader，继续上面的提到的写流程。
    

总之，采用 Zookeeper 作为分布式锁，你要么就获取不到锁，一旦获取到了，必定节点的数据是一致的，不会出现 redis 那种异步同步导致数据丢失的问题。**时间跳跃问题** Zookeeper 不依赖全局时间，不存在该问题。**超时导致锁失效问题** Zookeeper 不依赖有效时间，不存在该问题。

### 5. 锁的其他特性比较

redis 的读写性能比 Zookeeper 强太多，如果在高并发场景中，使用 Zookeeper 作为分布式锁，那么会出现获取锁失败的情况，存在性能瓶颈。

Zookeeper 可以实现读写锁，Redis 不行。

Zookeeper 的 watch 机制, 客户端试图创建 znode 的时候，发现它已经存在了，这时候创建失败, 那么进入一种等待状态，当 znode 节点被删除的时候，Zookeeper 通过 watch 机制通知它，这样它就可以继续完成创建操作（获取锁）。这可以让分布式锁在客户端用起来就像一个本地的锁一样：加锁失败就阻塞住，直到获取到锁为止。这套机制，redis 无法实现。