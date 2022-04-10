> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [cloud.tencent.com](https://cloud.tencent.com/developer/news/600722)

![](https://ask.qcloudimg.com/http-save/yehe-8507090/ubqaov34je.jpeg?imageView2/2/w/1620)

当你为 Docker 镜像选择基础镜像时，Alpine Linux 可能被推荐。有人告诉你，用 Alpine 将使你的镜像更小，并能加快你的 builds。如果你正在用 Go，这无疑是个合理的建议。

但如果你使用 Python，Alpine Linux 会经常：

1.  让你的构建更慢
2.  让你的镜像更大
3.  浪费你的时间
4.  偶尔引入一些令人费解的运行时 Bug

让我们看看为什么人们推荐使用 Alpine，以及为什么不应该在 Python 应用程序中使用它。

为什么人们推荐使用 Alpine
----------------

假设我们需要安装 gcc 作为镜像构建的一部分，并且我们想看看 Alpine Linux 在构建时间和镜像大小方面与 Ubuntu 18.04 有何不同。

首先，我将拉取两个镜像，并检查他们的大小：

如你所见，Alpine 的基础镜像要小得多。

接下来，我们将尝试在它们两个中安装 gcc。首先，在 Ubuntu 中：

**注意：**在我们讨论的主题之外，本文中的 Dockerfile 并不是最佳实践的示例，因为增加的复杂性会掩盖本文的主要观点。因此，如果你打算用 Docker 在生产环境中运行你的 Python 应用程序，这里有两种方法可以应用最佳实践：

如果你想 DIY：[一个详细的清单、例子和参考资料](https://pythonspeed.com/products/productionchecklist/)；

如果你想要尽快拥有一个基本够用的设置：[一个模板和为你实现的最佳实践](https://pythonspeed.com/products/pythoncontainer/)。

然后，我们可以构建并记录时间：

现在，我们编制一个类似的 Alpine Dockerfile：

同样地，构建镜像并检查大小：

就像我们所说的那样，Alpine 镜像构建速度更快，体积更小：15 秒而不是 30 秒，镜像大小是 105MB 而不是 150MB。这很好！

但是当我们打包 Python 应用程序时，情况就开始变得糟糕了。

让我们构建一个 Python 镜像
-----------------

我们希望打包一个使用了 panda 和 matplotlib 的 Python 应用程序。因此，一种选择是使用基于 Debian 的官方 Python 镜像（我提前拉取的），和以下这个 Dockerfile：

然后，我们构建它：

结果镜像大小为 363MB。

用 Alpine 会获得更好的结果吗？让我们试一试。

然后，我们构建它：

发生了什么？

标准的 PyPI wheel 在 Alpine 上无效
---------------------------

如果你仔细查看上面基于 Debian 的构建，就会看到它正在下载 matplotlib-3.1.2-cp38-cp38-manylinux1_x86_64.whl。这是一个预编译的二进制 wheel。与此相反，Alpine 会下载源代码（matplotlib-3.1.2.tar.gz），因为标准的 Linux wheel 在 Alpine Linux 上无效。

为什么？大多数 Linux 发行版使用标准 C 库的 GNU 版本（glibc），Python 以及几乎所有的 C 程序都需要它。但是 Alpine Linux 使用 musl，这些二进制 wheel 是针对 glibc 编译的，因此 Alpine 禁用了 Linux wheel 支持。

现在，大多数 Python 包都包含了 PyPI 上的二进制 wheel，这大大缩短了安装时间。**但是，如果你使用的是 Alpine Linux，那么你就需要编译你所使用的每个 Python 包中的所有 C 代码。**

这也意味着你需要自己找出每个系统库依赖项。在这种情况下，为了找出依赖项，我做了一些研究，最后得到下面这个经过更新的 Dockerfile：

然后我们构建它，它需要……25 分钟 57 秒！得到的镜像是 851MB。 下面是两个基本镜像的对比：

![](https://ask.qcloudimg.com/http-save/yehe-8507090/farg35ev3e.jpeg)

Alpine 的构建速度要慢得多，镜像要大得多，而且我不得不做了很多研究。

你不能解决这些问题吗？
-----------

### 构建时间

为了缩短构建时间，Alpine Edge（最终将成为下一个稳定版本）会包含 matplotlib 和 panda。而且安装系统包非常快。但是，到 2020 年 1 月为止，当前的稳定版本还不包括这些流行的包。

然而，即使它们可用了，系统包也几乎总是滞后于 PyPI 上的包，Alpine 也不太可能打包 PyPI 上的所有东西。据我所知，在实践中，大多数 Python 团队并没有将系统包用于 Python 依赖项，而是依赖于 PyPI 或 Conda Forge。

### 镜像大小

一些读者指出，你可以删除最初安装的包，或者添加不缓存包下载的选项，或者使用[多阶段构建](https://pythonspeed.com/articles/smaller-python-docker-images/)。一位读者尝试生成了[一个 470MB 的镜像](https://lobste.rs/s/ahzafb/alpine_makes_python_docker_builds_50x#c_5qzhdx)。

是的，你可以得到一个与基于 slim 的镜像大致相当的镜像，但是 Alpine Linux 的全部动机是更小的镜像和更快的构建。如果工作做够了，你可能会得到一个更小的镜像，但是你仍然要忍受长达 1500 秒的构建时间，当你使用 python:3.8-slim 镜像时，构建时间只有 30 秒。

但是等等，还有！

Alpine Linux 会导致意料之外的运行时 Bug
----------------------------

虽然理论上，Alpine 使用的 musl C 库与其他 Linux 发行版使用的 glibc [基本兼容](https://wiki.musl-libc.org/functional-differences-from-glibc.html)，但在实践中，这种差异可能会导致问题。当问题确实发生时，可能会很奇怪且出乎意料。

下面是一些例子：

1.  Alpine 线程的默认堆栈大小更小，这可能[导致 Python 崩溃](https://bugs.python.org/issue32307)。
2.  Alpine 的一位用户发现，由于 musl 分配内存的方式与 glibc 不同，他们的 Python 应用程序要[慢很多](https://superuser.com/questions/1219609/why-is-the-alpine-docker-image-over-50-slower-than-the-ubuntu-image)。
3.  在使用 WeWork 工作空间的 WiFi 时，我曾经无法在 minikube（虚拟机中的 Kubernetes）上运行的 Alpine 镜像中查找 DNS。原因是 WeWork 糟糕的 DNS 设置、Kubernetes 和 minikube 实现 DNS 的方式，以及 musl 对这种边缘情况的处理与 glibc 的方式不同。musl 没有错（它符合 RFC），但是我不得不浪费时间找出问题所在，然后切换到基于 glibc 的镜像。
4.  另一个用户发现了[时间格式和解析](https://github.com/iron-io/dockers/issues/42#issuecomment-290763088)的问题。

大多数或者说所有这些问题可能都已经得到解决，但毫无疑问，还有更多的问题有待发现。这种出人意料的破坏是又一件需要担心的事情。

不要将 Alpine Linux 用于 Python 镜像
-----------------------------

除非你想要更长的构建时间、更大的镜像、更多的工作，以及潜在的隐藏 Bug，否则你应该避免使用 Alpine Linux 作为基础镜像。

关于应该使用哪些镜像的建议，请参阅我的文章 “[选择一个好的基础镜像](https://pythonspeed.com/articles/base-image-python-docker-images/)”。

英文原文：

[Using Alpine can make Python Docker builds 50× slower](https://pythonspeed.com/articles/alpine-docker-python)