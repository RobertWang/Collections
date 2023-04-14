> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/syz201558503103/article/details/111058193)

在大家开始[深度学习](https://so.csdn.net/so/search?q=%E6%B7%B1%E5%BA%A6%E5%AD%A6%E4%B9%A0&spm=1001.2101.3001.7020)时，几乎所有的入门教程都会提到 CUDA 这个词。那么什么是 CUDA？她和我们进行深度学习的环境部署等有什么关系？通过查阅资料，我整理了这份简洁版 CUDA 入门文档，希望能帮助大家用最快的时间尽可能清晰的了解这个深度学习赖以实现的基础概念。

本文在以下资料的基础上整理完成，感谢以下前辈提供的资料：  
[CUDA——“从入门到放弃”](https://blog.csdn.net/fu6543210/article/details/90020330)  
[我的 CUDA 学习之旅——启程](https://zhuanlan.zhihu.com/p/27916653)  
[介绍一篇不错的 CUDA 入门博客](https://bbs.csdn.net/topics/390798229?list=lz) （该文引用的原链接失效，因此直接引用了此地址）  
[CUDA 编程入门极简教程](https://zhuanlan.zhihu.com/p/34587739)  
[显卡、GPU 和 CUDA 简介](https://blog.csdn.net/wu_nan_nan/article/details/45603299)

### 本文内容

*   [CPU、GPU](#CPUGPU_11)
*   *   [CPU](#CPU_12)
    *   [GPU](#GPU_18)
    *   [CPU 与 GPU](#CPUGPU_29)
*   [CUDA 编程模型基础](#CUDA_36)
*   *   [CUDA](#CUDA_37)
    *   [编程模型](#_44)
    *   [线程层次结构](#_60)
    *   [CUDA 的内存模型](#CUDA_92)

CPU、GPU
=======

CPU
---

CPU（Central Processing Unit）是一块超大规模的集成电路，是一台计算机的运算核心（Core）和控制核心（ Control Unit）。它的功能主要是解释计算机指令以及处理计算机软件中的数据。

CPU 与内部存储器（Memory）和输入 / 输出（I/O）设备合称为电子计算机三大核心部件。CPU 主要包括运算器（算术逻辑运算单元，ALU，Arithmetic Logic Unit）、控制单元（CU, Control Unit）、寄存器（Register）、和高速缓冲存储器（Cache）及实现它们之间联系的数据（Data）、控制及状态的总线（Bus）。简单来说就是：计算单元、控制单元和存储单元。CPU 遵循的是冯诺依曼架构，其核心就是：存储程序，顺序执行。

因为 CPU 的架构中需要大量的空间去放置存储单元和控制单元，相比之下计算单元只占据了很小的一部分，所以它在大规模并行计算能力上极受限制，而更擅长于逻辑控制。

GPU
---

**显卡**（Video card，Graphics card）全称显示接口卡，又称显示适配器，是计算机最基本配置、最重要的配件之一。显卡是电脑进行数模信号转换的设备，承担输出显示图形的任务。具体来说，显卡接在电脑主板上，它将电脑的数字信号转换成模拟信号让显示器显示出来，同时显卡还是有图像处理能力，可协助 CPU 工作，提高整体的运行速度。在科学计算中，显卡被称为显示加速卡。

原始的显卡一般都是集成在主板上，只完成最基本的信号输出工作，并不用来处理数据。显卡也分为独立显卡和集成显卡。一般而言，同期推出的独立显卡的性能和速度要比集成显卡好、快。

<table><thead><tr><th>类型</th><th>位置</th><th>内存</th></tr></thead><tbody><tr><td>集成显卡</td><td>集成在主板上，不能随意更换</td><td>使用物理内存</td></tr><tr><td>独立显卡</td><td>作为一个独立的器件插在主板的 AGP 接口上的，可以随时更换升级</td><td>有自己的显存</td></tr></tbody></table>

随着显卡的迅速发展，**GPU** 这个概念由 NVIDIA 公司于 1999 年提出。GPU 是显卡上的一块芯片，就像 CPU 是主板上的一块芯片。集成显卡和独立显卡都是有 GPU 的。  
![](https://img-blog.csdnimg.cn/20200309114408993.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_0,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEyODc1OA==,size_16,color_FFFFFF,t_70)

CPU 与 GPU
---------

在没有 GPU 之前，基本上所有的任务都是交给 CPU 来做的。有 GPU 之后，二者就进行了分工，CPU 负责逻辑性强的事物处理和串行计算，GPU 则专注于执行高度线程化的并行处理任务（大规模计算任务）。GPU 并不是一个独立运行的计算平台，而需要与 CPU 协同工作，可以看成是 CPU 的协处理器，因此当我们在说 GPU 并行计算时，其实是指的基于 CPU+GPU 的异构计算架构。在异构计算架构中，GPU 与 CPU 通过 PCIe 总线连接在一起来协同工作，CPU 所在位置称为为主机端（host），而 GPU 所在位置称为设备端（device）。  
![](https://img-blog.csdnimg.cn/20200309115739389.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_0,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEyODc1OA==,size_16,color_FFFFFF,t_0)  
GPU 包括更多的运算核心，其特别适合数据并行的计算密集型任务，如大型矩阵运算，而 CPU 的运算核心较少，但是其可以实现复杂的逻辑运算，因此其适合控制密集型任务。另外，CPU 上的线程是重量级的，上下文切换开销大，但是 GPU 由于存在很多核心，其线程是轻量级的。因此，基于 CPU+GPU 的异构计算平台可以优势互补，CPU 负责处理逻辑复杂的串行程序，而 GPU 重点处理数据密集型的并行计算程序，从而发挥最大功效。GPU 无论发展得多快，都只能是替 CPU 分担工作，而不是取代 CPU。

GPU 中有很多的运算器 ALU 和很少的缓存 cache，缓存的目的不是保存后面需要访问的数据的，这点和 CPU 不同，而是为线程 thread 提高服务的。如果有很多线程需要访问同一个相同的数据，缓存会合并这些访问，然后再去访问 DRAM。

CUDA 编程模型基础
===========

CUDA
----

2006 年，NVIDIA 公司发布了 CUDA(Compute Unified Device Architecture)，是一种新的操作 GPU 计算的硬件和软件架构，是建立在 NVIDIA 的 GPUs 上的一个通用并行计算平台和编程模型，它提供了 GPU 编程的简易接口，基于 CUDA 编程可以构建基于 GPU 计算的应用程序，利用 GPUs 的并行计算引擎来更加高效地解决比较复杂的计算难题。它将 GPU 视作一个数据并行计算设备，而且无需把这些计算映射到图形 API。操作系统的多任务机制可以同时管理 CUDA 访问 GPU 和图形程序的运行库，其计算特性支持利用 CUDA 直观地编写 GPU 核心程序。

CUDA 提供了对其它编程语言的支持，如 C/C++，Python，Fortran 等语言。只有安装 CUDA 才能够进行复杂的并行计算。主流的深度学习框架也都是基于 CUDA 进行 GPU 并行加速的，几乎无一例外。还有一个叫做 cudnn，是针对深度卷积神经网络的加速库。

CUDA 在软件方面组成有：一个 CUDA 库、一个应用程序编程接口（API）及其运行库 (Runtime)、两个较高级别的通用数学库，即 CUFFT 和 CUBLAS。CUDA 改进了 DRAM 的读写灵活性，使得 GPU 与 CPU 的机制相吻合。另一方面，CUDA 提供了片上（on-chip）共享内存，使得线程之间可以共享数据。应用程序可以利用共享内存来减少 DRAM 的数据传送，更少的依赖 DRAM 的内存带宽。

编程模型
----

CUDA 的架构中引入了主机端（host）和设备（device）的概念。CUDA 程序中既包含 host 程序，又包含 device 程序。同时，host 与 device 之间可以进行通信，这样它们之间可以进行数据拷贝。

**主机 (Host)**：将 CPU 及系统的内存（内存条）称为主机。

**设备 (Device)**：将 GPU 及 GPU 本身的显示内存称为设备。

**动态随机存取存储器 (DRAM)**：Dynamic Random Access Memory，最为常见的系统内存。DRAM 只能将数据保持很短的时间。为了保持数据，DRAM 使用电容存储，所以必须隔一段时间刷新（refresh）一次，如果存储单元没有被刷新，存储的信息就会丢失。（关机就会丢失数据）

典型的 CUDA 程序的执行流程如下：

1.  分配 host 内存，并进行数据初始化；
2.  分配 device 内存，并从 host 将数据拷贝到 device 上；
3.  调用 CUDA 的核函数在 device 上完成指定的运算；
4.  将 device 上的运算结果拷贝到 host 上；
5.  释放 device 和 host 上分配的内存。  
    ![](https://img-blog.csdnimg.cn/2020030912115333.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_0,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEyODc1OA==,size_16,color_FFFFFF,t_0)

线程层次结构
------

**核 kernel**  
CUDA 执行流程中最重要的一个过程是调用 CUDA 的核函数来执行并行计算，kernel 是 CUDA 中一个重要的概念。在 CUDA 程序构架中，主机端代码部分在 CPU 上执行，是普通的 C 代码；当遇到数据并行处理的部分，CUDA 就会将程序编译成 GPU 能执行的程序，并传送到 GPU，这个程序在 CUDA 里称做核（kernel）。设备端代码部分在 GPU 上执行，此代码部分在 kernel 上编写（.cu 文件）。kernel 用__global__符号声明，在调用时需要用 <<<grid, block>>> 来指定 kernel 要执行及结构。

CUDA 是通过函数类型限定词区别在 host 和 device 上的函数，主要的三个函数类型限定词如下：

*   **global**：在 device 上执行，从 host 中调用（一些特定的 GPU 也可以从 device 上调用），返回类型必须是 void，不支持可变参数参数，不能成为类成员函数。注意用__global__定义的 kernel 是异步的，这意味着 host 不会等待 kernel 执行完就执行下一步。
*   **device**：在 device 上执行，单仅可以从 device 中调用，不可以和__global__同时用。
*   **host**：在 host 上执行，仅可以从 host 上调用，一般省略不写，不可以和__global__同时用，但可和__device__同时使用，此时函数会在 device 和 host 都编译。

**网格 grid**  
kernel 在 device 上执行时，实际上是启动很多线程，一个 kernel 所启动的所有线程称为一个网格（grid），同一个网格上的线程共享相同的全局内存空间。grid 是线程结构的第一层次。

**线程块 block**  
网格又可以分为很多线程块（block），一个 block 里面包含很多线程。各 block 是并行执行的，block 间无法通信，也没有执行顺序。block 的数量限制为不超过 65535（硬件限制）。第二层次。

grid 和 block 都是定义为 dim3 类型的变量，dim3 可以看成是包含三个无符号整数（x，y，z）成员的结构体变量，在定义时，缺省值初始化为 1。grid 和 block 可以灵活地定义为 1-dim，2-dim 以及 3-dim 结构。

CUDA 中，每一个线程都要执行核函数，每一个线程需要 kernel 的两个内置坐标变量（blockIdx，threadIdx）来唯一标识，其中 blockIdx 指明线程所在 grid 中的位置，threaIdx 指明线程所在 block 中的位置。它们都是 dim3 类型变量。

一个线程在 block 中的全局 ID，必须还要知道 block 的组织结构，这是通过线程的内置变量 blockDim 来获得。它获取 block 各个维度的大小。对于一个 2-dim 的 block（D_x,D_y），线程 (x,y) 的 ID 值为 (x+y∗D_x)，如果是 3-dim 的 block (D_x,D_y,D_z)，线程(x,y,z) 的 ID 值为(x+y∗D_x+z∗D_x∗D_y) 。另外线程还有内置变量 gridDim，用于获得 grid 各个维度的大小。

每个 block 有包含共享内存（Shared Memory）, 可以被线程块中所有线程共享，其生命周期与线程块一致。  
每个 thread 有自己的私有本地内存（Local Memory）。此外，所有的线程都可以访问全局内存（Global Memory），还可以访问一些只读内存块：常量内存（Constant Memory）和纹理内存（Texture Memory）。

**线程 thread**  
一个 CUDA 的并行程序会被以许多个 threads 来执行。数个 threads 会被群组成一个 block，同一个 block 中的 threads 可以同步，也可以通过 shared memory 通信。

**线程束 warp**  
GPU 执行程序时的调度单位，SM 的基本执行单元。目前在 CUDA 架构中，warp 是一个包含 32 个线程的集合，这个线程集合被 “编织在一起” 并且 “步调一致” 的形式执行。同一个 warp 中的每个线程都将以不同数据资源执行相同的指令，这就是所谓 SIMT 架构(Single-Instruction, Multiple-Thread，单指令多线程)。  
![](https://img-blog.csdnimg.cn/2020030912130875.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_0,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEyODc1OA==,size_16,color_FFFFFF,t_0)  
![](https://img-blog.csdnimg.cn/20200309121553650.png)

CUDA 的内存模型
----------

**SP**：最基本的处理单元，streaming processor，也称为 CUDA core。最后具体的指令和任务都是在 SP 上处理的。GPU 进行并行计算，也就是很多个 SP 同时做处理。

**SM**：GPU 硬件的一个核心组件是流式多处理器（Streaming Multiprocessor）。SM 的核心组件包括 CUDA 核心、共享内存、寄存器等。SM 可以并发地执行数百个线程。一个 block 上的线程是放在同一个流式多处理器（SM）上的，因而，一个 SM 的有限存储器资源制约了每个 block 的线程数量。在早期的 NVIDIA 架构中，一个 block 最多可以包含 512 个线程，而在后期出现的一些设备中则最多可支持 1024 个线程。一个 kernel 可由多个大小相同的 block 同时执行，因而线程总数应等于每个块的线程数乘以块的数量。  
![](https://img-blog.csdnimg.cn/20200309122037319.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_0,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEyODc1OA==,size_16,color_FFFFFF,t_0)  
![](https://img-blog.csdnimg.cn/2020030912184833.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_0,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEyODc1OA==,size_16,color_FFFFFF,t_0)  
一个 kernel 实际会启动很多线程，这些线程是逻辑上并行的，但是网格和线程块只是逻辑划分，SM 才是执行的物理层，在物理层并不一定同时并发。原因如下：

1.  当一个 kernel 被执行时，它的 gird 中的 block 被分配到 SM 上，一个 block 只能在一个 SM 上被调度。SM 一般可以调度多个 block，这要看 SM 本身的能力。有可能一个 kernel 的各个 block 被分配至多个 SM 上，所以 grid 只是逻辑层，SM 才是执行的物理层。
2.  当 block 被划分到某个 SM 上时，它将进一步划分为多个 wraps。SM 采用的是 SIMT，基本的执行单元是 wraps，一个 wrap 包含 32 个线程，这些线程同时执行相同的指令，但是每个线程都包含自己的指令地址计数器和寄存器状态，也有自己独立的执行路径。所以尽管 wraps 中的线程同时从同一程序地址执行，但是可能具有不同的行为，比如遇到了分支结构，一些线程可能进入这个分支，但是另外一些有可能不执行，它们只能死等，因为 GPU 规定 warp 中所有线程在同一周期执行相同的指令，线程束分化会导致性能下降。

综上，SM 要为每个 block 分配 shared memory，而也要为每个 warp 中的线程分配独立的寄存器。所以 SM 的配置会影响其所支持的线程块和线程束并发数量。所以 kernel 的 grid 和 block 的配置不同，性能会出现差异。还有，由于 SM 的基本执行单元是包含 32 个线程的 warp，所以 block 大小一般要设置为 32 的倍数。