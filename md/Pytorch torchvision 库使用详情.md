> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/476596668)

**一、简介**
--------

tochvision 主要处理图像数据，包含一些常用的数据集、模型、转换函数等。torchvision 独立于 PyTorch，需要专门安装。  
torchvision 主要包含以下四部分：

*   torchvision.models: 提供深度学习中各种经典的网络结构、预训练好的模型，如：Alex-Net、VGG、ResNet、Inception 等。
*   torchvision.datasets：提供常用的数据集，设计上继承 torch.utils.data.Dataset，主要包括：MNIST、CIFAR10/100、ImageNet、COCO 等。
*   torchvision.transforms：提供常用的数据预处理操作，主要包括对 Tensor 及 PIL Image 对象的操作。
*   torchvision.utils：工具类，如保存张量作为图像到磁盘，给一个小批量创建一个图像网格。

**二、安装**
--------

```
pip3 install torchvision

```

torchvision 要注意与 pytorch 版本和 Cuda 相匹配。  
要查询 pytorch 和 torchvision 的版本，可以使用下面语句 ：

```
import torch
import torchvision
print(torch.__version__)
print(torchvision.__version__)

```

三、torchvision 的主要功能示例
---------------------

### 1. 加载 model

(1) 加载几个预训练模型  
通过 pretrined=True 可以加载预训练模型。pretrained 默认值是 False，不赋值和赋值 False 效果一样。

```
import torchvision.models as models
resnet18 = models.resnet18(pretrained=True)
vgg16 = models.vgg16(pretrained=True)
alexnet = models.alexnet(pretrained=True)
squeezenet = models.squeezenet1_0(pretrained=True)
densenet = models.densenet_161()

```

预训练模型期望的输入：

*   RGB 图像的 mini-batch：(batch_size, 3, H, W)，并且 H 和 W 不能低于 224。
*   图像的像素值必须在范围 [0,1] 间，并且用均值 mean=[0.485, 0.456, 0.406]和方差 std=[0.229, 0.224, 0.225]进行归一化。

下载的模型可以通过 state_dict() 来打印状态参数、缓存的字典。

```
import torchvision.models as models
vgg16 = models.vgg16(pretrained=True)
# 返回包含模块所有状态的字典，包括参数和缓存
pretrained_dict = vgg16.state_dict()

```

![](https://pic2.zhimg.com/v2-21c4de87bb0fe108b23215a0ebd851a1_b.jpg)![](https://pic3.zhimg.com/v2-ad5dd5b7bd11ecbb815a476838f252e6_b.jpg)

(2) 只加载模型，不加载预训练参数 如果只需要网络结构，不需要训练模型的参数来初始化，可以将 pretrained = False

```
# 导入模型结构
resnet18 = models.resnet18(pretrained=False)
# 加载预先下载好的预训练参数到resnet18
resnet18.load_state_dict(torch.load('resnet18-5c106cde.pth'))

```

(3) 加载部分预训练模型  
实际使用中可能会对预训练模型进行调节，就是对预训练模型中的层进行修改。  
下面示例中，对原模型中不匹配的键进行了删除 ， 注意新模型改变了的层需要和原模型对应层的名字不一样，比如：resnet 最后一层的名字是 fc(PyTorch 中)，那么我们修改过的 resnet 的最后一层就不能取这个名字，可以叫 fc_

```
import torchvision.models as models
resnet152 = models.resnet152(pretrained=True)
# 提取参数
pretrained_dict = resnet152.state_dict()
# 预训练模型也可以通过model_zoo下载参数
# pretrained_dict = model_zoo.load_url(model_urls['resnet152'])
model_dict = resnet152.state_dict()
# 将pretrained_dict里不属于model_dict的键剔除掉
pretrained_dict = {k: v for k, v in pretrained_dict.items() if k in model_dict}
# 更新现有的model_dict
model_dict.update(pretrained_dict)
# 加载真正需要的state_dict
resnet152.load_state_dict(model_dict)

```

(4) 调整模型  
预训练的模型有些层并不是直接能用，需要我们微微改一下，比如，resnet 最后的全连接层是分 1000 类，而我们只有 21 类；或 resnet 第一层卷积接收的通道是 3， 我们可能输入图片的通道是 4，那么可以通过以下方法修改：

```
# 修改通道数
resnet.conv1 = nn.Conv2d(4, 64, kernel_size=7, stride=2, padding=3, bias=False)
# 这里的21即是分类
resnet.fc = nn.Linear(2048, 21)


from torchvision import models
from torch import nn
# 加载预训练好的模型，保存到 ~/.torch/models/ 下面
resnet34 = models.resnet34(pretrained=True, num_classes=1000)
# 默认是ImageNet上的1000分类，这里修改最后的全连接层为10分类问题
resnet34.fc = nn.Linear(512, 10)

```

(5) 加载非预训练模型的方法： 这个与 torchvision 没有关系。

1. 保存和加载整个模型

```
torch.save(model_object, 'resnet.pth')
model = torch.load('resnet.pth')

```

2. 分别加载网络的结构和参数

```
# 将my_resnet模型字典储存为my_resnet.pth
torch.save(my_resnet.state_dict(), "my_resnet.pth")
# 加载resnet，模型存放在my_resnet.pth
my_resnet.load_state_dict(torch.load("my_resnet.pth"))

```

### 2. 加载数据集

torchvision.datasets 是从 torch.utils.data.Dataset 的子类，可以使用 torch.utils.data.DataLoader 进行多线程处理。官网参考地址：[https://pytorch.org/vision/stable/datasets.html#](https://link.zhihu.com/?target=https%3A//pytorch.org/vision/stable/datasets.html%23)

![](https://pic3.zhimg.com/v2-2e39a446a35dacb919ebb673a46d52a6_b.jpg)![](https://pic1.zhimg.com/v2-a8ed543a447c970f2048c87060ff32fc_b.jpg)![](https://pic2.zhimg.com/v2-7f17c11acfa4fba3dadfaf2bd3fa54a5_b.jpg)

**(1) 示例：加载 MNIST**

```
from torchvision import datasets
dataset = datasets.MNIST('data/', download=True, train=False, transform=None)

```

![](https://pic3.zhimg.com/v2-eec8fc1243e3b005f184ffa34c0a374a_b.jpg)

**(2) 示例：加载 Fashion-MNIST**

```
from torchvision import datasets
dataset = datasets.FashionMNIST('data/', download=True, train=False, transform=None)

```

**(3) ImageFolder 实现数据导入**  
datasets.ImageFolder 方法可以实现数据导入。

```
ImageFolder(root,transform=None,target_transform=None,loader=default_loader)

```

参数说明：

*   root : 在指定的 root 路径下面寻找图片。
*   transform: 接收 PIL 图像的函数 / 转换并返回已转换的版本。 可以直接使用上面的 Compose 方法组合需要的变换。
*   target_transform : 对 label 进行变换。
*   loader: 指定加载图片的函数，默认操作是读取 PIL image 对象。

这个方法返回的是 list，可以使用 data.DataLoader 转成 Tensor 数据。

**示例：**

```
data_transforms = {    'train': transforms.Compose([        
                               transforms.RandomResizedCrop(224), #Random cutting of an image (224, 224) from the original image        
                               transforms.RandomHorizontalFlip(), #Reversal at 0.5 probability level        
                               transforms.ToTensor(),  #Convert a PIL. Image with a range of [0,255] or numpy. ndarray to a shape of [C, H, W], and a FloadTensor with a range of [0,1.0].        
                               transforms.Normalize([0.485, 0.456, 0.406], [0.229, 0.224, 0.225]) #Normalization    
                               ]),    
                       'val': transforms.Compose([        
                               transforms.Resize(256),        
                               transforms.CenterCrop(224),        
                               transforms.ToTensor(),        
                               transforms.Normalize([0.485, 0.456, 0.406], [0.229, 0.224, 0.225])    
                               ]),} #image data 
filedata_root = ''
image_datasets = {x: datasets.ImageFolder(os.path.join(data_root, x),     
                  data_transforms[x]) for x in ['train', 'val']}
                  # wrap your data and label into Tensor
                  dataloders = {x: torch.utils.data.DataLoader(image_datasets[x],
                  batch_size=10, 
                  shuffle=True,                                             
                  num_workers=4) for x in ['train', 'val']}

```

dataloaders 是 Variable 类型，可以作为模型的输入参数。

### **3. transforms**

transforms 包含了一些图像预处理操作，这些操作可以使用 torchvison.transforms.Compose 连在一起进行串行 操作。这些操作有：

```
__all__ = ["Compose", "ToTensor", "ToPILImage", "Normalize", "Resize", "Scale", "CenterCrop", "Pad",
           "Lambda", "RandomApply", "RandomChoice", "RandomOrder", "RandomCrop", "RandomHorizontalFlip", 
           "RandomVerticalFlip", "RandomResizedCrop", "RandomSizedCrop", "FiveCrop", "TenCrop", "LinearTransformation",           
           "ColorJitter", "RandomRotation", "RandomAffine", "Grayscale", "RandomGrayscale"]

```

*   Compose()：用来管理所有的 transforms 操作。
*   ToTensor()：把图片数据转换成张量并转化范围在 [0,1] 区间内。
*   Normalize(mean, std)：归一化。
*   Resize(size)：输入的 PIL 图像调整为指定的大小，参数可以为 int 或 int 元组。
*   CenterCrop(size)：将给定的 PIL Image 进行中心切割，得到指定 size 的 tuple。
*   RandomCrop(size, padding=0)：随机中心点切割。
*   RandomHorizontalFlip(size, interpolation=2)：将给定的 PIL Image 随机切割，再 resize。
*   RandomHorizontalFlip()：随机水平翻转给定的 PIL Image。
*   RandomVerticalFlip()：随机垂直翻转给定的 PIL Image。
*   ToPILImage()：将 Tensor 或 numpy.ndarray 转换为 PIL Image。
*   FiveCrop(size)：将给定的 PIL 图像裁剪成 4 个角落区域和中心区域。
*   Pad(padding, fill=0, padding_mode=‘constant’)：对 PIL 边缘进行填充。
*   RandomAffine(degrees, translate=None, scale=None)：保持中心不变的图片进行随机仿射变化。
*   RandomApply(transforms, p=0.5)：随机选取变换。