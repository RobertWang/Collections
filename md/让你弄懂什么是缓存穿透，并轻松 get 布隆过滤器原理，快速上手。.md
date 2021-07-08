> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6899381130231808014)

> 一图胜千言，如下图，当访问到数据库中没有的数据的时候，每次访问同样的数据，都会绕过缓存系统（一般是 Redis），直接访问到数据库层，这就是我们说的缓存穿透。

一、什么是缓存穿透？
----------

一图胜千言，如下图，当访问到数据库中没有的数据的时候，每次访问同样的数据，都会绕过缓存系统（一般是 Redis），直接访问到数据库层，这就是我们说的缓存穿透。

解决办法：

我们可以在缓存中将这样的数据也缓存起来，只是缓存的 key 的 value 为空。

但是如果有特别多的 key 是这种情况的，我们的缓存中要缓存好多这样的 key，比较占用缓存。

另外如果这种 key 访问的次数比较少，我们缓存这种 key 的效果也不是很好。

于是引出了一种办法：

我们有什么办法可以让这样的数据在访问缓存之前，就可以被判断出来是否在我们的系统中呢？

对，这就是布隆过滤器要做的事情了。他可以判断一个 key 是否存在。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/730fff58b2024ddb80b1c3692ef0b9e7~tplv-k3u1fbpfcp-watermark.image)

二、布隆过滤器原理
---------

布隆过滤器是位图的数据结构，与一系列哈希算法的结合。

我们有四个 key：key1、key2、key3、key4。

两种哈希算法：hash1、hash2。

我们将每个 key 用多种哈希算法映射到我们下面的位图中。

当我们判断一个 key 是否存在的时候，就去对应的位置上看值是否都为 1。

但是有一个问题，如果 key 对应的位置不都为 1，那么这个 key 一定不存在。

但是 key 对应位置都为 1，这个 key 不一定存在于我们的系统当中，有些晕了吧，看下图的 key3 不一定存在与我们系统中哦，因为他对应的位置上可能是其他的 key 对应的。

也就是说**布隆过滤器是有误差的**。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c12e0f26c12e4aedadc64ecadcef6635~tplv-k3u1fbpfcp-watermark.image)

三、上手布隆过滤器
---------

我知道了布隆过滤器的原理，那么如何来使用呢？

难道这些算法要我们自己来实现吗？当然不用啦。

### 1、guava 的实现

guava 里面已经有实现了，我们直接引入 jar 包后，直接调用就好了。

```
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>19.0</version>
</dependency>
复制代码

```

代码如下：

```
// 预计要插入多少数据
private static int size = 1000000;

// 期望的误判率
private static double fpp = 0.01;

private static BloomFilter<Integer> bloomFilter = BloomFilter.create(Funnels.integerFunnel(), size, fpp);

public static void main(String[] args) {
    // 插入数据
    for (int i = 0; i < 1000000; i++) {
        bloomFilter.put(i);
    }
    int count = 0;
    for (int i = 1000000; i < 2000000; i++) {
        if (bloomFilter.mightContain(i)) {
            count++;
        }
    }
    System.out.println("总共的误判数: " + count);
}
复制代码

```

### 2、Redis 的实现

我们去 [RedisBloom](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fjiangxiuli%2FRedisBloom%2Freleases "https://github.com/jiangxiuli/RedisBloom/releases") 的官网下载源文件。

然后呢，将下载的 jar 包 make 一下。

Redis 会以 module 的方式将 RedisBloom 加载进来使用，类似于扩展的效果。

我们只需要在 Redis 的配置文件 redis.conf 中加入编译完的 RedisBloom 生成的可执行命令的路径，即可，如下：

```
loadmodule /root/RedisBloom-2.2.4/redisbloom.so
复制代码

```

在我们的命令行中，就可以使用如下命令操作布隆过滤器了：

```
-- 向我们的布隆过滤器当中加入要判断的内容
bf.add 
-- 判断内容是否存在于我们的布隆过滤器当中
bf.exists
-- 初始化我们的布隆过滤器，如打算里面放入多少个元素，能容忍的误差率
bf.reserve
复制代码

```

根据我们的预期，布隆过滤器底层会来判断使用多少位的位图，使用几个 hash 算法等，实现我们的目标。

四、总结时间
------

布隆过滤器利用较小的空间，应用于判断大量 key 是否存在的场景。

guava 的实现，利用应用程序所在服务器的内存资源，多个服务器的话，需要自己判断自己的。

Redis 的实现，利用 Redis 的内存资源，应用程序可以调用 Redis 中的布隆过滤器来做相应判断。

本文从缓存穿透的问题，到如何解决缓存穿透的问题，再到布隆过滤器的原理，以及如何应用布隆过滤器，和对比其实现方式，期望能够对大家有所帮助。