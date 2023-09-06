> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/snVLySXv4E5xMlbD153FcQ)

上周末，一个 Gopher 在微信上与我交流了一个有关 Go 程序编译的问题。他的述求说起来也不复杂，那就是合作公司提供的 API 包仅包括 golang archive(使用 go build -buildmode=archive 构建的. a 文件)，没有 Go 包的源码。如何将这个. a 链接到项目构建出的最终可执行程序中呢？

对于 C、C++、Java 程序员来说，仅提供静态链接库或动态链接库 [1](包括头文件)、jar 包而不提供源码的 API 是十分寻常的。但对于 Go 来说，仅提供 Go 包的 archive(.a) 文件，而不提供 Go 包源码的情况却是极其不常见的。究其原因，简单来说就是 **go build 或 go run 不支持**！

> 注：《Go 语言精进之路 vo1》[2] 一书的第 16 条 “理解 Go 语言的包导入” 对 Go 的编译过程和原理做了系统说明。

那么真的就没有方法实现没有 source、仅基于. a 文件的 Go 应用构建了吗？也不是。的确有一些 hack 的方法可以实现这点，本文就来从技术角度来探讨一下这些 hack 方法，但并**不推荐使用**！

1. 回顾 go build 不支持 "no source, only .a"
---------------------------------------

我们首先来回顾一下 go build 在 "no source, only .a" 下的表现。为此，我们先建立一个实验环境，其目录和文件布局如下：

```
// 没有外部依赖的api包: foo

$tree goarchive-nodeps 
goarchive-nodeps
├── Makefile
├── foo.a
├── foo.go
└── go.mod

$tree library 
library
└── github.com
    └── bigwhite
        └── foo.a

// 依赖foo包的app工程
$tree app-link-foo
app-link-foo
├── Makefile
├── go.mod
└── main.go


```

这里我们已经将 app-link-foo 依赖的 foo.a 构建了出来 (通过 go build -buildmode=arhive)，并放入了 library 对应的目录下。

> 注：可通过 ar -x foo.a 命令可以查看 foo.a 的组成。

现在我们使用 go build 来构建 app-link-foo 工程：

```
$cd app-link-foo
$go build
main.go:6:2: no required module provides package github.com/bigwhite/foo; to add it:
 go get github.com/bigwhite/foo


```

我们看到：go build 会分析 app-link-foo 的依赖，并要求获取其依赖的 foo 包的代码，但我们无法满足 go build 这一要求！

有人可能会说：go build 支持向 go build 支持向 compiler 和 linker 传递参数，是不是将 foo.a 的位置告知 compiler 和 linker 就可以了呢？我们来试试：

```
$go build -x -v -gcflags '-I /Users/tonybai/Go/src/github.com/bigwhite/experiments/build-with-archive-only/library' -ldflags '-L /Users/tonybai/Go/src/github.com/bigwhite/experiments/build-with-archive-only/library' -o main main.go
main.go:6:2: no required module provides package github.com/bigwhite/foo; to add it:
 go get github.com/bigwhite/foo
make: *** [build] Error 1


```

我们看到：即便向 go build 传入 gcflags 和 ldflags 参数，告知了 foo.a 的搜索路径，go build 依然报错，仍然提示需要 foo 包的源码！也就是说 go build 还没到调用 go tool compile 和 go tool link 那一步就开始报错了！

go build 不支持在无源码情况下链接. a，那么我们**只能绕过 go build** 了！

2. 绕过 go bulid
--------------

认真读过《Go 语言精进之路 vo1》[3] 一书的朋友都会知道：go build 实质是调用 go tool compile 和 go tool link 两个命令来完成 go 应用的构建过程的，使用 go build -x -v 可以查看到 go build 的详细构建过程。

接下来，我们就来扮演一下 "go build"，以手动的方式分别调用 go tool compile 和 go tool link，看看是否能达到无需依赖包源码就能成功构建的目标。

我们以 foo.a 这个自身没有外部依赖的 go archive 为例，用手动方式构建一下 app-link-foo 这个工程。

首先确保通过 - buildmode=archive 构建出的 foo.a 被正确放入 library/github.com/bigwhite 下面。

接下来，我们通过 go tool compile 编译一下 app-link-foo：

```
$cd app-link-foo
$go tool compile -I /Users/tonybai/Go/src/github.com/bigwhite/experiments/build-with-archive-only/library -o main.o main.go


```

我们看到：手动执行 go tool compile 在通过 - I 传入依赖库的. a 文件时是可以正常编译出 object file(目标文件) 的。go tool compile 的手册告诉我们 - I 选项为 compile 提供了搜索包导入路径的目录：

```
$go tool compile -h
  ... ...
  -I directory
     add directory to import search path
  ... ...


```

接下来我们用 go tool link 将 main.o 和 foo.a 链接在一起形成可执行二进制文件 main：

```
$cd app-link-foo
$go tool link -L /Users/tonybai/Go/src/github.com/bigwhite/experiments/build-with-archive-only/library -o main main.o


```

通过 go tool link 并在 - L 传入 foo.a 的链接路径的情况下，我们成功地将 main.o 和 foo.a 链接在了一起，形成了最终的可执行文件 main。

go tool link 的 - L 选项为 link 提供了搜索. a 的路径：

```
$go tool link -h
  ... ...
  -L directory
     add specified directory to library path
  ... ...


```

执行一下编译链接后的二进制文件 main，我们将看到与预期相同的输出结果：

```
$./main
invoke foo.Add
11


```

有些童鞋在执行 go tool compile 时可能会遇到找不到 fmt.a 或 fmt.o 的错误！这是因为 Go 1.20 版本及以后，Go 安装包默认将不会在 GOOS_$GOARCH 下面安装标准库的. a 文件集合，这样 go tool compile 在这个路径下面就找不到 app-link-foo 所依赖的 fmt.a：

```
➜  /Users/tonybai/.bin/go1.20/pkg git:(master) ✗ $ls
darwin_amd64/    include/    tool/
➜  /Users/tonybai/.bin/go1.20/pkg git:(master) ✗ $cd darwin_amd64
➜  /Users/tonybai/.bin/go1.20/pkg/darwin_amd64 git:(master) ✗ $ls


```

解决方法也很简单，那就是手动执行下面命令编译和安装一下标准库的. a 文件：

```
$GODEBUG=installgoroot=all  go install std

➜  /Users/tonybai/.bin/go1.20/pkg/darwin_amd64 git:(master) ✗ $ls
archive/    database/    fmt.a        index/        mime/        plugin.a    strconv.a    time/
bufio.a        debug/        go/        internal/    mime.a        reflect/    strings.a    time.a
bytes.a        embed.a        hash/        io/        net/        reflect.a    sync/        unicode/
compress/    encoding/    hash.a        io.a        net.a        regexp/        sync.a        unicode.a
container/    encoding.a    html/        log/        os/        regexp.a    syscall.a    vendor/
context.a    errors.a    html.a        log.a        os.a        runtime/    testing/
crypto/        expvar.a    image/        math/        path/        runtime.a    testing.a
crypto.a    flag.a        image.a        math.a        path.a        sort.a        text/


```

这样无论是 go tool compile，还是 go tool link 都会找到对应的标准库包了！

在这个例子中，foo.a 仅依赖标准库，没有依赖第三方库，这样相对简单一些。通常合作伙伴提供的. a 中的包都是依赖第三方的包的，下面我们就来看看如果. a 有第三方依赖，上面的编译链接方法是否还能奏效！

3. 要链接的. a 文件自身也依赖第三方包
----------------------

goarchive-with-deps 目录下的 bar.a 就是一个自身也依赖第三方包的 go archive 文件，它依赖的是 uber 的 zap 日志包 [4] 以及 zap 包的依赖链，下面是 bar 的 go.mod 文件的内容：

```
// goarchive-with-deps/go.mod

module github.com/bigwhite/bar

go 1.20

require go.uber.org/zap v1.25.0

require go.uber.org/multierr v1.10.0


```

我们先来安装 app-link-foo 的思路来编译链接一下 app-link-bar：

```
$cd app-link-bar
$make
go tool compile -I /Users/tonybai/Go/src/github.com/bigwhite/experiments/build-with-archive-only/library -o main.o main.go
go tool link -L /Users/tonybai/Go/src/github.com/bigwhite/experiments/build-with-archive-only/library -o main main.o
/Users/tonybai/.bin/go1.20/pkg/tool/darwin_amd64/link: cannot open file /Users/tonybai/.bin/go1.20/pkg/darwin_amd64/go.uber.org/zap.o: open /Users/tonybai/.bin/go1.20/pkg/darwin_amd64/go.uber.org/zap.o: no such file or directory
make: *** [all] Error 1


```

上面报的错误符合预期，因为 zap.a 尚没有放入 build-with-archive-only/library 下面。接下来我们基于 uber zap 的源码构建出一个 zap.a 并放入指定目录。bar.a 依赖的 uber zap 的版本为 v1.25.0，于是我们 git clone 一下 uber zap，checkout 出 v1.25.0 并执行构建：

```
$cd go/src/go.uber.org/zap
$go build -o zap.a -buildmode=archive .
$cp zap.a /Users/tonybai/Go/src/github.com/bigwhite/experiments/build-with-archive-only/library/go.uber.org/


```

再来编译一下 app-link-bar：

```
$make
go tool compile -I /Users/tonybai/Go/src/github.com/bigwhite/experiments/build-with-archive-only/library -o main.o main.go
go tool link -L /Users/tonybai/Go/src/github.com/bigwhite/experiments/build-with-archive-only/library -o main main.o
/Users/tonybai/.bin/go1.20/pkg/tool/darwin_amd64/link: fingerprint mismatch: go.uber.org/zap has b259b1e07032c6d9, import from github.com/bigwhite/bar expecting 8118f660c835360a
make: *** [all] Error 1


```

我们看到 go tool link 报错，提示 “fingerprint mismatch”。这个错误的意思是 bar.a 期望的 zap 包的指纹与我们提供的在 Library 目录下的 zap 包的指纹不一致！

我们重新用 go build -v -x 来看一下 bar.a 的构建过程：

```
$go build -x -v  -o bar.a -buildmode=archive
WORK=/var/folders/cz/sbj5kg2d3m3c6j650z0qfm800000gn/T/go-build3367014838
github.com/bigwhite/bar
mkdir -p $WORK/b001/
cat >/var/folders/cz/sbj5kg2d3m3c6j650z0qfm800000gn/T/go-build3367014838/b001/importcfg << 'EOF' # internal
# import config
packagefile fmt=/Users/tonybai/Library/Caches/go-build/d3/d307b52dabc7d78a8ff219fb472fbc0b0a600038f43cd4c737914f8ccbd2bd70-d
packagefile go.uber.org/zap=/Users/tonybai/Library/Caches/go-build/00/006d48e40c867a336b9ac622478c1e5bf914e6a5986f649a096ebede3d117bba-d
EOF
cd /Users/tonybai/Go/src/github.com/bigwhite/experiments/build-with-archive-only/goarchive-with-deps
/Users/tonybai/.bin/go1.20/pkg/tool/darwin_amd64/compile -o $WORK/b001/_pkg_.a -trimpath "$WORK/b001=>" -p github.com/bigwhite/bar -lang=go1.20 -complete -buildid mIMNOXMPJH00mEpw6WVc/mIMNOXMPJH00mEpw6WVc -goversion go1.20 -c=4 -nolocalimports -importcfg $WORK/b001/importcfg -pack ./bar.go
/Users/tonybai/.bin/go1.20/pkg/tool/darwin_amd64/buildid -w $WORK/b001/_pkg_.a # internal
cp $WORK/b001/_pkg_.a /Users/tonybai/Library/Caches/go-build/60/604b60360d384c49eb9c030a2726f02588f54375748ce1421e334bedfda2af47-d # internal
mv $WORK/b001/_pkg_.a bar.a
rm -r $WORK/b001/


```

我们看到在编译 bar.a 的过程中，go tool compile 用的是 - importcfg 来得到的 go.uber.org/zap 的位置，而从打印的内容来看，go.uber.org/zap 指向的是 go module cache 中的某个文件：packagefile go.uber.org/zap=/Users/tonybai/Library/Caches/go-build/00/006d48e40c867a336b9ac622478c1e5bf914e6a5986f649a096ebede3d117bba-d。

那是不是在 build app-link-bar 时也使用这个同样的 go.uber.org/zap 就可以成功通过 go tool link 的过程呢？我们来试一下：

```
$cd app-link-bar
$make build-with-importcfg
go tool compile -importcfg import.link -o main.o main.go
go tool link -importcfg import.link -o main main.o

$./main
invoke foo.Add
{"level":"info","ts":1693203940.0701509,"caller":"goarchive-with-deps/bar.go:14","msg":"invoke bar.Add\n"}
11


```

使用 - importcfg 的确成功的编译链接了 app-link-bar，其执行结果也符合预期！注意：这里我们放弃了之前使用的 - I 和 - L，即便应用 - I 和 - L，在与 - importcfg 联合使用时，go tool compile 和 link 也会以 - importcfg 的信息为准！

现在还有一个问题摆在面前，那就是上述命令行中的 import.link 这个文件的内容是啥，又是如何生成的呢？这里的 import.link 文件十分 “巨大”，有 500 多行，其内容大致如下：

```
// app-link-bar/import.link

# import config
packagefile internal/goos=/Users/tonybai/Library/Caches/go-build/fa/facce9766a2b3c19364ee55c509863694b205190c504a3831cde7c208bb09f37-d
packagefile vendor/golang.org/x/crypto/chacha20=/Users/tonybai/Library/Caches/go-build/e0/e042b43b78d3596cc00e544a40a13e8cd6b566eb8f59c2d47aeb0bbcbd52aa56-d
... ...

packagefile github.com/bigwhite/bar=/Users/tonybai/Go/src/github.com/bigwhite/experiments/build-with-archive-only/library/github.com/bigwhite/bar.a
packagefile go.uber.org/zap=/Users/tonybai/Library/Caches/go-build/00/006d48e40c867a336b9ac622478c1e5bf914e6a5986f649a096ebede3d117bba-d
packagefile go.uber.org/zap/zapcore=/Users/tonybai/Library/Caches/go-build/e0/e0d81701b5d15628ce5bf174e5c1b7482c13ac3a3c868e9b054da8b1596eaace-d
packagefile go.uber.org/zap/internal/pool=/Users/tonybai/Library/Caches/go-build/bf/bfa96ebb89429b870e2c50c990c1945384e50d10ba354a3dab2b995a813c56a3-d
packagefile go.uber.org/zap/internal=/Users/tonybai/Library/Caches/go-build/33/33cb66c30939b8be915ddc1e237a04688f52c492d3ae58bfbc6196fff8b6b2b5-d
packagefile go.uber.org/zap/internal/bufferpool=/Users/tonybai/Library/Caches/go-build/68/68e58338a5acd96ee1733de78547720f26f4e13d8333defbc00099ac8560c8e8-d
packagefile go.uber.org/zap/buffer=/Users/tonybai/Library/Caches/go-build/7b/7bf00a1d4a69ddb1712366f45451890f3205b58ba49627ed4254acd9b0938ef8-d
packagefile go.uber.org/multierr=/Users/tonybai/Library/Caches/go-build/e7/e7cc278d56fc8262d9cf9de840a04aa675c75f8ac148e955c1ae9950c58c8034-d
packagefile go.uber.org/zap/internal/exit=/Users/tonybai/Library/Caches/go-build/18/187b2b490c810f37c3700132fba12b805e74bd3c59303972bcf74894a63de604-d
packagefile go.uber.org/zap/internal/color=/Users/tonybai/Library/Caches/go-build/e4/e419c93bea7ff2782b2047cf9e7ad37b07cf4a5a5b7f361bf968730e107a495b-d


```

这里包含了编译链接 app-link-bar 是依赖的标准库包、bar.a 以及 bar 包依赖的所有第三方包的实际包. a 文件的位置，显然这里用的大多数都是 go module cache 中的包缓存。

那么这个 import.link 如何得到呢？Go 在 golang.org/x/tools 包中有一个 importcfg.go 文件 [5]，基于该文件中的 Importcfg 函数可以获取标准库相关所有包的 package link 信息。我将该文件放在了 build-with-archive-only/importcfg 下了，大家可以自行取用。

importcfg 生成了大部分 package link，但仍会有一些 bar.a 依赖的第三方的包的 link 没有着落，go tool link 在链接时会报错，根据报错信息中提供的包导入路径信息，比如：找不到 go.uber.org/zap/internal/exit、go.uber.org/zap/internal/color，我们可以利用下面 go list 命令找到这些包的在本地 go module cache 中的 link 位置：

```
$go list -export -e -f "{{.ImportPath}} {{.Export}}" go.uber.org/zap/internal/exit go.uber.org/zap/internal/color
go.uber.org/zap/internal/exit /Users/tonybai/Library/Caches/go-build/18/187b2b490c810f37c3700132fba12b805e74bd3c59303972bcf74894a63de604-d
go.uber.org/zap/internal/color /Users/tonybai/Library/Caches/go-build/e4/e419c93bea7ff2782b2047cf9e7ad37b07cf4a5a5b7f361bf968730e107a495b-d


```

然后可以手工将这些信息 copy 到 import.link 中。import.link 文件就是在这样自动化 + 手工的过程中生成的（当然你完全可以自己编写一个工具，获取 app-link-bar 所需的所有 package 的 link 信息）。

4. 小结
-----

到这里，我们通过 hack 的方法实现了在没有源码只有. a 文件情况下的可执行程序的编译。

不过上述仅仅是纯技术上的探索，并非标准答案，也更非理想的答案。经过上述探索后，更巩固了我的观点：**不要仅使用. a 来构建 go 应用**。

但非要这么做，如果你是. a 的提供方，考虑 fingerprint mismatch 的情况，你估计要考虑在提供. a 的同时，还要提供 import.link、你构建. a 时所有用到的 go module cache 的副本，并提供安装这些副本到目标主机上的脚本。这样你的. a 用户才可能使用相同的依赖版本完成对. a 文件的链接过程。

本文试验的代码都是在 Go 1.20 版本 [6] 下编译链接的。如果编译. a 的 Go 版本与编译链接可执行文件的 Go 版本不同，是否会失败呢？这个问题就当做作业留个大家去探索了！

本文涉及的代码可以从这里 [7] 下载。

* * *

### 参考资料

[1] 

动态链接库: _https://tonybai.com/2011/07/07/also-talk-about-shared-library-2/_

[2] 

《Go 语言精进之路 vo1》: _https://item.jd.com/13694000.html_

[3] 

《Go 语言精进之路 vo1》: _https://item.jd.com/13694000.html_

[4] 

zap 日志包: _https://tonybai.com/2021/07/14/uber-zap-advanced-usage_

[5] 

importcfg.go 文件: _https://raw.githubusercontent.com/golang/tools/master/internal/goroot/importcfg.go_

[6] 

Go 1.20 版本: _https://tonybai.com/2023/02/08/some-changes-in-go-1-20/_

[7] 

这里: _https://github.com/bigwhite/experiments/blob/master/build-with-archive-only_

[8] 

“Gopher 部落” 知识星球: _https://public.zsxq.com/groups/51284458844544_

[9] 

链接地址: _https://m.do.co/c/bff6eed92687_