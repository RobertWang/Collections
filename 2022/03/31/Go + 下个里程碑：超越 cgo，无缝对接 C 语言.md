> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/iMCgNQaQ0aNUtR3GkPW6Qw)

去年（2021 年）Go+ 的 slogan 从 “面向数据科学” 的语言升级到了 “面向工程、STEM 教育与数据科学” 三位一体的语言。也就是说，我们希望 Go+ 可以同时被软件工程师、中小学生、数据分析师这三个截然不同的人群所广泛使用。

因为到目前为止，Go+ 从 “面向工程” 这个目标来说，它只是 “更好的 Go”。客观来讲，这个 “更好” 并不足以让软件工程师们心动到改变自己的习惯，采用 Go+ 来进行日常的开发工作。这个 Why Go+ 如果回答不好，那么 Go+ 的面向工程这个目标就仅仅停留于口号。

我们以 Web3 为例。这很可能是与服务端开发最为接近，Go 最容易取得压倒性优势的领域。但是实际的战绩如何呢？我花时间对几十个头部的 Web3 项目进行了分析，最后的统计结果很意外：在这个领域 Go 语言的确采用率很高的确没错，但是有另一个语言 Rust 的采用率也很高，两者的采用率基本上接近 1:1，甚至一些项目同时采用 Go 和 Rust。

为什么会这样？在正统的服务端开发中 Rust 的采用率可能连 Go 的 1/10 都没有，为什么一个看起来完全类似的同样都是网络应用类的场景，其采用率居然有这么大的差别？

我知道一些人把这个事情归因到 Go 是 GC 语言，Rust 性能更好这一点上。但我认为这并不成立。如果对网络服务性能是重要的，那么在服务端开发（尤其是云计算）这样一个成本很敏感的领域，Rust 应该比 Go 更有市场竞争力才对。

我认为最有可能的关键因素是两个：

*   语言带给人的安全感（更少的安全漏洞）；
    
*   C 语言的兼容性。
    

Rust 的确能够带来更好的安全感（至少表面上看是这样），但这个因素能够影响到底有多少不得而知。但我认为 C 语言的兼容性是更为重要的因素。Web3 是一个快赛道，发展日新月异，大家都会抢时间。从抢时间这个角度来看，只对比语言角度 Go 比 Rust 有优势得多。是什么原因让大家觉得用 Rust 开发起来更快？

因为 Rust 能够让软件工程师们复用大量的 C 语言的社区资源。编程语言史发展到今天，哪门语言资源最多我想不用我在这里多费口舌，对于 Web3 这样一个日新月异的新领域来说，能够复用既有 C 语言的资源，可以轻易打败 Go 仅仅靠语言特性本身形成的便捷性优势。

简单一句话，cgo 太鸡肋，与 C 语言的兼容上，Go 也就是做到了聊胜于无而已。

这里 Web3 只是一个例子。无论进入到任何服务端之外的新领域，对 Go 来说，兼容 C 都是至关重要，没有之一。

想清楚了这一点，Go+ 面向工程领域的第一个执行目标就出来了：实现对 C 语言的完美兼容。要么让 cgo 变好，要么提供一个远超 cgo 的新的兼容 C 语言的方案。

这就是去年年底，为什么我们调整 Go+ v1.2 版本的迭代目标的原因。

目标有了，但是怎么做到呢？刚刚过去的这个春节里我几乎每天一有空就在思考这个问题。为此我查阅了大量的 github 上的代码，基本上围绕着 c2go、c2goasm、cgo、binary/asm2go、bytecode/vm2go 这些话题。

最终，我没有选择优化 cgo，而是选择了：c2go。简单说，就是把 C 代码转换为 Go 代码，然后重新用 Go 编译器进行编译。

当我把要做 c2go 这个想法发到 Go+ 贡献者群的时候，大家第一反应都是觉得不可能，这太难了。

这条路并非没有人走过。

大家都知道，Go 团队自己就做过这事（参考：https://github.com/rsc/c2go），当然他们的目标并不是做一个通用版本的 c2go，而是要实现 Go 的自举，把 Go 编译器（也包括运行时等）所有的 C 代码都转成 Go。所以它的代码可以针对 Go 编译器写特殊的转换规则，只要满足不别人肉去改编译器就好。

这样就得到 C 语言的抽象语法树（C AST）。这一步自己做也可以，但是既然有现成的，能够先省心就先省心吧，以后有空了再改写不迟（注意 github.com/elliotchance/c2go 这个项目用的是 -ast-dump 开关，而不是 -ast-dump=json，这样就不得不在再写一个额外的文本解析代码来解析 clang 的输出）。

接着，就是将 C AST 转为 Go AST。这一步最为核心，我们的大部分工作都集中在这里。类似的工作我们在 Go+ 中也已经做过了，只不过我们之前做的是 Go+ AST 转为 Go AST 而已。考虑 C 语言的 spec 比 Go+ 语言 spec 要小很多，我们心里对工作量就有了大体的估计。

这一步会有少量的技术难点，比如 C 的指针是可以移动的，比如 C 语言有 union 数据类型。但总体来说这些语法都不是特别复杂，工作量是可控的。

C 是手工管理内存的，这点在转为 Go 后可以仍然保持不变。C 语言的 malloc 函数可以从 Go 进程里面划出一片内存来进行手工管理。

还有一些 corner case，比如在 C 代码里面插入汇编指令的。我们可以考虑引入一个通用的过滤机制来解决：在 c2go 转换过程的配置文件中，指定哪些函数或 C 源文件应该被忽略，然后在该工程中加入一个手工实现的 Go 源文件来自己实现这些被忽略的函数即可。

在介绍 Go+ 编译器实现原理（请移步 bilibili 搜索 "Go+ 公开课 · 第 1 期｜Go+ v1.x 的设计与实现" 进行查看）的时候，我提到过 github.com/goplus/gox 这个项目，它是用来辅助生成 Go AST 的模块。在 C AST 转为 Go AST 中，我们也会借助它来大幅减少开发工作量。

生成了 Go AST，剩下来的工作就和我们在 Go+ 一样了，通过 Go AST 调用 go/format 这个标准库来生成 Go 源文件，最后用 Go 编译器编译它。

在我解释以上 c2go 的全过程后，小伙伴们开始相信这条路走得通。

当然，在我内心中，还有一个更为重要的理由：我们有机会干得比 cgo 更好。如果选择优化 cgo 的话，是达不到这个目标的，我们唯一能够做的只是让 cgo 更快。

**如何让 Go+ 对 C 语言的支持更好？**

在 Go+ 与 C 语言的互操作性上，我有以下这些核心的考量准则：

其一，C 代码不需要经过额外包装，可以直接由 Go+ 来调用。以 Hello world 这个经典程序为例，Go+ 调用 C 标准库的 printf 代码如下：

它挺像 cgo，但是它并不是。你可以拿它和 cgo 的代码对比一下：

```
package main

/*
#include <stdio.h>
#include <stdlib.h>
*/

import "C"
import "unsafe"

func main() {
    cstr := C.CString("Hello World\n")
    defer C.free(unsafe.Pointer(cstr))
    C.printf(cstr)
}

```

可以看到，在 Go 语言中 import "C" 并不是真导入一个包，而只是说这里有 C 代码被插入。相比 Go 语言，Go+ 对 C 的支持更像是把 C 看做 Go+ 原生代码的一部分，而不是外来者。具体体现在：

*   不用写任何 C 代码就可以引用。C 实现的模块如同 Go 实现的模块一样，就是正常的一个 package 而已。import "C" 表示导入 C 语言的标准库（当然不同平台不同 C 编译器的 C 标准库并不完全相同，这个考虑在 gop.mod 中对 C 库这个模块进行版本绑定）。
    
*   引入了 C 的字符串常量语法 C"..."，以增强 C 函数使用上的便捷性。代码中的 C"Hello, world\n" 等价于 []int8{'H', 'e', 'l', 'l', 'o', ',', ' ', 'w', 'o', 'r', 'l', 'd', '\n', '\0'}，是一个以 '\0' 为结尾的 C 字符串。
    

那么对于非 C 标准库，比如类似 sqlite3 这样的知名 C 库，怎么导入？我们设想是这样的：

```
import "C/sqlite3"
sqlite3.XXX(...)

```

那么如果是用户自己实现的 C 库呢？嗯，最通用的形式是这样的：

```
import "C/github.com/foo/bar"
bar.XXX(...)

```

很简洁，和 Go 语言写的库看起来没啥大区别，对吧？

其二，C 与 Go 的类型系统尽可能一致，以降低互操作的成本。在 C 语言中的 void (*)() 到 Go 里面就是 func()，两者属于同一个 type。其他还有 C 语言的 int 类型也是如此。不考虑引入 C.int 这类数据结构，它就等同于 Go 语言的 int。为此 Go+ 还会引入内建的 char 类型，实际上它只是 int8 的别名。

至于要不要引入 union，我估计大概率 Go+ 不支持 union 概念。但是 C 代码中定义的 union 类型，在 Go+ 中可以识别。比如 sqlite3.XXX 如果是一个 union 类型，那就会有正常的 union 行为，包括可以正确访问其中的成员变量。但是我们不允许在 Go+ 源文件中显式定义一个 union 类型。

在 C 语言中，函数会有调用约定（Call Convention）的概念。最常见的调用约定有 cdecl 和 stdcall。其中，cdecl 是 C 函数默认的调用约定，它的好处是支持类似 printf 这种允许有不定参数的函数。而 stdcall 是 Windows API 的调用约定，这个调用方式有时候会被叫做 pascal，这当然是因为它与 Pascal 语言的函数调用约定一致的原因。

这种函数的调用约定的差异在翻译成 Go 代码后会被取消。也就是说，void (cdecl *)() 和 void (stdcall *)() 这两种函数指针类型在 C 语言里面是两个完全不同的类型并且无法相互转换（强制转换后进行函数调用会导致程序崩溃），但是翻译成 Go 后就都是 func()，没有区别。

其三，翻译成 Go 代码后的程序语义，也就是 C 程序员对 C 形成的常规的语义理解都仍然正确。包括：C 字符串以 '\0' 为结尾，C 是手工管理内存的等等，这些都仍然不变。如上面已经提到过的那样，在实现上 C.malloc 会从 Go 进程中划出一片内存来进行手工管理。

最后总结就一句话：我对这个 c2go 项目相当认真。目前它已经放到 github 上 goplus 这个组织里面了，链接如下：

*   https://github.com/goplus/c2go
    

我觉得这事对 Go 未来十年的生态繁荣是至关重要的一项工作，否则 Go 就真的只能偏安在服务端开发领域了。

当然从 Go+ 的角度来说，它是我们在 “面向工程” 领域的目标下，给软件工程师们交出的最重要的一份答卷。