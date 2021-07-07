> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.yisu.com](https://www.yisu.com/zixun/313071.html)

> 领先的全球云计算和云安全提供商！

发布时间：2020-10-16 07:50:06 来源：脚本之家 阅读：122 作者：weixin_39092993 栏目：[开发技术](https://www.yisu.com/zixun/kf/)

**安装 golang**

使用 homebrew 安装 golang。homebrew 是 MacOS 平台下的软件包管理工具，拥有安装、卸载、更新、查看、搜索等功能。开发者不需要关心依赖和文件路径。如果系统没有安装 homebrew，终端内执行以下命令安装 homebrew。

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"

```

安装完 homebrew 后执行以下命令安装 golang，如果下载过慢可能是由于网络原因，可以通过更改 homebrew 的镜像地址或者开启科学上网解决。

> brew install golang

**配置环境变量**

安装成功后，执行 _go env_ 查看 golang 的环境变量。顺便可以测试是否安装成功。在本地的 shell。配置相应环境变量。  
zsh 执行 **vim ～/.zshrc**，bash **vim ～/.bashrc**。从交互及易用的角度 zsh 更好一些，zsh 完全兼容 bash，并且提供自动补全的功能。如果 shell 默认不是 zsh。可以通过如下命令切换默认 zsh，并安装 oh-my-zsh。oh-my-zsh 是基于 zsh 命令行的一个扩展工具集，提供了丰富的扩展功能。

```
sudo chsh -s zsh
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

```

然后在～/.zshrc 声明环境变量，下面的示例中设置 GOPATH 为根目录下的 golang 目录，可以指定自己的目录为 GOPATH。

```
export GOPATH=$HOME/golang
export GOROOT=/usr/local/opt/go/libexec

export GOPROXY=https:

```

安装完成执行 _source ~/.zshrc_，或者重新打开 shell，使环境变量生效。  
`GOROOT` 就是 golang 的安装路径。  
`GOPATH` 作为 Go 语言的环境变量，相当于个人的工作区，每个工作区中都会有以代码包为基本组织形式的源码文件。**goalng 的项目必须放在 GOPATH 路径下，才能正常执行**。这个目录用来存放 Go 源码，Go 的可运行文件，以及相应的编译之后的包文件。这个目录下有三个子目录：src、bin、pkg。  
按照约定这三个目录的作用是:

*   src 存放项目的源码
*   pkg 存放编译后生成的文件
*   bin 存放编译后生成的可执行文件

`GOPROXY` 如果设置完成该变量，下载源代码时将会通过该环境变量设置的代理地址，不会直接从代码库下载。而且某些代码库所在[服务器](https://www.yisu.com/ "服务器")需要科学上网才可以访问。设置`GOPROXY`可以避免由于网络环境的原因下载不了某些代码库。  
`GOPRIVATE` 正常情况下是从公共镜像 `goproxy.io` 上下载依赖包，并且会对下载的软件包和代码库进行安全校验，所以设置环境变量 `GOPRIVATE`，可以对指定仓库地址，跳过 proxy server 和校验检查。  
通过设置`GONOPROXY`和 `GONOSUMDB`等环境变量。 可以更灵活的控制哪些依赖软件包经过 proxy server 和 sumdb 校验，这两个环境变量的被设置后将覆盖 GOPRIVATE 环境变量。  
`GONOSUMDB` 通过这个环境变量设置不做校验的代码仓库地址。设置完成后从该地址上下载的依赖都不需要做校验。

**Goland 设置**

VSCode 需要额外配置插件，这里不讨论 VSCode，如果团队开发使用 Goland 比较方便管理。毕竟 Jenbrains 家族的产品，从开发效率上来说，是极高的。但是还是希望懂得底层远离。不要离开 IDE 就不会写代码，无法启动项目。  
IDE 的额外配置：  
1. 自动保存格式化。避免因代码未格式化提交到 git 历史里。`Preferences ->Plugins`搜索 _save actions_，然后设置自动保存格式化。

![](https://cache.yisu.com/upload/information/20200903/112/823.png)

2. 设置 goimports 格式化代码。`Preferences -> Tools -> File Watchers`。添加 goimports 然后设置本地包单独分组参数。

```
-local amap-aos -w $FilePath$

```

![](https://cache.yisu.com/upload/information/20200903/112/824.jpg)

3. 项目配置  
在 Goland 的右上方找到 “Add Configuration” 并单击。在弹出的窗口中点击 “+”，并在下拉菜单中选择“Go Build”。点击“Go Build” 之后，在窗口中填写对应的信息。

*   名称：为本条配置信息的名称，可以自定义，也可以使用系统默认的值；
*   Run kind：这里需要设置为 “Directory”。
*   Directory：用来设置 main 包所在的目录，不能为空。一般是项目的根目录。
*   Output directory：用来设置编译后生成的可执行文件的存放目录，可以为空，为空时默认不生成可执行文件。
*   Working directory：用来设置程序的运行目录，可以与 “Directory” 的设置相同，但是不能为空。

设置完成后就可以使用 Goland 在本地启动项目，注意**项目代码要放在 GOPATH 下**。

**总结**

到此这篇关于 MacOS 下本地 golang 环境搭建详细教程的文章就介绍到这了, 更多相关 MacOS golang 环境搭建内容请搜索亿速云以前的文章或继续浏览下面的相关文章希望大家以后多多支持亿速云！

免责声明：本站发布的内容（图片、视频和文字）以原创、转载和分享为主，文章观点不代表本网站立场，如果涉及侵权请联系站长邮箱：is@yisu.com 进行举报，并提供相关证据，一经查实，将立刻删除涉嫌侵权内容。

[![](https://cache.yisu.com/www/images/information/zixun-detail-guanggao.jpg)](https://www.yisu.com/coupon/?f=zixun&ystimes=zc2020)

[![](https://cache.yisu.com/www/images/information/list-head-img.png)](https://www.yisu.com/coupon/?f=zixun&ystimes=zc2020=right)