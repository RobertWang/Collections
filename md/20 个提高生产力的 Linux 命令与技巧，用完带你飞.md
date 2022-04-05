> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650536153&idx=3&sn=6a70980d68e9de60b5828e5969ddc080&chksm=83ba9d3db4cd142b99fc0e0a975618357dee674df24dc27164acf6c921142be2a2dd65f4062f&mpshare=1&scene=1&srcid=0403CZRQV4eYcm3KZnPCJ2g3&sharer_sharetime=1648982389997&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

> **文章来源 ：****云计算就该这么学**

在本文中，我将向您演示一些专业的 Linux 命令技巧，这些技巧将使您节省大量时间，在某些情况下还可以避免很多麻烦，而且它也将帮助您提高工作效率。

并不是说这些只是针对初学者的 Linux 技巧。即使有经验的 Linux 用户也有可能没有发现这些，尽管你这些年来一直在使用 Linux。很酷的 Linux 终端技巧，帮助您节省时间和提高生产力

您很可能已经知道这些 Linux 命令中的一些或全部。无论哪种情况，都欢迎您在评论部分中分享您喜欢的技巧。其中一些技巧还取决于 shell 的配置方式。现在让我们开始！

**1、使用 tab 键进行自动完成**

我将从一些看得见但又非常重要的事情开始：tab 补全。当您开始在 Linux 终端中键入内容时，您可以按 Tab 键，它会建议所有可能的选项，这些选项以您到目前为止所键入的字符串开头。例如，如果您要复制名为 linuxidc.txt 的文件，则只需键入 “cp l”，然后按 tab 键查看可能的选项。

![](https://mmbiz.qpic.cn/mmbiz_png/SBMicibv4qpZSs8E46OVVMVqqWwbusQrk2ejHFmIF03k0Ayy6GJ9N2At5CLQOY97yGzHUBYDgicP3tDIgRI1fn03w/640?wx_fmt=png)

使用 Tab 键进行自动完成  

您也可以在完成命令时使用 Tab 键。

**2、切换回上一个工作目录**

假设您以长目录路径结尾，然后转到完全不同的路径中的另一个目录。然后您意识到必须返回到先前所在的目录。在这种情况下，您要做的就是键入以下命令：

```
cd -
```

这会将您带回到上一个工作目录。您无需输入长目录路径，也无需复制粘贴。

![](https://mmbiz.qpic.cn/mmbiz_png/SBMicibv4qpZSs8E46OVVMVqqWwbusQrk2WtHjZDOJyGTqiaFFTxdMNryESkYe1AbrSe8FibvbFtGmV8WSWicdEbTow/640?wx_fmt=png)

在目录之间轻松切换  

如果如下所示：

```
[linuxidc@localhost ~/www.linuxidc.com]$cd -bash: cd: OLDPWD 未设定
```

![](https://mmbiz.qpic.cn/mmbiz_png/SBMicibv4qpZSs8E46OVVMVqqWwbusQrk2gr7sX3qZn6YfNRFUnFWDHiaE9DkmFt6opoWuWhiciciaEbEcSylwe1m4Kw/640?wx_fmt=png)

是因为 cd 命令设置了 OLDPWD 环境变量值。除非你至少执行了一次 cd 命令，否则 OLDPWD 环境变量不会包含任何值 cd - 和 cd $OLDWPD 命令的执行结果并非在所有环境下都相同。  

**3、返回主目录**

这太明显了。您可以使用以下命令从 Linux 命令行中的任何位置移至主目录：

```
cd ~
```

但是，您也可以仅使用 cd 返回主目录：

```
cd
```

大多数现代 Linux 发行版均已为此命令预配置了 shell。在这里至少可以节省两次击键。

![](https://mmbiz.qpic.cn/mmbiz_png/SBMicibv4qpZSs8E46OVVMVqqWwbusQrk2bic2ReDy9dRxc3KVcz8VEIGLUeohAN5bktIRqj9X8Yg6c2vB5qTsxCg/640?wx_fmt=png)

快速返回主目录  

**4、列出目录的内容**

您一定在想在列出目录内容的命令中还有了什么技巧。每个人都知道在这种情况下使用 ls -l。

就是这样。大多数人使用 ls -l 来列出目录的内容，而同样的事情也可以用下面的命令来完成：

```
ll<br mp-original-font-size="12" mp-original-line-height="19" style="outline: 0px;max-width: 100%;box-sizing: border-box;line-height: 19px;word-wrap: break-word !important;" data-darkmode-bgcolor-164918539285910="rgb(49, 54, 63)" data-darkmode-original-bgcolor-164918539285910="#fff|rgb(255, 255, 255)|rgb(40, 44, 52)" data-darkmode-color-164918539285910="rgb(171, 178, 191)" data-darkmode-original-color-164918539285910="#fff|rgb(0, 0, 0)|rgb(171, 178, 191)">
```

同样，这也取决于 Linux 发行版和 shell 配置，但是您很可能能够在大多数 Linux 发行版中使用它。

![](https://mmbiz.qpic.cn/mmbiz_png/SBMicibv4qpZSs8E46OVVMVqqWwbusQrk2fdhDIweRJ6H4XAzKP4arDndCcstxGQd1TMcPB2iblVI6cOZnMBuibgsA/640?wx_fmt=png)

使用 ll 而不是 ls -l  

**5、在一个命令中运行多个命令**

假设您必须一个接一个地运行几个命令。您是否在等待第一个命令完成运行，然后执行下一个命令？

那么，您可以使用 “;” 分隔符。这样，您可以在一行中运行许多命令。无需等待先前的命令完成后再执行其他任务。

```
command_1; command_2; command_3<br mp-original-font-size="12" mp-original-line-height="19" style="outline: 0px;max-width: 100%;box-sizing: border-box;line-height: 19px;word-wrap: break-word !important;" data-darkmode-bgcolor-164918539285910="rgb(49, 54, 63)" data-darkmode-original-bgcolor-164918539285910="#fff|rgb(255, 255, 255)|rgb(40, 44, 52)" data-darkmode-color-164918539285910="rgb(171, 178, 191)" data-darkmode-original-color-164918539285910="#fff|rgb(0, 0, 0)|rgb(171, 178, 191)">
```

**6、仅在上一个命令成功的情况下，才能在一个命令中运行多个命令**

在上一个命令中，您了解了如何在一个命令中运行多个命令以节省时间。但很多时候你必须确保命令不会失败才能执行下一条命令，那怎么办？

比如您要构建代码，然后在构建成功的情况下才接着运行 make。

在这种情况下，可以使用 && 分隔符。&& 确保下一条命令仅在上一条命令成功执行时运行。

```
command_1 && command_2<br mp-original-font-size="12" mp-original-line-height="19" style="outline: 0px;max-width: 100%;box-sizing: border-box;line-height: 19px;word-wrap: break-word !important;" data-darkmode-bgcolor-164918539285910="rgb(49, 54, 63)" data-darkmode-original-bgcolor-164918539285910="#fff|rgb(255, 255, 255)|rgb(40, 44, 52)" data-darkmode-color-164918539285910="rgb(171, 178, 191)" data-darkmode-original-color-164918539285910="#fff|rgb(0, 0, 0)|rgb(171, 178, 191)">
```

此命令的一个很好的例子是当您使用 sudo apt update && sudo apt upgrade 升级系统时。

**7、轻松搜索您使用过的命令**

想象一下一种情况，您在几分钟 / 几小时前使用了很长的命令，而您不得不再次使用它。问题是您不再记得确切的命令了。

反向搜索是您的救星。您可以使用搜索词在历史记录中搜索命令。

只需使用 ctrl + r 键即可启动反向搜索并键入命令的某些部分。它将查询历史记录，并向您显示与搜索词匹配的命令。

```
ctrl + r 搜索词<br mp-original-font-size="12" mp-original-line-height="19" style="outline: 0px;max-width: 100%;box-sizing: border-box;line-height: 19px;word-wrap: break-word !important;" data-darkmode-bgcolor-164918539285910="rgb(49, 54, 63)" data-darkmode-original-bgcolor-164918539285910="#fff|rgb(255, 255, 255)|rgb(40, 44, 52)" data-darkmode-color-164918539285910="rgb(171, 178, 191)" data-darkmode-original-color-164918539285910="#fff|rgb(0, 0, 0)|rgb(171, 178, 191)">
```

默认情况下，它将仅显示一个结果。要查看更多与您的搜索字词匹配的结果，您将不得不反复使用 ctrl + r。要退出反向搜索，只需使用 Ctrl + C。

![](https://mmbiz.qpic.cn/mmbiz_png/SBMicibv4qpZSIJecddCd0eiankShnSku5m4QW3sjVw7uciaHo4BXKDa1gGBOUjx7DoZiacrsXia4C0CMAUXyI0GC53w/640?wx_fmt=png)

在命令历史记录中进行反向搜索

请注意，在某些 Bash Shell 中，还可以在搜索词中使用 Page Up 和 Down 键，它将自动完成命令。

**8、解除 Linux 终端意外冻结的 Ctrl + S**

在很多类 Unix 的系统上，Ctrl-S 都有特殊的含义：它会 “冻结” 终端（它曾经被用来暂停快速滚动）。因为 “保存” 一般也是用这个快捷键，所以经常会有人不假思索地按下这个快捷键，结果大多数人都会被搞糊涂（我也经常犯这个错误）。解冻终端是用 Ctrl-Q，所以如果你忽然发觉终端看起来被冻结了，试一下 Ctrl-Q，看能不能释放它。

**9、移至行首或行尾**

假设您正在键入一个长命令，并且在途中您意识到必须在开始时进行一些更改。您将使用几次向左键击移动到行的开头。并且类似地进行到该行的末尾。

当然，您可以在此处使用 Home 和 End 键，但是也可以使用 Ctrl + A 转到行的开头，并使用 Ctrl + E 转到结尾。

图示如下

![](https://mmbiz.qpic.cn/mmbiz_png/SBMicibv4qpZSIJecddCd0eiankShnSku5mMibo8DDwW8OLNqIg6OXbVNo7a1SB0TlHrYeChBRicRicb8zTAuOyfrlHA/640?wx_fmt=png)

移至该行的开头或结尾

我发现它比使用 Home 和 End 键更方便，尤其是在笔记本电脑上。

**10、实时读取日志文件**

在需要在应用程序运行时分析日志的情况下，可以将 tail 命令与 - F 选项一起使用。

```
tail -F linuxidc_log<br mp-original-font-size="12" mp-original-line-height="19" style="outline: 0px;max-width: 100%;box-sizing: border-box;line-height: 19px;word-wrap: break-word !important;" data-darkmode-bgcolor-164918539285910="rgb(49, 54, 63)" data-darkmode-original-bgcolor-164918539285910="#fff|rgb(255, 255, 255)|rgb(40, 44, 52)" data-darkmode-color-164918539285910="rgb(171, 178, 191)" data-darkmode-original-color-164918539285910="#fff|rgb(0, 0, 0)|rgb(171, 178, 191)">
```

等同于—follow=name —retry，根据文件名进行追踪，并保持重试，即该文件被删除或改名后，如果再次创建相同的文件名，会继续追踪。

![](https://mmbiz.qpic.cn/mmbiz_png/SBMicibv4qpZSIJecddCd0eiankShnSku5m1pjhLNvwNmWrelwtLelsaqydpH7LMPWhJPSWEJETRKV4SfEK71zAAw/640?wx_fmt=png)

**11、读取压缩日志而不解压缩**

服务器日志通常被 gzip 压缩以节省磁盘空间。这给分析日志的开发人员或系统管理员带来了一个问题。您可能必须将其 scp 到本地，然后提取它来访问文件，因为有时您没有提取日志的写权限。

值得庆幸的是，在这种情况下，z 命令可以帮助您。z 命令提供了用于处理日志文件（例如 less，cat，grep 等）的常规命令的替代方法。

这样您就可以使用 zless，zcat，zgrep 等命令查看压缩包的内容，甚至不必显示提取压缩文件。

```
[linuxidc@localhost ~/www.linuxidc.com]$zcat linuxidc_log.zip | more
```

![](https://mmbiz.qpic.cn/mmbiz_png/SBMicibv4qpZSs8E46OVVMVqqWwbusQrk24ibX8EDSqYkqYUehIHicyjnVPuGYicesyxicqL773WsXOdLWMMCHteF1Lg/640?wx_fmt=png)

不解压缩读取压缩文件  

**12、使用 less 读取文件**

要查看文件的内容，cat 不是最佳选择，特别是如果文件很大。cat 命令将在屏幕上显示整个文件。

您可以使用 Vi，Vim 或其他基于终端的文本编辑器，但是如果您只想读取文件，则 less 命令是更好的选择。

```
less -N linuxidc.txt<br mp-original-font-size="12" mp-original-line-height="19" style="outline: 0px;max-width: 100%;box-sizing: border-box;line-height: 19px;word-wrap: break-word !important;" data-darkmode-bgcolor-164918539285910="rgb(49, 54, 63)" data-darkmode-original-bgcolor-164918539285910="#fff|rgb(255, 255, 255)|rgb(40, 44, 52)" data-darkmode-color-164918539285910="rgb(171, 178, 191)" data-darkmode-original-color-164918539285910="#fff|rgb(0, 0, 0)|rgb(171, 178, 191)">//按下v键来编辑文件<br mp-original-font-size="12" mp-original-line-height="19" style="outline: 0px;max-width: 100%;box-sizing: border-box;line-height: 19px;word-wrap: break-word !important;" data-darkmode-bgcolor-164918539285910="rgb(49, 54, 63)" data-darkmode-original-bgcolor-164918539285910="#fff|rgb(255, 255, 255)|rgb(40, 44, 52)" data-darkmode-color-164918539285910="rgb(171, 178, 191)" data-darkmode-original-color-164918539285910="#fff|rgb(0, 0, 0)|rgb(171, 178, 191)">//退出编辑器后，你可以继续用less浏览了<br mp-original-font-size="12" mp-original-line-height="19" style="outline: 0px;max-width: 100%;box-sizing: border-box;line-height: 19px;word-wrap: break-word !important;" data-darkmode-bgcolor-164918539285910="rgb(49, 54, 63)" data-darkmode-original-bgcolor-164918539285910="#fff|rgb(255, 255, 255)|rgb(40, 44, 52)" data-darkmode-color-164918539285910="rgb(171, 178, 191)" data-darkmode-original-color-164918539285910="#fff|rgb(0, 0, 0)|rgb(171, 178, 191)">
```

您可以在更少的范围内搜索字词，按页移动，高亮与行号等。

**13、使用 !$ 重新使用上一个命令中的最后一项**

在许多情况下，使用上一个命令的参数很方便。

假设您必须创建一个目录，然后进入新创建的目录。那么，您可以使用!$ 选项。

![](https://mmbiz.qpic.cn/mmbiz_png/SBMicibv4qpZSs8E46OVVMVqqWwbusQrk2h2MpkhF6Zq6iba7gwOKy41Wa9gky9wAcQdM3Ixsic7WGr9z4KkRnNHcg/640?wx_fmt=png)

使用 !$  

更好的方法您可以使用使用 alt + . 。在最后一个命令的选项之间来回移动的次数。

**13、用!! 重用当前命令中的上一个命令。**

您可以使用!! 调用前面的整个命令。当您必须运行一个命令并意识到它需要 root 特权时，这一点特别有用。

一个快速 sudo !! 省去了很多击键。

![](https://mmbiz.qpic.cn/mmbiz_png/SBMicibv4qpZSs8E46OVVMVqqWwbusQrk2qWeW6EFGw77JbATPX2WxSnJnwN4LJWicXFn1jqLltLq1Kkb1sG982Fg/640?wx_fmt=png)

用!! 重用当前命令中的上一个命令。  

**15、使用别名来修正错别字**

您可能已经知道 Linux 中的别名命令是什么。你能做的是，用它们来修正打字错误。

例如，您可能经常将 grep 输入为 gerp。如果您以这种方式在您的 bashrc 中放置一个别名：

```
alias gerp=grep
```

这样，您无需再次输入命令。

**16、在 Linux 终端中复制粘贴**

这一点有点模棱两可，因为它取决于 Linux 发行版和终端应用程序。但通常，您应该能够使用以下快捷键复制粘贴命令：

选择要复制的文本，然后右键单击以粘贴（在 Putty 和其他 Windows SSH 客户端中有效）

选择要复制的文本，然后单击鼠标中键（滚动按钮）以进行粘贴 Ctrl + Shift + C 表示复制，Ctrl + Shift + V 表示粘贴

**17、终止正在运行的命令 / 进程**

这可能太明显了。如果有一个命令正在运行运行，并且您想退出该命令，则可以按 Ctrl + C 停止该正在运行的命令。

**18、清空文件而不删除它** 

如果只想清空文本文件的内容而不删除文件本身，则可以使用类似于以下命令：

```
> 文件名<br mp-original-font-size="12" mp-original-line-height="19" style="outline: 0px;max-width: 100%;box-sizing: border-box;line-height: 19px;word-wrap: break-word !important;" data-darkmode-bgcolor-164918539285910="rgb(49, 54, 63)" data-darkmode-original-bgcolor-164918539285910="#fff|rgb(255, 255, 255)|rgb(40, 44, 52)" data-darkmode-color-164918539285910="rgb(171, 178, 191)" data-darkmode-original-color-164918539285910="#fff|rgb(0, 0, 0)|rgb(171, 178, 191)">
```

**19、查找是否有包含特定文本的文件**

在 Linux 命令行中有多种搜索和查找方法。但是，当您只想查看是否有包含特定文本的文件时，可以使用以下命令：

```
grep -Pri 要搜索的字符串 路径<br mp-original-font-size="12" mp-original-line-height="19" style="outline: 0px;max-width: 100%;box-sizing: border-box;line-height: 19px;word-wrap: break-word !important;" data-darkmode-bgcolor-164918539285910="rgb(49, 54, 63)" data-darkmode-original-bgcolor-164918539285910="#fff|rgb(255, 255, 255)|rgb(40, 44, 52)" data-darkmode-color-164918539285910="rgb(171, 178, 191)" data-darkmode-original-color-164918539285910="#fff|rgb(0, 0, 0)|rgb(171, 178, 191)">
```

我强烈建议您精通 find 命令。

**20、对任何命令都可使用帮助命令（help）**

最后我将用一个更明显但却非常重要的 “技巧” 来结束本文，即使用命令或命令行工具的帮助命令（help）。

几乎所有的命令和命令行工具都带有一个帮助页面，显示如何使用该命令。经常使用帮助会告诉你这个工具 / 命令的基本用法。

比如 bc 命令的帮助：

```
[linuxidc@localhost ~/www.linuxidc.com]$bc -help
```

![](https://mmbiz.qpic.cn/mmbiz_png/SBMicibv4qpZSs8E46OVVMVqqWwbusQrk2kxdKibib37kvAUj0SicNqfL137Zcc4qicBGP3HDUqicyJBDAm8bXGZhab8Q/640?wx_fmt=png)