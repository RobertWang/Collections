> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/HB-BXBP4ss7Kx8lQyj6vMw)

作者 | Serdar Yegulalp

策划 | 云昭

Mojo 很狂！它的目标非常有野心：“**与 Python 一样易于使用，但与 Rust 一样强大和快速**。”

新推出的 Mojo 语言，被宣传为多个领域中最好的：Python 的易用性和清晰的语法，以及 Rust 的速度和内存安全。这些多少有些夸大其词。由于 Mojo 仍处于开发的早期阶段，用户还需要一段时间才能亲眼看到这种语言是如何达到他们的要求的。

Mojo 的创建者，一家名为 Modular 的公司，提供了一个早期的在线运行环境：一个 Jupyter Notebook 环境，用户可以在这里运行 Mojo 代码并了解该语言的功能和行为。

因为 Mojo 还没有作为最终用户下载，所以我们首先关注的是 Mojo 作为一种语言是什么样子的。我们将研究它与 Python 的相似之处，它的不同之处，以及它能为熟悉 Python 或其他语言的程序员提供什么。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQoOJQQprmIGVMDdTdhZl4ib6SlG4gLUichgR8iaYjPK3ZttOJch09cCxSSgq7E1eLAlSa4af2VQFice2g/640?wx_fmt=png)

**Mojo：Python 的超集**

  



----------------------------

Mojo 可以被描述为 Python 的 “**超集**”。用 Python 编写的程序是有效的 Mojo 程序，尽管有些 Python 行为尚未实现。目前在 Python 中找不到的 Python 行为的一些示例包括，函数的关键字参数、global 关键字以及 list 和 dict 理解。也可以使用实际的 Python 运行时来处理现有的 Python 模块，尽管这会带来性能成本。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQqJtuicaS3cibv5mtMnjB8Ihg1iaHGHYBeSibMg1d1ZmMxoCMNbicK21Kcun23owq6o5cj6xpiaJiaiaGRZvg/640?wx_fmt=png)

当 Mojo 引入新语法时，它用于系统级编程功能，主要是手动内存处理。换句话说，可以为随意的用例编写 Python 代码（或几乎完全类似的代码），然后将 Mojo 用于更高级、性能密集型的编程场景。在这两种情况下，都可以利用现有的 Python 库，但性能成本更高。

Mojo 与 Python 的另一大区别是，Mojo 不像 Python 那样通过运行时进行解释。Mojo 是提前编译的，使用 LLVM 工具链来加工本地代码。为此，最好的性能来自于使用 Mojo 特有的功能。Python 功能很可能是以模仿 Python 的动态行为为代价的，这些动态行为本质上很慢，或者仅仅通过使用 Python 运行时来实现。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQoOJQQprmIGVMDdTdhZl4ib6ojD8pHUQOsKpMfyhXp67maIRcFLyAdv0lgYYV3OI2g2zwpRHL6ljYg/640?wx_fmt=png)

**Mojo vs. Python 语法**


--------------------------

Mojo 的许多母语特征有两个作用。它们要么是 Python 中根本没有的全新功能，要么是 Python 功能的扩展，使其更具性能，尽管 Python 的**动态性较低**。

Mojo 的许多母语特征有两个作用。它们要么是 Python 中根本没有的全新功能，要么是 Python 功能的扩展，使其更具性能（尽管 Python 的动态性较低）。

例如，在 Python 中，没有办法在运行时正式声明变量引用不可变，尽管类型提示和其他机制可以在编辑时模仿这一点。在 Mojo 中，则可以使用关键字 let 和 var 来声明 Mojo 特定的变量，这与**在 Rust 中使用的方法非常相似**。let 关键字表示变量是不可变的；var 表示它是可变的。这些限制是在编译时强制执行的，所以试图变异不可变引用的程序甚至不会编译。

![](https://mmbiz.qpic.cn/mmbiz_jpg/MOwlO0INfQqJtuicaS3cibv5mtMnjB8IhgGMpsHicnwnKX4MrLiboGkwtQdIgYwccOZkUgJzrL0oPPkiap6l23Ut70A/640?wx_fmt=jpeg)

Mojo 的语法与 Python 非常相似，但提供了新的关键字来启用 Mojo 特定的功能，如可变行为。

Mojo 还有自己的 struct 关键字，与 Python 的 class 形成对比。类只是 Python 类，具有所期望的所有动态行为。不过，struct 类型更像它们的 C/C++ 和 Rust 对应类型，在编译时确定了固定的布局，但针对机器本机速度进行了优化。另一个旨在区分 Mojo 的行为和 Python 的行为的 Mojo 关键字是 fn。如果你使用 def 来定义一个函数，那么就会得到了一个 Python 函数，以及与这些函数相关联的所有动态。fn 关键字也定义了一个函数，但定义为 Mojo 函数。这意味着参数在默认情况下是不可变的，并且必须显式类型化，并且必须声明所有局部变量（除其他外）。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQoOJQQprmIGVMDdTdhZl4ib6O3T8iahkmTbcEjAuV6rXuzmq4SSibXZ8Tqk3TfvrIP54wMx2PtEYzraw/640?wx_fmt=png)

**Modular Playground**

  



-------------------------------

如果想知道用 Mojo 有多爽，现在还需要**取号排队**。Modular 通过 Modular Playground 提供了对 Mojo 的早期访问，这是一个基于网络的 Jupyter Notebook 环境，运行在 Modular 的服务器上。目前，Mojo 还没有可在自己的系统上下载的运行时。从好的方面来说，这意味着你可以通过任何带有网络浏览器的计算机运行 Mojo。

Mojo 在线环境附带了一些 notebook 示例，以及关于在某些任务中使用 Mojo 的详细内联注释。其中一个例子是一个常见的程序员演示，绘制了 Mandelbrot 集算法。乍一看，代码与 Python 非常相似。即使是新的特定于 Mojo 的关键字也能很好地与现有的 Python 语法集成，因此可以仔细查看代码并大致了解发生了什么。

![](https://mmbiz.qpic.cn/mmbiz_jpg/MOwlO0INfQqJtuicaS3cibv5mtMnjB8Ihglew3DBOcxFTXDHwN63iclde4lYt3WePZLiaZHxWoxZhJPibbXM2sHpURA/640?wx_fmt=jpeg)

Mojo 游乐场在行动，运行 Mandelbrot 情节演示。用于生成此演示的代码是本地 Mojo 代码和 Python 库的混合，通过 Mojo 的 Python 运行时接口调用。

notebook 演示还举例说明了如何通过并行、向量化和 “平铺”（增加操作的缓存位置）来加速 Mojo 代码。其中一个演示是 128x128 矩阵乘法演示，通过简单地按原样运行而无需特别修改，它的速度据称是 Python 的 17 倍（使用 Mojo 游乐场中的 Python 运行时）。

Mojo 通过添加类型注释增加了 **1866x** 的加速，通过添加矢量化操作增加了 **8500x** 的加速和通过添加并行化增加了 **15000x** 的加速。

同样，验证这些声明的最佳方法是让 Mojo 在本地可用，但值得在同一代码中同时试验 Python 运行时和 Mojo 编译器，看看会发生什么。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQoOJQQprmIGVMDdTdhZl4ib6hstzqmSo5zAnrjSfHCLrHJscT0p587IuvAq9TqRlESXsnuhOSenJVQ/640?wx_fmt=png)

**Mojo 能取代 Python 吗？**


--------------------------

Mojo 的第一次公开演讲就证明了它是数据科学和机器学习的一种语言。这两个主题构成了 Python 现代用例的很大一部分，这并不是因为 Python 本身很快，而是因为它为难以使用的快速事物提供了一个方便的编程接口。

Mojo 显然是为了提供该用例的默认快速版本，在该版本中，不必通过外部库来快速实现。Mojo 的目标不是 Python 更广泛的用例集：web 后端、流程自动化等等，至少在早期不是这样。这可能会在 Mojo 更完整、拥有更好的第三方库时出现，但这显然不是首要任务。

即使默认情况下 Mojo 更快，也**很难取代** Python 在机器学习和数据科学中的地位。Python 的用户社区、现有的软件文化和便利性都使其成为这些领域的支柱。Mojo 必须做的不仅仅是快速地取代 Python 来完成这项工作。尽管如此，看看 Mojo 如何继续沿着其 Python 兼容和快速用例的路径，进行开发还是很有趣的。