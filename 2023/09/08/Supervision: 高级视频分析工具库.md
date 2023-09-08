> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/MMCy7D8EPQlOhYacranlVQ)

> 项目简介 Roboflow 的开源计算机视觉工具库 Supervision 更新了 \ x26lt; 高级视频分析 \ x26gt; 功能 \ x0d\x0a\x0d\x0a 带来了一系

项目简介
----

![](https://mmbiz.qpic.cn/sz_mmbiz_png/BOAjv711EFjia9nQb9XtiaRgwo0YlDBCM4aWzA7Ug04VsKibeo5VtQAMCJRy4mibQaWb07dm1EsdCusG8vuzDKiazyQ/640?wx_fmt=png)

Roboflow 的开源计算机视觉工具库 Supervision 更新了 <高级视频分析> 功能 带来了一系列令人兴奋的视频分析工具：

1. 视频跟踪器：可以在视频中追踪物体的移动。- ** 举例说明 **: 想象一下，你正在观看一场足球比赛的录像。视频跟踪器可以帮助你追踪球员的移动，甚至分析他们的表现。

2. 区域工具：可以让你选择视频中的特定区域进行分析。- ** 举例说明 **: 如果你想观察商店中某个货架上的商品销售情况，区域工具可以帮助你专注于那个特定的货架。

3. 注释器：可以让你在视频上添加文字、标签或其他信息。- ** 举例说明 **: 想象你正在制作一个烹饪教程视频。注释器可以让你在视频上添加食材名称、烹饪时间等信息，让观众更容易跟随。

Roboflow 是一个全方位的的计算机视觉平台，它提供了一整套工具来帮助工程师创建数据集、训练模型并部署到生产环境。该平台支持 40 多种注释和图像格式，并提供了过滤、标签、分割、预处理和增强图像数据的功能。Roboflow 还集成了 OpenAI、Meta AI 等的模型，并提供了一系列工具来组织视觉数据、自动化标签和部署基础模型。

安装

在 3.11>=Python>=3.8 环境下 pip 安装监管包。

```
pip install supervision[desktop]

```

在我们的指南中阅读有关桌面、无头和本地安装的更多信息。

快速启动
----

### 检测处理

```
>>> import supervision as sv
>>> from ultralytics import YOLO
>>> model = YOLO('yolov8s.pt')
>>> result = model(IMAGE)[0]
>>> detections = sv.Detections.from_ultralytics(result)
>>> len(detections)
5

```

### 数据集处理

```
>>> import supervision as sv
>>> dataset = sv.DetectionDataset.from_yolo(
...     images_directory_path='...',
...     annotations_directory_path='...',
...     data_yaml_path='...'
... )
>>> dataset.classes
['dog', 'person']
>>> len(dataset)
1000

```

### 模型评估

```
>>> import supervision as sv
>>> dataset = sv.DetectionDataset.from_yolo(...)
>>> def callback(image: np.ndarray) -> sv.Detections:
...     ...
>>> confusion_matrix = sv.ConfusionMatrix.benchmark(
...     dataset = dataset,
...     callback = callback
... )
>>> confusion_matrix.matrix
array([
    [0., 0., 0., 0.],
    [0., 1., 0., 1.],
    [0., 1., 1., 0.],
    [1., 1., 0., 0.]
])

```

项目链接
----

> https://github.com/roboflow/supervision