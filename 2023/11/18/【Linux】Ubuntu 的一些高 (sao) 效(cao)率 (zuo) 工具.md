> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/BZ47m17QtB3KiIfUmXxrBg)

你是否在用 Ubuntu 时为了找一个历史命令曾疯狂地按↑？

你是否因为手抖或者记不清名字经常输入错误指令？

你是否需要进行多任务而苦于频繁切换命令行终端？  

你是否因为长时间使用键盘和鼠标而感到肩颈难受？

……

不知道你有没有这些经历，反正我有！！直到之前一位朋友华哥和我推荐了几款工具之后，上面的情况就完美解决了。

今天，我就把这些高 (sao) 效(cao)率 (zuo) 工具整理一下，大家按需自提:-D

**１　分屏工具 tmux**

有时候，当处理多任务或者需要对比结果时，打开几个终端来回切换的确有点麻烦。

尤其对于 Vim 党来说，一个终端打天下，要是每次编辑完都得:wq，完了有问题再重新打开编辑，似乎也是不方便。

那么这个时候要是可以把一个终端屏幕分成几块，同时在一个窗口显示的话岂不是美滋滋！

比如你可以看到这样的界面。是不是有点酷？

![](https://mmbiz.qpic.cn/mmbiz_png/A58Z2c2MMZPlcAhCuMnBia2tfoxOgrL1vYyAQlUvvGyMeV1NicOxJmdeW2yWaiaQYicKZRt45iaCnfKwxNyibX9lA3Fw/640?wx_fmt=png)

嗯，tmux 就可以办到！

> tmux is a "terminal multiplexer: it enables a number of terminals (or windows), each running a separate program, to be created, accessed, and controlled from a single screen. tmux may be detached from a screen and continue running in the background, then later reattached."
> 
> https://wiki.archlinux.org/index.php/Tmux

看起来可还行，那就试试吧。不用怕试试就逝世。其实安装很简单的。

```
~$ sudo apt-get install tmux

```

就这么一行命令就安装好了，然后在终端输入 tmux 就可以用啦！

安装完之后你可能会觉得奇怪，为什么你的界面看起来如此单调，和我图中的颜值根本不是一回事？

别慌，这不是照骗。你还缺少对 tmux 的配置，需要 oh-my-tmux 的加持:-D

在 GitHub 搜索 oh my tmux 或者. tmux，这是个比较受欢迎的 tmux 配置项目。

![](https://mmbiz.qpic.cn/mmbiz_png/A58Z2c2MMZN9Bmqs5qySmVBjgLPBicKibtyWNEWp364TVjBlNsIZ6m3OjTRicNpKdrT94EPuiaYkTx8sX2CBnEBcuw/640?wx_fmt=png)

按照 README 文档操作就能配置好了。

```
~$ cd
~$ git clone https://github.com/gpakosz/.tmux.git
~$ ln -s -f .tmux/.tmux.conf
~$ cp .tmux/.tmux.conf.local .

```

你还可以安装 Powerline fonts，这样就能用上 Powerline 的一些标志图案。其实下一个工具也要用上它，干脆现在就装上吧。

```
~$ sudo apt-get install fonts-powerline

```

之后，再通过 Ctrl+a e 组合键打开. tmux.conf.local 文件，就可以自己更改里面的一些配置，弄好之后就能看到这般炫酷的界面啦！

当然徒有外表还不行，还得学一些常规的 tmux 操作，方便你我他:)

还是用刚才那张界面作为例子吧！

![](https://mmbiz.qpic.cn/mmbiz_png/A58Z2c2MMZPlcAhCuMnBia2tfoxOgrL1vlYq9PwuAiaShDYYWibbicp2mYLu1iauNuXCtkBOJhZdovRBWh04bfTzib1Q/640?wx_fmt=png)

上图最上侧的蓝色方框代表的是 tmux 的 Session(会话)，当你在终端通过 tmux 启动时，就会创建一个会话，如果没有特意命名，它就从０开始有固定的编号。

我们可以通过用以下指令对会话进行操作：  

```
~$ tmux new -s <session_name>   #创建一个名字为session_name的Session
~$ tmux ls   #显示当前存在的所有Sessions
~$ tmux detach   #解除当前Session，并未杀死，还在后台继续存在
~$ tmux attach -t <session_name>   #重新接上名字为session_name的Session
~$ tmux kill-session -t <session_name>   #杀死Session，再也找不回来

```

图中下侧的红色椭圆代表的是 tmux 的 Window(窗口)，当通过 tmux 启动并创建一个会话时，会同时创建一个窗口。如果没有特意命名，它就从１开始固定编号。

可以用快捷键 Ctrl+a c 创建一个新的窗口，Ctrl+a & 关闭当前窗口，Ctrl+a 1/2/3... 切换到特定编号的窗口。

一般用快捷键操作 tmux 的时候都要先加 Ctrl+b 作为前缀，而前面安装的 oh-my-tmux 配置可以用 Ctrl+a 来代替，毕竟按键 b 离 Ctrl 键还是有点远的 (~_~;)

最后，上图中间的绿色三角框代表的是 tmux 的 Pane(窗格)，当 tmux 启动时也会同时创建一个窗格。

比如下面就是通过 tmux 启动时的界面，它同时创建了一个会话 (蓝色)、一个窗口(红色) 和一个窗格(绿色)。

![](https://mmbiz.qpic.cn/mmbiz_png/A58Z2c2MMZPlcAhCuMnBia2tfoxOgrL1vQfgpOr6pOYGuX3kra0YkC92aXw1n3FgoCrOUl5Q63Pm6gxBew3ic70Q/640?wx_fmt=png)

我们前面说到的分屏，可以理解为在一个窗口中同时划分多个窗格，前面放的那张图就是一个窗口中划分了４个窗格。

以前需要开４个终端来操作，或者在同一个终端下操作，然后再往上翻记录，现在就可以同时展示在一个窗口中了！

对于在窗口中划分窗格，我们只需要熟悉那么几个常用的快捷操作就行。

Ctrl+a % 是将当前窗格均分为左右两格，Ctrl+a " 是均分为上下两格，Ctrl+a ↑/↓/←/→可以切换到其他窗格。

如果熟悉 Vim 的话，也可以通过 Ctrl+a k/j/h/l 切换窗格，如果要调整窗格大小则用对应按键的大写形式 Ctrl+a K/J/H/L。

当然操作不只这么一点，可以在网上找对应的教程，再进一步学习它的操作！

一开始我也会觉得这玩意对我没太大作用，可是用起来后莫名创造了一些其他需求！

比如有时候写个简单程序验证功能，懒得再动用 IDE 就可以通过分屏，一边用来写代码，另一边用来编译运行，如果报错就切换回代码那边继续修改...

![](https://mmbiz.qpic.cn/mmbiz_png/A58Z2c2MMZPlcAhCuMnBia2tfoxOgrL1v0O3mEmnZOiaE1d6EJfXIkyNAXibvEZy4Z9FwHBzQl3X3nRFKwVN407Jw/640?wx_fmt=png)

细心的你可能会发现，即使弄到这个地步，你的终端操作界面和我的还是有点不一样。下面就来介绍第二个高效工具——zsh！

**２　命令行工具 zsh**

zsh 其实是一个 shell，也就是命令行解释器，在 Ubuntu 下默认的 shell 是 bash，可以通过指令 echo $SHELL 查看当前的 shell。

bash 也有一些方便的操作，比如 tab 键可以补全输入命令或者文件，↑/↓键可以找到往前或者往后输入的命令。

但多多少少还是不够方便，比如历史命令比较多，得不停地往前翻才能找到。或者要是记错命令或者文件名，再怎么 tab 也没办法补全！！

这时候 zsh 的优势就体现出来了，这玩意真是谁用谁知道。

和 tmux 一样，一条命令语句就搞定安装了！

```
~$ sudo apt-get install zsh

```

为了方便起见，直接将 zsh 设为默认的 shell，bash 自此可以下班了...  

```
~$ chsh -s $(which zsh)

```

重启后打开终端测试一下。

```
~$ echo $SHELL
~$ $SHELL --version

```

如果显示 / usr/bin/zsh 和 zsh 5.4.2 或者类似的，就说明已经安装并且设置好啦:-D

装好之后就可以享受 zsh 行云流水般的操作，比如下面这个。

![](https://mmbiz.qpic.cn/mmbiz_gif/A58Z2c2MMZPlcAhCuMnBia2tfoxOgrL1vSXIWvelZM9gKiaySp5lsayP5CMQmkLukVtpzaibYia8klm1nNp8yNngcA/640?wx_fmt=gif)

你会发现，切换到某个路径下不需要用 cd 命令，直接输入路径就可以了。

当然如果习惯 cd 操作这个倒也无可厚非，比如我下面的 cd build && make -j4 命令，现在都已经形成肌肉记忆了，要改的话还需要时间。

关键是，输入指令不用这么准确也能补全了！比如输入 zq/sl/orb 就能补全正确路径 ZQC/SLAM/ORB_SLAM2，简直不要太友好。

有时记错名字或者手抖打漏了某个字符，zsh 也能自动帮你修正过来: D

如果你的命令比较模糊，没办法一下子补全的话，tab 还有一个更神奇的地方，就是可以让你手动选择，看下图。

![](https://mmbiz.qpic.cn/mmbiz_gif/A58Z2c2MMZPlcAhCuMnBia2tfoxOgrL1vGXVRiaMuJMw3ImRaTLiaZqia5KFaZa4pnD3icPiaDiadYAgSpaNjJ9MB7wSw/640?wx_fmt=gif)

当补全比较模糊的时候，按两次 tab 键就会弹出所有可补全的结果，通过↑/↓/←/→就可以选择你想要的命令或者路径。

就连命令的参数也是可以补全的，上图的 git commit -<tab> 就是对命令参数的补全选择。是不是已经忍不住要安装 zsh 了，心动不如行动！

你又发现了，为什么你的界面和我的还是不一样？和 tmux 一样，你还缺一个 oh-my-zsh！0.0

这个 oh-my-zsh 可比 oh-my-tmux 要强大，它的网站上第一句话就是 “Your terminal never felt this good before.”

> Oh My Zsh is a delightful, open source, community-driven framework for managing your Zsh configuration. It comes bundled with thousands of helpful functions, helpers, plugins, themes, and a few things that make you shout...
> 
> https://ohmyz.sh/

嗯，名副其实，好歹也是一个 11 万 Star 的项目啊！

也是只要一条命令就搞定安装  

```
~$ sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

```

你可以设置自己喜欢的主题，还可以安装各种插件。当然，要是喜欢倒腾还可以 DIY。

详情请查阅 GitHub 的 Wiki，里面有很详细的介绍，我这里就不多说了！

我现在用的主题是 agnoster，感觉就很可。

如果当前路径是一个 git 仓库，它会显示当前所在的分支。要是仓库没有变化，那么分支就是绿色的；如果做了某些更改，就会变成黄色: D

![](https://mmbiz.qpic.cn/mmbiz_png/A58Z2c2MMZPlcAhCuMnBia2tfoxOgrL1vI9x8icWBOS3W5SnySNscA0tkOnHLoShenX7v3K2zDTrxdic1kLdukXIw/640?wx_fmt=png)

还有一点很高效的就是，当你输入命令的一部分时，按↑/↓键就可以翻看具有相同字符的历史命令！

比如下图，输入 (c 再按↑键就能找到以往的 (cd build && make -j4) 命令，输入 xdg 再按↑键也是一样。如此这般就能很快定位到自己想要的历史命令啦 0.0

![](https://mmbiz.qpic.cn/mmbiz_gif/A58Z2c2MMZPlcAhCuMnBia2tfoxOgrL1vt1CmM9ugscOOIDv7DJFlx2D5J5qIQBT3icHpdzCRYyyHxRTjFNTPRow/640?wx_fmt=gif)

但有些命令比较久远了，这么上下翻一次只能看一条还是嫌不够方便怎么办？还有一个小工具可以帮上忙！

**３　模糊搜索工具 fzf**

fzf 是一个命令行的模糊搜索工具，它搭配 zsh 一起使用简直太爽了。

以前搜索历史命令只能一条条往回翻，而 fzf 可以一次性全展示出来。

用 bash 的话历史记录只能存 1 千条，而 zsh 可以存５万条，这还不香？我们可以通过 echo $HISTSIZE 命令看一下就知道了！

它的安装命令又是短短的两条就完事。

```
~$ git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~$ ~/.fzf/install

```

安装完后重启终端，通过 Ctrl+r 键就可以打开 fzf，输入要搜索的关键字符就能找到历史命令，多模糊的都能给你找出来。

![](https://mmbiz.qpic.cn/mmbiz_gif/A58Z2c2MMZPlcAhCuMnBia2tfoxOgrL1vsAiaFJyZ2UDibqgibqC6KwsdWECvtKzPkORpKqOUib74csZK8iahu3ugoLA/640?wx_fmt=gif)

如果嫌搜索结果太杂，通过在搜索字符前加'就能搜索完全匹配字符的历史命令！

当然还有更多的操作，可以到 GitHub 的 Wiki 深入挖掘一下。篇幅好像有点长，就不再做过多介绍了。

以上的三个工具都是针对命令行的，所以在 Ubuntu 命令行模式也是适用的！

****４　Chr****ome 插件 Vimium****

还有最后一个神器，可以让你在用 Chrome 浏览器时，基本忘记了鼠标的存在: D

那就是 Chrome 的插件 Vimium，一看这名字肯定就知道它和 Vim 有什么瓜葛。

的确，它的一些操作键位和 Vim 很类似。装了这个插件，就可以直接通过键盘操作浏览和控制 Chrome 了。

一般的操作流程就是：t 打开新标签 -> 在地址栏输入网址或搜索内容 -> 回车 ->f/F 选择链接 ->k/j/h/l 上下左右滚动屏幕 ->J/K 左右切换标签...

f/F 可以把界面中可以跳转的都用字母标出来了，想选哪个就按相应的字母键，相当于用鼠标点击链接。

![](https://mmbiz.qpic.cn/mmbiz_png/A58Z2c2MMZPQLbpnoe89t9DI0yCcQ3kyhicichaBlIr8wQYQYdqMQFEEo9QMRkficib6CwdmgrwcXjfv4z8JjdBpew/640?wx_fmt=png)

看着似乎很繁琐，可是只要花十几分钟把几个键位练熟，你真的会觉得碰一次鼠标都嫌碍事！

即使忘了快捷键，在界面敲个?，就能显示出来，贴不贴心？

![](https://mmbiz.qpic.cn/mmbiz_png/A58Z2c2MMZPQLbpnoe89t9DI0yCcQ3kyQDjIeuuB5tHeWITrjCSj5zESSicUMTK4ODibunqqdLJXjzWHQPQCvwgg/640?wx_fmt=png)

某些情况下 Vimium 不能操作，这时再搭配一些 Chrome 原生的快捷键，就真的完美了！

不知道为什么，我在实验室长时间使用鼠标的话右肩颈会酸痛，而上面４个工具居然成功解决了这个问题 0.0

习惯了键盘操作之后，现在每次切换到 Windows 系统都会很不习惯，相信你用上一段时间也会有这种感觉！

也许你会认为这些工具看上去很麻烦，但是只要花点时间练练基本操作，等到用时需要什么再查什么。假以时日，在外人看来就是一顿操作猛如虎了。

配置一个赏心悦目的环境，外加一些行云流水的快捷键，可以让日常的学习工作多一份乐趣，也是对自己好的一种方式: D