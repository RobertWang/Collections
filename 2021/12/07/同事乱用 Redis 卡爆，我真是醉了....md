> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzkxNzI2ODY4OQ==&mid=2247492102&idx=2&sn=80530587dbeeb70730567c789438bea7&chksm=c14193b8f6361aae4419850a0323a4f6ded46008b59a26c21e523aab8e72bb6a48b2f55bc47e&mpshare=1&scene=1&srcid=1206IdVwV4tVrGbUfZOvNfry&sharer_sharetime=1638764220925&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

  

  

来源：my.oschina.net/xiaomu0082/blog/2990388

首先说下问题现象：内网sandbox环境API持续1周出现应用卡死，所有api无响应现象

刚开始当测试抱怨环境响应慢的时候 ，我们重启一下应用，应用恢复正常，于是没做处理。但是后来问题出现频率越来越频繁，越来越多的同事开始抱怨，于是感觉代码可能有问题，开始排查。

top 命令
------

首先发现开发的本地ide没有发现问题，应用卡死时候数据库，redis都正常，并且无特殊错误日志。开始怀疑是sandbox环境机器问题（测试环境本身就很脆!_!）

于是ssh上了服务器 执行以下命令

top

![图片](https://mmbiz.qpic.cn/mmbiz/6mychickmupWLhh8Mf5kpgQac9FOJMTfluka2T75ibq6MWbKXFE8PfK8n6A9lpUS3IYwsjsibLtBGibTEJ4Yx41O4Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图片

这时发现机器还算正常，于是打算看下jvm 堆栈信息

先看下问题应用比较耗资源的线程

执行 `top -H -p 12798`

![图片](https://mmbiz.qpic.cn/mmbiz/6mychickmupWLhh8Mf5kpgQac9FOJMTflUtvWiat1f5biaBJoY9bsmbhKcsketD0zGWwKgUshctsSpyd2WTX23SlA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图片

找到前3个相对比较耗资源的线程

jstack 命令
---------

jstack 查看堆内存

```
jstack 12798 |grep 12799的16进制 31ff


```

![图片](https://mmbiz.qpic.cn/mmbiz/6mychickmupWLhh8Mf5kpgQac9FOJMTflaMOOYGWib49cGV5VJQvUPqLt2e6I5iaG4SJ78BY6qvoUaC4LPgZEcLgQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图片

没看出什么问题，上下10行也看看，于是执行

![图片](https://mmbiz.qpic.cn/mmbiz/6mychickmupWLhh8Mf5kpgQac9FOJMTflcjYf2Qtyno4AkZ62eCxmNwMpUtELF1VnZsUHw1Baqc48arynFCPXicQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图片

看到一些线程都是处于lock状态。但没有出现业务相关的代码，忽略了。这时候没有什么头绪。思考一番。决定放弃这次卡死状态的机器

为了保护事故现场 先 dump了问题进程所有堆内存，然后debug模式重启测试环境应用，打算问题再显时直接远程debug问题机器

Tomcat 远程 debug
---------------

第二天问题再现，于是通知运维nginx转发拿掉这台问题应用，自己远程debug tomcat。

自己随意找了一个接口，断点在接口入口地方，悲剧开始，什么也没有发生！API等待服务响应，没进断点。这时候有点懵逼，冷静了一会,在入口之前的aop地方下了个断点，再debug一次，这次进了断点，f8 N次后发现在执行redis命令的时候卡主了。继续跟，最后在到jedis的一个地方发现问题：

```
/**
 * Returns a Jedis instance to be used as a Redis connection. The instance can be newly created or retrieved from a
 * pool.
 *
 * @return Jedis instance ready for wrapping into a {@link RedisConnection}.
 */
protected Jedis fetchJedisConnector() {
   try {
      if (usePool && pool != null) {
         return pool.getResource();
      }
      Jedis jedis = new Jedis(getShardInfo());
      // force initialization (see Jedis issue #82)
      jedis.connect();
      return jedis;
   } catch (Exception ex) {
      throw new RedisConnectionFailureException("Cannot get Jedis connection", ex);
   }
}


```

上面pool.getResource()后线程开始wait

```
public T getResource() {
  try {
    return internalPool.borrowObject();
  } catch (Exception e) {
    throw new JedisConnectionException("Could not get a resource from the pool", e);
  }
}


```

return internalPool.borrowObject(); 这个代码应该是一个租赁的代码，接着跟

```
public T borrowObject(long borrowMaxWaitMillis) throws Exception {
    this.assertOpen();
    AbandonedConfig ac = this.abandonedConfig;
    if (ac != null && ac.getRemoveAbandonedOnBorrow() && this.getNumIdle() < 2 && this.getNumActive() > this.getMaxTotal() - 3) {
        this.removeAbandoned(ac);
    }

    PooledObject<T> p = null;
    boolean blockWhenExhausted = this.getBlockWhenExhausted();
    long waitTime = 0L;

    while(p == null) {
        boolean create = false;
        if (blockWhenExhausted) {
            p = (PooledObject)this.idleObjects.pollFirst();
            if (p == null) {
                create = true;
                p = this.create();
            }

            if (p == null) {
                if (borrowMaxWaitMillis < 0L) {
                    p = (PooledObject)this.idleObjects.takeFirst();
                } else {
                    waitTime = System.currentTimeMillis();
                    p = (PooledObject)this.idleObjects.pollFirst(borrowMaxWaitMillis, TimeUnit.MILLISECONDS);
                    waitTime = System.currentTimeMillis() - waitTime;
                }
            }

            if (p == null) {
                throw new NoSuchElementException("Timeout waiting for idle object");
            }


```

其中有段代码

```
if (p == null) {
    if (borrowMaxWaitMillis < 0L) {
        p = (PooledObject)this.idleObjects.takeFirst();
    } else {
        waitTime = System.currentTimeMillis();
        p = (PooledObject)this.idleObjects.pollFirst(borrowMaxWaitMillis, TimeUnit.MILLISECONDS);
        waitTime = System.currentTimeMillis() - waitTime;
    }
}


```

borrowMaxWaitMillis<0会一直执行，然后一直循环了 开始怀疑这个值没有配置

找到redis pool配置，发现确实没有配置MaxWaitMillis，配置后else代码也是一个Exception 并不能解决问题

继续F8

```
public E takeFirst() throws InterruptedException {
    this.lock.lock();

    Object var2;
    try {
        Object x;
        while((x = this.unlinkFirst()) == null) {
            this.notEmpty.await();
        }

        var2 = x;
    } finally {
        this.lock.unlock();
    }

    return var2;
}


```

到这边 发现lock字眼，开始怀疑所有请求api都被阻塞了

于是再次ssh 服务器 安装 arthas ,(Arthas 是Alibaba开源的Java诊断工具)

执行thread命令

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/6mychickmupWLhh8Mf5kpgQac9FOJMTflNvTulQEibOTPb9H6aicqRkibUWTVbabzT8Qs7zgZxKvuHhxpz38cBLGUg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图片

发现大量http-nio的线程waiting状态,http-nio-8083-exec-这个线程其实就是出来http请求的tomcat线程

arthas
------

随意找一个线程查看堆内存

thread -428

![图片](https://mmbiz.qpic.cn/mmbiz/6mychickmupWLhh8Mf5kpgQac9FOJMTflRZicJ9VOLLXUo8B3TWLibleM4mldY1jkhek1JSQpB67eCbeDLcSX1QKQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图片

这是能确认就是api一直转圈的问题,就是这个redis获取连接的代码导致的，

解读这段内存代码 所有线程都在等 @53e5504e这个对象释放锁。于是jstack 全局搜了一把53e5504e ，没有找到这个对象所在线程。

自此。问题原因能确定是 redis连接获取的问题。但是什么原因造成获取不到连接的还不能确定

再次执行 arthas 的thread -b (thread -b, 找出当前阻塞其他线程的线程)

![图片](https://mmbiz.qpic.cn/mmbiz/6mychickmupWLhh8Mf5kpgQac9FOJMTfl1ibpK0SyYVLDBWict5FicF19ItMNJLZXLmz3CvUW5V7u27BBIAyV4yy1w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图片

没有结果。这边和想的不一样，应该是能找到一个阻塞线程的，于是看了下这个命令的文档，发现有下面的一句话

![图片](https://mmbiz.qpic.cn/mmbiz/6mychickmupWLhh8Mf5kpgQac9FOJMTfltVqDsaQYtKvaBiag2ggY0fsia7H1pG5q26gCXaXMv6W6KHXG0F7CQ76A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图片

好吧，我们刚好是后者。。。。

减短连接超时时间
--------

再次整理下思路。这次修改redis pool 配置，将获取连接超时时间设置为2s,然后等问题再次复现时观察应用最后正常时干过什么。

添加一下配置

```
JedisConnectionFactory jedisConnectionFactory = new JedisConnectionFactory();
.......
JedisPoolConfig config = new JedisPoolConfig();
config.setMaxWaitMillis(2000);
.......
jedisConnectionFactory.afterPropertiesSet();


```

重启服务，等待。。。。

又过一天，再次复现

ssh 服务器，检查tomcat accesslog ,发现大量api 请求出现500，

```
org.springframework.data.redis.RedisConnectionFailureException: Cannot get Jedis connection; nested exception is redis.clients.jedis.exceptions.JedisConnectionException: Could not get a resource fr
om the pool
    at org.springframework.data.redis.connection.jedis.JedisConnectionFactory.fetchJedisConnector(JedisConnectionFactory.java:140)
    at org.springframework.data.redis.connection.jedis.JedisConnectionFactory.getConnection(JedisConnectionFactory.java:229)
    at org.springframework.data.redis.connection.jedis.JedisConnectionFactory.getConnection(JedisConnectionFactory.java:57)
    at org.springframework.data.redis.core.RedisConnectionUtils.doGetConnection(RedisConnectionUtils.java:128)
    at org.springframework.data.redis.core.RedisConnectionUtils.getConnection(RedisConnectionUtils.java:91)
    at org.springframework.data.redis.core.RedisConnectionUtils.getConnection(RedisConnectionUtils.java:78)
    at org.springframework.data.redis.core.RedisTemplate.execute(RedisTemplate.java:177)
    at org.springframework.data.redis.core.RedisTemplate.execute(RedisTemplate.java:152)
    at org.springframework.data.redis.core.AbstractOperations.execute(AbstractOperations.java:85)
    at org.springframework.data.redis.core.DefaultHashOperations.get(DefaultHashOperations.java:48)


```

找到源头第一次出现500地方，

发现以下代码

```
.......
Cursor c = stringRedisTemplate.getConnectionFactory().getConnection().scan(options);
while (c.hasNext()) {
.....,,
   }


```

分析这个代码，stringRedisTemplate.getConnectionFactory().getConnection()获取pool中的redisConnection后，并没有后续操作，也就是说此时redis 连接池中的链接被租赁后并没有释放或者退还到链接池中，虽然业务已处理完毕 redisConnection 已经空闲，但是pool中的redisConnection的状态还没有回到idle状态

![图片](https://mmbiz.qpic.cn/mmbiz/6mychickmupWLhh8Mf5kpgQac9FOJMTfldsgyuPzbttDOjg4qUsj2b8NqCY0qXs2O0yLMPYzVlufbfhNqs0a1ibA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图片

正常应为

![图片](https://mmbiz.qpic.cn/mmbiz/6mychickmupWLhh8Mf5kpgQac9FOJMTfls7wtd65Od8cLssMNiccPtmUOM7LUuQko5d12AY7jwOBAPpbufRGRoPg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图片

自此问题已经找到。

小小总结
----

总结：spring stringRedisTemplate 对redis常规操作做了一些封装，但还不支持像 Scan SetNx等命令，这时需要拿到jedis Connection进行一些特殊的Commands

使用

```
stringRedisTemplate.getConnectionFactory().getConnection()


```

是不被推荐的

我们可以使用

```
stringRedisTemplate.execute(new RedisCallback<Cursor>() {

     @Override
     public Cursor doInRedis(RedisConnection connection) throws DataAccessException {

       return connection.scan(options);
     }
   });


```

来执行，

或者使用完connection后 ，用

```
RedisConnectionUtils.releaseConnection(conn, factory);


```

来释放connection.

同时，redis中也不建议使用keys命令，redis pool的配置应该合理配上，否则出现问题无错误日志，无报错，定位相当困难。