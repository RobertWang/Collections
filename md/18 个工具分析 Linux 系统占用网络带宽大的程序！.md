> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666556780&idx=2&sn=d84342f8cc985a0bf32d0d4314b657fd&chksm=80dca9c7b7ab20d1fc52c0cc1331d6b3752fc2c286e2a0a26a8971338f1f9bcb44a84e81ed54&mpshare=1&scene=1&srcid=0818Cw5swlxTYGz5vpPn7Jv3&sharer_sharetime=1629291146378&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

```
来源：五月的仓颉
链接：cnblogs.com/xrq730/p/4865416.html
```

### 导读

本文介绍了一些可以用来监控网络使用情况的 Linux 命令行工具。这些工具可以监控通过网络接口传输的数据，并测量目前哪些数据所传输的速度。入站流量和出站流量分开来显示。  

一些命令可以显示单个进程所使用的带宽。这样一来，用户很容易发现过度使用网络带宽的某个进程。

这些工具使用不同的机制来制作流量报告。nload 等一些工具可以读取 "proc/net/dev" 文件，以获得流量统计信息；而一些工具使用 pcap 库来捕获所有数据包，然后计算总数据量，从而估计流量负载。

下面是按功能划分的命令名称。

*   监控总体带宽使用――nload、bmon、slurm、bwm-ng、cbm、speedometer 和 netload
    
*   监控总体带宽使用（批量式输出）――vnstat、ifstat、dstat 和 collectl
    
*   每个套接字连接的带宽使用――iftop、iptraf、tcptrack、pktstat、netwatch 和 trafshow
    
*   每个进程的带宽使用――nethogs
    

### 1、nload

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJylA7jhFHb5Y6ibcAwYZ2Jd8umPCvPp3BV9GgCXCeX8nMWATt7ibLQwUw/640?wx_fmt=jpeg)

nload 是一个命令行工具，让用户可以分开来监控入站流量和出站流量。它还可以绘制图表以显示入站流量和出站流量，视图比例可以调整。用起来很简单，不支持许多选项。

所以，如果你只需要快速查看总带宽使用情况，无需每个进程的详细情况，那么 nload 用起来很方便。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJPXVmBy7QDEu070GjmCExy6XQpxMlzw8cw89g2WaiaiaYo3aRU1BhqAJA/640?wx_fmt=jpeg)

安装 nload：Fedora 和 Ubuntu 在默认软件库里面就有 nload。CentOS 用户则需要从 Epel 软件库获得 nload。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJ331fUibCAY2VvESQW46lBzwk3To0iaBBwBFKOWFR1dAfxAfSPsIiaqH9Q/640?wx_fmt=jpeg)

### 2、iftop

iftop 可测量通过每一个套接字连接传输的数据；它采用的工作方式有别于 nload。iftop 使用 pcap 库来捕获进出网络适配器的数据包，然后汇总数据包大小和数量，搞清楚总的带宽使用情况。

虽然 iftop 报告每个连接所使用的带宽，但它无法报告参与某个套按字连接的进程名称 / 编号（ID）。不过由于基于 pcap 库，iftop 能够过滤流量，并报告由过滤器指定的所选定主机连接的带宽使用情况。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJDjhuh10fbh8ujOGncqgPC2Bt44w2eC9k4E6Ms84EL4AFcyWgG9PbUA/640?wx_fmt=jpeg)

n 选项可以防止 iftop 将 IP 地址解析成主机名，解析本身就会带来额外的网络流量。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJyxfTk1RHtNpbcZTaK2icN7fgj8dF1pJojYZ2pPVicTcVKWUa57tic2sGA/640?wx_fmt=jpeg)

安装 iftop：Ubuntu/Debian/Fedora 用户可以从默认软件库获得它。CentOS 用户可以从 Epel 获得它。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJBficFsXT0WxQMN1Gu5icjjl3UCeNBxyUgZkic8AsQLHP1uIdHXKzJU8AQ/640?wx_fmt=jpeg)

### 3、iptraf

iptraf 是一款交互式、色彩鲜艳的 IP 局域网监控工具。它可以显示每个连接以及主机之间传输的数据量。下面是屏幕截图。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJJT14LhNcujianV4tGPm9fxVUaiaAUEGicOiaYfSTXdTaqQxnkrX1iaXBKbg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJl09nogb6CqRSMRC6iaotic2XN21PeOYn1GQ8VfpmnzzbKYqCFvD7vshg/640?wx_fmt=jpeg)

安装 iptraf：

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJKCYGbNe22YmMDGDrc3w9pHgR5mtJ7d3OFhVvyo9icmGYnYa2bz3HJlw/640?wx_fmt=jpeg)

### 4、nethogs

nethogs 是一款小巧的 "net top" 工具，可以显示每个进程所使用的带宽，并对列表排序，将耗用带宽最多的进程排在最上面。万一出现带宽使用突然激增的情况，用户迅速打开 nethogs，就可以找到导致带宽使用激增的进程。nethogs 可以报告程序的进程编号（PID）、用户和路径。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJ83tq00YMYtHdCPDepcFZ7w1QEyib1yawya7BGCHY6PpGdHGP2sWdnDQ/640?wx_fmt=jpeg)

安装 nethogs：Ubuntu、Debian 和 Fedora 用户可以从默认软件库获得。CentOS 用户则需要 Epel。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJ9SWs2de0Yc7KWWA3iaSctGfL3zOYaicl9LRXmeSI3m7hEvHuL1LP39fg/640?wx_fmt=jpeg)

### 5. bmon

bmon（带宽监控器）是一款类似 nload 的工具，它可以显示系统上所有网络接口的流量负载。输出结果还含有图表和剖面，附有数据包层面的详细信息。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJlU8Dia3d5MEnajEEjo3VrnxRGkCz7OQCdUMVuB0I3Ed301JADicvE9aA/640?wx_fmt=jpeg)

安装 bmon：Ubuntu、Debian 和 Fedora 用户可以从默认软件库来安装。CentOS 用户则需要安装 repoforge，因为 Epel 里面没有 bmon。

### 6. slurm

slurm 是另一款网络负载监控器，可以显示设备的统计信息，还能显示 ASCII 图形。它支持三种不同类型的图形，使用 c 键、s 键和 l 键即可激活每种图形。slurm 功能简单，无法显示关于网络负载的任何更进一步的详细信息。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJWAOVL6zpyLIGFTSAiaQZnn1JOkRdSciaiagDiawqL3o7H8MdsK2KyYMJ5Q/640?wx_fmt=jpeg)

安装 slurm

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJNBz0p18PaibKAicog7rEhFItiaiczjrwve6QibC8X6xibibj4vibaDvhyib8LjA/640?wx_fmt=jpeg)

### 7. tcptrack

tcptrack 类似 iftop，使用 pcap 库来捕获数据包，并计算各种统计信息，比如每个连接所使用的带宽。它还支持标准的 pcap 过滤器，这些过滤器可用来监控特定的连接。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJKgVqR1FJCc6zx156WUibS6ddcamFe99uic0Iv0O6JQLtG1O50dwQo6Yw/640?wx_fmt=jpeg)

安装 tcptrack：Ubuntu、Debian 和 Fedora 在默认软件库里面就有它。CentOS 用户则需要从 RepoForge 获得它，因为 Epel 里面没有它。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJJg9CEDSSrxTWYBeWgVxFQPMAt2083p8sI9eNBxTE3RygLkHUr9Uic3g/640?wx_fmt=jpeg)

### 8. vnstat

vnstat 与另外大多数工具有点不一样。它实际上运行后台服务 / 守护进程，始终不停地记录所传输数据的大小。之外，它可以用来制作显示网络使用历史情况的报告

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJlr9DsTUibH9I85qycAeL7D9etZ5XibOjiapBiaPlkuktSEagcktTJFs9KQ/640?wx_fmt=jpeg)

运行没有任何选项的 vnstat，只会显示自守护进程运行以来所传输的数据总量。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJ83jkSM5jjWFtQz1DDT0QqTL301cVNtaAHBX6fcSSliaN0osqZnV1Fdw/640?wx_fmt=jpeg)

想实时监控带宽使用情况，请使用 "-l" 选项（实时模式）。然后，它会显示入站数据和出站数据所使用的总带宽量，但非常精确地显示，没有关于主机连接或进程的任何内部详细信息。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJ1jnVWRPaHHibTnQ6iaCpPwcuNZxcYIibALKFicrJDibCS5adTY2PxQ35LDA/640?wx_fmt=jpeg)

vnstat 更像是一款制作历史报告的工具，显示每天或过去一个月使用了多少带宽。它并不是严格意义上的实时监控网络的工具。

vnstat 支持许多选项，支持哪些选项方面的详细信息请参阅参考手册页。

安装 vnstat

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJtuYPwMowJCic3icEibm8betgGlW2f1uECgdhMyb7MMicYQRg42ZGsMb3DA/640?wx_fmt=jpeg)

### 9. bwm-ng

bwm-ng（下一代带宽监控器）是另一款非常简单的实时网络负载监控工具，可以报告摘要信息，显示进出系统上所有可用网络接口的不同数据的传输速度。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJFy1vicHzJ84nPWnL0GuGpBs5wm6UvibqFGVVO2oher8ibF5b3zcPxQfLw/640?wx_fmt=jpeg)

如果控制台足够大，bwm-ng 还能使用 curses2 输出模式，为流量绘制条形图。

安装 bwm-ng：在 CentOS 上，可以从 Epel 来安装 bwm-ng。

### 10. cbm：Color Bandwidth Meter

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJC01WFjKJJpag5azmNviazWD8TN1n09vAQRIN1ZXRg1nicc9ngWQob9Hg/640?wx_fmt=jpeg)

这是一款小巧简单的带宽监控工具，可以显示通过诸网络接口的流量大小。没有进一步的选项，仅仅实时显示和更新流量的统计信息。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJt3yjaLgTerHfLNjoxFKmyP4aeWCQtNGqEXWDH5MooUXHYG6txAicnCQ/640?wx_fmt=jpeg)

### 11. speedometer

这是另一款小巧而简单的工具，仅仅绘制外观漂亮的图形，显示通过某个接口传输的入站流量和出站流量。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJPeX1EGmhr5nmYVzOJ0jpkGoHKRSUVMD9oXHiaqlJMX01DIXMnJz3j4Q/640?wx_fmt=jpeg)

安装 speedometer

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJTVMFUXicuDvXKrzQOaibPUibYNHEmvl7UE4gRHxzHT1M7paTwjDN08ehA/640?wx_fmt=jpeg)

### 12. pktstat

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJvsDXhz02u3Qmy2ULl3Qg2jVR8JlZtg5icB0ELMib0ibQm1eMajQgJ6DJQ/640?wx_fmt=jpeg)

pktstat 可以实时显示所有活动连接，并显示哪些数据通过这些活动连接传输的速度。它还可以显示连接类型，比如 TCP 连接或 UDP 连接；如果涉及 HTTP 连接，还会显示关于 HTTP 请求的详细信息。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJPAAovxb6fqY9PEHKLnblT0obrzdh8NWcpwqd51jZPRnk048mcriactQ/640?wx_fmt=jpeg)

### 13. netwatch

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJWw02z9dNTmJF3myUNkdhvyRpqcxyaHwYiaoAPUVCcuCkoxJJGqXtZNw/640?wx_fmt=jpeg)

netwatch 是 netdiag 工具库的一部分，它也可以显示本地主机与其他远程主机之间的连接，并显示哪些数据在每个连接上所传输的速度。

### 14. trafshow

与 netwatch 和 pktstat 一样，trafshow 也可以报告当前活动连接、它们使用的协议以及每条连接上的数据传输速度。它能使用 pcap 类型过滤器，对连接进行过滤。

只监控 TCP 连接

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJYLrIqpBKmTJicIQibVcia8MeIiaZaqEBf26A2wpYicMrFWZB1DbfDk8ecfw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJLic22icxH5ZlrvqeZMzjpGr5icmv5mSVeuhCzaO14pWyJHsSlw09URASA/640?wx_fmt=jpeg)

### 15. netload

netload 命令只显示关于当前流量负载的一份简短报告，并显示自程序启动以来所传输的总字节量。没有更多的功能特性。它是 netdiag 的一部分。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJicADh3VPuVibNzXBTs2RgeXhFP1ThCvPXNIO2eibW8hpZhiaHIzpMWjsfg/640?wx_fmt=jpeg)

### 16. ifstat

ifstat 能够以批处理式模式显示网络带宽。输出采用的一种格式便于用户使用其他程序或实用工具来记入日志和分析。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJLnLyk5z60dLfiagFxWyNf5PFASf2YoXju2OYicKNy3FPY0FvRZEMaYSQ/640?wx_fmt=jpeg)

安装 ifstat：Ubuntu、Debian 和 Fedora 用户在默认软件库里面就有它。CentOS 用户则需要从 Repoforge 获得它，因为 Epel 里面没有它。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJRHcHtxzIdPWqBFaAYociaPOicrkJib18W0FHKIayQaDObNdQ3ibGwcUr1A/640?wx_fmt=jpeg)

### 17. dstat

dstat 是一款用途广泛的工具（用 python 语言编写），它可以监控系统的不同统计信息，并使用批处理模式来报告，或者将相关数据记入到 CSV 或类似的文件。这个例子显示了如何使用 dstat 来报告网络带宽。

安装 dstat

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJhRVoy6WzicUo6kliaqPo6siaviaicibwlbyacs6h6Xsob6N1OgyrCkPZzvjQ/640?wx_fmt=jpeg)

### 18. collectl

collectl 以一种类似 dstat 的格式报告系统的统计信息；与 dstat 一样，它也收集关于系统不同资源（如处理器、内存和网络等）的统计信息。这里给出的一个简单例子显示了如何使用 collectl 来报告网络使用 / 带宽。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPo9RTGHW5RtPkY5QIRzSvuJQzj7v2VbdkWavMETktEZmlTk9F9y5YtQg6vdxvs3p3acKiagGr3WYqg/640?wx_fmt=jpeg)

结束语：上述几个使用方便的命令可以迅速检查 Linux 服务器上的网络带宽使用情况。不过，这些命令需要用户通过 SSH 登录到远程服务器。另外，基于 Web 的监控工具也可以用来实现同样的任务。

ntop 和 darkstat 是面向 Linux 系统的其中两个基本的基于 Web 的网络监控工具。除此之外还有企业级监控工具，比如 nagios，它们提供了一批功能特性，不仅仅可以监控服务器，还能监控整个基础设施。

- EOF -