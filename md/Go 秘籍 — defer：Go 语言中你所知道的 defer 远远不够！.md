> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/yHdttEsXeYHqwpF3_Ky_Mg)

> 本文深入探讨了 Go 语言中 “defer” 语句的一些鲜为人知的复杂性，揭示了你可能还不知道的一些秘密。

嘿，Go 开发者们！你认为你对 **defer** 已经了然于胸了吗？再想想吧！

相信我，它远不止于将函数执行推迟到当前函数的末尾。无论你是刚开始接触 Go 还是经验丰富的专业人士，你都肯定会从这篇指南中学到一些新的、超级有用的东西。

让我们开始吧。

1. 评估参数
=======

你知道在 Go 语言中，deferred 函数的参数在执行`defer`语句时进行评估，而不是在调用 deferred 函数时进行评估吗？

看下面的示例：

```
package main

import "fmt"

func main() {
    i := 0
    defer fmt.Println(i)
    i++
}

```

> 你预期在控制台中看到什么输出？

尽管`fmt.Println(i)`将在最后执行，也就是在 i 递增为 1 之后，但它仍会将 0 打印到控制台。有点奇怪，对吧？

这是因为在执行`defer`语句时，`i`的值被评估为 0，此时`i`还是 0。

> “该怎么修复这个问题？”

修复这个问题非常简单，只需将参数移动到 defer 函数的函数体内即可。

```
func main() {
    i := 0
    defer func() {
        fmt.Println(i)
    }()
    i++
}

// 1

```

> “哦，这个我早就知道了！”

如果这个并没有让你吃惊，那么让我们继续下一个部分，用一个不同的 Go 特性来扩展这个想法。

2. 评估函数接收器
==========

由于在 Go 中，deferred 函数在`defer`语句执行时进行评估，而不是在调用 deferred 函数时进行评估，让我们看另一个使用 defer 与函数接收器的例子：

```
package main

import "fmt"

type Person struct {
    Name string
}

func (p Person) SayHi() {
    fmt.Println("Hi, my name is", p.Name)
}

func main() {
    writer := Person{Name: "Joe"}
    defer writer.SayHi()

    // fix the wrong name
    writer.Name = "Aiden"
}

```

在这个例子中，你可以看到`writer.SayHi()`函数没有任何参数，与之前的例子不同。

> 那么，你期望在控制台上看到什么输出？

`writer.SayHi()`函数将在最后执行。它应该输出 "Hi, my name is Aiden"。

然而，`writer`的值是在`defer`语句定义时进行评估的，所以实际结果将是 "Hi, my name is Joe"。

由于`SayHi()`方法具有值接收器，当调用`defer`语句时，它将在一个副本上执行`writer`变量，并稍后在该复制版本上执行方法。

当你想在初始化或更改对象后对其进行操作时，这可能是危险的。

> “我可以使用第一节中的解决方案来解决这个问题吗？”

是的，绝对可以。

然而，还有另一种方法可以实现期望的结果。你可以将值接收器更改为指针接收器：

```
// Use *Person, not Person
func (p *Person) SayHi() {
    fmt.Println("Hi, my name is", p.Name)
}

```

这种解决方案也不是理想的或优雅的，因为我们实际上没有改变`person`对象，这里使用指针是多余的。我只是提及这个解决方案供学习目的。

3. 不要担心 panic，`defer` 依然会被调用
============================

`defer` 关键字允许在包含它的函数完成后执行一个函数，即使该函数发生 panic 也是如此。这对于执行清理任务很有用，比如关闭文件或解锁（分布式）互斥锁。

为了避免关闭服务，通常会使用 `defer` 结合 `recover` 来处理 panic 或意外异常：

```
func main() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered from panic:", r)
        }
    }()

    panic("This is a panic!")
}

// Recovered from panic: This is a panic!

```

这就是为什么许多 Web 服务器库（如 gin、iris 等）都内置了基于这种技术的 `recover` 中间件。

4. Defer 可以改变函数的结果
==================

如果你使用了具名返回值，比如下面的例子中的 `x`：

```
func deferMe() (x int) {
    x = 1
    return
}

```

使用 `defer` 技巧，你可以在函数返回之前修改函数的结果，就像这样：

```
func main() {
    fmt.Println(deferMe()) // 2
}

func deferMe() (x int) {
    defer func() { x *= 2 }()

    x = 1

    return
}

```

请记住，使用这种方法可能会使代码变得更难理解和调试。只在必要的情况和适当的场景下使用这种方法（也许不要在生产代码中使用）。

5. Defer 顺序 — 后进先出
==================

提醒一下，当使用`defer`语句时，这些语句按照后进先出（Last In First Out）的顺序执行。因此，如果你有多个`defer`语句，它们将按照调用顺序的相反顺序执行：

```
func main() {
    defer fmt.Println("Defer 1")
    defer fmt.Println("Defer 2")
    defer fmt.Println("Defer 3")
}

// Defer 3
// Defer 2
// Defer 1

```

总结一下，`defer`是 Go 语言中非常有用的工具。它允许你在调用它的函数执行完毕后运行一个函数，即使该函数出现了恐慌。

请始终保持学习的状态，并享受编程的乐趣，**祝你编码愉快**！