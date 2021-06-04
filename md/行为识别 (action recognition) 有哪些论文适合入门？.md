> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU2NTUwNjQ1Mw==&mid=2247503109&idx=2&sn=cb6769c8791b78769d0c89e6d8b5d6b4&chksm=fcb835ffcbcfbce958b0cd1b060b6553c3c4be57a8f456283e9f2b61b47a9a1d4a64326d2fe2&mpshare=1&scene=1&srcid=0603fx9sDezSsDEM9IKeZ6J9&sharer_sharetime=1622729902272&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> 链接：https://www.zhihu.com/question/33272629
> 
> 编辑：深度学习与计算机视觉
> 
> 声明：仅做学术分享，侵删

有基本的图像处理和计算机视觉的课程基础（修过数字图像处理、矩阵分析、机器学习、计算机视觉、模式识别、小波分析、最优化算法等课程），对行为识别领域感兴趣，不知道知乎的大神们是否能推荐几篇值得开始研读的行为识别领域的论文？有了开端，就可以沿着参考文献一直读下去仔细研究了。但是现在对该领域还不了解，不知道从哪些论文开始着手比较好？

  

**作者：Xiaolong Wang  
https://www.zhihu.com/question/33272629/answer/60279003**

有关 action recognition in videos, 最近自己也在搞这方面的东西，该领域水很深，不过其实主流就那几招，我就班门弄斧说下 video 里主流的：

Deep Learning 之前最 work 的是 INRIA 组的 Improved Dense Trajectories(IDT) + fisher vector, paper and code:  
LEAR - Improved Trajectories Video Description：https://lear.inrialpes.fr/people/wang/improved_trajectories  
基本上 INRIA 的东西都挺 work 恩..

然后 Deep Learning 比较有代表性的就是 VGG 组的 2-stream:  
http://arxiv.org/abs/1406.2199  
其实效果和 IDT 并没有太大区别，里面的结果被很多人吐槽难复现，我自己也试了一段时间才有个差不多的数字。

然后就是在这两个 work 上面就有很多改进的方法，目前的 state-of-the-art 也是很直观可以想到的是 xiaoou 组的 IDT+2-stream:  
http://wanglimin.github.io/papers/WangQT_CVPR15.pdf

还有前段时间很火，现在仍然很多人关注的 G 社的 LSTM+2-stream:  
http://static.googleusercontent.com/media/research.google.com/zh-CN//pubs/archive/43793.pdf

然后安利下 zhongwen 同学的 paper:  
http://www.cs.cmu.edu/~zhongwen/pdf/MED_CNN.pdf

最后你会发现 paper 都必需和 IDT 比，然后很多还会把自己的 method 和 IDT combine 一下说有提高 恩..

**作者：水哥  
https://www.zhihu.com/question/33272629/answer/60163859**

视频方面的不了解，可以聊一聊静态图像下的~  
[1] Action Recognition from a Distributed Representation of Pose and Appearance, CVPR,2010

[2]Combining Randomization and Discrimination for Fine-Grained Image Categorization, CVPR,2011

[3] Object and Action Classification with Latent Variables, BMVC, 2011

[4] Human Action Recognition by Learning Bases of Action Attributes and Parts, ICCV, 2011

[5] Learning person-object interactions for action recognition in still images, NIPS, 2011

[6] Weakly Supervised Learning of Interactions between Humans and Objects, PAMI, 2012

[7] Discriminative Spatial Saliency for Image Classification, CVPR, 2012

[8] Expanded Parts Model for Human Attribute and Action Recognition in Still Images, CVPR, 2013

[9] Coloring Action Recognition in Still Images, IJCV, 2013

[10] Semantic Pyramids for Gender and Action Recognition, TIP, 2014

[11] Actions and Attributes from Wholes and Parts, arXiv, 2015

[12] Contextual Action Recognition with R*CNN, arXiv, 2015

[13] Recognizing Actions Through Action-Specific Person Detection, TIP, 2015

2010 之前的都没看过，在 10 年左右的这几年（11,12）主要的思路有 3 种：

1. 以所交互的物体为线索（person-object interaction），建立交互关系，如文献 5,6；

2. 建立关于姿态（pose）的模型，通过统计姿态（或者更广泛的，部件）的分布来进行分类，如文献 1,4，还有个 poselet 上面好像没列出来，那个用的还比较多；

3. 寻找具有鉴别力的区域（discriminative），抑制那些 meaningless 的区域，如文献 2,7。10 和 11 也用到了这种思想。

文献 9,10 都利用了 SIFT 以外的一种特征：color name，并且描述了在动作分类中如何融合多种不同的特征。

文献 12 探讨如何结合上下文（因为在动作分类中会给出人的 bounding box）。

比较新的工作都用 CNN 特征替换了 SIFT 特征（文献 11,12,13），结果上来说 12 是最新的。

静态图像中以分类为主，检测的工作出现的不是很多，文献 4,13 中都有关于检测的工作。可能在 2015 之前分类的结果还不够 promising。现在 PASCAL VOC 2012 上分类 mAP 已经到了 89%，以后的注意力可能会更多地转向检测。

视频的个别看过几篇，与静态图像相比，个人感觉最大的区别在于特征不同。到了中层以后，该怎么做剩下的处理，思路还是差的不远。

**作者：木比白  
https://www.zhihu.com/question/33272629/answer/231800319**

今天调研了 CVPR2017 的论文，对 2 维图像的视频流（非骨架结构）、深度学习方法的行为识别论文进行了调研。可以从这方面入手去阅读，在 Related Work 中找前人工作去学习。

1、Deep Temporal Linear Encoding Networks 该论文是对 two-stream ConvNets 的一种补充与改进。

2、Spatiotemporal Multiplier Networks for Video Action Recognition

3、Spatiotemporal Pyramid Network for Video Action Recognition

4、Spatio-Temporal Vector of Locally Max Pooled Features for Action Recognition in Videos

5、Asynchronous Temporal Fields for Action Recognition

6、AdaScan:Adaptive Scan Pooling in Deep Convolutional Neural Networks for Human Action Recognition in Videos

从最新的文章读，读不懂就看 related work，毕竟是从前人工作的基础上进行改进和创新。

  

**☆ END ☆**

如果看到这里，说明你喜欢这篇文章，请转发、点赞。微信搜索「uncle_pn」，欢迎添加小编微信「 mthler」，每日朋友圈更新一篇高质量博文。