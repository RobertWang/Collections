> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1649506515&ver=3728&signature=f*vnMenUyR*IuPf*1KAu1Pwbjn3lvji89ylrM29BAtvd2RxIBMyQw9we4TWFNRb9BGgH9PdhjFpKlYcIiCHXE-a1ZN*eKH*R4pPIWC-yFq6zTbXcPW4FiH2Wbjz19OKL&new=1)

> 恕我直言，你运行的可能是假 time

大家好，我是**肖邦**，这是我的第 **16** 篇原创文章。

最近在使用 `time` 命令时，无意间发现了一些隐藏的小秘密和强大功能，今天分享给大家。

`time` 在 Linux 下是比较常用的命令，可以帮助我们方便的计算程序的运行时间，对比采用不同方案时程序的运行性能。看似简单的命令，其实蕴藏着很多细节和技巧，来跟着肖邦一起学习吧。

1 基础用法详解
--------

先来看下最基础的用法，也可能是大家最常见的用法了

```
root@chopin:~$ time find . -name "chopin.txt"......real   0m0.174suser   0m0.084ssys    0m0.084s
```

可以很清楚看到，_find_ 命令执行的时间为 _0.174s_，是不是很简单，很方便呢

不过，`time` 命令输出了三个参数，我们只用到了第一个参数，其它两个参数代表什么含义呢？

这里我来解释一下：

*   _real_：表示的是墙上时间，说白了，其实就是从程序运行开始到结束所经历的时间；
    
*   _user_：表示程序运行期间，cpu 在用户态所花费的时间；
    
*   _sys_：表示程序运行期间，cpu 在内核态所花费的时间；
    

细心的读者会发现，上述案例中的 _user_ + _sys_ 不等于 _real_，这是怎么回事呢？

其实上边解释的 _user_ 和 _sys_，是 cpu 执行指令所消耗的时间，并不包含：进程阻塞 IO、调度排队，这些非 cpu 运行时间。

案例中 _find_ 执行查找文件过程中，会有磁盘 IO 读取，这时 cpu 会被释放出来干别的事情，这些 IO 消耗的时间，是不包含在 _user_ 和 _sys_ 统计数据中，所以就出现了 _real_ 时间大于 _user_ + _sys_ 了。

再通过一个示例来验证并加强我们的理解

```
root@chopin:~$ time sleep 2<br data-darkmode-color-16495065289408="rgb(221, 221, 221)" data-darkmode-original-color-16495065289408="#fff|rgb(62, 62, 62)|rgb(221, 221, 221)" data-darkmode-bgcolor-16495065289408="rgb(59, 61, 52)" data-darkmode-original-bgcolor-16495065289408="#fff|rgb(39, 40, 34)">real   0m2.001s<br data-darkmode-color-16495065289408="rgb(221, 221, 221)" data-darkmode-original-color-16495065289408="#fff|rgb(62, 62, 62)|rgb(221, 221, 221)" data-darkmode-bgcolor-16495065289408="rgb(59, 61, 52)" data-darkmode-original-bgcolor-16495065289408="#fff|rgb(39, 40, 34)">user   0m0.000s<br data-darkmode-color-16495065289408="rgb(221, 221, 221)" data-darkmode-original-color-16495065289408="#fff|rgb(62, 62, 62)|rgb(221, 221, 221)" data-darkmode-bgcolor-16495065289408="rgb(59, 61, 52)" data-darkmode-original-bgcolor-16495065289408="#fff|rgb(39, 40, 34)">sys    0m0.000s<br data-darkmode-color-16495065289408="rgb(221, 221, 221)" data-darkmode-original-color-16495065289408="#fff|rgb(62, 62, 62)|rgb(221, 221, 221)" data-darkmode-bgcolor-16495065289408="rgb(59, 61, 52)" data-darkmode-original-bgcolor-16495065289408="#fff|rgb(39, 40, 34)">
```

可以清楚地看到，sleep 命令基本上没有消耗 cpu，程序真实的运行时间就是 2 秒

那我们是不是可以得出如下结论了呢:

**`real >= user + sys`**

其实这个结论在单个 cpu 情况下，是正确的。

如果服务器是多个 cpu，你的程序正好可以将多个 cpu 充分利用起来，程序运行期间是多核心并行的，那么 _user_ + _sys_ 统计的 cpu 时间可能就会大于 _real_ 时间啦

所以这 3 个时间之间的关系并不是恒定的，你需要清楚的了解服务器是否为多个核心。

通过统计到的 cpu 消耗时间，我们也可以大概知道，程序运行期间 cpu 利用情况。对于单核，计算密集型的程序，_real_ 会很接近 _user_ 和 _sys_ 时间之和的。

> Tips：有些同学可能对操作系统可能不太熟悉，这里简单科普下**内核态**和**用户态**的基本概念

_Linux 为使系统更稳定，采取了隔离保护的措施，运行状态分为内核态和用户态_：

*   _用户态_：用户代码不具备直接访问底层资源的能力，需要借助内核提供的系统调用 API。在这种隔离保护下，即使用户程序崩溃，也不会影响整个系统的功能。
    
*   _内核态_：内核代码具备最大权限，可执行任意 cpu 指令，不受任何限制。内核态通常是操作系统提供的最底层、最可靠的代码运行的，内核态的代码崩溃是灾难性的，影响整个系统的正常运行。
    

2 你运行的可能是假 time
---------------

`time` 还有其它功能吗？看一下帮助文档吧

```
root@chopin:~$ time --help--help: command not foundreal 0m0.129suser 0m0.084ssys 0m0.036s
```

竟然报错，将 `--help` 当成了命令来执行了，难道 `time` 就这么点能耐吗？

好吧，我也不卖关子了，直接说答案：**你运行的可能是假 time**。你可能有点懵逼，怎么就假的了。

其实在 Linux 系统上，使用 `time` 时，你可能会遇到三种版本：

```
# 1. Bashtime is a shell keyword# 2. Zshtime is a reserved word# 3. GNU timetime is /usr/bin/time
```

我们当前 Shell 是 _Bash_，可以通过 _type_ 命令

```
root@chopin:~$ type timetime is a shell keyword
```

可以看到，我们刚才执行的 `time` 是 Shell 的内置命令，如果你用的是 _zsh_，默认使用的 `time` 也是对应内置命令。

_GNU time_ 命令路径是 _/usr/bin/time_，一般的 Linux 发行版都带有这个命令，它才是我们今天的猪脚。

3 更强大的功能
--------

_GNU time_ 命令提供了更强大的功能：

*   更详细的统计信息
    
*   更丰富的格式输出
    
*   支持保存统计数据到文件
    

下边我们来学习写 _GNU time_ 的使用

_1._ 最简单的用法

```
root@chopin:~$ /usr/bin/time sleep 2<br data-darkmode-color-16495065289408="rgb(221, 221, 221)" data-darkmode-original-color-16495065289408="#fff|rgb(62, 62, 62)|rgb(221, 221, 221)" data-darkmode-bgcolor-16495065289408="rgb(59, 61, 52)" data-darkmode-original-bgcolor-16495065289408="#fff|rgb(39, 40, 34)">0.00user 0.00system 0:02.00elapsed 0%CPU (0avgtext+0avgdata 1784maxresident)k<br data-darkmode-color-16495065289408="rgb(221, 221, 221)" data-darkmode-original-color-16495065289408="#fff|rgb(62, 62, 62)|rgb(221, 221, 221)" data-darkmode-bgcolor-16495065289408="rgb(59, 61, 52)" data-darkmode-original-bgcolor-16495065289408="#fff|rgb(39, 40, 34)">0inputs+0outputs (0major+72minor)pagefaults 0swaps<br data-darkmode-color-16495065289408="rgb(221, 221, 221)" data-darkmode-original-color-16495065289408="#fff|rgb(62, 62, 62)|rgb(221, 221, 221)" data-darkmode-bgcolor-16495065289408="rgb(59, 61, 52)" data-darkmode-original-bgcolor-16495065289408="#fff|rgb(39, 40, 34)">
```

使用 _GNU time_ 命令，直接使用绝对路径即可，我们可以看到输出信息更多了，不过格式有点丑，后边会讲如何自定义格式。

_2._ 保持内置 `time` 的输出样式

有同学会问，能输出内置 Shell 那种的格式么？可以的，使用 `-p` 选项即可

```
root@chopin:~$ /usr/bin/time -p sleep 2<br data-darkmode-color-16495065289408="rgb(221, 221, 221)" data-darkmode-original-color-16495065289408="#fff|rgb(62, 62, 62)|rgb(221, 221, 221)" data-darkmode-bgcolor-16495065289408="rgb(59, 61, 52)" data-darkmode-original-bgcolor-16495065289408="#fff|rgb(39, 40, 34)">real 2.00<br data-darkmode-color-16495065289408="rgb(221, 221, 221)" data-darkmode-original-color-16495065289408="#fff|rgb(62, 62, 62)|rgb(221, 221, 221)" data-darkmode-bgcolor-16495065289408="rgb(59, 61, 52)" data-darkmode-original-bgcolor-16495065289408="#fff|rgb(39, 40, 34)">user 0.00<br data-darkmode-color-16495065289408="rgb(221, 221, 221)" data-darkmode-original-color-16495065289408="#fff|rgb(62, 62, 62)|rgb(221, 221, 221)" data-darkmode-bgcolor-16495065289408="rgb(59, 61, 52)" data-darkmode-original-bgcolor-16495065289408="#fff|rgb(39, 40, 34)">sys  0.00<br data-darkmode-color-16495065289408="rgb(221, 221, 221)" data-darkmode-original-color-16495065289408="#fff|rgb(62, 62, 62)|rgb(221, 221, 221)" data-darkmode-bgcolor-16495065289408="rgb(59, 61, 52)" data-darkmode-original-bgcolor-16495065289408="#fff|rgb(39, 40, 34)">
```

_3._ 输出更详细的信息

还可以输出更加详细的信息，让你对程序运行信息一目了然。请使用 `-v` 选项

```
root@chopin:~$ /usr/bin/time -v sleep 2Command being timed: "sleep 2"User time (seconds): 0.00System time (seconds): 0.00Percent of CPU this job got: 0%Elapsed (wall clock) time (h:mm:ss or m:ss): 0:02.00Average shared text size (kbytes): 0Average unshared data size (kbytes): 0Average stack size (kbytes): 0Average total size (kbytes): 0Maximum resident set size (kbytes): 1804Average resident set size (kbytes): 0Major (requiring I/O) page faults: 0Minor (reclaiming a frame) page faults: 71Voluntary context switches: 1Involuntary context switches: 1Swaps: 0File system inputs: 0File system outputs: 0Socket messages sent: 0Socket messages received: 0Signals delivered: 0Page size (bytes): 4096Exit status: 0
```

这里详细介绍下 `time` 命令输出各项指标

**（一）时间相关**

![](https://mmbiz.qpic.cn/mmbiz_png/bSXeamhD9uyG2MIdgsJAbf7lauyuEj9ZIQmulyL9UyDf2ndt2Lib91919bJtbwdranuZhhofdfJKBUCYzI4aqQg/640?wx_fmt=png)

**（二）内存相关**

![](https://mmbiz.qpic.cn/mmbiz_png/bSXeamhD9uyG2MIdgsJAbf7lauyuEj9ZXibllKtpEVLnu04oE8nh9QoqgqpPBq9NVdRw3VYN2AMdvrm3ZUOICTg/640?wx_fmt=png)

**（三）IO 相关**

![](https://mmbiz.qpic.cn/mmbiz_png/bSXeamhD9uyG2MIdgsJAbf7lauyuEj9ZtZ84iaibVZRGaoKp6JkeUDdvb0P8gvpuyY0TmtQtTzAprxdcciaXaEKpQ/640?wx_fmt=png)

_4._ 统计信息输出到文件

如果你希望将 `time` 统计的信息输出到文件，可以使用 `-o` 选项

```
root@chopin:~$ /usr/bin/time -v -o a.txt sleep 2<br data-darkmode-color-16495065289408="rgb(221, 221, 221)" data-darkmode-original-color-16495065289408="#fff|rgb(62, 62, 62)|rgb(221, 221, 221)" data-darkmode-bgcolor-16495065289408="rgb(59, 61, 52)" data-darkmode-original-bgcolor-16495065289408="#fff|rgb(39, 40, 34)">
```

统计信息直接保存到了 a.txt，如果你希望统计信息能够_追加_到文件，可以额外加 `-a` 选项

_5._ 自定义格式输出

如果命令中内置的输出格式，不符合你的需求，_GNU time_ 可以支持自定义输出格式，通过选项 `-f` 可以各种指标参数

```
/usr/bin/time -f "real %e\nuser %U\nsys %S\n" sleep 1real 1.00user 0.00sys  0.00
```

具体支持的格式，贴心的肖邦已经帮你整理好了

![](https://mmbiz.qpic.cn/mmbiz_png/bSXeamhD9uyG2MIdgsJAbf7lauyuEj9ZAoa7SqaOwcLB8zpukkuGsvRvASQfdSt2icgch9OCKJSGffp124LyQDQ/640?wx_fmt=png)

这些格式参数太多了，平时大部分情况用不到，可以收藏起来，以便后期使用时可以快速参考。

4 在性能分析中的作用
-----------

看到这么多系统参数指标，难免会有同学会感到疑惑，这些参数能干什么呀？

其实这些指标，对应到操作系统 cpu、内存、IO 这几方面。深刻的理解了这些指标参数，可以帮助你从本质上把握程序的运行情况，甚至可以协助你分析程序的性能瓶颈。

下边我简单解释几个概念，希望能起到抛砖引玉的作用。

**（一）CPU 时间**

cpu 时间包括：_real_、_user_、_sys_，当 _user_ + _sys_ >= _real_ 时，说明该程序是计算密集型；当 _user_ + _sys_ 远小于 _real_ 时，说明存在较多的 IO 等待。

**（二）上下文切换**

平时所说的_上下文_，是指进程的运行环境，包括当时的寄存器值、内存堆栈等信息，内核可以根据上下文完全恢复一个被打断的进程任务。

当执行系统调用、进程切换时，都会产生上下文切换。切换上下文时，操作系统需要为进程保存和恢复上下文信息。

上下文切换分为_主动_和_被动_两种，主动上下文切换多，说明存在较多的阻塞调用；被动上下文切换说明 cpu 使用率高。

当上下文切换过多时，意味着较多的 cpu 时间花费在上下文切换上，导致 cpu 处理进程任务的有效时间大大减少。

**（三）缺页异常**

次缺页异常较多，说明程序的内存布局相对合理，命中率高；当主缺页异常较多时，说明程序对内存的访问跳跃性大，命中率低。

处理缺页异常和切换上下文的时间，不包含在 _user_ 和 _sys_ 中，当发现 _user_ + _sys_ 远小于 _real_ 时，则很可能大部分时间都消耗在这些地方，需要重点分析这两点。