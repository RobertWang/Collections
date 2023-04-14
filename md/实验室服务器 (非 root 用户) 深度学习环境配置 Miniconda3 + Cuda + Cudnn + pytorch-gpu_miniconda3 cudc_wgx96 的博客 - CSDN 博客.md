> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_43473554/article/details/104987481)

### 目录

*   [1. 连接服务器（需要知道服务器 ip 地址）](#1_ip_1)
*   *   [如果本地是 Linux](#Linux_4)
    *   [如果本地是 Windows](#Windows_12)
*   [2. 安装 Miniconda3 或者 Anaconda3](#2_Miniconda3Anaconda3_20)
*   *   [下载安装包](#_21)
    *   [进行安装](#_26)
*   [3. 配置虚拟环境](#3__46)
*   *   [创建一个虚拟环境](#_53)
    *   [查看环境列表](#_60)
    *   [激活虚拟环境](#_66)
    *   [退出虚拟环境](#_73)
    *   [删除虚拟环境](#_79)
    *   [其他一些命令 (需要时可以查看)](#_86)
*   [4. 安装第三方库](#4__107)
*   *   [conda 安装](#conda_113)
    *   [pip 安装](#pip_118)
    *   [用 conda pip 安装很慢](#conda_pip_124)
*   [5. 安装 cuda 和 cudnn](#5_cudacudnn_141)
*   *   [安装 cuda](#cuda_142)
    *   *   [创建安装文件夹](#_161)
        *   [安装 cuda](#cuda_167)
    *   [安装 cudnn（其实是拷贝文件）](#cudnn_198)
    *   [最后配置 cuda 环境变量](#cuda_214)
    *   *   *   [查看当前 cuda 版本，可以看到是 10.0](#cuda100_227)
*   [6. 安装 pytorch-gpu](#6_pytorchgpu_238)
*   *   [安装 torchvision](#torchvision_259)

1. 连接服务器（需要知道服务器 ip 地址）
=======================

我们在跑[深度学习](https://so.csdn.net/so/search?q=%E6%B7%B1%E5%BA%A6%E5%AD%A6%E4%B9%A0&spm=1001.2101.3001.7020)代码时一般都要用 gpu 加速训练，我们自己的电脑上没有 gpu，所以需要连接实验室的服务器（服务器是 linux 系统），然后将代码传到服务器上运行，下面的方法都要保证本地电脑与服务器在一个局域网内。

如果本地是 Linux
-----------

如果你的电脑是 Linux 系统，那么恭喜你，直接用在终端输入

```
ssh 服务器ip地址

```

就可以连上服务器

如果本地是 Windows
-------------

如果你使用的是 windows，就需要下载相应的软件，一种是 MobaXterm，一种是 Xshell6（Xttp 是传输文件时要用的），选择任意一个下载就好  
![](https://img-blog.csdnimg.cn/20200517182503750.png#pic_center)  
![](https://img-blog.csdnimg.cn/20200517182607899.png#pic_center)  
在这里我选用的是上面的 MobaXterm，打开该软件后新建会话（Session），用 ssh 连接，输入服务器 Ip 地址，和你的用户名，最后点击确定  
![](https://img-blog.csdnimg.cn/20200517185310614.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDczNTU0,size_16,color_FFFFFF,t_70#pic_center)  
进入后就要输入一下自己服务器用户的密码，然后就可以进入到自己的 home / 用户名 / 下，要注意自己没有 root 权限安装一些文件，所以一般安装文件比较复杂。  
如果自己的电脑和服务器不在同一个局域网内，我们需要远程控制，可以使用 teamviewer 或者向日葵。

2. 安装 Miniconda3 或者 Anaconda3
=============================

下载安装包
-----

首先安装 Miniconda3，我之前使用的是 Ananconda3，其实 Miniconda3 与 Anaconda3 的命令相同，是一个更轻量级的 Aanaconda，安装速度也更快。  
在自己的本地电脑上下载服务器所要的 Miniconda 安装包，然后上传到服务器上进行安装。在[官网](https://docs.conda.io/en/latest/miniconda.html)或者[清华源](https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/)上下载对应服务器版本的 Miniconda3，我选择的是 linux 版本 x64_64。  
下载完成是一个. sh 文件，然后上传到自己的服务器目录下，可以在 MobaXterm 中点击上传小图标即可上传文件  
![](https://img-blog.csdnimg.cn/20200517192016901.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDczNTU0,size_16,color_FFFFFF,t_70)

进行安装
----

在服务器上进入上传的 Miniconda 目录下，用 bash 运行. sh 文件

```
bash Minconda3-latest-Linux-x86_64.sh    # 安装miniconda

```

最后 init 选择 yes，即可将 miniconda 添加到环境变量中，环境变量在. bashrc 中，是一个隐藏文件，一般在 / home / 用户名 / 目录下

```
ls -a    # 查看当前目录所有文件

```

打开. bashrc（可以使用 vi），在. bashrc 最下方我们可以看到 [conda](https://so.csdn.net/so/search?q=conda&spm=1001.2101.3001.7020) initialize 的语句块就代表成功添加环境变量  
![](https://img-blog.csdnimg.cn/20200517193252178.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDczNTU0,size_16,color_FFFFFF,t_70)  
如果要对. bashrc 进行修改，记着修改完之后运行下面的命令来激活环境变量。

```
source .bashrc # 激活环境变量

```

安装完之后目录下也多出了 miniconda3 的文件夹，

3. 配置[虚拟环境](https://so.csdn.net/so/search?q=%E8%99%9A%E6%8B%9F%E7%8E%AF%E5%A2%83&spm=1001.2101.3001.7020)
=========================================================================================================

用 conda 创建虚拟环境是 conda 很好的用法，因为每一个深度学习项目用的不同的框架，如 [pytorch](https://so.csdn.net/so/search?q=pytorch&spm=1001.2101.3001.7020)，tensorFlow，caffe 等，并且有些包的版本也有着严格要求。  
我们需要为每一个项目建立一个独立的虚拟环境，在各自的虚拟环境里安装其需要的包，这样可以保证各个代码运行的环境不产生冲突。虚拟环境的名字就是最终端前面（）里面显示的。  
我们现在可以看到终端最前面有（base）的符号，代表现在在 conda 最基本的环境中，我们可以输入

```
conda list  # 查看当前环境安装的包

```

创建一个虚拟环境
--------

创建一个名为 py37_torch 的虚拟环境，代表我运行的项目需要用 Python3.7，pytorch 框架，名字可以自行定义；最后是 Python 的版本，可以自己选择。

```
conda create -n py37_torch python=3.7   # 创建虚拟环境

```

我们可以创建多个不同的虚拟环境

查看环境列表
------

列出所有环境

```
conda env list  # 查看环境列表

```

激活虚拟环境
------

每次跑程序时，先要激活相应的虚拟环境，激活后最前面小括号中就会显示当前所在的虚拟环境的名字

```
conda activate py37_torch   # 激活虚拟环境py37_torch

```

退出虚拟环境
------

直接使用以下命令可以退出当前虚拟环境

```
conda deactivate   # 退出虚拟环境

```

删除虚拟环境
------

```
conda env remove -n py37_torch # 删除名为py37_torch的虚拟环境

```

其他一些命令 (需要时可以查看)
----------------

```
conda --version
或者conda -V
# 获取版本号

```

```
# 检查更新当前conda
conda update conda

```

#查看–安装–更新–删除包

conda search package_name# 查询包  
conda install package_name  
conda install package_name=1.5.0  
conda update package_name  
conda remove package_name

4. 安装第三方库
=========

自己的程序通常包含了许多包，我们需要将他们全部装好程序才能正确运行，首先查看现有的包

```
conda list

```

conda 安装
--------

安装第三方包时可以采用 conda 安装：

```
conda install package_name

```

pip 安装
------

我们可以在 [pypi 网站](https://pypi.org/)输入自己的包的名字查看对应的命令

```
pip install package_name

```

也可以在该网站上下载离线安装包，使用 pip 安装

用 conda pip 安装很慢
----------------

在用 conda 或者 pip 进行安装时发现速度十分地慢，我们可以添加清华镜像源，使用以下命令，添加 channel

```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/menpo/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge

conda config --set show_channel_urls yes
conda config --get channels

```

添加的 channel 可以在 / home / 用户名 / 下的. condarc 进行查看

5. 安装 cuda 和 cudnn
==================

安装 cuda
-------

安装 cuda 时踩了许多坑，说多了都是泪呀  
每个人需要安装的 cuda 版本可能都不同，要看自己的服务器 gpu 版本决定。  
看了一些教程，说要首先安装驱动什么的，然而我不是 root 用户，装驱动一般不可能，所以直接装 cuda  
首先用命令

```
nvidia-smi

```

可以看到我们的 nivdia 驱动版本和 cuda 版本  
![](https://img-blog.csdnimg.cn/20200517204057713.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDczNTU0,size_16,color_FFFFFF,t_70)  
但是我试了 cuda10.1 安装失败，安装 cuda10.0 是成功的。  
首先进入 [Nvidia 网站](https://developer.nvidia.com/cuda-toolkit-archive)下载安装包，这个页面比较难找，尽量收藏一下，之后可能还要安装其他版本的 cuda  
我们直接下载相应的 cuda 安装包进行安装

```
cat /etc/lsb-release   # 查看发行版本
arch # 查看系统架构

```

![](https://img-blog.csdnimg.cn/20200517225356364.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDczNTU0,size_16,color_FFFFFF,t_70)

### 创建安装文件夹

我们再安装 cuda 之前在 / home / 用户名 /  
下创建文件夹 cuda-10.0，一会儿将 cuda10.0 安装在此目录下，不要安装在默认路径下，因为没有权限。之后如果要安装其他版本的 cuda，可以创建多个‘’cuda - 版本‘’ 文件夹

```
mkdir cuda-10.0  # 安装之前创建安装文件夹

```

### 安装 cuda

```
sh cuda_10.0.130_410.48_linux.run

```

安装时首先一直按空格运行到 100% 然后选择

```
-----------------
Do you accept the previously read EULA?
accept/decline/quit: accept

Install NVIDIA Accelerated Graphics Driver for Linux-x86_64 430.40?
(y)es/(n)o/(q)uit: n  #不需要安装驱动

Install the CUDA 10.0 Toolkit?
(y)es/(n)o/(q)uit: y 

Enter Toolkit Location
 [ default is /usr/local/cuda-10.0 ]: /home/wanggexuan/cuda-10.0 
 #不要安装在user目录下，没有权限；安装在刚刚创建的cuda-10.0目录下

Do you want to install a symbolic link at /usr/local/cuda?
(y)es/(n)o/(q)uit: n

Install the CUDA 10.0 Samples?
(y)es/(n)o/(q)uit: n

Installing the CUDA Toolkit in /home/wanggexuan/cuda-10.0 ...

```

安装 cudnn（其实是拷贝文件）
-----------------

进入 [cudnn 网页](https://developer.nvidia.com/rdp/cudnn-archive)下载对应 cuda 版本的 cudnn，首次下载需要注册一下，用邮箱注册一下就好了  
直接选择 linux 版本  
![](https://img-blog.csdnimg.cn/20200517234553788.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDczNTU0,size_16,color_FFFFFF,t_70)  
下载好后文件名是 cudnn-10.0-linux-x64-v7.6.4.38.tgz，

1.  解压

```
tar -zxvf cudnn-10.0-linux-x64-v7.6.4.38.tgz

```

2.  拷贝解压后 cuda 文件夹中一些文件到 cuda-10.0 的文件夹中去

```
cp cuda/include/cudnn.h /home/wanggexuan/cuda-10.0/include/
cp cuda/lib64/libcudnn* /home/wanggexuan/cuda-10.0/lib64
chmod a+r cuda/include/cudnn.h cuda/lib64/libcudnn*

```

最后配置 cuda 环境变量
--------------

将 cuda 加入到环境变量中去，在. bashrc 最后添加上

```
export PATH="$PATH:/home/wgx/cuda-10.0/bin"
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/home/wgx/cuda-10.0/lib64/"
export LIBRARY_PATH="$LIBRARY_PATH:/home/wgx/cuda-10.0/lib64"


```

最后更新系统环境变量

```
source .bashrc

```

#### 查看当前 cuda 版本，可以看到是 10.0

```
nvcc -V  # 或者 nvcc --version

nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2018 NVIDIA Corporation
Built on Sat_Aug_25_21:08:01_CDT_2018
Cuda compilation tools, release 10.0, V10.0.130

```

如果我们想要换他版本的 cuda，只需要将上述环境变量. bashrc 目录换掉，然后在 source 一下就好了

6. 安装 pytorch-gpu
=================

安装 pytorch 也有许多坑  
由于 [pytorch 官网](https://pytorch.org/get-started/locally/)中最新的版本没有支持 cuda10.0 的，所以我在 [pytorch 之前版本](https://pytorch.org/get-started/previous-versions/)的页面上安装之前的 pytorch 版本  
** 注意，有坑！** 我要安装的是 pytorch1.0.0，linux，cuda100 的版本，按照下面给出的用 conda 命令装根本装不上，超级慢，最终显示 http error  
![](https://img-blog.csdnimg.cn/20200517230730738.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDczNTU0,size_16,color_FFFFFF,t_70)  
查了许多资料，说去掉 - c pytorch，添加清华镜像源下载，但是还是安装不上，显示在这几个 channel 中找不到包  
最终我在[清华镜像网站](https://mirror.tuna.tsinghua.edu.cn/help/anaconda/)上才看到  
![](https://img-blog.csdnimg.cn/20200517232057752.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDczNTU0,size_16,color_FFFFFF,t_70)  
好吧，那么现在还是老老实实下载 torch 离线包，使用离线安装，在刚刚 [pytorch 网页](https://pytorch.org/get-started/previous-versions/)上最下方可以进入对应 cuda 的下载页面，下载对应版本的 torch whl 文件![](https://img-blog.csdnimg.cn/20200517232407301.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDczNTU0,size_16,color_FFFFFF,t_70)  
我选择的是 torch1.0.0，python37，linux 版本  
![](https://img-blog.csdnimg.cn/2020051723292449.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDczNTU0,size_16,color_FFFFFF,t_70)  
下载好后，直接用 pip 进行安装

```
pip install torch-1.0.0-cp37-cp37m-linux_x86_64.whl

查看是否安装成功，进入python
> import torch  不报错
> torch.cuda.is_available()
显示True

```

安装 torchvision
--------------

我装的版本是 0.2，较高版本安装不上，torchvision 的安装是在 [pypi 网站](https://pypi.org/project/torchvision/0.2.1/#files)上用 pip 进行安装的，很方便

安装好后可以用 conda list，查看是否安装成功

```
conda list

```