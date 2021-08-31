> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [jingyan.baidu.com](https://jingyan.baidu.com/article/380abd0a4870111d90192cae.html)

> 多终端管理器 TMUX 使用详解

在日常工作中，总是感觉用 PUTTY 连接 Linux 一个窗口不够用，再开新的窗口又比较麻烦，于是想到是否可以在一个 SSH 会话中可以打开多个终端，最后我找到了很强大而且使用广泛的 tmux 多终端管理器。

tmux 是一个优秀的终端复用软件，类似 GNU Screen，但来自于 OpenBSD，采用 BSD 授权。使用它最直观的好处就是，通过一个终端登录远程主机并运行 tmux 后，在其中可以开启多个控制台而无需再使用更多的 SSH 会话来连接这台远程主机；其功能远不止于此。

1、安装

在 freebsd 中可以直接使用 ports 工具安装，位置在：/usr/ports/sysutils/tmux/，ubuntu 系统下默认自带 byou，与 tmux 很像，甚至快捷键都是一样的。这里只详细说明在 Centos6.3 下如何安装和使用 tmux 的。

Centos6.3 的软件库里没有 tmux，只有 screen，所以要想使用 tmux 需要自己编译安装。

(1) 下载 tmux：

wget http://sourceforge.net/projects/tmux/files/tmux/tmux-1.6/tmux-1.6.tar.gz/download

(2) 编译安装：

tar zxvf tmux-1.6.tar.gz

cd tmux-1.6

./configure

make;make install

2、启动 tmux

安装完成后输入命令 tmux 即可打开软件，界面十分简单，类似一个下方带有状态栏的终端控制台；但根据 tmux 的定义，在开启了 tmux 服务器后，会首先创建一个会话，而这个会话则会首先创建一个窗口，其中仅包含一个面板；也就是说，这里看到的所谓终端控制台应该称作 tmux 的一个面板，虽然其使用方法与终端控制台完全相同。

tmux 使用 C/S 模型构建，主要包括以下单元模块：

一个 tmux 命令执行后启动一个 tmux 服务

一个 tmux 服务可以拥有多个 session，一个 session 可以看作是 tmux 管理下的伪终端的一个集合

一个 session 可能会有多个 window 与之关联，每个 window 都是一个伪终端，会占据整个屏幕

一个 window 可以被分割成多个 pane

多个 pane 的编号规则，以 3 个 pane 为例:

0

1 | 2

3、tmux 快捷键

tmux 在会话中使用大量的快捷键来控制多个窗口、多个会话等。

[cpp] view plaincopy 在 CODE 上查看代码片派生到我的代码片

Ctrl+b // 激活控制台；此时以下按键生效

系统操作

tmux ls // 列出已有会话 (list-sessions)

tmux a -t 1// 来连接到第一个会话

c-b t // 钟表

? // 列出所有快捷键；按 q 返回

d // 脱离当前会话；这样可以暂时返回 Shell 界面，输入 tmux attach 能够重新进入之前的会话

D // 选择要脱离的会话；在同时开启了多个会话时使用

Ctrl+z // 挂起当前会话

r // 强制重绘未脱离的会话

s // 选择并切换会话；在同时开启了多个会话时使用

: // 进入命令行模式；此时可以输入支持的命令，例如 kill-server 可以关闭服务器

[ // 进入复制模式；此时的操作与 vi/emacs 相同，按 q/Esc 退出

~ // 列出提示信息缓存；其中包含了之前 tmux 返回的各种提示信息

窗口操作

c // 创建新窗口

& // 关闭当前窗口

数字键 // 切换至指定窗口

p // 切换至上一窗口

n // 切换至下一窗口

l // 在前后两个窗口间互相切换

w // 通过窗口列表切换窗口

, // 重命名当前窗口；这样便于识别

. // 修改当前窗口编号；相当于窗口重新排序

f // 在所有窗口中查找指定文本

面板操作

” // 将当前面板平分为上下两块

% // 将当前面板平分为左右两块

x // 关闭当前面板

! // 将当前面板置于新窗口；即新建一个窗口，其中仅包含当前面板

Ctrl + 方向键 // 以 1 个单元格为单位移动边缘以调整当前面板大小

Alt + 方向键 // 以 5 个单元格为单位移动边缘以调整当前面板大小

Space // 在预置的面板布局中循环切换；依次包括 even-horizontal、even-vertical、main-horizontal、main-vertical、tiled

q // 显示面板编号

o // 在当前窗口中选择下一面板

方向键 // 移动光标以选择面板

{ // 向前置换当前面板

} // 向后置换当前面板

Alt+o // 逆时针旋转当前窗口的面板

Ctrl+o // 顺时针旋转当前窗口的面板

4、配置文件

tmux 配置文件在~/.tmux.conf 和 / etc/tmux.conf 中，配置文件中可以修改默认绑定的快捷键

配置文件示例：

[cpp] view plaincopy 在 CODE 上查看代码片派生到我的代码片

// 此类配置可以在命令行模式中输入 show-options -g 查询

set-option -g base-index 1 // 窗口的初始序号；默认为 0，这里设置为 1

set-option -g display-time 5000 // 提示信息的持续时间；设置足够的时间以避免看不清提示，单位为毫秒

set-option -g repeat-time 1000 // 控制台激活后的持续时间；设置合适的时间以避免每次操作都要先激活控制台，单位为毫秒

set-option -g status-keys vi // 操作状态栏时的默认键盘布局；可以设置为 vi 或 emacs

set-option -g status-right “#(date +%H:%M’ ‘)” // 状态栏右方的内容；这里的设置将得到类似 23:59 的显示

set-option -g status-right-length 10 // 状态栏右方的内容长度；建议把更多的空间留给状态栏左方（用于列出当前窗口）

set-option -g status-utf8 on // 开启状态栏的 UTF-8 支持

// 此类设置可以在命令行模式中输入 show-window-options -g 查询

set-window-option -g mode-keys vi // 复制模式中的默认键盘布局；可以设置为 vi 或 emacs

set-window-option -g utf8 on // 开启窗口的 UTF-8 支持

// 将激活控制台的快捷键由 Ctrl+b 修改为 Ctrl+a，Ctrl+a 是 Screen 的快捷键

set-option -g prefix C-a

unbind-key C-b

bind-key C-a send-prefix

// 添加自定义快捷键

bind-key z kill-session // 按 z 结束当前会话；相当于进入命令行模式后输入 kill-session

bind-key h select-layout even-horizontal // 按 h 将当前面板布局切换为 even-horizontal；相当于进入命令行模式后输入 select-layout even-horizontal

bind-key v select-layout even-vertical // 按 v 将当前面板布局切换为 even-vertical；相当于进入命令行模式后输入 select-layout even-vertical