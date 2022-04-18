> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.lijiaocn.com](https://www.lijiaocn.com/%E7%BC%96%E7%A8%8B/2017/03/28/golang-ide.html)

> 李佶澳的博客: Go 编程。李佶澳、Go 编程

目录
--

*   [目录](#目录)
*   [安装 go](#安装go)
*   [运行环境](#运行环境)
*   [workspace](#workspace)
*   [版本管理工具](#版本管理工具)
    *   [godep](#godep)
    *   [gotag](#gotag)
    *   [安装 gocode](#安装gocode)
*   [配置 VIM](#配置vim)
    *   [安装 vim-go](#安装vim-go)
    *   [vim-go 安装](#vim-go-安装)
    *   [YouCompleteMe](#youcompleteme)
*   [Golang 零碎事项](#golang零碎事项)
    *   [换行](#换行)
    *   [使用在其他包中定义的结构体](#使用在其他包中定义的结构体)
    *   [取消证书验证](#取消证书验证)
*   [参考](#参考)

安装 go
-----

下载安装文件 go1.3.linux-amd64.tar.gz, 解压到 / opt

运行环境
----

在. bashrc 中添加:

```
#golang的安装目录
export GOROOT=/opt/go            

#golang工程目录
export GOPATH=/opt/go-workspace

export PATH=$PATH:$GOROOT/bin:$GOPATH/bin:$GOPATH/bin
```

执行`source ./bashrc`后，查看 go 环境变量:

```
[root@localhost dockerImages]## go env
GOARCH="amd64"
GOBIN=""
GOCHAR="6"
GOEXE=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GOOS="linux"
GOPATH="/opt/go-workspace"       ##GOPATH is workspace's location
GORACE=""
GOROOT="/opt/go-1.3.1"
GOTOOLDIR="/opt/go-1.3.1/pkg/tool/linux_amd64"
CC="gcc"
GOGCCFLAGS="-fPIC -m64 -pthread -fmessage-length=0"
CXX="g++"
CGO_ENABLED="1"
```

workspace
---------

设置 workspace:

```
mkdir -p $GOPATH/src
mkdir -p $GOPATH/bin
mkdir -p $GOPATH/pkg
```

版本管理工具
------

go get 支持 git 等版本管理工具，可以自动使用版本工具拉取代码。

[go get tools](https://github.com/golang/go/wiki/GoGetTools "go get tools") 中给出了 go 支持的版本管理工具，根据需要安装:

```
svn - Subversion, download at: http://subversion.apache.org/packages.html
hg - Mercurial, download at https://www.mercurial-scm.org/downloads
git - Git, download at http://git-scm.com/downloads
bzr - Bazaar, download at http://wiki.bazaar.canonical.com/Download
```

如果需要通过代理获取代码，设置环境变量`http_proxy=代理地址`，例如:

### godep

godep 是用于第三方依赖包的管理，可以将工程中依赖的 package 打包 (godep save) 到 Godeps 目录中，以及安装依赖包(go restore).

安装：

```
http_proxy=127.0.0.1:80
```

### gotag

[gotag](https://github.com/jstemmer/gotags "gotag") 用来与 vim 配合，生成 ctags 文件:

安装:

```
go get github.com/tools/godep
```

### 安装 gocode

[gocode](https://github.com/nsf/gocode "gocode") 用于 golang 代码的自动补全，与 vim 等配合使用

```
go get -u github.com/jstemmer/gotags
```

配置 VIM
------

可以直接使用 [github.com/lijiaocn/vim](https://github.com/lijiaocn/vim "lijiaocn vim") 中的 vim 配置，已经包含了多种常用插件。

### 安装 vim-go

[vim-go](https://github.com/fatih/vim-go "https://github.com/fatih/vim-go") 是 vim 的一个 golang 全家桶插件，提供了很多非常实用的命令。

```
go get -u github.com/nsf/gocode
```

快捷键需要自行设置。vim-go/doc/vim-go.txt 的 go-mappings 中给出可用的映射:

```
:GoBuild
:GoInstall
:GoTest
:GoCoverage
:GoCoverageBrowser
:GoDef
:GoDecls 
:GoDeclsDir
:GoDoc
:GoDocBrowser
:GoRun
:GoImplements
:GoCallees
:GoReferrers
:GoPath
:GoMetaLinter
:GoRename
:GoPlay
:GoAlternate
:GoAddTags
:GoImport
:GoDrop
```

vim-go 的作者 [fatih](https://www.patreon.com/fatih "https://www.patreon.com/fatih") 专门写了一个[教程](https://github.com/fatih/vim-go-tutorial "https://github.com/fatih/vim-go-tutorial")。

### vim-go 安装

安装很简单，将插件加入到 vim 中即可：

```
au FileType go nmap <leader>r <Plug>(go-run)
au FileType go nmap <leader>b <Plug>(go-build)
au FileType go nmap <leader>t <Plug>(go-test)
au FileType go nmap <leader>c <Plug>(go-coverage)
```

然后直接在 vim 中执行:

但是要注意这个过程会联网安装，会访问 golang.org，可能需要翻墙。

好在 github 上有 mirror, [https://github.com/golang/tools](https://github.com/golang/tools "https://github.com/golang/tools")，修改 vim-go 的 plugin 目录下 go.vim 中的依赖包:

在. vim/bundle/vim-go/plugin/go.vim 中可以看到依赖包:

```
Pathogen:
    git clone https://github.com/fatih/vim-go.git ~/.vim/bundle/vim-go
vim-plug:
    Plug 'fatih/vim-go'
Vim packages:
    git clone https://github.com/fatih/vim-go.git ~/.vim/pack/plugins/start/vim-go
```

手动用 mirror 安装依赖包:

```
:GoInstallBinaries
```

### YouCompleteMe

[YCM](https://github.com/Valloric/YouCompleteMe "https://github.com/Valloric/YouCompleteMe") 需要 7.4 的 vim，在 mac 系统上可以通过下面的命令升级 [Mac 自带的 Vim 怎么升级？](https://www.zhihu.com/question/34113076 "https://www.zhihu.com/question/34113076")：

```
let s:packages = [
            \ "github.com/nsf/gocode",
            \ "github.com/alecthomas/gometalinter",
            \ "golang.org/x/tools/cmd/goimports",
            \ "golang.org/x/tools/cmd/guru",
            \ "golang.org/x/tools/cmd/gorename",
            \ "github.com/golang/lint/golint",
            \ "github.com/kisielk/errcheck",
            \ "github.com/jstemmer/gotags",
            \ "github.com/klauspost/asmfmt/cmd/asmfmt",
            \ "github.com/fatih/motion",
            \ "github.com/zmb3/gogetdoc",
            \ ]
```

更多参数通过`brew info vim`查看，例如支持 python3：

或者使用 YCM 推荐的 macvim:

```
go get github.com/golang/tools/cmd/goimports
go get github.com/golang/tools/cmd/guru
go get github.com/golang/tools/cmd/gorename
```

在 macOS 上安装 YCM：

```
brew install vim --with-lua --with-override-system-vi
```

如果要支持 C 系列和 Go 语言：

```
—with-python3
```

否则：

```
brew install macvim    //YCM推荐使用macvim
```

如果需要支持其它语言，使用对应的参数：

```
brew install cmake
cd .vim/bundle
git clone https://github.com/Valloric/YouCompleteMe.git
```

支持所有语言：

Golang 零碎事项
-----------

### 换行

没错，在 Go 中换行是一个需要注意的问题。因为 Go 没有向 C 语言那样用”;” 作为一行代码的结束。因此有时候需要一些特别处理

例如 1:

```
cd ~/.vim/bundle/YouCompleteMe
git submodule update --init --recursive
./install.py --clang-completer --go-completer
```

例如 2

```
cd ~/.vim/bundle/YouCompleteMe
./install.py
```

> 吐槽: 在用 Go 做一些小东西过程中遇到这个问题, 突然发现整个编码过程中基本就没用的”;”, 突然感觉 C 中的”;” 好冗余。。

### 使用在其他包中定义的结构体

假设在包 A 中定义了:

```
--go-completer   //go
--js-completer   //js
--rust-completer //rust
--all            //所有支持的语言
```

当在 B 包使用时, 只能使用大写字母开头的成员 (X Y Z):

```
./install.py
```

### 取消证书验证

golang 的 net/http 中提供了 http 客户端 client, 可以用 client 发起 http 操作。但是当目标是 https 时，client 默认会检查证书。在做 demo 的时候，往往会做一个自己做一个临时证书, 因此需要关闭 Client 的证书检查。

这里有同样的问题: [https://groups.google.com/forum/##!topic/golang-nuts/TC5DVxYLjjg](https://groups.google.com/forum/#!topic/golang-nuts/TC5DVxYLjjg)

上述连接中，给出的解决方法:

```
res, err := db.Execute("insert into Session set ItemID=?,ItemType=?,LoginOK=0,"+ 
		 "LoginErr=0,CreateTime=NOW(),LastConnect=NOW()", humanid, itemtype)

 注意: 连接字符串的"+"必须在第一行，不然Go会以为第一行已经结束了
```

经过实际验证可行！如下:

```
err = db.QueryRow("select NickName, MailIdentify, TIMEDIFF(UnLockTime, NOW()) from "+ 
		"Human where HumanID=? and Passwd=SHA(?)",humanid, zainar.AddSalt(password, email)).
		Scan(&nickname, &realmail, &timewait)

注意"."号必须位于第二行的行尾，原理同上
```

> 其实就是替换了默认的设置

参考
--

1.  [vim-go](https://github.com/fatih/vim-go "https://github.com/fatih/vim-go")
2.  [go get tools](https://github.com/golang/go/wiki/GoGetTools "go get tools")
3.  [gotag](https://github.com/jstemmer/gotags "gotag")
4.  [gocode](https://github.com/nsf/gocode "gocode")
5.  [lijiaocn vim](https://github.com/lijiaocn/vim "lijiaocn vim")
6.  [vim-go](https://github.com/fatih/vim-go "https://github.com/fatih/vim-go")
7.  [vim-go author: fatih Arslan](https://www.patreon.com/fatih "https://www.patreon.com/fatih")
8.  [vim-go tutorial](https://github.com/fatih/vim-go-tutorial "https://github.com/fatih/vim-go-tutorial")
9.  [golang tools mirror](https://github.com/golang/tools "https://github.com/golang/tools")