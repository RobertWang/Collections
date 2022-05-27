> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU3Mzg4OTMyNQ==&mid=2247497279&idx=1&sn=9aa9e98c4491dd11016fdab720a5d2aa&chksm=fd38787dca4ff16b049e9a04e926afba4dcef648b96a385c3c6e00a604be20c9c3981a67c690&scene=132#wechat_redirect)

> 原 Flink Forward Asia 2021 演讲中的 JCP 项目已改名为 PEMJA，并且于 2022 年 1 月 14 日正式开源，开源地址为：
> 
> https://github.com/alibaba/pemja
> 
>   
> 
> Ps：JCP 已在本文替换为 PEMJA。

**一、PyFlink 新功能** 

易用性方面，主要新增了以下几项功能：

*   在依赖管理部分支持了 tar.gz 格式。
    
      
    
*   Profile 功能。用户写 PyFlink 会用到一些 Python 的自定义函数，但并不清楚这部分函数的性能瓶颈在哪里。而有了 profile 功能之后，Python 函数出现性能瓶颈时，便可以通过 profile 分析它的瓶颈具体是由原因什么引起，从而可以针对这部分进行一些优化。
    
      
    
*   Print 功能。在 1.14 以前，打印自定义的 log 信息必须使用 Python 自定义的 logging 模块。但对于 Python 用户来说，print 是他们比较习惯使用的一种输出日志信息的方式。所以在 1.14 添加上了这部分功能。
    
      
    
*   Local Debug 模式。在 1.14 以前，用户如果使用 Python 自定义的函数在本地开发 PyFlink 作业，必须使用 remote debug 方式调试自定义逻辑，但它使用起来相对比较繁琐，而且使用门槛较高。在 1.14 改变了这种模式，如果在本地编写一个 PyFlink 作业使用了 Python 自定义函数，可以自动切到 local debug 模式，可以在 ide 里面直接 debug 自定义 Python 函数。
    

性能方面主要新增了以下功能：

*   Operator Fusion。这个功能主要针对在 Python Datastream API 的作业中做连续几个算子操作的场景。比如两次 .map 操作，在 1.14 以前，这两个 .map 会分别运行在两个 Python worker 中，而实现了 Operator Fusion 后，它们会被 merge 并运行在同一个 operator 中，然后由 Python worker 执行总的结果，达到了很好的性能优化。
    
      
    
*   State 序列化 / 反序列化优化。在 1.14 以前，State 序列化 / 反序列化优化是使用 Python 内置的序列化器 pickle，它能序列化各种 Python 自定义的数据结构，但需要把 State type 的信息序列化到数据结构中，这会导致序列化的结构体会更大。1.14 中对其进行了优化，使用了自定义的序列化器，一个 type 对应一个序列化器来做优化，使得序列化信息更小。
    
      
    
*   Finish Bundle 优化。在 1.14 以前 Finish Bundle 是同步的操作，如今把它改成了异步的操作，提高了它的性能，而且能解决一些 Checkpoint 无法完成的场景。
    

**二、PyFlink Runtime**

![](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu6PAplz7wDYNrPU9abAAWKzlJEwj8ejVsmYAPzpGDcFN5TK6EX8OpD5G9Edu2f2TVcxAFNyV9LJbA/640?wx_fmt=jpeg)

上图是 PyFlink 现有的框架图。

图左侧的最上方的 Python Table API & SQL 和 Datastream API 是提供给用户的 Python API。用户通过这两个 Python API 编写 PyFlink 作业，再通过一个 py4j 的三方库把 Python API 转换成 Java API，即可对应到 Flink Java API 来描述这个作业。

针对 Table 和 SQL 的作业有个额外的 optimizer，它有两种 rule，一种是常见的 common rules，另一种是 Python rules。这里为什么会有 Python rules？众所周知，common rules 针对各种 Table 和 SQL 现有的作业都是有效的，而 Python rules 做的优化是针对 PyFlink 作业中使用了自定义的 Python 函数的场景，能够把对应的 operator 抽取出来。

描述完了作业之后，它会被翻译成一个 jobgraph，里面有对应的 Python operators。Python operators 描述的 jobgraph 会提交到 TM (Runtime) 上去运行， Runtime 中也有个 Python operators。

图右侧是 Python operators 的各种组件，描述了 PyFlink Runtime 最核心的部分。主要分为两个部分：Java operator 和 Python worker。

Java operator 中它有很多个组件，包括 data service 和 State service，以及针对 checkpoint、watermark 和 State request 的一些处理。因为自定义 Python 函数无法直接运行在 Flink 现有的架构之上，Flink 现有的架构是基于 JVM 的，但是编写 Python 函数需要一个 Python Runtime，所以用 operator worker 来解决这个问题。

解决方案如下：发起一个 Python 进程运行 Python 自定义的函数，同时使用 Java operator 处理上游来的数据，再经过特殊处理之后发送给对应的 Python worker。这里使用的是进程间通信的方案，也就是图中的 data service。State service 针对 Python Datastream API 对 State 的操作，通过在 Python 里操作 State，数据会从 Python worker 返回到 Java operator，Java operator 再通过访问 State backend 拿到对应的 State 数据，并回传给 Python worker，最后用户就可以操作 State 的结果了。

![](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu6PAplz7wDYNrPU9abAAWKzhVgpoNWwBjgeDDicWicN3yFVpoznFANXMdDGeicfuUztl4jnK1LoccPlQ/640?wx_fmt=jpeg)

上图是 PyFlink Runtime Workflow。里面的角色分别是 Python operator、Python runner、bundle processor、coder、Python operation，这几个不同的角色运行在不同的地方。其中 Python operator 和 Python runner 是运行在  Java JVM 里，负责对接上游和下游的 Java operator，而 bundle processor、coder 以及 Python operation 运行在 PVM 里，bundle processor 利用了现有的 Apache Bean 框架，能够接收来自于  Java Python 的数据，它们之间使用了进程间通信。coder 是在 Python 端的一个自定义的序列化器，Java 端发送了一条数据，经过 Python operator 发送给 Python runner，由 Python runner 进行序列化后，再通过进程间的通信发送给 bundle processor。bundle processor 再把序列化后的二进制数组通过 coder 将它反序列化并得到一个 Python 对象。最后通过 Python operation 把反序列化之后的 Python 参数作为一个函数体的入参，然后调用自定义的 Python 函数，得到自定义的结果。

上述流程的瓶颈主要存在以下几个方面：首先是计算端调用用户自定义函数以及在调用之前，存在框架层 Python 写的开销；其次是自定义序列化部分，在 Java 端和 Python 端都需要序列化和反序列化数据；第三部分是进程间的通信。

针对上述瓶颈，进行了一些列优化：

*   计算方面，利用 codegen 将现有的 Python 调函数的变量全都改为常量，函数的执行效率会更高；另外，将现有的 Python operation 的实现全都改为 cython，相当于将 Python 转化为 .c 的实现方式，性能得到了大幅提升；
    
      
    
*   序列化方面，提供了自定义序列化器，全都是纯 c 的实现，比 Python 更高效
    
      
    
*   通信方面，目前暂未实现优化。
    
      
    
*   序列化和通信的问题，本质上就是 Java 和 Python 互相调用的问题，也就是如何优化 PyFlink 的 Runtime 架构的问题。
    

**三、基于 FFI 的 PEMJA**

Java 和 Python 互相调用已经是一个比较通用的问题，目前也已经有很多种实现方案。

![](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu6PAplz7wDYNrPU9abAAWKz9YoxsvULMAatSh7F4DJaZjncA43Y0EibeiaibXBFZj9Sw9gbxNFoOThJw/640?wx_fmt=jpeg)

第一种是进程间互相调用的方案，即网络通信的方案，包括以下几种：

*   socket 方案，它所有的通讯协议都是通过自己实现，可以很灵活，但是比较繁琐；
    
      
    
*   py4j 方案，即 PyFlink 和 PySpark 在客户端编写作业时都使用 py4j；
    
      
    
*   Alink 方案，它是在 Runtime 运行时使用 py4j，也有自定义的 Python 函数；grpc 方案，它利用现有的 grp service，不需要自定义的协议，有自定义的 service 和 message；
    
      
    
*   此外，共享内存的方案也是另一种进程间通信的方案，比如 Tensorflow on Flink，它是通过共享内存的方式实现的。还有 PyArrow Plasma，也是一种对象式的共享内存存储。
    

上述方案都是针对进程间通信，那么能否让 Python 和 Java 运行在同一个进程里，从而完全消除进程间通信带来的困扰？

确实有一些现有的库在这方面做了尝试，第一种方案是将 Python 转成 Java。比如 p2j 是把 Python 的 source code 转成 Java 的 source code，voc 是把 Python 代码直接转成 Java 的 bytecode，这种方案的本质就是将 Python 转成一套可以直接运行在 JVM 之上的代码。但这套方案也存在不小的缺陷，因为 Python 是在不断地发展，它有各种语法，而将 Python 语法映射到  Java 中对应的对象是很困难的，它们毕竟是不同的语言。

第二种方案是基于 Java 实现的 Python 解释器。首先是 Jython 方案，Python 其实是用 c 语言写的一套 Python 解释器，c 写的 Python 解释器可以运行在 c 之上，那么 Java 实现的 Python 解释器也就可以直接运行在 JVM 之上。另外一种方案是 Graalvm，它提供了一种 truffle framework 的方式，可以支持各种编程语言使用共同的结构，这种结构能运行在 JVM 之上，也就可以让各种语言运行在同一个进程里。

上述方案实现的前提是能够识别 Python code，也就意味着要能兼容现有的各种 Python code，但是目前来看，兼容是一个难以解决的问题，因此也就阻止了这套 Python 转成 Java 方案继续推广的可能性。

第三种是基于 FFI 的一套方案。

![](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu6PAplz7wDYNrPU9abAAWKzMlA7CiazJsZ9kKdSicn6ibVibmXibm8YKpKueCMfmS8z9kq1X40mNgbYOhg/640?wx_fmt=jpeg)

FFI 的本质就是 host language 如何调用一个 guest language，即 Java 与 Python 之间的互相调用，对应的具体实现方案有很多种。

Java 提供了 JNI (Java native interface)，让 Java 用户能够通过 JNI 的接口调用 c 实现的一些 lib，反过来也同样适用。有了这套接口之后， JVM 的厂商就会根据这套接口去实现 JNI，从而实现 Java 与 c 之间的互相调用。

Python/C API 也是类似的， Python 是一套 c 实现的解释器，因此能很好地支持 Python 代码调用 c 的三方库，反之也同样适用。

Cython 提供了一个工具，能够将 source code 转换成另一种语言能识别的代码。比如将 Python 代码转换成一套非常高效的 c 语言代码，再嵌入到 cPython 解释器中即可直接运行，非常高效。

Ctypes 是通过将 c 的 library 封装起来，使得 Python 能高效地调用 c 的 library。

![](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu6PAplz7wDYNrPU9abAAWKz9QuzGnTLiaSUFCIF1Os1orN6NTSs0hCSs6EGbMNkBbDhwU7piaaqgGrg/640?wx_fmt=jpeg)

上述提到基于 FFI 的方案的核心就是 c。有了 c 这个桥梁之后，一个 Java 写成的代码，通过 JNI 接口就能调用到 c，然后由 c 去调用 cPython API 的接口，最终实现 Java 和 Python 运行在同一个线程里，这就是 PEMJA 的整体思路。解决了进程间通信的问题，以及因为它本身是使用的是自己提供的 Python/C API，也就不存在兼容性的问题，克服了 Java 实现解释器的缺陷。

![](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu6PAplz7wDYNrPU9abAAWKzTdYjrOEYbfJ7NkzMDL5Mu09OwugoTd6ibfWARgiaDib0icXMZ1xGN3IunA/640?wx_fmt=jpeg)

上图展示了基于这套思想的几种实现，但这几种实现都或多或少存在一些问题。

JPype 解决的问题是 Python 调用 Java 的问题，不支持 Java 调用 Python，所以它并不适用这个场景。

JEP 实现了 Java 调用 Python，但它的具体实现存在很多限制，一是只能用源码安装，对环境的要求非常高，以及它需要依赖 cPython 三方的一些 .source 文件，非常不利于跨平台的安装使用。JEP 的启动入口必须是 JEP 的程序，需要动态加载类库，必须提前在环境变量中设置好，非常不利于它作为一个第三方的中间件插件运行在另一个架构上。此外还有性能上问题，它没有很好地克服现有的 Python GIL 的问题，所以导致它的性能并不是那么高效。

而 PEMJA 基本克服了上述问题，更好的实现了 Java 和 Python 互相调用。

![](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu6PAplz7wDYNrPU9abAAWKzBWFJ8XX2ZSnA4ST0tudNmdWQgeNKhDjntL1DEgXPCapWJA0Lof4Arw/640?wx_fmt=jpeg)

上图是几种框架的性能对比。这里使用了一个比较标准简单的 String upper 函数。这里主要比较的是框架层的开销，并不是自定义函数的性能，所以使用了一个最简单的函数。同时，考虑到现有的各种函数最常用的数据结构是 String，所以这里使用了 String。

这里分别对比的是 100 个 bytes 和 1000 个 bytes 在这 4 种解释器下的性能，可以看到 Jython 并没有像想象中那么高效，反而是这 4 种实现方案中性能最低的。JEP 的性能也远远比不上 PEMJA，PEMJA 在 100 bytes 的时候大概是纯 Java 实现的 40%，1000 bytes 的情况下性能居然超越了纯 Java 的实现。

如何解释这个现象呢？String upper 本身是一套 Java 的实现，而在 Python 中它是 .c 的实现，函数本身的执行效率比 Java 高，再结合框架开销足够小的情况，整体的性能反而比  Java 更高，也就意味着在某些场景下，Python UDF 的性能是有可能超越 Java UDF 的。

现在很多用户使用 Java UDF 而不使用 Python UDF 的一个关键点是 Python UDF 性能远远比不上 Java。但是如果 Java 的性能并没有比 Python 更好的话，Python 反而就有了优势，因为它毕竟是一种脚本语言，写起来是更方便。

![](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu6PAplz7wDYNrPU9abAAWKzJPTyxNAH8RAPtiacVHqLj4lxJXXUHEuHryU8FvDID1WxATLeMBfjZNw/640?wx_fmt=jpeg)

上图展示了 PEMJA 的架构。

Java 中的 damond thread 负责初始化以及最后的销毁以及在 PEMJA 和对应的 Python PVM 里创建及释放资源。用户使用的是 Java 中的 PEMJA 实例，实例映射到 PEMJA 中对应 PEMJA 的 instance，instant 会创建每一个 Python 的 sub interpreter。Python double interpreter 相对于全局 Python interpreter，是一个更小的能够掌控 GIL 的概念，它有自己独立的 hip 空间，所以能够实现命名空间的隔离。这里的每一个 thread 都会对应一个 Python sub interpret，可以在对应的 PVM 里执行自己的 Python function。

**四、PyFlink Runtime 2.0**

![](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu6PAplz7wDYNrPU9abAAWKzNjPYwtRjCHSJRZgerLZxR8L1AxR1iaH6nvaLkgt2YhsYHRRt0u7WGLA/640?wx_fmt=jpeg)

PyFlink Runtime 2.0 就是基于 PEMJA 做的。

上图左边是 PyFlink 1.0 的架构。里面有两个进程，一个是 Java 进程，一个是 Python 进程。它们之间的数据交互是通过 data service 和 State service 实现，使用了进程 IPC 通信。

有了 PEMJA 之后，就可以把 data service 和 State service 替换成 PEMJA Lib，随即可以把左边原来的 JVM 和右边的 PVM 运行在同一个进程里，从而彻底解掉的 IPC 进程通信的问题。

![](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu6PAplz7wDYNrPU9abAAWKzCkM6Qf0MRZAdZNDia08TtnN65uJ1Db7k7XibvpzNX9hibteR9H9lLqDpA/640?wx_fmt=jpeg)

上图将现有的 PyFlink UDF、PyFlink 基于 PEMJA 的一套 UDF 以及 Java UDF 做了性能对比。也是使用 String upper 函数，比较 100 bytes 和 1000 bytes 的性能。可以看到，在 100 bytes 的情况下，UDF on PEMJA 的实现已经基本达到 Java UDF 的 50% 的性能。在 1000 bytes 的情况下，UDF on PEMJA 的性能已经超越了 Java UDF。虽然这和实现了自定义的函数有关，但也能说明这套 PEMJA 框架的性能之高效。

**五 、Future Work**

![](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu6PAplz7wDYNrPU9abAAWKzuSuXK6ukN47Wuh99icuAibcrD1y59w3fkOeqoQWLsEprR5oEotTBI0JA/640?wx_fmt=jpeg)

未来，会开源 PEMJA 框架 (已于 2022 年 1 月 14 日正式开源)，因为它涉及到通用的解决方案，不仅仅是运用在 PyFlink 之上，各种 Java 和 Python 互相调用的方案也都可以利用这套框架，所以会对 PEMJA 框架做一个独立的开源。它的第一个版本暂时只支持 Java 调用 Python 功能，后续会支持 Python 调用 Java 的功能，因为 Python Datastream API 用 Python 写的函数调用 State 是依赖于 Python 调用 Java 的功能。此外，将实现 PEMJA 支持 Numpy 原生数据结构，实现了这个支持之后，pandas UDF 也就得以运用，性能将会得到质的飞跃。

欢迎大家加入 “PyFlink 交流群”，交流 PyFlink 相关的问题。