> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/WUrsmXo3vvb_rZRrKgWCQg)

> 介绍简洁的代码对于创建可维护、可阅读和高效的软件至关重要。Go 是一种强调简单和代码整洁的语言。在本文中，我

介绍  

简洁的代码对于创建可维护、可阅读和高效的软件至关重要。Go 是一种强调简单和代码整洁的语言。在本文中，我们将结合代码示例，探讨编写简洁 Go 代码的最佳实践。

有意义的变量和函数名称
===========

使用能表达变量和函数用途的描述性名称。避免使用隐晦或过于简短的名称。

```
// Bad:
func fn(x int) int {
    // ...
}

// Good:
func calculateFactorial(number int) int {
    // ...
}


```

格式一致
====

使用 gofmt 遵守 Go 社区的格式化指南，实现一致的代码外观。

```
// Bad:
func calculate(x int){y:=x+10;fmt.Println(y);}

// Good:
func calculate(x int) {
    y := x + 10
    fmt.Println(y)
}


```

正确缩进
====

保持适当的缩进，以提高代码的可读性。

```
// Bad:
if true {
for i := 0; i < 5; i++ {
fmt.Println(i)
}
}

// Good:
if true {
    for i := 0; i < 5; i++ {
        fmt.Println(i)
    }
}


```

简短函数和方法
=======

将函数分解成更小的单元，使其更加清晰。也更有利于编写单元测试。

```
// Bad:
func complexLogic(x int, y int) int {
    // 50 lines of code
}

// Good:
func calculateSum(x int, y int) int {
    return x + y
}
func calculateDifference(x int, y int) int {
    return x - y
}


```

避免深度嵌套
======

保持浅层嵌套，以提高代码的可读性。

```
// Bad:
if conditionA {
    if conditionB {
        if conditionC {
            // ...
        }
    }
}

// Good:
if conditionA && conditionB && conditionC {
    // ...
}


```

注释和文档
=====

必要时提供注释，但应优先编写不言自明的代码。

```
// Bad:
// Increment counter
counter++

// Good:
counter++


```

错误处理
====

认真处理错误，使代码更加稳健。

```
// Bad:
func divide(a, b int) int {
    return a / b // Division by zero not handled
}

// Good:
func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}


```

避免使用全局变量
========

限制全局变量的使用，以提高代码的清晰度。

```
// Bad:
var globalCounter int

func increment() {
    globalCounter++
}

// Good:
func increment(counter int) int {
    return counter + 1
}


```

单一责任原则
======

遵循 SRP，保持职能集中。

```
// Bad:
func processUserData(user User) {
    // Does everything related to user data
}

// Good:
func saveUserData(user User) {
    // Save user data to the database
}
func sendWelcomeEmail(user User) {
    // Send a welcome email to the user
}


```

DRY（不要重复自己）
===========

将普通代码重构为可重复使用的函数。

```
// Bad:
func calculateAreaOfCircle(radius float64) float64 {
    return 3.14159 * radius * radius
}
func calculateAreaOfRectangle(width float64, height float64) float64 {
    return width * height
}

// Good:
func calculateArea(shape string, params ...float64) float64 {
    switch shape {
    case "circle":
        return 3.14159 * params[0] * params[0]
    case "rectangle":
        return params[0] * params[1]
    }
    return 0
}


```

测试驱动开发（TDD）
===========

在实现功能前编写测试

```
// Test
func TestCalculateSum(t *testing.T) {
    result := calculateSum(3, 5)
    if result != 8 {
        t.Errorf("Expected 8, but got %d", result)
    }
}


```

使用标准库
=====

利用标准库实现常用功能。

```
// Bad:
func customStringReverse(s string) string {
    // Custom implementation of string reversal
}

// Good:
import "strings"
func reverseString(s string) string {
    return strings.Reverse(s)
}


```

利用接口
====

使用接口定义组件之间的契约。

```
type Shape interface {
    Area() float64
}

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return 3.14159 * c.Radius * c.Radius
}


```

一致的错误处理模式
=========

保持一致的错误处理方法。

```
// Bad:
func fetchData() {
    if err := retrieveData(); err != nil {
        log.Fatal(err)
    }
}

// Good:
func fetchData() error {
    data, err := retrieveData()
    if err != nil {
        return err
    }
    // Process data
    return nil
}


```

结论
==

整洁的代码对于可维护、高效的 Go 软件开发至关重要。坚持这些最佳实践并将其融入您的编码习惯，您将提高代码的可读性、协作性和整体代码质量。整洁的代码不仅仅关乎美观，它还是一种有助于项目成功的基本方法。

最近微信读书上架了《代码整洁之道：程序员的职业素养》，之后会再出一个阅读记录的合集。感兴趣的同学可以多多关注。  

附录
==

[go 最佳实践：如何舒适地编码](http://mp.weixin.qq.com/s?__biz=MzU0MDkyOTg5Nw==&mid=2247483931&idx=1&sn=7ef4d42b7c9cb24f1f794efb4ac7d7ab&chksm=fb30f81dcc47710b0d86b6358d97f9165964256eeb0ce1a2601d96eabe5d85e951cc43628880&scene=21#wechat_redirect)  

[go 语言最佳实践](http://mp.weixin.qq.com/s?__biz=MzU0MDkyOTg5Nw==&mid=2247483875&idx=1&sn=cd1050b8ae53896e4541f80250486a43&chksm=fb30fbe5cc4772f30967e4997161d0337016ec6b77ba6e4ef057afc399fc5510536124e22add&scene=21#wechat_redirect)