> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/gNAdglGGgoOay3c7kjA7Tw)

make 和 new 是 Go 语言中非常重要的两个内置函数, 用于内存分配和对象初始化。合理正确使用 make 和 new 是 Go 语言开发的基础技能。

> 1.  new 和 make 函数介绍
>     
> 2.  new 和 make 的区别
>     
> 3.  new 函数的实现原理
>     
> 4.  make 函数的实现原理
>     
> 5.  make 初始化切片
>     
> 6.  make 初始化 map
>     
> 7.  make 初始化 channel
>     
> 8.  new 和 make 的性能比较
>     
> 9.  new 和 make 的使用指南
>     
> 10.  示例代码
>     
> 11.  最佳实践
>     

通过详细的概念讲解、源码解析和大量示例, 可以全面深入地掌握 Go 语言中 new 和 make 的用法和实现原理, 运用自如。

**1. new 和 make 函数介绍**

new 和 make 都是 Go 语言中用于内存分配和初始化的内置函数:

> *   new 主要用来分配基本类型、数组、结构体等类型的内存空间, 返回指向这些空间的指针
>     
> *   make 主要用来分配切片、map、channel 等内置引用类型并初始化, 返回引用类型本身
>     

new 和 make 的函数签名:

```
func new(Type) *Type 
func make(Type, size ...) Type

```

new 返回指针, 而 make 返回引用类型本身。

**2. new 和 make 的区别**

尽管两者都用于内存分配, 但新 new 和 make 有以下关键区别:

> *   new 返回指针, make 返回引用类型
>     
> *   new 没有参数, make 需要指定类型
>     
> *   make 返回初始化后可以直接使用的对象, new 返回还需要初始化的指针
>     
> *   make 只用于 slice、map、channel 等内置引用类型, new 可用于任意类型
>     

一般来说, 所有内置引用类型优先使用 make 进行初始化。

**3. new 函数实现原理**

new 函数的作用很简单, 就是分配内存, 并返回该内存空间地址的指针。

new 函数源码实现如下:

这个过程对应的 runtime 底层实现:

> 1.  调用 mallocgc 分配一段内存
>     
> 2.  将这段内存空间清零
>     
> 3.  返回这段内存的指针
>     

以 new 一个 int 指针为例:

```
p := new(int)

```

对应的汇编实现:

```
MOVQ runtime.mallocgc(SB), AX  
CALL AX
MOVQ ZEROBASE, BX
MOVUPS ZERO(SB), X0
...
RET

```

new 直接返回内存指针, 对应的对象还需要进一步初始化才能使用。

**4. make 函数实现原理**

make 函数将 slices、maps 和 channels 等引用类型初始化之后返回引用:

```
make([]int, 100, 200) 
make(map[string]int)
make(chan int)

```

make 的实现原理是:

> 1.  根据参数申请对应大小的内存
>     
> 2.  对内存进行必要的初始化
>     
> 3.  返回类型的引用
>     

初始化的工作包括:

> *   切片: 初始化相关字段, 如 ptr、len、cap 等
>     
> *   map: 创建散列数组、桶数组等数据结构
>     
> *   channel: 创建环形缓冲区
>     

make 返回的引用可以直接使用, 无需再初始化。

**5. 切片 make 实现**

切片在 make 时其实是创建了一个 slice 对象:

```
type slice struct {
  array *int
  len int
  cap int   
}

```

切片 make 的要点是计算切片容量 cap 的分配原则:

> 1.  如果指定了容量 cap, 直接使用该值
>     
> 2.  仅指定 len, 则 cap=len
>     
> 3.  如果两者都没指定, 容量 cap 取默认值
>     

这样可以很灵活地控制切片容量。

**6. map make 实现**

make 初始化 map 整体实现步骤:

> 1.  根据容量估算需要的 bucket 数
>     
> 2.  分配连续内存块作为 bucket 数组
>     
> 3.  初始化 bucket 内存空间
>     
> 4.  安装扩容因子、阈值等字段
>     
> 5.  返回 map 引用
>     

举个例子:

```
m := make(map[string]int, 100)

```

根据传入的容量 100 估算 bucket 数, 初始化 bucket 数组, 然后返回一个可以直接使用的 map 引用。

**7. channel make 实现**

make 初始化 channel 的核心是创建一个环形缓冲区:

```
// 分配缓冲区内存空间
buf := make([]Type, cap)  
// 初始化读写指针
c.sendx = 0 
c.recvx = 0
c.recvq = nil
c.sendq = nil
...
// 返回channel
return c

```

这样在 make 之后 channel 可以直接发送和接收数据。

**8. new 和 make 性能比较**

new 和 make 在性能上的区别主要如下:

> *   new 分配内存比较快, 但是不会进行初始化
>     
> *   make 需要初始化, 相对比较慢
>     

但是 make 返回的对象可以直接使用, 总体上性能更好。

此外, new/make 分配小对象时采用对象池技术, 可以提高分配效率。

**9. 使用指南**

综合上面的分析, 使用 new 和 make 的一些原则:

> *   new 用于值类型和用户定义的类型, 如数组、结构体
>     
> *   make 用于内置引用类型, 如切片、map、channel
>     

一个简单的区分记忆法则是:

> *   new 给类型本身分配内存
>     
> *   make 从类型生成对象
>     

日常使用中, 对内置引用类型要优先考虑使用 make 进行初始化。

**10. 示例代码**

下面示例代码演示 make/new 的基本用法:

new 一个指针:

```
p := new(int)
*p = 123 // 初始化后使用

```

make 一个切片:

```
s := make([]int, 0) 
s = append(s, 1) // 直接使用

```

make 一个带 cap 的 map:

```
m := make(map[string]int, 100) 
m["a"] = 1 // 直接使用

```

make 一个带缓冲 channel:

```
c := make(chan int, 10)
c <- 1 // 直接发送

```

**11. 最佳实践**

基于上面的讲解, 总结一些使用 make 和 new 的最佳实践:

> *   优先为内置引用类型使用 make 初始化
>     
> *   新建值类型较多的结构体或数组时选用 new
>     
> *   需要指针对象才使用 new
>     
> *   根据类型特点和场景选择最优方案
>     
> *   注意复用对象减少内存分配次数
>     

比如缓冲 channel 等引用类型使用 make, 大批结构体使用 new 会更好。

**总结**

通过这篇全面详细的文章, 包括概念讲解、实现原理、使用指南、示例代码等内容, 可以深刻理解 Go 语言中 make 和 new 的区别与用法, 掌握它们的设计思想, 熟练应用。