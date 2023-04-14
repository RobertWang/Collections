> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/rothschild666/article/details/121720776)

基于 [Pytorch](https://so.csdn.net/so/search?q=Pytorch&spm=1001.2101.3001.7020) 中安装 torchvision 简单详细完整版
=====================================================================================================

介绍：很多基于 Pytorch 的工具集都非常好用，比如处理自然语言的 torchtext，处理音频的 torchaudio，以及处理图像视频的 torchvision。torchvision 包含一些常用的数据集、模型、转换函数等等。包括图片分类、语义切分、目标识别、实例分割、关键点检测、视频分类等工具。
----------------------------------------------------------------------------------------------------------------------------------------------------------

两种安装方案（博主建议采用第一种方案）：
--------------------

第一种方案：
------

一、在 CMD 控制平台查看电脑已经安装好的 Anaconda 中的 Python 版本，Pytorch 中 [torch](https://so.csdn.net/so/search?q=torch&spm=1001.2101.3001.7020) 版本和 Cuda 版本，若没有安装可点击下面的博主的文章链接按操作先进行安装。
---------------------------------------------------------------------------------------------------------------------------------------------------------------------

基于 Windows 中学习 Deep Learning 之搭建 Anaconda+Pytorch（Cuda+Cudnn）+Pycharm 工具和配置环境完整最简版：[点击打开文章链接](https://blog.csdn.net/rothschild666/article/details/121593090)  
![](https://img-blog.csdnimg.cn/dd3f769f51664577b36098a27aac0ee3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAcm90aHNjaGlsZGxobA==,size_20,color_FFFFFF,t_70,g_se,x_16)

二、复制下面的命令在 CMD 控制平台中的 D:\Anaconda\Scripts > 路径中运行。
--------------------------------------------------

```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple torchvision

```

![](https://img-blog.csdnimg.cn/39e35e9b8c674afca844a1f17d926deb.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAcm90aHNjaGlsZGxobA==,size_20,color_FFFFFF,t_70,g_se,x_16)

三、验证是否安装成功，输入下面的命令在 CMD 平台运行，没有报错说明安装导入包 torchvision 成功。
--------------------------------------------------------

```
python -c "import torchvision"

```

![](https://img-blog.csdnimg.cn/868dbf8dc38f4df99547130d1872ee8e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAcm90aHNjaGlsZGxobA==,size_20,color_FFFFFF,t_70,g_se,x_16)

四、发现自己 torchvision 版本太低，不支持分割模型、检测模型等，所以需要只更新一下 torchvision 版本。
---------------------------------------------------------------

![](https://img-blog.csdnimg.cn/a195d73f3135445e828d0147563fc0ec.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAcm90aHNjaGlsZGxobA==,size_20,color_FFFFFF,t_70,g_se,x_16)

五、复制下面的命令在 CMD 控制平台中的 D:\Anaconda\Scripts > 路径中运行。
--------------------------------------------------

```
pip install --no-deps torchvision==0.4.1

```

![](https://img-blog.csdnimg.cn/5e81890a482443d1b0e4592ec7a532d0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAcm90aHNjaGlsZGxobA==,size_20,color_FFFFFF,t_70,g_se,x_16)

六、torchvision 版本更新成功。
---------------------

![](https://img-blog.csdnimg.cn/ea8b602e0a0e4324b8cae8fd409bb550.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAcm90aHNjaGlsZGxobA==,size_20,color_FFFFFF,t_70,g_se,x_16)

第二种方案：
------

一、点击打开下面的链接。
------------

torchvision 官方链接：[点击链接打开官方下载文件包网页](https://download.pytorch.org/whl/torch_stable.html)  
![](https://img-blog.csdnimg.cn/bdce5d7760614a788fef2e188f8776d9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAcm90aHNjaGlsZGxobA==,size_20,color_FFFFFF,t_70,g_se,x_16)  
![](https://img-blog.csdnimg.cn/231fa0e1833d4edea88cb8caa442d642.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAcm90aHNjaGlsZGxobA==,size_20,color_FFFFFF,t_70,g_se,x_16)

二、（重要）根据第一步的 torch 版本选择对应的链接，比如博主电脑已经下载好的 torch 版本是 1.10.0，Cuda 版本对应的是 10.2，操作系统是 Windows 和 Python 的版本是 3.7.0，所以选择对应的 whl 文件分别点击进行下载。（注意：cp37m 对应 3.7.0，win 对应 Windows，amd64 对应 64 位）
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

![](https://img-blog.csdnimg.cn/e7ef236f9a804679842092f307907ce9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAcm90aHNjaGlsZGxobA==,size_20,color_FFFFFF,t_70,g_se,x_16)

三、下载完成之后，将这些 whl 文件复制粘贴到 Anaconda 中的 Scripts 文件夹中，方便后面使用 pip 进行安装。
------------------------------------------------------------------

![](https://img-blog.csdnimg.cn/7ccda409ed6343328da89e9946660f13.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAcm90aHNjaGlsZGxobA==,size_20,color_FFFFFF,t_70,g_se,x_16)  
![](https://img-blog.csdnimg.cn/f11172ebf4ba4700a7b4ff444e6482d1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAcm90aHNjaGlsZGxobA==,size_20,color_FFFFFF,t_70,g_se,x_16)

四、复制下面的命令在 CMD 控制平台中的 D:\Anaconda\Scripts > 路径中运行。
--------------------------------------------------

```
pip install torchvision-0.9.0-cp37-cp37m-win_amd64.whl

```

![](https://img-blog.csdnimg.cn/9c212aaf99c84b49b3c4c379191ebd38.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAcm90aHNjaGlsZGxobA==,size_20,color_FFFFFF,t_70,g_se,x_16)

五、验证是否安装成功，输入下面的命令在 CMD 平台运行，没有报错说明安装导入包 torchvision 成功。
--------------------------------------------------------

```
python -c "import torchvision"

```

![](https://img-blog.csdnimg.cn/868dbf8dc38f4df99547130d1872ee8e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAcm90aHNjaGlsZGxobA==,size_20,color_FFFFFF,t_70,g_se,x_16)