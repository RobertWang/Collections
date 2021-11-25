> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/9CRFeHXXV9mG22idr9-XTg)

**二、常见三维表示方法介绍**
----------------

**Fig.1 立方体深度示意图**

深度图是一张 2D 图片，每个像素都记录了从视点（Viewpoint）到遮挡物表面（遮挡物就是阴影生成物体）的距离，相当于只有三维信息里 Z 轴上的信息，这些像素对应的顶点对于观察者而言是 “可见的”。

**2.2 体素（Voxel）**

![](https://mmbiz.qpic.cn/mmbiz_jpg/h2xc1OKOGckD60mQJVef09ZF8eS8A2HyBvicL1evwgIUAckrA1hYSpxvmaygSqXDajUwZ61nwaCh0lAyUI2ic6yw/640?wx_fmt=jpeg)

**Fig.2 游戏《我的世界》**

体素或立体像素是体积像素 (VolumePixel) 的简称，是模型数据于三维空间上的最小单位，是一种规则数据。体素概念上类似于二维图像中的像素，其本身并不含有空间中位置的数据（即它们的座标），然而却可以从它们相对于其他体素的位置来推敲。如图 Fig.2，是比较流行的一款 PC 端 3D 游戏《我的世界》，玩家可以在自己的世界中将一个个体素块任意堆叠，构建出自己专属的、个性的 3D 人物和世界。

**2.3 网格（Mesh）**

![](https://mmbiz.qpic.cn/mmbiz_png/h2xc1OKOGckD60mQJVef09ZF8eS8A2Hy35fiaekNice4gSg8YicGJVVLOicTCXvibzrqNQkr5Lds0sZAJNnhv9V6oIA/640?wx_fmt=gif)

**Fig.3 海豚网格图**

多边形网格（Polygonmesh）是三维计算机图形学中表示多面体形状的顶点与多边形的集合，是一种非规则结构数据。这些网格通常由三角形、四边形或者其它的简单凸多边形组成。其中，最常用的是三角网格，三角网格通常需要存储三类信息：顶点、边和面。

**顶点：**每个三角网格都有三个顶点，各顶点都有可能和其他三角网格共享。

**边：**连接两个顶点的边，每个三角网格有三条边。

**面：**每个三角网格对应一个面，我们可以用顶点或边的列表来表示面。

**2.4 点云（Point Cloud）**

![](https://mmbiz.qpic.cn/mmbiz_gif/h2xc1OKOGckD60mQJVef09ZF8eS8A2Hys7ibibEXtYCeDGL7iaJXVPNW07Qtbp4nQq22MP0q2iaPuHCLeRZxjt4tJQ/640?wx_fmt=gif)

**Fig.4 甜甜圈式点云图**

点云是指以点的形式记录的数据。每一个点包含了丰富的信息，包括三维坐标 X、Y、Z、颜色、分类值、强度值、时间等等。点云可以将现实世界原子化，通过高精度的点云数据可以还原现实世界。

那选哪个作为我们常用的 3D 模型表示呢？根据介绍我们可以知道 Voxel，受到分辨率和表达能力的限制，会缺乏很多细节；Point Cloud，点之间没有连接关系，会缺乏物体的表面信息；相比较而言 Mesh 的表示方法具有轻量、形状细节丰富的特点。

三维重建技术上大体可分为接触式和非接触式两种。其中，较常见的是非接触式中基于主动视觉的如激光扫描法、结构光法、阴影法和 Kinect 技术等，和基于机器学习的如统计学习法、神经网络法、深度学习与语义法等。

**三、基于主动视觉的三维重建技术 [7]**
-----------------------

**3.1 激光扫描法**

激光扫描法其实就是利用激光测距仪来进行真实场景的测量。首先，激光测距仪发射光束到物体的表面，然后，根据接收信号和发送信号的时间差确定物体离激光测距仪的距离，从而获得测量物体的大小和形状。

**3.2 结构光法**

**3.3 阴影法**

阴影法是一种简单、可靠、低功耗的重建物体三维模型的方法。这是一种基于弱结构光的方法，与传统的结构光法相比，这种方法要求非常低，只需要将一台相机面向被灯光照射的物体，通过移动光源前面的物体来捕获移动的阴影，再观察阴影的空间位置，从而重建出物体的三维结构模型。

**3.4 Kinect 技术**

Kinect 传感器是最近几年发展比较迅速的一种消费级的 3D 摄像机，它是直接利用镭射光散斑测距的方法获取场景的深度信息，Kinect 传感器如下图所示。Kinect 传感器中间的镜头为摄像机，左右两端的镜头被称为 3D 深度感应器，具有追焦的功能，可以同时获取深度信息、彩色信息、以及其他信息等。Kinect 在使用前需要进行提前标定，大多数标定都采用张正友标定法。

**四、基于 Pixel2Mesh 的三维重建技术**
---------------------------

Pixel2Mesh（Generating3D Mesh Models From Single RGB Images）

**4.1 整体框架**

![](https://mmbiz.qpic.cn/mmbiz_jpg/h2xc1OKOGckD60mQJVef09ZF8eS8A2HymqJusmvrteJa0ScqiaQMsmjoKiaIia7EavIXVu6liaH92FUicwibQuWd1mcg/640?wx_fmt=jpeg)

**Fig.5 网络架构图**

1. 首先给定一张输入图像：InputImage。

2. 为任意的输入图像都初始化一个固定大小的椭球体（三轴半径分别为 0.2、0.2、0.8m）作为其初始三维形状：Ellipsoid Mesh。

3. 整个网络可以分成上下两个部分：图像特征提取网络和级联网格变形网络。

1 上面部分负责用全卷积神经网络（CNN）[8] 提取输入图像的特征。

2 下面部分负责用图卷积神经网络（GCN）[9] 来提取三维 mesh 特征，并不断地对三维 mesh 进行形变，逐步将椭球网格变形为所需的三维模型，目标是得到最终的飞机模型。

4. 注意到图中的 PerceptualFeature Pooling 层将上面的 2D 图像信息和下面的 3Dmesh 信息联系在了一起，即通过借鉴 2D 图像特征来调整 3D mesh 中的图卷积网络的节点状态。这个过程可以看成是 Mesh Deformation。

5. 还有一个很关键的组成是 GraphUnpooling。这个模块是为了让图节点依次增加，从图中可以直接看到节点数是由 156-->628-->2466 变换的，这其实就 Coarse-To-Fine 的体现。

**4.2 图卷积神经网络 GCN**

我们先来看看图卷积神经网络 [6] 是如何提取特征的。一般的，对欧几里得空间中一维序列的语音数据和二维矩阵的图像数据我们会分别采取 RNN 和 CNN 两种神经网络来提取特征，那 GCN 其实就是针数据结构中另一种形式图的结构做特征提取。GCN 在表示 3D 结构上具有天然的优势，由前面我们知道 3D mesh 是由顶点、边和面来描述三维对象的，这正好对应于图卷积神经网络 G =（V，E，F），分别为顶点 Vertex、边 Edge 和特征向量 Feature。

图卷积的公式定义如下：

其中，f 表示一种传播规则；每个隐藏层 Hi 都对应一个维度为 Nxfi 的形状特征矩阵（N 是图数据中的节点个数，fi 表示每个节点的输入特征数），该矩阵中的每一行代表的是该行对应节点的 fi 维特征表征；A 是 NxN 的邻接矩阵。加上第 i 层的权重矩阵 fixfi+1，则有输入层为 Nxfi 时，输出维度为 NxNxNxfixfi+1，即为 Nxfi+1。在每一个隐藏层中，GCN 会使用传播规则 f 将这些信息聚合起来，从而形成下一层的特征，这样在每个连续层中图结构的特征就会变得越来越抽象。  

从以上两式可以看出图卷积神经网络的节点是根据自身特征和邻接节点的特征来进行更新的。

**4.3 Mesh Deformation Block**

作用：输入 2D CNN 特征和 3D 顶点位置、形状特征，输出新 3D 顶点位置、形状特征

![](https://mmbiz.qpic.cn/mmbiz_jpg/h2xc1OKOGckD60mQJVef09ZF8eS8A2Hynrws0HwiazATIUBWkbfYPxSAGDwcmo5r58WFYIJbybBsgD3ZFEoyiaxw/640?wx_fmt=jpeg)

**Fig.6 MeshDeformation Block**

为了生成与输入图像中显示物体所对应的 3D mesh 模型，Mesh Deformation Block 需要从输入图像中引入 2D CNN 特征（即图示 P），这需要将图像特征网络和当前网格模型中的顶点位置（Ci-1）相融合。然后将上述融合的特征与附着在输入图顶点上的 mesh 形状特征（Fi-1）级联起来，合并输入到基于 G-ResNet 模块中。G-ResNet 是基于图结构的 ResNet 网络，为每个顶点生成新的顶点位置坐标（Ci）和 3D 形状特征（Fi）。

**4.4 Perceptual Feature Pooling Layer**

作用：将 3D 顶点位置和 2D CNN 特征融合

![](https://mmbiz.qpic.cn/mmbiz_jpg/h2xc1OKOGckD60mQJVef09ZF8eS8A2HyUt3cib9tFY5bicvM8icvNBEPL4Dqdhia251fbJeWVbEfnXJcjxJP27mxvQ/640?wx_fmt=jpeg)

**Fig.7 PerceptualFeature Pooling Layer**

该模块根据三维顶点坐标从图像特征 P 中提取对应的信息，然后将提取到的各个顶点特征再与上一时刻的顶点特征做融合。具体做法是假设给定一个顶点的三维坐标，利用摄像机内参计算该顶点在输入图像平面上的二维投影，然后利用双线性插值将相邻四个像素点的特征集中起来，就可以输入到 GCN 中提取图结构特征。特别的是，将从 “conv3_3”、“conv4_3” 和“conv5_3”层提取的特征级联起来，得到总通道数为 1280（256+512+512）。然后将该感知特征与来自输入网格的 128 维 3D 特征相连接，从而得到 1408 的总维数。

**4.5 G-ResNet**

作用：用来提取图结构中的特征

在获得能表征三维 mesh 信息和二维图像信息的各顶点 1408 维特征后，该模型设计了一个基于 ResNet 结构的 GCN 来预测每个顶点新的位置和形状特征，这需要更高效的交换顶点之间的信息 [10]。然而，如四（2）中 GCN 介绍，每个卷积只允许相邻像素之间的特征交换，这大大降低了信息交换的效率。为了解决这个问题，通过短连接的的方法构建了一个非常深 G-ResNet 网络。在这个框架中，所有 block 的 G-ReNet 具有相同的结构，由 14 个 128 通道的图残差网络层组成。

**4.6 Graph Unpooling Layer**

作用：增加 GCNN 的顶点数目

![](https://mmbiz.qpic.cn/mmbiz_jpg/h2xc1OKOGckD60mQJVef09ZF8eS8A2HyXGM3M8GOvH0T6Ahl9Umq3YpR5RYxRTfAXX8FbUZEEKArFLj65nE6pw/640?wx_fmt=jpeg)

**Fig.8 GraphUnpooling 示意图**

因为每一个图卷积 block 本身顶点数量是固定的，它允许我们从一个顶点较少的网格开始，只在必要时添加更多的顶点，这样可以减少内存开销并产生更好的结果。一个简单的方法是在每个三角形的中心添加一个顶点 [11]，并将其与三角形的三个顶点连接。但是，这会导致顶点度数不平衡。受计算机图形学中普遍使用的网格细分算法中顶点添加策略的启发，我们在每条边的中心添加一个顶点，并将其与该边的两个端点连接起来（如 Fig.8.a）。新添加顶点的三维特征设置为其两个相邻顶点的平均值，如果将三个顶点添加到同一个三角形（虚线）上，我们还将它们连接起来。因此，我们为原始网格中的每个三角形创建 4 个新三角形，并且顶点的数量将随着原始网格中边的数量而均匀增加。

**4.7 Loss**

1.Chamfer Loss 倒角损失 [3]

![](https://mmbiz.qpic.cn/mmbiz_png/h2xc1OKOGckD60mQJVef09ZF8eS8A2HynUJuiaev5kYYtfuIPScESOicCQ1FFe7nVSZsSZ9du2sN5kM0QQQbUia7Q/640?wx_fmt=gif)

Chamfer distance 倒角距离是指两点之间的距离 lc，p 表示某具体节点，q 为 p 节点的邻居节点，目的是约束网格顶点之间的位置，将顶点回归到其正确方位，但是并不足以产生良好的 3D 网格。

2.Normal Loss 法向损失

![](https://mmbiz.qpic.cn/mmbiz_png/h2xc1OKOGckD60mQJVef09ZF8eS8A2HyE5fzZGyMP9lfmVvCo38Ev2wZsSX7nyoJonpfPZ0wbiaOdUbyIxalcGg/640?wx_fmt=gif)

Normal loss 需要顶点与其相邻顶点之间的边垂直于可观测到的网格真值，优化这一损失相当于强迫局部拟合切平面的法线与观测值一致。  

3.LaplacianRegularization 拉普拉斯正则化

![](https://mmbiz.qpic.cn/mmbiz_png/h2xc1OKOGckD60mQJVef09ZF8eS8A2HyWJ60ibAah8djDMbNx8CzhhiaRdx25MNXkicH5CZCFz3VbTNa1o6IMiajKA/640?wx_fmt=gif)

Laplacian Regularization 鼓励相邻的顶点具有相同的移动，防止顶点过于自由移动而避免网格自交，保持变形过程中相邻顶点之间的相对位置。

4.Edge Length Regularization 边长正则化

![](https://mmbiz.qpic.cn/mmbiz_png/h2xc1OKOGckD60mQJVef09ZF8eS8A2Hy2zrVucBg9egK1Rt1x2c3bA0lL77dwEial6F5dRd3Z2XM4yBVicWFDKjg/640?wx_fmt=gif)

Edge Length Regularization 作用是防止产生离群点，顶点间距离偏差过大从而约束边长。

最终 loss 为：

![](https://mmbiz.qpic.cn/mmbiz_png/h2xc1OKOGckD60mQJVef09ZF8eS8A2Hykcia2pPcYcZ8SGLKib42rxuuHd8LZXNplEKkd6a8RIGI0r9LKgJukibEA/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_png/h2xc1OKOGckD60mQJVef09ZF8eS8A2Hymia0SWpibuWH00YUzibLPCcubZW9nwjmmVx7dia8YDGIe9DicrAPV0cClTQ/640?wx_fmt=gif)

**五、小结**  

-----------

![](https://mmbiz.qpic.cn/mmbiz_jpg/h2xc1OKOGckD60mQJVef09ZF8eS8A2HyPdPmfnVdeRuRQSYPnFDSDEV39XCQV1k4Z0ibL1yibB9YeHIuLOxCA7Kw/640?wx_fmt=jpeg)

**Fig.9 Results**

受深度神经网络的限制，此前的方法多是通过 Voxel 和 Point Cloud 呈现，而将其转化为 Mesh 并不是一件易事，Pixel2Mesh 则利用基于图结构的神经网络逐渐变形椭圆体来产生正确的几何形状。本文着重介绍了 3D mesh 重建的背景、表示方法和 Pixel2Mesh 算法。文章贡献归纳如下：

（1）实现了用端到端的神经网络实现了从单张彩色图直接生成用 mesh 表示的物体三维数据

（2）采用图卷积神经网络来表示 3D mesh 信息，利用从输入图像提取到的特征逐渐对椭圆进行变形从而产生正确的几何形状

（3）为了让整个形变的过程更加稳定，文章还采用 Coarse-To-Fine 从粗粒度到细粒度的方式

（4）为生成的 mesh 设计了几种不同的损失函数来让整个模型生成的效果更加好

![](https://mmbiz.qpic.cn/mmbiz_gif/h2xc1OKOGckD60mQJVef09ZF8eS8A2HyQCSrOhGrH4EfjIibiaeicqSxKV5bib9oy4vjP1NC0dAF25EbC7zmSjVGpA/640?wx_fmt=gif)

**Fig.10 飞机 Mesh 效果**

![](https://mmbiz.qpic.cn/mmbiz_gif/h2xc1OKOGckD60mQJVef09ZF8eS8A2HyXTMoZvQA7MAxRH9WN4rGXkZ8clicA3aF1PM0FLrUKGibYTNeIzMyWEBw/640?wx_fmt=gif)

**_Fig.11_ _凳子 Mes__h_ _效果_**

**参考文献：**

1. Zhiqin Chen and HaoZhang. Learning implicit fifields for generative shape modeling. In _Proceedings of the IEEE Conference on Computer Visionand Pattern Recognition_, pages 5939–5948, 2019.

2. Angjoo Kanazawa,Shubham Tulsiani, Alexei A. Efros, and Jitendra Malik. Learningcategory-specifific mesh reconstruction from image collections. In _ECCV_, 2018.

3. Fan, H., Su, H., Guibas,L.J.: A point set generation network for 3d object reconstruction from a singleimage. In CVPR, 2017.

4. Choy, C.B., Xu, D., Gwak,J., Chen, K., Savarese, S.: 3d-r2n2: A unifified approach for single andmulti-view 3d object reconstruction. In ECCV, 2016.

5. Rohit Girdhar, DavidF. Fouhey, Mikel Rodriguez, and Abhinav Gupta. Learning a predictable andgenerative vector representation for objects. In _ECCV_, 2016.

6. Thomas N. Kipf and MaxWelling. Semi-supervised classifification with graph convolutional networks.In _ICLR_, 2016.

7. 郑太雄, 黄帅, 李永福, 冯明驰. 基于视觉的三维重建关键技术研究综述. 自动化学报, 2020, 46(4): 631-652. doi:10.16383/j.aas.2017.c170502

8. Lars Mescheder, MichaelOechsle, Michael Niemeyer, Sebastian Nowozin, and Andreas Geiger. Occupancynetworks: Learning 3d reconstruction in function space. In _CVPR_, pages 4460–4470, 2019.

9. Peng-Shuai Wang, Yang Liu,Yu-Xiao Guo, Chun-Yu Sun, and Xin Tong. O-cnn: Octree-based convolutional neuralnetworks for 3d shape analysis. _ACMTransactions on Graphics (TOG)_, 36(4):72, 2017.

10. Christian Hane,Shubham Tulsiani, and Jitendra Malik. Hierarchical surface prediction for 3dobject reconstruction. In _3DV_,2017.

11. Sunghoon Im, Hae-GonJeon, Stephen Lin, and In SoKweon. Dpsnet: End-to-end deep plane sweep stereo.In _ICLR_, 2018.

12. Chao Wen and Yinda Zhangand Zhuwen Li and Yanwei Fu: Multi-View 3D Mesh Generation via Deformation. InECCV, 2019.

**文章来自一点资讯 AI 图像图形实验室 (AIIG) 团队**