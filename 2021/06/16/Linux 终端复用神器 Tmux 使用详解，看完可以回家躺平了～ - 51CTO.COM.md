> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [os.51cto.com](https://os.51cto.com/art/202106/664989.htm)

> Tmux 可用于在一个终端窗口中运行多个终端会话。不仅如此，还可以通过 Tmux 使终端会话运行于后台或是按需接入、断开会话，这个功能非常实用。

Tmux 是 Terminal Multiplexer 的简称，它是一款优秀的终端复用软件，类似 GNU screen，但比 screen 更出色。tmux 来自于 OpenBSD，采用 BSD 授权。使用它最直观的好处就是, 通过一个终端登录远程主机并运行 tmux 后，在其中可以开启多个控制台而无需再 “浪费” 多余的终端来连接这台远程主机, 还有一个好处就是当终端关闭后该 shell 里面运行的任务进程也会随之中断，通过使用 tmux 就能很容易的解决这个问题。

Tmux 可用于在一个终端窗口中运行多个终端会话。不仅如此，还可以通过 Tmux 使终端会话运行于后台或是按需接入、断开会话，这个功能非常实用。

### Tmux 的使用场景

*    可以某个程序在执行时一直是输出状态，需要结合 nohup、& 来放在后台执行，并且 ctrl+c 结束。这时可以打开一个 Tmux 窗口，在该窗口里执行这个程序，用来保证该程序一直在执行中，只要 Tmux 这个窗口不关闭
*    处于异地的两人可以对同一会话进行操作，一方的操作另一方可以实时看到
*    可以在单个屏幕的灵活布局下开出很多终端，然后就能协作地使用它们
*    下班后，你需要断开 ssh 或关闭电脑，将运行的命令或任务放置后台运行
*    关闭终端，再次打开时原终端里面的任务进程依然不会中断

### Tmux 功能：

*    提供了强劲的、易于使用的命令行界面。
*    可横向和纵向分割窗口。
*     窗格可以自由移动和调整大小，或直接利用四个预设布局之一。
*    支持 UTF-8 编码及 256 色终端。
*    可在多个缓冲区进行复制和粘贴。
*    可通过交互式菜单来选择窗口、会话及客户端。
*    支持跨窗口搜索。
*    支持自动及手动锁定窗口。

### Tmux 安装

Ubuntu 版本下可以直接使用 apt 安装

```
sudo apt-get install tmux 
```

CentOS 版本下使用 yum 安装

```
yum install -y tmux 
```

### 在 macOS 中安装

安装 Homebrew

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" 
```

安装 tmux

```
brew install tmux 
```

### 查看版本 

```
tmux -V 
```

[![](https://s6.51cto.com/oss/202106/02/c6cf38783a72f382fca20233869bf1f9.png-wh_651x-s_93943085.png)](https://s6.51cto.com/oss/202106/02/c6cf38783a72f382fca20233869bf1f9.png-wh_651x-s_93943085.png)

### Tmux 的使用

安装完成后输入命令 tmux 即可打开软件，界面十分简单，类似一个下方带有状态栏的终端控制台；但根据 tmux 的定义，在开启了 tmux 服务器后，会首先创建一个会话，而这个会话则会首先创建一个窗口，其中仅包含一个面板；也就是说，这里看到的所谓终端控制台应该称作 tmux 的一个面板，虽然其使用方法与终端控制台完全相同。

# tmux // 直接进入面板，如下效果：

[![](https://s3.51cto.com/oss/202106/02/d8b4bbef7823f53151e0ec5c79f2a70d.png)](https://s3.51cto.com/oss/202106/02/d8b4bbef7823f53151e0ec5c79f2a70d.png)

### Tmux 的快捷键使用说明：

<table><tbody><tr><td colspan="2"><p>Ctrl+b</p></td><td><p>激活控制台；此时以下按键生效</p></td></tr><tr><td rowspan="9"><p>系统操作</p></td><td><p>?</p></td><td><p>列出所有快捷键；按 q 返回</p></td></tr><tr><td><p>d</p></td><td><p>脱离当前会话；这样可以暂时返回 Shell 界面，输入 tmux attach 能够重新进入之前的会话</p></td></tr><tr><td><p>D</p></td><td><p>选择要脱离的会话；在同时开启了多个会话时使用</p></td></tr><tr><td><p>Ctrl+z</p></td><td><p>挂起当前会话</p></td></tr><tr><td><p>r</p></td><td><p>强制重绘未脱离的会话</p></td></tr><tr><td><p>s</p></td><td><p>选择并切换会话；在同时开启了多个会话时使用</p></td></tr><tr><td><p>:</p></td><td><p>进入命令行模式；此时可以输入支持的命令，例如 kill-server 可以关闭服务器</p></td></tr><tr><td><p>[</p></td><td><p>进入复制模式；此时的操作与 vi/emacs 相同，按 q/Esc 退出</p></td></tr><tr><td><p>~</p></td><td><p>列出提示信息缓存；其中包含了之前 tmux 返回的各种提示信息</p></td></tr><tr><td rowspan="10"><p>窗口操作</p></td><td><p>c</p></td><td><p>创建新窗口</p></td></tr><tr><td><p>&amp;</p></td><td><p>关闭当前窗口</p></td></tr><tr><td><p>数字键</p></td><td><p>切换至指定窗口</p></td></tr><tr><td><p>p</p></td><td><p>切换至上一窗口</p></td></tr><tr><td><p>n</p></td><td><p>切换至下一窗口</p></td></tr><tr><td><p>l</p></td><td><p>在前后两个窗口间互相切换</p></td></tr><tr><td><p>w</p></td><td><p>通过窗口列表切换窗口</p></td></tr><tr><td><p>,</p></td><td><p>重命名当前窗口；这样便于识别</p></td></tr><tr><td><p>.</p></td><td><p>修改当前窗口编号；相当于窗口重新排序</p></td></tr><tr><td><p>f</p></td><td><p>在所有窗口中查找指定文本</p></td></tr><tr><td rowspan="14"><p>面板操作</p></td><td><p>”</p></td><td><p>将当前面板平分为上下两块</p></td></tr><tr><td><p>%</p></td><td><p>将当前面板平分为左右两块</p></td></tr><tr><td><p>x</p></td><td><p>关闭当前面板</p></td></tr><tr><td><p>!</p></td><td><p>将当前面板置于新窗口；即新建一个窗口，其中仅包含当前面板</p></td></tr><tr><td><p>Ctrl + 方向键</p></td><td><p>以 1 个单元格为单位移动边缘以调整当前面板大小</p></td></tr><tr><td><p>Alt + 方向键</p></td><td><p>以 5 个单元格为单位移动边缘以调整当前面板大小</p></td></tr><tr><td><p>Space</p></td><td><p>在预置的面板布局中循环切换；依次包括 even-horizontal、even-vertical、main-horizontal、main-vertical、tiled</p></td></tr><tr><td><p>q</p></td><td><p>显示面板编号</p></td></tr><tr><td><p>o</p></td><td><p>在当前窗口中选择下一面板</p></td></tr><tr><td><p>方向键</p></td><td><p>移动光标以选择面板</p></td></tr><tr><td><p>{</p></td><td><p>向前置换当前面板</p></td></tr><tr><td><p>}</p></td><td><p>向后置换当前面板</p></td></tr><tr><td><p>Alt+o</p></td><td><p>逆时针旋转当前窗口的面板</p></td></tr><tr><td><p>Ctrl+o</p></td><td><p>顺时针旋转当前窗口的面板</p></td></tr></tbody></table>

### tmux 的窗口

一个 tmux 的会话中可以有多个窗口，每个窗口又可以分割成多个窗格。我们工作的最小单位其实是窗格。默认情况下在一个窗口中，只有一个大窗格，占满整个窗口区域。我们在这个区域工作。

先来看下 tmux 窗口的相关操作，后面我们再说一下关于窗格的相关知识。首先在新创建的一个会话里面是会默认创建一个窗口的。正如我们上面提到过的图一样。

新创建的会话中会默认创建一个窗口，本例中的窗口名字是 0:bash，0 是序号，我们可以通过 crtl+b , (组合键之后按一个逗号) 来修改当前窗口的名字，如上图所示的窗口名字 linuxmi 就是修改之后的名字。该名字后面有一个 * 号，表示该窗口是活动窗口 (键盘输入会输入到该窗口中)

[![](https://s6.51cto.com/oss/202106/02/8a66ebce3cd100198df20c315350f372.png)](https://s6.51cto.com/oss/202106/02/8a66ebce3cd100198df20c315350f372.png)

修改窗口名称中

[![](https://s5.51cto.com/oss/202106/02/59124134d64013be307bd553e0e6931b.png)](https://s5.51cto.com/oss/202106/02/59124134d64013be307bd553e0e6931b.png)

修改窗口名称后

### 创建窗口

可以在当前会话窗口中创建多个窗口，例如 ctrl+b c 创建之后会多出一个窗口如下图所示：

[![](https://s3.51cto.com/oss/202106/02/c3c37e8708ba8d8ff3c1cb339dd27384.png)](https://s3.51cto.com/oss/202106/02/c3c37e8708ba8d8ff3c1cb339dd27384.png)

默认情况下创建出来的窗口由窗口序号 + 窗口名字组成，窗口名字可以由上面提到的方法修改，可以看到新创建的窗口后面有 * 号，表示是当前窗口。

### 切换窗口

这么多窗口，那么如何在同一个会话的多个窗口之间进行切换呢？可以通过如下快捷键进行操作：

ctrl+b p (previous 的首字母) 切换到上一个窗口。

ctrl+b n (next 的首字母) 切换到下一个窗口。

ctrl+b 0 切换到 0 号窗口，依次类推，可换成任意窗口序号

ctrl+b w (windows 的首字母) 列出当前 session 所有窗口，通过上、下键切换窗口

ctrl+b l (字母 L 的小写) 相邻的窗口切换

3. ctrl+b & 关闭窗口

ctrl+b & 关闭当前窗口，会给出提示是否关闭当前窗口，按下 y 确认即可。

### tmux 的窗格

tmux 的一个窗口可以被分成多个窗格，可以做出分屏的效果。

1. ctrl+b % 垂直分屏 (组合键之后按一个百分号)，用一条垂线把当前窗口分成左右两屏。

[![](https://s6.51cto.com/oss/202106/02/c2bd9be4fd43c1d88ab2fbc6d9111b67.png)](https://s6.51cto.com/oss/202106/02/c2bd9be4fd43c1d88ab2fbc6d9111b67.png)

2. ctrl+b " 水平分屏 (组合键之后按一个双引号)，用一条水平线把当前窗口分成上下两屏。

[![](https://s2.51cto.com/oss/202106/02/b081a5e1137c8e32bc98643c63832030.png)](https://s2.51cto.com/oss/202106/02/b081a5e1137c8e32bc98643c63832030.png)

分屏之后光标停留在哪个窗格上，表示该窗格是活动的，另外一般情况下当前窗格会被绿色的线条围起来。一般分屏之后当前窗口名字会重置为默认窗口名字。通过多次分屏操作，我们可以得到各种样子的分屏效果，例如下图显示的是一次垂直分屏之后，在右边窗格中再次水平分屏的效果：

[![](https://s3.51cto.com/oss/202106/02/7faaf3ea20e91b6af7aaf38765c01c6e.png)](https://s3.51cto.com/oss/202106/02/7faaf3ea20e91b6af7aaf38765c01c6e.png)

可以看到右下角的分屏是绿色框，说明是当前活动窗格

### 如何切换窗格

ctrl+b o 依次切换当前窗口下的各个窗格。

ctrl+b Up|Down|Left|Right 根据按箭方向选择切换到某个窗格。

ctrl+b Space (空格键) 对当前窗口下的所有窗格重新排列布局，每按一次，换一种样式。

ctrl+b z 最大化当前窗格。再按一次后恢复。

还有一种切换方法是 ctrl+b q，tmux 会显示每个窗格的序号，按这个序号就可以跳到这里窗格去了（按慢了可不行，得在数字消失前按）。

[![](https://s6.51cto.com/oss/202106/02/3f3e02500687c7fada689b20a4c9b59c.png)](https://s6.51cto.com/oss/202106/02/3f3e02500687c7fada689b20a4c9b59c.png)

### 关闭窗格

ctrl+b x 关闭当前使用中的窗格，操作之后会给出是否关闭的提示，按 y 确认即关闭。

终端内显示时间

快捷键：先按 ctrl+b, 放开后再按 t

[![](https://s4.51cto.com/oss/202106/02/b157b0b95420c258f5f5af4c3a202fef.png)](https://s4.51cto.com/oss/202106/02/b157b0b95420c258f5f5af4c3a202fef.png)

退出时间界面：按 q 键

tmux ls 终端环境查看会话列表

在终端环境中，我们可以通过 tmux ls 命令来查看后台运行中的 tmux 的会话列表，例如：

[![](https://s3.51cto.com/oss/202106/02/1751f6a9ee25a356e8b37ec3897f3540.png)](https://s3.51cto.com/oss/202106/02/1751f6a9ee25a356e8b37ec3897f3540.png)

可以看到在列出的列表中，只有 1 行，说明只有一个会话，其中左边的 0 表示该会话的名字，中间 3 windows 说明该会话 0 会话中有 3 个窗口，右边表示该会话创建的时间。如果该机器中有多个 tmux 会话在后台运行，那么这里会列出多行。因为 tmux 会话在后台运行，我们猜测实际上肯定是有 tmux 的进程在后台运行来维持这些会话。

### 总结

tmux 中的最重要的三个概念：会话，窗口，窗格的使用方法已经介绍完毕，这也是我们操作 tmux 的最常用功能，只要熟练掌握，就足以应付大多数日常工作。另外 tmux 还有一些高级用法，例如可以个性化的配置其组合键 (官方默认的 ctrl+b 组合键按起来不太方便可以修改，UI 设置，鼠标支持，复制粘贴等)。

【责任编辑：[庞桂玉](mailto:sunsj@51cto.com) TEL：（010）68476606】