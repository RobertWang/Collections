> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/hs2JRWuUcUXBzfxlFxrHJw)

> 重学 Go 语言 | 如何使用 Go Modules

Go 语言在`Go.1.11`版本发布了`Go Modules`，这是一种新的 Go 项目依赖管理解决方案，可以让 Go 项目的依赖包关系更加清晰，也更容易管理。

Module
------

在`Go Modules`这种新的依赖机制中定义了`Module`这个概念，也就是所谓的**模块**。

### 模块的组成

一个模块就是一个或者多个包 (`package`) 的集合，创建模块时，会在根目录下生成一个`go.mod`文件，`go.mod`文件包含了模块的路径 (`module path`) 以及该模块依赖的其他模块列表，在下载依赖模块后，还会生成一个`go.sum`文件，因此大体上，模块的构成如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/gz1ggv1GsJXXXA86xndn5OwAPWfb2ibeuiba8WSGaPIEyVnEKRpUSgNiciajibXnNc3Ec4B5cNhZDw1Iic3iay6ib1Ag8A/640?wx_fmt=png)

### module path

`module path`，即模块路径，是该模块的身份标识，在模块内或者其他模块导入该模块的包时，必须以模块路径为前缀，比如模块路径为`github.com/test/test`，那么导入该模块的`hello`包时，则其导入的路径为：

```
import "github.com/test/test/hello"


```

下载依赖模块时也需要使用到模块路径：

```
go get github.com/test/test


```

### 语义化版本号

同一个模块可能会存在不同的版本，Go 模块采用语义化版本号来定义模块的版本，其版本号格式为：

```
v<major>.<minor>.<patch>


```

*   **major**：主版本号，当模块做了不兼容的修改后，需更改主版本号。
    
*   **minor**：次版本号，当模块做了向下兼容的功能时，需要更改次版本号。
    
*   **patch**：修订版本号，当模块做了向下兼容的 bug 修改时，需要更改修订版本号。
    

前面我们使用`go get`命令下载依赖模块时并没有指定义版本号，那么默认下载`master`分支上最新的`commit`：

```
go get github.com/test/test


```

如果指定版本号，且该版本号有对应的`tag`，那么就会下载对应的`tag`，如果没有对应`tag`，那么还是下载`master`分支上最新的`commmit`：

```
go get github.com/test/test@v1.1.1


```

使用`@latest`可以下载最新版本的模块：

```
go get github.com/test/test@latest


```

当主版本号为`v0`或者`v1`时，在下载和导入时，可以省略版本号，当主版本大于或等于 v2 时，不可省略主版本号：

比如我们现在要下载有模块有`v1.6.0`、`v1.7.0`以及`v2.0.5`三个版本，此时：

```
go get github.com/test/test
go get github.com/test/test@latest


```

上面的语句会下载`v1.7.0`，这是因为主版本 v1 版本可以省略且未指定版本，其效果与`@latest`一样。

因为`1.6.0`版本不是`v1`的最版本，所以要下载`v1.6.0`版本，则需要指定版本号：

```
go get github.com/test/test@1.6.0


```

如果要下载`v2.0.5`版本，下载路径为：

```
go get github.com/test/test/v2


```

创建一个新的模块
--------

创建模块的命令为`go mod init`，该命令后面的参数就是模块路径，在目录名为`hello`的空目录执行以下语句：

```
$ go mod init example.com/hello


```

此时在`hello`目录会生成一个`go.mod`文件：

```
module example.com/hello

go 1.20


```

创建`hello.go`文件，代码如下：

```
package hello

func Hello() string {
    return "Hello, world."
}


```

创建`hello_test.go`文件，代码如下：

```
package hello

import "testing"

func TestHello(t *testing.T) {
    want := "Hello, world."
    if got := Hello(); got != want {
        t.Errorf("Hello() = %q, want %q", got, want)
    }
}


```

运行`go test`命令：

```
$ go test
PASS
ok      example.com/hello       0.024s


```

这样，我们就已经创建了一个自己的模块。

依赖其他的模块
-------

`Go Modules`的作用就在于定义与规范了模块之间的版本依赖关系，通过这套机制，我们可以在自己的模块中导入其他的模块。

现在我们假设有一个其他开发者创建好的模块：`github.com/test/MyUtil`，该模块当前的版本为`v1.13`，我们使用`import`语句导出，并在`Hello()`方法中使用：

```
package hello

import "github.com/test/MyUtil"

func Hello() string {
    return MyUtil.Hello()
}


```

现在代码还不能运行，因为我们还没将该模块下载到本地，下载依赖的模块使用`go get`命令：

```
$ go get github.com/test/MyUtil
go: downloading github.com/test/MyUtil v1.1.3
go: added github.com/test/MyUtil v1.1.3


```

下载了依赖的模块之后，`go.mod`文件会记录当前模块的依赖项：

```
module example.com/hello

go 1.20

require github.com/test/MyUtil v1.1.3


```

另外，下载依赖模块时也会生成一个`go.sum`文件，其内容如下：

```
github.com/test/MyUtil v1.1.3 h1:A2x2RDcKl0rxZaOKk0Mn71HQfISoS/0Vex+UHjtou4o=
github.com/test/MyUtil v1.1.3/go.mod h1:8PqlR4GowL1as1JuS979xEp6TmJjrCIG+Bnjrdo3bfE=


```

`go.sum`文件在下载依赖时会发生变化，每个依赖模块可能会生成两行记录，一行表示依赖模块的全体文件的`SHA-256`哈希值，另一行则为依赖模块的`go.mod`文件的哈希值。

再次运行`go test`命令：

```
$ go test
PASS
ok      example.com/hello       0.024s


```

现在，我们就已经成功在自己的模块中调用其他模块的功能了。

升级依赖的模块
-------

我们依赖的模块也会更新升级，现在假设模块`MyUtil`增加了其他函数，并且版本升级到`v.1.12.10`，我们可以调用`go get`命令模块版本升级。

在模块路径后面可以指定义升级的版本：

```
 go get github.com/test/MyUtil@1.12.10


```

也可以在路径后直接跟上`@latest`升级最新的版本：

```
github.com/test/MyUtil@latest 


```

因为当前最新版本为`v1.12.10`，因此上面两种路径的效果是一样的：

```
$ go get github.com/test/MyUtil@latest  
go: downloading github.com/test/MyUtil v1.12.10
go: upgraded github.com/test/MyUtil v1.1.3 => v1.12.10


```

从`go.mod`文件可以看出依赖模块版本的变化：

```
module example.com/hello

go 1.20

require github.com/test/MyUtil v1.12.10


```

添加一个新的主版本
---------

当模块的主版本号发生变化时，比如从`v1`升级到`v2`，说明新版本做出之前不兼容的变更，我们可以通过在模块路径后面跟上版本来下载对应的版本：

```
 go get github.com/test/MyUtil/v2


```

此时`import`语句后面的路径也发生了变化：

```
package hello

import (
 "github.com/test/MyUtil"
 v2 "github.com/test/MyUtil/v2"
)

func Hello() string {
 return MyUtil.Hello()
}

func Hello2() string {
 return v2.HelloV2("%s", "Hello")
}


```

小结
--

`Go Module`是一种比较复杂的依赖解决方案，今天的这篇文章算是讲解了`Go Modules`的基本使用与一些基本概念，没有涉及到`Go Modules`的全部知识，在以后的文章，我们再详细展开。