> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [my.oschina.net](https://my.oschina.net/u/155323/blog/5320917)

> ebpf 之前一直有研究 linux 下的 dtrace 技术，包括 systemtap，ftrace，perf 等，但是进入 kubernetes 领域后在网络领域接触到了 cilium 项目，cil......

ebpf
----

之前一直有研究 linux 下的 dtrace 技术，包括 systemtap，ftrace，perf 等，但是进入 kubernetes 领域后在网络领域接触到了 cilium 项目，cilium 在 kubernetes 中体现出来优势太明显了，因此接触了 ebpf 技术。我在下面大概列出了 ebpf 相关的模块。 后面我详细介绍各个模块。

![](http://hushi55.github.io/images/ebpf/ebpf-introduction.png)

### 介绍

ebpf 是一种革命性的 linux kernel 可编程技术，而不依赖于重新编译 kernel 或者通过加载 kernel modules。 最近几年 ebpf 正在成为一个需要通过 linux kernel 才能解决问题的标准技术。 ebpf 成为新一代的 networking, security, application profiling 领域内的事实标准。

*   monitoring/observability
*   networking
*   security

从上面的简单描述中可以知道这个技术应该具备以下的特点

*   可编程 ---> 所以 ebpf 被设计成了一个 RISC 精简指令虚拟机
*   linux ---> 这个技术和 linux kernel 强绑定
*   kernel ---> 要求这个技术必须保证 kernel 的稳定性和安全性

![](http://hushi55.github.io/images/ebpf/ebpf-runtime.png)

上图可以看出一个 ebpf 是如何在 linux kernel 中运行的

1.  需要使用 LLVM 编译器将 C code 编译为 ebpf RISC 指令
2.  ebpf 校验器会校验这个 ebpf 程序是否对 kernel 是安全的
3.  ebpf JIT 将 ebpf 程序翻译为 native code 运行到 kernel 的各个 probe 点

### Security

![](http://hushi55.github.io/images/ebpf/intro_security-e714bea99d4351c1097477e8920d94ec.png)

在 Security 领域，可以借助 ebpf 在 system calls 和 network filtering 两个方面 上的功能做出革命性的 Security 系统。

### Tracing & Profiling

![](http://hushi55.github.io/images/ebpf/intro_tracing-ffa5e3fa3407ecb445b1549f85f590f5.png)

通过 ebpf 能够 trace kernel 和 用户态的所有行为，这样可以在系统层面和用户应用层面提供一个系统的诊断。

### Networking

![](http://hushi55.github.io/images/ebpf/intro_networking-46255f740daa161407f59190a8774e9a.png)

借助 ebpf 的 kernel 可编程性，能够非常方便和高性能完成网络协议解析，这样非常适合 networking 领域。

### Observability & Monitoring

![](http://hushi55.github.io/images/ebpf/intro_observability-fcba5bd29e9179954764bb0ee9385905.png)

ebpf 在可见性领域可以不在依赖操作系统的静态数据导入，而是可以在 kernel 内部自定义收集各种统计数据。 可以大大提高可见性的深度。

参考
--

*   [ebpf.io](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Febpf.io%2F)
*   [cilium](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fdocs.cilium.io%2Fen%2Fstable%2Fbpf%2F)
*   [New GKE Dataplane](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fcloud.google.com%2Fblog%2Fproducts%2Fcontainers-kubernetes%2Fbringing-ebpf-and-cilium-to-google-kubernetes-engine)