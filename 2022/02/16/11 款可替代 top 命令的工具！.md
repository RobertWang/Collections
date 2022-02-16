> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg4NDQwNTI0OQ==&mid=2247554830&idx=3&sn=2f816f588c67c8d1c800e6e97df7170f&chksm=cfbaf260f8cd7b760f6000ff1f007718b1be2dac4b2c775936a3280e670f23188de361c98cd1&mpshare=1&scene=1&srcid=0207pVsVbfpulQBs0m9S5rjJ&sharer_sharetime=1644230697965&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

‍

‍

来源 | 杰哥的 IT 之旅

在 Linux 环境下 top 命令都不陌生，它以实时动态的方式查看系统的整体运行情况，综合了多方信息监测系统性能和运行信息的实用工具，通过 top 命令所提供的互动式界面，可以用热键来进行管理。

在介绍本篇 top 命令的替代工具时，我们先来回顾下 top 的基本语法、常用选项、交互时的热键以及实例解释，从而加深对 top 的理解，同时也希望能进一步的运用在实际场景中。

### 一、top

#### 1.1 top 语法

```
top(选项)
```

#### 1.2 top 选项

*   -b：以批处理模式操作；
    
*   -c：显示完整的治命令；
    
*   -d：屏幕刷新间隔时间；
    
*   -I：忽略失效过程；
    
*   -s：保密模式；
    
*   -S：累积模式；
    
*   -i <时间>：设置间隔时间；
    
*   -u <用户名>：指定用户名；
    
*   -p <进程号>：指定进程；
    
*   -n <次数>：循环显示的次数；
    

#### 1.3 top 交互时的热键

*   h：显示帮助信息并给出简短的命令总结说明提示；
    
*   k：终止一个进程；
    
*   i：忽略闲置和僵死的进程；
    
*   q：退出 top；
    
*   r：重新安排一个进程的优先级别；
    
*   S：切换到累计模式；
    
*   s：改变两次刷新之间的延迟时间（单位为 s），如果有小数，就换算成 ms。输入 0 值则系统将不断刷新，默认值为：5s；
    
*   f 或者 F：从当前显示中添加或者删除；
    
*   o 或者 O：改变显示的顺序；
    
*   l：切换显示平均负载和启动时间信息；
    
*   m：切换显示内存信息；
    
*   t：切换显示进程和 CPU 状态信息；
    
*   c：切换显示命令名称和完整命令行；
    
*   M：根据驻留内存大小进行排序；
    
*   P：根据 CPU 使用百分比大小进行排序；
    
*   T：根据时间或累计时间进行排序；
    
*   w：将当前设置写入 ~/.toprc 文件中；
    

```
top - 09:48:47 up 1 day, 10:54,  2 users,  load average: 0.00, 0.02, 0.00
任务: 293 total,   2 running, 291 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.2 sy,  0.0 ni, 99.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   1945.1 total,    160.5 free,    849.7 used,    934.9 buff/cache
MiB Swap:    923.2 total,    921.4 free,      1.8 used.    904.4 avail Mem

PID USER      PR  NI    VIRT    RES    SHR    %CPU  %MEM     TIME+ COMMAND
6775 root      20   0   21752   4280   3424 R   0.3   0.2   0:05.58 top
```

#### 1.4 top 实例解释

**第一行：任务队列信息**

*   top - 09:48:47：显示当前系统时间
    
*   1 day：系统已经运行了 1 天
    
*   2 users：2 个用户当前登录
    
*   load average: 0.00, 0.02, 0.00：系统负载，即任务队列的平均长度。三个数值分别为 1 分钟、5 分钟、15 分钟前到现在的平均值。
    

**第二行：进程信息**

*   Tasks: 293 total：总进程数
    
*   2 running：正在运行的进程数
    
*   291 sleeping：睡眠的进程数
    
*   0 stopped：停止的进程数
    
*   0 zombie：僵尸进程数
    

**第三行：CPU 信息**

*   %Cpu(s):  0.0 us：用户空间占用 CPU 的百分比
    
*   0.2 sy：内核空间占用 CPU 的百分比
    
*   0.0 ni：用户进程空间内改变过优先级的进程占用 CPU 的百分比
    
*   99.8 id：空闲 CPU 的百分比
    
*   0.0 wa：等待输入输出 CPU 的时间百分比
    
*   0.0 hi：硬中断占用 CPU 的百分比
    
*   0.0 si：软中断占用 CPU 的百分比
    
*   0.0 st：用于有虚拟 CPU 的情况，用来指示被虚拟机偷掉的 CPU 时间
    

**第四 / 五行：内存信息**

*   MiB Mem :   1945.1 total：物理内存总量
    
*   160.5 free：空闲内存总量
    
*   849.7 used：使用的物理内存总量
    
*   934.9 buff/cache：用于内核缓存的内存量
    
*   MiB Swap:    923.2 total：交换区总量
    
*   921.4 free：空闲交换区总量
    
*   1.8 used：使用的交换区总量
    
*   904.4 avail Mem：可用于进程下一次分配的物理内存数量
    

**第六行：进程详细信息**

*   PID：进程 PID 号
    
*   USER：用户
    
*   PR：优先级
    
*   NI：nice 值，负值表示高优先级，正值表示低优先级
    
*   VIRT：进程使用的虚拟内存总量，单位为 KB
    
*   RES：进程使用的、未被换出的物理内存大小，单位为 KB
    
*   SHR：共享内存大小，单位为 KB
    
*   %CPU：上次更新到现在的 CPU 时间占用百分比
    
*   %MEM：进程使用的物理内存百分比
    
*   TIME+：进程使用的 CPU 时间总计，单位 1/100 秒
    
*   COMMAND：命令名 / 命令行
    

以上就是针对 top 命令的回顾。当然了，进程详细信息里还有很多信息，就不一一介绍了，在日常学习和工作中有需要用到的解释说明，可参阅相关资料进行了解。

不过 top 已经满足我们在学习以及工作中排查相关问题的基本条件了。接下来，给大家介绍一些针对 top 命令的替代工具，也许做了对比后，你会更喜欢这些替代工具。

### 二、bashtop

bashtop[1] 基于 Shell 语言编写，是用于展示当前 Linux 操作系统的处理器、内存、硬盘、网络和进程等各项资源的使用情况与状态，可在 Fedora、CentOS 8、RHEL 8、Ubuntu、Debian、FreeBSD、OSX 等多种操作系统中安装。

#### 2.1 bashtop 的特征  

*   易使用，具有游戏灵感的菜单系统；
    
*   快速且 “大部分” 响应式 UI，带有 UP、DOWN 键进程选择；
    
*   显示所选进程的详细统计信息的功能；
    
*   显示当前磁盘的读写速度；
    
*   过滤流程的能力；
    
*   在排序选项之间轻松切换；
    
*   向选定的进程发送 SIGTERM、SIGKILL、SIGINT；
    
*   可更改所有配置文件选项的 UI 菜单；
    
*   网络使用情况的自动缩放图；
    
*   如有新版本可用，则在菜单中进行显示；
    
*   Linux 环境下可切换的多种数据采集方式；
    

#### 2.2 bashtop 的安装

**CentOS 8 安装：**

```
# dnf config-manager --set-enabled PowerTools 
# dnf install epel-release 
# dnf install bashtop
```

**RHEL 8 安装：**

```
ARCH= $( /bin/arch ) 
subscription-manager repos --enable " codeready-builder-for-rhel-8- ${ARCH} -rpms " 
dnf install epel-release 
dnf install bashtop
```

**Ubuntu 安装：**

```
# sudo add-apt-repository ppa:bashtop-monitor/bashtop 
# sudo apt update 
# sudo apt install bashtop
```

安装非常简单，我用的是 Ubuntu，所以用 apt install 直接安装即可，如果你用的是其他操作系统，可参考：https://github.com/aristocratos/bashtop#installation 章节进行安装。

#### 2.3 bashtop 的使用

安装完毕后，直接执行命令`bashtop`即可。

```
# bashtop
```

  

![](https://mmbiz.qpic.cn/mmbiz_gif/nDMNE6lrvW7TzbTmJ11kiaAtOviceTp3qjHtbHvo2KN1kwmIJFIs0Gd9LCcFKvrztkPpFeN8a8kn4nKVjR2Cjagg/640?wx_fmt=gif)

### 三、bpytop

bpytop[2] 是 bashtop 的延续，基于 Python 语言编写，主要用于展示当前 Linux 操作系统的处理器、内存、磁盘、网络和进程的使用情况和统计信息的资源监视器，可在 Mac OSX、Arch Linux、Debian、FreeBSD、Fedora、CentOS 8、Ubuntu 等多种操作系统中安装。

#### 3.1 bpytop 的特征  

*   易使用且具有游戏灵感的菜单系统；
    
*   完全支持鼠标，所有带有突出显示键的按钮都是可点击的，并且鼠标滚动可以在进程列表和菜单框中工作；
    
*   快速且响应迅速的 UI，带有 UP、DOWN 键进程选择；
    
*   显示所选进程的详细统计信息的功能；
    
*   能够过滤进程，可以输入多个过滤器；
    
*   在排序选项之间轻松切换；
    
*   向选定的进程发送 SIGTERM、SIGKILL、SIGINT；
    
*   用于更改所有配置文件选项的 UI 菜单；
    
*   网络使用情况的自动缩放图；
    
*   如有新版本可用，则在菜单中显示消息；
    
*   显示当前磁盘的读写速度；
    

#### 3.2 bpytop 的安装

**Mac OSX 安装：**

```
# brew install bpytop
```

**Fedora / CentOS 8 安装：**

```
# sudo dnf install bpytop
```

**Debian / Ubuntu 安装：**

```
# sudo apt install bpytop
```

安装非常简单，我用的是 Ubuntu，所以用 apt install 直接安装即可，如果你用的是其他操作系统，可参考：https://github.com/aristocratos/bpytop#installation 章节进行安装。

#### 3.3 bpytop 的使用

安装完毕后，直接执行命令`bpytop`即可。

```
$ bpytop
```

  

![](https://mmbiz.qpic.cn/mmbiz_gif/nDMNE6lrvW7TzbTmJ11kiaAtOviceTp3qjb9ATPRgliagfYCuCqnH2vrGQicSg2B7jeNn9nqhqdDkuUL3yd5TZCQ5Q/640?wx_fmt=gif)

更多 bpytop 命令行选项

```
usage: bpytop [-h] [-b BOXES] [-lc] [-v] [--debug]

optional arguments:
  -h, --help            show this help message and exit
  -b BOXES, --boxes BOXES
                        which boxes to show at start, example: -b "cpu mem net proc"
  -lc, --low-color      disable truecolor, converts 24-bit colors to 256-color
  -v, --version         show version info and exit
  --debug               start with loglevel set to DEBUG overriding value set in config
```

### 四、btop

btop[3] 基于 C++ 语言编写，主要用于展示当前 Linux 操作系统的处理器、内存、磁盘、网络和进程的使用情况和统计信息的资源监视器。

#### 4.1 btop 的特征  

*   易使用，具有游戏灵感的菜单系统；
    
*   支持鼠标所有带有突出显示键的按钮都是可点击的，并且鼠标滚动在进程列表和菜单框中工作；
    
*   快速且响应迅速的 UI，带有 UP、DOWN 键进程选择；
    
*   显示所选进程的详细统计信息的功能；
    
*   过滤流程的能力；
    
*   在排序选项之间轻松切换；
    
*   进程的树视图；
    
*   向选定的进程发送任何信号；
    
*   用于更改所有配置文件选项的 UI 菜单；
    
*   网络使用情况的自动缩放图；
    
*   显示磁盘的 IO 活动和速度；
    
*   电池电量计；
    
*   图表的可选符号；
    

#### 4.2 btop 的安装

```
# snap install btop
```

安装非常简单，我用的是 Ubuntu，所以用 snap  install 直接安装即可，如果你用的是其他操作系统或通过其他方式进行安装，可参考：https://github.com/aristocratos/btop#installation 章节进行安装。

#### 4.3 btop 的使用

安装完毕后，直接执行命令`btop`即可。

```
# btop
```

  

![](https://mmbiz.qpic.cn/mmbiz_png/nDMNE6lrvW7TzbTmJ11kiaAtOviceTp3qj8kBdYFIWkIqu7rFaCLtlw8kwrEnRoXxCXj30FQ4xVkwp65aJCwib4eg/640?wx_fmt=png)

更多 bpytop 命令行选项  

```
usage: btop [-h] [-v] [-/+t] [-p <id>] [--utf-force] [--debug]

optional arguments:
  -h, --help            show this help message and exit
  -v, --version         show version info and exit
  -lc, --low-color      disable truecolor, converts 24-bit colors to 256-color
  -t, --tty_on          force (ON) tty mode, max 16 colors and tty friendly graph symbols
  +t, --tty_off         force (OFF) tty mode
  -p, --preset <id>     start with preset, integer value between 0-9
  --utf-force           force start even if no UTF-8 locale was detected
  --debug               start in DEBUG mode: shows microsecond timer for information collect
                        and screen draw functions and sets loglevel to DEBUG
```

以上三款开源 top 替代工具可以说是 Jakob P. Liljenberg 的三剑客。

### 五、bottom

bottom[4] 是用于终端的可定制跨平台图形进程 / 系统监视器，支持 Linux、macOS 和 Windows。

#### 5.1 bottom 的特征  

*   随着时间变化的 CPU 使用率、平均水平和每个核心水平；
    
*   随着时间变化的 RAM 和交换使用情况；
    
*   一段时间内的网络 I/O 使用情况；
    
*   支持放大或缩小显示的当前时间间隔；
    
*   支持显示磁盘容量、使用情况、温度传感器、电池使用情况的信息；
    
*   支持显示、排序、搜索有关流程信息的小部件（CPU、内存、网络、进程、磁盘、温度、电池）；
    
*   支持使用命令行标志或配置文件控制的可定制行为（自定义和预建的颜色主题、更改某些小部件的默认行为、更改小部件的布局、过滤掉磁盘和温度小部件中的条目）；
    
*   支持键、鼠标绑定相关快捷键；
    

#### 5.2 bottom 的安装

可以在 Arch Linux、Debian/Ubuntu、Fedora/CentOS 多种平台或以多种方式都可进行安装。

```
# curl -LO https://github.com/ClementTsang/bottom/releases/download/0.6.6/bottom_0.6.6_amd64.deb 
# sudo dpkg -i bottom_0.6.6_amd64.deb
```

#### 5.3 bottom 的使用  

安装完毕后，执行命令`btm`即可

```
# btm
```

  

![](https://mmbiz.qpic.cn/mmbiz_png/nDMNE6lrvW7TzbTmJ11kiaAtOviceTp3qjmfxmL0iaSnpQodMuRNfdMqGLPXsSZwUbbIqpp0RNVN2ccfibdxSgLaAg/640?wx_fmt=png)

### 六、glances

Glances[5] 是基于 Python 语言编写的一款跨平台的监控工具，旨在通过 curses 或基于 Web 的界面呈现大量系统监控信息，该信息可根据用户界面的大小动态调整，是 GNU/Linux、BSD、Mac OS 和 Windows 操作系统的 top/htop 替代品。

它可以在客户端 / 服务器模式下工作，远程监控可以通过终端、Web 界面或 API（XML-RPC 和 RESTful）完成，统计数据也可以导出到文件或外部时间 / 值数据库。  

除了列出所有进程及其 CPU 和内存使用情况之外，它还可以显示有关系统的其他信息，比如：

*   网络及磁盘使用情况
    
*   文件系统已使用的空间和总空间
    
*   来自不同传感器（例如电池）的数据
    
*   以及最近消耗过多资源的进程列表
    

#### 6.1 glances 的安装

```
# apt install glances
```

#### 6.2 glances 的使用

独立模式只需执行：

```
# glances
```

  

![](https://mmbiz.qpic.cn/mmbiz_png/nDMNE6lrvW7TzbTmJ11kiaAtOviceTp3qjSibHegEHZC3WV0IUcblkfCoV3MaibOrUbSPEq5ttjJf6r7r00vuTge7Q/640?wx_fmt=png)

Web 服务器模式可执行：  

```
# glances -w
```

使用 Web 界面监控本地机器并启动 RESTful 服务器，从`http://0.0.0.0:61208/`开始浏览 Web 服务器。

客户端 / 服务器模式可执行：

```
# glances -s
```

在服务器端并运行：

```
# glances -c <ip_server>
```

更多 glances 使用、可选参数选项以及使用案例可执行：`glances -h`查看相关帮助信息。

### 七、gotop

gotop[6] 是基于 Go 语言编写，是一个基于终端的图形活动监视器，可在 Linux、FreeBSD 和 macOS 上运行。

#### 7.1 gotop 的安装  

```
# snap install gotop
```

#### 7.2 gotop 的使用

安装完毕后，执行命令`gotop`即可，更多热键可参考 GitHub 存储库的用法部分。

![](https://mmbiz.qpic.cn/mmbiz_png/nDMNE6lrvW5G1gd6xesO5GQhHfD3Kda0VOA8BTSfkrspTVs2sAzTgzwXQ6b5pFcG4VCjnrjmQsKmJK1W15b1iag/640?wx_fmt=png)

### 八、gtop

gtop[7] 是基于 JavaScript 语言编写的一款终端系统监控仪表板。

#### 8.1 gtop 的安装  

```
# apt install npm
# npm install gtop -g
```

#### 8.2 gtop 的使用

安装完毕后，执行命令`gtop`即可，要停止 gtop 可使用`q`键或者`Ctrl+c`。

```
# gtop
```

  

![](https://mmbiz.qpic.cn/mmbiz_png/nDMNE6lrvW5G1gd6xesO5GQhHfD3Kda0UoBaXQwR03KibENpdJWvSAIPQB0Kib92HWkTFM94M6wEvg5hFGsbekPA/640?wx_fmt=png)

### 九、htop

htop[8] 可以说是 top 的绝佳替代品，它是用 C 写的，是一个跨平台的交互式的进程监控工具，具有更好的视觉效果，一目了然更容易理解当前系统的状况，允许垂直和水平滚动进程列表以查看它们的完整命令行和相关信息，如内存和 CPU 消耗。还显示了系统范围的信息，例如平均负载或交换使用情况。

显示的信息可通过图形设置进行配置，并且可以交互排序和过滤，与进程相关的任务（例如终止和重新处理）可以在不输入其 PID 的情况下进行完成。  

安装也很简单，只需执行命令：`apt install htop`即可完成。

![](https://mmbiz.qpic.cn/mmbiz_png/nDMNE6lrvW5G1gd6xesO5GQhHfD3Kda071V7HhrOdtNJmHEAzPDBeMod63Yl7jUduoIQLDqIbAiauIRqKZMl3mg/640?wx_fmt=png)

### 十、nvtop

Nvtop[9]：NVidia TOP，一个用于 NVIDIA GPU 的 (h)top 类似任务监视器，它可以处理多个 GPU 并以 htop 熟悉的方式打印有关它们的信息。

**Ubuntu disco (19.04) / Debian buster (stable)**  

```
# sudo apt install nvtop
```

  

![](https://mmbiz.qpic.cn/mmbiz_png/nDMNE6lrvW5d2iaRVhe0wzJFS7ZyYUZ4B1UmZ33McOgUqLicAB1oNr4pghsXNn536EQQkynX6oRfBKUJmrXEsibug/640?wx_fmt=png)

更多 nvtop 命令行选项  

```
# nvtop -h
nvtop version 1.1.0
Available options:
  -d --delay        : Select the refresh rate (1 == 0.1s)
  -v --version      : Print the version and exit
  -s --gpu-select   : Colon separated list of GPU IDs to monitor
  -i --gpu-ignore   : Colon separated list of GPU IDs to ignore
  -p --no-plot      : Disable bar plot
  -r --reverse-abs  : Reverse abscissa: plot the recent data left and older on the right
  -C --no-color     : No colors
  -N --no-cache     : Always query the system for user names and command line information
  -f --freedom-unit : Use fahrenheit
  -E --encode-hide  : Set encode/decode auto hide time in seconds (default 30s, negative = always on screen)
  -h --help         : Print help and exit
```

### 十一、vtop

vtop[10] 是一款命令行的图形活动监视器，它是使用 drawille 绘制带有 Unicode 盲文字符的 CPU 和内存图表，通过可视化的方式进行展示，还将具有相同名称的进程分组在一起，安装也很简单，只需执行命令`npm install -g vtop`即可完成。

运行只需执行命令`vtop`即可。  

![](https://mmbiz.qpic.cn/mmbiz_png/nDMNE6lrvW5d2iaRVhe0wzJFS7ZyYUZ4Bsia9RmVTdhicXI9pTTBpr3RKZtHmEetqibtf2YGUib3W7ArliabwhpKfibFw/640?wx_fmt=png)

  

*   dd：杀死该组中的所有进程
    
*   按向下箭头或`j`键向下移动
    
*   按向上箭头或`k`键向上移动进程列表
    
*   按`g`键转到进程列表的顶部
    
*   按`G`键移动到列表的末尾
    
*   按`c`键可按 CPU 进行排序
    
*   按`m`键可按内存进行排序
    

### 十二、zenith

zenith[11] 是基于 Rust 语言编写的一款具有可缩放的图表、CPU、GPU、网络和磁盘使用的终端图形。

#### 12.1 zenith 的特征  

*   可选的 CPU、内存、网络和磁盘使用情况图表
    
*   支持浏览磁盘可用空间、NIC IP 地址、CPU 频率
    
*   支持显示 CPU、内存和磁盘的用户
    
*   电池百分比、充电或放电时间、已用电量
    
*   类似于 top 的可过滤进程表，包括每个进程的磁盘使用情况
    
*   更改流程优先级
    
*   可缩放图表视图（支持及时回滚）
    
*   使用信号管理流程
    
*   运行之间保存的性能数据
    
*   NVIDIA GPU 的 GPU 利用率指标（带有 --features nvidia），包括每个进程的 GPU 使用率
    
*   磁盘可用空间图表
    

#### 12.2 zenith 的安装

zenith 我是通过 deb 软件包安装的，不过最新的 64 位 deb 软件包需要基于 Debian >= 9 或 Ubuntu >= 16.04 的发行版才可安装。

```
# curl -LO https://github.com/bvaisvil/zenith/releases/download/0.12.0/zenith_0.12.0-1_amd64.deb
# dpkg -i zenith_0.12.0-1_amd64.deb
```

#### 12.3 zenith 的使用

安装 zenith 完毕后，不带任何参数运行 zenith 将以 CPU、磁盘和网络的默认可视化和 2000 毫秒（2 秒）的刷新率启动。

```
# zenith
```

  

![](https://mmbiz.qpic.cn/mmbiz_png/nDMNE6lrvW5d2iaRVhe0wzJFS7ZyYUZ4BvADsrfx2uv5Jqw3OrUibjGfrEXpRrfyU6x33eNHGibDzVIlYJ4kicgeDA/640?wx_fmt=png)

更多 zenith 命令行选项

```
Usage: zenith [OPTIONS]

Optional arguments:
  --disable-history          Disables history when flag is present (default: false)
  -h, --help
  -V, --version
  -c, --cpu-height INT       Min Percent Height of CPU/Memory visualization. (default: 17)
  --db STRING                Database to use, if any.
  -d, --disk-height INT      Min Percent Height of Disk visualization. (default: 17)
  -n, --net-height INT       Min Percent Height of Network visualization. (default: 17)
  -p, --process-height INT   Min Percent Height of Process Table. (default: 32)
  -r, --refresh-rate INT     Refresh rate in milliseconds. (default: 2000)
  -g, --graphics-height INT  Min Percent Height of Graphics Card visualization. (default: 17)
```

### References

[1] bashtop：_https://github.com/aristocratos/bashtop_  
[2] bpytop：_https://github.com/aristocratos/bpytop_  
[3] btop：_https://github.com/aristocratos/btop_  
[4] bottom：_https://github.com/ClementTsang/bottom_  
[5] glances：_https://github.com/nicolargo/glances_  
[6] gotop：_https://github.com/xxxserxxx/gotop_  
[7] gtop：_https://github.com/aksakalli/gtop_  
[8] htop：_https://github.com/htop-dev/htop_  
[9] nvtop：_https://github.com/Syllo/nvtop_  
[10] vtop：_https://github.com/MrRio/vtop_  
[11] zenith：_https://github.com/bvaisvil/zenith_