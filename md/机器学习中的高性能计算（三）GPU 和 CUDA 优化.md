> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/410757610)

### **1 GPGPU 简介**

在上两篇中，我们使用多种技术提高了 Blur 算子的速度，效果最好的技术是利用多核和 SIMD 指令集，通过并行计算来提升速度。这给我们一个很大的启发，如果我们的 CPU 能扩展到数百数千甚至数万的核，不就能进一步加速了吗？事实上，传统超算就是这么干的，比如我国的[神威 - 太湖之光](https://link.zhihu.com/?target=http%3A//www.nsccwx.cn/swsource/5d2fe23624364f0351459262)。但是传统超算的做法是一种分布式计算的思路，即用数十个机柜，每个机柜内放置数百个计算单元（看成一台服务器），每个计算单元有几个 CPU，以此组成一个巨大的 CPU 集群。集群间通过高速网络，配合上分布式的编程来使用整个集群的数以万记的 CPU 执行计算任务。

之所以用分布式计算的思路提高整个系统的并行计算的能力，主要原因是 CPU 的发展已面临物理定律的限制，人们无法在有限的成本下创造出核心数更多的 CPU。此时人们发现，原本用来做图形渲染的专用硬件，GPU（Graphic Processing Unit），非常适合用于并行计算。

我个人认为原因有三点：首先，GPU 在渲染画面时需要同时渲染数以百万记的顶点或三角形，因此 GPU 硬件设计时就利用多种技术优化并行计算 / 渲染的效率；第二，GPU 作为专用的硬件，只负责并行计算 / 并行渲染这一类任务，不承担复杂的逻辑控制和分时切换等任务，可更专注于计算本身，通过专用指令集和流水线等技术，优化计算效率和吞吐；最后，GPU 作为一类外插设备，在尺寸、功率、散热、兼容性等方面的限制远远小于 CPU，让 GPU 可以有更大的缓存、显存和带宽，可以放置更多的计算核心，而这进一步体现到成本上的优势（成本对于技术的普及非常重要）。

![](https://pic1.zhimg.com/v2-95c85a2e6ee5d3f4013cb1136cb31fec_b.jpg)

上图是 CPU 和 GPU 的一个对比示意图。由于 CPU 除了计算任务外，还需要负责大量的逻辑控制，分时调度，兼容多种指令集等等 “包袱”，真正用来计算的部分其实很少。而 GPU 则不然，GPU 可以在更大的自由度下，放置更多的计算核心。

其实从最本质的角度来讲，GPU 之所以适合并行计算场景，是因为 GPU 使用的是 SIMT（Single Instruction Multiple Threads）模型。SIMT 可以看成是 SIMD（Single Instruction Multiple Data）模型的一种增强。

Nvidia 公司在 2000 年推出第一款 GPU，但是直到 2006 年才把 GPU 用于通用计算任务，其转折点就是 CUDA（Compute Unified Device Architecture）的出现。CUDA 是一整套软件技术，把 Nvidia GPU 的计算能力封装成一套编程规范和编程语言接口，同时提供相应的编译器、调试器等工具，这使得程序员能方便的使用 GPU 并行计算能力，执行通用的计算任务，因此又被称为 GPGPU（General-purpose computing on graphics processing units）。在 CUDA 出现 6 年之后，Alex Krizhevsky 和其老师 Geoffrey Hinton 在 CUDA 的帮助下成功地实现了 AlexNet 深度神经网络，一举打开了此轮人工智能浪潮的大门。AlexNet 论文中提到：模型的深度对于提高算法性能至关重要，但是其计算成本很高，GPU 的使用让深度神经网络的训练具有可行性。

甚至可以武断地说，如果没有 GPU 和 CUDA，尤其是后者，以深度神经网络为主要代表的人工智能算法很可能是另外一种命运。当然，Nvidia 并不是唯一一家生产 GPU 和提供上层编程框架的公司，除了 Nvidia GPU 和 CUDA 之外，Intel、AMD 和最近几年爆火的如寒武纪、海思等公司，也推出了不同的硬件和软件平台，但是无论完善度、丰富性尤其是生态上，Nvidia 仍然是毫无疑问的王者。因此这篇文章介绍的还是 Nvidia 的 GPU，下文中提到的 GPU，没有特殊说明都代指 Nvidia GPU。

![](https://pic3.zhimg.com/v2-0e77ee140aaa2289878bc7b4cbc1ec7e_b.jpg)

上图是 Nvidia 提供的一整套软硬件平台。Nvidia 针对嵌入式端，桌面级消费卡、专业工作站和数据中心提供了不同的硬件产品，但是在软件层，通过 CUDA 抽象成了统一的编程接口并提供 C/C++/Python/Java 等多种编程语言的支持。CUDA 的这一层抽象非常重要，因为有了这层抽象，向下开发者的代码可在不同的硬件平台上快速迁移；向上，在 CUDA 基础上封装了用于科学计算的 cuBLAS（兼容 BLAS 接口），用于深度学习的 cuDNN 等中间件和代码库。这些中间件对于 Tensorflow，PyTorch 这一类的深度学习框架非常重要，可以更容易地使用 CUDA 和底层硬件进行机器学习的计算任务。

### **2 GPU 硬件架构**

对于一般开发者而言，大多数情况下接触到的都是 CUDA 层。但是由于 GPU 的特殊性，为了能正确高效使用 CUDA，非常有必要学习一下 GPU 的硬件架构。由于 Nvidia 的 GPU 发展非常迅速，平均 1-2 年就会推出一款新 GPU 或核心架构，因此这里简单介绍下，不涉及硬件细节（主要是我也不太懂硬件）。

![](https://pic2.zhimg.com/v2-cf5c794f143ea9d5da205a72b365a931_b.jpg)

我们以 Nvidia 2020 年推出的 Ampere 架构和 A100 GPU 为例（A10、A30、RTX3090 等型号也是 Ampere 架构）为例。上图是整个 A100 的硬件架构，从上到下，其主要组成部分：

*   PCIe 接口层，目前大部分 GPU 都是通过 PCIe 以外部设备的方式集成到服务器上；
*   GigaThread Engine with MIG 是 GPU 用于调度资源的引擎，Ampere 架构还支持了 MIG（Multiple Instance GPU）控制即允许对 GPU 进行硬件隔离切分；
*   中间占据面积最大的绿色部分是 GPU 的核心计算单元，共有 6912 个 CUDA Cores，这是重点部分，我们在后面详细介绍；
*   中间蓝色部分是 L2 缓存，在 GPU 层面提供大于 40MB 的共享缓存；
*   最下层的 NVLink 是用于多个 GPU 间快速通信的模块（在分布式训练中非常重要）；
*   两侧的 HBM2（High Bandwidth Memory）即我们通常说的显存。A100 提供了高达 1.56TB 带宽，40GB/80GB 的显存；

其中最重要的绿色部分是计算核心，在 A100 中这些计算单元又进一步被 “分组” 成了 GPC（Graphic Processing Cluster）- TPC（Texture Processing Cluster） - SM（Streaming Multiprocessor）几个概念。最需要了解的是 SM 的架构，把其中一个 SM 放大：

![](https://pic2.zhimg.com/v2-fc91131608e842f7fe5e807142128ced_b.jpg)

SM 可以看成是 GPU 计算调度的一个基本单位，只有对其架构和硬件规格有一定认识，才能更高效地利用 GPU 的硬件能力。SM 中自上而下分别有：

*   L1 Shared Memory，在 SM 层面提供 192KB 的共享缓存
*   接下来是四个 PB（Process Block），每个 PB 内部分别有

*   L0 缓存
*   Warp Scheduler 用于计算任务调度
*   Dispatch Unit 用于计算任务发射
*   Register File 寄存器
*   16 个专用于 INT32 计算的核心
*   16 个专用于 FP32 计算的核心
*   8 个专用于 FP64 计算的核心
*   4 个 Tensor Core，这是受 Google TPU 的影响，专门设计用于执行 A*B+C 这类矩阵计算的硬件单元，甚至还支持稀疏矩阵计算，Tensor Core 对深度学习的加速非常明显
*   8 个用于 LOAD/STORE 数据的硬件单元
*   4 个 SFU（Special Function Unit）用于执行超越函数、插值等特殊计算

因此一个 SM 内就有 64 个用于计算的核心，Nvidia 称之为 CUDA Core（但是我没搞明白 64 是怎么算出来的）。而一整块 A100 GPU 则密密麻麻地放置了 108 个 SM，总计 6912 个 CUDA Core，可以简单理解成这是一个拥有 6912 个核心的巨型 CPU！

至此，我们应该能搞懂为什么 GPU 比 CPU 更适合执行并行计算了，就是 “专业人干专业事，大力就会出奇迹 “嘛。

### **3 CUDA 基本概念**

为了方便地使用 GPU 的硬件能力，Nvidia 在软件层做了抽象，形成软件上的一系列逻辑概念，这些逻辑概念跟 GPU 的硬件有一定的对应关系。

![](https://pic1.zhimg.com/v2-dcc0f678850d5bf1683753c34ca4b308_b.jpg)

如上图，CUDA 编程中的最小单元称之为 Thread，可以简单认为一个软件 Thread 会在一个硬件 CUDA Core 中执行，而 Thread 中执行的内容或函数称之为 Kernel。多个相同的 Thread 组成一个 Thread Block，软件 Thread Block 会被调度到一个硬件 SM 上执行，同一个 Thread Block 内的多个 Thread 执行相同的 kernel 并共享 SM 内的硬件资源。而多个 Thread Block 又可以进一步组成一个 Grid，一个软件 Grid 可以看成一次 GPU 的计算任务，被提交到一整个 GPU 硬件中执行。这几个概念非常重要，简单总结下：

*   kernel：Thread 执行的内容 / 代码 / 函数
*   Thread：执行 kernel 的最小单元，被 SM 调度到 CUDA Core 中执行（其实还有一个 Warp 的概念，为了简单，这里先略过）
*   Thread Block：多个 Thread 组合，GPU 任务调度的最小单元（这个描述不太准确，应该是 Warp，为了简单暂时先不细究），被调度到 SM 中执行。一个 SM 可以同时执行多个 Thread Block，但是一个 Thread Block 只能被调度到一个 SM 上。
*   Grid：多个 Thread Block 的组合，被调度到整个 GPU 中执行

同时，Thread、Thread Block 和 Grid 由于所处层次不同，他们可以访问的存储资源也不同。如 Thread 只能访问自身的寄存器，Thread Block 可以访问 SM 中的 L1 缓存，而 Grid 则可以访问 L2 缓存和更大的 HBM 显存。在第一篇文章中我们就介绍过不同层次的存储其访问速度往往是数量级的差别，GPU 也不例外，在后续的文章中我们会看到，针对 CUDA 的优化很大一部分就是如何正确高效的使用 GPU 中的多级存储来提高 GPU 的方寸比，从而进一步提高 GPU 的计算效率。

![](https://pic2.zhimg.com/v2-6456af75530956da6bc5bab7418ff9e5_b.jpg)

到底如何使用 CUDA？由于 GPU 是作为一类外挂设备，通过 PCIe 之类的接口插到服务器上，因此在 CUDA 编程中，称 GPU 为 Device，而服务器上的 CPU 和内存统称为 Host。在编写 CUDA 代码时，最主要的工作就是编写 kernel 函数，然后利用 CUDA 提供的接口，把 kernel 函数从 Host 发射到 Device 中执行，Device 则从 Grid → Thread Block → Warp → Thread 一层层的调度下最终完成 kernel 的计算。

在计算开始前，还需要用 CUDA 接口把计算要用到的数据从 Host 拷贝到 Device 中。Device 完成计算后，则需要利用 CUDA 接口把计算结果从 Device 拷贝回 Host 中，因此总结下来，CUDA 编程分为三步：

1.  从 Host 拷贝数据到 Device
2.  把需要 Device 执行的 kernel 函数发射给 Device
3.  从 Device 拷贝计算结果到 Host

如果有点抽象，我们看个例子。

### **4 CUDA HelloWorld**

CUDA HelloWorld 代码很简单，求两个数组 A 和 B 的和。代码分两部分，首先实现上面介绍的三步流程。

```
#include <cuda.h>
 
// Host code
int main() {
  int N = 1024;
  size_t size = N * sizeof(float);
 
  // Host端申请内存
  float* h_A = (float*)malloc(size);
  float* h_B = (float*)malloc(size);
  float* h_C = (float*)malloc(size);
 
  // Device端申请显存，分别用来存放A、B和结果C
  float* d_A, d_B, d_C;
  cudaMalloc(&d_A, size);
  cudaMalloc(&d_B, size);
  cudaMalloc(&d_C, size);
 
  // 把需要计算的结果从Host拷贝到Device
  cudaMemcpy(d_A, h_A, size, cudaMemcpyHostToDevice);
  cudaMemcpy(d_B, h_B, size, cudaMemcpyHostToDevice);
 
  // 执行kernel，需要指定两个参数
  int threadsPerBlock = 256;
  int blocksPerGrid = (N + threadsPerBlock - 1) / threadsPerBlock;
  VecAdd<<<blocksPerGrid, threadsPerBlock>>>(d_A, d_B, d_C, N);
 
  // 把计算结果拷贝回Host
  cudaMemcpy(h_C, d_C, size, cudaMemcpyDeviceToHost);
 
  // 释放Device中的显存
  cudaFree(d_A);
  cudaFree(d_B);
  cudaFree(d_C);
 
  // 释放Host的内存
  ...
}


```

代码比较直白，其中 cuda 开头的函数都是 CUDA 库中提供的，比如 cudaMalloc 会在 Device 中申请一段显存，cudaMemcpy 则可以在 Host 和 Device 之间进行数据拷贝等，不再赘述。

核心是 kernel 函数部分，先看看 kernel 函数的实现：

```
__global__ void VecAdd(float* A, float* B, float* C, int N) {
  int i = blockDim.x * blockIdx.x + threadIdx.x;
  if (i < N) C[i] = A[i] + B[i];
}

```

Kernel 函数只有两行并不复杂，但是需要好好解释。

这个 kernel 函数把数组 A 和数组 B 逐元素相加，结果写到数组 C 中。即 C[i] = A[i] + B[i]。如果在 CPU 上编程，一个循环遍历就搞定了。但是当用 CUDA 编程时，我们希望利用 GPU 中数千个计算核心同时计算，每个核心只负责 A 和 B 的一个元素。但是，如何分配任务？

如果 GPU 中的每个核心都有唯一的 ID，比如 0 号核心执行 C[0] = A[0] + B[0]，127 号核心执行 C[127] = A[127] + B[127] 就好了。但是如果我们要操作的数据大于 GPU 的核心数怎么办？因此 CUDA 做了一层逻辑抽象。当一个计算任务在 GPU 中执行时，这个任务相关的 Grid、Thread Block 和 Thread 都有一系列的身份标识，即几个全局变量：

*   blockDim：表明 kernel 所在的 Thread Block 的尺寸，包含 x, y, z 三个维度
*   blockIdx：表明 kernel 所在的 Thread Block 的 index，包含 x, y, z 三个维度
*   threadIdx：表明 kernel 在在的 Thead 在 Thread Block 内的 idx，包括 x, y, z 三个维度

只要引入了 cuda.h 的头文件，在代码中可以直接使用上面几个全局变量，这几个全局变量的值会在执行时由 CUDA 自动维护更新。因此，kernel 函数 i 的计算方法就是用 blockDim.x 乘以 blockIdx.x 再加上 threadIdx.x，就可以算出当前一个唯一的、逻辑 ID 被当前的 thread 使用。那为什么只用到了 x 维度，没有用到 y 和 z 维度？

答案在于 Host 端代码中，在启动 kernel 时，代码中指定了两个模板类参数：blocksPerGrid 和 threadsPerBlock。其含义通过名字就能看懂。这两个参数被我们指定成两个 int 型数字。

```
int threadsPerBlock = 256;
int blocksPerGrid = (N + threadsPerBlock - 1) / threadsPerBlock;
 
VecAdd<<<blocksPerGrid, threadsPerBlock>>>(d_A, d_B, d_C, N);

```

其实这两个参数还可以是 dim2 或 dim3 类型。如果是 dim2 类型，那么逻辑 ID 可以通过逻辑 x 和逻辑 y 唯一确定，其计算方法如下。

```
int y = blockIdx.y * blockDim.y + threadIdx.y;
int x = blockIdx.x * blockDim.x + threadIdx.x;
int i = blockDim.y * y + x;

```

![](https://pic4.zhimg.com/v2-9e6a1f3616a8d2f8997f2911676d74c3_b.jpg)

原理可以参考上图。这与二维数组索引转换成一维数组索引的过程是类似的，相信有基本编程经验的同学很快就能搞懂。当然这里我们只是举了两个最简单的例子，在实际编程中这块会更复杂一些。网上关于这部分的内容比较多，大家可以自行学习。

PS：原本想介绍如何用 CUDA 继续优化 Blur 算子，结果花太多篇幅介绍基础知识，越发觉得自己废话有点多，有点碎碎念。

那就下篇继续吧。。。

### **参考资料**

[Programming Guide :: CUDA Toolkit Documentation (nvidia.com)](https://link.zhihu.com/?target=https%3A//docs.nvidia.com/cuda/cuda-c-programming-guide/index.html)

[NVIDIA-ampere-GA102-GPU-Architecture-Whitepaper-V1.pdf](https://link.zhihu.com/?target=https%3A//images.nvidia.com/aem-dam/en-zz/Solutions/geforce/ampere/pdf/NVIDIA-ampere-GA102-GPU-Architecture-Whitepaper-V1.pdf)

[NVIDIA GPU 架构演进 | Chenfan Blog (jcf94.com)](https://link.zhihu.com/?target=https%3A//jcf94.com/2020/05/24/2020-05-24-nvidia-arch/)

[英伟达 GPU 架构演进近十年，从费米到安培 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/413145211)

[CUDA 微架构与指令集（4）- 指令发射与 warp 调度 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/166180054)

[Microsoft Word - CUDA PG 1 1 chs.doc (nvidia.cn)](https://link.zhihu.com/?target=https%3A//www.nvidia.cn/docs/IO/51635/NVIDIA_CUDA_Programming_Guide_1.1_chs.pdf)

[GPU CUDA 编程中 threadIdx, blockIdx, blockDim, gridDim 之间的区别与联系 - 华为云 (huaweicloud.com)](https://link.zhihu.com/?target=https%3A//www.huaweicloud.com/articles/13638082.html)