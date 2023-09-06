> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ie7LqXZQOUX0Bfn4QhHzLA)

> google 出品的，基于 lz77 压缩算法，在压缩比可接受情况下尽量提高压缩速度的压缩算法。

大家好，我是渔夫子。

今天给大家推荐的是一个 google 开源的快速、无损的压缩包：**snappy。**

snappy 算法是 google 开源的。该包是 google 使用 go 语言来实现的。项目地址如下：

项目地址：https://github.com/golang/snappy

星标：1.4k

使用者：97.7k

简介
--

该包的目标并不是最大化的压缩比例，也不是和其他压缩库兼容；相反，snappy 算法的目标是在**合理的压缩率**下尽可能的**提高压缩速度**。

例如，与 zlib 的最快压缩模式相比，snappy 依然比其快了一个数量级，但产生的压缩文件要比 zip 的大 20% 到 100%。

特性
--

snappy 压缩算法具有以下特性：

*   **快速：**压缩速度大概在 250MB / 秒及更快的速度进行压缩。
    
*   **稳定：**在过去的几年中，Snappy 在 Google 的生产环境中压缩并解压缩了数 P 字节（petabytes）的数据。Snappy 位流格式是稳定的，不会在版本之间发生变化
    
*   **健壮性：**Snappy 解压缩器设计为不会因遇到损坏或恶意输入而崩溃
    

性能
--

Snappy 的目标是快速。在 64 位模式下，一个 Corei7 处理器的单核上，其压缩速度约为 250MB / 秒或更快，解压缩速度约为 500MB / 秒或更快。（这些数字是在我们的基准测试套件中最慢的输入情况下得出的；其他输入会快得多。）在我们的测试中，Snappy 通常比同一级别的算法（如 LZO、LZF、QuickLZ 等）更快，同时实现了类似的压缩率。

示例
--

我们看下 snappy 的使用。如下：

```
package main

import (
    "fmt"
    "github.com/golang/snappy"
)

func main() {
    fmt.Println([]byte("aaa"))
    src1 := []byte("akakakakakakakakakakdddddddddcccccceeeeeeeegggggggggsssss")

    var dst1 []byte
    c := snappy.Encode(dst1, src1)
    fmt.Printf("src1 before compression len:%d\n", len(src1)) 
    fmt.Printf("src1 after compression len:%d\n", len(c))
}


```

运行代码，可知压缩前字符串是 57 个字节，压缩后是 34 个字节。结果如下：

```
src1 before compression len:57
src1 after compression len:34


```

但是，有时候你会发现，压缩后会比压缩前字节数变大。这是和原字符串有关系，如果原字符串中重复的字符越少，那么压缩后的长度就有可能会比之前变长。如果原字符串中重复的字符比较多，那么压缩比率就会很高。这也是压缩的基本原理。