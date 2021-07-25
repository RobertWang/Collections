> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU2NTUwNjQ1Mw==&mid=2247504333&idx=1&sn=d4e738e9802ab1e9abd102d042d12347&chksm=fcb82937cbcfa021bc89f28127113ebaf6b225be3574136b86acee310621f278192aff32e2aa&mpshare=1&scene=1&srcid=0724YwaP9JDlIUGQyyMZC0VJ&sharer_sharetime=1627116228242&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

另一方面，Inception 网络经过精心设计，非常深入和复杂。它使用了许多不同的技术来推动其性能；无论是速度还是准确性。

### 什么是 Inception？

后来演化出了不同版本的 Inception 网络。这是 Sergey Ioffe、Christian Szegedy、Jonathon Shlens、Vincent Vanhouck 和 Zbigniew Wojna 在 2015 年题为 “Rethinking the Inception Architecture for Computer Vision” [2] 的论文中提出的。Inception 模型被归类为最受欢迎和最常用的深度学习模型之一。

### 设计原则

–如果网络具有更多不同的过滤器，这些过滤器将具有更多不同的特征图，则网络将学习得更快。

–降维的空间聚合可以在低维嵌入上完成，而不会损失太多表示能力。

–通过宽度和深度之间的平衡，可以实现网络的最佳性能。

### 初始模块

初始模块（naive）

![](https://mmbiz.qpic.cn/mmbiz_png/ABvEnMciauWtN4GoGA9rCrn2UJ1MNRia8wXxyoUqhZp692wPc30uUia7XHB1Diag24KdlicZTBxGBeXic0Xyw7lBkBLQ/640?wx_fmt=png)

资料来源：'Going Deeper with Convolution' 论文

最优局部稀疏结构的近似

*   处理各种尺度的视觉 / 空间信息，然后聚合
    
*   从计算上看，这有点乐观
    
*   5×5 卷积非常花费开销
    

**Inception 模块（降维）**

![](https://mmbiz.qpic.cn/mmbiz_png/ABvEnMciauWtN4GoGA9rCrn2UJ1MNRia8wsLjSY3qAYeyqWjSftBPVe3u1CvvEELdPibYPtDmSzXjIUdfgVEdpw4g/640?wx_fmt=png)

来源：'Going Deeper with Convolution' 论文

降维是必要和有动力的（网络中的网络）

*   通过 1×1 卷积实现
    
*   深入思考学习池化，而不是高度 / 宽度的最大 / 平均池化。
    

### 初始架构

使用降维的 inception 模块，构建了深度神经网络架构（Inception v1）。架构如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/ABvEnMciauWtN4GoGA9rCrn2UJ1MNRia8wqs72fA046OvCEH2axFRQPJfiasWCoa9tP4eWHNETplmDbhv9eNuZ33w/640?wx_fmt=png)

Inception 网络线性堆叠了 9 个这样的 Inception 模块。它有 22 层深（如果包括池化层，则为 27 层）。在最后一个 inception 模块的最后，它使用了全局平均池化。

*   对于降维和修正线性激活，使用了 128 个滤波器的 1×1 卷积。
    
*   具有 1024 个单元的全连接层的修正线性激活。
    
*   使用 dropout 层丢弃输出的比例为 70%。
    
*   使用线性层作为分类器的 softmax 损失。
    

Inception 网络的流行版本如下：

*   Inception v1
    
*   Inception v2
    
*   Inception v3
    
*   Inception v4
    
*   Inception-ResNet
    

### 让我们从头开始构建 Inception v1(GoogLeNet)：

Inception 架构多次使用 CNN 块和 1×1、3×3、5×5 等不同的过滤器，所以让我们为 CNN 块创建一个类，它采用输入通道和输出通道以及 batchnorm2d 和 ReLu 激活.

```
class conv_block(nn.Module):
    def __init__(self, in_channels, out_channels, **kwargs):
        super(conv_block, self).__init__()
        self.relu = nn.ReLU()
        self.conv = nn.Conv2d(in_channels, out_channels, **kwargs)
        self.batchnorm = nn.BatchNorm2d(out_channels)
   def forward(self, x):
       return self.relu(self.batchnorm(self.conv(x)))
```

然后为 inception module 创建一个降维的类，参考上图，从 1×1 filter 输出，reduction 3×3，然后从 3×3 filter 输出，reduction 5×5，然后从 5×5 输出 和 1×1 池中输出。

```
class Inception_block(nn.Module):
    def __init__(
        self, in_channels, out_1x1, red_3x3, out_3x3, red_5x5, out_5x5, out_1x1pool
    ):
        super(Inception_block, self).__init__()
        self.branch1 = conv_block(in_channels, out_1x1, kernel_size=(1, 1))

self.branch2 = nn.Sequential(
conv_block(in_channels, red_3x3, kernel_size=(1, 1)),
conv_block(red_3x3, out_3x3, kernel_size=(3, 3), padding=(1, 1)),
)

self.branch3 = nn.Sequential(
conv_block(in_channels, red_5x5, kernel_size=(1, 1)),
conv_block(red_5x5, out_5x5, kernel_size=(5, 5), padding=(2, 2)),
)

self.branch4 = nn.Sequential(
nn.MaxPool2d(kernel_size=(3, 3), stride=(1, 1), padding=(1, 1)),
conv_block(in_channels, out_1x1pool, kernel_size=(1, 1)),
)

def forward(self, x):
return torch.cat(
[self.branch1(x), self.branch2(x), self.branch3(x), self.branch4(x)], 1
)
```

让我们保留下图作为参考并开始构建网络。

![](https://mmbiz.qpic.cn/mmbiz_png/ABvEnMciauWtN4GoGA9rCrn2UJ1MNRia8wwmvZXxwGbLQhhcPlNxBgpanK8f35dL7Zmv0icqqlMm6Ficqcw0391GOQ/640?wx_fmt=png)

来源：'Going Deeper with Convolution' 论文

创建一个类作为 GoogLeNet

```
class GoogLeNet(nn.Module):
    def __init__(self, aux_logits=True, num_classes=1000):
        super(GoogLeNet, self).__init__()
        assert aux_logits == True or aux_logits == False
        self.aux_logits = aux_logits

# Write in_channels, etc, all explicit in self.conv1, rest will write to
# make everything as compact as possible, kernel_size=3 instead of (3,3)
self.conv1 = conv_block(
in_channels=3,
out_channels=64,
kernel_size=(7, 7),
stride=(2, 2),
padding=(3, 3),
)

self.maxpool1 = nn.MaxPool2d(kernel_size=3, stride=2, padding=1)
self.conv2 = conv_block(64, 192, kernel_size=3, stride=1, padding=1)
self.maxpool2 = nn.MaxPool2d(kernel_size=3, stride=2, padding=1)

# In this order: in_channels, out_1x1, red_3x3, out_3x3, red_5x5, out_5x5, out_1x1pool
self.inception3a = Inception_block(192, 64, 96, 128, 16, 32, 32)
self.inception3b = Inception_block(256, 128, 128, 192, 32, 96, 64)
self.maxpool3 = nn.MaxPool2d(kernel_size=(3, 3), stride=2, padding=1)

self.inception4a = Inception_block(480, 192, 96, 208, 16, 48, 64)
self.inception4b = Inception_block(512, 160, 112, 224, 24, 64, 64)
self.inception4c = Inception_block(512, 128, 128, 256, 24, 64, 64)
self.inception4d = Inception_block(512, 112, 144, 288, 32, 64, 64)
self.inception4e = Inception_block(528, 256, 160, 320, 32, 128, 128)
self.maxpool4 = nn.MaxPool2d(kernel_size=3, stride=2, padding=1)

self.inception5a = Inception_block(832, 256, 160, 320, 32, 128, 128)
self.inception5b = Inception_block(832, 384, 192, 384, 48, 128, 128)

self.avgpool = nn.AvgPool2d(kernel_size=7, stride=1)
self.dropout = nn.Dropout(p=0.4)
self.fc1 = nn.Linear(1024, num_classes)

if self.aux_logits:
self.aux1 = InceptionAux(512, num_classes)
self.aux2 = InceptionAux(528, num_classes)
else:
self.aux1 = self.aux2 = None

def forward(self, x):
x = self.conv1(x)
x = self.maxpool1(x)
x = self.conv2(x)
# x = self.conv3(x)
x = self.maxpool2(x)

x = self.inception3a(x)
x = self.inception3b(x)
x = self.maxpool3(x)

x = self.inception4a(x)

# Auxiliary Softmax classifier 1
if self.aux_logits and self.training:
aux1 = self.aux1(x)

x = self.inception4b(x)
x = self.inception4c(x)
x = self.inception4d(x)

# Auxiliary Softmax classifier 2
if self.aux_logits and self.training:
aux2 = self.aux2(x)

x = self.inception4e(x)
x = self.maxpool4(x)
x = self.inception5a(x)
x = self.inception5b(x)
x = self.avgpool(x)
x = x.reshape(x.shape[0], -1)
x = self.dropout(x)
x = self.fc1(x)

if self.aux_logits and self.training:
return aux1, aux2, x
else:
return x
```

然后为输出层定义一个类，如论文中提到的 dropout=0.7 和一个带有 softmax 的线性层来输出 n_classes。

```
class InceptionAux(nn.Module):
    def __init__(self, in_channels, num_classes):
        super(InceptionAux, self).__init__()
        self.relu = nn.ReLU()
        self.dropout = nn.Dropout(p=0.7)
        self.pool = nn.AvgPool2d(kernel_size=5, stride=3)
        self.conv = conv_block(in_channels, 128, kernel_size=1)
        self.fc1 = nn.Linear(2048, 1024)
        self.fc2 = nn.Linear(1024, num_classes)
```

```
    def forward(self, x):
        x = self.pool(x)
        x = self.conv(x)
        x = x.reshape(x.shape[0], -1)
        x = self.relu(self.fc1(x))
        x = self.dropout(x)
        x = self.fc2(x)

        return x
```

**然后程序应该如下所示对齐。**

– Class GoogLeNet

– Class Inception_block

– Class InceptionAux

– Class conv_block

然后最后让我们写一小段测试代码来检查我们的模型是否工作正常。

```
if __name__ == "__main__":
    # N = 3 (Mini batch size)
    x = torch.randn(3, 3, 224, 224)
    model = GoogLeNet(aux_logits=True, num_classes=1000)
    print(model(x)[2].shape)
```

输出应如下所示

![](https://mmbiz.qpic.cn/mmbiz_png/ABvEnMciauWtN4GoGA9rCrn2UJ1MNRia8w2vlodcNljmUySa99ianpB211b2dyySDAWqxud0FGnuUZn4G4RyjcdIg/640?wx_fmt=png)

#### 整个代码可以在这里访问：

https://github.com/BakingBrains/Deep_Learning_models_implementation_from-scratch_using_pytorch_/blob/main/Inception_.py

[1]. Christian Szegedy, Wei Liu, Yangqing Jia. Pierre Sermanet, Scott Reed, Dragomir Anguelov, Dumitru Erhan, Vincent Vanhoucke and Andrew Rabinovich: Going deeper with convolutions, Sep 2014, DOI: https://arxiv.org/pdf/1409.4842v1.pdf

[2]. Sergey Ioffe , Christian Szegedy, Jonathon Shlens, Vincent Vanhouck and Zbigniew Wojna: Rethinking the Inception Architecture for Computer Vision, Dec 2015, DOI:https://arxiv.org/pdf/1512.00567.pdf