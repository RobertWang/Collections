> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651169460&idx=1&sn=eed35bfb086513da3ec452225cd7ea02&chksm=806473ebb713fafde1b5296a4ab79208515b185c3ff32d14dc44468be54e12c00becbe04397d&scene=21#wechat_redirect)

关于汇编的第一篇文章：

Debug 是什么
---------

Debug 是 Windows / Dos 操作系统提供的一种功能。使用 Debug 能让我们方便查看 CPU 各种寄存器的值、内存情况，方便我们调试指令、跟踪程序的运行过程。

接下来我们会用到很多 debug 命令，但是使用这些命令的前提是，你需要在电脑上安装一下 debug，Windows/Mac 都可以安装，获取链接我已经给你找出来了。阿，忘记说了，我们这里使用的是 _Dos box_ 来模拟汇编的操作环境。

传送门（Mac 和 Windows 都是）：https://www.dosbox.com/download.php?main=1

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdG8gLlCd587ISJlCzkv40uemJ5M99q5wVoOJDPVrroTOurrbxesz8Qew/640?wx_fmt=jpeg)

下载完成后打开 DosBox ，打开之后是这样的。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdGjAD2jGicSgvyszSXD5xsmVB6jIa2O69gicYnDFZYapY01afbtlNNZjwQ/640?wx_fmt=jpeg)

此时我们输入 debug 命令应该提示的是

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdGrduibHrAp1JsU0Q12JAr8wxSpkodL71tmNYqrCLggM8ibqF2aJ1ORyfA/640?wx_fmt=jpeg)

因为我们还没有进行连接和挂载，此时我们执行

```
 mount c D:\debug
```

执行这条命令时，你需要现在 D 盘下创建一个 debug 文件夹，然后我们挂载到 debug 下面。

并且执行 `C:` 切换到 C 盘路径下。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdG8VTBaiaBjz7reUxcCpYr1aLZtW7oRf8USudxB7hIq67dTobdJIg5r6w/640?wx_fmt=jpeg)

此时我们就可以执行 debug 命令了。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdGsEH0SblEUdg3pdLcdBuf276LlAqOcwZak9EV9TjlnSuzdf73ibgWZgg/640?wx_fmt=jpeg)

这里需要注意一点，我在 Windows 10 系统下搭建 Debug 环境时，在挂载完成后输入 debug ，还是提示 _Illegal command:debug_ ，此时你需要再下载一个 debug.exe ，贴心的我也把下载地址给你了。

下载地址：https://pan.baidu.com/s/177arSA34plWqV-iyffWpEw#list/path=%2F 密码：3akd

需要下载里面的 _debug.exe_，然后把它放在你挂载的路径下，这里我挂载的路径时 D 盘下的 debug 文件夹。

放置完成之后，再输入 debug 就可以了。

因为每次打开 Dosbox 都会执行上面这些命令，真的好烦，那怎么办呢？一个简单的办法是在 Dosbox 安装路径下找到

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdGkNibykico269ns7fD05J5DmuicS5IZN6hS3pkfQQKqZVaRFicAxiawFBQlA/640?wx_fmt=jpeg)

打开之后，在末尾键入

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdGVYcyKCUg9J6tGOfrWURQrtSXmLkcENhYTU8Dbk0QCKpx53Nv8WbySg/640?wx_fmt=jpeg)

就 OK 了，下次直接打开 Dosbox ，会默认执行这三条命令，至此，就是我搭建 Dosbox 遇到的所有问题了。

Debug 实战
--------

玩儿汇编得学会用 Debug ，Debug 是一种调试程序，通过 Debug 能让我们能够看到内存值，跟踪堆栈情况，看到寄存器所暂存的内容等，同时也能够更好地帮助我们理解汇编代码，所以学会 Debug ，非常重要，这是一种不可或缺的动手能力。

下面我们会用到几种 Debug 命令，这里先简单介绍下。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdGAp3hcmFctQib06UUKr1IzwskLxlmujjwAaXricKQLkpRibeOpd1jDRKgQ/640?wx_fmt=jpeg)

Debug 命令有很多，不过常用的一般就上面这几个。

好了，现在我们直接进入正题，开始在 Dosbox 上正式进行 Debug 操作，首先打开 Dosbox。

嗯。。。。。。这个界面我们打开很多次了。

那我写个命令呢？好吧，没演示过，下面就来了！

### Debug -r

亲，用 Debug -r 就可以查看和修改 CPU 寄存器内容了呢。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdGlic1de1mficxRTtsa5Fd2mKwXohhVIh4yqI2ymd11y3Be46sjibdHtwpg/640?wx_fmt=jpeg)

查看寄存器内容。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdGPXwDhtSjG9xXK0sZ1RzDMZaZwIy4FKCAibKGSr0bwWt96m4wEm7hyag/640?wx_fmt=jpeg)

> 这里需要注意一下 -r 大小写的问题，Debug -r 是查看寄存器内容。而 -R 则是无效指令。

上图列出来了很多寄存器，你可能觉得无从下手，不要乱，我们先从最基本的开始入手，也就是 CS 和 IP，CS（Code Segment）是代码段寄存器，一般也被称为段基址，可以认为是程序访问的入口，CPU 需要从 CS 中找到从哪个位置开始取指执行，但是我们还不知道要取哪一段，这时候 IP 的作用就体现出来了，IP（Instruction Pointer）就是指令指针寄存器，也叫做偏移地址，它会告诉我们从段基址开始，取哪一段的地址。

可以使用段基址: 偏移地址来确定内存中的指定地址。

> 这里我们只是简单聊一下这两个寄存器的概念，要了解这两个寄存器的具体作用，可以看笔者的上一篇文章

使用 -r 也能够修改寄存器的内容，如下所示

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdGPCXiabUbQQRbU4r8gGLElEjaiaVWncgCScBTibww11ygN3u7o7g2galDg/640?wx_fmt=jpeg)

-r 一般的格式是 -r 寄存器，然后系统会进行冒号提示，后面就是你要修改的内容。

### Debug -d

使用 -d 指令可以查看内存中的内容。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdGGcictfrhIDQh0QnIc7MJGyfEibqvhfdXMCOOncibD6d3TiblxLUgmImzYQ/640?wx_fmt=jpeg)

输出的内存值默认是按照 CS:IP 的地址开始的，由于 CS 的值默认是 073F，而 IP 默认是 0100，所以 -d 的内存值是 073F:0100 。

-d 的格式很多，下面只介绍一下常用的几种格式。

形似 -d 1000:0 这种 -d 段基址 偏移地址的格式可以产生如下输出。

> 为什么都是 00 呢，因为内存单元的值没有被改写，说白了就是这块内存区域没有存值，如何改写我们后面会说。

右侧几个 ...... 表示每个内存单元可显示的 ASCII 码字符，因为内存没有值，所以也没有对应的 ASCII 码。我们可以数一下，每行有 16 个 . ，这表示每一个 00 都对应了一个 ASCII 码。

我们可以使用 -d 1000:9 这种 -d 段基址: 起始偏移地址 格式来显示从 1000 的第几位开始。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdGgSxUE5MhvNP95E2ZBeejdzD8AZpxGhFV8Cib3EdrVsoca7JFQbEL8cg/640?wx_fmt=jpeg)

Debug 从 1000:9 开始，一直到 1000:88，一共是 128 个字节，第一行中的 1000:0 ~ 1000:8 中的内容没有显示。

还可以使用 -d 1000:0 9 这种 -d 段基址: 起始偏移地址 结尾偏移地址的格式来输出。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdGbr4qo5zmduLF9SL29ZMo5Libm9gyJfOiaGVeYlBpdMibXncsXx64z3Cdg/640?wx_fmt=jpeg)

还可以是使用 -d 偏移地址来在不指定段基址的情况下，查看内存值。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdGMZEib9PQPL6icQxo9MvK9zhBbPK5h8r72j7O5wmKrsfTVibRjKbKeNOuA/640?wx_fmt=jpeg)

### Debug -e

上面说的都是查看内存中指定位置或者区域的值，下面我们要来改写一下内存值。

使用 `-e` 可以改写内存值，比如我们想要改写 1000:0 ~ 1000:f 中的内容，可以使用 -e 1000:0 0 1 2 3 4 5 6 7 8 9 0 a b c d e f 这种方式，如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdGspBYH1owMgA5G6bbFzrJWicCAiaicX0sic4jv5A42IfwicNI9WMpJDjU5eA/640?wx_fmt=jpeg)

这里需要注意下，在进行 -e 改写的时候，每个值中间都有一个空格，如果没有空格的话，会当做一个内存值来看待。

然后用 -d 1000:0 看到我们刚改写的内存值。

还可以使用提问的方式来逐个修改从某一地址开始的内存单元的内容。

还是用 1000:100 来举例子，输出 -e 1000:100 后按下回车键。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdGtgLMpTcnRK94yViaEJgQgF1qTJclibcsQKshBibOTyqyyvvCBhzTqEMqQ/640?wx_fmt=jpeg)

如上图所示，可以看到我们先输入了一次 -e 1000:100 这个指令，然后按下了回车键。

> 注意，如果这里你按下了回车键，就相当于整个 -e 改写的过程已经完成。

如果你想要继续改写后面内存中的值，你需要按下空格键。

我们改写了 1000:100 之后的内存值，然后使用 -d 1000:100 查看我们改写的内容是否生效。

-e 命令还可以支持写入字符，比如我们可以向 1000:0 这个位置开始写入数值和字符，-e 1000:0 1 'a' 2 'b' e 'c' 。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdGKRKztEgG98tiafGM7FlFLRb2QhhFxNzdrVP91YZy1t2dLjibdpObPs0Q/640?wx_fmt=jpeg)

如上图所示，当我们向内存写入字符'a' 'b' 'c' 的时候，会自动转换为 ASCII 码进行存储，在最右侧可以找到刚刚写入的字符。

### Debug -u

如何向内存中写入一段机器码呢？比如我们想要在内存中写入一段机器码。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdGHTZKqwFPDZmHV164b0ML1v0XYyibxicdzyX9WjlQllBgXETL2WDv3X9A/640?wx_fmt=jpeg)

我们可以使用 -e 来进行写入，向内存中写入 b8 01 00 b9 02 00 01 c8 这个机器码，如下所示

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdGiansvvEvtia26RKBvFNiaHJib29UMfic7VRIibicibH8s51b0AKQYa2TBHyVNA/640?wx_fmt=jpeg)

我们使用 -e 写入之后，使用 -d 查看内存值，可以发现我们刚刚写入的值，但是却看不到机器码，所以机器码该如何看呢？

-u 输出的结果分为三部分显示：

*   最左侧是每一条机器指令的地址；
    
*   中间是机器指令；
    
*   最右侧是机器指令执行的汇编指令。
    

1000:0 处存放的是写入的机器码 B8 01 00 组成的机器指令，对应的汇编指令是 MOV AX,0001。

1000:0003 处存放的是写入的机器码 B9 02 00 组成的机器指令，对应的汇编指令是 MOV CX,0002。

1000:0006 处存放的是写入的机器码 C1 C8 所组成的机器指令，对应的汇编指令是 add ax,cx。

### Debug -t

上面介绍的一系列指令包括我们上面提到的 Debug -e 机器码都是向内存中进行写入，那么如何执行这些指令呢？

我们可以使用 Debug -t 来执行写入的指令。使用 Debug -t 可以执行由 CS:IP 指向的指令。

既然是 -t 能够执行从 CS:IP 指向的命令，所以我们有必要将 CS:IP 指向 1000:0（因为我们前面将指令写在了 1000:0 处）。

首先我们需要执行 -r cs 1000 ，-r ip 0 把 CS:IP 赋值为 1000:0。

然后执行 -t 指令，下图是已经执行过的指令截图。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdGm1hjOylNUe63iboz4wibLjxSK5rbbcHxaXTG7r9tLMBnLz86Q197NiaKA/640?wx_fmt=jpeg)

可以看到，执行完 -t 指令之后，MOV AX,0001 这条指令被执行，当前 AX 寄存器的内容变为了 0001，这条汇编指令的意思就是把 0001 移动到 AX 寄存器中。

继续执行 -t 之后，我们可以看到寄存器的变化。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdGmaiay2iaIUy0KYTe9o230Zx8JkgoDOBh8yxrLdkyaTHtwpThtl0U5K8w/640?wx_fmt=jpeg)

### Debug -a

毕竟机器指令不是那么好懂，写入很不方便，所以有没有办法能够支持我们直接写入汇编指令呢？还真有，Debug 提供了 -a 这种方式来实现汇编指令的写入。如下图所示

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdG5bqMoDuNGGOAKhMQJquSrzm9nyvf77hW5o7mGib9bEdEHdAVW5fhWEg/640?wx_fmt=jpeg)

可以看到，我们使用了 -a 命令来对 1000:0 进行写入，分别输入 mov ax,1 mov bx,2 mov cx,3 add ax,bx add ax,cx add ax,ax 指令，然后按回车进行确定执行。

我们使用 -d 1000:0 f 可以看到从偏移地址 0 处开始的第 f 个内存指令（因为最大写入的地址只是 f）。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaSBbC03gQ0CCqFopkLaibZdGgXtdHO4qyTvNTD2L9DYcSDEI3jnJv11bLLwbWCYL5T5J7icyaDRSnEg/640?wx_fmt=jpeg)

上图中的 1000:000F 为什么有值呢，因为我们上面已经执行过这个写入了。

另外，使用 -a 可以从一个预设的地址处开始输入指令。

总结
--

今天和大家聊了一下 Debug 的基本用法，主要包括

*   -r 查看、修改寄存器中的内容
    
*   -d 查看内存中的指令
    
*   -e 修改内存中的内容
    
*   -u 可以将内存中的内容解释为机器指令和对应的汇编指令
    
*   -t 执行 CS:IP 处的指令
    
*   -a 以汇编得形式向内存写入内容
    

汇编指令的选项有很多，上面介绍的这些属于经常用到的指令，这些指令要能够熟练使用。

- EOF -

推荐阅读  点击标题可跳转

1、[微软 Debug CRT 库是如何追踪 C++ 内存泄露的？](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651163272&idx=2&sn=392d0fcd76f77ed6a3c81e546510125c&chksm=80645bd7b713d2c187fb4960b362eff24498eb3977c1ae0cbb7f51ebe61384d9111a4701c6b7&scene=21#wechat_redirect)

2、[Windows 内存泄露分析之 DebugDialog](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651163196&idx=2&sn=987419106898e57662bedad30b095d6e&chksm=80645b63b713d27588889eb8f4d8623e44efbdb9327ce5fe5803aa6561ef0b61a8a5bb46c7ce&scene=21#wechat_redirect)

3、[Debug 模式和 Release 模式有什么区别？](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651162090&idx=1&sn=1a322defd1802ec2a2f05d5dbbb2635b&chksm=806454b5b713dda377940233393c25cb5dd4f9dab8323a20899394fbfc83848c007e165de9b1&scene=21#wechat_redirect)