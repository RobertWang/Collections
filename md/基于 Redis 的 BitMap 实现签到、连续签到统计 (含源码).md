> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/GnT43XoSV1wsJay3gbLfGA)

**一、简介**

基于 Redis 的 BitMap 相关命令，实现用户签到、连续签到统计等功能。

**1.1、背景**

**1.2、BitMap**  

        bitmap 不是一个独立的数据类型，而是一种特殊的 string 类型，它可以将一个 string 类型的值看作是一个由二进制位组成的数组，并提供了一系列操作二进制位的命令。一个 bitmap 类型的键最多可以存储 2^32 - 1 个二进制位。

         bitmap 类型的底层实现是 SDS（simple dynamic string），它和 string 类型相同，只是在操作时会将每个字节拆分成 8 个二进制位。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm46gBmBeicibKJAO3m8BxD1ZYL7rrJDIadicHeMhmgqg878L0WukiagtKlC9MNoKYvCibuyTbAicQr9ClQmQ/640?wx_fmt=png)

**常见用法：**

       Redis 中的 Bitmap 是一种数据结构，用于存储和操作位数组（bit array）。它可以有效地表示指定范围内的位状态，每个位的值可以是 0 或 1。使用 Bitmap 可以进行高效的位级别操作，例如对某个位进行设置、获取、翻转等操作，以及位的逻辑运算，如 AND、OR、XOR 等。

**在 Redis 中，Bitmap 的应用场景：**

**1、统计在线用户 (签到)**：可以使用一个 Bitmap，每个位代表一个用户 ID，如果某个用户在线，则将对应位设置为 1，否则设置为 0。可以通过位操作来统计在线用户的数量。

2、**频率统计**：可以使用一个 Bitmap，每个位代表一个事件，如果事件发生，则将对应位设置为 1。可以通过位操作来统计某段时间内事件发生的频率。

**3、实现布隆过滤器**，利用 setbit 和 getbit 命令实现快速判断一个元素是否存在于一个集合中。

**4、实现位图索引**，利用 bitop 和 bitpos 命令实现对多个条件进行位运算和定位。

**5、统计用户活跃度**，利用 setbit 和 bitcount 命令实现每天或每月用户登录次数的统计

**常用的命令：**  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm46gBmBeicibKJAO3m8BxD1ZYLOxUcwwNaZxLhsoWiaKmRDArWR2xbJrGOicPA6BJ9Yj3U7wlH0ib9qdNBw/640?wx_fmt=png)

        **需要注意的是**，由于 Bitmap 是以字节为单位存储的，因此对于较大的位图，可能会占用较多的内存。在使用 Bitmap 时，需要根据实际情况评估内存消耗。

**1.3、BITFIELD 使用说明**  

      Redis 中的 BITFIELD 命令是用于对位域（bit field）进行操作的，位域是由多个位组成的数据结构。它允许你对位域进行读取、设置和计算等操作。下面是 BITFIELD 命令的用法示例：

```
bitfield bitfield_test get u4 0    #从第一个位开始取4个位(0110)，结果为无符号数(u)
结果：6
bitfield bitfield_testget u3 2    #从第三个位开始取3个位(101)，结果为无符号数(u)
结果：5
bitfield bitfield_testget get i4 0   #从第一个位开始取4个位(0110)，结果为有符号数(i)
结果：6
因为结果为有符号数所以，第一位符号位为0代表是正数。110为6，结果直接为6
bitfield bitfield_testget get i3 2   #从第三个位开始取3个位(101)，结果为有符号数(i)
结果：-3
取到的结果首位为1代表是负数，01需要取补码运算。01取反为10,10+1为11。11十进制为3，因为符号位为1所以最终结果为-3

```

**命令操作案例：**  

```
redis6.3:0>setbit qd_key 0 1
"0"
redis6.3:0>setbit qd_key 1 1
"0"
redis6.3:0>setbit qd_key 2 1
"0"
redis6.3:0>getbit qd_key 10
"1"
redis6.3:0>bitcount qd_key
"6"
redis6.3:0>bitfield qd_key get u2 0
1) "3"

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm44kVUrNBBYJSngJ38aBMueUZVohXRFGeNUMOeUKjrPOsr94McR1iafUoUGNLw9mdg7LAoiavTlPxhpQ/640?wx_fmt=png)

**二、签到功能实现**

**2.1、需求分析**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm46gBmBeicibKJAO3m8BxD1ZYLibqjs4aZmAtqVT5YcgdEYlzia76AL0xdMOv1pPA9iaCCSib339qqzCTcKg/640?wx_fmt=png)

**2.2、代码实现**  

```
  public Result sign() {
        // 1.获取当前登录用户
//        Long userId = UserHolder.getUser().getId();
        Long userId =999L;
        // 2.获取日期 使用hutool的日期时间工具-DateUtil
        Date date = DateUtil.date();
        // 3.拼接key
        String keySuffix = DateUtil.format(date, ":yyyyMM");
        String key = USER_SIGN_KEY + userId + keySuffix;
        // 4.获取今天是本月的第几天
        int dayOfMonth =  DateUtil.dayOfMonth(date);
        // 5.写入Redis SETBIT key offset 1
        stringRedisTemplate.opsForValue().setBit(key, dayOfMonth - 1, true);
        return Result.ok();
    }

```

**结果展示：**  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm46gBmBeicibKJAO3m8BxD1ZYL9sWnDMuNVu092N6pD94OxdSR4oQJazlW4ibv6eZwqjIqN2gVdJTqpzw/640?wx_fmt=png)

**三、连续签到统计功能实现**

****3.1、需求分析****

**问题 1：什么叫做连续签到天数？**

      从最后一次 (当前时间) 签到开始向前统计，直到遇到第一次未签到为止，计算总的签到次数，就是连续签到天数。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm44kVUrNBBYJSngJ38aBMueUF2iciaMibq0wZlEOotuj7P6ceRSs4oZgfEBERSG1Gx4VBEqHZrLDBhqAQ/640?wx_fmt=png)

**问题 2：如何得到本月到今天为止的所有签到数据？**

```
  BITFIELD key GET u[dayOfMonth]0

```

**问题 3：如何从后向前遍历每个 bit 位？**

      与 1 做与运算，就能得到最后一个 bt 位。随后右移一位，下一个 Bit 位就成了最后一个 Bit 位。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm44kVUrNBBYJSngJ38aBMueUcBds2iaMw5h1BHL2OuEcCAeCnicODialUVoGttjHRMeVse2tKkMNbGHyw/640?wx_fmt=png)

****3.2、代码实现****  

```
public Result signCount() {
        //        Long userId = UserHolder.getUser().getId();
        Long userId =999L;  //暂时写死
        // 2.获取日期 使用hutool的日期时间工具-DateUtil
        Date date = DateUtil.date();
        // 3.拼接key
        String keySuffix = DateUtil.format(date, ":yyyyMM");
        String key = USER_SIGN_KEY + userId + keySuffix;
        // 4.获取今天是本月的第几天
        int dayOfMonth =  DateUtil.dayOfMonth(date);
        // 5.获取本月截止今天为止的所有的签到记录，返回的是一个十进制的数字 BITFIELD sign:999:202308 GET u18 0
        List<Long> result = stringRedisTemplate.opsForValue().bitField(
                key,
                BitFieldSubCommands.create()
                        .get(BitFieldSubCommands.BitFieldType.unsigned(dayOfMonth)).valueAt(0)
        );
        if (result == null || result.isEmpty()) {
            // 没有任何签到结果
            return Result.ok(0);
        }
        //num为0，直接返回0
        Long num = result.get(0);
        if (num == null || num == 0) {
            return Result.ok(0);
        }
        // 6.循环遍历
        int count = 0;
        while (true) {
            // 6.1.让这个数字与1做与运算，得到数字的最后一个bit位  // 判断这个bit位是否为0
            if ((num & 1) == 0) {
                // 如果为0，说明未签到，结束
                break;
            }else {
                // 如果不为0，说明已签到，计数器+1
                count++;
            }
            // 把数字右移一位，抛弃最后一个bit位，继续下一个bit位
            num >>>= 1;
        }
        return Result.ok(count);
    }

```

****3.3、结果展示：****  

****当天还没有签到统计：****

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm44kVUrNBBYJSngJ38aBMueUuuaia2lS3IicjbMpVxbZUUBEV5mEwY16DlhUXddp5UkiaWFTNDHO2MB0w/640?wx_fmt=png)

**当天已经签到统计：**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm44kVUrNBBYJSngJ38aBMueUqfs7JOYy3jFtJ3jb6IPIcpjhBTpWcJVmbWjmRiaYKYL2dU4HRrXWtXg/640?wx_fmt=png)