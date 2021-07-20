> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAwNTAyMDY0MQ==&mid=2652597591&idx=1&sn=f1a34316896860fcff6237d42c28cde4&chksm=80ccc3d9b7bb4acf9b336f9cd073f43c64a336a901a2a42a705264376b99658196d0bc958ab2&scene=21#wechat_redirect)

象征兼具速度和安全的程序员的漫画。绘画：Project Twins

Rust 是 2006 年由 Graydon Hoare 在加利福尼亚州浏览器公司 Mozilla 工作的时候作为小项目开发的。Rust 混合了 C++ 这类语言的性能、友好的语法、对代码安全性的关注和一套精心设计的工具用以简化开发流程。Mozilla 的 Firefox 中有一部分就是用 Rust 写的。据报道微软也使用 Rust 重写了 Windows 操作系统中的一部分。**每年一度的 Stack** **Overflow 开发者调查已经连续五年将 Rust 列为 “最受喜爱的” 编程语言**。代码共享网站 GitHub 说，**Rust 是该平台 2019 年增长第二快的语言，比去年增长了 235%。**

科学家们同样看向了 Rust。例如，Köster 用它写了一个叫做 Varlociraptor 的应用。该应用能将数百万的基因序列与几十亿种基因碱基比对，以找出基因突变。“数据量极大，” 他说，“所以比对必须尽可能地快。” 但是，Rust 的强大是有代价的：**学习曲线很陡。**

“上手之前是得花些时间。” 宾夕法尼亚州咨询公司 Integer 32 的创办者，也是 Rust 核心团队成员的 Carol Nichols 说。“但是它让我能够做一些此前无法做到的事。我认为这些时间花得值。”

**警告：没有护栏**

分析科学数据的工作流程通常会使用 Python、R 和 Matlab 这样的语言。这类语言会逐行解释代码并执行。**该模式在探索数据的时候很有用，但是速度不会快。**

C 和 C++ 很快，但是没有 “安全护栏”。斯德哥尔摩的 Rust 程序员（他们管自己叫 Rustacean），Ashley Hauck 说。例如，没什么能阻止 C 和 C++ 的程序员访问已经释放回系统的内存，或是把同一块内存释放两次。最好的情况下，这样做会让程序崩溃，但也有可能返回无意义的数据或是产生安全漏洞。根据微软的研究，他们公司每年修补的安全漏洞中有 70% 都和内存安全有关。

**内存规则**

**Rust 的模型所使用的规则将每一片内存都分配给了一个单一的所有者**，并限制了谁能访问它。违反规则的代码根本不会有机会崩溃——它根本就无法编译。“他们的内存管理系统是基于这个生命周期概念的，允许编译器可以在编译的时候追踪内存何时分配，何时释放，由谁持有，谁能访问，”Rob Patro 说。他是马里兰大学的一位计算生物学家。“有一整类正确性问题都可以通过语言的设计方法避免。”

这套理念还会帮助保证并行计算——使用多个处理器同时进行计算的软件——可以安全执行。例如，可以避免多个线程同时访问同一份数据的可能性。

**结果是，这种语言更易于维护和调试，但是学习起来就更难了。**“其他任何一种主流语言都没有这些概念，而它们是理解 Rust 编程方式的核心。”Nichols 说。在都柏林圣三一大学研究地理数据可视化的 Stephan Hügel 估计，他花费了两三个月把一个将地理空间坐标转化进另一个参照系的 Python 算法改写成了 Rust，执行速度快了 4 倍。加利福尼亚州的一个化学信息学软件公司 Metamolecular 的创办者 Richard Apodaca 说他为熟练使用 Rust 花了六个月。

**聚焦易用性**

为了弥补这一问题，Rust 的开发者花功夫改进了用户体验，在加利福尼亚州的 Rust 开发者工具团队的主管 Manish Goregaokar 说。例如，编译器会返回特别有信息量的错误信息，甚至会将出错的代码高亮出来，并建议如何修正。“如果你的语言想要引入新的概念，那最好用起来方便一些。”Goregaokar 解释道。

**Rust 社区还提供了详尽的文档和在线帮助**，其中包括一本大受欢迎的在线详解 “Book” 和一份介绍如何解决常见问题的“Cookbook”。Rust 工具链——程序员用来将代码转化成程序的工具——很受用户好评（见下文 “大家一起氧化吧”）。“Rust 的工具和架构真的很棒。”Patro 说。相较于 C 程序员需要面对的很多种编译器和辅助应用，Rust 程序员只需要一个叫做 Cargo 的工具就可以编译 Rust 代码、运行测试、自动生成文档、将代码上传到代码库中，等等。它还会自动下载并安装第三方软件包。Cargo 的一个插件 Clippy 可以标亮常见错误和 “不怎么规范” 的 Rust 代码，Patro 评价这一特性是“绝对棒”。

**大家一起氧化吧**

下面以创建一个读取 GenBank 文件的程序的步骤为例，探索 Rust 语言的一些功能。

• 访问 www.rust-lang.org/learn/get-started 安装 Rust

• 从 GitHub 上复制代码 https://github.com/jperkel/gb_read

• 在命令行中执行 “cargo run” 以下载外部依赖，并编译程序。程序的默认设置是读取 GitHub 代码库中的 GenBank 文件 “nc_005816.gb”，但是你可以通过“cargo run <文件名>” 选择读取其他文件。

• 使用 “cargo test” 运行代码库里面的测试。

• 使用 “cargo doc --open” 创建并阅读文档。

流行的开发环境中也有 Rust 的插件，例如微软的 Visual Studio Code 和 JetBrains 的 IntelliJ。还有一个在线的 Rust Playground，允许实时在线实验 Rust 代码。住在悉尼的 David Lattimore 在 Jupyter 计算笔记本里面创建了一个 Rust 的内核，以及一个类似于 Python 的交互式环境 REPL。

Rust 程序员的另外一大助力是它的第三方软件包（Rust 管它叫 “crate”）生态系统，目前已经有了 5 万多个（见下文 “Rust 越来越火”）。软件包封装了例如生物信息学（Kösterd 的 Rust-Bio）、地理科学（Geo-Rust 项目）、数学（nalgebra）等学科的算法。不过，Nichols 说，“要是你想要的库没有 Rust 版本，那就是 Rust 的大劣势了。”当然，程序员有时候还是可以使用 Rust 的 “外部函数接口” 来搭一座桥。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/0OWoGbRW1ic8DmTibxxPWn5ic83enef67KXLoXJnL0rMK08X2rT5t2Uh2yw0rCvuKUa7hGibJpfiaMbOdcDu5Wic4H3g/640?wx_fmt=png)

来源：http://www.modulecounts.com

**氧化代码**

不论编程流程如何，**不可否认的一点是 Rust 非常快。**五月，马萨诸塞州 Dana-Farber 癌症研究所的生物信息学家 Heng Li 用一个计算生物学的任务测试了多种编程语言，其中需要读取 570 万条基因记录。Rust 超越了 C 语言获得了第一名。“如果我们想要写一段高性能的并行计算程序，还需要速度快、占用内存少，Rust 是理想的选择。”Li 说。

加利福尼亚大学戴维斯分校的生物信息学家 Luiz Irber 使用 Rust 重写（用 Rust 界的俚语说叫 “氧化”）了一个叫做 Sourmash 的工具——它可以进行基因搜索和分类——以让代码易于维护，使用现代的编程特性，并能在网页上使用，他说。

在团队成员 Avi Srivastava 从用 Rust 开发开源工具的加利福尼亚州生物技术公司 10x Genomics 实习归来之后，Patro 的团队就由研究生 Hirak Sarker 带领，使用 Rust 写了一个叫做 Terminus 的基因表达分析工具。“**Rust 的美妙之处在于，它让调试非常简单，因为内存管理好太多太多了。**” 现在在纽约基因组中心工作的 Srivastava 说。

**对很多 Rust 程序员来说，这里的 “人性化” 也很吸引人。**LGBT + 社群的成员 Hauck 说 Rust 的用户群下了功夫欢迎她。这个社群，她说，“一直以来尽力成为具有包容性的社群——例如，非常理解多元化会产生什么影响；非常理解如何编写并执行行为规范。”

“我之所以还在用 Rust，这可能是很重要的原因之一。”Hauck 说，“Rust 的社群实在太棒了。”

_原文以_ _Why scientists are turning to Rust_ _标题发表在 2020 年 12 月 1 日的《自然》的技术特写版块上_

**© nature**

**doi: 10.1038/d41586-020-03382-2**

点击**阅读原文**查看英文原文

版权声明：  

本文由施普林格 · 自然上海办公室负责翻译。中文内容仅供参考，一切内容以英文原版为准。欢迎转发至朋友圈，如需转载，请邮件 China@nature.com。未经授权的翻译是侵权行为，版权方将保留追究法律责任的权利。

© 2021 Springer Nature Limited. All Rights Reserved