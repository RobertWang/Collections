> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/nyru8lpNiJNhE7BmDEWDTQ)

### 乐观地看 FUSE

我喜欢文件。每个计算机系统都理解文件。每个程序都知道如何读取和写入文件。这是一个真正通用的 API。因此，我喜欢 FUSE 的想法。FUSE 的名字来源于 Filesystem in Userspace，也就是 “用户态文件系统”，是一套允许用户模式程序定义文件系统的 Linux 接口。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/A9lbmbycqtrDgtiaB8QF6oaEC7fHeJ22yy3T54cgFaQedVdfKMnMhSIyUw6g6E7hT3UiaQ5oMfSta1qbPIC648rA/640?wx_fmt=png)

有了 FUSE，不需要内核模块就可以构建文件系统驱动程序。Fuse 是大量文件系统客户端的基础，包括 NTFS 甚至像 SFTP 或 Amazon S3 这样的远程 “文件系统”。它还可以用来制作奇怪的文件系统。这些文件系统实际上并不是真正的文件系统，比如 WikipediaFS，它允许人们使用自己的文本编辑器编辑维基百科文章。

在 Xethub，我们想要帮你用已有的工具本地访问任何版本的数据集。不用通过 S3 命令就能够直接浏览图像数据集还是很香的。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/A9lbmbycqtrDgtiaB8QF6oaEC7fHeJ22yib0hQU1dzKM7AAVEBagryVdlbhIYodw8NAWicnbBMbnxhT0pzJvB5gAA/640?wx_fmt=png)

 _Webflow 图片说明: 想象一下，如果你可以这样与任何文件互动_

启用这种超能力的明显解决方案是 FUSE。

然而，FUSE 是一个对着写起来很麻烦的 API:

*   要在底层和更抽象的两种 API 类中选择
    
*   有两个不兼容的 API 版本 (libfuse2 和 libfuse3)
    
*   随着时间的推移还有很多其他的小变化 (参见 FUSE_USE_VERSION)。
    

而且 FUSE 在 Mac 和 Windows 上不能原生地用，需要用户安装第三方驱动程序 (MacFuse, WinFuse)。每一个这种驱动程序都可能存在细微的 API 不兼容性。

### 问题的关键

基于这些问题，我问自己：

> **是否有可能构建一个真正跨平台的用户空间文件系统接口？**

为了回答这个问题，我回看了 20 年计算机科学历史并偶然发现了 **NFSv3**。

### NFS

NFSv3 是一个已经有 20 年历史的网络文件系统协议。它如此简单和普遍，以至于几乎每个操作系统都把它实现了一遍。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/A9lbmbycqtrDgtiaB8QF6oaEC7fHeJ22ymdicnKwtNGYql9LtHicdh46TdAb2e4D9IsLzcicbt3QSMBhKLAhqicdThA/640?wx_fmt=png)

NFSv3 协议有一套简洁优雅的设计原则：

1.  **服务器是完全无状态的**：这大大简化了实现。
    
2.  **傻瓜服务器，智能客户端**: (RFC 1813 在第 1.6 节第 4 段中有明确说明) 这很好，因为我们只需要实现服务器，而智能的客户端已经被实现并优化了 20 多年。
    
3.  **简单的缓存一致性规则**: **** 服务器不定义缓存策略。客户端多智能都无所谓。因为协议定义了一种机制让服务器一有变化就通知客户端。这种实现比 FUSE 更简单、更高效。在实际应用中，FUSE 守护进程本身必须明确地实现大量的缓存。使用 NFS，我们可以避免所有这些额外的复杂性。
    
4.  **NFS 客户端知道它在通过网络通信**: 这意味着 NFS 客户端和协议内置了我们可以现成用的超时、重试和失败的机制。无状态协议使得这些机制非常简洁。用在 FUSE 上，超时 / 失败行为必须在守护进程的每个地方都被可靠地实现。如果你卡在一次 API 调用，很容易就连带卡住守护进程和所有读取文件系统的程序。
    
5.  **实际上性能非常好。** 至少在 * nix 上，本地网络与管道一样快。 Windows 的话不确定，但我会很惊讶如果它不快。
    

总而言之，使用本地 NFS 而不是 FUSE 实现用户态文件系统使得更容易获得性能和韧性。我们可以利用现有的缓存支持和超过 20 年的强化而只需要实现_一次_服务器协议。

所以去年当我感染新冠隔离的时候，我试了试用 Rust 实现了一个 NFSv3 服务器，结果非常好。

### 我们在 XetHub 是怎么用 NFS 的

XetHub 做出了世界上第一个**原生、跨平台、用户态的文件系统实现**，让你不需要任何内核驱动程序就可以挂载任意大的数据集。

这使你能够在几秒钟内本地挂载约 660 GB 的 Llama 2 模型或查询 DuckDB 数据库来分析 parquet 大文件并选出你需要的数据。

所有这些目前都支持 Linux、Mac 和 Windows Pro（不幸的是不支持 Windows Home）。Windows 在体验上有一些小的奇特之处，但总体上是可用的。

### 开源 nfsserve

我们在 Github 上开源了我们的 Rust NFS 服务器实现 nfsserve。如果你也是一个🦀Rust-acean，你可以使用 cargo 安装`nfsserve="0.10"`。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/A9lbmbycqtrDgtiaB8QF6oaEC7fHeJ22y8FCs09wnbxTUoDDeicupULWyuJfN44PDibXZoo0xW2bSAfibOqu8Td9VA/640?wx_fmt=png)

文档绝对需要多多维护，我们也欢迎接受任何的 PR 优化！你可以在 readme 中找到一些贡献的线索。

我们会持续维护这个库，因为我们在 pyxet 中的`xet mount`实现和 xet-core(我们同样开源了) 实际上都依赖它。

我们已经达到的初步效果是：

*   读取性能相当好
    
*   写入功能可用，但仍然需要大量的优化
    

我确信 nfsserve 还有很多重构和性能提升的空间，希望大家从这篇文章中有所收获！

_注解:_ XetHub 开发了一个使用 NFSv3 协议而不是 FUSE 的跨平台用户态文件系统，从而实现了更好的性能和可靠性。他们已经开源了他们的 NFS 服务器 Rust 实现，用于他们的`xet mount`实现，并且支持 Linux、Mac 和 Windows Pro。

https://about.xethub.com/blog/nfs-fuse-why-we-built-nfs-server-rust