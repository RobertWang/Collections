> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI4MDE1NjUxNQ==&mid=2247498012&idx=1&sn=c1940258a51f9433f7de89520066ebb3&chksm=ebbe7910dcc9f0067858dfa7255c7e625ffc76fc281c7a63890572e1c9c4d9b371799a41ed0e&mpshare=1&scene=1&srcid=1110H9F7e86anpTR6ltOIOWV&sharer_sharetime=1668029725390&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

点击上方 "[**Linux 中文社区**](http://mp.weixin.qq.com/s?__biz=MzI4MDE1NjUxNQ==&mid=2247483657&idx=1&sn=3e0e82cdeef29d6eadcb7e68b7a7bf66&chksm=ebbd8105dcca0813d729036e77ee6cb032c1393bdd7c011d5fa90922d9ecce7790736c89e079&scene=21#wechat_redirect) " 关注，星标或者置顶

21 点 00 分准时推送，第一时间送达

> 责编：中文妹 | 来源：入门小站

上一篇：[50 个应知必会的 Linux 常识和操作！](http://mp.weixin.qq.com/s?__biz=MzI4MDE1NjUxNQ==&mid=2247497970&idx=1&sn=4b824892587ce5bb73469dc2a21ee887&chksm=ebbe78fedcc9f1e8133746f4e63808a518cd6c5f35ab8213ae83d83ba07d4389269de2ae1c84&scene=21#wechat_redirect)

****大家好，我是中文妹。****

![](https://mmbiz.qpic.cn/mmbiz_png/Z4N0CzO8W9AF5UAbY6LKjnwbmoVhxQGWLyp9YIWaQKHFgkb5hRReiaQuXzzAjzBibQeAZX8c49TOibR17bOx8AR6A/640?wx_fmt=png)

1 zsh 介绍
========

1.1 Linux shell
---------------

Linux/Unix 提供了很多种 Shell，为毛要这么多 Shell？

难道用来炒着吃么？那我问你，你同类型的衣服怎么有那么多件？花色，质地还不一样。写程序比买衣服复杂多了，而且程序员往往负责把复杂的事情搞简单，简单的事情搞复杂。牛程序员看到不爽的 Shell，就会自己重新写一套，慢慢形成了一些标准，常用的 Shell 有这么几种，sh、bash、csh 等，想知道你的系统有几种 shell，可以通过以下命令查看：

显示如下：

![](https://mmbiz.qpic.cn/mmbiz_png/8v3VjEJ7GTqQxm3fyXLwoCdSCiaqrcMrbrD7iaG9Gr0Vfg8goYjU9QRUWqGJDtCdVbmaEdRFN30iaQxNoLsTL27Sw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

1.2 zsh 简介
----------

Zsh 是一个 Linux 下强大的 shell, 由于大多数 Linux 产品安装，以及默认使用`bash shell`, 但是丝毫不影响极客们对 zsh 的热衷, 几乎每一款 Linux 产品都包含有 zsh，通常可以用 apt-get、urpmi 或 yum 等包管理器进行安装

Zsh 具有以下主要功能

*   开箱即用、可编程的命令行补全功能可以帮助用户输入各种参数以及选项
    
*   在用户启动的所有 shell 中共享命令历史
    
*   通过扩展的文件通配符，可以不利用外部命令达到 find 命令一般展开文件名
    
*   改进的变量与数组处理
    
*   在缓冲区中编辑多行命令
    
*   多种兼容模式，例如使用 / bin/sh 运行时可以伪装成 Bourne shell
    
*   可以定制呈现形式的提示符；包括在屏幕右端显示信息，并在键入长命令时自动隐藏
    
*   可加载的模块，提供其他各种支持：完整的 TCP 与 Unix 域套接字控制，FTP 客户端与扩充过的数学函数
    
*   完全可定制化
    

1.3 zsh 与 oh-my-zsh 终极配置
------------------------

之前是因为看到这篇文章：终极 Shell——Zsh 才选择使用 zsh，被它的自动完成、补全功能吸引了。官网：www.zsh.org

选择 oh-my-zsh, oh-my-zsh 是基于 zsh 的功能做了一个扩展，方便的插件管理、主题自定义，以及漂亮的自动完成效果。

在 Github 上找关于 zsh 的项目时发现的，试用了一下觉得很方便，不用像上面文章里面提到的那么复杂，配置一些插件的名称即可使用相应的功能。

```
牛逼啊！接私活必备的 N 个开源项目！赶快收藏

```

官网：https://github.com/robbyrussell/oh-my-zsh

2 安装 zsh
========

2.1 安装 zsh
----------

> 对于一般的 Ubuntu 系统，配置好正确的源之后，就能直接键入以下命令安装：

```
sudo apt-get install zsh

```

2.2 配置 zsh
----------

> zsh 的配置是一门大学问，这里不赘述，直接给出一个配置文件，大家可以下载后放入 zsh 配置文档直接使用。（我的一个法国朋友手配的，相当顺手）

> 把. zshrc 拷贝到相应用户的 home 目录即可 (也可以把你的 bash 的配置文件 (~/.bash_prorile 或者~/.profile 等) 给拷贝到 zsh 的配置文件~/.zshrc 里，因为 zsh 兼容 bash)

2.3 取代 bash，设为默认 shell
----------------------

```
sudo usermod -s /bin/zsh username

```

或者

```
chsh -s /bin/zsh

```

```
chsh -s `which zsh`

```

如果要切换回去 bash：

```
chsh -s /bin/bash

```

当然你实在不愿意把 zsh 当成默认的 shell, 而又想使用它, 那么你可以每次进入是都使用`zsh`进入, 而输入`exit`退出

![](https://mmbiz.qpic.cn/mmbiz_png/8v3VjEJ7GTqQxm3fyXLwoCdSCiaqrcMrbjAiaonpRgGSbs4f7iaZBwriaQibMQYWib9OtnmVw270m1OC2HAP6icW4PIgw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

2.4 安装 oh-my-zsh
----------------

直接用 zsh 会很蛋疼，因为 zsh 功能很强大但是太复杂，所以需要 oh-my-zsh 来将它简单化。另外，搜索公众号 Linux 就该这样学后台回复 “Linux”，获取一份惊喜礼包。

**直接用 git 从 github 上面下载包**

```
git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh

```

**备份已有的 zshrc, 替换 zshrc**

```
cp ~/.zshrc ~/.zshrc.origcp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc

```

**直接使用脚本安装**

```
cd oh-my-zsh/tools./install.sh

```

> 你可以直接直接使用如下命令安装

**curl**

```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

```

**wget**

```
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"

```

其本质就是下载并执行了 github 上的 install.sh 脚本, 该脚本位于`oh-my-zsh/tools/install.sh`

**配置主题**

oh-my-zsh 集成了大量的主题, 位于 `oh-my-zsh/theme`

配置主题, 可以通过修改`~/.zshrc`中的环境变量`ZSH_THEME`来完成

```
ZSH_THEME="agnoster" # (this is one of the fancy ones)

```

如果你觉得主题太多你可以选择使用随机模式, 来由系统随机选择

```
ZSH_THEME="random" # (...please let it be pie... please be some pie..)

```

![](https://mmbiz.qpic.cn/mmbiz_png/8v3VjEJ7GTqQxm3fyXLwoCdSCiaqrcMrbDV09tfAloKtC2QBr9bl5xGeia95opg01ib9EG32yUdibAl25jXiaXia0ACA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

详细的主题信息, 可以参见 zsh 主题介绍

**配置插件**

修改`～/.zshrc`中`plugins`

```
plugins=(git bundler osx rake ruby)

```

详细的插件信息, 可以参见 zsh 插件 Plugins 介绍

**更新 oh-my-zsh**

默认情况下, 您将被提示检查每几周的升级. 如果你想我 ZSH 自动升级本身没有提示你, 修改 `~/.zshrc。另外，搜索公众号顶级算法后台回复 “算法”，获取一份惊喜礼包。

```
disable_update_prompt = true

```

禁用自动升级, 修改~/.zshrc

```
disable_auto_update = true

```

当然你也可以选择手动更新

如果你想在任何时间点升级（也许有人刚刚发布了一个新的插件，你不想等待一个星期？) 你只需要运行：

```
upgrade_oh_my_zsh

```

**卸载 oh-my-zsh**

如果你想卸载`oh-my-zsh`, 只需要执行`uninstall_oh_my_zsh zsh`， 从命令行运行. 这将删除本身和恢复你以前的 bash 或者 zsh 配置.