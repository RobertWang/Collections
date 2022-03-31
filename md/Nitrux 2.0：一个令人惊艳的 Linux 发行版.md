> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/L0CTBsqeQ4mSL5TlhGsQUA)

> 以下是对 Nitrux 2.0 在性能稳定性方面的点评，以及我们对是否可以将其用作日常使用的看法。

Nitrux Linux[1] 是基于 Debian 的，其特点是采用了修改版的 KDE Plasma 桌面，它被称为 NX 桌面。这个独特的 Linux 发行版带来了它自己的一套 Nitrux 应用程序，它们建立在 Maui 套件和 Qt 上。Nitrux 是没有 systemd 的，它使用 OpenRC 作为初始化系统。凭借所有这些独特的功能和令人惊叹的外观，它是当今最好的 Linux 发行版之一。

Nitrux 团队在 2022 年 2 月发布了它的主要版本 2.0，最近又发布了第一个小版本。因此，我们觉得现在是对这个漂亮的桌面进行评价的好时机。

![](https://mmbiz.qpic.cn/mmbiz_jpg/9aPYe0E1fb1NWMc7G53W4sOrW6kCOgv1boj4fAozxNQfQnqMfFKxD9lHN3j3hy7zFNNmIR1ZxmKicutAZevebLg/640?wx_fmt=jpeg)

Nitrux 2.0 应用程序视图

### Nitrux 2.0 点评

#### 安装

Nitrux 使用了一个修改版的 Calamares 安装程序。该操作系统的 ISO 包含临场 live 桌面，在此你可以访问安装操作系统的快捷方式。启动选项包括更多的选项，包括 `nomodset` 内核启动选项。

在虚拟机测试中，安装很顺利，但在实际的硬件中却失败了，因为我的 NVIDIA 340 有点老。因此，如果你打算安装较新的硬件，应该没有问题。

#### 第一印象和桌面

在外观方面，Nitrux 可以说是与当今所有外观优秀的发行版不相上下，比如深度、Cutefish OS。它们都为用户和 Nitrux 操作系统带来了开箱即用的定制功能。但 Nitrux 的优势在于 KDE Plasma、Plasmoid、Kvuntum 主题与基于 Maui 套件组件的奇妙组合。

当你第一眼看到它的时候，它看起来很好，而且有良好的组织性，底部预配置了 Latte Dock、友好而干净的顶部栏。

它基于 KDE Plasma，你可以轻松地改变外观和感觉，并通过设置在深色和浅色模式之间切换。默认字体 Fire Sans 使它成为一个整体设计完美的桌面。

这个版本采用了 KDE Plasma 5.24+，KDE 框架 5.91 和 xanmod 版的 Linux 内核 5.16。

到目前为止，我的第一印象没有任何槽点。

#### 登录和 Shell

不久前，该团队 引入 [2] 了 Maui Shell，这是一个以 Cask 为特色的融合性桌面，即 Shell 层。这是这个体验式 Shell 的第一个版本，用户通过登录窗口就可以看到。

但唯一困扰我的是，Cask（仍然是实验性的）被定为默认登录会话。那些知道的人会把它改成 Plasma，但那些不知道的人会有些吃惊！

![](https://mmbiz.qpic.cn/mmbiz_jpg/9aPYe0E1fb1NWMc7G53W4sOrW6kCOgv1JACwYeTopqBKkibqBxwv1PZMtg9UpwwcG4yUtDzEVWJr0QwYCFGP6iaw/640?wx_fmt=jpeg)

使用 Cask 的登录会话

#### 应用程序

Nitrux 使用 AppImage 格式来发布应用程序。大多数预装的应用程序都是 AppImage。而且在通知方面，它们与整个桌面很好地结合在一起。Nitrux 也会检测你的下载文件夹中外部下载的 AppImage 进行安装。

默认情况下，它预装了以下本地 Maui 应用程序 [3]：

*   Index 文件管理器
    
*   Station 终端
    
*   Pix 图像浏览器
    
*   Nota 文本编辑器
    
*   Nitro 分享
    
*   NX 软件中心
    

Firefox 和 LibreOffice 也预装了，以满足基本需求。你可以根据你的工作流程需要，通过 NX 软件中心安装其他应用程序。

关于 Firefox 和更新 Nitrux 的一点提醒。有一些报告说 Firefox 在进行基本系统更新后被删除。在你点击升级之前，请确保你通过终端用 `apt get —upgradable` 检查文件的变化。

#### 性能和资源消耗

因为我无法把它安装在我的物理机上。因此，下面提到的性能指标是在 virt-manager[4] 虚拟管理器下测量的。

在空闲状态下，它使用大约 1GB 的内存，CPU 在 9% 到 10%。KWin 窗口管理器和 Latte Dock 在底部消耗了大部分的资源。

![](https://mmbiz.qpic.cn/mmbiz_jpg/9aPYe0E1fb1NWMc7G53W4sOrW6kCOgv1OHB3M9dG1erFF3aQlpbTomunn9Ps0uN2OuY2n6J7pbWOXAYNL2fV7w/640?wx_fmt=jpeg)

Nitrux 2.0 系统在空闲状态下的性能

现在是时候通过一些繁重的工作负载来运行它了。这包括一个文本编辑器、LibreOffice、文件管理器、图像查看器和 Firefox 的五个标签，其中一个标签正在播放一个 YouTube 视频。

你可以在下面的图片中看到资源使用的峰值。在这种状态下，它使用了接近 2GB 的内存，CPU 为 26%。和往常一样，Firefox 浏览器消耗了大部分资源。

![](https://mmbiz.qpic.cn/mmbiz_jpg/9aPYe0E1fb1NWMc7G53W4sOrW6kCOgv1F2zJfIptiaJf8usKuwQLOmgXcHp7qe8XqGBTibu9lyLbxwzia3yGf0cgQ/640?wx_fmt=jpeg)

Nitrux 2.0 系统在繁重工作状态下的性能

我想说的是，从性能上来说，它的表现还算可以。因为开箱即用的定制版，Latte Dock 和 Kvuntum 主题确实占用了一些资源。而这个指标在空闲和重载状态下要高于基本的 KDE Plasma。

#### 一些有问题的地方

不幸的是，在我的测试过程中，Nitrux 2.0 有几个小问题：

*   在我的 i3+SSD+4gb+NVIDIA+Broadcom 的旧系统中尝试安装了一个小时后 - 我无法让 Calamares 安装程序开始安装。
    
*   在临场 live 会话中没有检测到 Wi-Fi。然而，到目前为止，我在这个设备上测试的所有发行版都能检测到它。
    
*   KWin 在临场会话开始时崩溃了。
    
*   由于网络连接的原因，Calamares 安装程序的 “下一步” 按钮被禁用。这对我来说有点奇怪。
    
*   而最小的安装 ISO 也给出了 plymouth failed to start #17[5] 的错误。
    

然后我找到了已知问题部分，其中提到了一些问题。我确信这与 xanmod 版的 Linux 内核 5.16 有关。主线内核本来是没有问题的。

在虚拟机环境中，实验性的 Maui Shell 是不能使用的。点击和触摸操作大多不工作。但考虑到这是一个测试版本，这是可以理解的。

我觉得 Calamares 安装程序的错误需要在下一个版本之前进行更多的测试。

### 下载 Nitrux 2.0 +

你可以从以下链接下载最新版本：

*   下载 Nitrux[6]
    

### 结束语

如果你喜欢 KDE Plasma，并且不想在定制上花费太多精力，你可以选择 Nitrux 2.0。另外，许多用户喜欢类似于 Debian 的稳定性，而不要 systemd。那么，对他们来说，这是一个完美的选择。

但因为有一些错误，我不会向超小白的新用户推荐这个发行版。如果你对 Linux 有一定的了解，并且知道如何用命令行解决一些小问题，那么它就很适合你。你可以使用 Nitrux 2.0 作为你的日常系统。只是要谨慎对待 Debian 的不稳定软件包，这些软件包在更新后偶尔会出现问题。

加油！

### 参考资料

[1]

Nitrux Linux: _https://nxos.org/_

[2]

引入: _https://www.debugpoint.com/2022/01/maui-shell-first-look-1/_

[3]

Maui 应用程序: _https://www.debugpoint.com/2022/03/top-nitrux-maui-applications/_

[4]

virt-manager: _https://www.debugpoint.com/2020/11/virt-manager/_

[5]

plymouth failed to start #17: _https://github.com/Nitrux/nitrux-bug-tracker/issues/17_

[6]

下载 Nitrux: _https://nxos.org/download/standard/_