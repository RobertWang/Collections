> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzkxNDE5NTQwMQ==&mid=2247490181&idx=3&sn=42c964ace45f3ab0f4acb0ac473cbba9&chksm=c1734c23f604c535720e8e452d1092666b492baea0aa1038d05da86f786027dd67736f0a8438&mpshare=1&scene=1&srcid=06193sbbrdL7Ai91OWX3vxy6&sharer_sharetime=1624092742674&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> 译者：诗书塞外  
> 原文：http://dwz.date/duAA

a. 你可以创建一个 Python 运行时的替代品，但是最后你会发现你重写了一遍 CPython。

下面是五种已有的方案，帮助你提高 Python 的性能。

![](https://mmbiz.qpic.cn/mmbiz_png/rO1ibUkmNGMnCKdEE7j8SNhLSGfznVVaxoficMujSfQlepp4MFjbV55vn1Be99ibIWggwvGVsuzzg9hXJguBhmWJQ/640?wx_fmt=png)

PyPy 使用适时编译来加速 Python，这项技术 Google 也在使用，Google 在 V8 引擎中使用它加速 Javascript。最近的版本 PyPy2.5 增加了一些提升性能的特性，其中有一项很受欢迎，它集成了 Numpy，Numpy 之前也一直被用来加速 Python 的运行。

使用 Python3 的代码需要对应地使用 PyPy3。PyPy 目前只支持到 Python3.2.5，对 Python3.3 的支持正在进行中。

**Pyston**

Pyston，由 Dropbox 资助，使用 LLVM 编译器架构来加速 Python，同样的它也使用了适时编译。相比于 PyPy，Pyston 还处于早期阶段，它只支持 Python 的部分特性。Pyston 把工作分成两个部分，一部分是语言的核心特性，另一部分是把性能提升到可接受的程度。Pyston 距离可以在生产环境使用还有一段距离

![](https://mmbiz.qpic.cn/mmbiz_jpg/rO1ibUkmNGMnCKdEE7j8SNhLSGfznVVaxupG1YiaNmm9sLEGLLk5KmLDZuErmx2fr20sBCw1LAIZiaQVpKsVDylaQ/640?wx_fmt=jpeg)

**Nuitka**

除了替换 Python 运行时，有些团队尝试将 Python 代码转换为能够在本地高效运行的其他语言的代码。其中著名的项目是 Nuitka-- 把 Python 代码转换为 C++ 代码 -- 虽然运行时还是依赖 Python 运行时。这样限制了它的可移植性，不过性能提升是可观的。长期规划中，Nuitka 还准备让 C 语言能够调用 Nuitka 编译的 Python 代码，这样性能提升将更加明显。

![](https://mmbiz.qpic.cn/mmbiz_png/rO1ibUkmNGMnCKdEE7j8SNhLSGfznVVaxGgITbEStOw42AN0fY3MhfJlqJKLQrljAM6hEibouynqsT1UQzucibo2A/640?wx_fmt=png)

**Cython**

Cython（Python 的 C 语言扩展）是 Python 的超集，它能把 Python 代码编译成 C 代码，并与 C 和 C++ 进行交互。它可以作为 Python 项目的扩展使用（重新性能要求高的部分），或者单独使用，不涉及传统的 Python 代码。缺点是你写的不是 Python，所以需要手动迁移，缺乏可移植性。

![](https://mmbiz.qpic.cn/mmbiz_jpg/rO1ibUkmNGMnCKdEE7j8SNhLSGfznVVaxoDpYRPuNfzjErFNhWDbP61Ly18CddicNj9Z7icHjzSxeJx4azDHOEGFw/640?wx_fmt=jpeg)

据说，Cython 提供了一些特性来让代码更高效，比如变量类型化，这本质上是 C 要求的。一些科学计算的包，如 scikit-learn 依赖 Cython 的一些特性来保持操作简洁快速。

**Numba**

Numba 结合了上面几个项目的想法。学习了 Cython，Numba 也采用了部分加速的策略，只加速 CPU 密集型的任务；同时它又学习了 PyPy 和 Pyston，通过 LLVM 运行 Python。你可以用一个装饰器指定你要用 Numba 编译的函数，Numba 继承 Numpy 来加速函数的执行，Numba 不做适时编译，它的代码是预先编译的。

![](https://mmbiz.qpic.cn/mmbiz_png/rO1ibUkmNGMnCKdEE7j8SNhLSGfznVVax4ufF55NUa2yUKR0fmniclbg5Vg148ib7L4CnJETvic8fwQbXiaN4via3obw/640?wx_fmt=png)

Python 之父说：大部分觉得 Python 慢的应用都是没有正确地使用 Python。对于 CPU 密集型的任务有多种方法来提升性能 -- 使用 Numpy 来做计算，调用外部 C 代码，以及尽量避免 GIL 锁。由于 GIL 锁目前还无法被替代，所以有很多项目开始尝试一些短期可行的替代方案，当然这些方案也可能转变为长期的可选项。