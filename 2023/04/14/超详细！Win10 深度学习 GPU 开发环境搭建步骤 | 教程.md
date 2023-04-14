> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/131600460)

作者 / 编辑 / 配图 | 橘子 来源 | 橘子 AI 笔记（ID：datawitch）

如果从现在开始决定学习深度学习，写代码、搭建自己的模型，那么准备开发环境将是你艰难的第一步。橘子当时搜必应、查谷歌，安装攻略没少看，结果真到自己上手实践的时候，依然碰到了不少问题。

本文将尝试站在萌新的视角，尽力还原这一过程中可能碰到的种种问题，希望它能指引着你顺利完成配置过程、帮你省下一些宝贵的时间。

**01. 硬件准备**
------------

一个完整的深度学习 GPU 开发环境需要硬件和软件两方面的支持，在**硬件部分**，我将从 **GPU、CPU、散热、主板、电源、内存、硬盘和显示器**几方面分别介绍。

*   **GPU**

深度学习需要进行大量矩阵运算，如果不考虑云端服务提供的 GPU 或 TPU 资源，一块足够好的 GPU 就是普通人的性价比之选。

目前最常用的是 **NVIDIA 显卡**，主要关注的 GPU 性能参数是**显存**和 **CUDA 计算能力**。

显存关系到你能训练多大的深度学习模型。**8-11GB 的显存对现阶段而言是够用的**，预算充足的前提下，建议选择更高的显存。

显卡的 [CUDA 计算能力](https://link.zhihu.com/?target=https%3A//developer.nvidia.com/cuda-gpus%23compute)会影响模型训练的速度。不同版本的 TensorFlow 要求不同，**最新版本的最低要求是 3.5 以上**。

2020 年 [Lambda 推荐的显卡型号](https://link.zhihu.com/?target=https%3A//lambdalabs.com/blog/choosing-a-gpu-for-deep-learning/)有：

*   **RTX 2060 (6 GB)**: 如果你只是想在空闲时间玩玩深度学习模型。
*   **RTX 2070/2080 (8 GB)**: 如果你是认真想做深度学习，但预算又非常有限。8 GB 的显存能满足大多数模型的需求。
*   **RTX 2080 Ti (11 GB)**: 如果你是认真想做深度学习，并且预算稍高一些。RTX 2080 Ti 比 RTX 2080 提速 40%。
*   **Titan RTX/Quadro RTX 6000 (24 GB)**: 如果你需要频繁使用当下最好的深度学习模型，但没有足够的预算购买 RTX 8000。
*   **Quadro RTX 8000 (48 GB)**: 你在为未来投资！甚至有可能幸运地在今年就研究出下一个最先进的深度学习模型。

你也可以选择购买二手的 **GTX 1080ti (11 GB)**。

*   **CPU**

首先要明确的是，相比于 GPU，深度学习并不那么需要高性能的 CPU，在运行深度神经网络时 CPU 承担的运算量很少，所以**挑选 CPU 不是越贵越好**。

在深度学习任务中，CPU 主要承担的工作是**数据预处理**。为了不让这一步成为瓶颈，至少保证**每个 GPU 能对应 4 个 CPU 线程**，CPU **主频最好在 3.6 GHz 以上**。

*   **散热**

不可忽视的环节。一般 GPU 温度达到 80℃就会触发保护、降低性能。因此无论是选择风冷还是水冷，**必须要配备散热系统**。

另外，由于 NVIDIA 显卡首先都是游戏显卡，已经针对 Windows 操作系统做了优化，可以很方便地在 Windows 系统中更改风扇相关选项。而大部分深度学习库都是为 Linux 操作系统写的。这一矛盾我将在之后提供一种解决方案。

*   **主板**

没什么特别的，**保证有足够的 PCIe 端口**。

*   **电源**

将 GPU 和 CPU 的功率相加乘以 110% 就是最低要求。保证有足够的 PCIe 端口。

*   **内存**

记住一个原则：**内存应该大于显存**。

理想情况是配备 32 GB 及以上的内存，如果预算有限，那么 16 GB 也可以接受。

*   **硬盘**

主要关注的性能参数是**存储容量**和**读取速度**。

其中，存储容量决定了你能用多大的数据集，**读取速度会影响到训练过程中的 I/O 操作**。

固态硬盘（SSD）的读取速度显著超过普通机械硬盘，建议选择一块 256 GB 以上的 SSD。

*   **显示器**

既然是做数据分析而不是做设计，没什么特殊要求，够用就行。而根据经验，多一个显示器可以显著提升你的工作效率。

橘子目前使用的工作站配备了 Intel Xeon CPU，NVIDIA GTX 1080ti (11 GB) GPU，16 GB 内存以及 256 GB 的 SSD。

这个配置大概是什么概念呢？

在这台电脑上训练一个拥有 10,484,994 参数的深度卷积神经网络，大致需要 7 分 38 秒。

**02. 软件准备**
------------

假设你已经有了足够好的电脑，接下来需要安装：

*   **MinGW-w64** 和 **MSYS2** – 用于在 Windows 下搭建类 Unix 环境（也就是上文提到的解决方案：Windows、Linux，一个都不能少~）
*   **NVIDIA 显卡驱动** – 允许系统使用 GPU 带来的运算加速
*   **Visual Studio** 和 **Windows 10 SDK** – CUDA 需要，如果不提前装好，CUDA 安装程序也会在最后一步提醒你的
*   **CUDA Toolkit** – GPU C 语言库，为高性能 GPU 加速应用程序提供开发环境
*   **cuDNN** – 基于 CUDA，深度学习使用的 GPU 加速基元库
*   **Python** 和 **pip** – 机器学习领域最常用的编程语言及其包管理工具
*   **Pytorch/TensorFlow** – 两个都是主流的深度学习框架，可以二选一

为了以后更好的编程体验，你还会需要：

*   **Sublime Text** – 文本编辑器，用来写代码和看代码，换一个同类的也可以
*   **Jupyter Notebook** – 以网页的形式打开，可以在浏览器页面中直接写代码、运行代码，运行结果也会直接在代码块下显示

*   **安装 MinGW-w64 和 MSYS2**

我想了想决定把这一步提到最前面。因为一旦类 Unix 环境配置好，后续操作就可以在 Powershell 中无缝使用 Linux 系统的常用命令，体验非常顺畅。

Win 10 (64 位) 的用户请在 [MSYS2 官网](https://link.zhihu.com/?target=https%3A//www.msys2.org/)下载页面选择 msys2-x86_64 的安装包。

![](https://pic2.zhimg.com/v2-4fae488c18f93887644281f717034309_b.jpg)

下载安装完成之后，打开 MSYS2，在窗口中输入以下命令安装：

```
pacman -S mingw-w64-x86_64-gcc

```

**（可选）**以上一步安装的 gcc 为例**，**如果还需要安装其他 Linux 库，可以先输入以下命令查询，mingw-w64-x86_64-gcc 可替换为其他任何库的名称：

```
pacman -Ss mingw-w64-x86_64-gcc

```

查询之后安装命令同前。

在搜索框中搜索「编辑系统环境变量」，「系统属性」–「高级」中选择「环境变量」

![](https://pic3.zhimg.com/v2-ebca0e6e81b7e0ec8c3aa8059c93546e_b.jpg)

将 MSYS2 相关的三个目录（如图）添加到「系统变量」的「Path」中。

![](https://pic3.zhimg.com/v2-1b07aa5cc728a6b26539f78ab1b550ca_b.jpg)

这样就可以在 Powershell 中使用 Linux 命令了。

*   **安装 NIVIDIA 显卡驱动**

在 [NVIDIA 官网](https://link.zhihu.com/?target=https%3A//www.nvidia.cn/Download/index.aspx%3Flang%3Dcn)可以按照 GPU 型号下载最新的显卡驱动程序。如果已经安装，强烈建议你在开始下一步之前先检查更新。

![](https://pic1.zhimg.com/v2-a2bd9bff8beda76358a63783899d8584_b.jpg)

在开始菜单图标上右键「Windows Powershell (管理员)」，**以管理员身份打开** Powershell，进入 nvidia-smi 所在目录：

```
cd ‘C:\Program Files\NVIDIA Corporation\NVSMI’

```

以后打开 Powershell 都默认管理员，不再重复。

运行「nvidia-smi」命令：

```
.\nvidia-smi

```

可以看到当前显卡的型号、显存、驱动版本以及正在使用显卡的进程。后续安装了 CUDA 之后，也可以看到 CUDA 版本。

![](https://pic3.zhimg.com/v2-ceb7a10beedb05ebc3396cd2710c317a_b.jpg)

如果这个命令不起作用，请先检查你的显卡驱动有没有安装好。

*   **Visual Studio 和 Windows 10 SDK**

在开始安装 CUDA 之前，先检查一下自己电脑上有无 Visual Studio 和 Windows 10 SDK。在 [VS 官网](https://link.zhihu.com/?target=https%3A//visualstudio.microsoft.com/)选择 Community 版本。

![](https://pic1.zhimg.com/v2-9dd5a4d961648ff1c166d04e6ab44278_b.jpg)

安装程序还会进行两次下载，需要耐心等待。

![](https://pic4.zhimg.com/v2-220f13c48a9b447d1067936b8ca5e3b3_b.jpg)

两次下载完成后，为了后续安装和测试 CUDA，「使用 C++ 的桌面开发」和「Windows 10 SDK」两项是必须安装的，其他组件可以看个人需要。

![](https://pic3.zhimg.com/v2-41806042ab50433c2216e6017c7e1f4e_b.jpg)

*   **安装 CUDA Toolkit**

根据当前显卡驱动版本，选择对应的 [CUDA 版本](https://link.zhihu.com/?target=https%3A//docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html)。最新版本的 TensorFlow (>= 2.1.0) 支持 CUDA 10.1，需要显卡驱动版本高于 418.x。

![](https://pic4.zhimg.com/v2-aabed68ae04fce81b7aef439d7874f2f_b.jpg)

[这里](https://link.zhihu.com/?target=https%3A//developer.nvidia.com/cuda-toolkit-archive)列出了所有 CUDA 版本的下载链接，以 CUDA 10.1 为例下载本地安装包。

![](https://pic3.zhimg.com/v2-1915739ffb7c2e537c795ed08b83b5ce_b.jpg)

双击下载好的文件，安装过程会自动开始。

![](https://pic2.zhimg.com/v2-9619bab275ab35f32fbcca7cf65dca89_b.jpg)

安装过程中，软件协议选择「同意并继续」

![](https://pic2.zhimg.com/v2-adb5e36028d33cc5d7c6766fe56b2d89_b.jpg)

选择默认的「精简」模式，然后下一步。

![](https://pic1.zhimg.com/v2-f0c6847a14edeb0fe432544bc6330768_b.jpg)

如果跳出 Windows 安全中心的提示，继续「安装」

![](https://pic2.zhimg.com/v2-bf00214a20579763701462752fb4f8c5_b.jpg)

最后一步会检查 Visual Studio，如果前面已经安装好，就没有问题，按要求重启电脑。

![](https://pic4.zhimg.com/v2-66cb55a87c26331ac94c2e7c343a94af_b.jpg)

正常安装完成后，CUDA 的目录应该已经存在于系统环境变量「Path」中，为了保险可以检查一下。

![](https://pic4.zhimg.com/v2-d5cf216c144bbde9fc65d53ef5107f0b_b.jpg)

打开图中目录，根据你的 Visual Studio 版本打开对应的文件，比如我选择「nbody_vs2017.sln」

![](https://pic2.zhimg.com/v2-46f24bb1f702125a23285fbd4548ea05_b.jpg)

注意「ProgramData」可能是隐藏文件夹，在「文件」–「查看」里勾选「隐藏的项目」就显示了。

![](https://pic3.zhimg.com/v2-8e3b512a35a7b215ed4b4de5d4a5e88e_b.jpg)

选中「nbody」，菜单栏中选生成解决方案。

![](https://pic3.zhimg.com/v2-00fe812d82482bf73d3e8e6cacbd02c2_b.jpg)

在「输出」栏中得到下图结果说明生成成功。

![](https://pic3.zhimg.com/v2-28259b1d10fa895e4d240698b5e99b6e_b.jpg)

在 Powershell 中输入以下命令打开 nbody 目录：

```
cd ‘C:\ProgramData\NVIDIA Corporation\CUDA Samples\v10.1\bin\win64\Debug’

```

运行 nbody：

```
.\nbody.exe

```

运行成功结果如图，可以看到 CUDA 使用的 GPU。

![](https://pic1.zhimg.com/v2-ea7f271e05fe16fc36fe3b3d7234660c_b.jpg)

在 [CUDA 的官方文档](https://link.zhihu.com/?target=https%3A//docs.nvidia.com/cuda/)中有更详细的安装步骤说明。

*   **安装 cuDNN**

接下来安装 cuDNN。进入 [cuDNN 网站](https://link.zhihu.com/?target=https%3A//developer.nvidia.com/cudnn)页面并选择「download」之后会要求登录。

![](https://pic3.zhimg.com/v2-16e97c08cdcff2139a69c942488763ce_b.jpg)

注册一个 NVIDIA 账号即可。

![](https://pic2.zhimg.com/v2-9266949fc910f5b021b21da3cedd3231_b.jpg)

按照当前 CUDA 版本选择对应的 [cuDNN 版本](https://link.zhihu.com/?target=https%3A//developer.nvidia.com/rdp/cudnn-download)下载。

![](https://pic3.zhimg.com/v2-5703c20291e40a94ba48b29c86557c0a_b.jpg)

将下载好的压缩包解压，将图示文件夹中的文件分别替换到 CUDA 目录下对应的文件夹中（这步操作需要管理员权限）

![](https://pic3.zhimg.com/v2-a6dd1bf75b6d7ac7d3bfb4edda7a029a_b.jpg)

在 Powershell 中检查 cuDNN 版本需要两步：

```
cd ‘\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v10.1\include’
cat cudnn.h | grep CUDNN_MAJOR -A 2

```

运行结果中对应的三个数字就是 cuDNN 的版本号，如图 cuDNN 版本为 7.6.5。

![](https://pic3.zhimg.com/v2-b7f96137b16ee063d52d81b276503762_b.jpg)

*   **安装 Python 和 pip**

在 [Python 官网](https://link.zhihu.com/?target=https%3A//www.python.org/downloads/windows/)下载所需安装包，需要哪个版本可以看深度学习框架的要求，比如 TensorFlow 2.0 要求 Python 版本为 3.5-3.7。

![](https://pic2.zhimg.com/v2-20006414d652e3014eae7d46d7d6ab29_b.jpg)

我安装的是 Python 3.6.7。

![](https://pic4.zhimg.com/v2-0817bede954b9b0733da5079465d767f_b.jpg)

完成后在 Powershell 中输入「python」，如果出现如下界面说明安装成功，quit() 退出。

![](https://pic2.zhimg.com/v2-9cb045f0fe5d1c9f3860870d777a3c39_b.jpg)

在 [pip 官网文件列表](https://link.zhihu.com/?target=https%3A//pypi.org/project/pip/%23files)中下载. tar.gz 安装包。

![](https://pic3.zhimg.com/v2-505aed933e026a4606012e12b5cdcfa6_b.jpg)

在 Powershell 中，先用「cd」命令打开安装包所在目录，然后输入如下命令解压：

```
gzip -dv pip-20.0.2.tar.gz
tar -xf .\pip-20.0.2.tar

```

解压之后输入如下命令，回车，将自动安装 pip：

```
python setup.py install

```

输入以下命令可查看 pip 是否可用：

```
python -m pip –version

```

*   **安装 Pytorch 或 TensorFlow**

**- Pytorch -**

在 [Pytorch 官网](https://link.zhihu.com/?target=https%3A//pytorch.org/)查询安装最新版本 Pytorch 的「pip」命令。如果需要旧版本的 pytorch 查[这里](https://link.zhihu.com/?target=https%3A//pytorch.org/get-started/previous-versions/)，在 Powershell 中输入对应的命令即可安装。

![](https://pic2.zhimg.com/v2-8241cb2a4ff6b01543c0ddad10849bad_b.jpg)

如果安装过程中因为网络问题超时报错，橘子在文末提供了适用于 Python 3.6、Pytorch 1.3.1 的. whl 文件，使用方法是在 Powershell 中「cd」到. whl 文件所在目录，然后敲以下命令：

```
pip3 install .\torch-1.3.1-cp36-cp36m-win_amd64.whl

```

安装完成后可以输入如下命令验证：

```
python -c 'import torch; print(torch.Tensor([1]))'

```

如果结果是「tensor([1.])」说明安装成功。

**- TensorFlow -**

在 TensorFlow 官网可以找到[最新版本](https://link.zhihu.com/?target=https%3A//www.tensorflow.org/install)和[以前版本](https://link.zhihu.com/?target=https%3A//www.tensorflow.org/install/pip%3Flang%3Dpython3%23older-versions-of-tensorflow)的 TensorFlow。与 Pytorch 类似，有两种安装方式，**第一种**是使用「pip」直接安装：

```
pip3 install --upgrade tensorflow

```

**第二种**是下载. whl 文件并安装，橘子使用的是适用于 Python 3.6 的、TensorFlow 2.0.0 的. whl 文件：

```
pip3 install .\tensorflow_gpu-2.0.0-cp36-cp36m-win_amd64.whl

```

安装完成后可以输入如下命令验证：

```
python -c "import tensorflow as tf; print(tf.reduce_sum(tf.random.normal([1000, 1000])))"

```

安装成功的结果如下。

![](https://pic2.zhimg.com/v2-e8a4a8796d18b3779e07a29916a20e41_b.jpg)

*   **安装 Sublime Text**

直接从 [Sublime Text 官网](https://link.zhihu.com/?target=https%3A//www.sublimetext.com/)下载安装就好。

![](https://pic2.zhimg.com/v2-fd625f66d7553bcd3b403696e0a3a3b5_b.jpg)

*   **安装及配置 Jupyter Notebook**

在 Powershell 中输入如下命令安装 Jupyter：

```
pip install notebook

```

除了[文档](https://link.zhihu.com/?target=https%3A//jupyter.readthedocs.io/en/latest/running.html%23running)中写的 Jupyter Notebook 运行方法，在 Windows 中还可以写一个. bat 文件，简单三步就可以将 Jupyter notebook 运行目录改到任何位置：

**1、**打开 Powershell 并输入如下命令，生成文件「C:\Users\lenovo\.jupyter\jupyter_notebook_config.py」记录默认配置（「lenovo」替换为你电脑中的用户名）

```
Jupyter notebook --generate-config

```

**2、**使用 Sublime Text 打开这个文件。

**3、**用快捷键「Ctrl+F」找到如下行：

```
#c.NotebookApp.notebook_dir = ‘’

```

去掉「#」注释，把你想要的目录写进去，比如在 D 盘新建一个目录叫「jupyter」

```
c.NotebookApp.notebook_dir = ‘D:/jupyter’

```

这三步操作完之后，打开 Jupyter 的默认方式是先「cd」到这个配置好的目录，然后输入「jupyter notebook」

这种方法有点麻烦，更省事的办法是写一个. bat 文件，无论这个文件放在哪，下次想要打开 Jupyter 的时候只要**以管理员身份运行**这个文件就可以了。

例如，打开 Sublime Text 新建一个文件，写入如下语句并将文件存为「open_jupyter.bat」

```
@echo off
D:
cd jupyter
jupyter notebook

```

成功在浏览器打开 Jupyter Notebook 并新建一个 Python 3 文件的效果。

![](https://pic4.zhimg.com/v2-83c3576e70b5b73f7636b4349d99dbd7_b.jpg)

**03. 最后的总结**
-------------

目前为止，橘子已经尽力回忆了安装过程中可能遇到的所有坑，但毕竟**没法面面俱到**。框架一直在更新换代，而我永远不知道哪里会出现新的问题。

如果你按照本文的步骤依然碰到了问题，我诚恳地建议你，**在开口问之前先自己去查**，因为搜索引擎和 [Stack Overflow](https://link.zhihu.com/?target=https%3A//stackoverflow.com/) 大概率会给出答案。

这里有份**重点**可以帮你避开大部分坑（敲黑板）：

*   更新 NVIDIA 显卡驱动为最新
*   始终以管理员身份打开 Powershell
*   安装 CUDA 之前检查一下电脑上有没有装好 Visual Studio 和 Windows SDK
*   根据 NVIDIA 显卡支持的 CUDA 版本选择对应的 cuDNN、Pytorch/TensorFlow
*   确定 Python 版本与 CUDA、cuDNN 和深度学习框架兼容
*   报错时优先检查 Windows 系统环境变量

> 参考资料：  
> [https://lambdalabs.com/blog/choosing-a-gpu-for-deep-learning/](https://link.zhihu.com/?target=https%3A//lambdalabs.com/blog/choosing-a-gpu-for-deep-learning/https%3A//timdettmers.com/2018/12/16/deep-learning-hardware-guide/)  
> [https://timdettmers.com/2018/12/16/deep-learning-hardware-guide/](https://link.zhihu.com/?target=https%3A//lambdalabs.com/blog/choosing-a-gpu-for-deep-learning/https%3A//timdettmers.com/2018/12/16/deep-learning-hardware-guide/)  
> [https://stackoverflow.com/questions/30069830/how-to-install-mingw-w64-and-msys2](https://link.zhihu.com/?target=https%3A//stackoverflow.com/questions/30069830/how-to-install-mingw-w64-and-msys2)