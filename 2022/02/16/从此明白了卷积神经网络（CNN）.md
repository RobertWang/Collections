> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5OTA1MDUyMA==&mid=2655468945&idx=2&sn=1e1660d1807b72099ca2d7eb484edd92&chksm=bd72f1e68a0578f0ce5c5f888a6d8ad75c4818e0df825e7f00232f44f94b43607fc6269614f8&scene=21#wechat_redirect)

**卷积神经网络**是一种曾经让我无论如何也无法弄明白的东西，主要是名字就太 “高级” 了，网上的各种各样的文章来介绍 “什么是卷积” 尤为让人受不了。听了吴恩达的网课之后，豁然开朗，终于搞明白了这个东西是什么和为什么。我这里大概会用 6~7 篇文章来讲解 CNN 并实现一些有趣的应用。看完之后大家应该可以自己动手做一些自己喜欢的事儿了。

一、引子：边界检测

我们来看一个最简单的例子：“边界检测（edge detection）”，假设我们有这样的一张图片，大小 8×8：

![](https://mmbiz.qpic.cn/mmbiz_png/QLDSy3Cx3YLeVWnKYupUyuvPzDbtrJKG6ah4GGnFuUULJ3IfBozS2awX0hbXgIMUicgTxACtgjB3dWYL1w1dD1Q/640?wx_fmt=png)

图片中的数字代表该位置的像素值，我们知道，像素值越大，颜色越亮，所以为了示意，我们把右边小像素的地方画成深色。图的中间两个颜色的分界线就是我们要检测的边界。

怎么检测这个边界呢？我们可以设计这样的一个 **滤波器（filter，也称为 kernel）**，大小 3×3：

![](https://mmbiz.qpic.cn/mmbiz_png/QLDSy3Cx3YLeVWnKYupUyuvPzDbtrJKGGOMcd8yjhwFIQYOvRFB4ZzRqYUITiciaZy9qokxZ2AmbMaJghfEKrUfg/640?wx_fmt=png)

然后，我们用这个 filter，往我们的图片上 “盖”，覆盖一块跟 filter 一样大的区域之后，对应元素相乘，然后求和。计算一个区域之后，就向其他区域挪动，接着计算，直到把原图片的每一个角落都覆盖到了为止。这个过程就是 **“卷积”**。  
（我们不用管卷积在数学上到底是指什么运算，我们只用知道在 CNN 中是怎么计算的。）  
这里的 “挪动”，就涉及到一个步长了，假如我们的步长是 1，那么覆盖了一个地方之后，就挪一格，容易知道，总共可以覆盖 6×6 个不同的区域。

那么，我们将这 6×6 个区域的卷积结果，拼成一个矩阵：

![](https://mmbiz.qpic.cn/mmbiz_png/QLDSy3Cx3YLeVWnKYupUyuvPzDbtrJKGqkA2JxCe7qnURXlpL8fEIT0Xp65iaqFEOZ4ylDibyxDWX9UqZI87HJsA/640?wx_fmt=jpeg)

**诶？！发现了什么？**  
这个图片，中间颜色浅，两边颜色深，这说明咱们的原图片中间的边界，在这里被反映出来了!

从上面这个例子中，我们发现，**我们可以通过设计特定的 filter，让它去跟图片做卷积，就可以识别出图片中的某些特征**，比如边界。  
上面的例子是检测竖直边界，我们也可以设计出检测水平边界的，只用把刚刚的 filter 旋转 90° 即可。对于其他的特征，理论上只要我们经过精细的设计，总是可以设计出合适的 filter 的。

**我们的 CNN（convolutional neural network），主要就是通过一个个的 filter，不断地提取特征，从局部的特征到总体的特征，从而进行图像识别等等功能。**

**那么问题来了**，我们怎么可能去设计这么多各种各样的 filter 呀？首先，我们都不一定清楚对于一大推图片，我们需要识别哪些特征，其次，就算知道了有哪些特征，想真的去设计出对应的 filter，恐怕也并非易事，要知道，特征的数量可能是成千上万的。

其实学过神经网络之后，我们就知道，**这些 filter，根本就不用我们去设计**，每个 filter 中的各个数字，不就是参数吗，我们可以通过大量的数据，来 **让机器自己去 “学习” 这些参数**嘛。这，就是 CNN 的原理。

二、CNN 的基本概念

**1.padding 填白**  
从上面的引子中，我们可以知道，原图像在经过 filter 卷积之后，变小了，从 (8,8) 变成了 (6,6)。假设我们再卷一次，那大小就变成了(4,4) 了。

**这样有啥问题呢？**  
主要有两个问题：

*   每次卷积，图像都缩小，这样卷不了几次就没了；
    
*   相比于图片中间的点，图片边缘的点在卷积中被计算的次数很少。这样的话，边缘的信息就易于丢失。
    

为了解决这个问题，我们可以采用 padding 的方法。我们每次卷积前，先给图片周围都补一圈空白，让卷积之后图片跟原来一样大，同时，原来的边缘也被计算了更多次。

![](https://mmbiz.qpic.cn/mmbiz_png/QLDSy3Cx3YLeVWnKYupUyuvPzDbtrJKGdg9TpCDUQyUUb1jQRn0HzYVlq0Q7V4d4Tj0bnTGibVlMmOgibD2O9amQ/640?wx_fmt=png)

比如，我们把 (8,8) 的图片给补成 (10,10)，那么经过(3,3) 的 filter 之后，就是(8,8)，没有变。

我们把上面这种 “让卷积之后的大小不变” 的 padding 方式，称为 **“Same”** 方式，  
把不经过任何填白的，称为 **“Valid”** 方式。这个是我们在使用一些框架的时候，需要设置的超参数。

**2.stride 步长**  
前面我们所介绍的卷积，都是默认步长是 1，但实际上，我们可以设置步长为其他的值。  
比如，对于 (8,8) 的输入，我们用 (3,3) 的 filter，  
如果 stride=1，则输出为 (6,6);  
如果 stride=2，则输出为 (3,3);（这里例子举得不大好，除不断就向下取整）

**3.pooling 池化**  
这个 pooling，是为了提取一定区域的主要特征，并减少参数数量，防止模型过拟合。  
比如下面的 MaxPooling，采用了一个 2×2 的窗口，并取 stride=2：

![](https://mmbiz.qpic.cn/mmbiz_png/QLDSy3Cx3YLeVWnKYupUyuvPzDbtrJKGS2EFcUsSJpVuEZTAPlQ6G5icbkOWhCS9Tqcic8Tl6fOGORObeicQHNQkg/640?wx_fmt=jpeg)

除了 MaxPooling, 还有 AveragePooling，顾名思义就是取那个区域的平均值。

**4. 对多通道（channels）图片的卷****积****（重要！）**  
这个需要单独提一下。彩色图像，一般都是 RGB 三个通道（channel）的，因此输入数据的维度一般有三个：**（长，宽，通道）**。  
比如一个 28×28 的 RGB 图片，维度就是 (28,28,3)。

前面的引子中，输入图片是 2 维的 (8,8)，filter 是 (3,3)，输出也是 2 维的 (6,6)。

如果输入图片是三维的呢（即增多了一个 channels），比如是 (8,8,3)，这个时候，我们的 filter 的维度就要变成(3,3,3) 了，它的 **最后一维要跟输入的 channel 维度一致。**  
这个时候的卷积，**是三个 channel 的所有元素对应相乘后求和**，也就是之前是 9 个乘积的和，现在是 27 个乘积的和。因此，输出的维度并不会变化。还是 (6,6)。

但是，一般情况下，我们会 **使用多了 filters 同时卷积**，比如，如果我们同时使用 4 个 filter 的话，那么 **输出的维度则会变为 (6,6,4)**。

我特地画了下面这个图，来展示上面的过程：

![](https://mmbiz.qpic.cn/mmbiz_png/QLDSy3Cx3YLeVWnKYupUyuvPzDbtrJKGGrMiaPPcKK2A0Kq2avjVHmKyCglsYKnmSxh8on68EX6rThq5uft3Deg/640?wx_fmt=jpeg)

图中的输入图像是 (8,8,3)，filter 有 4 个，大小均为 (3,3,3)，得到的输出为 (6,6,4)。  
我觉得这个图已经画的很清晰了，而且给出了 3 和 4 这个两个关键数字是怎么来的，所以我就不啰嗦了（这个图画了我起码 40 分钟）。

其实，如果套用我们前面学过的神经网络的符号来看待 CNN 的话，

*   我们的输入图片就是 X，shape=(8,8,3);
    
*   4 个 filters 其实就是第一层神金网络的参数 W1,，shape=(3,3,3,**4**), 这个 4 是指有 4 个 filters;
    
*   我们的输出，就是 Z1，shape=(6,6,4);
    
*   后面其实还应该有一个激活函数，比如 relu，经过激活后，Z1 变为 A1，shape=(6,6,4);
    

所以，在前面的图中，我加一个激活函数，给对应的部分标上符号，就是这样的：

![](https://mmbiz.qpic.cn/mmbiz_png/QLDSy3Cx3YLeVWnKYupUyuvPzDbtrJKG0pNDYia5M7fQjmRd4qDDibYFicANAcblrs1a3zQsy6lm9bzO5cKp9HxRw/640?wx_fmt=jpeg)【个人觉得，这么好的图不收藏，真的是可惜了】

三、CNN 的结构组成

上面我们已经知道了卷积（convolution）、池化（pooling）以及填白（padding）是怎么进行的，接下来我们就来看看 CNN 的整体结构，它包含了 3 种层（layer）：

**1. Convolutional layer（卷积层—CONV）**  
由滤波器 filters 和激活函数构成。  
一般要设置的超参数包括 filters 的数量、大小、步长，以及 padding 是 “valid” 还是“same”。当然，还包括选择什么激活函数。

**2. Pooling layer （池化层—POOL）**  
这里里面没有参数需要我们学习，因为这里里面的参数都是我们设置好了，要么是 Maxpooling，要么是 Averagepooling。  
需要指定的超参数，包括是 Max 还是 average，窗口大小以及步长。  
通常，我们使用的比较多的是 Maxpooling, 而且一般取大小为 (2,2) 步长为 2 的 filter，这样，经过 pooling 之后，输入的长宽都会缩小 2 倍，channels 不变。

**3. Fully Connected layer（全连接层—FC）**  
这个前面没有讲，是因为这个就是我们最熟悉的家伙，**就是我们之前学的神经网络中的那种最普通的层，就是一排神经元**。因为这一层是每一个单元都和前一层的每一个单元相连接，所以称之为 “全连接”。  
这里要指定的超参数，无非就是神经元的数量，以及激活函数。

接下来，我们随便看一个 CNN 的模样，来获取对 CNN 的一些感性认识：

![](https://mmbiz.qpic.cn/mmbiz_png/QLDSy3Cx3YLeVWnKYupUyuvPzDbtrJKGoy7Jia8eBVsFKnIkDjxYQyicE5icw0FPWWzAN37R4opkv6mjat8Lp7Z4Q/640?wx_fmt=jpeg)

上面这个 CNN 是我随便拍脑门想的一个。它的结构可以用：  
X→CONV(relu)→MAXPOOL→CONV(relu)→FC(relu)→FC(softmax)→Y  
来表示。

这里需要说明的是，在经过数次卷积和池化之后，我们 **最后会先将多维的数据进行 “扁平化”，**也就是把 **(height,width,channel)** 的数据压缩成长度为 **height × width × channel** 的一维数组，然后再与 **FC 层**连接，**这之后就跟普通的神经网络无异了**。

可以从图中看到，随着网络的深入，我们的图像（严格来说中间的那些不能叫图像了，但是为了方便，还是这样说吧）越来越小，但是 channels 却越来越大了。在图中的表示就是长方体面对我们的面积越来越小，但是长度却越来越长了。

四、卷积神经网络 VS. 传统神经网络

其实现在回过头来看，CNN 跟我们之前学习的神经网络，也没有很大的差别。  
**传统的神经网络，其实就是多个 FC 层叠加起来**。  
CNN，无非就是把 FC 改成了 CONV 和 POOL，就是把传统的由一个个神经元组成的 layer，变成了由 filters 组成的 layer。

那么，为什么要这样变？有什么好处？  
具体说来有两点：

**1. 参数共享机制（parameters sharing）**  
我们对比一下传统神经网络的层和由 filters 构成的 CONV 层：  
假设我们的图像是 8×8 大小，也就是 64 个像素，假设我们用一个有 9 个单元的全连接层：

![](https://mmbiz.qpic.cn/mmbiz_png/QLDSy3Cx3YLeVWnKYupUyuvPzDbtrJKG0a527yUMiakOUVdgCAp6cNecdk22tnnS2uz0VBMshux8Vm0RO5WZFJQ/640?wx_fmt=jpeg)

那这一层我们需要多少个参数呢？需要 **64×9 = 576 个参数**（先不考虑偏置项 b）。因为每一个链接都需要一个权重 w。

那我们看看 **同样有 9 个单元的 filter** 是怎么样的：

![](https://mmbiz.qpic.cn/mmbiz_png/QLDSy3Cx3YLeVWnKYupUyuvPzDbtrJKGw3KkWgoaRnErUqprTYibSOUSLJ51CZkpJDh7frL6tnYG3JeOW2EChZw/640?wx_fmt=png)

其实不用看就知道，**有几个单元就几个参数，所以总共就 9 个参数**！

因为，对于不同的区域，我们都共享同一个 filter，因此就共享这同一组参数。  
这也是有道理的，通过前面的讲解我们知道，filter 是用来检测特征的，**那一个特征一般情况下很可能在不止一个地方出现**，比如 “竖直边界”，就可能在一幅图中多出出现，那么 **我们共享同一个 filter 不仅是合理的，而且是应该这么做的。**

由此可见，参数共享机制，**让我们的网络的参数数量大大地减少**。这样，我们可以用较少的参数，训练出更加好的模型，典型的事半功倍，而且可以有效地 **避免过拟合**。  
同样，由于 filter 的参数共享，即使图片进行了一定的平移操作，我们照样可以识别出特征，这叫做 **“平移不变性”**。因此，模型就更加稳健了。

**2. 连接的稀疏性（sparsity of connections）**  
由卷积的操作可知，输出图像中的任何一个单元，**只跟输入图像的一部分有关**系：

![](https://mmbiz.qpic.cn/mmbiz_png/QLDSy3Cx3YLeVWnKYupUyuvPzDbtrJKGe6Yic2U6CV7pQDicmKudwmoxyhZHFBQxO9aYHQLOOic1YGXr2EZfzQicaQ/640?wx_fmt=png)

而传统神经网络中，由于都是全连接，所以输出的任何一个单元，都要受输入的所有的单元的影响。这样无形中会对图像的识别效果大打折扣。比较，每一个区域都有自己的专属特征，我们不希望它受到其他区域的影响。

**正是由于上面这两大优势，使得 CNN 超越了传统的 NN，开启了神经网络的新时代。**

- EOF -