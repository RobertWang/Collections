> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/621142457)

**导语**
------

2023-4-11

对于机器学习 er 配置环境一直是个头疼的事，尤其是在 windows 系统中。尤其像博主这样的懒人，又不喜欢创建虚拟环境，过段时间又忘了环境和包的人，经常会让自己电脑里装了各种深度学习环境和 python 包。长时间会导致自己的项目文件和环境弄的很乱。且各个项目间的兼容性又会出现问题。

不仅如此，windows 系统独特的 “尿性” 真的让开发者苦不堪言！

好在微软爸爸推出了 WSL，WSL 可以实现在 windows 电脑上运行 linux 系统。目前已经是越来越接近原生 linux 系统。

利用 wsl 部署深度学习训练环境，无论是从**便捷性**上还是**性能上**均有优势。

博主浏览**目前 wsl 配置深度学习环境的各种文章，采坑无数**，终于完成了最详细的教程。

目前现在很多网上的**教程都是老版本**的**，真的有很多坑**！

由于现在很多环境和软件的升级，很多老教程不必要的操作就能直接省去！

这个教程就一个字：**新**！相比以往的老教程，无需走很多弯路! 下面开始吧！

**需要在 WSL 上玩深度学习，需要以下几个条件**

*   win11，最好更新到最新版本
*   电脑上有显卡，Nvdia
*   windows 上安装显卡驱动及 CUDA 和 CuDNN（第一章）
*   安装 WSL2 （2 版本更好）
*   WLS2 安装好 Ubuntu20.04（本人之前试过 22.04，有些版本不兼容的问题，无法跑通，时间多的同学可以尝试）（第二章）

**在做好准备工作后，本文将介绍两种方法在 WSL 部署深度学习环境**

*   Docker 法（**光速部署，不需要在 WSL 内安装任何驱动，方便迁移**）（第三章）
*   wls2 本地法（**在 WSL2 内直接安装 CUDA**）（第四章）

**1、windows 显卡环境及 CUDA 安装**
---------------------------

*   安装显卡驱动  
    截止 2023 年, 4 月, 全系显卡驱动已经指出  
    [GPU in Windows Subsystem for Linux (WSL) | NVIDIA Developer](https://link.zhihu.com/?target=https%3A//developer.nvidia.com/cuda/wsl)  
    查看显卡驱动版本，及最大支持 cuda 版本，例如，如下图，本人最大支持 12.1cuda

```
nvidia-smi

```

![](https://pic4.zhimg.com/v2-bab60cbccd5e59fcf6ae7e6c9ff80523_b.jpg)

*   安装合适的 cuda 版本和 Cudnn  
    [CUDA Toolkit Archive | NVIDIA Developer](https://link.zhihu.com/?target=https%3A//developer.nvidia.com/cuda-toolkit-archive)

[cuDNN Download | NVIDIA Developer](https://link.zhihu.com/?target=https%3A//developer.nvidia.com/rdp/cudnn-download)

安装教程网上很多，推荐

[CUDA 与 cuDNN 安装教程（超详细）_kylinmin 的博客 - CSDN 博客](https://link.zhihu.com/?target=https%3A//blog.csdn.net/anmin8888/article/details/127910084)

**2、WSL 安装 Ubuntu**
-------------------

### **2.1 wsl 安装**

如果你的 windows 为 11 版本，那么安装 wsl 会非常方便，这里教程可以参考微软官方 wsl。wsl 有两个版本，wsl1 和 wsl2，具体安装过程见下。一般我们默认使用 wsl2 版本。

[安装 WSL | Microsoft Learn](https://link.zhihu.com/?target=https%3A//learn.microsoft.com/zh-cn/windows/wsl/install)

wsl 的安装这里不介绍了，图太多。百度教程很多，这里就主要介绍下 WSL 安装 Ubuntu。

以下步骤均在 WSL2 安装好的背景下镜像。

### **2.2 安装 Ubuntu**

查看发行版本，

```
wsl.exe --list --online

```

如果遇到以下错误

无法从 “[https://raw.githubusercontent.com/microsoft/WSL/master/distributions/DistributionInfo.json](https://link.zhihu.com/?target=https%3A//raw.githubusercontent.com/microsoft/WSL/master/distributions/DistributionInfo.json)” 中提取列表分发。无法解析服务器的名称或地址

![](https://pic3.zhimg.com/v2-9368dfdc91d5b147e1569d9a95483b8e_b.png)

*   修改 dns 服务器

`网络和internet设置`-`高级网络设置`-`-`-`更多适配器选项`，选择你连接的网络，右键属性，将 ipv4 DNS 设置为`8.8.8.8`。

然后即可显示分装版本。

![](https://pic2.zhimg.com/v2-2c143ec2767eb5925cfbf09e847ce485_b.jpg)![](https://pic1.zhimg.com/v2-43a6ce11f2824a9d12cb1174f4b09b28_b.jpg)

**安装合适版本**

建议选择 20.04。22.04 版本我试过，有不少坑点。

```
wsl --install -d Ubuntu-20.04

```

### **2.3 迁移至非系统盘**

这一步主要是为了减少 C 盘的占用空间，默认 WSL 装 的 linux 子系统在 C 盘

docker-data 默认安装在 c 盘，且设置中难以更改，因此采用如下操作。

*   1、shutdown 子系统

```
wsl --shutdown

```

*   2、导出 Ubuntu

```
wsl --export Ubuntu-20.04 F:\Ubuntu\ubuntu.tar

```

*   3、注销 docker-desktop 和 docker-desktop-data

```
wsl --unregister Ubuntu-20.04

```

*   4、导入

```
wsl --import Ubuntu-20.04 F:\Ubuntu\ F:\Ubuntu\ubuntu.tar --version 2

```

*   5、删除压缩包

```
del D:\docker-desktop-data\docker-desktop-data.tar
del D:\docker-desktop-data\docker-desktop-data.tar

```

重新启动 docker

### **2.4 更改镜像源**

这一步主要是为了加速 linux 各种包的下载速度。

首先做一个备份，这里就把 Ubuntu 默认安装的镜像源备份，接着更改源

```
sudo cp  /etc/apt/sources.list /etc/apt/sources.list.bak
sudo vim /etc/apt/sources.list

```

## 进入 vim `ggdG`清空所有文本，复制清华源进 vim 编辑器中

打开如下源进行更新。注意 Ubuntu 版本选择 20.04

[ubuntu | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://link.zhihu.com/?target=https%3A//mirror.tuna.tsinghua.edu.cn/help/ubuntu/)

![](https://pic4.zhimg.com/v2-3f55fa59022426460dce594fb40b184b_b.jpg)

更新一下

```
sudo apt-get update
sudo apt-get upgrade

```

**一般情况下：**

在 win11 版本 wsl 和最新版（2023.4) 的 NVIDIA 驱动场景下，在 WSL2 内，通过`nvidia-smi`已经可以穿透显示显卡信息。

![](https://pic4.zhimg.com/v2-da91cd2ee0ac77509b18c9c88b7bc41b_b.jpg)

**3、wls-Docker 运行深度学习环境**
-------------------------

适用 Linux Docker 配置深度学习环境。该方法使用 Docker 直接部署深度学习环境，方便快捷, 并且不影响子系统本地。

Docker 的部署方式分为两种，一种是使用 desktop。即 windows 下载软件去部署 docker；一种是使用 linux 版本的 docker-engine。这就有点类似，一个使用图形化界面，一个使用小黑盒子。

这种 desktop 部署方式有优缺点：

优点：部署方便，可视化，适合小白

缺点：**极大地占用宿主（windows）的空间！**

*   1、Docker Data 文件为 **ext4.vhdx** 动态磁盘形式，该磁盘特点是，占用容量只会增大，即使删除其内部镜像文件，占用空间也不会减少。
*   2、**即使使用 3.2 节方法，虽然 docker-data 可以迁移至别的盘，但对镜像进行操作（如 push），windows 会产生临时文件夹，同样会占用 C 盘空间！**  
    留下贫穷的泪水! o(╥﹏╥)o
*   3、性能问题，docker-desktop 目前还有些 bug 未修复。

**对于不喜欢折腾的选 3.1 即可**。

**3.1 与 3.2 方式二选一即可，但博主建议使用 3.2 节 linux 原生 docker-Engine 进行配置！！！！**

**3.1 与 3.2 方式二选一即可，但博主建议使用 3.2 节 linux 原生 docker-Engine 进行配置！！！！**

**3.1 与 3.2 方式二选一即可，但博主建议使用 3.2 节 linux 原生 docker-Engine 进行配置！！！！**

**3.1 docker desktop 深度学习环境配置**
-------------------------------

### **3.1.1 docker desktop 安装**

[Install Docker Desktop on Windows](https://link.zhihu.com/?target=https%3A//docs.docker.com/desktop/install/windows-install/)

*   在 windows 上安装 docker-Desktop  
    别选 4.17.1 版本！！！！该版本无法调用 gpus all  
    **目前 4.18.0 已结修复该 bug**
*   设置  
    **1、镜像源**  
    将如下代码加入 Docker Engine 中  
    

![](https://pic3.zhimg.com/v2-0f1a581c08b2b1bacf85b631e202a796_b.jpg)

```
"registry-mirrors": [
      "https://registry.docker-cn.com",
      "http://hub-mirror.c.163.com",
      "https://docker.mirrors.ustc.edu.cn"
  ],

```

**2- 系统设置**

打开两个按钮

`Settings`---->`General`--->`Use the WSL 2 based engine`

`Settings`---->`Resources`--->`Enable integration with additional distros`

![](https://pic1.zhimg.com/v2-5c1763f2dde3a9f5d085baa5e11ac5c8_b.jpg)

测试是否安装成功，在 wsl Ubuntu 系统下输入，

```
docker run hello-world

```

### **3.1.2 wsl 切换 docker 数据存储位置**

docker-data 默认安装在 c 盘，占用很大空间，且设置中难以更改，因此采用如下操作。

*   1、shutdown docker

```
wsl --shutdown

```

*   2、导出 docker-desktop 和 docker-desktop-data

```
wsl --export docker-desktop-data F:\wsl2\docker-desktop-data.tar
wsl --export docker-desktop  F:\wsl2\docker-desktop.tar

```

*   3、注销 docker-desktop 和 docker-desktop-data

```
wsl --unregister docker-desktop-data
wsl --unregister docker-desktop

```

*   4、导入

```
wsl --import docker-desktop-data F:\wsl2\docker-desktop-data F:\wsl2\docker-desktop-data.tar --version 
wsl --import docker-desktop F:\wsl2\docker-desktop F:\wsl2\docker-desktop.tar --version 2

```

*   5、删除压缩包

```
del F:\wsl2\docker-desktop-data.tar
del F:\wsl2\docker-desktop-data.tar

```

重新启动 docker

### **3.1.3 调用测试 docker gpu**

dock 官网给的条件已结满足，由于目前版本已结最新, 可直接调用，无需安装其他内容。

Starting with Docker Desktop 3.1.0, Docker Desktop supports WSL 2 GPU Paravirtualization (GPU-PV) on NVIDIA GPUs. To enable WSL 2 GPU Paravirtualization, you need:

*   A machine with an NVIDIA GPU（有英伟达显卡）
*   The latest Windows Insider version from the Dev Preview ring（windows 版本更细）
*   [Beta drivers](https://link.zhihu.com/?target=https%3A//developer.nvidia.com/cuda/wsl) from NVIDIA supporting WSL 2 GPU Paravirtualization（最新显卡驱动即可）
*   Update WSL 2 Linux kernel to the latest version using `wsl --update` from an elevated command prompt（最新 WSL 版本）
*   Make sure the WSL 2 backend is enabled in Docker Desktop （Docker 中设置 WSL 2 backend 勾选上）

运行如下代码测试

```
docker run --rm -it --gpus=all nvcr.io/nvidia/k8s/cuda-sample:nbody nbody -gpu -benchmark

```

如果得到, 则说明安装成功。

> Compute 7.5 CUDA device: [NVIDIA GeForce RTX 2070 SUPER] 40960 bodies, total time for 10 iterations: 56.638 ms = 296.218 billion interactions per second = 5924.356 single-precision GFLOP/s at 20 flops per interaction

**3.2 wsl-linux 原生 docker-engine 深度学习环境**
-----------------------------------------

**docker engine 装在 wsl 内部，相比于 docker-desktop，能极大节省 windows 宿主空间**！！！！

且运行效率更高

### **3.2.1 linux-docker-engine 安装**

wsl 原生 docker 安装方式和 docker doc 安装方法一致

[Install Docker Engine on Ubuntu](https://link.zhihu.com/?target=https%3A//docs.docker.com/engine/install/ubuntu/)

**首先卸载老版本**

```
sudo apt-get remove docker docker-engine docker.io containerd runc

```

**建立储存库**

```
sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg

```

**添加官方的 gpgkey**

```
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

```

**设置版本库**

```
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

```

**更新 apt**

```
sudo apt-get update

```

**安装 Docker Engine, containerd, and Docker Compose.**

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

```

**验证 helloword**

```
sudo docker run hello-world

```

**配置镜像源**

和 docker desktop 一样配置镜像源，建议使用阿里云

登录 ->`控制台`-> 搜索`容器镜像服务`->'镜像加速器'

![](https://pic2.zhimg.com/v2-a60198807d4ea169909a03f0390ba985_b.jpg)

通过**修改 daemon 配置文件 / etc/docker/daemon.json 来使用加速器**

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://你自己的.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

```

**ps: 在上述流程中如果碰到 `Temporary failure resolving ‘gb.archive.ubuntu.com’`**

Temporary resolve 问题，加入 dns 服务器

如果这能解决你的临时解析信息，那么要么等待 24 小时，看看你的 ISP 是否为你解决了问题（或者直接联系你的 ISP）-- 或者你可以永久性地在你的系统中添加一个 DNS 服务器：

```
echo "nameserver 8.8.8.8" | sudo tee /etc/resolvconf/resolv.conf.d/base > /dev/null

```

### **3.2.2 wsl - docker-engine 自启动 (实现 systemctl)**

在单独的 linux 系统中, systemctl 可以对 docker 进行自启动，但是 wsl 初始系统并不支持 systemctl ，每次重新启动 wsl，都需要`sudo service docker start`

**现在 WSL2 有了 systemd 的支持，我们可以在没有 Docker 桌面的情况下在 WSL 中运行 Docker！！**

教程：

**step1 启动 systemctl**

**启动 wsl 内部的 linux 分发版本**，运行如下代码

```
# Enable systemd via /etc/wsl.conf
{
cat <<EOT
[boot]
systemd=true
EOT
} | sudo tee /etc/wsl.conf
​
exit

```

或者编辑 vim /etc/wsl.conf

使得

```
[boot]
systemd=true

```

ps: 上述设置，`sudo systemctl start docker`有还有问题的小伙伴。，可以换成

```
[boot]
#systemd=true
command = sudo service docker start

```

**step2 重启，测试 systemctl**

```
wsl --shutdown
wsl --distribution Ubuntu ##Ubuntu指你的镜像版本，按wsl -l -vs输出的第一列来

```

**测试过 systemd 是否在运行**

```
systemctl --no-pager status user.slice > /dev/null 2>&1 && echo 'OK: Systemd is running' || echo 'FAIL: Systemd not running'

```

**step3 docker 设置**

```
# Install docker-compose
sudo ln -s /usr/libexec/docker/cli-plugins/docker-compose /usr/bin/docker-compose
# 加入docker组
sudo usermod -aG docker $USER
​
#退出bash
exit

```

**重新登陆**

```
wsl --distribution Ubuntu

```

这时你会发现，不用`sudo service docker start`,docker 也能启动了

而且，也能使用一些列如如下命令。

```
sudo systemctl restart docker
sudo systemctl start docker

```

### **3.2.3 NVIDIA Container Toolkit 安装**

nvidia-container-toolkit 是使得 linux 系统在 docker 容器中具有调用 gpu 的能力。

![](https://pic2.zhimg.com/v2-5c108a9a5b88dd6d5aa9ae4b12638d99_b.jpg)

使用条件:

[NVIDIA/nvidia-docker: Build and run Docker containers leveraging NVIDIA GPUs (github.com)](https://link.zhihu.com/?target=https%3A//github.com/NVIDIA/nvidia-docker)

已经说明：

Make sure you have installed the [NVIDIA driver](https://link.zhihu.com/?target=https%3A//docs.nvidia.com/datacenter/tesla/tesla-installation-notes/index.html) and Docker engine for your Linux distribution.

Note that you do not need to install the CUDA Toolkit on the host system, but the NVIDIA driver needs to be installed.

**因此只用去 x 显卡装驱动和 Docker 就完事了！**

安装教程 [Installation Guide — NVIDIA Cloud Native Technologies documentation](https://link.zhihu.com/?target=https%3A//docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html%23docker)

建立 package repository 和 GPG key:

```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
      && curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
      && curl -s -L https://nvidia.github.io/libnvidia-container/experimental/$distribution/libnvidia-container.list | \
         sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
         sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit

```

设置 Docker daemon 守护进程识别 Nvidia 容器 Runtime

```
sudo nvidia-ctk runtime configure --runtime=docker
##重启一下docker
udo systemctl restart docker

```

### **3.2.4 调用测试 gpu**

```
docker run --rm -it --gpus=all nvcr.io/nvidia/k8s/cuda-sample:nbody nbody -gpu -benchmark

```

> Compute 7.5 CUDA device: [NVIDIA GeForce RTX 2070 SUPER] 40960 bodies, total time for 10 iterations: 56.638 ms = 296.218 billion interactions per second = 5924.356 single-precision GFLOP/s at 20 flops per interaction

则运行成功

**3.3 自定义自己 docker**
--------------------

无论是利用 3.1Docker-desktop 建立的还是 Docker-engine 建立的环境，均可以进行以下操作。

**出发点:**

考虑到目前官方 docker-hub 中提供的 imag 并不能满足日常的开发包的需要，且如果容器丢失或删除，所有包的数据和项目文件均会丢失。因此，我们可以定制自己的 image。并将 container 的项目文件映射到自己的本地文件目录中。

可以使用 vscode 完成以下操作，方便编辑一点。

进入 vscode，安装 wsl 插件，连接 wls（或着在 wsl linux 终端下，输入`code .`进入

ps: docker 开发 ide 也可以使用 vscode，只需要对安装 Docker 插件即可。

打开终端，如下图，当左下角绿色显式 WSL:Ubutun... 为连接成功

![](https://pic4.zhimg.com/v2-18e463509bd9cdaa018a2deb521b9747_b.jpg)

创建一个以后存 docker 镜像的目录

```
mkdir docker-deeplearning-project
cd docker-deeplearning-project

```

### **3.3.1 自定义所需要安装的包**

我们想要在 `tensorflow/tensorflow:latest-gpu`的基础上增加一些别的包，以满足日常需求，可以使用如下方法。打开 vscode 的终端，创建额外的 requirments.txt！

`code requirments.txt`

**安装以下包，这些包可以随各位去配**

**torch 那可以指定版本，在 torch 官网可以挑选**

```
jupyterlab
pandas
matplotlib
torch --index-url https://download.pytorch.org/whl/cu118
torchvision --index-url https://download.pytorch.org/whl/cu118
torchaudio --index-url https://download.pytorch.org/whlcu118

```

### **3.3.2 建立 Dockerfile**

dockfile 建立规则

![](https://pic2.zhimg.com/v2-04b19133e41a5ab29dbf276c598e2ac1_b.jpg)

```
code Dockerfile

```

Dockerfile 输入一下代码:

*   FROM: 拉取现有 base image
*   WORKDIR: 定制工作目录
*   COPY 拷贝本地目录的 requirements.txt 到当前 dockerfile 中，通过此去安装额外的 requirments 包，
*   RUN 运行安装程序，-i 指定镜像源
*   EXPOSE 开辟一定端口
*   ENTRYPOINT 定义入口点，通过列表构建命令

```
FROM tensorflow/tensorflow:latest-gpu
​
WORKDIR /tf-jinqiu
​
COPY requirements.txt requirements.txt
​
RUN pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple some-package
​
EXPOSE 8888
​
ENTRYPOINT ["jupyter","lab","--ip=0.0.0.0","--allow-root","--no-browser"] 

```

![](https://pic2.zhimg.com/v2-2c971a6ca974d52a61649169b7d08169_b.jpg)

### **3.3.3 建立 docker-compose.yaml**

![](https://pic3.zhimg.com/v2-8c48d1e8a8d5fd1dc8ea2c85ff5ec2b6_b.jpg)

```
code docker-compose.yaml

```

解析：

*   version：版本
*   services：定义服务
*   jupyter-lab：image 名称
*   build: . 在当前目录
*   ports：映射端口
*   volumes: 保存访问文件的部分。将其映射到本地文件中
*   deploy：share GPU

```
version: '1.0'
services:
  jupyter-lab:
    build: .
    ports:
      - "8888:8888"
    volumes:
      - /tf-jinqiu:/tf-jinqiu
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]

```

### **3.3.4 运行并测试**

```
docker-compose up

```

出现如下输出，则证明安装成功。

![](https://pic4.zhimg.com/v2-1fa269939b1251aba858d45806f8660b_b.jpg)

复制底下的：[http://127.0.0.1:8888/lab?token=d6e406efda3edf086b2baf4296a9400d53bee0d4221e1b9e](https://link.zhihu.com/?target=http%3A//127.0.0.1%3A8888/lab%3Ftoken%3Dd6e406efda3edf086b2baf4296a9400d53bee0d4221e1b9e)

即可 jupyter -lab

测试 pytorch 和 TensorFlow ,gpu 版本均能运行！！！

**注意文件映射位置**：

因此我们项目地址可以存在 container 中`/tf-jinqiu`中。

开发项目的地址为`docker-compose.yaml`中设置的`/tf-jinqiu`。因此在 container 中`/tf-jinqiu`和 wls ubuntu `/tf-jinqiu`目录下均有映射的文件。

![](https://pic2.zhimg.com/v2-dd4642e151d739c0fff1cde2f9a8d305_b.jpg)

进入 vscode docker 中，在容器中右键 Attach Visual Studio Code 即可进入容器内，进行相关 project 的开发。

![](https://pic4.zhimg.com/v2-98caa0b8fb126db04a4ef9d13477fe1b_b.jpg)

### **3.3.5 push 上传 dockerhub**

dockerhub 可以上传官网的 dockerhub，也可以上传阿里云的。

上传自己建好的 docker 可以方便以后迁移机器，直接部署。

**(1) 官方 dockerhub**

保存自己 docker 的方式，可以采用 docker-file 配合 docker-compose.yaml 的方式，也可采用将整个镜像上传至 Docker-hub 中，一劳永逸。

保存 dockerhub 中，每次换机器只需要从 docker 中拉取镜像，即可快速部署相关环境。

*   登录 docker

```
docker login -u zhujinqiu   # zhujinqiu为自己的账号

```

![](https://pic4.zhimg.com/v2-0dd71d599808754edafffd6d35451357_b.jpg)

*   docker 容器变成镜像

```
// 找到运行中的容器 (复制你要打包的容器的id)
docker ps
// 打包为镜像 （86d78d59b104：容器的id 、  my_centos：我们要打包成的镜像的名字）
docker commit zhujinqiu my_centos
// 找到打包的镜像
docker images

```

*   将镜像打为标签

```
// my_centos：镜像的名字   、 zhujinqiu：我们docker仓库的用户名 、 my_centos：我们刚才新建的仓库名 、 v1：版本号，可以不设置
docker tag my_centos zhujinqiu/my_centos:v1
​

```

*   将镜像上传到镜像仓库

```
docker push zhujinqiu/my_centos:v1

```

![](https://pic2.zhimg.com/v2-f147b79a3d4fe617f4c7fd0c42d09881_b.jpg)

*   下载自己的镜像

```
// 记得先登录
docker login
// 根据版本号拉取
docker pull zhujinqiu/my_centos:v1

```

*   修改镜像名字

```
// d583c3ac45fd  ID   后者是需要修改的名字
docker tag d583c3ac45fd myname:latest

```

**(2) 阿里云镜像仓库**

[容器镜像服务 (aliyun.com)](https://link.zhihu.com/?target=https%3A//cr.console.aliyun.com/cn-hangzhou/instances)

阿里云有详细的

首先创建镜像仓库仓库创建指南。

![](https://pic2.zhimg.com/v2-56a86fda8897ce114c58e96c21c8abc5_b.jpg)

1.  **登录阿里云 Docker Registry**

```
$ docker login --username=你的用户名 registry.cn-hangzhou.aliyuncs.com

```

用于登录的用户名为阿里云账号全名，密码为开通服务时设置的密码。

您可以在访问凭证页面修改凭证密码。

1.  **从 Registry 中拉取镜像**

```
$ docker pull registry.cn-hangzhou.aliyuncs.com/命名空间/仓库名称:[镜像版本号]

```

1.  **将镜像推送到 Registry**

```
$ docker login --username=你的用户名 registry.cn-hangzhou.aliyuncs.com
$ docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/命名空间/仓库名称:[镜像版本号]
$ docker push registry.cn-hangzhou.aliyuncs.com/命名空间/仓库名称:[镜像版本号]

```

请根据实际镜像信息替换示例中的 [ImageId] 和[镜像版本号]参数。

1.  **选择合适的镜像仓库地址**

从 ECS 推送镜像时，可以选择使用镜像仓库内网地址。推送速度将得到提升并且将不会损耗您的公网流量。

如果您使用的机器位于 VPC 网络，请使用 [http://registry-vpc.cn-hangzhou.aliyuncs.com](https://link.zhihu.com/?target=http%3A//registry-vpc.cn-hangzhou.aliyuncs.com) 作为 Registry 的域名登录。

1.  **示例**

使用 "docker tag" 命令重命名镜像，并将它通过专有网络地址推送至 Registry。

```
$ docker imagesREPOSITORY                                                         TAG                 IMAGE ID            CREATED             VIRTUAL SIZEregistry.aliyuncs.com/acs/agent                                    0.7-dfb6816         37bb9c63c8b2        7 days ago          37.89 MB$ docker tag 37bb9c63c8b2 registry-vpc.cn-hangzhou.aliyuncs.com/acs/agent:0.7-dfb6816

```

使用 "docker push" 命令将该镜像推送至远程。

```
$ docker push registry-vpc.cn-hangzhou.aliyuncs.com/acs/agent:0.7-dfb6816

```

**4、wsl 本地运行深度学习环境**
--------------------

该方法不借助 Docker，因此需要直接在本地配置好 WSL-Cuda，并安装各类深度学习框架，类似于把 Windows 上跑的模型搬到 WSL 跑。

该方法只需要安装 Cuda。并不要装显卡驱动和 Cudnn（wsl2 已经支持本地 Windows 映射）。

### **4.1 安装 wsl Cuda**

[CUDA Toolkit 12.1 Downloads | NVIDIA Developer](https://link.zhihu.com/?target=https%3A//developer.nvidia.com/cuda-downloads%3Ftarget_os%3DLinux%26target_arch%3Dx86_64%26Distribution%3DWSL-Ubuntu%26target_version%3D2.0%26target_type%3Ddeb_local)

![](https://pic2.zhimg.com/v2-f7f3c50d4df60c3d5b0fdd170412f825_b.jpg)

**根据驱动情况、pytorch、Tensorflow 情况选择合适的 wsl——cuda 版本。需要更改如下适应 “12-1 为 11-8”**

```
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-wsl-ubuntu.pin
sudo mv cuda-wsl-ubuntu.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/11.8.0/local_installers/cuda-repo-wsl-ubuntu-11-8-local_11.8.0-1_amd64.deb
sudo dpkg -i cuda-repo-wsl-ubuntu-11-8-local_11.8.0-1_amd64.deb
sudo cp /var/cuda-repo-wsl-ubuntu-11-8-local/cuda-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install cuda

```

安装完需要，**设置环境变量**

```
vim ~/.bashrc

```

在最后加入，配置环境变量

```
export PATH=/usr/bin:$PATH
export PATH=/usr/local/cuda-11.8/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-11.8/lib64:$LD_LIBRARY_PATH

```

更新下 source

```
source ~/.bashrc

```

使用 nvcc -V 验证安装成功与否

```
nvcc -V

```

![](https://pic1.zhimg.com/v2-c7f5647ced0121b141103df35610758c_b.jpg)

### **4.2 安装 pytorch 环境**

1、miniconda 安装

```
# 下载安装包
wget -c https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
chmod 777 Miniconda3-latest-Linux-x86_64.sh
# 安装conda
bash Miniconda3-latest-Linux-x86_64.sh


```

安装完需要，**设置环境变量**

```
vim ~/.bashrc

```

更新下 source

```
source ~/.bashrc

```

配置 miniconda 环境变量

```
export PATH=~/miniconda3/bin:$PATH

```

### **4.3 创建虚拟环境与 pytorch 安装、tensorflow 安装**

```
conda create -n env_pytorch python=3.8

# conda env list 查看虚拟环境list
# 激活
conda activate env_pytorch

```

1、安装 pytorch

[Start Locally | PyTorch](https://link.zhihu.com/?target=https%3A//pytorch.org/get-started/locally/)

根据 cuda 版本选择合适的 torchgpu 版本

```
## -i 清华镜像加速 安装torchgpu
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118 -i https://pypi.tuna.tsinghua.edu.cn/simple some-package

```

2、安装 tensorflow

ps: 新版本直接安装即可

[TensorFlow (google.cn)](https://link.zhihu.com/?target=https%3A//tensorflow.google.cn/install%3Fhl%3Dzh-cn)

```
# Requires the latest pip
pip install --upgrade pip

# Current stable release for CPU and GPU
pip install tensorflow


```

### **4.4 测试 gpu**

```
## torch
import torch
torch.cuda.is_available()
## tensorflow
import tensorflow as tf
tf.config.list_physical_devices()

```