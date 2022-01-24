> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [sspai.com](https://sspai.com/post/71018)

> 终端是很多人日常打交道的工具之一。

终端是很多人日常打交道的工具之一。比如，深度学习是目前一个大热的研究课题，由于训练和推理过程需要强大的 GPU，研究生们共享 GPU 服务器，并通过终端使用 SSH 连接并编写代码。而公司中的运维人员，也需要通过 SSH 进行远程访问服务器，对系统或服务进行维护。

在日常操作中，最大的问题就是如何优雅地从远程主机中复制文本并粘贴到本机中。

问题：跨平台系统剪贴板不通
-------------

远程主机一般我们不能直接通过显示器和键盘控制，也没有图形界面，所有的操作都需要通过 SSH 访问并在终端操作。

而本机和远程主机都各自有自己的一套剪切板，在命令操作中复制的文本只会保存在远程主机的剪贴板中。那我们通过终端的图形界面选择并复制不可以吗？

![](https://cdn.sspai.com/editor/u_/c7j37vtb34ta8bd6mda0.png)

上图是一个非常日常的场景，我们需要从一个配置文件里复制一行或是多行的代码，这里我们选择的工具是 VIM。但我们真的开始复制时就会遇到如下的困难：

*   困难一：如果只想要复制正文，那么左侧的代码行号也不得不被选择。尽管这个可以通过绑定快捷键快速开关行号。
*   困难二：细心观察，图中文本的第三行超过了终端的宽度，自动换行到下一行展示。如果同时选择这两行，粘贴出来的效果也会是两行而不是它本该的一行。
*   困难三：整份文档并不只有 24 行，如果想要复制整份文档，则不得不分几次逐次选择并复制全终端屏幕。

而这些情况也的确困扰了我很久。

解决方案：OSC codes
--------------

OSC 代表的是 Operating System Controls, 是一种约定俗成的用于终端程序中的逃逸序列表达，终端会根据 OSC codes 所定义的行文处理方式处理它所包围的文本。

而正巧的是就有一种定义决定了「如何从终端中复制内容到系统剪贴板中」，那就是 [OSC 52 escape sequence](https://chromium.googlesource.com/apps/libapps/ /master/nassh/doc/FAQ.md#Is-OSC-52-aka-clipboard-operations_supported)。

OSC 52 一次最长接受 100000 个字节，其中前 7 个字节为 `"\_033_]_52_;_c_;"`，中间 99992 个字节为待复制文本，最后一个字节为 `"\a"`。待复制文本需要编码为 base64 表达，因此实际可用的复制长度为 74994 个字节。

一般 74994 个字节可以超过普通的纯文本范围了，完全能够满足日常的复制粘贴的需求。接下来就是要思考如何将 OSC 52 的定义优雅地应用到我们的日常工作流中。

就那我常用的两个工作软件举例，一个是 tmux 用于持久化会话，另一个是 vim 用于编辑文本文件，而其他软件可以借助搜索引擎输入`软件名 OSC52` 获得有效的解决办法。

很幸运的是，我通过搜索引擎找到了 Github 上的一个开源 [Bash 的 OSC 52 实现](https://github.com/sunaku/home/blob/master/bin/yank)，顺藤摸瓜，我进一步找到了 [@sunaku](https://github.com/sunaku) 制作的 [tmux](https://github.com/sunaku/home/blob/master/.tmux.conf.erb) 和 [vim](https://github.com/sunaku/.vim/blob/master/plugin/yank.vim) 的 OSC 52 实现，在此感谢他们的贡献。

首先，我在这个路径`~/.local/bin/` 中创建了名为 yank 文件，赋予执行权限，并加入到 PATH 路径中。

```
#!/bin/sh

# copy via OSC 52
buf=$( cat "$@" )
len=$( printf %s "$buf" | wc -c ) max=74994
test $len -gt $max && echo "$0: input is $(( len - max )) bytes too long" >&2
printf "\033]52;c;$( printf %s "$buf" | head -c $max | base64 | tr -d '\r\n' )\a"
```

接下来就要让 VIM 实现 OSC52 了，因为我主要用 neovim，所以日常使用 packer.nvim 管理我使用的插件。因此在 `~/.config/nvim/lua/plugins.lua` 中加入了一个依赖 `ojroques/vim-oscyank`。

```
return require('packer').startup({function(use)
  use 'ojroques/vim-oscyank'
end})
```

复制的动作是通过 VIM 的 visual 模式选择中想要复制的文本，通过 vim 命令 `:OSCYank` 即可快速复制，然后在本机中随意粘贴。

在 tmux 实现 OSC52 页很简单，只需要在 `~/.tmux.conf` 中添加一行绑定快捷键 Y 即可。

```
# transfer copied text to attached terminal with yank
bind-key -T copy-mode-vi Y send-keys -X copy-pipe 'yank > #{pane_tty}'
```

使用方法是先触发 tmux 的热键并依次键入序列：`ctrl+b`, `[`, `v`, `y`。其中，ctrl+b 是 tmux 的热键，[ 进入会话冻结状态， 代表使用 vim 方式控制光标，v 进入选择模式并选择文本，最后 y 复制选中的文本。然后就可以在本机任意粘贴了。

当然有的时候可能既不会打开 tmux 也不会使用 VIM，那么我们还可以在普通的终端环境下，通过管道直接将文件输送给 yank 命令进行复制，具体命令如下：

```
$ cat your_file.txt | yank
```

接下来只需要在本机粘贴即可。

唯二的使用限制
-------

虽然有了代码，但想要实现上面的效果，还有两个硬性条件：

*   远程主机中运行软件或编辑器需要支持 OSC
*   本地的终端需要支持 OSC

必须在支持 OSC 的软件或者编辑器下，才能用 OSC 52 来传递复制的内容，比如我上面提到的 tmux 和 vim。当然，这个动作可能还需要额外的脚本 / 插件来辅助，但总比手动一个个复制强。

而本地终端作为「桥梁」也必须支持 OSC 才能正确传递 OSC 信息，因此我搜索并汇总了一些常见的平台的终端模拟器软件对于 OSC 52 支持情况：

*   Windows 平台 - Windows Terminal：支持
*   Mac 平台 - iTerm2：支持
*   Mac 平台 - terminal.app：不支持
*   Ubuntu 平台 - Gnome Terminal：不支持
*   Chromebook - hterm：支持
*   跨平台 - alacritty：支持
*   跨平台 - kitty：支持
*   终端复用 - tmux：支持
*   终端复用 - screen：支持

![](https://cdn.sspai.com/editor/u_/c7j3805b34ta854v5c30.gif)

最后是一个展示，希望能帮助到各位终端人提高效率，下次见。