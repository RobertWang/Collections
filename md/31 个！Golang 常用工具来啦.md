> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [new.qq.com](https://new.qq.com/rain/a/20220728A08TJW00.html)

导语 | 本文主要分享 Golang 相关的一些使用工具，简单介绍工具作用和使用场景，不会详细介绍其使用，列举的工具也不是最全的，具体可以参考链接或自行搜索学习。

Go 官方的工具可以使用 go help xxx 命令查看帮助文档，比如查看 go get 的参数标记和使用文档：

可以参考 Golang 官方的文档：https://golang.google.cn/cmd/go/

**一、G0 官方工具**

（一）go get

该命令可以根据要求和实际情况从互联网上下载或更新指定的代码包及其依赖包，下载后自动编译，一般引用依赖用 go get 就可以了。

参考：

参考：

https://www.kancloud.cn/cattong/go_command_tutorial/261349

（二）go build

该命令用于编译我们指定的源码文件或代码包以及它们的依赖包。命令的常用标记说明如下：

![](http://inews.gtimg.com/newsapp_bt/0/15127015524/1000)

编译过程输出到文件：go build -x > result 2>&1，因为 go build -x 最终是将日志写到标准错误流当中。

如果只在编译特定包时需要指定参数，可以参考包名 = 参数列表的格式，比如 go build -gcflags='log=-N -l' main.go

参考：

https://www.kancloud.cn/cattong/go_command_tutorial/261347

（三）go install

该命令用于编译并安装指定的代码包及它们的依赖包。当指定的代码包的依赖包还没有被编译和安装时，该命令会先去处理依赖包。与 go build 命令一样，传给 go install 命令的代码包参数应该以导入路径的形式提供。

并且，go build 命令的绝大多数标记也都可以用于 go install 命令。实际上，go install 命令只比 go build 命令多做了一件事，即：安装编译后的结果文件到指定目录。

参考：

https://www.kancloud.cn/cattong/go_command_tutorial/261348

（四）go fmt 和 gofmt

Golang 的开发团队制定了统一的官方代码风格，并且推出了 gofmt 工具（gofmt 或 go fmt）来帮助开发者格式化他们的代码到统一的风格。

gofmt 是一个 cli 程序，会优先读取标准输入，如果传入了文件路径的话，会格式化这个文件，如果传入一个目录，会格式化目录中所有. go 文件，如果不传参数，会格式化当前目录下的所有. go 文件。

gofmt 默认不对代码进行简化，使用 - s 参数可以开启简化代码功能

gofmt 是一个独立的 cli 程序，而 go 中还有一个 go fmt 命令，go fmt 命令是 gofmt 的简单封装。go fmt 在调用 gofmt 时添加了 - l -w 参数，相当于执行了 gofmt -l -w

参考：

https://blog.csdn.net/whatday/article/details/97682094

（五）go env

该命令用于打印 Go 语言的环境信息，常见的通用环境信息如下：

![](http://inews.gtimg.com/newsapp_bt/0/15127015526/1000)

设置或修改环境变量值：

参考：

https://www.kancloud.cn/cattong/go_command_tutorial/261359

（六）go run

该命令可以运行命令源码文件，只能接受一个命令源码文件以及若干个库源码文件（必须同属于 main 包）作为文件参数，且不能接受测试源码文件。它在执行时会检查源码文件的类型。如果参数中有多个或者没有命令源码文件，那么 go run 命令就只会打印错误提示信息并退出，而不会继续执行。

在通过参数检查后，go run 命令会将编译参数中的命令源码文件，并把编译后的可执行文件存放到临时工作目录中。

参考：

https://www.kancloud.cn/cattong/go_command_tutorial/261352

（七）go test

该命令用于对 Go 语言编写的程序进行测试，这种测试是以代码包为单位的，命令会自动测试每一个指定的代码包。当然，前提是指定的代码包中存在测试源码文件。

参考：

https://www.kancloud.cn/cattong/go_command_tutorial/261353

（八）go clean

该命令会删除掉执行其它命令时产生的一些文件和目录。

参考：

https://www.kancloud.cn/cattong/go_command_tutorial/261350

（九）go list

该命令的作用是列出指定的代码包的信息。与其他命令相同，我们需要以代码包导入路径的方式给定代码包。被给定的代码包可以有多个。这些代码包对应的目录中必须直接保存有 Go 语言源码文件，其子目录中的文件不算在内。

标记 - e 的作用是以容错模式加载和分析指定的代码包。在这种情况下，命令程序如果在加载或分析的过程中遇到错误只会在内部记录一下，而不会直接把错误信息打印出来。

为了看到错误信息可以使用 - json 标记。这个标记的作用是把代码包的结构体实例用 JSON 的样式打印出来。-m 标记可以打印出 modules 而不是 package。

参考：

https://www.kancloud.cn/cattong/go_command_tutorial/261354

（十）go mod xxx

go mod init

该命令初始化并写入一个新的 go.mod 至当前目录中，实际上是创建一个以当前目录为根的新模块。文件 go.mod 必须不存在。如果可能，init 会从 import 注释（参阅 “go help importpath”）或从版本控制配置猜测模块路径。要覆盖此猜测，提供模块路径作为参数 module 为当前项目名。比如：

参考：

https://www.jianshu.com/p/f6d2d6db2bca

go mod tidy

该命令确保 go.mod 与模块中的源代码一致。它添加构建当前模块的包和依赖所必须的任何缺少的模块，删除不提供任何有价值的包的未使用的模块。它也会添加任何缺少的条目至 go.mod 并删除任何不需要的条目。

参考：

https://www.jianshu.com/p/f6d2d6db2bca

go mod vendor

该命令重置主模块的 vendor 目录，使其包含构建和测试所有主模块的包所需要的所有包。不包括 vendor 中的包的测试代码。即将 GOPATH 或 GOROOT 下载的包拷贝到项目下的 vendor 目录，如果不使用 vendor 隔离项目的依赖，则不需要使用该命令拷贝依赖。

参考：

https://www.jianshu.com/p/f6d2d6db2bca

go moddownload

该命令下载指定名字的模块，可为选择主模块依赖的模块匹配模式，或 path@version 形式的模块查询。如果 download 不带参数则代表是主模块的所有依赖。download 的只会下载依赖，不会编译依赖，和 get 是有区别的。

参考：

https://www.jianshu.com/p/f6d2d6db2bca

go mod edit

该命令提供一个编辑 go.mod 的命令行接口，主要提供给工具或脚本使用。它只读取 go.mod；不查找涉及模块的信息。默认情况下，edit 读写主模块的 go.mod 文件，但也可以在标志后指定不同的目标文件。

参考：

https://www.jianshu.com/p/f6d2d6db2bca

go mod graph

该命令以文本形式打印模块间的依赖关系图。输出的每一行行有两个字段（通过空格分割）；模块和其所有依赖中的一个。每个模块都被标记为 path@version 形式的字符串（除了主模块，因其没有 @version 后缀）。

参考：

https://www.jianshu.com/p/f6d2d6db2bca

go mod verify

该命令查存储在本地下载源代码缓存中的当前模块的依赖，是否自从下载之后未被修改。如果所有模块都未被修改，打印 “all modules verified”。否则，报告哪个模块已经被修改并令“go mod” 以非 0 状态退出。

参考：

https://www.jianshu.com/p/f6d2d6db2bca

go mod why

该命令输出每个包或者模块的引用块，每个块以注释行 “# package” 或“# module”开头，给出目标包或模块。随后的行通过导入图给出路径，一个包一行。每个块之间通过一个空行分割，如果包或模块没有被主模块引用，该小节将显示单独一个带圆括号的提示信息来表明该事实。

参考：

https://www.jianshu.com/p/f6d2d6db2bca

（十一）go tool xxx

go tool 的可执行文件在 GOROOT 或 GOPATH 的 pkg/tool 目录。go doc cmd 可以查看具体 cmd 的使用说明，比如：

go tool pprof

在 Golang 中，可以通过 pprof 工具对应于程序的运行时进行性能分析，包括 CPU、内存、Goroutine 等实时信息。

参考：

https://www.kancloud.cn/cattong/go_command_tutorial/261357

go tool trace

该命令可以追踪请求链路，清晰的了解整个程序的调用栈，可以通过追踪器捕获大量信息。

参考：

https://zhuanlan.zhihu.com/p/410590497

go tool compile

该命令可以编译 Go 文件生成汇编代码，-N 参数表示禁止编译优化， -l 表示禁止内联，-S 表示打印汇编，比如

go tool vet 和 go vet

该命令是一个用于检查 Go 语言源码中静态错误的简单工具。

go vet 命令是 go tool vet 命令的简单封装。它会首先载入和分析指定的代码包，并把指定代码包中的所有 Go 语言源码文件和以 “.s” 结尾的文件的相对路径作为参数传递给 go tool vet 命令。其中，以 “.s” 结尾的文件是汇编语言的源码文件。如果 go vet 命令的参数是 Go 语言源码文件的路径，则会直接将这些参数传递给 go tool vet 命令。

参考：

https://www.kancloud.cn/cattong/go_command_tutorial/261356

go tool doc 和 go doc

该命令可以打印附于 Go 语言程序实体上的文档。我们可以通过把程序实体的标识符作为该命令的参数来达到查看其文档的目的。

所谓 Go 语言的程序实体，是指变量、常量、函数、结构体以及接口。而程序实体的标识符即是代表它们的名称。

参考：

https://www.kancloud.cn/cattong/go_command_tutorial/261351

go tool addr2line

该命令可以调用栈的地址转化为文件和行号。

go tool asm

该命令可以将汇编文件编译成一个. o 文件，后续这个. o 文件可以用于生成. a 归档文件，命令的 file 参数必须是汇编文件。

go tool buildid

每一个 Go 二进制文件内，都有一个独一无二的 Build ID，详情参考 src/cmd/go/internal/work/buildid.go 。Go Build ID 可以用以下命令来查看：

参考：

https://www.anquanke.com/post/id/215419

go tool cgo

该命令可以使我们创建能够调用 C 语言代码的 Go 语言源码文件。这使得我们可以使用 Go 语言代码去封装一些 C 语言的代码库，并提供给 Go 语言代码或项目使用。

参考：

https://www.kancloud.cn/cattong/go_command_tutorial/261358

go tool cover

该命令对单元测试过程中生成的代码覆盖率统计生成 html 文件，可以本地打开展示。

覆盖度工具不仅可以记录分支是否被执行，还可以记录分支被执行了多少次。

go test -covermode=set|count|atomic:-covermode：

set: 默认模式，统计是否执行

count: 计数

atomic: count 的并发安全版本，仅当需要精确统计时使用

通过 go tool cover -func=count.out 查看每个函数的覆盖度。

go tool dist

dist 工具是属于 go 的一个引导工具，它负责构建 C 程序（如 Go 编译器）和 Go 工具的初始引导副本。它也可以作为一个包罗万象用 shell 脚本替换以前完成的零工。通过 “go tool dist” 命令可以操作该工具。

go tool fix 和 go fix

该命令会把指定代码包的所有 Go 语言源码文件中的旧版本代码修正为新版本的代码。这里所说的版本即 Go 语言的版本。代码包的所有 Go 语言源码文件不包括其子代码包（如果有的话）中的文件。修正操作包括把对旧程序调用的代码更换为对新程序调用的代码、把旧的语法更换为新的语法等等。

这个工具其实非常有用。在编程语言的升级和演进的过程中，难免会对过时的和不够优秀的语法及标准库进行改进。

参考：

https://www.kancloud.cn/cattong/go_command_tutorial/261355

go tool link

该命令链接 Go 的归档文件比如静态库，以及链接其所有依赖，生成一个可执行文件（含 main package）。

go tool nm

该命令可以查看符号表的命令，等同于系统的 nm 命令，非常有用。在断点的时候，如果你不知道断点的函数符号，那么用这个命令查一下就知道了（命令处理的是二进制程序文件），第一列是地址，第二列是类型，第三列是符号。等同于 nm 命令。

参考：

https://studygolang.com/articles/29906

go tool objdump

该命令可以反汇编二进制的工具，等同于系统 objdump，命令解析的是二进制格式的程序文件。

参考：

https://studygolang.com/articles/29906

go tool pack

该命令把二进制文件打包成静态库。

go tool test2json

该命令用于把测试可执行文件转化可读的 json 格式。

godoc

该命令可以生成一个 Go 官方文档的本地 http 服务，可以在线查看标准库和第三方库文档，以及项目文档，但是需要按照一定的格式去写注释。

Web 页面如下：

![](http://inews.gtimg.com/newsapp_bt/0/15127015747/1000)

参考：https://www.fujieace.com/golang/godoc.html

**二、第三方工具**

Go 工具和组件汇总项目：

https://github.com/avelino/awesome-go

（一）delve

本地代码调试工具

参考：https://github.com/go-delve/delve

（二）goconvey

goconvey 是一款针对 Golang 的测试框架，可以管理和运行测试用例，同时提供了丰富的断言函数，并支持很多 Web 界面特性。

参考：https://github.com/smartystreets/goconvey

（三）goleak

本地排查内存泄露的工具

参考：https://github.com/uber-go/goleak

（四）go-wrk

Go 接口压测工具

参考：https://github.com/adjust/go-wrk

（五）golint

代码风格检查

参考：https://github.com/golang/lint

（六）revive

代码风格检查，比 golint 速度更快

参考：https://github.com/mgechev/revive

（七）gocode

代码自动补全工具，可以在 vim 中使用

参考：https://github.com/nsf/gocode

（八）godoctor

代码重构工具

参考：https://github.com/godoctor/godoctor

（九）gops

查看 go 进程和相关信息的工具，用于诊断线上服务。

参考：https://github.com/google/gops

（十）goreplay

GoReplay 是一个开源网络监控工具，可以将实时 HTTP 流量捕获并重放到测试环境。

参考：https://github.com/buger/goreplay

https://blog.51cto.com/axzxs/5102596

（十一）depth

一个有用的 Golang 工具，Depth 可帮助 Web 开发人员检索和可视化 Go 源代码依赖关系树。它可以用作独立的命令行应用程序或作为项目中的特定包。你可以通过在解析之前在 Tree 上设置相应的标志来添加自定义。

参考：https://github.com/KyleBanks/depth

（十二）go-swagger

该工具包包括各种功能和功能。Go-Swagger 是 Swagger 2.0 的一个实现，可以序列化和反序列化 swagger 规范。它是 RESTful API 简约但强大的代表。

通过 Go-Swagger，你可以 swagger 规范文档，验证 JSON 模式以及其他额外的规则。其他功能包括代码生成，基于 swagger 规范的 API 生成，基于代码的规范文档生成，扩展了的字符串格式，等等。

参考：https://github.com/go-swagger/go-swagger

（十三）gox

交叉编译工具，可以并行编译多个平台。

参考：https://github.com/mitchellh/gox

（十四）gocyclo

gocyclo 用来检查函数的复杂度。

参考：https://github.com/fzipp/gocyclo

（十五）deadcode

deadcode 会告诉你哪些代码片段根本没用。

参考：https://github.com/tsenart/deadcode

（十六）gotype

gotype 会对 go 文件和包进行语义 (semantic) 和句法 (syntactic) 的分析，这是 google 提供的一个工具。

参考：https://golang.org/x/tools/cmd/gotype

（十七）misspell

misspell 用来拼写检查，对国内英语不太熟练的同学很有帮助。

参考：https://github.com/client9/misspell

（十八）staticcheck

staticcheck 是一个超牛的工具，提供了巨多的静态检查，就像 C# 生态圈的 ReSharper 一样。

参考：

https://staticcheck.io/docs/

https://github.com/dominikh/go-tools/tree/master/staticcheck

（十九）goconst

goconst 会查找重复的字符串，这些字符串可以抽取成常量。

参考：https://github.com/jgautheron/goconst

**参考资料：**

1.Go 命令教程：

https://www.kancloud.cn/cattong/go_command_tutorial/261351

2.Golang 指南：顶级 Golang 框架、IDE 和工具列表：

https://zhuanlan.zhihu.com/p/30432648

3.Go 代码检修工具集：

http://t.zoukankan.com/binHome-p-14149941.html

**作者简介**

**罗元国**

腾讯后台开发工程师

腾讯后台开发工程师，目前负责腾讯游戏广告推荐后台开发工作，在广告推荐和 Golang 性能优化方面有着丰富的开发经验。

[![](http://inews.gtimg.com/newsapp_bt/0/1012205723968_6694/0)举报](https://new.qq.com/jvbao/index.htm?url=https://new.qq.com/rain/a/20220728A08TJW00.html)免责声明：本文来自腾讯新闻客户端创作者，不代表腾讯网的观点和立场。