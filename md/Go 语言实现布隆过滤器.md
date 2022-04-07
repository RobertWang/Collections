> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/wMQ1vll40xXgwq9vDQp_1Q)

【导读】本文介绍了布隆过滤器原理和 go 语言实现。  

布隆过滤器简介
-------

布隆过滤器（Bloom Filter）是一个基于 hash 的概率性的数据结构，它实际上是一个很长的二进制向量，可以检查一个元素可能存在集合中，和一定不存在集合中。它的优点是空间效率高，但是有一定 false positive(元素不在集合中，但是布隆过滤器显示在集合中)。

布隆过滤器原理
-------

布隆过滤器就是一个长度为`m`个 bit 的 bit 数组，初始的时候每个 bit 都是 0，另外还有`k`个 hash 函数。

![](https://mmbiz.qpic.cn/mmbiz_png/IgylNib7ZE2KhzRll49cicPS3fb13jmmfkG53EJpTXcSuhGcQhCG6ib7rgzJPaTUWBhWBnPnia8ia28yTcsibGYvaxjQ/640?wx_fmt=png)

 

### 布隆过滤器加入元素

当加入一个元素时，先用`k`个 hash 函数得到`k`个 hash 值，将`k`个 hash 值与 bit 数组长度取模得到个`k`个位置，将这`k`个位置对应的 bit 置位 1。

![](https://mmbiz.qpic.cn/mmbiz_jpg/IgylNib7ZE2KhzRll49cicPS3fb13jmmfkP75sgetLjk9ia0GicG1UiamQfy34RoLNAOGFO2PZkBaTg1XhPGayygJJQ/640?wx_fmt=jpeg)

 

在加入了`bloom`之后，再加入`filter`。

![](https://mmbiz.qpic.cn/mmbiz_jpg/IgylNib7ZE2KhzRll49cicPS3fb13jmmfkCiafFlLDGytCkQvu0AulXvJ8EfgeEGL1W8g4yicM6rfDfTP1LprhtTew/640?wx_fmt=jpeg)

 

### 布隆过滤器查询元素

在布隆过滤器中查询元素比较简单，同样地，先用`k`个 hash 函数得到`k`个 hash 值，将`k`个 hash 值与 bit 数组长度取模得到个`k`个位置，然后检查这`k`个位置的 bit 是否是 1。如果都是 1，布隆过滤器返回这个原始存在。

![](https://mmbiz.qpic.cn/mmbiz_jpg/IgylNib7ZE2KhzRll49cicPS3fb13jmmfkCiafFlLDGytCkQvu0AulXvJ8EfgeEGL1W8g4yicM6rfDfTP1LprhtTew/640?wx_fmt=jpeg)

 

### 布隆过滤器的 false positive

查询元素中，有可能`k`个 hash 值对应的位置都已经置一，但这都是其他元素的操作，实际上这个元素并不在布隆过滤器中，这就是 false positive。看下面这个例子，添加完`bloom`,`filter`后，检查`cat`是否在 布隆过滤器中。

![](https://mmbiz.qpic.cn/mmbiz_jpg/IgylNib7ZE2KhzRll49cicPS3fb13jmmfkV58Iian9T5xRB7CZAIDOY9NB3LibN4Kcgyh1dDCNpoLzHdIGuL6fBefg/640?wx_fmt=jpeg)

 

实际上，`cat`并不在布隆过滤器中。所以说，布隆过滤器返回 true，元素不一定在其中；但是返回 false，元素一定不在布隆过滤器中。

### 布隆过滤器的 false positive 计算

false positive 计算，有 3 个重要的参数。1. `m`表示 bit 数组的长度 2. `k`表示散列函数的个数 3. `n`表示插入的元素个数

布隆过滤器中，一个元素插入后，某个 bit 为 0 的概率是

```
(1−1/m)^k


```

n 元素插入后，某个 bit 为 0 的概率是

```
(1−1/m)^(nk)


```

false positive 的概率是

```
(1−(1−1/m)^nk)^k


```

因为需要的是`k`个不同的 bit 被设置成 1，概率是大约是

```
(1−e^(−kn/m))^k


```

这个就是 false positive 的概率

Golang 代码实现
-----------

代码实现在我的 github 仓库。这个 Golang 实现，支持并发操作，批量加入`byte`数组，字符串，数字等。

### bit 数组的大小选择

代码中，bit 数组表示成`[]byte`数组。由于后续在`[]byte`定位 hash 需要取余操作，`%`操作是一个比较慢的操作，如果数组的长度是 2 的 n 次方，`%`可以被优化成`& (2^n-1)`。因此，`New()`函数初始化的时候，会将`[]byte`数组的长度拉长到`2^n`，加快计算。

```
type Filter struct {
    lock       *sync.RWMutex
    concurrent bool

    m     uint64 // bit array of m bits, m will be ceiling to power of 2
    n     uint64 // number of inserted elements
    log2m uint64 // log_2 of m
    k     uint64 // the number of hash function
    keys  []byte // byte array to store hash value
}

func New(size uint64, k uint64, race bool) *Filter {
    log2 := uint64(math.Ceil(math.Log2(float64(size))))
    filter := &Filter{
        m:          1 << log2,
        log2m:      log2,
        k:          k,
        keys:       make([]byte, 1<<log2),
        concurrent: race,
    }
    if filter.concurrent {
        filter.lock = &sync.RWMutex{}
    }
    return filter
}

// location returns the bit position in byte array
// & (f.m - 1) is the quick way for mod operation
func (f *Filter) location(h uint64) (uint64, uint64) {
    slot := (h / bitPerByte) & (f.m - 1)
    mod := h & mod7
    return slot, mod
}


```

### hash 函数的选择

因为需要快速的操作，因此不选择`md5`,`sha`等耗时比较久的 hash 操作。经过比较之后，我选择使用`murmur3`的 hash 算法，来对 key 进行 hash。

```
// baseHash returns the murmur3 128-bit hash
func baseHash(data []byte) []uint64 {
    a1 := []byte{1} // to grab another bit of data
    hasher := murmur3.New128()
    hasher.Write(data) // #nosec
    v1, v2 := hasher.Sum128()
    hasher.Write(a1) // #nosec
    v3, v4 := hasher.Sum128()
    return []uint64{
        v1, v2, v3, v4,
    }
}


```

输入一段元素的字节数组，将其 hash 值返回，计算出这个元素的位置。

> 转自：
> 
> zhuanlan.zhihu.com/p/165130282

- EOF -

推荐阅读  点击标题可跳转

1、[一些著名的软件都用什么语言编写？](http://mp.weixin.qq.com/s?__biz=MzI1MTIzMzI2MA==&mid=2650579301&idx=1&sn=d5deec44dade39ce3f647c53c4185122&chksm=f1fe17e6c6899ef07f44dcfd6056860f48631e2bb39e08c1eb2d2d012af127d6ddab24fe6ba4&scene=21#wechat_redirect)

2、[“阿里味” PUA 编程语言火上 GitHub 热榜](http://mp.weixin.qq.com/s?__biz=MzI1MTIzMzI2MA==&mid=2650579587&idx=1&sn=b1f4edb5f94f966ebe9913b9f2e02404&chksm=f1fe1400c6899d1618954a9fd22ebfeabbb52d9551befe372728bade1601693f4cfd457d9642&scene=21#wechat_redirect)

3、[深入理解 CPU 的调度原理](http://mp.weixin.qq.com/s?__biz=MzI1MTIzMzI2MA==&mid=2650579557&idx=1&sn=0cd2ceec7fe8fda53ace0725ea8ce657&chksm=f1fe14e6c6899df040bdd29291a171febfed66c7bb1409649c3b0e1ea09976a665bda8859dea&scene=21#wechat_redirect)