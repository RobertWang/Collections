> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/liuguanglin/p/9290345.html)

tmux 是什么
--------

tmux（terminal multiplexer）是 Linux 上的终端复用神器，可从一个屏幕上管理多个终端（准确说是伪终端）。使用该工具，用户可以连接或断开会话，而保持终端在后台运行。类似的工具还有 screen，个人对这二者的使用感受是，用过 tmux 就再也不想用 screen 了。

tmux 基本结构
---------

tmux 的结构包括**会话** (session)、**窗口** (window)、**窗格** (pane) 三部分，会话实质是伪终端的集合，每个窗格表示一个伪终端，多个窗格展现在一个屏幕上，这一屏幕就叫窗口。基本结构及状态信息如下图所示：

![](https://images2018.cnblogs.com/blog/771521/201807/771521-20180713112124434-879435876.png)

tmux 基本操作
---------

基本的操作无非就是对会话、窗口、窗格进行管理，包括创建、关闭、重命名、连接、分离、选择等等。

一般使用命令和快捷键进行操作，可在系统 shell 终端和 tmux 命令模式（类似 vim 的命令模式）下使用命令，或者在 tmux 终端使用快捷键。

tmux 默认的快捷键前缀是 **Ctrl+b**(下文用 **prefix** 指代)，按下前缀组合键后松开，再按下命令键进行快捷操作，比如使用 **prefix d** 分离会话（应该写作 **prefix d** 而不是 **prefix+d，**因为 **d** 键不需要与 **prefix** 同时按下）。

快捷键可以自定义，比如将前缀改为 **Ctrl+a**，但需要保留 shell 默认的 **Ctrl+a** 快捷键，按如下所示修改~/.tmux.conf 文件：

```
1 set-option -g prefix C-a
2 unbind-key C-b
3 bind-key C-a send-prefix
4 bind-key R source-file ~/.tmux.conf \; display-message "~/.tmux.conf reloaded."

```

现在已将原先的 **Ctrl+a** 用 **prefix Ctrl+a** 取代，即需要按两次 **Ctrl+a** 生效。

第 4 行的作用是使用 **prefix r** 重新加载配置文件，并输出提示，否则需要关闭会话后配置文件才能生效，也可手动加载配置文件，在 tmux 终端输入 "**prefix :"** 进入命令模式，用 **source-file** 命令加载配置文件。

**注意，将多个命令写在一起作为命令序列时，命令之间要用空格和分号分隔。** 

### 会话管理　

****常用命令****

**tmux new**　　创建默认名称的会话（在 tmux 命令模式使用 **new** 命令可实现同样的功能，其他命令同理，后文不再列出 tmux 终端命令）

**tmux new -s mysession**　　创建名为 mysession 的会话

**tmux ls**　　显示会话列表

**tmux a**　　连接上一个会话

**tmux a -t mysession**　　连接指定会话

**tmux rename -t s1 s2** 重命名会话 s1 为 s2

**tmux kill-session**　　关闭上次打开的会话

**tmux kill-session -t s1**　　关闭会话 s1

**tmux kill-session -a -t s1**　　关闭除 s1 外的所有会话

**tmux kill-server**　　关闭所有会话

**常用快捷键**

**prefix s**　　列出会话，可进行切换

**prefix $**　　重命名会话

**prefix d**　　分离当前会话

**prefix** **D**　　分离指定会话

### 窗口管理

**prefix c** 创建一个新窗口

**prefix ,** 重命名当前窗口

**prefix w**　　列出所有窗口，可进行切换

**prefix n**　　进入下一个窗口

**prefix p**　　进入上一个窗口

**prefix l**　　进入之前操作的窗口

**prefix 0~9**　　选择编号 0~9 对应的窗口

**prefix .** 修改当前窗口索引编号

**prefix '** 切换至指定编号（可大于 9）的窗口

**prefix f**　　根据显示的内容搜索窗格

**prefix &**　　关闭当前窗口

### **窗格管理**

**prefix %**　　水平方向创建窗格

**prefix "**　　垂直方向创建窗格

**prefix Up|Down|Left|Right**　　根据箭头方向切换窗格

**prefix q**　　显示窗格编号

**prefix o**　　顺时针切换窗格

**prefix }**　　与下一个窗格交换位置

**prefix {**　　与上一个窗格交换位置

**prefix x**　　关闭当前窗格

**prefix space(空格键)**　　重新排列当前窗口下的所有窗格

**prefix !**　　将当前窗格置于新窗口

**prefix Ctrl+o**　　逆时针旋转当前窗口的窗格

**prefix t**　　在当前窗格显示时间

**prefix z**　　放大当前窗格 (再次按下将还原)

**prefix i**　　显示当前窗格信息

### **其他命令**

**tmux list-key**　　列出所有绑定的键，等同于 **prefix ?**

**tmux list-command**　　列出所有命令

以上为 tmux 的常见操作，基本可以满足大部分的工作需求，至于更高端的操作待日后再整理。

--------------------------------------------------

**参考资料：**

1. man tmux;

2. 《[Tmux 速成教程：技巧和调整](http://blog.jobbole.com/87584/)》。