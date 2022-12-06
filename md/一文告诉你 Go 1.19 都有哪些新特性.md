> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg2MzE5MjMxNQ==&mid=2247510145&idx=1&sn=62b79109a1ad5ffe0b90ff19e64a3df3&chksm=ce7ebd15f90934034edb990a76acb1afb03159a5ce4e7a15867f3babb28e31609b5bbd498c6f&mpshare=1&scene=1&srcid=0721wNsTp0dJb6PopOcswHRZ&sharer_sharetime=1658393238570&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

虽然开发周期少了近一个月，但 Go 1.19 版本仍然会按计划在 2022 年 8 月份发布。而 Go 1.19 的第一个 beta 版也于今天凌晨发布了。Go 1.19 版本都有哪些重要变化呢，我通过这篇文章带大家先睹为快。

> 注 1：版本特性变化以最终发布为准！注 2：本文仅是前瞻，不会过于深入细节。细节待 Go 1.19 正式发布后再聊。

### **泛型问题的 fix**

尽管 Go 核心团队在 Go 1.18 泛型上投入了很多精力，但 Go 1.18 发布后泛型这块依然有已知的天生局限，以及后续逐渐发现的一些问题，而 Go 1.19 版本将继续打磨 Go 泛型，并重点 fix Go 1.18 中发现的泛型问题。目前 Go 1.19 开发版本中大约有 5-6 个泛型问题待解决。之前谈到的可能放开一些泛型约束，在 Go 1.19 估计不会如期兑现了。  

不过可以确定的是 Go 1.19 将包含 Go 语法规范中的一处关于泛型的修正，即由下面表述：

> The scope of an identifier denoting a type parameter of a function or declared by a method receiver is the function body and all parameter lists of the function.(译文：一个用于表示函数的类型参数或由方法接收器声明的类型参数的标识符的作用域范围包括函数体和函数的所有形式参数列表。)

改为下面更新版的表述：

> The scope of an identifier denoting a type parameter of a function or declared by a method receiver starts after the function name and ends at the end of the function body.(译文：一个用于表示函数的类型参数或由方法接收器声明的类型参数的标识符的作用域始于函数名，终止于函数体末尾。)

这样一个改动，使得原本在当前版本 Go 编译器 (Go 1.18.x) 下编译报错的源码，在 Go 1.19 版本中可以正常编译通过：  

### **修订 Go memory model**

Go memory model 是 Go 文档中最抽象的一篇，没有之一！随着 Go 的演进，原先的 Go memory model 描述有很多地方不够正式，也缺少对一些同步机制的说明，如 atomic 等。  

这次修订，参考了 Hans-J. Boehm 和 Sarita V. Adve 在 “Foundations of the C++ Concurrency Memory Model，(PLDI 2008)” 中对 C++ memory model 的描述方式，对 Go memory model 做了更正式的整体描述，增加了对 multiword 竞态、runtime.SetFinalizer、更多 sync 类型、atomic 操作以及编译器优化方面的描述。  

### **修订 go doc comment 格式**

Go 内置了将 comment 直接提取为包文档的能力，这与其他语言通过第三方工具生成文档不同。go doc comment 为 Gopher 提供了很大便利。但 go doc comment 设计于 2009 年，有些过时。对很多呈现形式的支持不够或缺少更为精确的格式描述，这次 Russ Cox 主导了 go doc comment 的修订，增加了对超链、列表、标题、标准库 API 引用等格式支持，修订后的 go doc comment 并非 markdown 语法，但从 markdown 语法中做了借鉴，同时兼容老 comment 格式。下面是 Russ Cox 提供的一些新 doc comment 的渲染后的效果图：  

![](https://mmbiz.qpic.cn/mmbiz_png/cH6WzfQ94maX7M6E4MXLXFicXsSssrhJMrDDkribvdq37XpRiaLPH0OMgpajzUfkgBibQwrujfWbEibsWP3LWITwopg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/cH6WzfQ94maX7M6E4MXLXFicXsSssrhJMJveB9eBTlL18W8DkIwGic16yd8xJ2fDzrIxlLzaxpYH0MOp8gQu0u5Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/cH6WzfQ94maX7M6E4MXLXFicXsSssrhJMqrnvGLiciaR11lyAEMosbVyAiaFkmOCtj7RDYI8EeJacJ5PGDPcqwGGBQ/640?wx_fmt=png)

同时，Go 团队还提供了 go/doc/comment 包，gopher 使用它可以轻松解析 go doc comment。  

### **runtime.SetMemoryLimit**

在 Go1.19 中，一个新的 runtime.SetMemoryLimit 函数以及一个 GOMEMLIMIT 环境变量被引入。有了这个 memory 软限制，Go 运行时将通过限制堆的大小，以及更积极地将内存返回给底层 os，来试图维持这个内存限制，以尽量避免 Go 程序因分配 heap 过多，超出系统内存资源限制而被 kill。  

默认 memory limit 是 math.MaxInt64。一旦通过 SetMemoryLimit 自行设定 limit，那么 Go 运行时将尊重这个 memory limit，通过调整 GC 回收频率以及及时将内存返还给 os 来保证 go 运行时掌控的内存总 size 在 limit 之下。  

注意：limit 限制的是 go runtime 掌控的内存总量，对于开发者自行从 os 申请的内存 (比如通过 mmap) 则不予考虑。limit 的具体措施细节可以参考该 proposal design 文档。  

另外要注意的是：该 limit 不能 100% 消除 out-of-memory 的情况。  

### **Go 1.19 在启动时将默认提高打开文件的限值**

经调查，一些系统对打开的文件数量设置了一个人为的 soft 限制, 主要是为了与使用 select 和其硬编码的最大文件描述符（由 fd_set 的大小限制）的代码兼容。通常限制为 1024，有的更小，比如 256。这样即便是 gofmt 这样的简单程序，当它们并行地遍历一个文件树时，也很容易遇到打开文件描述符超量的错误。  

Go 不使用 select，所以它不应该受这些限制的影响。于是对于导入 os 包的 go 程序，Go 将在 1.19 中默认提高这些限制值到 hard limit。  

### **Go 1.19 race detector 将升级到 v3 版 thread sanitizer**

升级后的新版 race detector 的 race 检测性能相对于上一版将提升 1.5 倍 - 2 倍，内存开销减半，并且没有对 goroutine 的数量的上限限制。

> 注：thread sanitizer 检测数据竞态的工作原理：记录每一个内存访问的信息，并检测线程对这块内存的访问是否存在竞争。基于这种原理，我们也可以知道一旦开启 race detect，Go 程序的执行效率将受到很大影响，运行的开销将大幅增加。v3 版 thread sanitizer 虽然得到了优化，但对程序的总体影响还是存在的并且依旧很大。

### **Go 1.19 增加 "unix" build tag**

Go 1.19 将增加 "unix" 构建标签：  

等价于

```
//go:build aix || linux || darwin || dragonfly || freebsd || openbsd || netbsd || solaris

```

不过要注意，"*_unix.go" 还保留原语义，不能被识别，以便向后兼容现有文件，尤其是 go 标准库之外的使用。  

### **标准库的一些变化**

#### **net 软件包将使用 EDNS**

在 Go 1.19 中，net 软件包将使用 EDNS 来增加 DNS 数据包的大小，以遵守现代 DNS 标准和实现。这应该有助于解决一些 DNS 服务器的问题。  

#### **flag 包增加 TextVar 函数**

Go flag 包增加 TextVar 函数，这样 flag 包便可以与任何实现了 encoding.Text{Marshaler,Unmarshaler} 的 Go 类型集成。比如：  

```
flag.TextVar(&ipaddr, "ipaddr", net.IPv4(192, 168, 0, 1), "what server to connect to?") // 与net.IPv4类型
flag.TextVar(&start, "start", time.Now(), "when should we start processing?") // 与time.Time类型

```

### **其它**

*   在 linux 上，Go 正式支持 64 位龙芯 cpu 架构 (GOOS=linux, GOARCH=loong64)。
    
*   当 Go 程序空闲时，Go GC 进入到周期性的 GC 循环的情况下 (2 分钟一次)，Go 运行时现在会在 idle 的操作系统线程上安排更少的 GC worker goroutine，减少空闲时 Go 应用对 os 资源的占用。
    
*   Go 行时将根据 goroutine 的历史平均栈使用率来分配初始 goroutine 栈，避免了一些 goroutine 的最多 2 倍的 goroutine 栈空间浪费。
    
*   sync/atomic 包增加了新的高级原子类型 Bool, Int32, Int64, Uint32, Uint64, Uintptr 和 Pointer，提升了使用体验。
    
*   Go 1.19 中 Go 编译器使用 jump table 重新实现了针对大整型数和 string 类型的 switch 语句，平均性能提升 20% 左右。
    

**小结**
------

相对于 Go 1.18，Go 1.19 的确是一个 “小版本”。但 Go 1.19 对 memory model 的更新、SetMemoryLimit 的加入、go doc comment 的修订以及对 go runtime 的持续打磨依然可以让 gopher 们产生一丝丝 “小兴奋”，尤其是 SetMemoryLimit 的加入，是否能改善 Go 应用因 GC 不及时被 kill 的情况呢，让我们拭目以待。