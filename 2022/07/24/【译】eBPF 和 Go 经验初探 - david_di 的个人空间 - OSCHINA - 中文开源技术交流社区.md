> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [my.oschina.net](https://my.oschina.net/daviddi/blog/5217071)

> 原文地址：https://networkop.co.uk/post/2021-03-ebpf-intro/ 首发地址： 【译】eBPF 和 Go 经验初探 本站相关文档：使用 Go 语言管理和分发 ebpf 程序 1. 前言 eBPF 的生态欣欣向荣，无论是 eBPF 本身及其各种应用（包括 XDP） 方面都有大量的学习资源。

原文地址：[https://networkop.co.uk/post/2021-03-ebpf-intro/](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fnetworkop.co.uk%2Fpost%2F2021-03-ebpf-intro%2F)  
首发地址： [【译】eBPF 和 Go 经验初探](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.ebpf.top%2Fpost%2Febpf_go_translation%2F)  
本站相关文档：[使用 Go 语言管理和分发 ebpf 程序](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.ebpf.top%2Fpost%2Febpf_go%2F)

1. 前言
-----

eBPF 的生态欣欣向荣，无论是 [eBPF 本身](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Febpf.io%2Fwhat-is-ebpf%2F)及其各种应用（包括 [XDP](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fxdp-project%2Fxdp-tutorial)） 方面都有大量的学习资源。但当涉及到选择库和工具来与 eBPF 进行交互时，会让人有所困惑。在选择时，你必须在基于 Python 的 [BCC](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fiovisor%2Fbcc) 框架、基于 C 的 [libbpf](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Flibbpf%2Flibbpf) 和一系列基于 Go 的 [Dropbox](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fdropbox%2Fgoebpf)、[Cilium](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fcilium%2Febpf)、[Aqua](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Faquasecurity%2Ftracee%2Ftree%2Fmain%2Flibbpfgo) 和 [Calico](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fprojectcalico%2Ffelix%2Ftree%2Fmaster%2Fbpf) 等库中选择。另一个经常被忽视的重要领域是 eBPF 代码的 "生产化"，即从手动编写的样例到生产级应用（例如 Cilium）。在本篇文章中，我将记录相关的经验，特别是在网络（XDP）应用程序场景中，使用 Go 编写的用户空间控制程序。

2. 选择 eBPF 库
------------

在大多数情况下，eBPF 库主要协助实现两个功能：

*   **将 eBPF 程序和 Map** 载入内核并执行[重定位](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fkinvolk.io%2Fblog%2F2018%2F10%2Fexploring-bpf-elf-loaders-at-the-bpf-hackfest%2F%23common-steps)，通过其文件描述符将 eBPF 程序与正确的 Map 进行关联。
    
*   **与 eBPF Map 交互**，允许对存储在 Map 中的键 / 值对进行标准的 CRUD 操作。
    

部分库也可以帮助你将 eBPF 程序附加到一个特定的[钩子](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Febpf.io%2Fwhat-is-ebpf%2F%23hook-overview)，尽管对于网络场景下，这可能很容易采用现有的 netlink API 库完成。

当涉及到 eBPF 库的选择时，我并不是唯一感到困惑的人（见 [[1]](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Ftwitter.com%2Fmaurovasquezb%2Fstatus%2F1146438190062063616), [[2]](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Ftwitter.com%2Fqeole%2Fstatus%2F1364521385138282497)）。事实是每个库都有各自的范围和限制。

*   [Calico](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fpkg.go.dev%2Fgithub.com%2Fprojectcalico%2Ffelix%40v3.8.9%2Bincompatible%2Fbpf) 在用 [bpftool](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Ftwitter.com%2Fqeole%2Fstatus%2F1101450782841466880) 和 iproute2 实现的 CLI 命令基础上实现了一个 Go 包装器。
*   [Aqua](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Faquasecurity%2Ftracee%2Ftree%2Fmain%2Ftracee-ebpf) 实现了对 libbpf C 库的 Go 包装器。
*   [Dropbox](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fdropbox%2Fgoebpf) 支持一小部分程序，但有一个非常干净和方便的用户 API。
*   IO Visor 的 [gobpf](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fiovisor%2Fgobpf) 是 BCC 框架的 Go 语言绑定，它更注重于跟踪和性能分析。
*   [Cilium 和 Cloudflare](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fcilium%2Febpf) 维护一个 [纯 Go 语言编写的库](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Flinuxplumbersconf.org%2Fevent%2F4%2Fcontributions%2F449%2Fattachments%2F239%2F529%2FA_pure_Go_eBPF_library.pdf) (以下简称 "libbpf-go")，它将所有 eBPF 系统调用抽象在一个本地 Go 接口后面。

基于我的网络特定用例，我最终选择了 `libbpf-go`，因为其被 Cilium 和 Cloudflare 使用，并且有一个活跃的社区，尽管我也非常喜欢简单易用的 Dropbox 库，并且也可以使用它。

为了熟悉开发过程，我决定实现一个 XDP 交叉连接的应用，它在网络拓扑模拟方面有一个非常小众但重要的[用例](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fnetdevops.me%2F2021%2Ftransparently-redirecting-packets%2Fframes-between-interfaces%2F)。我们的目标是要有一个应用程序来观察一个配置文件，并确保本地接口根据该文件的 YAML 规范进行互连。下面是对 [`xdp-xconnect`](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fnetworkop%2Fxdp-xconnect) 工作高层次概述。

![](https://www.ebpf.top/post/ebpf_go_translation/imgs/xdp-xconnect.png)

下面的章节将逐步描述应用的构建和交付过程，更多的是关注集成，而不是实际的代码。`xdp-xconnect` 的完整代码在 Github 上[可用](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fnetworkop%2Fxdp-xconnect)。

3. 步骤 1 - 编写 eBPF 代码
--------------------

通常情况下，这将是任何 "eBPF 入门" 文章的主要部分，然而这一次它并不是重点。我并不认为自己可以帮助别人学习如何编写 eBPF，然而，我可以参考一些非常好的资源。

*   通用的 eBPF 理论在网站 [ebpf.io](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Febpf.io%2Fwhat-is-ebpf%2F) 和 Cilium 的 eBPF 和 XDP [参考指南](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fdocs.cilium.io%2Fen%2Fstable%2Fbpf%2F)中有大量的细节。
*   对 eBPF 和 XDP 进行实践的最好地方是 [xdp-tutorial](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fxdp-project%2Fxdp-tutorial)。这是一个了不起的资源，即使你最终选择不完成作业，也绝对值得阅读。
*   Cilium 的[源代码](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fcilium%2Fcilium%2Ftree%2Fmaster%2Fbpf)和其在 [[1]](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fk8s.networkop.co.uk%2Fcni%2Fcilium%2F%23a-day-in-the-life-of-a-packet) 和 [[2]](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Farthurchiao.art%2Fblog%2Fcilium-life-of-a-packet-pod-to-service%2F) 的分析。

我的 eBPF 程序非常简单，它包括对 eBPF [帮助函数](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fman7.org%2Flinux%2Fman-pages%2Fman7%2Fbpf-helpers.7.html)的一次调用，可根据传入接口的索引将_所有_数据包从一个接口重定向到另一个。

```
#include <linux/bpf.h>
#include <bpf/bpf_helpers.h>

SEC("xdp")
int  xdp_xconnect(struct xdp_md *ctx)
{
    return bpf_redirect_map(&xconnect_map, ctx->ingress_ifindex, 0);
}


```

为了编译上述程序，我们需要为所有包含的头文件提供包含路径。最简单的方法是在 [linux/tools/lib/bpf/](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgit.kernel.org%2Fpub%2Fscm%2Flinux%2Fkernel%2Fgit%2Fnetdev%2Fnet-next.git%2Ftree%2Ftools%2Flib%2Fbpf) 下复制所有文件，然而，这将包括很多不必要的文件。因此，另一种方法是创建一个依赖性列表。

```
$ clang -MD -MF xconnect.d -target bpf -I ~/linux/tools/lib/bpf -c xconnect.c


```

现在我们可以只对 `xconnect.d` 中指定的少量文件进行本地拷贝，并使用以下命令为本地 CPU 架构编译 eBPF 代码。

```
$ clang -target bpf -Wall -O2 -emit-llvm -g -Iinclude -c xconnect.c -o - | \
llc -march=bpf -mcpu=probe -filetype=obj -o xconnect.o


```

The resulting ELF file is what we’d need to provide to our Go library in the next step.

编译生成的 ELF 文件就是我们在下一步需要提供给 Go 库的程序。

4. 步骤 2 - 编写 Go 代码
------------------

编译好的 eBPF 程序和 Map 可以通过 `libbpf-go` 加载，这只需几个指令。通过添加带有 `ebpf` 标签的结构，我们可以自动进行重定位程序，并且知道何处发现 Map。

```
spec, err := ebpf.LoadCollectionSpec("ebpf/xconnect.o")
if err != nil {
  panic(err)
}

var objs struct {
	XCProg  *ebpf.Program `ebpf:"xdp_xconnect"`
	XCMap   *ebpf.Map     `ebpf:"xconnect_map"`
}
if err := spec.LoadAndAssign(&objs, nil); err != nil {
	panic(err)
}
defer objs.XCProg.Close()
defer objs.XCMap.Close()


```

`ebpf.Map` 类型有一组方法，可对加载的 Map 内容进行标准的 CRUD 操作：

```
err = objs.XCMap.Put(uint32(0), uint32(10))

var v0 uint32
err = objs.XCMap.Lookup(uint32(0), &v0)

err = objs.XCMap.Delete(uint32(0))


```

唯一没有被 `libbpf-go` 包含的步骤是将程序附加到网络钩子上。然而，这可以通过任何现有的 netlink 库轻松实现，例如 [vishvananda/netlink](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fvishvananda%2Fnetlink)，通过将网络连接与加载程序的文件描述符联系起来：

```
link, err := netlink.LinkByName("eth0")
err = netlink.LinkSetXdpFdWithFlags(*link, c.objs.XCProg.FD(), 2)


```

请注意，我使用 [SKB_MODE](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Ftorvalds%2Flinux%2Fblob%2Fmaster%2Ftools%2Finclude%2Fuapi%2Flinux%2Fif_link.h%23L966) XDP 标志来绕过退出的 veth 驱动程序 [caveat](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fxdp-project%2Fxdp-tutorial%2Ftree%2Fmaster%2Fpacket03-redirecting%23sending-packets-back-to-the-interface-they-came-from) 。尽管本地 XDP 模式比任何其他 eBPF 钩子[快得多](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.netronome.com%2Fmedia%2Fimages%2Ffig3.width-800.png)，但 SKB_MODE 可能没有那么快，因为数据包头必须由网络栈预先解析（见[视频](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3Dq3gjNe6LKDI)）。

5. 步骤 3 - 代码分发
--------------

在这一点上，如果不是因为一个问题 -- eBPF [代码可移植性](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Ffacebookmicrosites.github.io%2Fbpf%2Fblog%2F2020%2F02%2F19%2Fbpf-portability-and-co-re.html%23the-problem-of-bpf-portability)，一切都应该已经准备好打包和发布应用。历史上，这个过程涉及将 eBPF 源代码复制到目标平台，拉取所需的内核头文件，并为特定的内核版本进行编译。这个问题对于追踪 / 监控 / 跟踪的用例尤其明显，因为这些用例可能需要访问几乎所有的内核数据结构，所以唯一的解决办法是引入中介层（见 [CO-RE](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Ffacebookmicrosites.github.io%2Fbpf%2Fblog%2F2020%2F02%2F19%2Fbpf-portability-and-co-re.html)）。

另一方面，网络用例依赖于一个相对较小且稳定的内核类型子集，所以它们不会像跟踪和性能分析程序那样遇到同样的问题。根据我目前看到的情况，两种最常见的代码打包方法是：

*   将 eBPF 代码与所需的内核头文件放在一起，假设它们与底层内核相匹配（见 [Cilium](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fcilium%2Fcilium%2Ftree%2Fmaster%2Fbpf)）。
*   分发 eBPF 代码并在目标平台上拉取内核头文件。

在这两种情况下，eBPF 代码仍然需要在目标平台上编译，这是一个额外的步骤，需要在用户空间应用程序启动之前进行。然而，还有一个选择，那就是预先将 eBPF 代码编译成 ELF 格式文件，最终只分发 ELF 文件。这正是 [`bpf2go`](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fpkg.go.dev%2Fgithub.com%2Fcilium%2Febpf%2Fcmd%2Fbpf2go) 可以做到的，它可以将编译后的代码嵌入到 Go 包中。其依靠 `go generate` 注解指令产生一个[新的文件](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fnetworkop%2Fxdp-xconnect%2Fblob%2Fmain%2Fpkg%2Fxdp%2Fxdp_bpf.go)，其中包含编译好的 eBPF 和 `libbpf-go` 脚手架代码，唯一的要求是 [`//go:generate`](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fnetworkop%2Fxdp-xconnect%2Fblob%2Fmain%2Fpkg%2Fxdp%2Fxdp.go%23L14) 指令。一旦生成，我们的 eBPF 程序只需几行就可以被加载（注意没有任何参数）。

```
specs, err := newXdpSpecs()
objs, err := specs.Load(nil)


```

这种方法明显的优点是，我们不再需要在目标机器上编译，可以在一个软件包或 Go 二进制文件中同时运送 eBPF 和用户空间 Go 代码。这很好，因为它允许我们不仅将应用程序作为二进制文件使用，还可以将其导入任何第三方 Go 应用程序中（见[使用实例](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fnetworkop%2Fxdp-xconnect%23usage)）。

6. 阅读和有趣的参考资料
-------------

通用理论：  
[https://github.com/xdp-project/xdp-tutorial](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fxdp-project%2Fxdp-tutorial)  
[https://docs.cilium.io/en/stable/bpf/](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fdocs.cilium.io%2Fen%2Fstable%2Fbpf%2F)  
[https://qmonnet.github.io/whirl-offload/2016/09/01/dive-into-bpf/](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fqmonnet.github.io%2Fwhirl-offload%2F2016%2F09%2F01%2Fdive-into-bpf%2F)

BCC 和 libbpf：  
[https://facebookmicrosites.github.io/bpf/blog/2020/02/20/bcc-to-libbpf-howto-guide.html](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Ffacebookmicrosites.github.io%2Fbpf%2Fblog%2F2020%2F02%2F20%2Fbcc-to-libbpf-howto-guide.html)  
[https://nakryiko.com/posts/libbpf-bootstrap/](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fnakryiko.com%2Fposts%2Flibbpf-bootstrap%2F)  
[https://pingcap.com/blog/why-we-switched-from-bcc-to-libbpf-for-linux-bpf-performance-analysis](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fpingcap.com%2Fblog%2Fwhy-we-switched-from-bcc-to-libbpf-for-linux-bpf-performance-analysis)  
[https://facebookmicrosites.github.io/bpf/blog/](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Ffacebookmicrosites.github.io%2Fbpf%2Fblog%2F)

eBPF/XDP 性能：  
[https://www.netronome.com/blog/bpf-ebpf-xdp-and-bpfilter-what-are-these-things-and-what-do-they-mean-enterprise/](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.netronome.com%2Fblog%2Fbpf-ebpf-xdp-and-bpfilter-what-are-these-things-and-what-do-they-mean-enterprise%2F)

Linus Kernel 代码风格：  
[https://www.kernel.org/doc/html/v5.9/process/coding-style.html](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.kernel.org%2Fdoc%2Fhtml%2Fv5.9%2Fprocess%2Fcoding-style.html)

`libbpf-go` 样例程序：  
[https://github.com/takehaya/goxdp-template](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Ftakehaya%2Fgoxdp-template)  
[https://github.com/hrntknr/nfNat](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fhrntknr%2FnfNat)  
[https://github.com/takehaya/Vinbero](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Ftakehaya%2FVinbero)  
[https://github.com/tcfw/vpc](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Ftcfw%2Fvpc)  
[https://github.com/florianl/tc-skeleton](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fflorianl%2Ftc-skeleton)  
[https://github.com/cloudflare/rakelimit](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fcloudflare%2Frakelimit)  
[https://github.com/b3a-dev/ebpf-geoip-demo](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fb3a-dev%2Febpf-geoip-demo)

`bpf2go`：  
[https://github.com/lmb/ship-bpf-with-go](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Flmb%2Fship-bpf-with-go)  
[https://pkg.go.dev/github.com/cilium/ebpf/cmd/bpf2go](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fpkg.go.dev%2Fgithub.com%2Fcilium%2Febpf%2Fcmd%2Fbpf2go)

XDP 样例程序：  
[https://github.com/cpmarvin/lnetd-ctl](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fcpmarvin%2Flnetd-ctl)  
[https://gitlab.com/mwiget/crpd-l2tpv3-xdp](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgitlab.com%2Fmwiget%2Fcrpd-l2tpv3-xdp)

--Edited from Rpc