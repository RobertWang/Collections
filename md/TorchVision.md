> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/145810572)

说明
--

很多基于 Pytorch 的工具集都非常好用，比如处理自然语言的 torchtext，处理音频的 torchaudio，以及处理图像视频的 torchvision。

torchvision 包含一些常用的数据集、模型、转换函数等等。当前版本 0.5.0 包括图片分类、语义切分、目标识别、实例分割、关键点检测、视频分类等工具，它将 mask-rcnn 功能也都包含在内了。mask-rcnn 的 Pytorch 版本最高支持 torchvision 0.2.*，0.3.0 之后 mask-rcnn 就包含到 tensorvision 之中了。

安装
--

torchvision 安装非常方便：

```
$ pip install torchvision 

```

但需要注意版本匹配：

```
    torch 1.1.0/1.1.0 + torchvision 0.2.* + CUDA 9
    torch 1.2.0/1.3.0 + torchvision 0.3.* + CUDA 10
    torch 1.2.0/1.3.0 + torchvision 0.4.* + CUDA 10
    torch 1.4.0 + torchvision 0.5.* + CUDA 10 

```

高版本的 torchvision 提供更多的功能，但需要升级 torch 库，同时还需要与 CUDA 版本匹配，否则无法正常工作。从 CUDA 9 升级成 CUDA 10，还需要升级与 CUDA 10 匹配的显卡驱动，如 nvidia-drivers-430。如果使用 docker，则需要升级宿主机的显卡驱动。

代码分析
----

由于源码中带有使用例程和训练模型的工具，建议下载源码。地址：

官方说明文档：

以目标检测为例，其源码在 vision/torchvision/models/detection 目录下，内容与 maskrcnn_benchmark 非常相似，同时还做了一些简化。

例程
--

下面示例使用预训练的模型识别图片中的物体：

```
    import torch
     
    import torchvision
    from PIL import Image
    import torchvision.transforms as transforms
    device = torch.device('cuda') if torch.cuda.is_available() else torch.device('cpu')
    model = torchvision.models.detection.fasterrcnn_resnet50_fpn(pretrained=True)
    model.to(device)
    model.eval()
    loader = transforms.Compose([transforms.ToTensor()])
    image = Image.open('xxx.jpg').convert('RGB')
    image = loader(image)
    image = image.to(device, torch.float)
    x = [image]
    predictions = model(x)
    print(predictions)

```

Fine-tune 目标检测模型
----------------

之前笔者尝试使用 Mask-RCNN 官方的 TensorFlow 和 Pytorch 版本实现目标识别和图片分割的 Fine-tune。与之相比，TorchVision 更加简单。

无论使用何种工具，Fine-tune 都以调库为主，比较复杂容易出错的是构造数据文件，在尝试过程中，要么需要下载大量的数据，比如 COCO 数据集，要么需要标注和调试自己的数据，在外围工作上花费大量时间。

“行人检测”是 “PyTorch 官方教程中文版” 中的 fine-tune 实例，全部代码 100 多行，数据也只有 100 多张图片，从例程中可以看到，自己做数据类，比下载 COCO 数据训练更加方便。如果使用 GPU，如 1080ti，十次迭代在十分钟以内即可完成。具体请见：

如需训练模型，则要下载 git 源码，其中的 references 目录中提供了一些训练使用的工具和例程。上述例程需要在源码的 references/detection 目录下运行，训练数据也需要放在该目录下。另外，需要注意分类的第 0 个索引是背景，在不同任务中，类别号一般需要做 + 1 处理。