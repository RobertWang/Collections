> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5OTA1MDUyMA==&mid=2655471564&idx=4&sn=0160c34b19e5d41bd782d808128043a9&chksm=bd728bbb8a0502ada5a489bf8dbfe10cc2f21882ddc3db662e0fab12fea6279f37fbc4c668aa&mpshare=1&scene=1&srcid=0214MKr5zNnoMJkg90lYHYJ2&sharer_sharetime=1644811703120&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

【导读】本文简单介绍了跳跃表并给出一个 Go 的实现。  

### 简介

跳跃表是一种可以用来替代平衡树的数据结构，Redis 中的有序集合（zset）就是用 skiplist 来做排序的，在 leveldb 中也能看到 skiplist 的身影。本文介绍跳跃表的基本原理，然后用 Go 完成一个简单的实现。

下面的图片展示了一个典型的跳跃表：

![](https://mmbiz.qpic.cn/mmbiz_jpg/IgylNib7ZE2IQx3WTkRknrZvskfrnwGaUrDLIJMyuR5Hic6xvMtlVBIFgXQ1vnkiaXc1dkGsuKZWFh3y9wm37NBsA/640?wx_fmt=jpeg)

跳跃表通过每个节点多出来的随机个数的前进指针达到加快数据检索的目的。与平衡树严格是时间复杂度不同，跳跃表的这种加速是基于概率的，虽然最坏的情况要比平衡树略差，但总体的平均复杂度也能达到 O(logn)。下面是跳跃表在各个操作上与不同实现的平衡树的比较：

![](https://mmbiz.qpic.cn/mmbiz_jpg/IgylNib7ZE2IQx3WTkRknrZvskfrnwGaUx4F1ic6SKEygQKfM6dMBicBMtv1sGGxHyLbrVzFbdBKDC3souoeskcJA/640?wx_fmt=jpeg)

虽然检索效率比平衡树略差，但跳跃表具有以下优点：

*   插入、删除要比平衡树块
    
*   实现简单
    
*   更节省空间
    
*   更容易拓展
    

以自平衡树红黑树为例，除了需要俩个子节点指针外还需要记录颜色，而跳跃表可以通过调整概率因子 p 使额外空间的使用小于红黑树。Redis 中的跳跃表实现就是做了拓展以支持双向检索等目的，类似的需求在红黑树中就不太好实现了。

下面使用 Go 完成一个简单的实现。

接口设计
----

下面是我这个跳跃表要有的特性和接口：

1.  键类型定义为 `float64`，值支持任意类型
    
2.  检索（Search）：存在返回包含键值的元素和 true，不存在返回 nil 和 false
    
3.  插入（Insert)
    
4.  删除（Delete）
    
5.  获取长度（Len）
    
6.  迭代（Front，Next)
    

将这些写成 ExampleTest：

```
package skiplist_test

import (
    "fmt"

    "github.com/protream/go-datastructures/skiplist"
)

func ExampleSkipList() {
    sl := skiplist.New()

    sl.Insert(float64(100), "foo")

    e, ok := sl.Search(float64(100))
    fmt.Println(ok)
    fmt.Println(e.Value)
    e, ok = sl.Search(float64(200))
    fmt.Println(ok)
    fmt.Println(e)

    sl.Insert(float64(20.5), "bar")
    sl.Insert(float64(50), "spam")
    sl.Insert(float64(20), 42)

    fmt.Println(sl.Len())
    e = sl.Delete(float64(50))
    fmt.Println(e.Value)
    fmt.Println(sl.Len())

    for e := sl.Front(); e != nil; e = e.Next() {
        fmt.Println(e.Value)
    }
    // Output:
    // true
    // foo
    // false
    // <nil>
    // 4
    // spam
    // 3
    // 42
    // bar
    // foo
}
```

定义
--

首先定义俩个常量，maxLevel 表示元素最多可以有多少个向前的指针，它决定了跳跃表的容量。p 是一个概率因子，它决定了随机生成 level 的大小分布。

```
const (
    maxLevel int     = 16   // Should be enough for 2^16 elements
    p        float32 = 0.25
)
```

p 的取值对检索时间和节点向前指针个数影响如图：

![](https://mmbiz.qpic.cn/mmbiz_jpg/IgylNib7ZE2IQx3WTkRknrZvskfrnwGaU4Zv9gRCTutsgiavyiazWAVlC3nU8CAiaHe74TdGOAZPJa1vaeLNAwVhicA/640?wx_fmt=jpeg)

现在定义一下跳跃表的节点。Score 存放可排序键，Value 存放键对应的值，forward 存放向前的指针列表，forward 的长度是插入元素随机生成的长度。

```
// Element is an Element of a skiplist.
type Element struct {
    Score   float64
    Value   interface{}
    forward []*Element
}

func newElement(score float64, value interface{}, level int) *Element {
    return &Element{
        Score:   score,
        Value:   value,
        forward: make([]*Element, level),
    }
}
```

再来定义跳跃表。header 是一个哑节点，作用和单链表的哑节点一样，方便操作，len 记录当前元素个数，level 记录当前所有元素最大 level。

```
// SkipList represents a skiplist.
// The zero value from SkipList is an empty skiplist ready to use.
type SkipList struct {
    header *Element // header is a dummy element
    len    int      // current skiplist length，header not included
    level  int      // current skiplist level，header not included
}

// New returns a new empty SkipList.
func New() *SkipList {
    return &SkipList{
        header: &Element{forward: make([]*Element, maxLevel)},
    }
}
```

需要一个生成随机 level 的函数：

```
func randomLevel() int {
    level := 1
    for rand.Float32() < p && level < maxLevel {
        level++
    }
    return level
}
```

### 迭代

要支持这样的迭代访问需要实现 Front 和 Next 方法。

```
for e := sl.Front(); e != nil; e = e.Next() {
    ...
}
```

如过只看 forward 的 0 层，相当于是在迭代一个单链表，很好写：

```
// Front returns first element in the skiplist which maybe nil.
func (sl *SkipList) Front() *Element {
    return sl.header.forward[0]
}

// Next returns first element after e.
func (e *Element) Next() *Element {
    if e != nil {
        return e.forward[0]
    }
    return nil
}
```

检索
--

检索是从跳跃表当前最大 level 由高到低进行的。以检索 17 的过程为例：

![](https://mmbiz.qpic.cn/mmbiz_jpg/IgylNib7ZE2IQx3WTkRknrZvskfrnwGaUnLqdRwvyf4QqUicUN1jakLmumC1YQrU71VSvia014WkLMjWlLgfMiah9A/640?wx_fmt=jpeg)

1.  从当前最高层也就是第 4 层开始，找到这一层最后一个小于 17 的元素，这里是 6
    
2.  来到 6 所在元素，层数减 1，找到这一层最后一个小于 17 的元素，这里是还是 6
    
3.  层数减 1，重复上面的过程，找到第 0 层最后一个小于 17 的元素
    

到这里显而易见，如果第 0 层最后一个小于 17 的元素的下一个元素就是我们要找的元素，当然，如果这个元素为 nil 或者 score 和要找的不相等，就说明要找的 score 不存在。

```
// Search the skiplist to findout element with the given score.
// Returns (*Element, true) if the given score present, otherwise returns (nil, false).
func (sl *SkipList) Search(score float64) (element *Element, ok bool) {
    x := sl.header
    for i := sl.level - 1; i >= 0; i-- {
        for x.forward[i] != nil && x.forward[i].Score < score {
            x = x.forward[i]
        }
    }
    x = x.forward[0]
    if x != nil && x.Score == score {
        return x, true
    }
    return nil, false
}
```

插入
--

下面是插入 17 到跳跃表：

![](https://mmbiz.qpic.cn/mmbiz_jpg/IgylNib7ZE2IQx3WTkRknrZvskfrnwGaU29T5hNibIfu9OrshkZHtXFRIRdjSDOacdynRr2yPzzy7CruthdiccaUw/640?wx_fmt=jpeg)

插入首先要定位插入的点，这个过程和检索基本一样，不同的是，要记录每个 level 最后一个小于 17 元素，因为插入点必然在这些元素后，它们 forward 需要更新，这里记录的地方就是 update，后面更新 update 里和待插入元素的指针就行了。这里的实现是不允许重复 score 的，如果 score 已经存在则替换为新的 value 值。

```
// Insert (score, value) pair to the skiplist and returns pointer of element.
func (sl *SkipList) Insert(score float64, value interface{}) *Element {
    update := make([]*Element, maxLevel)
    x := sl.header
    for i := sl.level - 1; i >= 0; i-- {
        for x.forward[i] != nil && x.forward[i].Score < score {
            x = x.forward[i]
        }
        update[i] = x
    }
    x = x.forward[0]

    // Score already presents, replace with new value then return
    if x != nil && x.Score == score {
        x.Value = value
        return x
    }

    level := randomLevel()
    if level > sl.level {
        level = sl.level + 1
        update[sl.level] = sl.header
        sl.level = level
    }
    e := newElement(score, value, level)
    for i := 0; i < level; i++ {
        e.forward[i] = update[i].forward[i]
        update[i].forward[i] = e
    }
    sl.len++
    return e
}
```

删除
--

删除和插入非常相似并且还要简单一点。

```
// Delete remove and return element with given score, return nil if element not present
func (sl *SkipList) Delete(score float64) *Element {
    update := make([]*Element, maxLevel)
    x := sl.header
    for i := sl.level - 1; i >= 0; i-- {
        for x.forward[i] != nil && x.forward[i].Score < score {
            x = x.forward[i]
        }
        update[i] = x
    }
    x = x.forward[0]

    if x != nil && x.Score == score {
        for i := 0; i < sl.level; i++ {
            if update[i].forward[i] != x {
                return nil
            }
            update[i].forward[i] = x.forward[i]
        }
        sl.len--
    }
    return x
}
```

总结
--

本文简单介绍了跳跃表并给出一个 Go 的实现。这个实现比较简单，也没有考虑并发安全，需要的话可以通过 `sync.RWLock` 来做。更详细的介绍建议直接看 skiplist 的原论文：Skip Lists：A Probabilistic Alternative to Balanced Trees(https://protream.com/2019/skiplist-and-its-go-implementation/)。

> 来源：zhuanlan.zhihu.com/p/72553601

- EOF -