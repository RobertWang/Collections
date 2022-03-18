> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651168462&idx=1&sn=a1cd029f42f49f360e63fb671222933a&chksm=80644f91b713c687f5c9a06ae366f82e37d895c1d7087dd32dcdc2725c425034835c04b6e8d7&scene=21#wechat_redirect)

0. 引言：
------

*   1、free 命令
    
*   2、 vmstat 命令
    
*   3、 /proc/meminfo 命令
    
*   4、 top 命令
    
*   5、 htop 命令
    
*   6、查看进程内存信息
    

### Linux 内存总览图

![](https://mmbiz.qpic.cn/mmbiz_png/icRxcMBeJfcibkWulcpw1BI3wVKicIHbSnuxdOrr3XFXeRXT8wGfN56foQOlAuXJpk3icHDKX736iagTs06PQ2ZiaiaAA/640?wx_fmt=png)

该图很好的描述了 OS 内存的使用和分配等详细信息。建议大家配合该图来一起学习和理解内存的一些概念。

一、free 命令
---------

free 命令可以显示当前系统未使用的和已使用的内存数目，还可以显示被内核使用的内存缓冲区。

### 1. free 命令语法：

```
free [options]
```

free 命令选项：

```
-b # 以Byte为单位显示内存使用情况；
-k # 以KB为单位显示内存使用情况；
-m # 以MB为单位显示内存使用情况；
-g # 以GB为单位显示内存使用情况。 
-o # 不显示缓冲区调节列；
-s<间隔秒数> # 持续观察内存使用状况；
-t # 显示内存总和列；
-V # 显示版本信息。
```

### 2. free 命令实例

```
free -t    # 以总和的形式显示内存的使用信息
free -h -s 10 # 周期性的查询内存使用信息，每10s 执行一次命令

free -h -c 10 #输出10次
  在版本 v3.2.8，就是输出一次！需要配合 -s 使用。
  在版本 v3.3.10，不加-s，就默认1秒输出一次。
free -V #查看版本号
```

![](https://mmbiz.qpic.cn/mmbiz_png/icRxcMBeJfcibkWulcpw1BI3wVKicIHbSnu9hok05Aib4ssV6gKZpvsW95iacDlPHJxEUYO35Keibicmnc5UnMrTK9SkQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icRxcMBeJfcibkWulcpw1BI3wVKicIHbSnu9hok05Aib4ssV6gKZpvsW95iacDlPHJxEUYO35Keibicmnc5UnMrTK9SkQ/640?wx_fmt=png)

<table><thead><tr data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-darkmode-bgcolor-16475451737574="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); min-width: 85px;">内容</th><th data-darkmode-bgcolor-16475451737574="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: left;">含义</th></tr></thead><tbody><tr data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">Mem</td><td data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">行 (第二行) 是内存的使用情况</td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">Swap</td><td data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">行 (第三行) 是交换空间的使用情况</td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">total</td><td data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">总可用物理内存。一般是总物理内存除去一些预留的和操作系统本身的内存占用，是操作系统可以支配的内存大小。这个在 v3.2.8 和 v3.3.10 一样。这个值是 / proc/meminfo 中 MemTotal 的值。</td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">used</td><td data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">列显示已经被使用的物理内存和交换空间。在 v3.2.8, 这个值是 (total - free) 得出来的。可以说是系统已经被系统分配，但是实际并不一定正在被真正的使用，其空间可以被回收再分配的。在 v3.3.10, 这个值是 (total - free - cache - buffers) 得出来的，是真正目前正在被使用的内存。</td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">free</td><td data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">系统还未使用的物理内存。这个值是 / proc/meminfo 中 MemFree 的值</td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">shared</td><td data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">共享内存的空间。这个值是 / proc/meminfo 中 Shmem 的值</td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">buff/cache</td><td data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">列显示被 buffer 和 cache 使用的物理内存大小</td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">available</td><td data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">v3.3.10 中的项。看起来这个值是可以使用的内存，不过 (available + used) &lt; total，也就是 available &lt; (free + cache + buffers)。而在 v3.2.8 中(free + cache + buffers) 是一般认为的可用内存，既然在新版本中有这个 available 数据，应该是更准确的吧。毕竟并不是所有的未使用的内存就一定是可用的。这个值是取的 / proc/meminfo 中 MemAvailable 的值，如果 meminfo 中没有这个值，会依据 meminfo 中的 Active(file),Inactive(file),MemFree,SReclaimable 等值计算一个。</td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">-/+ buffers/cache</td><td data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">v3.2.8 有这一行，v3.3.10 没有。其中，used 这一项是 (used - buffers - cached) 的值，即 (total - free - buffers - cached) 的值, 是真正在使用的内存的值。free 这一项是 (free + buffers + cached) 的值，是真正未使用的内存的值。个人觉得有 -/+ buffers/cache，这一栏看的挺习惯。。不过新版本 v3.3.10 的 used 更明确。相信有不少人和我一样，刚看到 v3.2.8 里面的 used 占了这么多内存的时候，有点摸不着头脑。</td></tr></tbody></table>

二、vmstat 指令
-----------

vmstat 命令是最常见的 Linux/Unix 监控工具，用于查看系统的内存存储信息，是一个报告虚拟内存统计信息的小工具，属于 sysstat 包。

vmstat 命令报告包括：**进程、内存、分页、阻塞 IO、中断、磁盘、CPU**。

可以展现给定时间间隔的服务器的状态值, 包括服务器的 CPU 使用率，内存使用，虚拟内存交换情况, IO 读写情况。

这个命令是我查看 Linux/Unix 最喜爱的命令，一个是 Linux/Unix 都支持，二是相比 top，我可以看到整个机器的 CPU, 内存, IO 的使用情况，而不是单单看到各个进程的 CPU 使用率和内存使用率 (使用场景不一样)。

### 1. 命令格式：

```
vmstat -s(参数)
```

### 2. 举例

一般 vmstat 工具的使用是通过两个数字参数来完成的，第一个参数是采样的时间间隔数，单位是秒，第二个参数是采样的次数，如:

```
 root@local:~# vmstat 2 1
procs -----------memory---------- ---swap-- -----io---- -system-- ----cpu----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa
 1  0      0 3498472 315836 3819540    0    0     0     1    2    0  0  0 100  0
```

2 表示每个两秒采集一次服务器状态，1 表示只采集一次。

实际上，在应用过程中，我们会在一段时间内一直监控，不想监控直接结束 vmstat 就行了, 例如:

![](https://mmbiz.qpic.cn/mmbiz_png/icRxcMBeJfcibkWulcpw1BI3wVKicIHbSnuJE8T7a9dCOsYb3TiaZwxj2JRDPq9fxF5r6etEpeh5JMAkFxicqUcg6og/640?wx_fmt=png)

这表示 vmstat 每 2 秒采集数据，按下 **ctrl + c** 结束程序，这里采集了 3 次数据我就结束了程序。

<table cellpadding="0" cellspacing="0" data-tool="mdnice编辑器"><tbody><tr data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)"><strong data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">类别</strong></p></td><td data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)"><strong data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">项目</strong></p></td><td width="197" data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)"><strong data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">含义</strong></p></td><td width="505" data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)"><strong data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">说明</strong></p></td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td rowspan="2" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)"><strong data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">Procs</strong><strong data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">（进程）</strong></p></td><td data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">r</p></td><td width="197" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)"><strong data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">等待执行的任务数</strong></p></td><td width="505" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)"><strong data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">展示了正在执行和等待 cpu 资源的任务个数。当这个值超过了 cpu 个数，就会出现 cpu 瓶颈。</strong></p></td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">B</p></td><td width="197" data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">等待 IO 的进程数量</p></td><td width="505" data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><br data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)"></td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td rowspan="6" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)"><strong data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">Memory(</strong><strong data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">内存</strong><strong data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">)</strong></p></td><td data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">swpd</p></td><td width="197" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">正在使用虚拟的内存大小，单位 k</p></td><td width="505" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><br data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)"></td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">free</p></td><td width="197" data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">空闲内存大小</p></td><td width="505" data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><br data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)"></td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">buff</p></td><td width="197" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">已用的 buff 大小，对块设备的读写进行缓冲</p></td><td width="505" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><br data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)"></td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">cache</p></td><td width="197" data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">已用的 cache 大小，文件系统的 cache</p></td><td width="505" data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><br data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)"></td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">inact</p></td><td width="197" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">非活跃内存大小，即被标明可回收的内存，区别于 free 和 active</p></td><td width="505" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">具体含义见：概念补充（当使用 - a 选项时显示）</p></td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">active</p></td><td width="197" data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">活跃的内存大小</p></td><td width="505" data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">具体含义见：概念补充（当使用 - a 选项时显示）</p></td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td rowspan="2" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)"><strong data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">Swap</strong></p></td><td data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">si</p></td><td width="197" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">每秒从交换区写入内存的大小（单位：kb/s）</p></td><td width="505" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><br data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)"></td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">so</p></td><td width="197" data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">每秒从内存写到交换区的大小</p></td><td width="505" data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><br data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)"></td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td rowspan="2" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)"><strong data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">IO</strong></p></td><td data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">bi</p></td><td width="197" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">每秒读取的块数（读磁盘）</p></td><td width="505" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">块设备每秒接收的块数量，单位是 block，这里的块设备是指系统上所有的磁盘和其他块设备，现在的 Linux 版本块的大小为 1024bytes</p></td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">bo</p></td><td width="197" data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">每秒写入的块数（写磁盘）</p></td><td width="505" data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">块设备每秒发送的块数量，单位是 block</p></td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td rowspan="2" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)"><strong data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">system</strong></p></td><td data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">in</p></td><td width="197" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)"><strong data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">每秒中断数，包括时钟中断</strong></p></td><td rowspan="2" width="505" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)"><strong data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">这两个值越大，会看到由内核消耗的 cpu 时间 sy 会越多</strong></p><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)"><strong data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">秒上下文切换次数，例如我们调用系统函数，就要进行上下文切换，线程的切换，也要进程上下文切换，这个值要越小越好，太大了，<span data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">要考虑调低线程或者进程的数目</span></strong></p></td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">cs</p></td><td width="197" data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)"><strong data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">每秒上下文切换数</strong></p></td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td rowspan="4" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)"><strong data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">CPU</strong><strong data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">（以百分比表示）</strong></p></td><td data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">us</p></td><td width="197" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">用户进程执行消耗 cpu 时间 (user time)</p></td><td width="505" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">us 的值比较高时，说明用户进程消耗的 cpu 时间多，<strong data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">但是如果长期超过 50% 的使用，那么我们就该考虑优化程序算法或其他措施了</strong></p></td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">sy</p></td><td width="197" data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">系统进程消耗 cpu 时间 (system time)</p></td><td width="505" data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">sys 的值过高时，说明系统内核消耗的 cpu 资源多，这个不是良性的表现，我们应该检查原因。<strong data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">这里 us + sy 的参考值为 80%，如果 us+sy 大于 80% 说明可能存在 CPU 不足</strong></p></td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">Id</p></td><td width="197" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">空闲时间 (包括 IO 等待时间)</p></td><td width="505" data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(248, 248, 248)">一般来说 us+sy+id=100</p></td></tr><tr data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">wa</p></td><td width="197" data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">等待 IO 时间</p></td><td width="505" data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; vertical-align: top;"><p data-darkmode-bgcolor-16475451737574="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475451737574="#fff|rgb(255,255,255)">wa 过高时，说明 io 等待比较严重，这可能是由于磁盘大量随机访问造成的，也有可能是磁盘的带宽出现瓶颈。</p></td></tr></tbody></table>

### 3. 常见问题处理

**常见问题及解决方法**

1.  如果 r 经常大于 4，且 id 经常少于 40，表示 cpu 的负荷很重。
    
2.  如果 pi，po 长期不等于 0，表示内存不足。
    
3.  如果 disk 经常不等于 0，且在 b 中的队列大于 3，表示 io 性能不好。
    

*   1. 如果在 processes 中运行的序列 (process r) 是连续的大于在系统中的 CPU 的个数表示系统现在运行比较慢, 有多数的进程等待 CPU。
    
*   2. 如果 r 的输出数大于系统中可用 CPU 个数的 4 倍的话, 则系统面临着 CPU 短缺的问题, 或者是 CPU 的速率过低, 系统中有多数的进程在等待 CPU, 造成系统中进程运行过慢。
    
*   3. 如果空闲时间 (cpu id) 持续为 0 并且系统时间 (cpu sy) 是用户时间的两倍 (cpu us) 系统则面临着 CPU 资源的短缺。
    

当发生以上问题的时候请先调整应用程序对 CPU 的占用情况. 使得应用程序能够更有效的使用 CPU. 同时可以考虑增加更多的 CPU.  关于 CPU 的使用情况还可以结合 mpstat,  ps aux top  prstat –a 等等一些相应的命令来综合考虑关于具体的 CPU 的使用情况, 和那些进程在占用大量的 CPU 时间. 一般情况下，应用程序的问题会比较大一些. 比如一些 sql 语句不合理等等都会造成这样的现象.

### 4. 内存问题现象:

内存的瓶颈是由 scan rate (sr) 来决定的. scan rate 是通过每秒的始终算法来进行页扫描的. 如果 scan rate(sr) 连续的大于每秒 200 页则表示可能存在内存缺陷. 同样的如果 page 项中的 pi 和 po 这两栏表示每秒页面的调入的页数和每秒调出的页数. 如果该值经常为非零值, 也有可能存在内存的瓶颈, 当然, 如果个别的时候不为 0 的话, 属于正常的页面调度这个是虚拟内存的主要原理.

解决办法:

*   1. 调节 applications & servers 使得对内存和 cache 的使用更加有效.
    
*   2. 增加系统的内存.
    
*   3.Implement priority paging in s in pre solaris 8 versions by adding line "set priority paging=1" in /etc/system. Remove this line if upgrading from Solaris 7 to 8 & retaining old /etc/system file.
    

关于内存的使用情况还可以结 ps aux top  prstat –a 等等一些相应的命令来综合考虑关于具体的内存的使用情况, 和那些进程在占用大量的内存.

一般情况下，如果内存的占用率比较高, 但是, CPU 的占用很低的时候, 可以考虑是有很多的应用程序占用了内存没有释放, 但是, 并没有占用 CPU 时间, 可以考虑应用程序, 对于未占用 CPU 时间和一些后台的程序, 释放内存的占用。

r 表示运行队列 (就是说多少个进程真的分配到 CPU)，我测试的服务器目前 CPU 比较空闲，没什么程序在跑，当这个值超过了 CPU 数目，就会出现 CPU 瓶颈了。

这个也和 top 的负载有关系，**一般负载超过了 3 就比较高，超过了 5 就高，超过了 10 就不正常了**，服务器的状态很危险。

top 的负载类似每秒的运行队列。如果运行队列过大，表示你的 CPU 很繁忙，一般会造成 CPU 使用率很高。

### 5. 常见性能问题分析

IO/CPU/men 连锁反应

```
1.free急剧下降
2.buff和cache被回收下降，但也无济于事
3.依旧需要使用大量swap交换分区swpd
4.等待进程数，b增多
5.读写IO，bi bo增多
6.si so大于0开始从硬盘中读取
7.cpu等待时间用于 IO等待，wa增加
```

内存不足

```
1.开始使用swpd，swpd不为0
2.si so大于0开始从硬盘中读取
```

io 瓶颈

```
1.读写IO，bi bo增多超过2000
2.cpu等待时间用于 IO等待，wa增加 超过20
3.sy 系统调用时间长，IO操作频繁会导致增加 >30%
4.wa io等待时间长
    iowait% <20%            良好
    iowait% <35%            一般
    iowait% >50%
5.进一步使用iostat观察
```

CPU 瓶颈：load,vmstat 中 r 列

```
    1.反应为CPU队列长度
    2.一段时间内，CPU正在处理和等待CPU处理的进程数之和，直接反应了CPU的使用和申请情况。
    3.理想的load average：核数*CPU数*0.7
        CPU个数：grep 'physical id' /proc/cpuinfo | sort -u
        核数：grep 'core id' /proc/cpuinfo | sort -u | wc -l
    4.超过这个值就说明已经是CPU瓶颈了
```

三、/proc/meminfo
---------------

用途：用于从 / proc 文件系统中提取与内存相关的信息。这些文件包含有 系统和内核的内部信息。其实 free 命令中的信息都来自于 /proc/meminfo 文件。/proc/meminfo 文件包含了更多更原始的信息，只是看起来不太直观。

### 1. 查看方法：

```
cat /proc/meminfo
```

### 2. 实例及信息解释

```
peng@ubuntu:~$ cat /proc/meminfo
MemTotal:        2017504 kB //所有可用的内存大小，
物理内存减去预留位和内核使用。系统从加电开始到引导完成，firmware/BIOS要预留一
些内存，内核本身要占用一些内存，最后剩下可供内核支配的内存就是MemTotal。这个值
在系统运行期间一般是固定不变的，重启会改变。
MemFree:          511052 kB //表示系统尚未使用的内存。
MemAvailable:     640336 kB //真正的系统可用内存，
系统中有些内存虽然已被使用但是可以回收的，比如cache/buffer、slab都有一部分可
以回收，所以这部分可回收的内存加上MemFree才是系统可用的内存
Buffers:          114348 kB //用来给块设备做缓存的内存，(文件系统的 metadata、pages)
Cached:           162264 kB //分配给文件缓冲区的内存,例如vi一个文件，就会将未保存的内容写到该缓冲区
SwapCached:         3032 kB //被高速缓冲存储用的交换空间（硬盘的swap）的大小
Active:           555484 kB //经常使用的高速缓冲存储器页面文件大小
Inactive:         295984 kB //不经常使用的高速缓冲存储器文件大小
Active(anon):     381020 kB //活跃的匿名内存
Inactive(anon):   244068 kB //不活跃的匿名内存
Active(file):     174464 kB //活跃的文件使用内存
Inactive(file):    51916 kB //不活跃的文件使用内存
Unevictable:          48 kB //不能被释放的内存页
Mlocked:              48 kB //系统调用 mlock 
SwapTotal:        998396 kB //交换空间总内存
SwapFree:         843916 kB //交换空间空闲内存
Dirty:               128 kB //等待被写回到磁盘的
Writeback:             0 kB //正在被写回的
AnonPages:        572776 kB //未映射页的内存/映射到用户空间的非文件页表大小
Mapped:           119816 kB //映射文件内存
Shmem:             50212 kB //已经被分配的共享内存
Slab:             113700 kB  //内核数据结构缓存
SReclaimable:      68652 kB //可收回slab内存
SUnreclaim:        45048 kB //不可收回slab内存
KernelStack:        8812 kB //内核消耗的内存
PageTables:        27428 kB //管理内存分页的索引表的大小
NFS_Unstable:          0 kB //不稳定页表的大小
Bounce:                0 kB //在低端内存中分配一个临时buffer作为跳转，把位
于高端内存的缓存数据复制到此处消耗的内存
WritebackTmp:          0 kB //FUSE用于临时写回缓冲区的内存
CommitLimit:     2007148 kB //系统实际可分配内存
Committed_AS:    3567280 kB //系统当前已分配的内存
VmallocTotal:   34359738367 kB //预留的虚拟内存总量
VmallocUsed:           0 kB //已经被使用的虚拟内存
VmallocChunk:          0 kB //可分配的最大的逻辑连续的虚拟内存
HardwareCorrupted:     0 kB //表示“中毒页面”中的内存量
即has failed的内存(通常由ECC标记). ECC代表“纠错码”. ECC memory能够纠正小错误并检测较大错误;
在具有非ECC内存的典型PC上,内存错误未被检测到.如果使用ECC检测到无法纠正的错误(在内存或缓存中,
具体取决于系统的硬件支持),则Linux内核会将相应的页面标记为中毒.
AnonHugePages:         0 kB //匿名大页
【/proc/meminfo的AnonHugePages==所有进程的/proc/<pid>/smaps中AnonHugePages之和】
ShmemHugePages:        0 kB  //用于共享内存的大页
ShmemPmdMapped:        0 kB
CmaTotal:              0 kB //连续内存区管理总量
CmaFree:               0 kB //连续内存区管理空闲量
HugePages_Total:       0    //预留HugePages的总个数
HugePages_Free:        0    //池中尚未分配的 HugePages 数量，
真正空闲的页数等于HugePages_Free - HugePages_Rsvd
HugePages_Rsvd:        0    //表示池中已经被应用程序分配但尚未使用的 HugePages 数量
HugePages_Surp:        0    //这个值得意思是当开始配置了20个大页，现在修改配置为16，那么这个参数就会显示为4，一般不修改配置，这个值都是0
Hugepagesize:       2048 kB //大内存页的size
//指直接映射(direct mapping)的内存大小，从代码上来看，值记录管理页表占用的内存,就是描述线性映射空间中，有多个空间分别使用了2M/4K/1G页映射
DirectMap4k:       96128 kB
DirectMap2M:     2000896 kB 
DirectMap1G:           0 kB
```

> 1.  注意这个文件显示的单位是 kB 而不是 KB，1kB=1000B, 但是实际上应该是 KB，1KB=1024B
>     
> 2.  还可以使用命令 less /proc/meminfo 直接读取该文件。通过使用 less 命令，可以在长长的输出中向上和向下滚动，找到你需要的内容。
>     

从中我们可以很清晰明了的看出内存中的各种指标情况，例如 MemFree 的空闲内存和 SwapFree 中的交换内存。

### 3. 代码实例

负责输出 / proc/meminfo 的源代码是：

```
fs/proc/meminfo.c : meminfo_proc_show()
```

```
static int meminfo_proc_show(struct seq_file *m, void *v)
{
 struct sysinfo i;
 unsigned long committed;
 long cached;
 long available;
 unsigned long pages[NR_LRU_LISTS];
 int lru;
 
 si_meminfo(&i);
 si_swapinfo(&i);
 committed = percpu_counter_read_positive(&vm_committed_as);
 
 cached = global_node_page_state(NR_FILE_PAGES) -
   total_swapcache_pages() - i.bufferram;
 if (cached < 0)
  cached = 0;
 
 for (lru = LRU_BASE; lru < NR_LRU_LISTS; lru++)
  pages[lru] = global_node_page_state(NR_LRU_BASE + lru);
 
 available = si_mem_available();
 
 show_val_kb(m, "MemTotal:       ", i.totalram);
 show_val_kb(m, "MemFree:        ", i.freeram);
 show_val_kb(m, "MemAvailable:   ", available);
 show_val_kb(m, "Buffers:        ", i.bufferram);
 show_val_kb(m, "Cached:         ", cached);
 show_val_kb(m, "SwapCached:     ", total_swapcache_pages());
 show_val_kb(m, "Active:         ", pages[LRU_ACTIVE_ANON] +
        pages[LRU_ACTIVE_FILE]);
 show_val_kb(m, "Inactive:       ", pages[LRU_INACTIVE_ANON] +
        pages[LRU_INACTIVE_FILE]);
 show_val_kb(m, "Active(anon):   ", pages[LRU_ACTIVE_ANON]);
 show_val_kb(m, "Inactive(anon): ", pages[LRU_INACTIVE_ANON]);
 show_val_kb(m, "Active(file):   ", pages[LRU_ACTIVE_FILE]);
 show_val_kb(m, "Inactive(file): ", pages[LRU_INACTIVE_FILE]);
 show_val_kb(m, "Unevictable:    ", pages[LRU_UNEVICTABLE]);
 show_val_kb(m, "Mlocked:        ", global_zone_page_state(NR_MLOCK));
 
#ifdef CONFIG_HIGHMEM
 show_val_kb(m, "HighTotal:      ", i.totalhigh);
 show_val_kb(m, "HighFree:       ", i.freehigh);
 show_val_kb(m, "LowTotal:       ", i.totalram - i.totalhigh);
 show_val_kb(m, "LowFree:        ", i.freeram - i.freehigh);
#endif
 
#ifndef CONFIG_MMU
 show_val_kb(m, "MmapCopy:       ",
      (unsigned long)atomic_long_read(&mmap_pages_allocated));
#endif
 
 show_val_kb(m, "SwapTotal:      ", i.totalswap);
 show_val_kb(m, "SwapFree:       ", i.freeswap);
 show_val_kb(m, "Dirty:          ",
      global_node_page_state(NR_FILE_DIRTY));
 show_val_kb(m, "Writeback:      ",
      global_node_page_state(NR_WRITEBACK));
 show_val_kb(m, "AnonPages:      ",
      global_node_page_state(NR_ANON_MAPPED));
 show_val_kb(m, "Mapped:         ",
      global_node_page_state(NR_FILE_MAPPED));
 show_val_kb(m, "Shmem:          ", i.sharedram);
 show_val_kb(m, "Slab:           ",
      global_node_page_state(NR_SLAB_RECLAIMABLE) +
      global_node_page_state(NR_SLAB_UNRECLAIMABLE));
 
 show_val_kb(m, "SReclaimable:   ",
      global_node_page_state(NR_SLAB_RECLAIMABLE));
 show_val_kb(m, "SUnreclaim:     ",
      global_node_page_state(NR_SLAB_UNRECLAIMABLE));
 seq_printf(m, "KernelStack:    %8lu kB\n",
     global_zone_page_state(NR_KERNEL_STACK_KB));
 show_val_kb(m, "PageTables:     ",
      global_zone_page_state(NR_PAGETABLE));
#ifdef CONFIG_QUICKLIST
 show_val_kb(m, "Quicklists:     ", quicklist_total_size());
#endif
 
 show_val_kb(m, "NFS_Unstable:   ",
      global_node_page_state(NR_UNSTABLE_NFS));
 show_val_kb(m, "Bounce:         ",
      global_zone_page_state(NR_BOUNCE));
 show_val_kb(m, "WritebackTmp:   ",
      global_node_page_state(NR_WRITEBACK_TEMP));
 show_val_kb(m, "CommitLimit:    ", vm_commit_limit());
 show_val_kb(m, "Committed_AS:   ", committed);
 seq_printf(m, "VmallocTotal:   %8lu kB\n",
     (unsigned long)VMALLOC_TOTAL >> 10);
 show_val_kb(m, "VmallocUsed:    ", 0ul);
 show_val_kb(m, "VmallocChunk:   ", 0ul);
 
#ifdef CONFIG_MEMORY_FAILURE
 seq_printf(m, "HardwareCorrupted: %5lu kB\n",
     atomic_long_read(&num_poisoned_pages) << (PAGE_SHIFT - 10));
#endif
 
#ifdef CONFIG_TRANSPARENT_HUGEPAGE
 show_val_kb(m, "AnonHugePages:  ",
      global_node_page_state(NR_ANON_THPS) * HPAGE_PMD_NR);
 show_val_kb(m, "ShmemHugePages: ",
      global_node_page_state(NR_SHMEM_THPS) * HPAGE_PMD_NR);
 show_val_kb(m, "ShmemPmdMapped: ",
      global_node_page_state(NR_SHMEM_PMDMAPPED) * HPAGE_PMD_NR);
#endif
 
#ifdef CONFIG_CMA
 show_val_kb(m, "CmaTotal:       ", totalcma_pages);
 show_val_kb(m, "CmaFree:        ",
      global_zone_page_state(NR_FREE_CMA_PAGES));
#endif
 
 hugetlb_report_meminfo(m);
 
 arch_report_meminfo(m);
 
 return 0;
}
```

四、top 指令
--------

用途：用于打印系统中的 CPU 和内存使用情况。输出结果中，可以很清晰的看出已用和可用内存的资源情况。top 最好的地方之一就是发现可能已经失控的服务的进程 ID 号（PID）。有了这些 PID，你可以对有问题的任务进行故障排除（或 kill）。

### 语法

```
top [-] [d delay] [q] [c] [S] [s] [i] [n] [b]
```

### 参数说明：

```
d : 改变显示的更新速度，或是在交谈式指令列( interactive command)按 s
q : 没有任何延迟的显示速度，如果使用者是有 superuser 的权限，则 top 将会以最高的优先序执行
c : 切换显示模式，共有两种模式，一是只显示执行档的名称，另一种是显示完整的路径与名称
S : 累积模式，会将己完成或消失的子进程 ( dead child process ) 的 CPU time 累积起来
s : 安全模式，将交谈式指令取消, 避免潜在的危机
i : 不显示任何闲置 (idle) 或无用 (zombie) 的进程
n : 更新的次数，完成后将会退出 top
b : 批次档模式，搭配 "n" 参数一起使用，可以用来将 top 的结果输出到档案内
```

举例
--

![](https://mmbiz.qpic.cn/mmbiz_png/icRxcMBeJfcibkWulcpw1BI3wVKicIHbSnuzUdw1ToJia55G99s1kJ0pH0UNP1gn6WR5XOgylN9Mvsmax88fTIHobQ/640?wx_fmt=png)

第一行，任务队列信息，同 uptime 命令的执行结果

> 系统时间：02:19:10 运行时间：up 2:26 min, 当前登录用户：1 user 负载均衡 (uptime)  load average: 0.00, 0.06, 0.07average 后面的三个数分别是 1 分钟、5 分钟、15 分钟的负载情况。load average 数据是每隔 5 秒钟检查一次活跃的进程数，然后按特定算法计算出的数值。如果这个数除以逻辑 CPU 的数量，结果高于 5 的时候就表明系统在超负荷运转了

第二行，Tasks — 任务（进程）

> 总进程: 229 total, 运行: 1 running, 休眠: 163 sleeping, 停止: 0 stopped, 僵尸进程: 0 zombie

第三行，cpu 状态信息

> 0.7%us【user space】— 用户空间占用 CPU 的百分比。1.0%sy【sysctl】— 内核空间占用 CPU 的百分比。0.0%ni【】— 改变过优先级的进程占用 CPU 的百分比 97.9%id【idolt】— 空闲 CPU 百分比 0.3%wa【wait】— IO 等待占用 CPU 的百分比 0.0%hi【Hardware IRQ】— 硬中断占用 CPU 的百分比 0.0%si【Software Interrupts】— 软中断占用 CPU 的百分比

第四行, 内存状态

> 2017504 total,   653616 free,  1154200 used,   209688 buff/cache【缓存的内存量】

第五行，swap 交换分区信息

> 998396 total,   771068 free,   227328 used.   635608 avail Mem

第七行以下：各进程（任务）的状态监控

> PID — 进程 idUSER — 进程所有者 PR — 进程优先级 NI — nice 值。负值表示高优先级，正值表示低优先级 VIRT — 进程使用的虚拟内存总量，单位 kb。VIRT=SWAP+RESRES — 进程使用的、未被换出的物理内存大小，单位 kb。RES=CODE+DATASHR — 共享内存大小，单位 kbS —进程状态。D = 不可中断的睡眠状态 R = 运行 S = 睡眠 T = 跟踪 / 停止 Z = 僵尸进程 %CPU — 上次更新到现在的 CPU 时间占用百分比 %MEM — 进程使用的物理内存百分比 TIME+ — 进程使用的 CPU 时间总计，单位 1/100 秒 COMMAND — 进程名称（命令名 / 命令行）

### 常用实例

*   显示进程信息
    

```
# top
```

*   显示完整命令
    

```
# top -c
```

*   以批处理模式显示程序信息
    

```
# top -b
```

*   以累积模式显示程序信息
    

```
# top -S
```

*   设置信息更新次数
    

```
top -n 2
```

// 表示更新两次后终止更新显示

*   设置信息更新时间
    

```
# top -d 3
```

// 表示更新周期为 3 秒

*   显示指定的进程信息
    

```
# top -p 139
```

// 显示进程号为 139 的进程信息，CPU、内存占用率等

*   显示更新十次后退出
    

```
top -n 10
```

五、htop 指令
---------

htop 它类似于 top 命令，但可以让你在垂直和水平方向上滚动，所以你可以看到系统上运行的所有进程，以及他们完整的命令行。

可以不用输入进程的 PID 就可以对此进程进行相关的操作 (killing, renicing)。

可以使用快捷键

```
F1,h,?:查看htop使用说明，
F2,s  :设置选项
F3,/  :搜索进程
F4,\  :过滤器，输入关键字搜索
F5,t  :显示属性结构
F6,<,>:选择排序方式
F7, [,:减少进程的优先级(nice)
F8，] :增加进程的优先级(nice)
F9，k :杀掉选中的进程
F10，q:退出htop
u:显示所有用户，并可以选中某一特定用户的进程
U：取消标记所有的进程
```

**Mem：**显示内存的使用情况，3887M 大概是 3.8G，此时的 Mem 不包含 buffers 和 cached 的内存，所以和 free -m 会不同 **Swp：**显示交换空间的使用情况，交换空间是当内存不够和其中有一些长期不用的数据时，ubuntu 会把这些暂时放到交换空间中

其他信息可以参考 top 命令说明。

> PS：如果你终端没安装 htop, 先通过指令来安装。sudo apt-get updatesudo apt install htop

六、查看制定进程的内存
-----------

通过 / proc/procid/status 查看进程内存

```
peng@ubuntu:~$ cat /proc/4398/status
Name: kworker/0:0    //进程名
Umask: 0000
State: I (idle)   //进程的状态
//R (running)", "S (sleeping)", "D (disk sleep)", "T (stopped)", "T(tracing stop)", "Z (zombie)", or "X (dead)"
Tgid: 4398 //线程组的ID,一个线程一定属于一个线程组(进程组).
Ngid: 0
Pid: 4398 //进程的ID,更准确的说应该是线程的ID.
PPid: 2  //当前进程的父进程
TracerPid: 0 //跟踪当前进程的进程ID,如果是0,表示没有跟踪
Uid: 0 0 0 0
Gid: 0 0 0 0
FDSize: 64 //当前分配的文件描述符,该值不是上限，如果打开文件超过64个文件描述符,将以64进行递增
Groups: //启动这个进程的用户所在的组
NStgid: 4398
NSpid: 4398
NSpgid: 0
NSsid: 0
Threads: 1
SigQ: 0/7640
SigPnd: 0000000000000000
ShdPnd: 0000000000000000
SigBlk: 0000000000000000
SigIgn: ffffffffffffffff
SigCgt: 0000000000000000
CapInh: 0000000000000000
CapPrm: 0000003fffffffff
CapEff: 0000003fffffffff
CapBnd: 0000003fffffffff
CapAmb: 0000000000000000
NoNewPrivs: 0
Seccomp: 0
Speculation_Store_Bypass: vulnerable
Cpus_allowed: 00000000,00000000,00000000,00000001
Cpus_allowed_list: 0
Mems_allowed: 00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000001
Mems_allowed_list: 0
voluntary_ctxt_switches: 5
nonvoluntary_ctxt_switches: 0
```

总结：
---

确定内存使用情况是 Linux 运维工程师必要的技能，尤其是某个应用程序变得异常和占用系统内存时。当发生这种情况时，知道有多种工具可以帮助你进行故障排除十分方便的。

- EOF -

推荐阅读  点击标题可跳转

1、[图解 Linux  |  管道通信的原理？](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651167900&idx=1&sn=4af1c2b142a724149b3647f2035056a8&chksm=80644dc3b713c4d58dad916ddad4e00f9961c370590909f6098a851ea6e4dd78c9393deddddc&scene=21#wechat_redirect)

2、[Linux 最大并发数是多少？](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651167793&idx=1&sn=f44c8c206391dfbe61f4e19a1f249ab9&chksm=80644d6eb713c478d23a4b166903c9dd33ed6dfadc0d73b783b1b24eb638c0c0346161d287c7&scene=21#wechat_redirect)

3、[深入理解 Linux 内存子系统](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651167786&idx=1&sn=afb37aa02790fa95de818fd2875f7e0c&chksm=80644d75b713c46381cdc5151c3253b0eda60381be02b0b52edc24c1afd9dfb76581c03573ed&scene=21#wechat_redirect)