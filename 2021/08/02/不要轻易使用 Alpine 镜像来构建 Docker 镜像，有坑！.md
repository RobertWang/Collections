> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1627905976&ver=3228&signature=Xg-nPOgs0q5dntdqmRnutYdyOilUkz4RT21Kx3wUFslbNqSObk-VM6kKmJCZT8pALNsXZEg6lsnZUTsG-BEdBqhzZKIBD-f4eGYCL-qvvr3w90nzYhLFHn08JA7Fnkb6&new=1)

第三部分将会探讨适用于大多数语言和框架的通用精简策略，例如使用常见的基础镜像、提取可执行文件和减小每一层的体积。同时还会介绍一些更加奇特或激进的工具，例如 `Bazel`，`Distroless`，`DockerSlim` 和 `UPX`，虽然这些工具在某些特定场景下能带来奇效，但大多情况下会起到反作用。

本文介绍第二部分。

1. Go 语言镜像精简
------------

`Go` 语言程序编译时会将所有必须的依赖编译到二进制文件中，但也不能完全肯定它使用的是静态链接，因为 `Go` 的某些包是依赖系统标准库的，例如使用到 DNS 解析的包。只要代码中导入了这些包，编译的二进制文件就需要调用到某些系统库，为了这个需求，Go 实现了一种机制叫 `cgo`，以允许 Go 调用 C 代码，这样编译好的二进制文件就可以调用系统库。

也就是说，如果 Go 程序使用了 `net` 包，就会生成一个动态的二进制文件，如果想让镜像能够正常工作，必须将需要的库文件复制到镜像中，或者直接使用 `busybox:glibc` 镜像。

当然，你也可以禁止 `cgo`，这样 Go 就不会使用系统库，使用内置的实现来替代系统库（例如使用内置的 DNS 解析器），这种情况下生成的二进制文件就是静态的。可以通过设置环境变量 `CGO_ENABLED=0` 来禁用 cgo，例如：

```
FROM golang
COPY whatsmyip.go .
ENV CGO_ENABLED=0
RUN go build whatsmyip.go

FROM scratch
COPY --from=0 /go/whatsmyip .
CMD ["./whatsmyip"]


```

由于编译生成的是静态二进制文件，因此可以直接跑在 `scratch` 镜像中 🎉

当然，也可以不用完全禁用 `cgo`，可以通过 `-tags` 参数指定需要使用的内建库，例如 `-tags netgo` 就表示使用内建的 net 包，不依赖系统库：

```
$ go build -tags netgo whatsmyip.go


```

这样指定之后，如果导入的其他包都没有用到系统库，那么编译得到的就是静态二进制文件。也就是说，只要还有一个包用到了系统库，都会开启 `cgo`，最后得到的就是动态二进制文件。要想一劳永逸，还是设置环境变量 `CGO_ENABLED=0` 吧。

2. Alpine 镜像探秘
--------------

上篇文章已经对 `Alpine` 镜像作了简要的介绍，并保证会在后面的文章中花很大的篇幅来讨论 `Alpine` 镜像，现在时候到了！

`Alpine` 是众多 Linux 发行版中的一员，和 `CentOS`、`Ubuntu`、`Archlinux` 之类一样，只是一个发行版的名字，号称小巧安全，有自己的包管理工具 `apk`。

与 CentOS 和 Ubuntu 不同，Alpine 并没有像 `Red Hat` 或 `Canonical` 之类的大公司为其提供维护支持，软件包的数量也比这些发行版少很多（如果只看开箱即用的默认软件仓库，Alpine 只有 `10000` 个软件包，而 Ubuntu、Debian 和 Fedora 的软件包数量均大于 `50000`。）

容器崛起之前，`Alpine` 还是个无名之辈，可能是因为大家并不是很关心操作系统本身的大小，毕竟大家只关心业务数据和文档，程序、库文件和系统本身的大小通常可以忽略不计。

容器技术席卷整个软件产业之后，大家都注意到了一个问题，那就是容器的镜像太大了，浪费磁盘空间，拉取镜像的时间也很长。于是，人们开始寻求适用于容器的更小的镜像。对于那些耳熟能详的发行版（例如 Ubuntu、Debian、Fedora）来说，只能通过删除某些工具（例如 `ifconfig` 和 `netstat`）将镜像体积控制在 `100M` 以下。而对于 Alpine 而言，什么都不用删除，镜像大小也就只有 `5M` 而已。

Alpine 镜像的另一个优势是包管理工具的执行速度非常快，安装软件体验非常顺滑。诚然，在传统的虚拟机上不需要太关心软件包的安装速度，同一个包只需要装一次即可，无需不停重复安装。容器就不一样了，你可能会定期构建新镜像，也可能会在运行的容器中临时安装某些调试工具，如果软件包的安装速度很慢，会很快消磨掉我们的耐心。

为了更直观，我们来做个简单的对比测试，看看不同的发行版安装 `tcpdump` 需要多长时间，测试命令如下：

```
🐳 → time docker run <image> <packagemanager> install tcpdump


```

测试结果如下：

```
Base image           Size      Time to install tcpdump
---------------------------------------------------------
alpine:3.11          5.6 MB      1-2s
archlinux:20200106   409 MB      7-9s
centos:8             237 MB      5-6s
debian:10            114 MB      5-7s
fedora:31            194 MB    35-60s
ubuntu:18.04          64 MB      6-8s


```

如果你想了解更多关于 Alpine 的内幕，可以看看 Natanel Copa 的演讲 [2]。

好吧，既然 Alpine 这么棒，为什么不用它作为所有镜像的基础镜像呢？别急，先一步一步来，为了趟平所有的坑，需要分两种情况来考虑：

1.  使用 Alpine 作为第二构建阶段（`run` 阶段）的基础镜像
    
2.  使用 ALpine 作为所有构建阶段（`run` 阶段和 `build` 阶段）的基础镜像
    

### run 阶段使用 Alpine

带着激动的心情，将 Alpine 镜像加入了 Dockerfile：

```
FROM gcc AS mybuildstage
COPY hello.c .
RUN gcc -o hello hello.c

FROM alpine
COPY --from=mybuildstage hello .
CMD ["./hello"]


```

第一个坑来了，启动容器出现了错误：

```
standard_init_linux.go:211: exec user process caused "no such file or directory"


```

这个报错在上篇文章已经见识过了，上篇文章的场景是使用 `scratch` 镜像作为 `C` 语言程序的基础镜像，错误的原因是 `scratch` 镜像中缺少动态库文件。可是为什么使用 Alpine 镜像也有报错，难道它也缺少动态库文件？

也不完全是，Alpine 使用的也是动态库，毕竟它的设计目标之一就是占用更少的空间。但 Alpine 使用的标准库与大多数发行版不同，它使用的是 `musl libc`，这个库相比于 `glibc` 更小、更简单、更安全，但是与大家常用的标准库 `glibc` 并不兼容。

你可能又要问了：『既然 `musl libc` 更小、更简单，还特么更安全，为啥其他发行版还在用 `glibc`？』

mmm。。。因为 `glibc` 有很多额外的扩展，并且很多程序都用到了这些扩展，而 `musl libc` 是不包含这些扩展的。详情可以参考 musl 的文档 [3]。

也就是说，如果想让程序跑在 Alpine 镜像中，必须在编译时使用 `musl libc` 作为动态库。

### 所有阶段使用 Alpine

为了生成一个与 `musl libc` 链接的二进制文件，有两条路：

*   某些官方镜像提供了 Alpine 版本，可以直接拿来用。
    
*   还有些官方镜像没有提供 Alpine 版本，我们需要自己构建。
    

golang 镜像就属于第一种情况，`golang:alpine` 提供了基于 Alpine 构建的 `Go` 工具链。

构建 Go 程序可以使用下面的 `Dockerfile`：

```
FROM golang:alpine
COPY hello.go .
RUN go build hello.go

FROM alpine
COPY --from=0 /go/hello .
CMD ["./hello"]


```

生成的镜像大小为 7.5M，对于一个只打印 『hello world』的程序来说确实有点大了，但我们可以换个角度：

*   即使程序很复杂，生成的镜像也不会很大。
    
*   包含了很多有用的调试工具。
    
*   即使运行时缺少某些特殊的调试工具，也可以迅速安装。
    

Go 语言搞定了，C 语言呢？并没有 `gcc:alpine` 这样的镜像啊。只能以 Alpine 镜像作为基础镜像，自己安装 C 编译器了，Dockerfile 如下：

```
FROM alpine
RUN apk add build-base
COPY hello.c .
RUN gcc -o hello hello.c

FROM alpine
COPY --from=0 hello .
CMD ["./hello"]


```

> 必须安装 `build-base`，如果安装 `gcc`，就只有编译器，没有标准库。build-base 相当于 Ubuntu 的 `build-essentials`，引入了编译器、标准库和 make 之类的工具。

最后来对比一下不同构建方法得到的 『hello world』镜像大小：

*   使用基础镜像 `golang` 构建：805MB
    
*   多阶段构建，build 阶段使用基础镜像 `golang`，run 阶段使用基础镜像 `ubuntu`：66.2MB
    
*   多阶段构建，build 阶段使用基础镜像 `golang:alpine`，run 阶段使用基础镜像 `alpine`：7.6MB
    
*   多阶段构建，build 阶段使用基础镜像 `golang`，run 阶段使用基础镜像 `scratch`：2MB
    

最终镜像体积减少了 `99.75%`，相当惊人了。再来看一个更实际的例子，上一节提到的使用 `net` 的程序，最终的镜像大小对比：

*   使用基础镜像 `golang` 构建：810MB
    
*   多阶段构建，build 阶段使用基础镜像 `golang`，run 阶段使用基础镜像 `ubuntu`：71.2MB
    
*   多阶段构建，build 阶段使用基础镜像 `golang:alpine`，run 阶段使用基础镜像 `alpine`：12.6MB
    
*   多阶段构建，build 阶段使用基础镜像 `golang`，run 阶段使用基础镜像 `busybox:glibc`：12.2MB
    
*   多阶段构建，build 阶段使用基础镜像 `golang` 并使用参数 CGO_ENABLED=0，run 阶段使用基础镜像 `ubuntu`：7MB
    

镜像体积仍然减少了 `99%`。

3. Java 语言镜像精简
--------------

`Java` 属于编译型语言，但运行时还是要跑在 `JVM` 中。那么对于 Java 语言来说，该如何使用多阶段构建呢？

### 静态还是动态？

从概念上来看，Java 使用的是动态链接，因为 Java 代码需要调用 `JVM` 提供的 `Java API`，这些 API 的代码都在可执行文件之外，通常是 `JAR` 文件或 `WAR` 文件。

然而这些 Java 库并不是完全独立于系统库的，某些 Java 函数最终还是会调用系统库，例如打开文件时需要调用 `open()`, `fopen()` 或它们的变体，因此 `JVM` 本身可能会与系统库动态链接。

这就意味着理论上可以使用任意的 `JVM` 来运行 Java 程序，系统标准库是 `musl libc` 还是 `glibc` 都无所谓。因此，也就可以使用任意带有 `JVM` 的基础镜像来构建 Java 程序，也可以使用任意带有 `JVM` 的镜像作为运行 Java 程序的基础镜像。

### 类文件格式

Java 类文件（Java 编译器生成的字节码）的格式会随着版本而变化，且大部分变化都是 `Java API` 的变化。还有一部分更改与 Java 语言本身有关，例如 `Java 5` 中添加了泛型，这种变化就可能会导致类文件格式的变化，从而破坏与旧版本的兼容性。

所以默认情况下，使用给定版本的 Java 编译器编译的类不能与更早版本的 JVM 兼容，但可以指定编译器的 `-target` （Java 8 及其以下版本）参数或者 `--release` （Java 9 及其以上版本）参数来使用较旧的类文件格式。`--release` 参数还可以指定类文件的路径，以确保程序运行在指定的 JVM 版本中（例如 Java 11），不会意外调用 Java 12 的 API。

### JDK vs JRE

如果你对大多数平台上的 Java 打包方式很熟悉，那你应该知道 `JDK` 和 `JRE`。

`JRE` 即 Java 运行时环境（`Java Runtime Environment`），包含了运行 Java 程序所需要的环境，即 `JVM`。

`JDK` 即 Java 开发工具包（`Java Development Kit`），既包含了 JRE，也包含了开发 Java 程序所需的工具，即 Java 编译器。

大多数 Java 镜像都提供了 JDK 和 JRE 两种标签，因此可以在多阶段构建的 `build` 阶段使用 `JDK` 作为基础镜像，`run` 阶段使用 `JRE` 作为基础镜像。

### Java vs OpenJDK

推荐使用 `openjdk`，因为开源啊，更新勤快啊~~

也可以使用 amazoncorretto[4]，这是 `Amazon` fork OpenJDK 后打了补丁的版本，号称企业级。

### 开始构建

说了那么多，到底该用哪个镜像呢？这里给出几个参考：

*   **openjdk:8-jre-alpine**（85MB）
    
*   **openjdk:11-jre**（267MB）或者 **openjdk:11-jre-slim**（204MB）
    
*   **openjdk:14-alpine**（338MB）
    

如果你想要更直观的数据，可以看我的例子，还是搬出屡试不爽的 『hello world』，只不过这次是 Java 版本：

```
class hello {
  public static void main(String [] args) {
    System.out.println("Hello, world!");
  }
}


```

不同构建方法得到的镜像大小：

*   使用基础镜像 `java` 构建：643MB
    
*   使用基础镜像 `openjdk` 构建：490MB
    
*   多阶段构建，build 阶段使用基础镜像 `openjdk`，run 阶段使用基础镜像 `openjdk:jre`：479MB
    
*   使用基础镜像 `amazoncorretto` 构建：390MB
    
*   多阶段构建，build 阶段使用基础镜像 `openjdk:11`，run 阶段使用基础镜像 `openjdk:11-jre`：267MB
    
*   多阶段构建，build 阶段使用基础镜像 `openjdk:8`，run 阶段使用基础镜像 `openjdk:8-jre-alpine`：85MB
    

所有的 Dockerfile 都可以在这个仓库 [5] 找到。

4. 解释型语言镜像精简
------------

对于诸如 `Node`、`Python`、`Rust` 之类的解释型语言来说，情况就比较复杂一点了。先来看看 Alpine 镜像。

### Alpine 镜像

对于解释型语言来说，如果程序仅用到了标准库或者依赖项和程序本身使用的是同一种语言，且无需调用 C 库和外部依赖，那么使用 `Alpine` 作为基础镜像一般是没有啥问题的。一旦你的程序需要调用外部依赖，情况就复杂了，想继续使用 Alpine 镜像，就得安装这些依赖。根据难度可以划分为三个等级：

*   **简单**：依赖库有针对 Alpine 的安装说明，一般会说明需要安装哪些软件包以及如何建立依赖关系。但这种情况非常罕见，原因前面也提到了，Alpine 的软件包数量比大多数流行的发行版要少得多。
    
*   **中等**：依赖库没有针对 Alpine 的安装说明，但有针对别的发行版的安装说明。我们可以通过对比找到与别的发行版的软件包相匹配的 Alpine 软件包（假如有的话）。
    
*   **困难**：依赖库没有针对 Alpine 的安装说明，但有针对别的发行版的安装说明，但是 Alpine 也没有与之对应的软件包。这种情况就必须从源码开始构建！
    

最后一种情况最不推荐使用 Alpine 作为基础镜像，不但不能减小体积，可能还会适得其反，因为你需要安装编译器、依赖库、头文件等等。。。更重要的是，构建时间会很长，效率低下。如果非要考虑多阶段构建，就更复杂了，**你得搞清楚如何将所有的依赖编译成二进制文件，想想就头大**。因此一般不推荐在解释型语言中使用多阶段构建。

有一种特殊情况会同时遇到 Alpine 的绝大多数问题：**将 Python 用于数据科学**。`numpy` 和 `pandas` 之类的包都被预编译成了 wheel[6]，`wheel` 是 Python 新的打包格式，被编译成了二进制，用于替代 Python 传统的 `egg` 文件，可以通过 `pip` 直接安装。但这些 wheel 都绑定了特定的 C 库，这就意味着在大多数使用 `glibc` 的镜像中都可以正常安装，但 Alpine 镜像就不行，原因你懂得，前面已经说过了。如果非要在 Alpine 中安装，你需要安装很多依赖，重头构建，耗时又费力，有一篇文章专门解释了这个问题：使用 Alpine 构建 Pyhton 镜像会将构建速度拖慢 50 倍！[7]。

既然 Alpine 镜像这么坑，那么是不是只要是 Python 写的程序就不推荐使用 Alpine 镜像来构建呢？也不能完全这么肯定，至少 Python 用于数据科学时不推荐使用 Alpine，其他情况还是要具体情况具体分析，如果有可能，还是可以试一试 Alpine 的。

### :slim 镜像

如果实在不想折腾，可以选择一个折衷的镜像 `xxx:slim`。slim 镜像一般都基于 `Debian` 和 `glibc`，删除了许多非必需的软件包，优化了体积。如果构建过程中需要编译器，那么 slim 镜像不适合，除此之外大多数情况下还是可以使用 slim 作为基础镜像的。

下面是主流的解释型语言的 Alpine 镜像和 slim 镜像大小对比：

```
Image            Size
---------------------------
node             939 MB
node:alpine      113 MB
node:slim        163 MB
python           932 MB
python:alpine    110 MB
python:slim      193 MB
ruby             842 MB
ruby:alpine       54 MB
ruby:slim        149 MB


```

再来举个特殊情况的例子，同时安装 `matplotlib`，`numpy` 和 `pandas`，不同的基础镜像构建的镜像大小如下：

```
Image and technique         Size
--------------------------------------
python                      1.26 GB
python:slim                  407 MB
python:alpine                523 MB
python:alpine multi-stage    517 MB


```

可以看到这种情况下使用 Alpine 并没有任何帮助，即使使用多阶段构建也无济于事。

但也不能全盘否定 Alpine，比如下面这种情况：包含大量依赖的 `Django` 应用。

```
Image and technique         Size
--------------------------------------
python                      1.23 GB
python:alpine                636 MB
python:alpine multi-stage    391 MB


```

最后来总结一下：**到底使用哪个基础镜像并不能盖棺定论，有时使用 Alpine 效果更好，有时反而使用 slim 效果更好，如果你对镜像体积有着极致的追求，可以这两种镜像都尝试一下。相信随着时间的推移，我们就会积累足够的经验，知道哪种情况该用 Alpine，哪种情况该用 slim，不用再一个一个尝试。**

5. Rust 语言镜像精简
--------------

`Rust` 是最初由 `Mozilla` 设计的现代编程语言，并且在 `Web` 和基础架构领域中越来越受欢迎。Rust 编译的二进制文件动态链接到 C 库，可以正常运行于 `Ubuntu`、`Debian` 和 `Fedora` 之类的镜像中，但不能运行于 `busybox:glibc` 中。因为 Rust 二进制需要调用 `libdl` 库，`busybox:glibc` 中不包含该库。

还有一个 `rust:alpine` 镜像，Rust 编译的二进制也可以正常运行其中。

如果考虑编译成静态链接，可以参考 Rust 官方文档 [8]。在 Linux 上需要构建一个特殊版本的 Rust 编译器，构建的依赖库就是 `musl libc`，你没有看错，就是 Alpine 中的那个 `musl libc`。如果你想获得更小的镜像，请按照文档中的说明进行操作，最后将生成的二进制文件扔进 `scratch` 镜像中就好了。

6. 总结
-----

本系列文章的前两部分介绍了优化 Docker 镜像体积的常用方法，以及如何针对不同类型的语言运用这些方法。最后一部分将会介绍如何在减少镜像体积的同时，还能减少 I/O 和内存使用量，同时还会介绍一些虽然与容器无关但对优化镜像有帮助的技术。

### 脚注

[1]

两个奇技淫巧，将 Docker 镜像体积减小 99%: https://fuckcloudnative.io/posts/docker-images-part1-reducing-image-size/

[2]

Natanel Copa 的演讲: https://dockercon.docker.com/watch/6nK1TVGjuTpFfnZNKEjCEr

[3]

musl 的文档: https://wiki.musl-libc.org/functional-differences-from-glibc.html

[4]

amazoncorretto: https://hub.docker.com/_/amazoncorretto

[5]

这个仓库: https://github.com/jpetazzo/minimage

[6]

wheel: https://pythonwheels.com/

[7]

使用 Alpine 构建 Pyhton 镜像会将构建速度拖慢 50 倍！: https://pythonspeed.com/articles/alpine-docker-python/

[8]

Rust 官方文档: https://doc.rust-lang.org/1.9.0/book/advanced-linking.html#static-linking