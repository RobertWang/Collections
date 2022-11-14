> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/NVkJswK4SUaXtTPb2-Od9g)

> 争做团队核心程序员，关注「幽鬼」

争做团队核心程序员，关注「幽鬼」

大家好，我是程序员幽鬼。

Dolt DB[1] 是世界上第一个可以像 git 存储库一样分支和合并、推送和拉取、分叉和克隆的 SQL 数据库。

我们从头开始构建 Dolt 的存储引擎，以加快这些操作。编写行存储引擎和 SQL 执行引擎并不容易。大多数人甚至不尝试它是有原因的，但在 DoltHub，我们的构建方式不同。

今天，我们将回顾几个在对 Dolt 进行基准测试以使行访问与 MySQL 一样快时遇到的性能问题的案例研究。每个案例研究都是我们遇到并解决的实际性能问题。

让我们开始吧。

案例研究：接口断言
---------

在尝试以更快的方式将行从磁盘中读出并通过执行引擎获取时，我们决定制作新的行迭代接口。因为我们已经有了许多原始接口的实现，所以我们认为可以扩展它，如下所示：

```
type RowIter interface {
    Next(ctx *Context) (Row, error)
}

type RowIter2 interface {
    RowIter
    Next2(ctx *Context, frame *RowFrame) error
}

```

我们的 SQL 引擎的架构涉及一个深度嵌套的执行图，图中的每个节点从下一层获取一行，对其进行一些工作，然后将其传递到链上。当我们在迭代器中实现该`Next2`方法时，它看起来像这样：

```
type iter struct {
    childIter  sql.RowIter
}

func (t iter) Next2(ctx *sql.Context, frame *sql.RowFrame) error {
    return t.childIter.(sql.RowIter2).Next2(ctx, frame)
}

```

但是当我们运行这段代码时，我们在分析器图表中注意到，由于这种模式，我们付出了非常可观的性能损失。在我们的 CPU 图表中，我们注意到很多时间都花在了 `runtime.assertI2I`：

![](https://mmbiz.qpic.cn/mmbiz_png/jdTRWcqrowQrCIIOHBeoWuk3TVTZHclcYn3RgODZibTrrTPzLaCE6ovgyicK1csxjZQ2Kdv8agfZ2JSkzVdU28oA/640?wx_fmt=png)

`runtime.assertI2I`是 go 运行时调用的方法，用于验证接口的静态类型是否可以在运行时转换为不同的类型。这涉及到接口表的查找和接口指针转换，这不是那么昂贵，但也肯定不是免费的。由于嵌套的执行图，我们在每行获取的数据中多次执行此操作，从而导致相当严重的性能损失。

### 解决方案：消除接口类型转换

为了消除这种损失，我们只需在必要的地方存储两个字段来消除转换，每个静态类型一个字段。

```
type iter struct {
    childIter  sql.RowIter
    childIter2 sql.RowIter2
}
func (t iter) Next(...) {
    return t.childIter.Next(ctx)
}
func (t iter) Next2(...) {
    return t.childIter2.Next2(ctx, frame)
}


```

这两个字段指向内存中的同一个对象，但由于它们具有不同的静态类型，因此它们需要不同的字段以避免转换损失。

案例研究：切片
-------

接下来，我们将研究分配切片的不同方法及其对性能的影响，尤其是垃圾收集器开销。我们将分析一种生成随机元素切片然后将它们相加的方法。我们将在我们的个人资料中一遍又一遍地调用它。

```
func sumArray() uint64 {
    a := randArray()
    var sum uint64
    for i := range a {
        sum += uint64(a[i].(byte))
    }
    return sum
}


```

我们将研究实现该`randArray()` 功能的 4 种不同方法，从最差到最好，并展示不同实现对程序性能的影响。

### 最差：append 到切片

获得这个切片的更糟糕的方法是创建一个长度为零的切片，然后一遍又一遍地调用 `append`，如下所示：

```
func randArray() []interface{} {
    var a []interface{}
    for i := 0; i < 1000; i++ {
        a = append(a, byte(rand.Int() % 255))
    }
    return a
}


```

不管我们是从上面的 nil 切片开始，还是用 `make([]interface{}, 0)` 制作一个长度为零的切片，效果都是一样的。当我们基于 profile 生成火焰图时，我们可以看到垃圾收集占用了巨大的开销，并且 `runtime.growslice`占用了整整四分之一的 CPU 周期。

![](https://mmbiz.qpic.cn/mmbiz_png/jdTRWcqrowQrCIIOHBeoWuk3TVTZHclc4ZicoLDc2nR9nBeljoZEicVCxkPev21dSEy6tichNNz8R7WphicuQJUhlQ/640?wx_fmt=png)

总的来说，只有不到一半的 CPU 周期直接用于有用的工作。

之所以如此昂贵，是因为 go slices 有一个底层数组，当调用 `append` 时，如果底层数组容量不足，运行时必须分配一个更大的数组，复制所有元素，并回收旧数组。

![](https://mmbiz.qpic.cn/mmbiz_png/jdTRWcqrowQrCIIOHBeoWuk3TVTZHclcEgrX09zmDU1WpwyrD1eOrUmhtWFXuGicKibs3XiaXs6HNUt8l0N0aib9LA/640?wx_fmt=png)

我们可以做得比这更好。

### 更好：静态数组大小

我们可以通过在创建时固定切片的大小来消除 `runtime.growslice` 开销，就像这样。

```
func randArray() []interface{} {
    a := make([]interface{}, 1000)
    for i := 0; i < len(a); i++ {
        a[i] = byte(rand.Int() % 255)
    }
    return a
}


```

当我们 profile 这段代码时，我们可以看到已经完全消除了 `runtime.growslice` 的开销，并减轻了垃圾收集器的压力。

![](https://mmbiz.qpic.cn/mmbiz_png/jdTRWcqrowQrCIIOHBeoWuk3TVTZHclc9gAQzk0Uic82FmY3HxgKCvialKBHvNMNSdsDgicJv06suRTuv4E7jD2Ow/640?wx_fmt=png)

你可以一眼看出这个实现花费了更多的时间做有用的工作。但是每次创建切片仍然会对性能造成重大影响。我们整整 13% 的运行时间都花在分配切片上，即 `runtime.makeslice`.

### 更好的是：原始切片类型

奇怪的是，我们可以通过分配 `byte` 切片而不是切片`interface{}`来做得更好。执行此操作的代码：

```
func randArray() []byte {
    a := make([]byte, 1000)
    for i := 0; i < len(a); i++ {
        a[i] = byte(rand.Int() % 255)
    }
    return a
}


```

当我们查看 profile 时，我们可以看到 `runtime.makeslice`CPU 的影响从超过 13% 下降到大约 3%。你甚至在火焰图上几乎看不到它，而且也很容易看到垃圾收集器开销的相应减少。

![](https://mmbiz.qpic.cn/mmbiz_png/jdTRWcqrowQrCIIOHBeoWuk3TVTZHclce1XnaujhPHy4HYfthPYkeE33pMHIeWy4gNW4b29mK3CPc2T5K2PZbA/640?wx_fmt=png)

造成这种差异的原因只是`interface{}`类型的分配成本更高（一对 8 字节指针，而不是单个字节），而且垃圾收集器推理和处理的成本也更高。这个故事的寓意是，在可能的情况下，分配原始切片类型而不是接口类型通常会在性能方面得到回报。

### 最佳：在循环外分配

但实现此功能的唯一最佳方法是根本不在其中分配任何内存。相反，让我们在外部范围内分配一次切片，然后将其传递给此函数以进行填充。

```
func randArray(a []interface{}) {
    for i := 0; i < len(a); i++ {
        a[i] = byte(rand.Int() % 255)
    }
}


```

当我们这样做时，我们完全消除了所有垃圾收集压力，并且我们有效地将所有 CPU 周期用于做有用的工作。

![](https://mmbiz.qpic.cn/mmbiz_png/jdTRWcqrowQrCIIOHBeoWuk3TVTZHclcnQiaNAO1en4tbnWpnQEJd2xJwe9QcEWVthoQDO0kgPNHLBLz82eb9sg/640?wx_fmt=png)

我们不必将切片作为参数传入，我们只需要避免每次调用此函数时都分配它。实现此目的的另一种方法是使用全局`sync.pool`变量在函数调用之间重用切片。

### 总结

通过调用 rand.Int 所花费的时间作为我们花费多少 CPU 时间做有用工作的示例，我们得到以下摘要：

*   appending: 20%
    
*   interface{} 类型静态切片: 38%
    
*   静态 byte 切片 : 62%
    
*   切片作为参数: 70%
    

底线：如何使用切片会对程序的整体性能产生非常显着的影响。

案例研究：结构体还是指针作为接收器
-----------------

关于在实现接口时是使用结构体还是指向结构体的指针作为接收器类型，golang 社区中一直存在着非常激烈的争论。意见不一。

现实情况是，这两种方法都需要权衡取舍。将结构体复制到堆栈上通常比指针更昂贵，尤其是对于大型结构。但是将对象保留在堆栈上而不是堆上可以避免垃圾收集器的压力，这在某些情况下可能会更快。从美学 / 设计的角度来看，也存在权衡：有时确实需要强制执行不变性语义，这可以通过结构体接收器获得。

让我们来说明你为大型结构付出的性能损失。我们将使用一个非常大的 36 个字段，并在其上一遍又一遍地调用一个方法。

```
type bigStruct struct {
    v1, v2, v3, v4, v5, v6, v7, v8, v9 uint64
    f1, f2, f3, f4, f5, f6, f7, f8, f9 float64
    b1, b2, b3, b4, b5, b6, b7, b8, b9 []byte
    s1, s2, s3, s4, s5, s6, s7, s8, s9 string
}

func (b bigStruct) randFloat() float64 {
    x := rand.Float32()
    switch {
    case x < .1:
        return b.f1
        …
}

```

当我们一次又一次地分析这个方法时，我们可以看到，我们在一个名为 `runtime.duffcopy` 的东西上付出了非常大的代价，35% 的 CPU 周期。

![](https://mmbiz.qpic.cn/mmbiz_png/jdTRWcqrowQrCIIOHBeoWuk3TVTZHclcV73zQBKfKRo8gH2dJ4sibGyx4PZ1Qk0iaokuRU7efzOTgK9svu5FvCag/640?wx_fmt=png)

`runtime.duffcopy` 是什么？在某些情况下，这是 go runtime 复制大量连续内存块（通常是结构）的方式。之所以这样称呼它，是因为 `duffcopy` 有点像 Duff 的设备，它是 Tom Duff 在 80 年代发现的 C 编译器黑客。他意识到你可以滥用 C 编译器通过交错循环和开关的构造来实现循环展开：

```
register short *to, *from;
register count;
{
    register n = (count + 7) / 8;
    switch (count % 8) {
    case 0: do { *to = *from++;
    case 7:      *to = *from++;
    case 6:      *to = *from++;
    case 5:      *to = *from++;
    case 4:      *to = *from++;
    case 3:      *to = *from++;
    case 2:      *to = *from++;
    case 1:      *to = *from++;
            } while (--n > 0);
    }
}


```

循环展开可以极大地加快速度，因为你不必在每次通过循环时检查循环条件。Go 的 `duffcopy` 实际上并不是 Duff 的设备，但它是一个循环展开：编译器发出 N 条指令来复制内存，而不是使用循环来这样做。

避免支付这种惩罚的方法是简单地使用指向结构的指针作为接收者，避免昂贵的内存复制操作。但是在概括这个建议时要小心——你对哪种技术性能更好的直觉可能是不正确的，因为它实际上取决于许多与你的应用程序不同的因素。归根结底，你确实需要对替代方案进行 profile 分析，以了解哪些方案总体上表现更好，包括垃圾收集的影响。

总结
--

要使 Dolt 与更成熟的数据库技术一样具有高性能，我们还有很多工作要做，尤其是在查询计划方面。但是我们已经设法通过这些简单的优化将引擎的性能提高了 2 倍，并通过完全重写我们的存储层再提高了 3 倍 [2]。在简单的基准测试中，我们现在的延迟大约是 MySQL 的 2.5 倍，但我们还没有完成。随着我们接近 1.0 版本，我们很高兴能够继续提高我们的数据库速度。

原文链接：https://www.dolthub.com/blog/2022-10-14-golang-performance-case-studies/

### 参考资料

[1]

Dolt DB: https://doltdb.com/

[2]

并通过完全重写我们的存储层再提高了 3 倍: https://www.dolthub.com/blog/2022-09-30-new-format-default/

* * *