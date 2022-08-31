> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/d7dcf46f3705)

相关
--

[ARKit 实战：如何实现任意门](https://www.jianshu.com/p/b377e8b8a8f2)  
[Animoji 实现方案分享](https://www.jianshu.com/p/04dd3ec22cbf)  
[Animoji 模型优化方案总结](https://www.jianshu.com/p/1ed810c02532)  
[ARKit 进阶：物理世界](https://www.jianshu.com/p/d7dcf46f3705)  
[ARKit 进阶：材质](https://www.jianshu.com/p/07b96c800a1d)  
[解决 ARKit 用 Metal 录制时颜色变暗的问题](https://www.jianshu.com/p/25a1823f857c)  
[记一个 SceneKit Morpher 引发的 Crash](https://www.jianshu.com/p/3944ff013112)

写在前面
----

> 其实准备 ARKit 已经很久了，确切地说当 WWDC 开始介绍时就开始了。其后参加了苹果的 ARKit workShop，加上自己有点事，所以文章一直没发出来，现在再发一篇上手文章，也没什么意义。所以本篇文章重在 workShop 上与苹果工程师的交流和我对 ARKit 的理解, 最后会简单介绍一下相关技术。

更新
--

苹果最近更新了 ARKit 的文档，加入了基于深度摄像头的人脸识别（目前就是 iPhone X 的前置摄像头）。在 work shop 上 ARKit 团队明确表示不会支持前置摄像头，这么快就被打脸…… 所以大家可以期待，ARKit 将在前置摄像头上有更多的应用。

Work Shop Demo
--------------

MarkDown 无法传视频，这里是[视频链接](https://www.jianshu.com/p/31986356b680)。  

![](http://upload-images.jianshu.io/upload_images/680728-d43303e1662d2dc4.png)

ARKit
-----

![](http://upload-images.jianshu.io/upload_images/680728-f52e6329e1973eb6.png) ARKit Logo

AR(Augment Reality) 大家都知道，就是将 3D 模型渲染在摄像头图像之上，混合渲染达到虚拟物品就好像是现实的一部分。ARKit 解决了模型定位难的问题，结合 CoreMotion 运动数据与图像处理数据，来建立一个非常准确的 SLAM 系统，构建虚拟世界和现实世界之间的映射。同时能够分析环境自动给模型添加光源，实际效果还是比较惊艳的。

从结构上看，ARKit 提供了一套简单易用的 AR 框架，但框架之外，需要很多的三维空间、游戏编程、3D 模型、GPU 渲染的知识来理解 AR 技术。ARKit 最重要的两个类是

`ARSession`

与

`ARSCNView`

![](http://upload-images.jianshu.io/upload_images/680728-d13227fb13d3069a.png) session-view

类似与 AVFoudation，ARKit 中由 ARSesseion 类来配置 SLAM 系统的建立。设置 RSession 的配置选项为 ARWorldTrackingSessionConfiguration 来追踪设备的方向与位置，并且能够检测平面。这个选项只有 A9 处理器之上才支持。其他型号处理器（6S 以下）只能追踪设备的方向。  
ARKit 的提供了自带的两个渲染类：ARSCNView 和 ARSKView，后者用来渲染 2D 模型。之前鲜有问津的 SceneKit 算是有了用武之地。这两个类会自动开启摄像头并建立虚拟空间与现实空间之间的映射。同时 ARKit 也支持自定义用 OpenGL 或 Metal 实现渲染类，但要自己管理与 ARSession 之间的通信，同时要遵循 iOS GPU 命令不能在后台调用的规则。

其他比较重要的类有`ARAnchor`、`ARHitTestResult`、`ARFrame`、`ARCamera`。

*   ARAnchor

世界中点，可以用来放置虚拟物品，也可以代指现实物品的放置位置。ARAnchor 在世界中是唯一的，并包含仿射变换的信息。

*   ARHitTestResult

HitTest 的返回，世界中的 ARAnchor。  
与 UIKit 中的 hitTest 不同，ARKit 的 HitTest 以设备方向配合视图坐标，建立一条世界中的射线，所有在射 线上的 ARAnchor, 会以由近到远的方式返回。此外 SCeneKit 的 HitTest 返回虚拟物品。

*   ARFrame

摄像头视频帧的包装类，包含位置追踪信息、环境参数、视频帧。重点是它包含了苹果检测的特征点，通过 rawFeaturePoints 可以获取，不过只是特征的位置，具体的特征向量并没有开放。

*   ARCamera

场景中的摄像机，用来控制模型视图变换和投影变换。同时提供 6DOF(自由度信息，方向 + 位置) 与追踪信息。

对 ARKit 的思考
-----------

从框架接口来看，ARKit 暴露出来的能力并不多且小心翼翼。  
AR 的能力，由三部分组成：

1.  3D 渲染
2.  空间定位与方向追踪
3.  场景理解（检测与识别）

目前看 ARKit 只提供了 3D 渲染的入口，其他两个都被封装起来了，所以目前来看渲染是差异化的主要途径，但不唯一。ARKit workShop 上，面对大家提出的苛刻问题，苹果工程师大量提到特征点。其实计算机视觉是可以在场景理解这一层面做一些自定义的。如果苹果开放更多的能力，那 AR 的能力完全可以作为任何一个 APP 的特性。  
此外，ARKit 还存在一些问题：

*   ARKit 是基于惯性 - 视觉来做空间定位的，需要平稳缓慢的移动 + 转向手机，才能构建更加准确的世界，这对用户来说是一种考验，需要积极提示。
*   理论上 ARKit 在双目摄像头上的表现应该优于单目，这里需要具体测试，如何来平衡用户体验。
*   .scn 文件还是知识一个简单的 3 维模型编辑器，支持的文件格式少，对模型、光照的编辑方式不太友好。对骨骼动画的支持还有只在能用的阶段。
*   一旦刚开始检测平面失败，出现时间久，飘逸的现象，后期很难再正确检测，要强制重启。

ARKit 最佳实践
----------

### 模型与骨骼动画

*   如果是使用. dae 转 .scn 文件，资源中包含骨骼动画时，加载. scn 文件到 scene 中会丢失动画，需要在加载时手动恢复一下（[方法](https://links.jianshu.com/go?to=https%3A%2F%2Fstackoverflow.com%2Fquestions%2F41130425%2Fscenekit-load-node-with-animation-from-separate-scn-file%2F44458805%2344458805)）。
*   设计骨骼动画是，要求设计师把动画放在根节点上，不要分散地放在每个 bone 上，这样可以方便地读取出动画到`CAAnimation`。
*   最好不要将太远的光照加载模型文件中，这样会导致加载文件到`SCNNdoe`时，你的 node 真实尺寸特别大，而你期望的尺寸可能只是模型对象的大小。
*   模型的`SCNMaterial` 是用 physically based lighting model 会有更好的表现，设置比较好的环境光也比较重要。

### 光照

*   合理的阴影会大大提高 AR 的效果，贴一张纹理当然可以，但动态阴影更让人沉浸，我们还是要有追求的。
*   使用 Bake ambient occlusion（ABO）效果，模型会更逼真。
*   光照 node 加载到 `SCNScene`的`rootNode`上，这对做碰撞检测尤其重要

ARKit workShop
--------------

汇总了一下 workShop 上，比较感兴趣的问题和苹果工程师的回答，掺杂自己的理解。

1 . ARFrame 提供的 YUV 特征，如何获取 RGB 特征？

答：使用 Metal 去获取特征点的 RGB 值。  
（这个我一般是用 OpenGL 的 shader 去做，我想苹果工程师是说将图像用 Metal 转成位图后，根据坐标去获取 RGB 值。但特征点不多的话，直接在 CPU 中利用公式计算一下不就行了吗？不过也许 Metal 有更强大的方法。）

2 . ARKit 中怎么做虚拟环境？

答：利用 Cube 背景。  
（这个在 VR 中用的比较多，就是用一个贴满背景的立方体包裹住摄像机所在的空间，网上的资料较多。）

3 . ARKit 的如何模拟光源的？为什么不产生阴影。

答：ARKit 通过图像的环境来设置模型的环境光强度，而环境光是不产生阴影的。  
（我猜苹果应该是通过像素值来确定环境光的，如果用高级一点的方法完全可以添加直射光。光照有许多模型，只有带方向的光才会产生阴影，如果想用 ARKit 做出阴影，可以看[我的回答](https://links.jianshu.com/go?to=https%3A%2F%2Fstackoverflow.com%2Fquestions%2F30975695%2Fscenekit-is-it-possible-to-cast-an-shadow-on-an-transparent-object%2F44774424%2344774424)。）

4 . AVFoudation 与 ARSession 之间的切换会有问题吗？

答： `ARSession`底层也是用`AVFoudation`的，如果重新打开 ARKit，只需要重新 run 一下 `ARSession` 可以了，但切换时会有卡顿。  
（我自己试了一下，切换时确实有轻微的卡顿，切换后 ARSession 就停止摄像头采集了，但 3D 渲染会继续，只是丧失了空间定位与检测识别的能力。）

5 . ARKit 是否支持前置摄像头？

答：不支持。ARKit 并不是一个用于前置摄像头环境的技术，因为空间有限，能提供的信息也非常有限。  
（这个问题是很多参会者关心的问题，但 ARKit 团队似乎不是很 care ，说到底还是因为前置摄像头的场景中，用户很少会移动，画面中一般大部分都是人脸，这样 ARKit 的定位与检测能力无法很好使用。建议由类似需求的同学好好梳理，是不是想要的是 3D 渲染而不是 AR。

6 . ARKit 的最大应用范围是多少？

答：100 米是 ARKit 在保持较好用户体验的最大测量距离。  
（这个其实我有点没太听清，实际数字应该是 100 米以上）

7 . ARKit 如何做 marker？

答：ARKit 不会提供这样的能力，如果想实现的，可以用 ARKit 提供的特征点来跑自己的计算机视觉。  
（熟悉计算机视觉的同学应该都明白，其实 marker 就是一种简单的图像识别，如果 ARKit 提供的特征点可靠的话，完全可以自己做特征匹配。现场问了苹果工程师，他们的特征点是什么特征，他们不愿回答，不过看使用场景的话，应该是一种边缘敏感的低维特征，应该类似 PCA + SURF）。

8 . ARKit 合适支持 A8？性能如何？

答：支持 A8 处理器并不在计划中（这里指的是空间定位能力，A8 只支持空间方向追踪），ARKit 的大部分计算都是在 CPU 上处理的，在 A8 处理器上的性能损耗在 15% ~ 25%, 在 A9 处理器上的性能损耗在 10% ~ 15%。  
（看他们的意思，大量的计算，在 A8 上应该是比较低效的，解释了为什么 A8 上的追踪能力是阉割版的。性能应该说还不错，与游戏类似）

9 . 如何追踪实际的物体？

答：可以在已识别的物体位置上，添加一个 node, 这样就能在之后的处理中一直保持这个物体的追踪。  
（这次的 wrokShop，苹果大量提到他们的特征点，如果他们真的足够重视的话，应该开放特征检测的过程与特征向量，希望后期能够开放吧）

10 . 如何连接两个不同 ARKit 世界？

答：ARKit 没有计划支持这些，比较 tricky 的做法是将两个手机紧挨着启动 ARKit。  
（这个也是很多参会者关注的问题，相信不少人已经有了自己的解决方案，这里我后期会出一篇文章讲解。）

AR 相关
-----

### 渲染

AR 说到底还是一种游戏技术，AR 提供了定位、检测平面的功能，这些功能并没有暴露出来供我们自定义，那么只能在渲染方面做出差异。  
目前 ARKit 支持的 3D 渲染引擎，有 sceneKit，Unity3D，UE。后两者都是成熟的游戏引擎，能够提供完整的游戏功能，但没有我们没有使用，主要因为：

1.  上手较慢，iOS11 9 月中旬就要发布了，时间紧促。
2.  接入 Unity3D 会给安装包造成很大压力，成本大约 10M。

最终决定还是用 sceneKit，主要出于一下考虑：

1.  ARKit 目前对 Unity3D，UE 的支持没有 sceneKit 好。
2.  sceneKit 用 OC 写，可以 OCS。
3.  sceneKit 是系统动态库，对安装包压力不大。
4.  sceneKit 虽然能力弱，但是对于 AR 来说足够了，AR 毕竟打造不了复杂的游戏。

### 坐标系

ARKit 和 OpenGL 一样，使用右手坐标系, 这个新建一个 camera 就可以看出来。

![](http://upload-images.jianshu.io/upload_images/680728-92da8e81f4b7b4fc.png) camera -w500

### 定位

将模型加载到空间中，需要 6 个自由度（6DOF）的信息来指定模型的表现：

![](http://upload-images.jianshu.io/upload_images/680728-01fbef58ed8e5395.png) 6DOF

分别是沿三个坐标轴的平移与旋转。  
可以使用旋转矩阵、欧拉角、四元数来定义空间旋转，ARKit 的这三种方式均有运用。

*   旋转矩阵

这个好理解，使用旋转的变换矩阵即可，维度 4*4，定义一次旋转需要 16 个数。

*   欧拉角

把空间旋转分解成绕三个局部坐标轴的平面旋转，分别是 pitch(俯仰角，绕 x 轴)，yaw（偏航角，绕 y 轴），roll（翻滚角，绕 z 轴），然后以一定顺序做旋转（sceneKit 中是 roll -> yew -> pitch），欧拉角是使用三个 3*3 矩阵连乘实现，而且存在万向锁的问题。

![](http://upload-images.jianshu.io/upload_images/680728-d2d49cbc5a550e39.png) rotation matirx

当 pitch 为 90° 时，pitch 与 yew 的旋转轴重合了，这时飞机丧失了一个旋转的维度。

![](http://upload-images.jianshu.io/upload_images/680728-3d5ad27cb380cee8.png) eulars1 angle

![](http://upload-images.jianshu.io/upload_images/680728-865233b8ade6054f.png) eulars2 angle

*   四元数

将三维空间的旋转表示成四维空间的超球面上位移, 概念有点复杂。简单来说，我们只需要旋转轴 ![](https://math.jianshu.com/math?formula=%5Coverrightarrow%7Bu%7D%3D%20(x%2C%20y%2C%20z)) ，和角度 𝛉 来构造一个单位四元数 q:  

![](http://upload-images.jianshu.io/upload_images/680728-83bf8cc74dbef051.png) quaternion

那么旋转可以定位为：  

![](http://upload-images.jianshu.io/upload_images/680728-36b67b1d9bc6d8eb.png) quaternion result

对任何需要旋转的点 , 将它扩展成一个纯四元数 , 代入上面的公式，就可以得到旋转后的点。

### 追踪

visual-inertial odometry : 基于视觉和惯性的测量方法，惯性数据是指角速度和加速度，这些都由`Core Motion`提供，加上图像特征，能够更准确地建立 SLAM 系统。ARKit 会将提取到的特征点映射的空间中，也就是说特征点是由三维坐标的，我们可以利用特征点来确定图像中物体的远近。实测效果不错，误差在分米以内。