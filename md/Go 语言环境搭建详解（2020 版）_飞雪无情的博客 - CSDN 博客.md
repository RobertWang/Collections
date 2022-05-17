> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/flysnow_org/article/details/109506751?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.pc_relevant_default&utm_relevant_index=2)

最近写了很多 [Go 语言](https://so.csdn.net/so/search?q=Go%E8%AF%AD%E8%A8%80&spm=1001.2101.3001.7020)的原创文章，其中 Go 语言实战系列 30 篇，近 15W 字，还有最近更新的 Go 经典库系列，不过通过大家的咨询来看，还是想要一些入门的知识，这一篇文章写于 2017 年初，这 3 年多 Go 更新了很多版本，所以需要更新下这篇文章。

> 提示：本文基于 Go 语言最新版 go1.15.3 写成。

有读者来信（微信公众号消息）说能不能写一篇关于 Go 语言环境的配置搭建，这样对于想学 Go 语言的可以快速的配置起来一个环境。这个的确是我忽略了，按照我写书的逻辑，也是先有环境搭建，才能有语言功能介绍，这个直接把 Go 语言的开发环境搭建等配置跳过去实在不应该，所以这篇特意针对 Go 语言的开发环境搭建、配置、编辑器选型、不同平台程序生成等做了详细的介绍。

下载
--

要搭建 Go 语言开发环境，我们第一步要下载 go 的开发工具包，目前最新稳定版本是 go1.15.3。Go 为我们所熟知的所有平台架构提供了开发工具包，比如我们熟知的 Linux、Mac 和 Windows，其他的还有 FreeBSD 等。

我们可以根据自己的机器操作系统选择相应的开发工具包，比如你的是 Windows 64 位的，就选择 windows-amd64 的工具包；是 Linux 32 位的就选择 linux-386 的工具包。可以自己查看下自己的操作系统，然后选择，Mac 的现在都是 64 位的，直接选择就可以了。

开发工具包又分为安装版和压缩版。安装版是 Mac 和 Windows 特有的，他们的名字类似于：

*   go1.15.3.darwin-amd64.pkg
    
*   go1.15.3.windows-386.msi
    
*   go1.15.3.windows-amd64.msi
    

安装版，顾名思义，双击打开会出现安装向导，让你选择安装的路径，帮你设置好环境比安康等信息，比较省事方便一些。

压缩版的就是一个压缩文件，可以解压得到里面的内容，他们的名字类似于：

*   go1.15.3.darwin-amd64.tar.gz
    
*   go1.15.3.linux-386.tar.gz
    
*   go1.15.3.linux-amd64.tar.gz
    
*   go1.15.3.windows-386.zip
    
*   go1.15.3.windows-amd64.zip
    

压缩版我们下载后需要解压，然后自己移动到要存放的路径下，并且配置环境变量等信息，相比安装版来说，比较复杂一些，手动配置的比较多。

根据自己的操作系统选择后，就可以下载开发工具包了，Go 语言的官方下载地址是 https://golang.org/dl/ 可以打开选择版本下载，如果该页面打不开，或者打开了下载不了，可以通过 Golang 的国内网站 https://golang.google.cn/dl/ 下载。

Linux 下安装
---------

我们以 Ubuntu 64 位为例进行演示，CentOS 等其他 Linux 发行版大同小异。

下载 go1.15.3.linux-amd64.tar.gz 后，进行解压，你可以采用自带的解压软件解压，如果没有可以在终端行使用 tar 命令行工具解压，我们这里选择的安装目录是`/usr/local/go`, 可以使用如下命令：

```
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin
```

如果提示没有权限，在最前面加上`sudo`以 root 用户的身份运行。运行后，在`／usr/local/`下就可以看到 go 目录了。如果是自己用软件解压的，可以拷贝到 / usr/local/go 下，但是要保证你的 go 文件夹下是 bin、src、doc 等目录，不要 go 文件夹下又是一个 go 文件夹，这样就双重嵌套了。

然后就要配置环境变量了，Linux 下有两个文件可以配置，其中`/etc/profile`是针对所有用户都有效的；`$HOME/.profile`是针对当前用户有效的，可以根据自己的情况选择。

针对所有用户的需要重启电脑才可以生效；针对当前用户的，在终端里使用 source 命令加载这个`$HOME/.profile`即可生效。

```
➜  ~ go version
go version go1.15.3 linux/amd64
```

使用文本编辑器比如 VIM 编辑他们中的任意一个文件，在文件的末尾添加如下配置保存即可：

```
➜  ~ go version
go version go1.15.3 darwin/amd64
```

其中 GOROOT 环境变量表示我们 GO 的安装目录，这样其他软件比如我们使用的 Go 开发 IDE 就可以自动的找到我们的 Go 安装目录，达到自动配置 Go SDK 的目的。

第二句配置是把`/usr/local/go/bin`这个目录加入到环境变量 PATH 里，这样我可以在终端里直接输入 go 等常用命令使用了，而不用再加上`/usr/local/go/bin`这一串绝对路径，更简洁方便。

以上配置好之后，我们打开终端，输入如下命令，就可以看到 go 的版本等信息了。

```
├── bin
├── pkg
└── src
```

这就说明我们已经安装 go 成功了，如果提示 go 这个命令找不到，说明我们配置还不对，主要在 PATH 这个环境变量，仔细检查，直到可以正常输出为止。

Mac 下安装
-------

Mac 分为压缩版和安装版，他们都是 64 位的。压缩版和 Linux 的大同小异，因为 Mac 和 Linux 都是基于 Unix，终端这一块基本上是相同的。

压缩版解压后，就可以和 Linux 一样放到一个目录下，这里也以`/usr/local/go/`为例。在配置环境变量的时候，针对所有用户和 Linux 是一样的，都是`/etc/profile`这个文件；针对当前用户，Mac 下是`$HOME/.bash_profile`，其他配置都一样，包括编辑 sudo 权限和生效方式，最后在终端里测试：

```
➜  tour go mod init flysnow.org/tour
go: creating new go.mod: module flysnow.org/tour
```

Mac 安装版下载后双击可以看到安装界面，按照提示一步步选择操作即可。安装版默认安装目录是`/usr/local/go`，并且也会自动的把`/usr/local/go/bin`目录加入到 PATH 环境变量中，重新打开一个终端，就可以使用`go version`进行测试了，更快捷方便一些。

Windows 下安装
-----------

Windows 也有压缩版和安装版，又分为 32 和 64 位以供选择，不过目前大家都是 64 位，选择这个更好一些。

Window 的压缩版是一个 ZIP 压缩包，下载后使用 winrar 等软件就可以解压，解压后要选择一个存放目录，比如`c:\Go`下，这个`c:\Go`就是 Go 的安装目录了，他里面有 bin、src、doc 等目录。

然后就是环境变量的配置，Window 也和 Linux 一样分为针对所有用户的系统变量，和针对当前用户的用户变量设置，可以自行选择，比如系统变量，针对所有用户都有效。

以 Window 7 为例，右击我的电脑 -> 属性会打开系统控制面板，然后在左侧找到`高级系统设置`点击打开，会在弹出的界面最下方看到`环境变量`按钮，点击它，就可以看到环境变量配置界面了。上半部分是用户变量配置，下半部分是系统变量配置。

我们在系统变量里点击新建，变量名输入 GOROOT，变量值是我们刚刚安装的 go 路径`c:\Go`, 这样就配置好了 GO 目录的安装路径了。

然后修改 PATH 系统变量，在变量值里添加`%%GOROOT\bin`路径，和其他 PATH 变量以;(分号，Linux 下是冒号) 分割即可。这样我们就可以在 CMD 里直接输入 go 命令使用了。

打开我们的终端，输入`go version`测试下，好了的话就可以看到输出的信息了。

Window 的安装版相比来说就比较简单一些，双击就可以按照提示一步步安装，默认安装路径是`c:\Go`, 并且会配置好 PATH 环境变量，可以直接打开 CMD 终端使用。

GOPATH 目录
---------

自动 Golang 采用 Module 的方式管理项目后，GOPATH 目录已经不是那么重要了，目前主要用来存放依赖的 Module 库，生成的可执行文件等。GOPATH 环境变量的配置参考上面的安装 Go，配置到`/etc/profile`或者 Windows 下的系统变量里。

这个目录我们可以根据自己的设置指定，比如我的 Mac 在`$HOME/code/go`下，Window 的可以放到 d:\code\go 下等。该目录下有 3 个子目录，他们分别是：

```
module flysnow.org/tour
 
go 1.15
```

*   bin 文件夹存放`go install`命名生成的可执行文件，可以把`$GOPATH/bin`路径加入到 PATH 环境变量里，就和我们上面配置的`$GOROOT/bin`一样，这样就可以直接在终端里使用我们 go 开发生成的程序了。
    
*   pkg 文件夹是存在 go 编译生成的文件。
    
*   src 存放的是非 Go Module 项目源代码。
    

go 项目工程结构
---------

配置好工作环境后，就可以编码开发了，在这之前，我们看下 go 的通用项目结构, 这里的结构主要是源代码相应的资源文件存放目录结构。

基于 Go Module，你可以在任意位置创建一个 Go 项目，而不再像以前一样局限在`$GOPATH/src`目录下。假设我要创建一个 tour 项目，它位于`~/Desktop/tour`目录下，那我现在打开终端，CD 到`~/Desktop/tour`目录下，输入如下命令即可创建一个 Go Module 工程。

```
import (
    "github.com/gohugoio/hugo/commands"
)
```

当前生成的 Go Module 工程只有一个`go.mod`文件，它的内容如下所示：

```
.
├── go.mod
├── lib1
├── lib2
└── main.go
```

其中`module flysnow.org/tour`代表该项目的 path, 也就是最顶层的 package，`go 1.15`表示该项目需要 go 1.15 版本及其以上才能编译运行。

`go.mod`文件是 Go 语言工具链用于管理 Go 语言项目的一个配置文件，我们不用手动修改它，Go 语言的工具链会帮我们自动更新，比如当我们的项目添加一个新的第三方库的时候。

使用第三方库，也就是使用第三方库里的包，那么我们如何引用一个包呢，使用的就是 go 语言的 import 关键字，比如：

```
package main
 
import (
    "fmt"
)
 
func main() {
    fmt.Println("Hello World")
}
```

以上引入的`github.com/gohugoio/hugo/commands`这个包是属于 `github.com/gohugoio/hugo/`这个 Go Module 的。

所以相应的，我们也可以在我们自己的 Go Module 工程里创建一些包 (其实就是子目录), 比如我创建了`lib1`目录，那么它的对应的包就是`flysnow.org/tour/lib1`, 其他包只有通过这个包名才能使用`flysnow.org/tour/lib1`包中的函数方法等。

```
➜  ~ tour
Hell World
```

所以最后你的项目目录类似上面的结构，每个子目录都是一个包，子目录里可以放 go 文件。

Hello World
-----------

好了，有了`tour`项目，就可以演示下 Go 语言版本的 Hello World 了，在`tour`根目录下的`main.go`（如没有这个文件，就新建一个）文件中，添加如下 Go 代码。

```
➜  ~ go env
GOARCH="amd64"
GOEXE=""
GOHOSTARCH="amd64"
GOHOSTOS="darwin"
GOOS="darwin"
GOROOT="/usr/local/go"
GOTOOLDIR="/usr/local/go/pkg/tool/darwin_amd64"
```

Go 版本的 Hello World 非常简单。在`~/Desktop/tour`目录下运行`go run main.go`命令就可以看到打印的输出`Hello World`，下面解释下这段代码。

1.package 是一个关键字，定义一个包，和 Java 里的 package 一样，也是模块化的关键。

1.  main 包是一个特殊的包名，它表示当前是一个可执行程序，而不是一个库。
    
2.  import 也是一个关键字，表示要引入的包，和 Java 的 import 关键字一样，引入后才可以使用它。
    
3.  fmt 是一个包名，这里表示要引入 fmt 这个包，这样我们就可以使用它的函数了。
    
4.  main 函数是主函数，表示程序执行的入口，Java 也有同名函数，但是多了一个 String[] 类型的参数。
    
5.  Println 是 fmt 包里的函数，和 Java 里的 system.out.println 作用类似，这里输出一段文字。
    

整段代码非常简洁，关键字、函数、包等和 Java 非常相似，不过注意，go 是不需要以;(分号) 结尾的。

安装程序
----

安装的意思，就是生成可执行的程序，以供我们使用，为此 go 为我们提供了很方便的 install 命令，可以快速的把我们的程序安装到`$GOAPTH/bin`目录下。在`~/Desktop/tour`目录下运行如下代码即可安装。

```
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.io,direct
```

打开终端，运行上面的命令即可，install 后紧跟全路径的包名。然后我们在终端里运行`tour`就看到打印的 Hello World 了。

```
[url "git@git.flysnow.org:"]
    insteadOf = http://git.flysnow.org/
```

跨平台编译
-----

以前运行和安装，都是默认根据我们当前的机器生成的可执行文件，比如你的是 Linux 64 位，就会生成 Linux 64 位下的可执行文件，比如我的 Mac，可以使用`go env`查看编译环境, 以下截取重要的部分。

```
# 设置不走 proxy 的私有仓库，多个用逗号相隔（可选）
go env -w GOPRIVATE=git.flysnow.org
```

注意里面两个重要的环境变量`GOOS`和`GOARCH`, 其中`GOOS`指的是目标操作系统，它的可用值为：

1.  aix
    
2.  android
    
3.  darwin
    
4.  dragonfly
    
5.  freebsd
    
6.  illumos
    
7.  js
    
8.  linux
    
9.  netbsd
    
10.  openbsd
    
11.  plan9
    
12.  solaris
    
13.  windows
    

一共支持 13 种操作系统。`GOARCH`指的是目标处理器的架构，目前支持的有：

1.  arm
    
2.  arm64
    
3.  386
    
4.  amd64
    
5.  ppc64
    
6.  ppc64le
    
7.  mips
    
8.  mipsle
    
9.  mips64
    
10.  mips64le
    
11.  s390x
    
12.  wasm
    

一共支持 12 种处理器的架构，`GOOS`和`GOARCH`组合起来，支持生成的可执行程序种类很多，具体组合参考 https://golang.org/doc/install/source#environment。如果我们要生成不同平台架构的可执行程序，只要改变这两个环境变量就可以了，比如要生成 linux 64 位的程序，命令如下：

```
GOOS=linux GOARCH=amd64 go build flysnow.org/tour
```

前面两个赋值，是更改环境变量，这样的好处是只针对本次运行有效，不会更改我们默认的配置。

获取远程包
-----

因为众所周知的原因，在获取远程包之前，我们需要先配置的代理, 这里推荐`goproxy.io`代理，设置命令如下：

```
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.io,direct
```

设置好代理后，就可以使用 go 提供的一个获取远程包的工具`go get`来获取远程包了, 它需要一个完整的包名作为参数，只要这个完整的包名是可访问的，就可以被获取到，比如我们获取一个 CLI 的开源库：

```
go get -v github.com/spf13/cobra
```

就可以下载这个库到我们`$GOPATH/pkg/mod`目录下了，这样我们就可以像导入其他包一样 import 了。

特别提醒，go get 的本质是使用源代码控制工具下载这些库的源代码，比如 git，hg 等，所以在使用之前必须确保安装了这些源代码版本控制工具。

如果我们使用的远程包有更新，我们可以使用如下命令进行更新, 多了一个`-u`标识。

```
go get -u -v github.com/spf13/cobra
```

获取 gitlab 私有库包
--------------

如果是私有的 git 库怎么获取呢？比如在公司使用 gitlab 搭建的 git 仓库，设置的都是 private 权限的。这种情况下我们可以配置下 git，就可以了，在此之前你公司使用的 gitlab 必须要在 7.8 之上。然后要把我们 http 协议获取的方式换成 ssh，假设你要获取`http://git.flysnow.org`，对应的 ssh 地址为`git@git.flysnow.org`，那么要在终端执行如下命令。

```
git config --global url."git@git.flysnow.org:".insteadOf "http://git.flysnow.org/"
```

这段配置的意思就是，当我们使用`http://git.flysnow.org/`获取 git 库代码的时候，实际上使用的是`git@git.flysnow.org`这个 url 地址获取的，也就是 http 到 ssh 协议的转换，是自动的，他其实就是在我们的`~/.gitconfig`配置文件中，增加了如下配置:

```
[url "git@git.flysnow.org:"]
    insteadOf = http://git.flysnow.org/
```

然后需要把`git.flysnow.org`加入`GOPRIVATE`环境变量中，因为它是你的私有仓库，不需要走`GOPROXY`代理。

```
# 设置不走 proxy 的私有仓库，多个用逗号相隔（可选）
go env -w GOPRIVATE=git.flysnow.org
```

现在我们就可以使用`go get`直接获取了，比如：

```
go get -v -insecure git.flysnow.org/hello
```

仔细看，多了一个`-insecure`标识，因为我们使用的是 http 协议， 是不安全的。当然如果你自己搭建的 gitlab 支持 https 协议，就不用加`-insecure`了，同时把上面的`url insteadOf`换成`https`的就可以了。

Go 还有很多命令行工具可以使用，更多的请参考 [Go 语言实战笔记（二）| Go 开发工具](http://mp.weixin.qq.com/s?__biz=MzI3MjU4Njk3Ng%3D%3D&chksm=eb310067dc4689712d42a501a75dcfbc7442ab221f97e7f6b6027bf62cc93ff9b98ac6c3da2a&idx=3&mid=2247483880&scene=21&sn=4a20a203c19793e001114a6ef43779b5#wechat_redirect)

Go 编辑器推荐
--------

Go 采用的是 UTF-8 的文本文件存放源代码，所以原则上你可以使用任何一款文本编辑器，这里推荐几款比较流行的。

对于新手来说，我推荐功能强大的 IDE，功能强大，使用方便，比如 jetbrains idea+golang 插件，上手容易，而且它家的 IDE 都一样，会一个都会了，包括菜单、快捷键等。

值得高兴的是 jetbrains 针对 Go 这门语言推出了专用 IDE goland，也足以证明 go 的流行以及 jetbrains 的重视。goland 地址为 https://www.jetbrains.com/go/, 可以前往下载使用。

其次可以推荐微软的 VS Code，这款编辑器插件强大，快捷键方便，对 Go 支持的很好，也拥有大量的粉丝。

编辑器只是为了提高开发效率，大家哪个顺手用哪个，不存在谁更 NB。

一些文章推荐
------

一个就是我的 GO 语言实战笔记系列，这个可以入门和深入，还有最近写的 Go 的第三方库介绍和分析，可以让我们快速上手以及了解原理实践。

*   [Go 语言实战笔记（一）| Go 包管理](http://mp.weixin.qq.com/s?__biz=MzI3MjU4Njk3Ng%3D%3D&chksm=eb3102f1dc468be7484ce8a09189a512fe9e09b3d21693a539613f1f1089c960fa0b96c1ac19&idx=3&mid=2247484286&scene=21&sn=73d1fc7225d9473e1ad62e1398898fbf#wechat_redirect)
    
*   [Go 语言实战笔记（二）| Go 开发工具](http://mp.weixin.qq.com/s?__biz=MzI3MjU4Njk3Ng%3D%3D&chksm=eb310067dc4689712d42a501a75dcfbc7442ab221f97e7f6b6027bf62cc93ff9b98ac6c3da2a&idx=3&mid=2247483880&scene=21&sn=4a20a203c19793e001114a6ef43779b5#wechat_redirect)
    
*   [Go 语言实战笔记（四）| Go 数组](http://mp.weixin.qq.com/s?__biz=MzI3MjU4Njk3Ng%3D%3D&chksm=eb3100d6dc4689c05d12569b2a97bbf78908eef581ac27a9b38fe80e626cee46a7579eb9690c&idx=1&mid=2247483737&scene=21&sn=589aa0bf65ce021d0c1ce78b84711a7f#wechat_redirect)
    
*   [Go 语言实战笔记（六）| Go Map](http://mp.weixin.qq.com/s?__biz=MzI3MjU4Njk3Ng%3D%3D&chksm=eb3100eddc4689fb8eb91fed50c58cac80c75f0217a13ecb3a07186e67dafa88e519ffd34801&idx=1&mid=2247483746&scene=21&sn=3be6dbb1c70d72ab363b535c9a878d12#wechat_redirect)
    
*   [Go 语言实战笔记（九）| Go 接口](http://mp.weixin.qq.com/s?__biz=MzI3MjU4Njk3Ng%3D%3D&chksm=eb3100e1dc4689f7e0db4c1eed029138240f3bf3d87f5110aaeaec2f1f055e7d1ef31b0dff2a&idx=1&mid=2247483758&scene=21&sn=4d4ac325cd8ec67e05357481c76d187b#wechat_redirect)
    
*   [Go 语言实战笔记（十二）| Go goroutine](http://mp.weixin.qq.com/s?__biz=MzI3MjU4Njk3Ng%3D%3D&chksm=eb3100fadc4689ecae860ff7ba529befb5b3780d5dcf234be8094c61d38aa5d9df3c14b8ba96&idx=1&mid=2247483765&scene=21&sn=8221f40c4d8a51e8f16a7329aaa37162#wechat_redirect)
    
*   [Go 语言实战笔记（二十六）| Go unsafe 包之内存布局](http://mp.weixin.qq.com/s?__biz=MzI3MjU4Njk3Ng%3D%3D&chksm=eb31020ddc468b1b96d64151a8b58abf5b1e65a6913d92ec7f4f03851049daa4370e35d40685&idx=7&mid=2247484290&scene=21&sn=fedb97a37753ea53df6dc7ed2c7336b5#wechat_redirect)
    
*   [Go 语言经典库使用分析（五）| Negroni 中间件（一）](http://mp.weixin.qq.com/s?__biz=MzI3MjU4Njk3Ng%3D%3D&chksm=eb31006cdc46897ab4b5dafd008f1739c34679fcde06ca57e55c3a3ec04e036f1d1aef907585&idx=3&mid=2247483875&scene=21&sn=1417ea2ad94cb6ad71b818fee005792b#wechat_redirect)
    
*   [Golang Gin 实战（十三）| 中间件详解看这一篇就够了](http://mp.weixin.qq.com/s?__biz=MzI3MjU4Njk3Ng%3D%3D&chksm=eb3105fadc468cec2f7efda7d0c0fb5f13c84885f4253e0bcba3c0d8aed2274170211f84869a&idx=1&mid=2247484533&scene=21&sn=730455e12397c1ac303a501eef877e9c#wechat_redirect)  
    

此外，关于 Go 学习的书等，我这里有一篇知乎比较高的回答，供大家参考 https://www.zhihu.com/question/30461290/answer/210414739

到这里，整个 Go 开发环境就详细介绍完了，不光有环境安装搭建，还有目录结构、常用命令使用等都进行了介绍，这篇文章看完后，已经入门了 Go 了，剩下的再看看 Go 的语法和库，就可以很流畅的编写 Go 程序了。

> 本文为原创文章，转载注明出处，欢迎扫码关注公众号`flysnow_org`或者网站 http://www.flysnow.org/，第一时间看后续精彩文章。觉得好的话，顺手分享到朋友圈吧，感谢支持。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X2pwZy95R0tUakpkWWtrdmFyV05hOVNlb2liSzd5WUVMcjQ0Uk55OFJCQjhtVU1aWEpUS2JieFZiY05mS2ljWG1ta0R1V2huVjBtcGpMSzVhaWFSV3I3OTFpY0l2N2cvNjQw?x-oss-process=image/format,png)

扫码关注