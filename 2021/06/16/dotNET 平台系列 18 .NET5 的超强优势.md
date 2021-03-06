> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/SavionZhang/p/14805610.html) **系列目录     [【已更新最新开发文章，点击查看详细】](https://www.cnblogs.com/SavionZhang/p/14764534.html "已更新最新开发文章，点击查看详细")**

****支持所有 .NET 应用程序类型****

　　.NET5 统一版本之后将支持所有 .NET 应用程序类型：Xamarin、ASP.NET、IoT 和桌面。此外，它将利用一个单独的 CoreFX / 基类库 (BCL)、两个独立的运行时和运行时代码库（因为很难将两个截然不同的运行时单独作为源）和一个工具链（比如 dotnet CLI）。结果将是行为、API 和开发人员体验之间的一致性。例如，在每个不同平台上将运行一组库，而不是三个 System.* API 实现。

![](https://img2020.cnblogs.com/blog/142275/202105/142275-20210525120545427-78735907.jpg)

****将框架、运行时和开发人员工具集统一到一个代码库中****

　　.NET 的统一有很多优点。将框架、运行时和开发人员工具集统一到一个代码库中，将减少开发人员（Microsoft 和社区）需要维护和扩展的重复代码量。此外，正如我们最近对 Microsoft 的期许，所有 .NET 5 源代码都将是开放源代码。

　　合并后，所有平台都可以使用每个单独框架独有的许多功能。例如，这些平台的 csproj 类型将统一为深受欢迎的、简单的 .NET Core csproj 文件格式。因此，.NET Framework 项目类型将能够利用 .NET Core csproj 文件格式。虽然 Xamarin 和 .NET Framework（包括 WPF 和 Windows 窗体）csproj 文件需要转换为 .NET Core csproj 文件格式，但该任务类似于从 ASP.NET 转换为 ASP.NET Core。幸运的是，得益于诸如 ConvertProjectToNETCore3 之类的工具，现在实现起来更加容易（请参阅 [bit.ly/2W5Lk3D](http://bit.ly/2W5Lk3D)）。

![](https://img2020.cnblogs.com/blog/142275/202105/142275-20210525120603484-2029666417.png)

**支持 JIT 与 AOT 两种编译模式**

　　另一个显著差异是 Xamarin 和 .NET Core/.NET Framework 的运行时行为。前者使用静态编译模型，使用提前 (AOT) 编译将源代码编译为平台的本机源代码。而 .NET Core 和 .NET Framework 使用即时 (JIT) 编译。幸运的是，在 .NET 5 中，JIT 和 AOT 这两种模型都将受支持，具体取决于项目类型目标。例如，可以选择将 .NET5 项目编译为单个可执行文件，该文件将在运行时使用 JIT 编译器 (jitter)，或使用本机编译器在 iOS 或 Android 平台上工作。大多数项目都会利用 JIT，但对于 iOS 来说，所有代码都是 AOT。对于客户端 Blazor，运行时是 Web 程序集 (WASM)，Microsoft 打算 AOT 编译少量托管代码（大约 100 kb 到 300 kb），而其余代码将被解释。（AOT 代码很大，因此网络成本是一个相当大的负担。）

**创建**单个可执行文件****

　　在 .NET Core 3.0 中，可以编译到单个可执行文件，但该可执行文件实际上是运行时所需执行的所有文件的压缩版本。在执行该文件时，它首先将自己展开到一个临时目录中，然后从包含所有文件的目录中执行应用程序的入口点。相反，.NET 5 将创建一个实实在在的、可直接就地执行的单个可执行文件。

****互操作性****

　　.NET 5 的另一个显著特性是与 Java 和 Objective-C（包括 Swift）中源代码的互操作性。自早期版本以来，这一直是 Xamarin 的一个特性，但将扩展到所有 .NET5 项目。例如，你将能够在 csproj 文件中包含 jar 文件，并且能够直接从 .NET 代码调用 Java 或 Objective-C 代码。（遗憾的是，对 Objective-C 的支持可能会比 Java 晚）。 需要注意的是，.NET5 和 Java/Objective-C 之间的互操作性只针对进程内通信。与同一台计算机上的其他进程甚至不同计算机上的进程的分布式通信可能需要序列化为基于 REST- 或 RPC- 的分布式调用。

**容器支持的优势**

　　新的互联网技术时代已经来临了，容器、Kubernetes、DevOps、微服务、云原生才是技术前进的方向，其中容器技术属于基石。从. NET Core 诞生直到. NET5，都能持续看到平台对容器技术的官方支持和适配改进，里面还强调了有着更小的容器镜像。.NET5+Docker 容器化后还有其他语言无可比拟的优势！

![](https://img2020.cnblogs.com/blog/142275/202105/142275-20210525120429793-1595982108.png)

**1、体积更小**

.NET5 的镜像体积都很小，alpine 的镜像更小，带上应用程序也才 80M，对于微服务分布式架构而言，更小的体积意味着更少的下载带宽，更快的分发下载速度。

**2、占用资源更少**

.NET5 的 CLR + 默认 http://ASP.NET Core 框架页面启动后，仅需 22M 内存，同比 Java8 已经需要 120M 了，运行时资源占用也更低，意味着更高的部署密度和更低的计算成本。

**3、启动速度更快**

.NET5 的 CLR 启动速度非常快，而启动速度就意味着交付效率和回滚效率，在动辄数百个副本微服务时，启动速度就是个非常重要的特性。

**4、容器感知，低配运行**

.NET5 默认更好的支持 Docker 资源限制，官方团队也在努力让. NET5 成为真正的容器运行时，使其在低内存环境中具有容器感知功能并高效运行，远超其他平台。

**云原生支持的优势**

.NET 团队一直将重点放在. NET5 领域，并引入了新的改进和功能：

*   **REST API** 可以更简单地构建测试，并将其发布到诸如 Azure API 管理之类的应用程序中。此外，还可以在默认情况下由 OpenAPI 生成客户端。
*   **gRPC** gRPC 可以构建与 WCF 类似的高性能基于合约的 API。
*   **较小，更快的微服务** .NET 团队在. NET5 中完成的一件很酷的事情是，您可以选择一个 ASP .NET 项目，然后选择要发布的项目，这将生成一个 20m 的小型自包含应用程序，完全不需要在计算机上运行. NET。
*   **使用 WSL 和 Linux 进行跨平台开发**
*   **高性能反向代理（YARP）**

参考文献：

*   https://devblogs.microsoft.com/dotnet/announcing-net-5-0/
*   https://docs.microsoft.com/zh-cn/archive/msdn-magazine/2019/july/csharp-net-reunified-microsoft%E2%80%99s-plans-for-net-5

**系列目录     [【已更新最新开发文章，点击查看详细】](https://www.cnblogs.com/SavionZhang/p/14764534.html "已更新最新开发文章，点击查看详细")**
