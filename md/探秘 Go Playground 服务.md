> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/_cpSc0e86l18JbIqDoAFfQ)

**什么是 Playground？**  

![](https://mmbiz.qpic.cn/mmbiz_png/Pz37AgkRjPzShm9lnIcN55fOcEssibP79dibjbdOibXWibMzn3KkhFhM3BQ1kkuBxTeibePCc7LqUt8bx8535SLLwOA/640?wx_fmt=png&from=appmsg)

Playground 是一个 Web 服务，允许任何拥有网络浏览器的人编写 Go 代码，可以立即进行编译、链接并在服务器上运行。这是给好奇的程序员一个在安装之前尝试 Go 语言的机会，并给有经验的 Go 用户一个方便的实验场所。

当然，在 Playground 上运行的程序种类有一些限制，我们不能简单地接受任意代码并在我们的服务器上无限制的运行它。这些程序是在一个删减的标准库的沙箱中构建和运行。程序与外界的唯一通信就是通过标准输出，CPU 和内存的使用也有限制。因此，不妨把这看作是对 Go 的世界的一种体验，要获得完整的体验，你需要自己下载 Go 安装包，并在本机上进行开发测试。

**Playground 如何实现？**

我们来了解一下，Playground 初始版本如何实现，以及我们如何与这些服务集成。该实现涉及到可变的操作系统环境变量和运行时。我们假设您对使用 Go 进行的系统编程有所熟悉。

Playground 架构视图：

![](https://mmbiz.qpic.cn/mmbiz_png/Pz37AgkRjPzShm9lnIcN55fOcEssibP79ibp7FWCZewMwDUZClNUhpkRO3icWru85lohOqn4fR0voY5aib4ibMAMCUQ/640?wx_fmt=png&from=appmsg)

Playground 服务由三个部分组成：

1，运行在 Google 服务器上的后端，它接收 RPC 请求，使用 gc 工具链编译用户程序，执行用户程序，并将程序输出（或编译错误）作为 RPC 响应返回。

2，在 Google 应用引擎上运行的前端。它从客户端接收 HTTP 请求，并向后端发出相应的 RPC 请求。它还进行一些缓存。

3，一个 JavaScript 客户端，用于实现用户界面并向前端发出 HTTP 请求。

**后端的实现**

后端程序本身是琐碎的，所以我们不在这里讨论它的实现。有趣的部分是我们如何在安全的环境中安全地执行任意用户代码，同时仍然提供核心功能，如时间、网络和文件系统。

为了将用户程序与谷歌的基础设施隔离开来，后端在 Native Client（或 “NaCl”）下运行这些程序，这是谷歌开发的一种技术，允许在网络浏览器中安全执行 x86 程序。后端使用生成 NaCl 可执行文件的 gc 工具链的特殊版本。（此专用工具链已合并到 Go 1.3 中。要了解更多信息，请阅读设计文档。）

NaCl 限制了程序可能消耗的 CPU 和 RAM 的数量，并阻止程序访问网络或文件系统。然而，这带来了一个问题。Go 的并发性和网络支持是其主要优势之一，对文件系统的访问对许多程序来说至关重要。为了有效地展示并发性，我们需要时间，为了展示网络和文件系统，我们显然需要一个网络和一个文件系统。

尽管所有这些东西今天都得到了支持，但 2010 年推出的第一个版本的游乐场却没有。目前的时间定为 2009 年 11 月 10 日。time.Sleep 没有影响，操作系统和网络包的大多数功能都被截断，返回 EINVALID 错误。

一年前，我们在操场上实施了假时间，这样睡眠程序就能正常工作。最近对游乐场的更新引入了一个假网络堆栈和一个假文件系统，使游乐场的工具链类似于普通 go 工具链。以下各节对这些设施进行了说明。

**伪造时间**

Playground 程序在 CPU 时间和内存使用量方面受到限制，但在实时使用量方面也受到限制。这是因为每个正在运行的程序都会消耗后端的资源以及它与客户端之间的任何有状态基础设施。限制每个游乐场程序的运行时间可以使我们的服务更加可预测，并保护我们免受拒绝服务攻击。

但是，当运行需要时间的代码时，这些限制会变得令人窒息。Go 并发模式讨论通过使用时间等计时函数的示例来演示并发性。像 time.Sleep 和 time.After。当在早期版本的 Playground 上运行时，这些程序的睡眠不会有任何影响，它们的行为也会很奇怪（有时甚至是错误的）。

通过使用一个巧妙的技巧，我们可以让 Go 程序认为它在睡觉，而实际上睡觉根本不需要时间。为了解释这个技巧，我们首先需要了解调度器是如何管理睡眠 goroutine 的。

当一个 goroutine 调用 time.Sleep（或类似）调度器将一个定时器添加到一个挂起的定时器堆中，并使 goroutine 进入睡眠状态。同时，一个特殊的定时器 goroutine 管理这个堆。当定时器 goroutine 启动时，它告诉调度程序在下一个挂起的定时器准备好启动时唤醒它，然后休眠。当它醒来时，它会检查哪些计时器已经过期，唤醒相应的 goroutines，然后返回睡眠。

诀窍是改变唤醒计时器 goroutine 的条件。我们修改调度程序以等待死锁，而不是在特定的时间段后将其唤醒；这时所有的 goroutines 应该是阻塞状态。

Playground 版本的运行时维护自己的内部时钟。当修改后的调度程序检测到死锁时，它会检查是否有任何计时器挂起。如果有，它会将内部时钟提前到最早计时器的触发时间，然后唤醒计时器 goroutine。执行仍在继续，程序认为时间已经过去，而事实上睡眠几乎是瞬间的。这些对调度程序的更改可以在 proc.c 和 time.goc 中找到。

伪造时间解决了后端资源耗尽的问题，但程序输出呢？如果看到一个睡眠程序在不花任何时间的情况下正确运行到完成，那将是很奇怪的。以下程序每秒打印当前时间，然后在三秒后退出。试着运行它。

```
func main() {
    stop := time.After(3 * time.Second)
    tick := time.NewTicker(1 * time.Second)
    defer tick.Stop()
    for {
        select {
        case <-tick.C:
            fmt.Println(time.Now())
        case <-stop:
            return
        }
    }
}

```

这是如何工作的？它是后端、前端和客户端之间的协作。我们捕获每次写入标准输出和标准错误的时间，并将其提供给客户端。然后，客户端可以以正确的时间 “回放” 写入，这样输出就好像程序在本地运行一样。

Playground 的运行时包提供了一个特殊的写入功能，在每次写入之前都包含一个小的 “回放头”。回放头包括魔术串、当前时间和写入数据的长度。具有回放头的写入具有以下结构：

```
0 0 P B <8-byte time> <4-byte data length> <data>

```

上面程序的原始输出如下所示：

```
\x00\x00PB\x11\x74\xef\xed\xe6\xb3\x2a\x00\x00\x00\x00\x1e2009-11-10 23:00:01 +0000 UTC
\x00\x00PB\x11\x74\xef\xee\x22\x4d\xf4\x00\x00\x00\x00\x1e2009-11-10 23:00:02 +0000 UTC
\x00\x00PB\x11\x74\xef\xee\x5d\xe8\xbe\x00\x00\x00\x00\x1e2009-11-10 23:00:03 +0000 UTC

```

前端将此输出解析为一系列事件，并将事件列表作为 JSON 对象返回给客户端：

```
{
    "Errors": "",
    "Events": [
        {
            "Delay": 1000000000,
            "Message": "2009-11-10 23:00:01 +0000 UTC\n"
        },
        {
            "Delay": 1000000000,
            "Message": "2009-11-10 23:00:02 +0000 UTC\n"
        },
        {
            "Delay": 1000000000,
            "Message": "2009-11-10 23:00:03 +0000 UTC\n"
        }
    ]
}

```

JavaScript 客户端（在用户的 web 浏览器中运行）然后使用所提供的延迟间隔播放事件。对用户来说，程序似乎是实时运行的。

**伪造文件系统**

使用 Go 的 NaCl 工具链构建的程序无法访问本地机器的文件系统。相反，系统调用包的文件相关功能（打开、读取、写入等）在由系统调用包本身实现的内存文件系统上运行。由于包 syscall 是 Go 代码和操作系统内核之间的接口，因此用户程序对文件系统的看法与对真实文件系统的理解完全相同。

以下示例程序将数据写入文件，然后将其内容复制到标准输出。试着运行它。（你也可以编辑它！）

```
func main() {
    const filename = "/tmp/file.txt"
    err := ioutil.WriteFile(filename, []byte("Hello, file system\n"), 0644)
    if err != nil {
        log.Fatal(err)
    }
    b, err := ioutil.ReadFile(filename)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("%s", b)
}

```

当进程启动时，文件系统中会填充 / dev / 下的一些设备和一个空的 / tmp 目录。程序可以像往常一样操作文件系统，但当进程退出时，对文件系统的任何更改都会丢失。

还有一项规定是在初始化时将 zip 文件加载到文件系统中（请参阅 unzip _ nacl.go）。到目前为止，我们只使用解压缩功能来提供运行标准库测试所需的数据文件，但我们打算为 Playg 程序提供一组文件，这些文件可用于文档示例、博客文章和 Go Tour。

该实现可以在 fs_nacl.go 和 fd_nacl.go 文件中找到（由于它们的_nacl 后缀，只有当 GOOS 设置为 nacl 时，它们才会内置到包 syscall 中）。文件系统本身由 fsys 结构表示，在初始化时会创建一个全局实例（名为 fs）。然后，各种与文件相关的函数在 fs 上操作，而不是进行实际的系统调用。例如，syscall.Open 函数：

```
func Open(path string, openmode int, perm uint32) (fd int, err error) {
    fs.mu.Lock()
    defer fs.mu.Unlock()
    f, err := fs.open(path, openmode, perm&0777|S_IFREG)
    if err != nil {
        return -1, err
    }
    return newFD(f), nil
}

```

文件描述符由名为 files 的全局切片跟踪。每个文件描述符对应一个文件，每个文件提供一个实现 fileImpl 接口的值。该接口有几种实现方法：

*   常规文件和设备（如 / dev/random）由 fsysFile 表示，
    
*   标准输入、输出和错误是 naclFile 的实例，它使用系统调用与实际文件交互（这些是 Playground 程序与外部世界交互的唯一方式），
    
*   网络套接字有自己的实现，将在下一节中讨论。
    

**伪造网络**

与文件系统一样，Playground 的网络堆栈也是由系统调用包实现的进程中的伪堆栈。它允许 Playground 项目使用环回接口（127.0.0.1）。对其他主机的请求将失败。

例如下面的示例。它在 TCP 端口上侦听，等待传入连接，将数据从该连接复制到标准输出，然后退出。在另一个 goroutine 中，它建立到侦听端口的连接，向连接写入一个字符串，然后关闭它。

```
func main() {
    l, err := net.Listen("tcp", "127.0.0.1:4000")
    if err != nil {
        log.Fatal(err)
    }
    defer l.Close()
    go dial()
    c, err := l.Accept()
    if err != nil {
        log.Fatal(err)
    }
    defer c.Close()
    io.Copy(os.Stdout, c)
}
func dial() {
    c, err := net.Dial("tcp", "127.0.0.1:4000")
    if err != nil {
        log.Fatal(err)
    }
    defer c.Close()
    c.Write([]byte("Hello, network\n"))
}

```

网络接口比文件接口更复杂，因此伪网络的实现比伪文件系统更大、更复杂。它必须模拟读写超时、不同的地址类型和协议等等。该实现可以在 net_nacl.go 中找到。开始阅读的一个好地方是 netFile，它是 fileImpl 接口的网络套接字实现。

**前端**

Playground 前端是另一个简单的程序（短于 100 行）。它从客户端接收 HTTP 请求，向后端发出 RPC 请求，并进行一些缓存。

前端在提供 HTTP 处理程序 https://golang.org/compile. 处理程序需要一个 POST 请求，其中包含一个 body 字段（要运行的 Go 程序）和一个可选的 version 字段（对于大多数客户端，应该是 “2”）。

当前端接收到编译请求时，它首先检查 memcache，看看它是否缓存了该源先前编译的结果。如果找到，则返回缓存的响应。缓存可防止诸如 Go 主页上的程序之类的流行程序使后端过载。如果没有缓存的响应，前端会向后端发出 RPC 请求，将响应存储在 memcache 中，解析播放事件，并将 JSON 对象作为 HTTP 响应返回给客户端（如上所述）。

**客户端**

使用 Playground 的各个站点都共享一些通用的 JavaScript 代码，用于设置用户界面（代码和输出框、运行按钮等）并与 Playground 前端通信。

此实现位于 go.tools 存储库中的 playground.js 文件中，该文件可以从 golang.org/x/tools/godoc/static 包导入。其中有些是干净的，有些是有点粗糙的，因为它是合并了客户端代码的几个不同实现的结果。

Playground 函数接受一些 HTML 元素，并将它们变成一个交互式 Playground 小部件。如果你想把 Playground 放在自己的网站上，你应该使用这个功能。

传输接口（未正式定义，这是 JavaScript）将用户接口从与 web 前端对话的方式中抽象出来。HTTPTransport 是 Transport 的一种实现，它使用前面描述的基于 HTTP 的协议。SocketTransport 是另一个使用 WebSocket 的实现 。

为了遵守同源策略，各种 web 服务器（例如 godoc）代理请求 / 编译到位于的 Playground 服务 https://golang.org/compile. 通用的 golang.org/x/tools/playground 包可以进行代理。

更多内容请参考 golang 官方文档。