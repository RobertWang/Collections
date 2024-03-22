> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/M2l0ZgSsVc7r69eFdTj/article/details/82754587)

![](https://img-blog.csdnimg.cn/img_convert/589b233f46b2713fbe51d5b6556680c9.png)

传统的 Container 由于隔离性差而不适合作为 Sandbox 运行不受信工作负载，VM 可以提供很好隔离但却额外消耗较多的内存。Google 开源的 gVisor 为我们提供另外一种选择：在牺牲掉一定性能的情况下，它只额外消耗非常少量的内存，却可以提供了类似等级的隔离性。在本文里我们深入 gVisor，最后了解一下我们增强 gVisor 以支持资源控制的方案。

gVisor 简介

**gVisor 是什么**

gVisor 为在 Container 中运行不受信代码提供了新的解决思路，gVisor 是一个 Sandbox 方案和实现。

**gVisor 尝试解决什么问题**

虽然 Container 上可以通过 Namespace 和 Cgroup 做资源的限制，但 Container 里的应用程序依然可以访问很多系统资源。事实上跟没有跑在 Container 里的应用程序一样，Container 里的应用程序可以直接通过 Linux 内核的系统调用陷入到内核。任何一个被允许（通过 Seccomp 过滤系统调用）的系统调用的缺陷都可以被恶意的应用程序利用。

主流的 Sandbox 基于 VM 虚拟机的方案，将潜在恶意的应用程序隔离在独立的虚拟机中，例如 Kata Linux，该项目与 Docker 和 Kubernetes 都有集成。基于 VM 的方案提高了很好的隔离，但相应额外消耗的内存会多一些。在有需要运行大量 Container 的场景下的额外资源消耗不能被忽略。

gVisor 提供了另外一种 Sandbox 思路，gVisor 非常轻量级，额外的内存消耗非常小，但同时提供了和 VM 方案相当隔离等级。该分享里介绍的基于 Ptrace 的 gVisor，系统调用的性能比较差，应用程序的兼容性也差一些。gVisor 可以和 Docker 很好的集成，但和 Kubernetes 的集成还处于实验阶段。在和 Docker 集成的时候，gVisor 遵循了 OCI（Open Containers Initiative）标准，所以可以作为 Docker 的一个 Runtime 执行。

gVisor 如何工作

以非特权用户运行的 gVisor 通过截获应用程序的系统调用，将应用程序和内核之间完全隔离。gVisor 没有简单的把应用程序发出的系统调用直接作用到内核，而是实现了大多数的系统调用，通过对系统调用模拟，让应用程序间接的访问到系统资源。gVisor 模拟系统调用本身时对操作系统执行系统调用，通过使用 Seccomp 对这些系统调用做过滤。那么 gVisor 是如何截获应用程序的系统调用的呢？

**gVisor 截获系统调用**

gVisor 存在两种运行模式，这次分享只介绍了基于 Ptrace 的 gVisor。

为了理解 gVisor 如何拦截系统调用，需要先了解一下 Ptrace：Ptrace 是 Linux 提供的一个系统调用接口，通过 Ptrace，可以在两个进程之间建立 Tracer 和 Tracee 之间的关系。Tracer 可以控制 Tracee，例如当 Tracee 收到信号的时候主动进入 stopped 状态，此时 Tracer 可以选择是否对 Tracee 做一些操作（比如设置 Tracee 的寄存器上下文或者内存中内容等），在操作执行后，Tracer 可以选择是否让 Tracee 继续执行。

Tracee 除了可以在接受到信号时候进入 stopped 状态外，也可以被 Tracer 告知在即将进入系统调用时或者即将离开系统调用时进入 stopped 状态。具体说 Ptrace 可以通过 PTRACE_SYSEMU 控制 Tracee 在即将进去系统调动时 stop。gVisor 也正是通过该命令来截获应用程序的系统调用。

Sentry 是通过 Ptrace 来控制应用程序，那么应用程序是如何变为 Tracee 的呢？

**将应用程序变成 Tracee 的**

下图是 gVisor 控制应用程序的进程关系：

![](https://img-blog.csdnimg.cn/img_convert/c47a07cf970dd9766a497ed4b4922725.png)

当 gVisor 以 Docker 的 Runtime 启动的时候，可以看到类似的进程间关系：docker-containerd-shim 是容器的启动器；sentry 是 gVisor 用于截获系统调用模拟内核的程序，他也正是 Tracer。Stub 可以暂时不用理会，stub 的子进程正是我们想要放到 Sandbox 里的应用程序。Sentry 创建 stub，随后 stub 创建应用程序进程，sentry 通过 Ptrace attach 到了 stub 和应用程序上。当应用程序在将要执行系统调用的时候会主动 stop，此时也正是 sentry 拦截和模拟系统调用的点。

这跟用 gdb 调试程序 C/C++ 程序类似，通过命令行给 gdb 指定一个要调试的目标程序的时候，该程序会以子进程的方式运行，gdb 作为 Tracer attach 到应用程序上来对应用程序进行控制。类似 gdb，sentry 也以类似的方式启动应用程序，只是 sentry 先启动了一个 stub，然后让 stub 以它的子进程方式启动了应用程序。

应用程序被创建并变为 Tracee 后，接下来就是 sentry 如何完成应用程序的启动流程了。

**启动应用程序**

应用程序被初始 attach 到 sentry 后，sentry 负责启动应用程序。

在操作系统启动应程序场景里，应用程序的二进制文件由操作系统加载，譬如分配虚拟内存空间用来存放二进制中的代码段、数据段、共享库或者初始化应用程序的栈空间。gVisor 启动应用程序的场景下，类似的过程由 sentry 完成：

为了了解 sentry 是如何初始化应用程序的虚拟内存空间，需要先了解一下上文提到的 stub 进程。

Stub 进程的一个重要的作用是作为应用程序的初始模板，以该模板创建应用程序。事实上 stub 作为 sentry 的子进程，在启动后会主动将虚拟内存地址空间里几乎所有的 memory region（通过查看 / proc/${pid}/maps 查看一个进程虚拟内存地址空间里的所有 memory region）甚至将代码段和数据段也 unmmap 掉了。只保留两个 memory region：

![](https://img-blog.csdnimg.cn/img_convert/49ca6e2bd55af83e9c27cbe921c464b5.png)

其中第一个 region 存放了最简化的代码，即上图中第一个段，执行该代码甚至不需要栈空间，所以连栈段也被 unmap 掉了。

这样的空洞的虚拟内容地址空间正好可以作为一个模板虚拟内存地址空间。当 stub 以子进程的方式启动应用程序后，应用程序的虚拟内存地址空间的 layout 与 stub 的一样。应用程序在创建出来后会立即被 sentry attach，此时正是 sentry 做应用程序初始化的过程：sentry 初始设置应用程序的 RIP（指令执行寄存器）的初始值为应用程序二进制中读出来的应用程序入口地址，该地址一般位于应用程序虚拟地址空间的较低位置，并通过 PTRACE_SYSEMU 指示应用程序开始执行，直到遇到以下两种事件的时候进入 stop 状态：

1.  将要执行一个系统调用
    
2.  收到了来自内核或者进程发给它的信号
    

因为应用程序的初始执行位置在用户态的虚拟内存地址里没有对应的 memory region，所以应用程序会收到来自内核发来的 SIGSEGV 信号（段错误）。这里的场景非常类似通常的 page fault，当一个应用程序试图访问的地址位于某个虚拟内存地址段内，但该段没有对应物理内存页的时候，操作系统会因此陷入 page fault，在 page fault 的 handler 中为该虚拟内存地址段映射物理页。

事实上，在 sentry 在启动应用程序运行环境之前，已经应用程序 “分配” 了一个虚拟内存地址段（这个分配并不是使用 mmap 或者 brk 真正的在应用程序的虚拟地址空间中分配地址段，该分配是一个提前占位）。上面说到当应用程序执行指令地址上因为没有实际分配虚拟内存地址段，所以收到来自内核的 SIGSEGV，并且进入 stopped 状态，此时 sentry 会通过 mmap 在该地址上真实分配一个虚拟地址内存段（类比操作系统为虚拟内存地址段上分配物理页），并且因为 mmap 的源文件是二进制文件本身，所以当 sentry 在处理完 SIGSEGV 指示应用程序继续执行的后，应用程序将实际执行到该二进制中的代码。

至此应用程序就已经启动起来了。接下来需要了解就是 sentry 如何控制应用程序的执行了。

**应用程序的执行**

应用程序被启动起来后，在执行的过程中可能会陆续遇到新的 SIGSEGV（譬如程序读写地址段，或者栈空间的扩展），或者执行系统调用。

在 “应用程序如何被启动” 里实际上已经描述了 SIGSEGV 信号处理的一种场景，即只读地址且有映射文件的场景，其他的场景譬如匿名地址段或者栈空间的区别在于该地址段没有 mmap 实际的文件，而是 mmap 了一个 sentry 提前准备好的 “空白” 文件中。

在 “gVisor 如何拦截系统调用” 中描述了系统调用的拦截，当应用程序在进入系统调用之前会自动进入 stopped 状态，此时 sentry 读取应用程序的系统调用号以及系统调用入参，试图模拟该系统调用。以文件的读 sys_read 为例，sys_read 的作用是找到指定的文件，打开并读取文件内容，并将内存写入到应用程序系统调用参数指定的虚拟内存地址上。Sentry 在接到这个的系统调用时，会将文件读取请求通过 9p 协议发给之前提到的 gofer 进程（sentry 和 gofer 之间有建立 socket pair 传输 9p 协议），由 gofer 进程执行真正的文件读取且将读到的内容通过 9p 协议返回给 sentry。sentry 把读取到的文件内容写入到应用程序的虚拟内存中（如果该地址没有对应的虚拟内存地址段，则分配后再复制），随后 sentry 将系统调用的实际模拟结果写入到应用程序的寄存器中，然后让应用程序继续执行。恢复执行后的应用程序因为得到了系统调用的结果，所以在应用程序在分不清实际上系统调用是直接由操作系统执行了还是由 sentry 做的模拟的情况下，系统调用得到了满足。

**应用程序的访问控制**

“应用程序的执行”中对于文件读系统调用处理的描述实际上也描述了对应用程序文件系统访问的控制，实际上在 “应用程序启动” 中为了省略的根文件系统挂载的描述，在根文件系统挂载的模拟中也涉及到了通过 9p 协议对文件系统的访问。文件写的处理也非常类似。

除了文件读写外，还有很多其他的系统调用，譬如共享内存或其他 IPC，锁，创建线程或者进程，发送信号，socket，execv，epoll，eventfd，pid namespace 等，gVisor 都进行了模拟，涉及到了操作系统的方方面面。这里仅仅介绍了 socket 相关的系统调用：

Sentry 在用户态实现了基本的 TCP/IP 协议栈，在启动应用程序之前，gVisor 会启动一个临时的 start 进程，在 start 进程会进入到 docker 创建的 network namespace，start 进程从该 network namespace 中获取 veth pair 中属于 gVisor 的一端的 veth 设备，创建 AF_PACKET 的 socket 绑定到该 veth 设备上来接管该设备上的网络流量（同时也将 ip 从该设备上去掉了），并将该 socket 传递给 sentry 进程。后续当 sentry 截获了应用程序的 socket 系统调用后，最终通过该 socket 将网络包实际从 veth 设备上发送出去；从该 veth 设备上接收到的网络包在经过 sentry 网络协议栈后递交给应用程序的 socket 层。

gVisor 当前缺少资源限制

gVisor 具备沙盒的能力，但是缺少 Cgroup 提供的资源使用量的限制的功能。在官方的 Roadmap 中计划提供 Cgroup 支持。但此时为了能够使用 gVisor 运行工作负载，需要让 gVisor 具备资源使用量限制的功能。

Docker 通过 Runtime 支持资源使用量限制，gVisor 则是 Docker 的另外一个 Runtime 名叫 runsc。通过了解 Docker 原生的 Runtime 即 runC，可以为 gVisor 中支持 Cgroup 提供思路。Docker 通过 OCI（Open Containers Initiative）spec 跟 Runtime 之间进行交互，符合该标准的 Runtime 可以通过 Docker 的命令来启动。OCI spec 里规范了应用程序启动资源使用量的限制的描述，Docker 在启动 Runtime 的时候，将 OCI spec 内容传递给 Runtime，由 Runtime 负责给应用程序应用这些资源限制：

runC 由 docker shim 启动，首先会创建一个 init 进程，该进程最终会通过 execv 转变为我们希望启动的应用程序，在 init 进程执行 execv 之前，runC 会为 init 进程创建 Cgroup，将实际上 init 进程放入到该 Cgroup 中。此后 init 进程通过 execv 变为应用程序，应用程序以及后来由它创建的子进程也都会进入到该 Cgroup 中，从而达到资源限制的功能。

gVisor 的启动流程中类似也可以嵌入类似的逻辑：runsc 启动流程中首先会创建 gofer 和 boot 进程，在 boot 进程真正启动应用程序之前，runsc 为 boot 进程创建新的 Cgroup，并将 boot 进程放入到该 Cgroup 中，后续 boot 进程以及被它 Ptrace 的应用程序就都会处于该 Cgroup 中，从而达到资源限制的效果。