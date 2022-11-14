> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/jZzNl1wAzFf9VrXOwOH4tQ)

（一）uuid
-------

uuid 有两种包：

*   github.com/google/uuid ，仅支持 V1 和 V4 版本。
    

*   github.com/gofrs/uuid ，支持全部五个版本。
    

下面简单说下五种版本的区别：

*   Version 1，基于 mac 地址、时间戳。
    
*   Version 2，based on timestamp，MAC address and POSIX UID/GID (DCE 1.1)
    

*   Version 3，Hash 获取入参并对结果进行 MD5。
    

*   Version 4，纯随机数。
    

*   Version 5，based on SHA-1 hashing of a named value。
    

### 特点

*   5 个版本可供选择。
    

*   定长 36 字节，偏长。
    

*   无序。
    

（二）shortuuid
------------

初始值基于 uuid Version4；第二步根据 alphabet 变量长度 (定长 57) 计算 id 长度(定长 22)；第三步依次用 DivMod（欧几里得除法和模）返回值与 alphabet 做映射，合并生成 id。

### 特点

*   基于 uuid，但比 uuid 的长度短，定长 22 字节。
    

```
package mian
import (
    "github.com/lithammer/shortuuid/v4"
    "fmt"
)
func main() {
    id := shortuuid.New()
    // id: iDeUtXY5JymyMSGXqsqLYX length: 22
    fmt.Println("id:", id, "length:", len(id))
    // V22s2vag9bQEZCWcyv5SzL 固定不变
    id = shortuuid.NewWithNamespace("http://127.0.0.1.com")
    // id: K7pnGHAp7WLKUSducPeCXq length: 22
    fmt.Println("id:", id, "length:", len(id))
    // NewWithAlphabet函数可以用于自定义的基础字符串，字符串要求不重复、定长57
    str := "12345#$%^&*67890qwerty/;'~!@uiopasdfghjklzxcvbnm,.()_+·><"
    id = shortuuid.NewWithAlphabet(str)
    // id: q7!o_+y('@;_&dyhk_in9/ length: 22
    fmt.Println("id:", id, "length:", len(id))
}

```

（三）xid
------

xid 是由时间戳、进程 id、Mac 地址、随机数组成。有序性来源于对随机数部分的原子 + 1。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95ZQ5ZpMKKJRxlM6entbgW4oiaK5LQM4WzLoRiaj1RTiaKYAG0UulvSYWOyKiazBxIelUQA8sicFzL83Bw/640?wx_fmt=png)

### 特点

*   长度短。
    

*   有序。
    

*   不重复。
    

*   时间戳这个随机数原子 + 1 操作，避免了时钟回拨的问题。
    

下面的代码根据需求进行了魔改。

```
package mian
import (
    "github.com/rs/xid"
    "fmt"
)
func main() {
    // hostname+pid+atomic.AddUint32
    id := xid.New()
    containerName := "test"
    // 由于xid默认使用可重复ip地址填充4 5 6位。
    // 实际场景中，服务都是部署在docker中，这里把ip地址位替换成了容器名
    // 这里只取了容器名MD5的前3位，验证会重复，放弃使用
    containerNameID := make([]byte, 3)
    hw := md5.New()
    hw.Write([]byte(containerName))
    copy(containerNameID, hw.Sum(nil))
    id[4] = containerNameID[0]
    id[5] = containerNameID[1]
    id[6] = containerNameID[2]
    // id: cbgjhf89htlrr1955d5g length: 12
    fmt.Println("id:", id, "length:", len(id))
}

```

（四）ksuid
--------

由随机数和时间戳组成。时间戳占前 4 字节，后面均为随机数：

```
package mian
import (
    "github.com/segmentio/ksuid"
    "fmt"
)
func main() {
    id := ksuid.New()
    // id: 2CWvPg766SUvezbiiV9nzrTZsgf length: 20
    fmt.Println("id:", id, "length:", len(id))
    id1 := ksuid.New()
    id2 := ksuid.New()
    // id1:2CTqTLRxCh48y7oUQzQHrgONT2k id2:2CTqTHf07C09CXyRMHdGKXnY5HP
    fmt.Println(id1, id2)
    // 支持ID对比，这个功能比较鸡肋了,目前没想到有用的地方
    compareResult := ksuid.Compare(id1, id2)
    fmt.Println(compareResult) // 1
    // 判断顺序性
    isSorted := ksuid.IsSorted([]ksuid.KSUID{id2, id1})
    fmt.Println(isSorted) // true 降序
}

```

（五）ulid
-------

随机数和时间戳组成

```
package mian
import (
    "github.com/oklog/ulid"
    "fmt"
)
func main() {
    t := time.Now().UTC()
    entropy := rand.New(rand.NewSource(t.UnixNano()))
    id := ulid.MustNew(ulid.Timestamp(t), entropy)
    // id: 01G902ZSM96WV5D5DC5WFHF8WY length: 26
    fmt.Println("id:", id.String(), "length:", len(id.String()))
}

```

（六）snowflake
------------

大名鼎鼎的雪花算法，这里不做过多介绍了。相对于 UUID 来说，雪花算法不会暴露 MAC 地址更安全、生成的 ID 也不会过于冗余。雪花的一部分 ID 序列是基于时间戳的，那么时钟回拨的问题就来了。上面提到的 xid，一定程度上避时钟回拨的影响。那么什么是时钟回拨，后面会提到。

```
package main
import(
    "fmt"
    "github.com/bwmarrin/snowflake"
)
func main() {
  node, err := snowflake.NewNode(1)
    if err != nil {
      fmt.Println(err)
      return
    }
    id := node.Generate().String()    
    // id: 1552614118060462080 length: 19
    fmt.Println("id:", id, "length:", len(id))
}

```

**二、数据库自增 ID**

这里常规是指数据库主键自增索引。特点如下：

*   架构简单容易实现。
    

*   ID 有序递增，IO 写入连续性好。
    

*   INT 和 BIGINT 类型占用空间较小。
    

*   由于有序递增，易暴露业务量。
    

*   受到数据库性能限制，对高并发场景不友好。
    

*   bigint 最大是 2^64-1，但是数据库单表肯定放不了这么多，那么就涉及到分表。如果业务量真的太大了，主键的自增 id 涨到头了，会发生什么？报错：主键冲突。
    

**三、Redis 生成 ID**

通过 redis 的原子操作 INCR 和 INCRBY 获得 id。相比数据库自增 ID，redis 性能更好、更加灵活。不过架构强依赖 redis，redis 在整个架构中会产生单点问题。在流量较大的场景下，网络耗时也可能成为瓶颈。

**四、ZooKeeper 唯一 ID**

ZooKeeper 是使用了 Znode 结构中的 Zxid 实现顺序增 ID。Zookeeper 类似一个文件系统，每个节点都有唯一路径名 (Znode)，Zxid 是个全局事务计数器，每个节点发生变化都会记录响应的版本 (Zxid)，这个版本号是全局唯一且顺序递增的。这种架构还是出现了 ZooKeeper 的单点问题。

**五、号段模式**

（一）Leaf-segment
---------------

把数据库自增主键换成了计数法。每个业务分配一个 biz_tag、并记录各业务最大 id(max_id)、号段跨度 (step) 等数据。这样每次取号只需要更新 biz_tag 对应的 max_id，就可以拿到 step 个 id。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95ZQ5ZpMKKJRxlM6entbgW4kXWWTrvP6eZAMGG5H2ib4GziabbKqBDnAzjbosr46IvSzbdBxekfgjhA/640?wx_fmt=png)

### 优点

*   除了拥有自增 ID 的优点之外，在性能上比自增 ID 更好
    

*   扩展灵活。
    

*   使用灵活、可配置性强。
    

*   缓存机制，突发状况下短时间内能保证服务正常运转。
    

### 缺点

*   id 是有序自增，容易暴露信息，不可用于订单。
    

*   在 leaf 的缓存 ID 用完再去获取新号段的间隙，性能会有波动。
    

*   强依赖 DB。
    

（二）增强版 Leaf-segment
-------------------

增强版是对上面描述的缺点 2 进行的改进——双 cache。在 leaf 的 ID 消耗到一定百分比时，常驻的后台进程会预先去号段服务获取新的号段并缓存。具体消耗百分比、及号段 step 根据业务消耗速度来定。 

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95ZQ5ZpMKKJRxlM6entbgW4443RmJUuVnvHic8e7B1SGDiamRvxJLOwcic5nAfbWuNxxdyZYichNVhUicA/640?wx_fmt=png)

（三）Tinyid
---------

和增强版 Leaf-segment 类似，也是号段模式，提前加载号段。 

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95ZQ5ZpMKKJRxlM6entbgW4wx9TX96bJql3MXBzL4u8cvwEFtibljGGCPTKOd04pjZ8ckPcGKlI0YA/640?wx_fmt=png)

（四）Leaf-snowflake
=================

### 时钟回拨

服务器上的时间突然倒退回之前的时间。可能是人为的调整时间；也可能是服务器之间的时间校对。

实现方案
----

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95ZQ5ZpMKKJRxlM6entbgW4yP9LhibJzlSkWnib3HaduxhIwl5gviboA7bZEWeRk3me6bS5KZbe8o7Iw/640?wx_fmt=png)

用 Zookeeper 顺序增、全局唯一的节点版本号，替换了原有的机器地址。解决了时钟回拨的问题。前面介绍 ZooKeeper 的缺点，强依赖 ZooKeeper、大流量下的网络瓶颈。下图的方案在 Leaf-snowflake 中通过缓存一个 ZooKeeper 文件夹，提高可用性。运行时运行时，时差小于 5ms 会等待时差两倍时间，如果时差大于 5ms 报警并停止启动。 

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95ZQ5ZpMKKJRxlM6entbgW44NcxbE2XQAgo81uyx7RVfDQKyPibVEIwqkrMLsvBzjyKibTtj0ymicqug/640?wx_fmt=png)

 **作者简介**

**陈冬**

腾讯后台开发工程师

腾讯后台开发工程师，目前负责腾讯视频后端中间件开发工作，在消息队列和 go 性能优化方面有丰富经验。