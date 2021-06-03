> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666554993&idx=3&sn=48f1bea1f440b964894bffd78e6787e5&chksm=80dca2dab7ab2bcc2ec287c172fd7df222a31e1008fc668b004f9f855129667315b5d6a32493&mpshare=1&scene=1&srcid=0603fIONuE02Ajly9jdpCPee&sharer_sharetime=1622696083890&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> 比较 tmux、GNU Screen、Konsole 和 Terminator，看看哪个最适合你。

Linux 用户通常需要大量的虚拟视觉空间。一个终端窗口是永远不够的，所以终端有了标签。一个桌面太受限制了，所以有了虚拟桌面。当然，应用程序窗口可以堆叠，但当它们堆叠起来时，又有多大的好处呢？哎呀，即使是后台文本控制台也有 `F1` 到 `F7`，可以在任务之间来回翻转。

有了这么多的多任务处理方式，有人发明了终端 _多路复用器_ 的概念就不奇怪了。诚然，这是一个令人困惑的术语。在传统的电子学中，“多路复用器 multiplexer” 是一个接收多个输入信号并将选定的信号转发到单一输出的部件。终端多路复用器的作用正好相反。它从一个输入（人类在键盘上向一个终端窗口打字）接收指令，并将该输入转发给任意数量的输出（例如，一组服务器）。

然后，“多路复用器”一词在美国也是一个流行的术语，指的是有许多屏幕的电影院（与 “影城 cineplex” 一词一个意思）。在某种程度上，这很好地描述了终端复用器的作用。它可以在一个框内提供许多屏幕。

不管这个词是什么意思，任何尝试过它的人都有自己的喜好的某一种多路复用器。因此，我决定考察一些流行的终端多路复用器，看看每一个都怎么样。就我的评估标准而言，最低限度，我需要每个多路复用器能够分割_和_堆叠终端窗口。

### tmux

![](https://mmbiz.qpic.cn/mmbiz_png/9aPYe0E1fb1bfyGSBmOuYiaU9v6NicGF5aTPp9Zd6dsYs0eMbgBLFQ034FdMjKlbok47pkEiadZ6MjgHcicWjuVLDg/640?wx_fmt=png)

tmux

据我所知，是从 tmux 开始使用 “多路复用器” 这个术语的。它工作的很出色。

它作为一个守护程序运行，这样即使你关闭了正在查看的终端模拟器，你的终端会话仍然处于活动状态。它将你的终端屏幕分割成多个面板，这样你就可以在每个面板上打开独特的终端提示符。

推而广之，这意味着你也可以远程连接到任何数量的系统，并在你的终端中打开它们。利用 tmux 的能力，将输入镜像（或者以电子学术语说是 “反向多路复用”）到其他打开的窗格，就能从一个中央命令窗格同时控制几台计算机。

tmux 在 GNU Screen 还只能水平分割的时候就有了垂直分割能力，这吸引了追求最大灵活性的粉丝。而灵活性正是用户在 tmux 中得到的。它可以分割、堆叠、选择和提供服务；几乎没有什么是它做不到的。

#### 📦 软件包大小

从软件包中安装 tmux 大约需要 700K，这还不算它所依赖的十几个共享库。

#### 🎛️ 控制键

tmux 的默认触发键是 `Ctrl+B`，尽管很容易在其配置文件中重新定义。

#### ⌨️ 黑客因子

即使你只是在学习如何使用终端，你也一定会觉得使用 tmux 的人很像黑客。它看起来很复杂，但一旦你了解了正确的键绑定，就很容易使用。它为你提供了很多有用的技巧，让你玩的飞起，而且它是一种快速构建 HUD（抬头显示器）的超简单方法，可以把你需要的所有信息摆在你面前。

### GNU Screen

![](https://mmbiz.qpic.cn/mmbiz_png/9aPYe0E1fb1bfyGSBmOuYiaU9v6NicGF5aR5Ou9REfMMVyhqo3YJBdTF5ib7icqmj5I3iapwv06Db0DuyJ2VlaKhh3w/640?wx_fmt=png)

GNU Screen

像 tmux 一样，GNU Screen 也运行一个守护程序，所以即使你关闭了用来启动它的终端，你的 shell 仍然可用。你可以从不同的计算机上连接并共享屏幕。它可以将你的终端屏幕分割成水平或垂直的窗格。

与 tmux 不同的是，GNU Screen 可以通过串行连接进行连接（`screen 9600 /dev/ttyUSB0` 就可以了），通过按键绑定可以方便地发出 `XON` 和 `XOFF` 信号。

与 SSH 会话相比，在串行连接中需要多路复用器的情况可能并不常见，所以大多数用户并不了解 Screen 这个真正特殊的功能。不过，GNU Screen 是一个很棒的多路复用器，有很多有用的选项。而如果你真的需要同时向多个服务器发送信号，还有专门的工具，比如 ClusterSSH 和 Ansible[1]。

#### 📦 软件包大小

从软件包中安装 GNU Screen 大约需要 970K，这还不算它所依赖的十几个共享库。

#### 🎛️ 控制键

GNU Screen 的默认触发键是 `Ctrl+A`，这对于熟悉 Bash 快捷键的人来说可能特别烦人。幸运的是，你可以在配置文件中轻松地重新定义这个触发键。

#### ⌨️ 黑客因子

当使用 Screen 通过串行连接到你的路由器或你的原型电路板时，你会成为你所有硬件黑客朋友羡慕的对象。

### Konsole

![](https://mmbiz.qpic.cn/mmbiz_png/9aPYe0E1fb1bfyGSBmOuYiaU9v6NicGF5accnBLdMxicVSickV76GAdluvlVfUkekuVicooFAzBhW0ibDoPaLRru6Lhg/640?wx_fmt=png)

Konsole

对于没有标榜自己是多路复用器的 Konsole 来说，令人惊讶的是它也是其中一个。它可以使用 Qt 窗格和标签进行必要的窗口分割和堆叠，但它也可以通过 “编辑（将输入复制到）” 菜单中的一个选项将输入从一个窗格传到另一个（或全部）。

然而，它所最明显缺乏的功能是作为一个守护程序运行以进行远程重新连接的能力。与 tmux 和 GNU Screen 不同，你不能远程连接到运行 Konsole 的机器并加入会话。对于一些管理员来说，这可能不是一个问题。许多管理员用 VNC[2] 连接到机器的次数比用 SSH[3] 还要多，所以 “重新加入” 一个会话就像在 VNC 客户端上点击 Konsole 窗口一样简单。

使用 Konsole 作为多路复用器是 KDE 极客们的大招。Konsole 是我使用的第一个 Linux 终端（直到今天，我有时也会按 `Ctrl+N` 来切换新标签），所以有能力使用这个熟悉的终端作为多路复用器是一个很大的便利。这绝不是必要的，因为无论如何 tmux 和 Screen 都可以在 Konsole 里面运行，但是通过让 Konsole 处理窗格，我就不必调整肌肉记忆。这种微妙的功能包容正是 KDE 的伟大之处 [4]。

#### 📦 软件包大小

Konsole 本身大约是 11KB，但它依赖于 105 个 KDE 和 Qt 库，所以实际上，它至少有 50MB。

#### 🎛️ 控制键

大多数重要的 Konsole 快捷键以 `Shift+Ctrl` 开始，分割屏幕、打开新标签、复制输入到其他窗格等都是如此。这是 KDE 里的主控台，所以如果你对 Plasma 桌面很熟悉，会感觉快捷键很熟悉。

#### ⌨️ 黑客因子

使用 Konsole 作为你的多路复用器让你有资格称自己为 KDE 高级用户。

### Terminator

![](https://mmbiz.qpic.cn/mmbiz_png/9aPYe0E1fb1bfyGSBmOuYiaU9v6NicGF5aZZibKiaO5ARsQ7iaiaLDgw0UdCjMtv5nnknmzia0LWUe1mXdxeO4X2UbyWA/640?wx_fmt=png)

Terminator

对于 GNOME 用户来说，Terminator 多路复用器是为他们原本极简的 GNOME 终端增加功能的一个简单方法。除了必要的多路复用功能外，Terminator 还可以向所有打开的窗格广播输入，但和 Konsole 一样，它不会在后台运行以便你可以通过 SSH 重新连接到它。话说回来，由于 GNOME 和 Wayland 让 VNC 变得如此简单，你有可能会觉得没有必要通过 SSH 来恢复终端会话。

如果你愿意，Terminator 可以完全由鼠标驱动。Konsole 通过其主菜单也有同样的能力。有了 Terminator，你可以在 Shell 的任何地方点击右键，弹出相关选项，以水平或垂直分割窗口，将窗格分组作为广播目标，广播输入，关闭窗格，等等。你还可以为所有这些动作配置键盘快捷键，所以在许多方面，你可以形成自己的体验。

我认为自己主要是一个 KDE 用户，所以当我说 Terminator 感觉像一个 KDE 应用时，我其实是一种极大的赞美。Terminator 是一个令人惊讶的可配置的和灵活的应用程序。在许多方面，它体现了开源的力量，把简陋的 GNOME 终端变成了一个强大的多路复用器。

#### 📦 软件包大小

Terminator 的安装容量为 2.2MB，其中大部分是 Python 模块。但它依赖于 GTK3 和 GNOME，所以如果你没有运行完整的 GNOME 桌面，可以预料你需要一个更大的安装来拉入这些依赖。

#### 🎛️ 控制键

Terminator 的默认控制键没有什么一致性。你可以用 `Alt` 键来执行一些命令，用 `Ctrl` 来执行其他命令，还可以用 `Shift+Ctrl`、`Ctrl+Alt`、`Shift+Super` 等等，还有鼠标。话说回来，这是我试过的最可配置的多路复用器之一，所以只要有想法，稍加努力，你就能设计出适合你的模式。

#### ⌨️ 黑客因子

当你使用 Terminator 时，你会觉得自己是最现代、最务实的黑客。由于它的各种极客选项，它是多路复用的最佳选择，而且由于它非常灵活，无论你的手是在键盘上，还是键盘和鼠标并用，你都可以同样轻松地使用它。

### 我全要

还有更多的多路复用器和一些具有类似多路复用能力的应用。你不必非要找到_一个_完全按照你想要的方式完成你需要的所有工作的多路复用器。你可以使用不止一个。事实上，你甚至可以同时使用多个，因为 tmux 和 Screen 实际上是 shell，而 Konsole 和 Terminator 是显示 shell 的终端。对唾手可得的工具感到舒适，而且它们能帮助你管理你的工作空间，使你能有效地工作，才是最重要的。

去尝试一下多路复用器，或者在你喜欢的应用程序中发现类似多路复用器的功能。它可能会改变你看待计算的方式。

### 参考资料

[1]

Ansible: _https://opensource.com/article/19/2/quickstart-guide-ansible_

[2]

VNC: _https://en.wikipedia.org/wiki/Virtual_Network_Computing_

[3]

SSH: _https://en.wikipedia.org/wiki/Secure_Shell_Protocol_

[4]

KDE 的伟大之处: _https://opensource.com/article/19/12/linux-kde-plasma_

- EOF -

推荐阅读  点击标题可跳转

1、[25 张图，一万字，拆解 Linux 网络包发送过程](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666554809&idx=1&sn=31381caf815f6b0dc266b6dc432959da&chksm=80dca112b7ab2804efc8a99772fc17a1217f791a988c5087a703b10c4bf3d27fee55abfbcd85&scene=21#wechat_redirect)

2、[深度剖析 Linux 的 3 种 “拷贝” 命令](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666554805&idx=2&sn=b610270de41641f20679a4012f6f2ef5&chksm=80dca11eb7ab28087e2ced0c692b150dc0daf9465e2e41adffaa2bff2dadb3ae941bf520ab43&scene=21#wechat_redirect)

3、[爱了！3 个受欢迎的 U 盘 Linux 发行版](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666554760&idx=2&sn=23541789aa0eb44bdff03a323b60647b&chksm=80dca123b7ab2835b68d559b7837000b05dcf3c567e8f8909a6928149ae2bd2bd57383a91ec2&scene=21#wechat_redirect)

看完本文有收获？请分享给更多人  

公众号