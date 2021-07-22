> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/rCtmW7tawr7zRIuMK1l5Iw)

> 创客大佬如何从头 DIY 一个超强码表

**X-TRACK 介绍**

X-TRACK 是由 FASTSHIFT 设计制作的一款开源 GPS 自行车码表。

其支持离线实时地图显示，支持记录和显示实时轨迹以及导出标准 GPX 格式的轨迹文件。

**硬件配置**  

1. 主控：AT32F403ACGU7

2. 显示：1.54 寸 IPS 屏，240x240 分辨率

3. 传感器：GPS + IMU + 地磁计

4. 输入设备：旋转编码器

![](https://mmbiz.qpic.cn/mmbiz_jpg/AVDJo94ibZ1ofhwzqjYLAoA0dpG3AksZTMeFozRKXaX962oZ7d9K9esY93bv2JdehIKMuDA07sKCvBU5eXc4kRw/640?wx_fmt=jpeg)

X-TRACK 硬件  

**开发环境**

软件开发环境分为两部分组成，Visual Studio 和 Keil MDK 。

1. 底层驱动部分，使用 Keil 进行开发调试。

2. 应用层代码基于 LVGL 模拟器运行，功能验证完成后，烧录到 MCU。

3. 两个工程复用同一套代码，通过 HAL 层进行驱动隔离。

**X-TRACK 软件架构**

X-TRACK 整体软件架构如下图所示，系统由 HAL、Framework 和 APP 三层组成 ：

1. HAL 层负责底层芯片驱动和传感器原始数据获取。

2. Framework 层是软件核心部分，包括 LVGL、页面调度和消息框架三个模块组成，分别负责图图形渲染、页面生命周期管理和消息分发处理。

3. APP 层为用户显示界面，由地图、表盘和系统信息三个部分组成。

![](https://mmbiz.qpic.cn/mmbiz_png/AVDJo94ibZ1pKmO8UghY4y75pFd39n3ibEhTLZK4tNIicgndDxGjuNylbF0u1zaicD5LIedtcMibNR6nnw2WlmZ3sfw/640?wx_fmt=png)

**Framework 层**

Framework 层作为整个软件核心，由三部分组成：LVGL、页面调度和消息框架 。

**LVGL 和页面调度**

LVGL 是开源的嵌入式图形库，其广泛应用嵌入式 GUI 中，提供了基础控件和底层图形渲染能力。

X-TRACK 使用 LVGL 作为图形库，提供用户显示界面，并使用了许多 LVGL 提供的基础控件。

例如在地图页面的轨迹显示中使用了 line 控件，地图显示使用了 image 控件，在表盘页面的页面切换选择，使用了 button 控件。  

由于 LVGL 自带页面管理功能较弱，需要规范页面行为，便于对页面生命周期进行管理。

页面调度器参考 **IOS ViewController** 进行设计，页面生命周期管理流程图如下：

![](https://mmbiz.qpic.cn/mmbiz_jpg/AVDJo94ibZ1pKmO8UghY4y75pFd39n3ibECcCsvAjNcQicLGibOrK2aAfHNHZZHn5SpZoB4icVcDCEeibBgZKsYluEGQ/640?wx_fmt=jpeg)

根据页面当前状态，对页面动作进行管理。例如地图页面的加载，在执行 onViewLoad (页面开始加载) 时初始化视图和数据；在 onViewDidAppear（页面即将显示） 执行过渡动画，在动画结束时显示地图容器。

在有外部按键事件触发后，依次执行 onViewWillDisappear(页面即将消失)、 onViewDidDisappear （页面消失完成）、onViewDidUnload（页面卸载完成）。

**消息框架**

消息框架提供数据的分发和处理。其使用订阅发布机制完成，将 HAL 层的接收到的传感器数据发布，转发给对应的订阅者进行数据处理。

以 GPS 为例，GPS 数据处理节点每秒读取一次卫星数据，然后发起 publish ，由消息框架将 GPS 数据推送给订阅者。

在运动数据处理节点中订阅 GPS 数据。在收到 GPS 数据之后，运动数据处理节点根据 GPS 数据计算总里程，平均速度等信息。在表盘页面中，拉取运动数据节点信息，将其显示在表盘页面中。

下图为 GPS 数据事件回调函数。

![](https://mmbiz.qpic.cn/mmbiz_png/AVDJo94ibZ1ofhwzqjYLAoA0dpG3AksZTbYCicB9ZYs4f78JIStjIoFcHficN3F9y46gL6syGVO9fBXyBccbhLwPA/640?wx_fmt=png)

**APP**

**“** 表盘 **”**

![](https://mmbiz.qpic.cn/mmbiz_png/AVDJo94ibZ1ofhwzqjYLAoA0dpG3AksZT7t3hKPSQ2vAHwNUxkZVhCQDlz9Qle3KDhCtgxFiaYibw3Y1pzMcdxeRw/640?wx_fmt=png)

1. 当前骑行数据显示

 2. 骑行信息汇总显示

3. 提供其他页面入口

表盘

表盘页由当前骑行速度、骑行信息和功能键三部分组成。  

当前骑行速度、骑行信息均为在显示页面的回调处理函数中订阅对应的数据源进行数据更新。

通过滚动编码器，可选择当前按钮光标，按下编码器进入相应页面，该部分功能由输入事件触发页面调度器进行页面切换。

  

**“** 地图 **”**

![](https://mmbiz.qpic.cn/mmbiz_jpg/AVDJo94ibZ1pKmO8UghY4y75pFd39n3ibErtcdgfWE8iaCKpYMthBJ1ibZdibP48ZLIqSUeEfk3t7qoiaaicgoEsvCLCA/640?wx_fmt=jpeg)

1. 实时位置显示

 2. 骑行信息显示

3. 运动实时轨迹显示 

地图

地图页由订阅 GPS 数据，通过 Tile System 转换为地图坐标数据，查找当前地图位置，将当前区域地图图片通过 LVGL 加载显示。

轨迹显示使用 LVGL 的 line 控件，将轨迹数据叠加在地图图层上，轨迹数据通过订阅 GPS 数据获取，将获取的数据过滤后传递给 line 控件进行显示。

注：文中所有图片素材和部分文字功能介绍由 FASTSHIFT 提供。  

  

✎ 参考链接 

  

****LVGL：**  
**https://github.com/lvgl/lvgl

**Bing Maps Tile System:**

https://docs.microsoft.com/en-us/bingmaps/articles/bing-maps-tile-system?redirectedfrom=MSDN

**IOS ViewController：**  

https://developer.apple.com/documentation/uikit/uiviewcontroller