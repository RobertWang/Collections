> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/YR5fEnfhGNNJt-POWJoZxw)

> Goland 激活码 & 个人账号开通授权，支持版本升级

在 Go 语言中，使用内置的 `testing` 包可以编写单元测试。下面是一个简单的示例，演示如何编写和运行单元测试：

```
package main

import "testing"

// 需要进行测试的函数，这里以一个简单的加法函数为例
func Add(a, b int) int {
    return a + b
}

// 单元测试函数，函数名以 Test 开头，接受一个 *testing.T 参数
func TestAdd(t *testing.T) {
    result := Add(2, 3)
    expected := 5

    // 使用 testing 包提供的断言函数来比较结果和预期值
    if result != expected {
        t.Errorf("Add(2, 3) returned %d, expected %d", result, expected)
    }
}

```

在上述示例中，我们定义了一个简单的加法函数 `Add`，并编写了一个对其进行单元测试的函数 `TestAdd`。注意，单元测试函数的函数名以 `Test` 开头，接受一个 `*testing.T` 参数。

[**Goland 激活教程，免费领取**](http://mp.weixin.qq.com/s?__biz=MzU2MDA0Mzk0OQ==&mid=2247485470&idx=1&sn=9bd7d012cdb16b19239315985d692f58&chksm=fc0f4adccb78c3cac4c0e59d98827f0af33576bdfa634867cedbe65de4a5dbc27efa7a550685&scene=21#wechat_redirect)  

在 `TestAdd` 函数中，我们调用 `Add` 函数并将结果与预期值进行比较。如果结果与预期值不相等，我们使用 `t.Errorf` 函数记录错误消息。

要运行单元测试，可以使用 `go test` 命令。在终端中进入源代码所在的目录，并执行以下命令：

```
go test


```

Go 语言会自动查找并运行当前目录及其子目录中所有以 `Test` 开头的函数，并报告测试结果。

输出示例：

```
PASS
ok      github.com/your/package   0.007s


```

输出中的 `PASS` 表示所有的单元测试通过。如果有任何测试失败，它们会被列出，并显示失败的详细信息。

除了简单的单元测试，Go 语言还提供了丰富的测试功能，包括表格驱动测试、子测试、测试覆盖率分析等。可以在单元测试中使用断言函数、辅助函数和各种测试选项，以满足不同的测试需求。

注意，单元测试应该是可重复的和独立的，应该只关注被测试函数的行为和输出。在编写测试时，应该考虑不同的边界情况和错误处理，以确保被测试的代码具有正确的行为和鲁棒性。

有关更详细的测试用法和示例，请参考 Go 语言的官方文档：https://golang.org/pkg/testing/