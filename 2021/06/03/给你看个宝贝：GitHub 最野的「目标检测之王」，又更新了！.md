> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg2MjUyNjM3Nw==&mid=2247486431&idx=1&sn=2c2f60a0fcf9a150f92ab46701817eb9&chksm=ce07c519f9704c0f22552e7c340961ac2fa260e8964fbf4904b70999da82de25e78c2f4f4876&mpshare=1&scene=1&srcid=0601X68861pmbXHLE7RSeJlS&sharer_sharetime=1622554054842&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

图像理解任务复杂多样，单纯的目标检测已经不能满足你了？  

作为目标检测领域的扛把子，PaddleDetection 当然不仅仅提供通用目标检测算法，还拥有多个业界先进、实用的**关键点检测**和**多目标跟踪算法**。除了可以准确识别、定位目标，还可以对移动的目标进行连续跟踪、分析路径，甚至进行姿态、行为分析！

这么用心研发的高水准产品，还不赶紧收藏上车！

https://github.com/PaddlePaddle/PaddleDetection

![](https://mmbiz.qpic.cn/mmbiz_png/libMic1J2Kj4pzWpd6n6oNa9kdvQT4Boz5GayXEYpGx0xmLFBHR8mCDXWRm1nibuwAwfFVJia5DftwoLib2bEKIVcDQ/640?wx_fmt=png)

下面，让我们来详细看看 PaddleDetection 最近上新的这两样 “宝贝”：关键点检测和多目标跟踪，有什么厉害之处！

  

****关键点检测****

  

  

正如下图中展示的，关键点检测技术可以提取目标特定关键点的位置以及整体结构。以人体关键点检测为例，提取的信息则是人体关节位置以及关节点间的整体关联结构。

![](https://mmbiz.qpic.cn/mmbiz_jpg/libMic1J2Kj4pzWpd6n6oNa9kdvQT4Boz5Vp6Zib1PJWlUmyJyMCVW4tcuia8XnfWEMePUhLicn7pKp38Ojvr5MkHIQ/640?wx_fmt=jpeg)

（From PaddleDetection HRNet BodyKeypoint Detection Network）

人体关键点检测在安防、医疗、影视等领域中有极广泛的应用场景，例如进行打斗等异常动作、运动分析评估、演员动作捕捉与姿态迁移等等。

除了可以检测人体关键点外，手部关键点也因为是实现人机、人车交互中手势识别的关键，被各界开发者高度重视。

![](https://mmbiz.qpic.cn/mmbiz_png/libMic1J2Kj4pzWpd6n6oNa9kdvQT4Boz5icdXnenyevHgKr7d7wUuISpaia16b75ynG10TnpKQaTnh4h4mzhXHENQ/640?wx_fmt=png)

（From Google AI Blog On-Device, Real-Time Hand Tracking with MediaPipe）

而人脸关键点检测，更是人脸识别、脸部动作捕捉、迁移技术的基础支撑及核心关键。

![](https://mmbiz.qpic.cn/mmbiz_png/libMic1J2Kj4pzWpd6n6oNa9kdvQT4Boz5OxNxic2eMZE8k0ibgva1yDg5oFcUqQkGpfL32UoZngxL6JFcJpUrjN2Q/640?wx_fmt=png)

（From Qualcomm Snapdragon and Qualcomm Neural Processing SDK are products of Qualcomm Technologies, Inc. and/or its subsidiaries.）

还有很多业界大神基于关键点检测技术进行生物行为分析研究。

![](https://mmbiz.qpic.cn/mmbiz_png/libMic1J2Kj4pzWpd6n6oNa9kdvQT4Boz5qiaLKtuyhJP9DEIiczVTy8r58ETfw4ibBML5C8O3XxWOmD4kf9GQ5NtKg/640?wx_fmt=png)

（From ICCV paper：Cross-Domain Adaptation for Animal Pose Estimation）

**关键点检测技术这么神通广大，那它原理具体是怎样的呢**？

以人体关键点检测为例，主流的算法分为 **top-down** 和 **bottom-up** 两大类。

像下图所示的，先检测出人，再对每个人分别检测关键点，就是 Top-down 的方式。也就是先使用检测算法得出图中每一个人体所在位置，对单个人的区域进行截图，再使用 top-down 关键点算法对每个单人截图进行单人关键点位置检测，最后再根据截图位置映射回原图。HRNet 就是典型的 Top-Down 算法。

![](https://mmbiz.qpic.cn/mmbiz_png/libMic1J2Kj4pzWpd6n6oNa9kdvQT4Boz5IoEKylSZAh9bib9u4ado8icUs92nHR5EX89P0WbU1f7xAoqkQSAZ3Z6g/640?wx_fmt=png)

而 bottom-up 方法则相反：先对整图直接查找每一类关键点所在位置，然后各类关键点再根据特定规则进行 group 组合以区分属于的不同的人。近年比较优秀的算法 HigherHRNet 以及在其基础上的改进版 SWAHR 都属于这类算法。

![](https://mmbiz.qpic.cn/mmbiz_png/libMic1J2Kj4pzWpd6n6oNa9kdvQT4Boz5B9PIhibwhMMaoavSPICh3I5AMoY6Tx1fTQc8QzSbc04JGRbibkGiasFdQ/640?wx_fmt=png)

图片来源：

https://beyondminds.ai/blog/an-overview-of-human-pose-estimation-with-deep-learning/

无论你是明确的希望使用某一类方法，还是想要对两类方法进行对比、组合试验，PaddleDetection 都能很好的满足你！

下面的表格就是 PaddleDetection 提供的两类经典算法在 COCO 数据集、不同输入尺寸下的数据展示。

![](https://mmbiz.qpic.cn/mmbiz_png/libMic1J2Kj4pzWpd6n6oNa9kdvQT4Boz5cACqnwaiaJicKfUONsu5PialzmOvtJIhsfSDiatT5KzkAtONDlMqiafpgEg/640?wx_fmt=png)

  

****多目标跟踪****

  

  

多目标跟踪（Multiple Object Tracking，MOT）指的是在视频序列中同时检测多个目标的轨迹。它的应用范围也非常广泛，比如安防监控和自动驾驶等场景中的行人、车辆跟踪及轨迹分析等等。

![](https://mmbiz.qpic.cn/mmbiz_gif/yNnalkXE7oWsgVPxPPwgiaMA0j59HkP5KFyl2BgVHZEP2tUYPmnqfnaEeOHGQ5xOu3zfNFKZCZNnU52F9zibrE8w/640?wx_fmt=gif)

当前主流的 MOT 策略是 tracking by detection，这种策略将整个多目标跟踪任务拆分为两部分：检测 + Embedding。检测部分即针对视频，检测出每一帧中的潜在目标。Embedding 则将检出的目标分配和更新到已有的对应轨迹上（即 ReID 重识别任务）。根据这两部分实现的顺序，主流的多目标跟踪算法可以划分为 SDE 系列和 JDE 系列 2 类。

下面，就让我们详细看看**这两系列算法的区别：**

1.  **SDE 系列：**
    

这类算法完全分离检测和 embedding 两个环节，检测器和 embedding 模型（ReID）解耦串联。这样的设计可以使系统无差别的适配各类检测器，并使开发者可以针对两个模块分别调优。但由于是两个算法串联，这类方法的缺点也比较明显，那就是耗时较长，在构建实时 MOT 系统中面临较大挑战。

DeepSORT 是 SDE 系列算法的代表，它扩展了原有的 SORT(Simple Online and Realtime Tracking) 算法，增加了一个 CNN 模型对检测出的人像提取特征，在深度外观描述的基础上整合外观信息。

1.  **JDE 系列：**
    

JDE（Joint Detection and Embedding）即在共享神经网络中同时学习目标检测任务和外观嵌入任务。这类算法的训练过程被构建为一个多任务联合学习问题，兼顾精度和速度。

JDE 原论文是基于 Anchor Base 的 YOLOv3 新加一个 Reid head 学习外观 embedding 特征。

FairMOT 则是以 Anchor Free 的 CenterNet 为基础，由两个齐次的分支去用于预测像素级的目标分数和 ReID 特征，任务之间实现的公平性联合学习，实现高精度的实时跟踪。

**不要以为我们只是说说罢了，这里我们介绍的这 2 系列 3 种多目标跟踪（MOT）算法，在 PaddleDetection 中都提供了高性能实现。**

而采用业界通用多目标检测评估方法：将 6 个公开数据集组成一个大规模、多标签的联合数据集（包括 Caltech Pedestrian, CityPersons, CUHK-SYSU, PRW, ETHZ, MOT17 和 MOT16），其中 MOT16 作为评测数据集。而采用 MOTA 作为评估指标，结果如下表所示：

**在 MOT-16 Test Set 上结果**

![](https://mmbiz.qpic.cn/mmbiz_png/sKia1FKFiafggNHxyu9nbBB09OZvWjJzLfjNz1OLjJfeLyBHZXeNSsBbIxx9ro0V7Bj5ddYibtSYD8oNtfh7QyjuA/640?wx_fmt=png)

除此之外，PaddleDetection 还可以将关键点检测和多目标技术相结合，获取更多人体姿态相关信息。例如我们使用多目标跟踪算法 FairMot 获取行人位置及 id 信息，结合关键点检测 HRNet 算法检测行人关键点得到最终输出结果，得到如下图所示效果：

![](https://mmbiz.qpic.cn/mmbiz_gif/sKia1FKFiafgiaj6NibSAbaib8VqsT1BCUAx3mstbiaiaGXhyXzkXDicy3u0coGnoN4pFUdjKqlWVU4cicJAWeeV2gU6JXA/640?wx_fmt=gif)

  

（From PaddleDetection HRNet BodyKeypoint Detection Network：https://github.com/PaddlePaddle/PaddleDetection/blob/release/2.1/docs/images/mot_pose_demo_640x360.gif）

  

****PaddleDetection 全貌****

  

  

对 PaddleDetection 熟悉的小伙伴可能比较清楚，除了本篇介绍的关键点检测及多目标跟踪能力外，PaddleDetection 作为中国产业实践中目标检测领域一柄重器，能力可谓完备而强大，简单的可以概括为以下三个方面：

全明星算法阵容:

*   拥有超越 YOLOv4、YOLOv5 的 PP-YOLO 系列算法
    
*   1.3M 超超超轻量目标检测算法 PP-YOLO Tiny
    
*   全面领先同类框架的 RCNN 系列算法
    
*   ‍‍‍‍‍‍‍以及 SOTA 的 Anchor Free 算法：PAFNet（Paddle Anchor Free）
    

全功能覆盖：

除全系列通用目标检测算法外，额外覆盖旋转框检测、实例分割、行人检测、人脸检测、车辆检测等垂类任务。  

![](https://mmbiz.qpic.cn/mmbiz_png/libMic1J2Kj4pzWpd6n6oNa9kdvQT4Boz5iaLKLnh0QLDSVEN3jJFnI39Tlx88sia8WypZPTSZCsbytNf1FS5pfG8w/640?wx_fmt=png)

简单易用、全流程打通：

不仅全面支持动态图开发，可以顺畅的完成动静转化；还从数据预处理、算法训练调优、压缩、多端部署等全流程、各环节顺畅打通，极大程度地提升了用户开发的易用性，加速了算法产业应用落地的速度。  

你还在等什么？！如此用心研发的高水准产品，还不赶紧 Star 收藏上车！

**传送门：**

https://github.com/PaddlePaddle/PaddleDetection

![](https://mmbiz.qpic.cn/mmbiz_png/libMic1J2Kj4pzWpd6n6oNa9kdvQT4Boz5GayXEYpGx0xmLFBHR8mCDXWRm1nibuwAwfFVJia5DftwoLib2bEKIVcDQ/640?wx_fmt=png)

想要与官方开发团队交流？马上微信扫码加入 PaddleDetection 技术交流群！更多课程及产品动态，将在微信群里第一时间公告。

  

  

**飞桨 (PaddlePaddle)** 以百度多年的深度学习技术研究和业务应用为基础，是中国首个开源开放、技术领先、功能完备的产业级深度学习平台，包括飞桨开源平台和飞桨企业版。飞桨开源平台包含核心框架、基础模型库、端到端开发套件与工具组件，持续开源核心能力，为产业、学术、科研创新提供基础底座。飞桨企业版基于飞桨开源平台，针对企业级需求增强了相应特性，包含零门槛 AI 开发平台 EasyDL 和全功能 AI 开发平台 BML。EasyDL 主要面向中小企业，提供零门槛、预置丰富网络和模型、便捷高效的开发平台；BML 是为大型企业提供的功能全面、可灵活定制和被深度集成的开发平台。