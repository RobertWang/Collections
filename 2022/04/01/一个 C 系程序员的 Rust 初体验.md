> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/_tZZNjbwS5tZwxKDYic5OQ)

引言：在工作里使用 Rust 已经有两个多月的时间了，谈谈我做为一名多年的 C 系（C、C++）程序员，对 Rust 的初体验。

一个 C 系程序员的 Rust 初体验
===================

最近由于工作的原因，使用上了 Rust 语言，在此之前我有多年的 C、C++ 编码经验（以下将 C、C++ 简称 C 系语言）。

使用 C 系语言编码时，最经常面对的问题就是内存问题，诸如：

*   野指针（Wild Pointe）：使用了不可知的指针变量，如已经被释放、未初始化、随机，等等。
    
*   内存地址由于访问越界等原因被覆盖（overflow），这不但是可能出错的问题，还有可能成为程序的内存漏洞被利用。
    
*   内存分配后未回收。
    

连 Chrome 的报告都指出，Chrome 中大约 70% 的安全漏洞都是内存问题，见：Memory safety[1]。

C 系语言发展到今天，已经有不少可以用于内存问题检测的利器了，其中最好用的莫过于 AddressSanitizer[2]，它的原理是在编译时给程序加上一些信息，一旦发生内存越界访问、野指针等错误都会自动检测出来。

但是即便有这些工具，内存问题也不好解决，其核心的原因在于：这些问题绝大部分都是运行时（Runtime）问题，即要在程序跑到特定场景的时候才会暴露出来，诸如上面提到的 AddressSanitizer 就是这样。

都知道解决问题的第一步是能复现问题，而如果一个问题是运行时问题，这就意味着：复现问题可能会是一件很麻烦的事情，有时候还可能到生产环境去复现。

以我之前经历的一个 Bug 来看这类工作的复杂度，见[线上存储服务崩溃问题分析记录 - codedump 的网络日志 [3]](http://mp.weixin.qq.com/s?__biz=MzAwMjgwMTEzNw==&mid=2652227263&idx=1&sn=d18fdd4acc4c854b619e2413b767e76b&chksm=81258a21b6520337c4fdd166a782fa05166668277256ec7da02af9f34a9e90197275b945237b&scene=21#wechat_redirect)，这是一个很典型的发生在生产环境上由于内存错误导致的崩溃问题：

*   不好复现，因为跟特定的请求相关，还跟线程的调度有关；
    
*   本质是由于使用了被释放的内存导致的错误。
    

这个线上问题，记得当时花了一周时间来复现问题解决。

换言之，如果一个问题要等到运行时才能发现，那么可以预见的是：一旦出现问题，要复现问题可能要花费大量的精力，以及需要很多经验才行。如果一个问题还是在特定场景，或者用户现场才出现的，那就更麻烦了，C 系程序员以往一般都是这样来保存 “现场”：

*   出现崩溃的时候保存 core 文件来查看调用堆栈、变量等信息。
    
*   发明了各种复制流量重放的工具，比如 tcpcopy[4] 等。
    

总而言之，运行时问题一旦出现是很麻烦的，而解决这类问题的时间是难以预期的。

Rust 给这类内存问题的解决提供了另一个解决思路：

*   一个内存地址同时只能被一个变量使用。
    
*   不能使用未初始化的变量。
    
*   ...
    

简而言之，凡是可能出现内存错误的地方，都在语言的语法层面给予禁止，换来的就是更多的编译时间，因为要做这么多检查嘛，而需要更多的编译时间反过来就需要更好的硬件。我想这也是 Rust 到了最近几年才开始慢慢流行开来的原因之一，毕竟即便是现在，一些大型的 Rust 项目普通的机器编译起来也还是很耗时。

“编译时间（compile time）”是一个可以预期的固定时间，能通过增加硬件性能（比如买更好的机器来写 Rust）来解决；而 “运行时问题” 一旦出现，查找起来的时间、精力、场景（比如出现在用户现场、几百万次才能重现一次等）不确定性可就很高了。

两者权衡，我选择解决 “编译时间” 问题。而且，在我意识到有这样的工具能够在编译期解决大部分内存问题时，反过来再看使用 C 系语言的项目，几乎可以预期的是：只要代码和复杂度上了一定规模，那么这类项目都要花上相当的一段时间才能稳定下来。原因在于：类似内存问题这样的运行时问题，是需要场景去积累，才能暴露出来的，而场景的积累，就需要很多的小白鼠和运行时间了。

总结一下我的观点：

*   C 系语言最多的问题就是各类内存问题，而这些问题大多是运行时问题。即便现在已经有了各种工具，解决其运行时问题也很困难。
    
*   Rust 解决这类问题的思路，是在语法层面禁止一切可能出现内存问题的操作，换来的代价就是更多的编译时间。
    
*   解决可预期的 “编译时间” 和难预期的“运行时问题”，我选择前者。
    

番外篇
===

rr
--

rr: lightweight recording & deterministic debugging[5] 也是出自 Mozilla 的另一款调试 C 系程序的利器，`rr`是`Record and Replay`的简称，目的还是为了解决各种运行时问题，由于运行时问题中存在着各种不确定的因素，包括：

*   变量值。
    
*   进程、线程环境，比如不同的线程调度顺序可能导致了不同的结果。
    
*   输入不同的数据，能得到不同的结果。
    

于是，`rr`要解决的核心问题，就是让一个程序在运行时有一个固定的环境，它可以抓取程序运行的环境保存下来。这样在出现问题之后，就能使用它可以记录下来程序运行时的环境，不停的重放来调试解决问题。

但是，即便是这样，`rr`可能更适合于明确知道问题的情况下去抓取环境，不可能在线上直接打开这个工具。所以又回到前面的结论了：调试运行时问题可能面对的困难，包括场景、时间、用户现场等等不确定因素。

`rr`和`Rust`一样，都出自 Mozilla，我想不是偶然的。Mozilla 和 chrome 等一样，都是使用 C++ 编码的超大型项目，而这里一定遇到了各种运行时问题，不止于内存问题，所以才要使用各种工具来辅助解决这类问题。

吃上硬件升级的红利了吗？
------------

前面提到过，Rust 目前较大的问题是编译时间过长，这可能是导致它最近几年才开始逐渐流行开来的原因。其实反过来说，在硬件升级之后，应该能尽量利用上硬件，在编译期尽量多检查出错误来，减少运行时发现问题的数量。这样，才能吃上硬件升级的红利，利用硬件来减少自己的犯错。

一方面硬件升级给了编程语言能施展更大、更快的的 “舞台”，随着舞台的更新，就会有更新、更好的工具出现；另一方面做为从业者，也应该与时俱进，多学习跟进这些工具的演进。

我看到有一些人，强调自己多早就已经用 C 语言写代码了，但是查内存问题还在用慢的不行的`Valgrind`，没听过更不知道怎么用`Address Sanitizer`。

想说如果技能点都已经不更新了，强调多早学的有什么意义？好比 1950 年就会打算盘，有意义吗？强调多早就用 C 语言类似的言论，在我看来就是 “倚老卖老”，但是技术日新月异的领域，卖老的意义不大。

《Rust for Rustaceans》
---------------------

推荐 Rust for Rustaceans[6] 作者 Jon Gjengset 的油管频道：https://www.youtube.com/c/JonGjengset/playlists

有很多很有深度的 Rust 分享，比如：

*   Implementing TCP in Rust (part 1) - YouTube[7]
    
*   Porting Java's ConcurrentHashMap to Rust (part 1) - YouTube[8]
    

### 参考资料

[1]

Memory safety: https://www.chromium.org/Home/chromium-security/memory-safety/

[2]

AddressSanitizer: https://en.wikipedia.org/wiki/AddressSanitizer

[3]

线上存储服务崩溃问题分析记录 - codedump 的网络日志: https://www.codedump.info/post/20190413-problem-fix/

[4]

tcpcopy: https://github.com/session-replay-tools/tcpcopy

[5]

rr: lightweight recording & deterministic debugging: https://rr-project.org/

[6]

Rust for Rustaceans: https://rust-for-rustaceans.com/

[7]

Implementing TCP in Rust (part 1) - YouTube: https://www.youtube.com/watch?v=bzja9fQWzdA&list=PLqbS7AVVErFivDY3iKAQk3_VAm8SXwt1X

[8]

Porting Java's ConcurrentHashMap to Rust (part 1) - YouTube: https://www.youtube.com/watch?v=yQFWmGaFBjk&list=PLqbS7AVVErFj824-6QgnK_Za1187rNfnl

**参考阅读：**

*   [并发编程中的大坑：你的直觉 & 有序性问题](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653559045&idx=1&sn=e6d0a8ae11427118f617937715724115&chksm=8139889db64e018beee3c8cc05e98d48bf863efb29a0d5022e437f2e34e1f3d7b26175a915ac&scene=21#wechat_redirect)
    
*   [从 C++ 转向 Rust：两大主题值得关注！](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653559038&idx=1&sn=478eb2fa1866affd2181bc0a94f4bf2c&chksm=81398966b64e007089e03fbdef623a487798db9e4dc7711805ee92f7aed7b71cfc91f115f167&scene=21#wechat_redirect)
    
*   [攻击者视角对 AntiSpam 工作的分析](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653559026&idx=1&sn=78751426d67a6f4e56a421a240e883ad&chksm=8139896ab64e007c83b4add546ede3ef139ade068c745b322e609e50414f0f5f2552d401972b&scene=21#wechat_redirect)
    
*   [高并发系统概念梳理](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653558983&idx=1&sn=9ce05e758a0bdc14a99cd224c783a32a&chksm=8139895fb64e004978fc5597b1ddecefe7f0ca2382b4afeba70e64185d2d064f03c852ac1fcf&scene=21#wechat_redirect)
    
*   [一文解密 Netflix 的快速事件通知系统是如何工作的](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653558963&idx=1&sn=cd4b4d277537e71d93fbb0b53a7c59ea&chksm=8139892bb64e003d4686f3f251e2d193ddf2b8a552cee986fce0d07161b065bb48b4c43d3716&scene=21#wechat_redirect)