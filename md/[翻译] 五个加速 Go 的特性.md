> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/sKMlczFAKVdybUHZwSNkpw)

1 译者说




===========

本文原文：

https://dave.cheney.net/2014/06/07/five-things-that-make-go-fast

2 译文




==========

大家下午好。

我叫 David。

我很高兴今天能来到 Gocon。我想参加这个会议已经两年了，我很感谢主办方能提供给我向你们演讲的机会。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBlJILibxVDXIrTralebaINwRGKOvDTeVGoeezts1Zcq5eo2GOYhFg5OQ/640?wx_fmt=jpeg)

我想以一个问题开始我的演讲。  

为什么选择 Go？

当大家讨论学习或在生产环境中使用 Go 的原因时，答案不一而足，但因为以下三个原因的最多。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBAicjaia20YibEzrbMajNtWpcuZmzgmBD1aJwonLdicmXeBsuYBv0S6fXibA/640?wx_fmt=jpeg)

这就是 TOP3 的原因。  

第一，并发。

Go 的 并发原语 Concurrency Primitives 对于来自 Nodejs，Ruby 或 Python 等单线程脚本语言的程序员，或者来自 C++ 或 Java 等重量级线程模型的语言都很有吸引力。

易于部署。

我们今天从经验丰富的 Gophers 那里听说过，他们非常欣赏部署 Go 应用的简单性。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBPia1CEtbMQLkRltB2cR3h0DGp3oAQGlkAkCdB2QPJQoQEdwNe4J6icGg/640?wx_fmt=jpeg)

然后是性能。  

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBPRhLoMVGhJhPKibLp2mRkgiaGFTLph8PhODSjYq3Xic2dyG4kYAUzAPicg/640?wx_fmt=jpeg)

这是 Go 中一个值的例子。编译时，`gocon` 正好消耗四个字节的内存。

让我们将 Go 与其他一些语言进行比较

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBk6nyK0dscEaoOdoWsVtE4kSI3ypYtHiakZwWMTa03Ry8M3ia3FBmvuOg/640?wx_fmt=jpeg)

由于 Python 表示变量的方式的开销，使用 Python 存储相同的值会消耗六倍的内存。

Python 使用额外的内存来跟踪类型信息，进行 引用计数 Reference Counting 等。

让我们看另一个例子：

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBcbdHia1n9OiaicTNsztvJqLReRSMDJ5ZsJDia5wIL4CJWdfk3LSKnJjSmw/640?wx_fmt=jpeg)

与 Go 类似，Java 消耗 4 个字节的内存来存储 `int` 型。  

但是，要在像 `List` 或 `Map` 这样的集合中使用此值，编译器必须将其转换为 `Integer` 对象。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBTee3hTcRGoKEk81NH1aNRMNpedbJzKvia0NKadHN9TYIEt90j3yxGcg/640?wx_fmt=jpeg)

因此，Java 中的整数通常消耗 16 到 24 个字节的内存。  

为什么这很重要？ 内存便宜且充足，为什么这个开销很重要？

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBRNibypNg0PMz7qsMjP0q5YzMDzahodUHcsLUnoFgzZctuEXoRibDXmCg/640?wx_fmt=jpeg)

这是一张显示 CPU 时钟速度与内存总线速度的图表。  

请注意 CPU 时钟速度和内存总线速度之间的差距如何继续扩大。

两者之间的差异实际上是 CPU 花费多少时间等待内存。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBuds1KXDn0Fsia9eLEOcUIWeke0rI4Y2ZdibThOzia7qfvHQ25IYKEjuWw/640?wx_fmt=jpeg)

自 1960 年代后期以来，CPU 设计师已经意识到了这个问题。  

他们的解决方案是一个缓存，一个更小、更快的内存区域，介入 CPU 和主存之间。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBKcQJ79iackyNmW6spIOHicrZze3iaY64cIKuaDktTus6hID2HuiclR00xA/640?wx_fmt=jpeg)

这是一个 `Location` 类型，它保存物体在三维空间中的位置。它是用 Go 编写的，因此每个 `Location`只消耗 24 个字节的存储空间。  

我们可以使用这种类型来构造一个容纳 1000 个 `Location` 的数组类型，它只消耗 24000 字节的内存。

在数组内部，`Location` 结构体是顺序存储的，而不是随机存储的 1000 个 `Location` 结构体的指针。

这很重要，因为现在所有 1000 个 `Location` 结构体都按顺序放在缓存中，紧密排列在一起。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqB7DeZJNMJqGrC6kdh9tolTlCIVotHBtt7libO10VhHH7R3KrkeHFfIAg/640?wx_fmt=jpeg)

Go 允许您创建紧凑的数据结构，避免不必要的填充字节。  

紧凑的数据结构能更好地利用缓存。

更好的缓存利用率可带来更好的性能。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBibicytyibcpEk5bypLNBoLAEBliabcibzxFRDUJ6BicCSEVX8MpicXo7vKUdQ/640?wx_fmt=jpeg)

函数调用不是无开销的。  

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBseYThMvgibJAZROB66d7QdX43BoxFxTmMbAeuLzPSycKqNeUiaCgGy8w/640?wx_fmt=jpeg)

调用函数时会发生三件事。

创建一个新的 栈帧 Stack Frame，并记录调用者的详细信息。

在函数调用期间可能被覆盖的任何寄存器都将保存到栈中。

处理器计算函数的地址并执行到该新地址的分支。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBx77lLNiccSFCwLtbCLMdMCL5rRFZ8ziaxwzmkVbhqeDGticGxPZKfcNLg/640?wx_fmt=jpeg)

由于函数调用是非常常见的操作，因此 CPU 设计师一直在努力优化此过程，但他们无法消除开销。  

函调固有开销，或重于泰山，或轻于鸿毛，这取决于函数做了什么。

减少函数调用开销的解决方案是 内联 Inlining。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBibnHCRNOIticJplt0UODp8bGia06icWWib5Gz16CZp1vGOo9CrnBTnicHp8w/640?wx_fmt=jpeg)

Go 编译器通过将函数体视为调用者的一部分来内联函数。  

内联也有成本，它增加了二进制文件大小。

只有当调用开销与函数所做工作关联度的很大时内联才有意义，因此只有简单的函数才能用于内联。

复杂的函数通常不受调用它们的开销所支配，因此不会内联。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBPwHSMpL6YcMvyUjWLPsse9J3jOcHkybqBpjZIBfNPhao6pozsr5ktw/640?wx_fmt=jpeg)

这个例子显示函数 `Double` 调用 `util.Max`。  

为了减少调用 `util.Max` 的开销，编译器可以将 `util.Max` 内联到 `Double` 中，就象这样：

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBsFMXxrwm33dTG5mjibHQDwSzs1WbqXXT5pE7uFs34icvjPEialkl70tDA/640?wx_fmt=jpeg)

内联后不再调用 `util.Max`，但是 `Double` 的行为没有改变。  

内联并不是 Go 独有的。几乎每种编译或及时编译的语言都执行此优化。但是 Go 的内联是如何实现的？

Go 实现非常简单。编译包时，会标记任何适合内联的小函数，然后照常编译。

然后函数的源代码和编译后版本都会被存储。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBAibePicQpbGoDmSmD1VcTPM4gIH1rpc30zjZicdrADibJ1Gbia9OZYntMyA/640?wx_fmt=jpeg)

此幻灯片显示了 `util.a` 的内容。源代码已经过一些转换，以便编译器更容易快速处理。  

当编译器编译 `Double` 时，它看到 `util.Max` 可内联的，并且 `util.Max` 的源代码是可用的。

就会替换原函数中的代码，而不是插入对 `util.Max` 的编译版本的调用。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBoz9CpQnuaAiacHybbKPqNBbReaQ5bPhlsKnNYegDlCJTtU0avicPE5cg/640?wx_fmt=jpeg)

在这个例子中，尽管函数 `Test` 总是返回 `false`，但 `Expensive` 在不执行它的情况下无法知道结果。  

当 `Test` 被内联时，我们得到这样的东西。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqByrpSRzULQiaOuJyDk8m5IYTeV3q1FsPzJob3n7picHLrc5RgrqhEpILA/640?wx_fmt=jpeg)

编译器现在知道 `Expensive` 的代码无法访问。  

这不仅节省了调用 `Test` 的成本，还节省了编译或运行任何现在无法访问的 `Expensive` 代码。

Go 编译器可以跨文件甚至跨包自动内联函数。还包括从标准库调用的可内联函数的代码。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBLQwwR00mtMdwsk6fhFNerkMEcl49t7FD8Zv1R40fsRZIjNVCBKn57Q/640?wx_fmt=jpeg)

强制垃圾回收 Mandatory Garbage Collection 使 Go 成为一种更简单，更安全的语言。

这并不意味着垃圾回收会使 Go 变慢，或者垃圾回收是程序速度的瓶颈。

这意味着在堆上分配的内存是有代价的。每次 GC 运行时都会花费 CPU 时间，直到释放内存为止。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBOHY8ha7XoBujgGdBgTjdl1QOMTAicEe3vpvlCqw0tZPBAF9hpnly6qg/640?wx_fmt=jpeg)

然而，有另一个地方分配内存，那就是栈。  

与 C 不同，它强制您选择是否将值通过 `malloc` 将其存储在堆上，还是通过在函数范围内声明将其储存在栈上；Go 实现了一个名为 逃逸分析 Escape Analysis 的优化。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBNjibQfmzXOicJrerQNeP2WXaVzyyz87JiaugianZwxsQm2E3CzmooxicTFQ/640?wx_fmt=jpeg)

逃逸分析决定了对一个值的任何引用是否会从被声明的函数中逃逸。  

如果没有引用逃逸，则该值可以安全地存储在栈中。

存储在栈中的值不需要分配或释放。

让我们看一些例子

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqB6EialB1iczQHm8XwiaBlNAOmnuicub3JQ0foYu8dv8RwSnXRUZVBmQwXPg/640?wx_fmt=jpeg)

`Sum` 返回 1 到 100 的整数的和。这是一种相当不寻常的做法，但它说明了逃逸分析的工作原理。

因为切片 `numbers` 仅在 `Sum` 内引用，所以编译器将安排到栈上来存储的 100 个整数，而不是安排到堆上。

没有必要回收 `numbers`，它会在 `Sum` 返回时自动释放。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBP6ib3FibWk2HIdy9kPicu1dFQIa1l5KgnFnZQzFRaeVaCkxEUj0meK06A/640?wx_fmt=jpeg)

第二个例子也有点尬。

在 `CenterCursor` 中，我们创建一个新的 `Cursor` 对象并在 `c` 中存储指向它的指针。  

然后我们将 `c` 传递给 `Center()` 函数，它将 `Cursor` 移动到屏幕的中心。

最后我们打印出那个'Cursor` 的 X 和 Y 坐标。

即使 `c` 被 `new` 函数分配了空间，它也不会存储在堆上，因为没有引用 `c` 的变量逃逸 `CenterCursor`函数。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBkLXqnmaibsGTMGHibNyRa2eXbHHxJZosJGRLLnkRibfhgAogr7DyN4OvA/640?wx_fmt=jpeg)

默认情况下，Go 的优化始终处于启用状态。可以使用 `-gcflags = -m` 开关查看编译器的逃逸分析和内联决策。  

因为逃逸分析是在编译时执行的，而不是运行时，所以无论垃圾回收的效率如何，栈分配总是比堆分配快。

我将在本演讲的其余部分详细讨论栈。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBsQZnXibDic8AX78D4X1z0NQL9IzraicjTobeibU1gk7fhuWbUIHxyjq8Mg/640?wx_fmt=jpeg)

Go 有 goroutine。 这是 Go 并发的基石。  

我想退一步，探索 goroutine 的历史。

最初，计算机一次运行一个进程。在 60 年代，多进程或 分时 Time Sharing 的想法变得流行起来。

在分时系统中，操作系统必须通过保护当前进程的现场，然后恢复另一个进程的现场，不断地在这些进程之间切换 CPU 的注意力。

这称为**进程切换**。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBUALL29fMDBLTUFLC92JMuL9EXLEq0w4ic2DXSXNAXO5AwJ7RI686MCA/640?wx_fmt=jpeg)

进程切换有三个主要开销。  

首先，内核需要保护该进程的所有 CPU 寄存器的现场，然后恢复另一个进程的现场。

内核还需要将 CPU 的映射从虚拟内存刷新到物理内存，因为这些映射仅对当前进程有效。

最后是操作系统 上下文切换 Context Switch 的成本，以及 调度函数 Scheduler Function 选择占用 CPU 的下一个进程的开销。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBfD2PGvlTLQhxibL1ianYXj89PgajpxbHIwd2JNpIbTmLeR3z6e76V6og/640?wx_fmt=jpeg)

现代处理器中有数量惊人的寄存器。我很难在一张幻灯片上排开它们，这可以让你知道保护和恢复它们需要多少时间。  

由于进程切换可以在进程执行的任何时刻发生，因此操作系统需要存储所有寄存器的内容，因为它不知道当前正在使用哪些寄存器。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBpHwwwIls3bx4chGe8lZDmmMGcnNiaZqpQekgGbOUxKrBI27xiazWr66g/640?wx_fmt=jpeg)

这导致了线程的出生，这些线程在概念上与进程相同，但共享相同的内存空间。  

由于线程共享地址空间，因此它们比进程更轻，因此创建速度更快，切换速度更快。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBWPdgxPibBP85zEB8VZ9okngviaich59iamtl1vAc2uVEoy9elRT92YZKRw/640?wx_fmt=jpeg)

Goroutine 升华了线程的思想。  

Goroutine 是 协作式调度 Cooperative Scheduled 的，而不是依靠内核来调度。

当对 Go 运行时调度器 Runtime Scheduler 进行显式调用时，goroutine 之间的切换仅发生在明确定义的点上。

编译器知道正在使用的寄存器并自动保存它们。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBpgSRlqkn0aUibuo9TohrNGUHTOx4UQxV5vCMhnvlTFiakFkzCX4ZA9kw/640?wx_fmt=jpeg)

虽然 goroutine 是协作式调度的，但运行时会为你处理。  

Goroutine 可能会给禅让给其他协程时刻是：

*   阻塞式通道发送和接收。
    
*   Go 声明，虽然不能保证会立即调度新的 goroutine。
    
*   文件和网络操作式的阻塞式系统调用。
    
*   在被垃圾回收循环停止后。
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBwyaBYkMDA8OmFyiaqxAG1nQ5gC6MJB914Oestprr1aBxzt4KNJUEPEw/640?wx_fmt=jpeg)

这个例子说明了上一张幻灯片中描述的一些调度点。

箭头所示的线程从左侧的 `ReadFile` 函数开始。遇到 `os.Open`，它在等待文件操作完成时阻塞线程，因此调度器将线程切换到右侧的 goroutine。

继续执行直到从通道 `c` 中读，并且此时 `os.Open` 调用已完成，因此调度器将线程切换回左侧并继续执行 `file.Read` 函数，然后又被文件 IO 阻塞。

调度器将线程切换回右侧以进行另一个通道操作，该操作在左侧运行期间已解锁，但在通道发送时再次阻塞。

最后，当 `Read` 操作完成并且数据可用时，线程切换回左侧。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBrUDODZvjeQMYoa3YHCMb6gmFHPga4tiaz14xFFu0Ppnb4wUHthWnEVg/640?wx_fmt=jpeg)

这张幻灯片显示了低级语言描述的 `runtime.Syscall` 函数，它是 `os` 包中所有函数的基础。  

只要你的代码调用操作系统，就会通过此函数。

对 `entersyscall` 的调用通知运行时该线程即将阻塞。

这允许运行时启动一个新线程，该线程将在当前线程被阻塞时为其他 goroutine 提供服务。

这导致每 Go 进程的操作系统线程相对较少，Go 运行时负责将可运行的 Goroutine 分配给空闲的操作系统线程。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBUf3U1rX8j3gANfQrtTZwJ98IvcI2Miaf3yBx7ibVg031WtDKNaLh4t5Q/640?wx_fmt=jpeg)

在上一节中，我讨论了 goroutine 如何减少管理许多（有时是数十万个并发执行线程）的开销。  

Goroutine 故事还有另一面，那就是栈管理，它引导我进入我的最后一个话题。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBspNjcujm2zentzlyzk8y1ruaLnAq35CWicmvaleQwqf7AXLabZUOjEA/640?wx_fmt=jpeg)

这是一个进程的内存布局图。我们感兴趣的关键是堆和栈的位置。  

传统上，在进程的地址空间内，堆位于内存的底部，位于程序（代码）的上方并向上增长。

栈位于虚拟地址空间的顶部，并向下增长。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqB9BoJDEZqmXWpItTxo4mcVXvQkdtG7VYAOr98ETEHOictzFuASyia2GWA/640?wx_fmt=jpeg)

因为堆和栈相互覆盖的结果会是灾难性的，操作系统通常会安排在栈和堆之间放置一个不可写内存区域，以确保如果它们发生碰撞，程序将中止。  

这称为保护页，有效地限制了进程的栈大小，通常大约为几兆字节。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBtJMYu3LnCs9YGTh9TPTrpGFoztZpTkUSXthzDPSexMqt08QCxic3Wdw/640?wx_fmt=jpeg)

我们已经讨论过线程共享相同的地址空间，因此对于每个线程，它必须有自己的栈。  

由于很难预测特定线程的栈需求，因此为每个线程的栈和保护页面保留了大量内存。

希望是这些区域永远不被使用，而且防护页永远不会被击中。

缺点是随着程序中线程数的增加，可用地址空间的数量会减少。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqB9sZqcnqwyeW5sv1z7mWiaBrGtcKlJmRTYfLwGQUyia7EtdpAjVygUicSQ/640?wx_fmt=jpeg)

我们已经看到 Go 运行时将大量的 goroutine 调度到少量线程上，但那些 goroutines 的栈需求呢？  

Go 编译器不使用保护页，而是在每个函数调用时插入一个检查，以检查是否有足够的栈来运行该函数。如果没有，运行时可以分配更多的栈空间。

由于这种检查，goroutines 初始栈可以做得更小，这反过来允许 Go 程序员将 goroutines 视为廉价资源。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBe7dnuAr0NBPjlUs0nUb9eteWJnbfO2tCxCmA2SW2taEYdA6bdVsBdw/640?wx_fmt=jpeg)

这是一张显示了 Go 1.2 如何管理栈的幻灯片。  

当 `G` 调用 `H` 时，没有足够的空间让 `H` 运行，所以运行时从堆中分配一个新的栈帧，然后在新的栈段上运行 `H`。当 `H` 返回时，栈区域返回到堆，然后返回到 `G`。

![](https://mmbiz.qpic.cn/mmbiz_png/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBnGI7dQGgk0PVDdcNrsgAJ874IYSiaLmPdQqz1icdezmgf92NXEeN80XQ/640?wx_fmt=png)

这种管理栈的方法通常很好用，但对于某些类型的代码，通常是递归代码，它可能导致程序的内部循环跨越这些栈边界之一。  

例如，在程序的内部循环中，函数 `G` 可以在循环中多次调用 `H`，

每次都会导致栈拆分。 这被称为 热分裂 Hot Split 问题。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqB5xWaTic2fictK04SYvMHEpqPibIiclh50AdCZiaBStjGpo6EVfG8bFUQoXw/640?wx_fmt=jpeg)

为了解决热分裂问题，Go 1.3 采用了一种新的栈管理方法。  

如果 goroutine 的栈太小，则不会添加和删除其他栈段，而是分配新的更大的栈。

旧栈的内容被复制到新栈，然后 goroutine 使用新的更大的栈继续运行。

在第一次调用 `H` 之后，栈将足够大，对可用栈空间的检查将始终成功。

这解决了热分裂问题。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBDgA3znlx6UXSkIsFgWeP9X9rLEBf2RCpGicGSeHptVt2Lc7vhlVJ3ow/640?wx_fmt=jpeg)

值，内联，逃逸分析，Goroutines 和分段 / 复制栈。  

这些是我今天选择谈论的五个特性，但它们绝不是使 Go 成为快速的语言的唯一因素，就像人们引用他们学习 Go 的理由的三个原因一样。

这五个特性一样强大，它们不是孤立存在的。

例如，运行时将 goroutine 复用到线程上的方式在没有可扩展栈的情况下几乎没有效率。

内联通过将较小的函数组合成较大的函数来降低栈大小检查的成本。

逃逸分析通过自动将从实例从堆移动到栈来减少垃圾回收器的压力。

逃逸分析还提供了更好的 缓存局部性 Cache Locality。

如果没有可增长的栈，逃逸分析可能会对栈施加太大的压力。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Rcon9f6LyEsG7S6Kwg3zJfkvcoKicuXqBn7ib97yj0vtCO26KVPrsI12YPnKbBVicRy3mlMaeIXia66tKr5wwLgbibA/640?wx_fmt=jpeg)

感谢 Gocon 主办方允许我今天发言  

*   twitter / web / email details
    
*   感谢 @offbymany，@billkennedy_go 和 Minux 在准备这个演讲的过程中所提供的帮助。
    

3 相关文章




============

**1. 听我在 OSCON 上关于 Go 性能的演讲**  

（https://dave.cheney.net/2015/05/31/hear-me-speak-about-go-performance-at-oscon）

**2. 为什么 Goroutine 的栈是无限大的？**

（https://dave.cheney.net/2013/06/02/why-is-a-goroutines-stack-infinite）

**3.Go 的运行时环境变量的旋风之旅**

（https://dave.cheney.net/2015/11/29/a-whirlwind-tour-of-gos-runtime-environment-variables）

**4. 没有事件循环的性能**

（https://dave.cheney.net/2015/08/08/performance-without-the-event-loop）

4 作者介绍




============

David 是来自澳大利亚悉尼的程序员和作者。  

自 2011 年 2 月起成为 Go 的 contributor，自 2012 年 4 月起成为 committer。

联系信息

*   dave@cheney.net
    
*   twitter: @davecheney
    
*   GitHub: @davechene
    

![](https://mmbiz.qpic.cn/mmbiz_gif/Rcon9f6LyEtnuNrHVVOILKpt8FhmlfzgLrBGIvxlEYWR60LLyciayjG8DNIzcu5G9UxZzvFf6VAu8BxPicIdbcYg/640?wx_fmt=gif)