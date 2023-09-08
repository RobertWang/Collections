> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/BIhIBS1E5kU15w29stkTog)

容器技术已经成为新常态，而 WebAssembly 是未来的趋势。

—— CNCF 2022 年度调查关键发现

WebAssembly（Wasm）最初是为了在网络浏览器中安全地运行编译后的 C/C++ 代码而创建的。但近年来其在服务器端的应用也日益受到关注。在云计算领域，Wasm 为各种工作负载提供了一个轻量、高速、安全、不依赖特定语言且跨平台的运行环境。**它正迅速地成为云原生技术堆栈的核心部分。**

随着 Wasm 在云原生的项目、产品和服务中的广泛采纳，CNCF 与 Wasm 社区联手创立了一个 Wasm 生态全景图，帮助大家更深入地理解 Wasm 生态系统的广度和深度。正如原先的云原生生态图谱为我们描绘了云原生技术的庞大生态系统，我们相信随着 Wasm 生态系统的不断进化与增长，同样需要一个相对应的图谱。

首版 Wasm 生态全景图已在 WasmCon 会议上发布，包含了 **11 个类别，120 个项目或产品**，总经济价值达 594 亿美元。这个 Wasm 生态图谱主要分为两大部分：Dev（应用程序开发）和 Ops（应用程序部署）。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/vHicVZXtcAzDf7gVokZA1K0w2HCZK7WibNQ8D0UtoouBHvgouo2DempWnulLUx4zqibNOA228YbIchcnCZnqJGibmA/640?wx_fmt=png)

******#01******

**应用程序开发**  

Wasm 应用程序的开发需要自己的编程语言和相关工具生态，例如编译器、框架、库、工具和运行时。

**编程语言**

当开发者打算开发一个应用时，首先要做的是选择合适的编程语言。**Wasm 的一个显著特点是它可以运行多种编程语言编写的应用程序**，但并不是所有的编程语言都具有相同的效果。

在 Wasm 的领域内，大致上有四种类别的编程语言：

1、编译型语言

这些语言可以直接编译成 Wasm 字节码，在 Wasm 的运行环境中独立执行。C、C++、Zig 和 Rust 都属于这一类。他们生成的 Wasm 应用既速度快又占用空间小。

以 Rust 为例，安装 Rust 语言后，你需要做的就是添加 wasm32-wasi 目标。

2、托管型语言

托管语言仍然是编译型语言，但它们需要一个 “托管运行时” 来确保正确运行，其中最典型的任务是垃圾收集（GC）。

对于 Kotlin 和 Dart，Wasm 的 GC 功能是足够的。一些主要的 Wasm 运行时，例如 WasmEdge、Wasmtime 和 v8，近期都添加了对 Wasm GC 的支持。

对于 Go 语言，它的编译器会将所需的运行时直接嵌入到生成的 Wasm 字节码中，这使得 Wasm 应用程序体积增大，但仍然为开发者带来了良好的体验。

对于更为复杂的托管语言，如 Java 和 .Net（例如 C#），需要将他们的托管运行时（如 JVM）与应用程序的字节码一同编译到 Wasm 中，这通常比较重。

3、脚本语言

如 JavaScript、Ruby、PHP 和 Python 这样的脚本语言也可以在 Wasm 中运行。这里的方法是将通常用 C 编写的解释器编译到 Wasm 中，这样基于 Wasm 的解释器就可以执行这些脚本了。

例如，VMware Labs 的 WebAssembly 语言运行时项目已经将 Python 和 PHP 的解释器移到了 Wasm 上。

另外，WasmEdge 的 QuickJS 项目为 JavaScript 提供了一个解释器，并配有一个 Wasm 库，支持 Node.js 的 API。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/vHicVZXtcAzDf7gVokZA1K0w2HCZK7WibNbnOJdjjVIMrAVKAicFCMWu1FleBDwlx1AtBmwNwzv0EePLUeXKwica7g/640?wx_fmt=png)

来源：WasmEdge QuickJS 文档

4、专为 Wasm 设计的编译型语言

这是一个全新的编程语言类别，专门用来优化 Wasm。尽管它们现在还处于起步阶段，但如果发展得当，未来它们很有可能在 Wasm 领域中占据主导地位。

Moonbit 和 Grain 是这一类别中的领头羊。它们都拥有与 Go 和 Rust 类似的现代化语言特性，并针对 Wasm 的高效编译和执行进行了优化。

Moonbit 尽管仍然处于早期阶段，但已经提供了从动态代码补全到在线 IDE 的全套工具。

**运行时**

一旦源代码被编译成 Wasm 字节码，你将需要一个 Wasm 运行时来执行它们。Wasm 运行时为我们提供了所有与 Wasm 相关的功能和优势，如沙箱安全、安全性、速度和跨平台移植性。它们是这个领域的核心。

在面向云原生的 Wasm 领域，我们主要关注在服务器端受欢迎的 Wasm 运行时，包括 WasmEdge（CNCF 沙箱项目）、Wasmtime/lucet、Wamr、WAVM、Wasmer、wasm3、Lunatic、wazero、Wasmer 和 V8。

*   WasmEdge（CNCF 沙箱项目）是一个为云原生、边缘和去中心化应用而设计的轻量级、高性能、可扩展的 WebAssembly 运行时。
    
*   Wasmtime 是一个专为 WebAssembly 和 WASI 而设计的独立 wasm-only 优化运行时。
    
*   WebAssembly Micro Runtime（WAMR）是一个轻量级、小尺寸、高性能、特性高度可配置的独立 WebAssembly（Wasm）运行时，适用于从嵌入式、物联网、边缘到信任执行环境（TEE）、智能合约、云原生等领域。
    
*   WAVM 是一个为非浏览器应用设计的 WebAssembly 虚拟机。
    
*   Wasm3 是一个快速的 WebAssembly 解释器，是最普遍适用的 WASM 运行时。
    
*   Lunatic 是一个受到 Erlang 启发的 WebAssembly 运行时。
    
*   wazero 是一个用 Go 编写的满足 WebAssembly 标准的解释器。
    
*   Wasmer 是一个允许轻量级容器在任何地方运行的 WebAssembly 运行时。
    
*   V8 是 Google 的开源高性能 JavaScript 和 WebAssembly 执行引擎。
    

**应用框架**

Wasm 运行时可以看作是一个操作系统。库和框架为应用开发者提供了高层次、易于使用的组件。

WasmEdge 运行时在支持超出 Wasm 标准的高级 POSIX API 方面具有独特之处，这使得许多受欢迎的 Rust 和 JS 应用框架，例如 tokio、hyper、reqwest、warp、Node.js 以及 MySQL、Postgres、Redis、Kafka、ElasticSearch 客户端库都能在 WasmEdge 中运行。但对于所有其他 Wasm 运行时，应用框架需要为它们提供关键功能，如 HTTP/HTTPS 网络和数据库访问。

在此领域中，我们涵盖了 Spin、WasmCloud（CNCF 沙箱项目）、SpiderLightning、WasmEdge 插件、Dapr SDK for WasmEdge、Homestar、Ambient、WASIX、Extism、Timecraft、vscode-wasm 和 WasmEx。

*   Spin 是一个流行的应用框架和组件库，用于构建 WebAssembly 微服务和 Web 应用程序。它与 wasmtime 运行时兼容。
    
*   WasmCloud 允许使用 WebAssembly 的参与者和功能提供者进行简单、安全、分布式应用程序开发。
    
*   SpiderLightning 包含一套 WIT 接口，它能够为分布式应用程序提供抽象化的能力，并配备了一个命令行界面，用于运行这些 Wasm 应用程序。
    
*   虽然 WasmEdge 已经支持了许多流行的 Rust 应用程序框架，如 HTTP 和数据库访问，但它可以通过插件得到进一步增强，支持各种框架和库，包括 TLS 网络、zlib、OpenCV、ffmpeg、PyTorch、Tensorflow 以及大型语言模型（LLM）推理。
    
*   Dapr SDK for WasmEdge 是一个实验性的 Rust 版 Dapr SDK。它允许基于 WasmEdge 的微服务通过 Dapr sidecar 访问超过 100 个企业服务。
    
*   Homestar 是 IPVM 的基于 Rust 的实现和运行时。Ambient 是一个用于构建高性能多人游戏和 3D 应用程序的运行时，由 WebAssembly、Rust 和 WebGPU 驱动。WASIX 是 WASI 的超集。
    
*   Extism 是通用的插件系统，由 WebAssembly 驱动。
    
*   Timecraft 是一个软件运行时，可以执行具有沙箱、任务编排和时间旅行功能的 WebAssembly 模块。
    
*   vscode-wasm 是一个使用 VS Code 的扩展主机作为实现 API 的 WASI 实现。
    
*   Wasmex 是一个用于 Elixir 的快速和安全的 WebAssembly 和 WASI 运行时。
    

**边缘 / 裸机**

Wasm 的关键特性是跨平台的可携带性，这意味着可以在不同的操作系统和 CPU 架构上执行同一应用程序。在 JVM 的时代，操作系统通常仅限于类 Unix 系统（如 Linux）、Windows 和 MacOS。然而，Wasm 可以运行在边缘和物联网计算中经常使用的非常规系统上。

Docker 和 Linux 容器不是真正的跨平台的。例如，容器镜像必须与主机 CPU 相匹配。

*   Genode 是一个由微内核抽象层和一组用户空间组件组成的免费开源操作系统框架。
    
*   SeL4 RTOS 是一个经过全面正式验证，性能不受影响的高保障、高性能的操作系统微内核。
    
*   Flatcar Container Linux 是一个为容器而设计的轻量级、简化的 Linux 发行版，它是 Wasm 运行时的理想选择。
    
*   Unikraft 是一个快速、安全的开源 Unikernel 开发套件。
    

**AI 推理**

随着 AI 工作负载在云数据中心的增长，Wasm 正逐渐被视为与复杂、低速的 Python 堆栈相对的轻量级选择。WASI NN 规范明确了 Wasm 运行时如何与原生的 AI/ML 库（例如 PyTorch 和 TensorFlow）互动，在如 Rust 这样的高性能语言中进行 AI 推理。

其中，Wasmtime、WasmEdge 和 WAMR 是支持 WASI NN 的 Wasm 运行时。

例如，WasmEdge 运行时支持如 OpenVINO、Pytorch、Tensorflow、TensorFlow Lite 以及 MMGL/Llama2 作为推理后端，还支持 OpenCV 和 FFmpeg 作为预处理或后处理程序。它还支持诸如 MediaPipe、Document AI、和 Llama 2 这样的模型。

**嵌入功能**

Wasm 能够安全地在软件产品中执行用户或社区提供的代码，作为嵌入式功能或插件。

在这一部分，我们展示了选择并将 Wasm 整合为其插件机制的软件产品。其中，最多的是数据库和数据流应用，这些应用使用 Wasm 来执行用户定义的函数（UDFs）。

例如，数据库如 libSQL、OpenGauss 和 Singlestore，以及消息队列如 Open Policy Agent、InfinyOn、YoMo、eKuiper、和 Redpanda 都采用了 Wasm 来执行 UDFs。而像 Envoy、Istio、APISIX、KubeWarden 和 NGINX 这样的流量代理则使用 Wasm 在数据层中执行自定义逻辑。此外，FaaS 平台如 OpenFunction 和 Knative 允许 Wasm 函数被嵌入到 Kubernetes Pod 中。

*   libSQL 是一个开源、公开贡献的 SQLite 分支。它的目标是将 SQLite 引入到服务器端。
    
*   OpenGauss 是一个企业级开源关系数据库，拥有高性能、高安全性和高可靠性等特点。
    
*   SingleStore 是一个为处理大量数据的应用程序而构建的分布式 SQL 数据库。
    
*   Open Policy Agent 是一个开源的、通用的策略引擎。
    
*   InfinyOn 是一个可组合的统一的数据流平台。
    
*   YoMo 是一个开源的、为构建低延迟地理分布式系统而设计的流式无服务器框架。
    
*   eKuiper 是一个轻量级的边缘物联网数据分析 / 流软件。
    
*   Redpanda 是一个简单、高效、与 Kafka® API 兼容的流数据平台，同时消减了 Kafka 的复杂性。
    
*   Envoy 是一个为云原生应用设计的开源边缘和服务代理。
    
*   Istio 是一个在已有分布式应用上透明叠加的开源服务网格。
    
*   Apache APISIX 是一个动态、实时、高性能的 API 网关。
    
*   Kubewarden 是一个为 Kubernetes 设计的策略引擎，目标是简化政策即代码的采用。
    
*   Nginx 是一个 Web 服务器，也可以被用作反向代理、负载均衡器、邮件代理和 HTTP 缓存。
    
*   OpenFunction 是一个云原生的开源函数即服务（FaaS）平台。
    

**工具**

最后，开发者需要依赖各种工具将语言、框架、库和运行时组合成完整的应用程序。工具的成熟度在很大程度上代表了整个开发生态的成熟度。在 Wasm 领域，有许多重要的工具供开发者构建 Wasm 应用。

*   Cargo：从 Rust 源代码中构建 Wasm 应用，提供了现代化的依赖管理和源代码仓库。
    
*   LLVM：从理论上说，任何拥有 LLVM 后端的语言都可以被编译到 Wasm。 
    
*   Binaryen：WebAssembly 的编译器和工具链库。
    
*   Emscripten：使用 LLVM 和 Binaryen 将 C 和 C++ 编译到 WebAssembly。
    
*   wasm-pack：使 Rust 编译成可与 JavaScript 交互的 Wasm，可在浏览器或 Node。js 中使用。
    
*   wasm-bindgen：促进 Wasm 模块与 JavaScript 之间的交互。
    
*   Wabt：一套 WebAssembly 工具集，包括 wat2wasm、wasm2wat 和 wasm2c 等。
    
*   Witc：针对 *.wit 文件的编译器。
    
*   Wit bindgen：WIT 和组件模型的绑定生成器。
    
*   Asyncify：用于与 Binaryen 的 Asyncify 特性结合使用的 JavaScript 包装器。
    

******#02******

**应用程序部署**  

当你创建了一个 Wasm 应用程序后，下一步是在生产环境中部署和扩展它。在云原生领域有大量的工具、框架和服务来管理应用程序部署。其中许多都集成了 Wasm 支持。

**编排与管理**

Wasm 容器可以通过现有的容器工具，如 Docker、containerd 和 Kubernetes 无缝管理。管理 Wasm 应用作为 “容器” 的方法有两种，这两种方法都可以让你在 Kubernetes 集群中并行运行 Linux 容器和 Wasm 容器。

**第一种方法是使用 OCI 运行时**，如 crun 和 youki，作为容器管理的基础。crun 可以根据镜像的目标 OS/CPU 平台判断一个 OCI 镜像是基于 Wasm 还是 Linux。如果目标是 wasi/wasm，crun 会跳过 Linux 容器的设置流程，直接使用 WasmEdge 来执行。基于这种方式，整个 Kubernetes 的技术栈，包括 CRI-O、containerd、Podman、kind、K8s、OpenYurt、SuperEdge、KubeEdge 等都能与 Wasm 镜像一起工作。

**第二种方法是使用 containerd-shim**，例如 runwasi，在 containerd 中执行 Wasm 应用。当 containerd 获取到一个镜像时，它会检查镜像的目标平台，若为 wasi/wasm 则通过 runwasi 执行，若为 x86 或 Arm 则通过 runc 执行。

基于 Kubernetes，还有几种新兴工具可以帮助管理 Wasm 工作负载：

*   Kuasar 是一个支持多种沙箱类型的容器运行时，包括 microVM、Linux 容器、app kernels 以及 WebAssembly 运行时。
    
*   Kwasm 是一个 Kubernetes Operator，为 Kubernetes 节点增加 WebAssembly 支持，可以与基于 Ubuntu/Debian 的本地和云管理的 K8s 配合使用。
    
*   container2wasm 能够将容器转换为 wasm 镜像，使其可以在 WASM 上运行。
    

**托管平台**

如果你不希望自己管理服务器和 Kubernetes 集群，那么托管平台是部署和扩展 Wasm 应用的理想选择。

*   Flows.network 是一个面向 AI 工作流自动化的无服务器 Wasm 平台。
    
*   Fermyon Cloud 允许在 Spin 框架中部署和管理无服务器 Wasm 函数。
    
*   Cosmonic 是部署在 WasmCloud 上的 Wasm-based actor 服务。
    
*   Cloudflare workers 是基于 v8 的 Wasm 和 JavaScript 无服务器函数运行时，运行在 Cloudflare 边缘网络上。
    
*   Fastly @Edge functions 是部署在 Fastly 边缘网络上的无服务器 Wasm 函数平台。
    
*   AKS 让你在 Azure Kubernetes Service 中创建 WASI 节点池，运行你的 WebAssembly 工作负载。
    
*   Taubyte 是一个云原生平台，特别适用于运行 Wasm 应用。
    
*   Golem Cloud 是一个持久计算平台，允许开发者使用 Wasm 构建和部署长时间运行的、有状态的无服务器应用。
    

**去中心化平台**

基于区块链的智能合约平台是去中心化的云计算网络。与中心运营者管理所有的任务不同，你的云应用（即智能合约）是由网络中的节点执行的。由于智能合约是不被信任的并需要非常频繁地执行（在每台计算机上每秒执行数百次），Wasm 成为此用例的理想执行引擎。实际上，几乎所有领先的智能合约区块链网络都选择了 Wasm。

*   Polkadot
    
*   NEAR
    
*   Dfinity
    
*   GEAR
    
*   Filecoin
    
*   Ripple
    
*   EOS
    
*   CosmWasm
    
*   Quai Network
    

**调试与可观测性**

调试与可观测性是允许运维团队不断监控应用程序并为开发团队提供反馈的重要特性。在 Wasm 领域，这是一个目前还相对薄弱的区域。随着 Wasm 在生产中的部署日益增多，我们预期这个领域也会随之增长。

*   WASI logging 是一个用于输出日志信息的 Wasm 规范。这被主要的 Wasm 运行时，如 wasmtime 和 WasmEdge 所支持。
    
*   Modsurfer 为运维和开发团队提供了第一个用于搜索、浏览、验证、审计和研究 Wasm 二进制文件的系统记录 + 诊断应用。
    

**工件**

工件仓库在这个领域是非常关键的。它们为存储、发现、验证、下载和跟踪在多个版本中发布的 Wasm 包提供了集中且始终可用的位置。这些不仅是为了方便，而且对于软件供应链的安全也至关重要。

*   Docker Hub 是一个创建、管理和交付团队容器应用程序的地方。如果你在 Docker Hub 上传一个纯 Wasm 镜像，你的镜像的操作系统 / 架构将被标记为 wasi/wasm。
    
*   Harbor 是一个开源的、值得信赖的云原生注册表项目，能够存储、签名和扫描内容。它也支持 Wasm 工件。
    
*   Warg 是一个 WebAssembly 组件的注册表。
    
*   wapm 是针对 WebAssembly 的包管理器。
    
*   crates.io 是 Rust 的 crates 注册表，这是最常使用的 Wasm 语言。
    

Wasm 领域的建立是社区的共同努力，随着 Wasm 的普及，这个领域也在快速演进。我们会持续更新，以便与时俱进。但我们需要整个 Wasm 社区的支持，欢迎你也能参与进来。

最后推荐一下分布式实验室出品的《Kubernetes 实战视频课》，本课程一共 62 个课时，15 个动手练习，帮助学员通过实战掌握 Kubernetes，最后会讲解 CKA 考试，帮助有计划考 CKA 的同学顺利通过认证。报名即可学习，视频永久有效。