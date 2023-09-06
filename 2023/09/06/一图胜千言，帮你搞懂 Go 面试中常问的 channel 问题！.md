> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/r4rnyXoHb5EInBpO9rhQqA)

> 详解面试中常问的 select 和 channel 问题

一图胜千言
-----

下面的表格中总结了对不同状态下的通道执行相应操作的结果。

![](https://mmbiz.qpic.cn/mmbiz_png/NUARwspaut7hACGKiab5CyF8icRAHN7dSbacVvIZwZqlptCIVHWgb5WrFialJDRrxMOuicujdyF8LDk8bD9fdeEIXg/640?wx_fmt=png)

注意：对已经关闭的通道再执行 close 也会引发 panic。

点击下方卡片，在公众号后台回复：**Go**，就可以获得**免费硬核**的《Go 语言学习专栏》，优质内容持续更新中。

又有朋友催更，让我出面试题系列了，安排！  

这篇文章将重点讲解 Go 面试进阶知识点：select 和 channel。

select
======

先说 switch...case...
-------------------

switch...case... 很常用，且很好理解。其作用和 if...else... 一样。

区别是 switch...case 相比于 if...else... 能让我们的代码看起来更清晰，更好理解。

再说 select...case..
------------------

golang 的 select 就是监听 IO 操作，当 IO 操作发生时，触发相应的动作。 

所说的 IO 操作就是对 channle 的操作：向通道发送数据，或者从通道中读取数据。

在执行 select 语句的时候，运行时系统会自上而下地判断每个 case 中的发送或接收操作是否可以被立即执行。

### 什么是立即执行呢？

立即执行：意思是当前 Goroutine 不会因当前操作而被阻塞

select 类比 switch
----------------

select 的用法与 switch 非常类似，由 select 开始一个新的选择块，每个选择条件由 case 语句来描述。

与 switch 语句可以选择任何可使用相等比较的条件相比，select 有比较多的限制，其中最大的一条限制就是每个 case 语句里必须是一个 IO 操作。

确切的说，应该是一个面向 channel 的 IO 操作。

经典示例
----

```
package main

import "fmt"

func main() {
   ch1 := make(chan int, 1)
   ch1 <- 2
   select {
   case v := <-ch1:
      fmt.Println("取到的数据：", v)
   case ch1 <- 1:
      fmt.Println("写入数据")
   }
}


```

### 运行结果

![](https://mmbiz.qpic.cn/mmbiz_png/NUARwspaut7hACGKiab5CyF8icRAHN7dSbb1icjFwpCoicWtzc7VmIgHPJ6rWmykUCFxnYhGzdUe3tq3wX6T2f8tLg/640?wx_fmt=png)

channel
=======

goroutine 和 channel 作为 go 语言中最重要的两个知识点，一定要搞清楚。

大家容易出错的知识点是以下 3 点，尤其是最后一点：

*   nil channel 代表 channel 未初始化，向未初始化的 channel 读写数据会造成阻塞
    
*   关闭 (close) 未初始化的 channel 会引起 panic。
    
*   从一个关闭的并且没有值的通道执行接收操作会得到对应类型的零值，并不会引起 panic。
    

1. 从已经关闭并且没有值的通道中取值
-------------------

```
package main

import "fmt"

//从关闭的通道中取值示例：
func main() {
   //声明实例化通道ch1
   ch1 := make(chan int, 1)
   //关闭通道
   close(ch1)
   select {
   //通通道ch1中取值
   case v := <-ch1:
      fmt.Printf("从ch1中取值：%d\n", v)
   default:
      fmt.Println("默认case")
   }
}


```

### 运行结果

和我们预想中的一样，取到了对应的零值： ![](https://mmbiz.qpic.cn/mmbiz_png/NUARwspaut7hACGKiab5CyF8icRAHN7dSbr0Xf19eCLNW17pEG44f4KnefISMeZLbyYts4icuAe8XpqMMdD81Mt5w/640?wx_fmt=png)

2. 从已经关闭并且有值的通道中取值
------------------

我们稍微修改一下上面的代码

```
package main

import "fmt"

//从关闭的通道中取值示例：
func main() {
   //声明实例化通道ch1
   ch1 := make(chan int, 1)
   //向通道中赋值
   ch1 <- 1
   //关闭通道
   close(ch1)
   //关闭之后取值
   after_close_value := <-ch1
   fmt.Printf("关闭之后取值：%d\n", after_close_value) //打印结果：关闭之后取值：1
   select {
   //通通道ch1中取值
   case v := <-ch1:
      fmt.Printf("从ch1中取值：%d\n", v) //打印结果：从ch1中取值：0
   default:
      fmt.Println("默认case")
   }
}


```

### 运行结果

运行结果和我们预想中的一样：

*   通道关闭后，如果通道中仍然有值，还是可以正常取到通道中的值的。
    
*   通道关闭后，如果通道中已经没有值了，再从通道中取值，并不会引起 panic，而是会取到对应类型的零值。
    

![](https://mmbiz.qpic.cn/mmbiz_png/NUARwspaut7hACGKiab5CyF8icRAHN7dSb1PB3uy0DPicZBESotfs3ecPyVn3lxiaYib8QqJ2ial5WGHYSXmOKadQ91g/640?wx_fmt=png)

一图胜千言
-----

下面的表格中总结了对不同状态下的通道执行相应操作的结果。

![](https://mmbiz.qpic.cn/mmbiz_png/NUARwspaut7hACGKiab5CyF8icRAHN7dSbacVvIZwZqlptCIVHWgb5WrFialJDRrxMOuicujdyF8LDk8bD9fdeEIXg/640?wx_fmt=png)

注意：对已经关闭的通道再执行 close 也会引发 panic。

总结
==

这篇文章解析了 Go 语言中 select 和 channel 在面试中可能遇到的进阶知识点。

感兴趣的小伙伴可以关注我的专栏：

[#Go 面试题必知必会系列](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzIyNjM0MzQyNg==&action=getalbum&album_id=2723656549751128070&scene=173&from_msgid=2247488684&from_itemidx=1&count=3&nolastread=1#wechat_redirect)