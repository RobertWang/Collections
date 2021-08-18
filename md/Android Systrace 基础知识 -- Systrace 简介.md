> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIyNTY4NjU0OQ==&mid=2247508494&idx=2&sn=e2b1eecd99c815e50b75ed06127b83cc&chksm=e8790774df0e8e62d4db31190d84fa55abc77748551a3d92398e59614d50dbb37b1bd2fad642&mpshare=1&scene=1&srcid=0816v5TaBoqbcpGAMqooCfdz&sharer_sharetime=1629112619073&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

本文是 Systrace 系列文章的第一篇，主要是对 Systrace 进行简单介绍，介绍其简单使用方法；如何去看 Systrace；如何结合其他工具对 Systrace 中的现象进行分析。

本系列的目的是通过 Systrace 这个工具，从另外一个角度来看待 Android 系统整体的运行，同时也从另外一个角度来对 Framework 进行学习。也许你看了很多讲 Framework 的文章，但是总是记不住代码，或者不清楚其运行的流程，也许从 Systrace 这个图形化的角度，你可以理解的更深入一些。

系列文章目录
======

1.  Systrace 简介
    
2.  Systrace 基础知识 - Systrace 预备知识
    
3.  Systrace 基础知识 - Why 60 fps ？
    
4.  Systrace 基础知识 - SystemServer 解读
    
5.  Systrace 基础知识 - SurfaceFlinger 解读
    
6.  Systrace 基础知识 - Input 解读
    
7.  Systrace 基础知识 - Vsync 解读
    
8.  Systrace 基础知识 - Vsync-App ：基于 Choreographer 的渲染机制详解
    
9.  Systrace 基础知识 - MainThread 和 RenderThread 解读
    
10.  Systrace 基础知识 - Binder 和锁竞争解读
    
11.  Systrace 基础知识 - CPU Info 解读
    

正文
==

Systrace 是 Android4.1 中新增的性能数据采样和分析工具。它可帮助开发者收集 Android 关键子系统（如 SurfaceFlinger/SystemServer/Kernel/Input/Display 等 Framework 部分关键模块、服务，View 系统等）的运行信息，从而帮助开发者更直观的分析系统瓶颈，改进性能。

Systrace 的功能包括跟踪系统的 I/O 操作、内核工作队列、CPU 负载以及 Android 各个子系统的运行状况等。在 Android 平台中，它主要由 3 部分组成：

*   内核部分：Systrace 利用了 Linux Kernel 中的 ftrace 功能。所以，如果要使用 Systrace 的话，必须开启 kernel 中和 ftrace 相关的模块。
    
*   数据采集部分：Android 定义了一个 Trace 类。应用程序可利用该类把统计信息输出给 ftrace。同时，Android 还有一个 atrace 程序，它可以从 ftrace 中读取统计信息然后交给数据分析工具来处理。
    
*   数据分析工具：Android 提供一个 systrace.py（ python 脚本文件，位于 Android SDK 目录 / platform-tools/systrace 中，其内部将调用 atrace 程序）用来配置数据采集的方式（如采集数据的标签、输出文件名等）和收集 ftrace 统计数据并生成一个结果网页文件供用户查看。从本质上说，Systrace 是对 Linux Kernel 中 ftrace 的封装。应用进程需要利用 Android 提供的 Trace 类来使用 Systrace. 关于 Systrace 的官方介绍和使用可以看这里：Systrace
    

Systrace 简单使用
-------------

使用 Systrace 前，要先了解一下 Systrace 在各个平台上的使用方法，鉴于大家使用 Eclipse 和 Android Studio 的居多，所以直接摘抄官网关于这个的使用方法，不过不管是什么工具，流程是一样的：

*   手机准备好你要进行抓取的界面
    
*   点击开始抓取 (命令行的话就是开始执行命令)
    
*   手机上开始操作 (不要太长时间)
    
*   设定好的时间到了之后，会将生成 Trace.html 文件，使用 Chrome 将这个文件打开进行分析
    

一般抓到的 Systrace 文件如下

![](https://mmbiz.qpic.cn/mmbiz_png/HjA9ygCONWmGthOSnLCgTZ3BjQtf6KH2ChhYXayj4H38df50m2TnVgAB0Dp0ianH9W5Svu5XMfItvxmn7uefTzQ/640?wx_fmt=png)

### Command Line Usage

命令行形式比较灵活，速度也比较快，一次性配置好之后，以后再使用的时候就会很快就出结果（强烈推荐） Systrace 工具在 Android-SDK 目录下的 platform-tools 里面, 下面是简单的使用方法

```
$ cd android-sdk/platform-tools/systrace
$ python systrace.py
```

可以在 Bash 中配置好对应的路径和 Alias，使用起来还是很快速的。另外 User 版本所抓的 Systrce 文件所包含的信息, 是比 eng 版本或者 Userdebug 版本要少的, 建议使用 Userdebug 版本的机器来进行 debug, 这样既保证了性能, 又能有比较详细的输出结果.

抓取结束后，会生成对应的 Trace.html 文件，注意这个文件只能被 Chrome 打开。关于如何分析 Trace 文件，我们下面的章节会讲。不论使用那种工具，在抓取之前都可以选择参数，下面说一下这些参数的意思：

*   -h, --help Show the help message.（帮助）
    
*   -o Write the HTML trace report to the specified file.（即输出文件名，）
    
*   -t N, --time=N Trace activity for N seconds. The default value is 5 seconds. （Trace 抓取的时间，一般是 ：-t 8）
    
*   -b N, --buf-size=N Use a trace buffer size of N kilobytes. This option lets you limit the total size of the data collected during a trace.
    
*   -k
    
*   —ktrace= Trace the activity of specific kernel functions, specified in a comma-separated list.
    
*   -l, --list-categories List the available tracing category tags. The available tags are(下面的参数不用翻译了估计大家也看得懂，贴官方的解释也会比较权威，后面分析的时候我们会看到这些参数的作业的):
    

*   gfx - Graphics
    
*   input - Input
    
*   view - View
    
*   webview - WebView
    
*   wm - Window Manager
    
*   am - Activity Manager
    
*   audio - Audio
    
*   video - Video
    
*   camera - Camera
    
*   hal - Hardware Modules
    
*   res - Resource Loading
    
*   dalvik - Dalvik VM
    
*   rs - RenderScript
    
*   sched - CPU Scheduling
    
*   freq - CPU Frequency
    
*   membus - Memory Bus Utilization
    
*   idle - CPU Idle
    
*   disk - Disk input and output
    
*   load - CPU Load
    
*   sync - Synchronization Manager
    
*   workq - Kernel Workqueues Note: Some trace categories are not supported on all devices. Tip: If you want to see the names of tasks in the trace output, you must include the sched category in your command parameters.
    

*   -a
    
*   —app= Enable tracing for applications, specified as a comma-separated list of package names. The apps must contain tracing instrumentation calls from the Trace class. For more information, see Analyzing Display and Performance.
    
*   —link-assets Link to the original CSS or JavaScript resources instead of embedding them in the HTML trace report.
    
*   —from-file= Create the interactive Systrace report from a file, instead of running a live trace.
    
*   —asset-dir= Specify a directory for the trace report assets. This option is useful for maintaining a single set of assets for multiple Systrace reports.
    
*   -e
    
*   —serial= Conduct the trace on a specific connected device, identified by its device serial number. 上面的参数虽然比较多，但使用工具的时候不需考虑这么多，在对应的项目前打钩即可，命令行的时候才会去手动加参数：
    

我们一般会把这个命令配置成 Alias，配置如下：

```
alias st-start='python /sdk/platform-tools/systrace/systrace.py'  
alias st-start-gfx-trace = ‘st-start -t 8 gfx input view sched freq wm am hwui workq res dalvik sync disk load perf hal rs idle mmc’
```

这样在使用的时候，可以直接敲 st-start 即可，当然为了区分和保持各个文件，还需要加上 -o xxx.html . 上面的命令和参数不必一次就理解，只需要记住如何简单使用即可，在分析的过程中，这些东西都会慢慢熟悉的。

一般来说比较常用的是

1.  -o : 指示输出文件的路径和名字
    
2.  -t : 抓取时间
    
3.  -b : 指定 buffer 大小
    
4.  -a : 指定 app 包名  
    

**---END---**