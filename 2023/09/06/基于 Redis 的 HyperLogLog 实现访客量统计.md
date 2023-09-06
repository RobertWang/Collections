> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/6TqugCYcn-J2qew2M2qwVw)

> 基于 Redis 的 HyperLogLog 实现访客量统计

**一、简介**

  我们先思考一个常见的业务问题：如果你负责开发维护一个大型的网站，有一天老板找产品经理要某个网站每个网页每天的 UV（访客量） 数据，然后让你来开发这个统计模块，你会如何实现？

  如果统计 PV(浏览量) 那非常好办，给每个网页一个独立的 Redis 计数器就可以了，这个计数器的 key 后缀加上当天的日期。这样来一个请求，incrby 一次，最终就可以统计出所有的 PV 数据。

      但是 UV 不一样，它要去重，同一个用户一天之内的多次访问请求只能计数一次。这就要求每一个网页请求都需要带上用户的 ID，无论是登录用户还是未登录用户都需要一个唯一 ID 来标识。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm450er8SlickycQHeqEEHEPpBK92Vjt24Za2gp3AA2YI1fNLTLHaR9UjSv2GNsTa1gIdQZDYIC1ZHqw/640?wx_fmt=png)**解决方案**：Redis 提供了 HyperLogLog 数据结构就是用来解决这种统计问题的。HyperLogLog 提供不精确的去重计数方案，虽然不精确但是也不是非常不精确，标准误差是 0.81%，这样的精确度已经可以满足上面的 UV 统计需求了。

      HyperLogLog 数据结构是 Redis 的高级数据结构，它非常有用，但是令人感到意外的是，使用过它的人非常少。

**二、HyperLogLog 用法**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm450er8SlickycQHeqEEHEPpBsrIxWZ96mZibmgiadJSJ0g7wibjSicvjLD7Xj4sTicbjUejFfErKpEHO9Pg/640?wx_fmt=png)

**具体代码：**

```
redis6.3:0>pfadd pf1 p1 p2 p3 p4
"1"
redis6.3:0>pfcount pf1
"4"
redis6.3:0>pfadd pf1 p3 p5 p6
"1"
redis6.3:0>pfcount pf1

```

**pfmerge 的用法：**

      HyperLogLog 除了上面的 pfadd 和 pfcount 之外，还提供了第三个指令 pfmerge，用于将多个 pf 计数值累加在一起形成一个新的 pf 值。比如：在网站中我们有两个内容差不多的页面，运营说需要这两个页面的数据进行合并。其中页面的 UV 访问量也需要合并，那这个时候 pfmerge 就可以派上用场了。

```
127.0.0.1:6379> pfmerge user2 user  //将user中的数据合并到user2中
OK
127.0.0.1:6379> pfcount user
(integer) 2

```

**三、实现案例**

**核心代码：**

```
 public Result testHyperLogLog(){
        String[] values = new String[1000];
        int j = 0;
        for (int i = 0; i < 1000000; i++) {
            j = i % 1000;
            values[j] = "user_" + i;
            if(j == 999){
                // 发送到Redis
                stringRedisTemplate.opsForHyperLogLog().add("hll", values);
            }
        }
        // 统计数量
        Long count = stringRedisTemplate.opsForHyperLogLog().size("hll");
        log.info("统计的用户数量count："+count);
        return Result.ok();
    }

```

**结果展示：**  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm450er8SlickycQHeqEEHEPpBx25icSuFSKLtU8iaSGSq94tvNefpcqrJ4mcawiaZfB3ZMTkOnnWGNqhiaQ/640?wx_fmt=png)

      100 万条数据，统计出来有 997461，误差是 0.253%, 小于标准误差是 0.81%，对于 UV 统计需求来说，误差率也不算高，然后我们把上面的脚本再跑一遍，也就相当于将数据重复加入一遍，查看输出，可以发现，pfcount 的结果没有任何改变，还是 997461，说明它确实具备去重功能。

**四、注意事项**

        HyperLogLog 这个数据结构不是免费的，不是说使用这个数据结构要花钱，它需要占据一定 12k 的存储空间，所以它不适合统计单个用户相关的数据。如果你的用户上亿，可以算算，这个空间成本是非常惊人的。但是相比 set 存储方案，HyperLogLog 所使用的空间那真是可以使用千斤对比四两来形容了。不过你也不必过于担心，因为 Redis 对 HyperLogLog 的存储进行了优化，在计数比较小时，它的存储空间采用稀疏矩阵存储，空间占用很小，仅仅在计数慢慢变大，稀疏矩阵占用空间渐渐超过了阈值时才会一次性转变成稠密矩阵，才会占用 12k 的空间。