> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.361way.com](http://www.361way.com/golang-install/4537.html)

> 做为运维人员，使用 python 语言足矣，理论上是无需了解 golang 的。

做为运维人员，使用 python 语言足矣，理论上是无需了解 golang 的。不过最近发现一个新潮的监控系统都是基于 golang 进行开发的，如小米公司的 [Open-Falcon](http://open-falcon.com/)（[github 项目页](https://github.com/xiaomi/open-falcon)） 、stack exchange 公司的 [Bosun](http://bosun.org/) （[github 项目页](https://github.com/bosun-monitor)）。

### 一、Go 的三种安装方式  

Go 有多种安装方式，你可以选择自己喜欢的。这里我们介绍三种最常见的安装方式：

*   Go 源码安装：这是一种标准的软件安装方式。对于经常使用 Unix 类系统的用户，尤其对于开发者来说，从源码安装可以自己定制。
*   Go 标准包安装：Go 提供了方便的安装包，支持 Windows、Linux、Mac 等系统。这种方式适合快速安装，可根据自己的系统位数下载好相应的安装包，一路 next 就可以轻松安装了。**推荐这种方式**
*   第三方工具安装：目前有很多方便的第三方软件包工具，例如 Ubuntu 的 apt-get、Mac 的 homebrew 等。这种安装方式适合那些熟悉相应系统的用户。

最后，如果你想在同一个系统中安装多个版本的 Go，你可以参考第三方工具 [GVM](https://github.com/moovweb/gvm)，这是目前在这方面做得最好的工具，除非你知道怎么处理。

### 二、Go 源码安装  

在 Go 的源代码中，有些部分是用 Plan 9 C 和 AT&T 汇编写的，因此假如你要想从源码安装，就必须安装 C 的编译工具。

在 Mac 系统中，只要你安装了 Xcode，就已经包含了相应的编译工具。

在类 Unix 系统中，需要安装 gcc 等工具。例如 Ubuntu 系统可通过在终端中执行 sudo apt-get install gcc libc6-dev 来安装编译工具。

在 Windows 系统中，你需要安装 MinGW，然后通过 MinGW 安装 gcc，并设置相应的环境变量。

由于本人平时主要涉及到 linux 平台下的使用，所以这里只以 linux 平台为例，不涉及 mac 和 win 下的安装。

#### 1、Go 源代码下载  

从 [官方页面](https://golang.org/dl/) 、 [国内镜像](http://www.golangtc.com/download) 或 [github 上](https://github.com/golang/go)下载 Go 的源码包到你的计算机上，然后将解压后的目录 go 通 1 过命令移动到 $GOROOT 所指向的位置。

```
#本人设置$GOROOT为/usr/local/go
export $GOROOT=/usr/local/go
wget https://storage.googleapis.com/golang/go<VERSION>.src.tar.gz
tar zxv go<VERSION>.src.tar.gz
mv go $GOROOT

```

#### 2、安装 gcc 等编译工具  

```
sudo apt-get install bison ed gawk gcc libc6-dev make
或
yum gcc mak gawk 

```

根据不同的发行版，自行选择相应的编译环境包安装的方式。

#### 3、编译  

```
cd $GOROOT/src
./all.bash

```

#### 4、环境变量配置  

```
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin
或者使用如下：
export PATH=$PATH:/usr/local/go/bin

```

根据需要可以在. bashrc、.profile、/etc/profile 等任一文件中配置相应的环境变量。

#### 5、测试  

直接运行 go 命令，出现如下部分，表示环境已经安装好。

```
[root@361way ~]# go
Go is a tool for managing Go source code.
Usage:
        go command [arguments]
The commands are:
    build       compile packages and dependencies
    clean       remove object files
    env         print Go environment information
    fix         run go tool fix on packages
    fmt         run gofmt on package sources
    generate    generate Go files by processing source
    get         download and install packages and dependencies
    install     compile and install packages and dependencies
    list        list packages
    run         compile and run Go program
    test        test packages
    tool        run specified go tool
    version     print Go version
    vet         run go tool vet on packages
Use "go help [command]" for more information about a command.
Additional help topics:
    c           calling between Go and C
    filetype    file types
    gopath      GOPATH environment variable
    importpath  import path syntax
    packages    description of package lists
    testflag    description of testing flags
    testfunc    description of testing functions
Use "go help [topic]" for more information about that topic.

```

### 三、go 标准包安装  

通过官方下的压缩包下载来看，标准包里包含源代码，不过区别的是其解包后已经有相应的 bin 目录存在，里面有已经编译好的 golang 二进制包。可以直接使用上面第 4 部的操作配置环境变量后生效。

### 四、第三方工具安装  

#### 1、centos 平台下  

需使用[第三方源 epel](http://www.361way.com/centos-install-epel/4538.html) ，如下：

```
yum -y install golang

```

#### 2、ubuntu 平台下  

```
sudo apt-get install python-software-properties
sudo add-apt-repository ppa:gophers/go
sudo apt-get update
sudo apt-get install golang-stable git-core mercurial

```

#### 3、mac 平台下  

```
brew update && brew upgrade
brew install go
brew install git
brew install mercurial

```

### 五、GVM 工具  

GVM 是一个 golang 虚拟环境配置工具，其允许一台机器上安装多个 golang 版本，gvm 是第三方开发的 Go 多版本管理工具，类似 ruby 里面的 rvm 工具。使用起来相当的方便，安装 gvm 使用如下命令：

```
bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)

```

安装完成后我们就可以安装 go 了：

```
gvm install go1.4.2
gvm use go1.4.2

```

也可以使用下面的命令，省去每次调用 gvm use 的麻烦：  
gvm use go1.4.2 --default 执行完上面的命令之后 GOPATH、GOROOT 等环境变量会自动设置好，这样就可以直接使用了。

### 六、Go 环境变量  

Go 开发环境依赖于一些操作系统环境变量，你最好在安装 Go 之间就已经设置好他们。如果你使用的是 Windows 的话，你完全不用进行手动设置，Go 将被默认安装在目录 c:/go 下。这里列举几个最为重要的环境变量：

*   **$GOROOT** 表示 Go 在你的电脑上的安装位置，它的值一般都是 $HOME/go，当然，你也可以安装在别的地方。
*   **$GOARCH** 表示目标机器的处理器架构，它的值可以是 386、amd64 或 arm。
*   **$GOOS** 表示目标机器的操作系统，它的值可以是 darwin、freebsd、linux 或 windows。
*   **$GOBIN** 表示编译器和链接器的安装位置，默认是 $GOROOT/bin，如果你使用的是 Go 1.0.3 及以后的版本，一般情况下你可以将它的值设置为空，Go 将会使用前面提到的默认值。

目标机器是指你打算运行你的 Go 应用程序的机器。

Go 编译器支持交叉编译，也就是说你可以在一台机器上构建运行在具有不同操作系统和处理器架构上运行的应用程序，也就是说编写源代码的机器可以和目标机器有完全不同的特性（操作系统与处理器架构）。

为了区分本地机器和目标机器，你可以使用 $GOHOSTOS 和 $GOHOSTARCH 设置目标机器的参数，这两个变量只有在进行交叉编译的时候才会用到，如果你不进行显示设置，他们的值会和本地机器（$GOOS 和 $GOARCH）一样。

*   **$GOPATH** 默认采用和 $GOROOT 一样的值，但从 Go 1.1 版本开始，你必须修改为其它路径。它可以包含多个包含 Go 语言源码文件、包文件和可执行文件的路径，而这些路径下又必须分别包含三个规定的目录：src、pkg 和 bin，这三个目录分别用于存放源码文件、包文件和可执行文件。
*   **$GOARM** 专门针对基于 arm 架构的处理器，它的值可以是 5 或 6，默认为 6。
*   **$GOMAXPROCS** 用于设置应用程序可使用的处理器个数与核数，详见第 14.1.3 节。

在接下来的章节中，我们将会讨论如何在 Linux、Mac OS X 和 Windows 上安装 Go 语言。在 FreeBSD 上的安装和  
Linux 非常类似。开发团队正在尝试将 Go 语言移植到其它例如 OpenBSD、DragonFlyBSD、NetBSD、Plan  
9、Haiku 和 Solaris 操作系统上，你可以在这个页面找到最近的动态：[Go Porting Efforts](http://go-lang.cat-v.org/os-ports)。

主机环境使用示例：

```
export GOARCH=amd64
export GOOS=linux 

```

### 七、hello word  

创建一个 hello.go 文件，内容如下：

```
package main
import "fmt"
func main() {
    fmt.Printf("hello, world\n")
}

```

运行结果如下：

```
# go run hello.go
hello, world

```

参考页面：

[golang 官方安装页](http://golang.org/doc/install) （需翻墙）

[go 语言入门指南](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/directory.md)

**本站的发展离不开您的资助，金额随意, 欢迎来赏！**

**You can donate through PayPal.  
My paypal id: itybku@139.com  
Paypal page: https://www.paypal.me/361way**