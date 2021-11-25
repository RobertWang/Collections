> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [make.quwj.com](https://make.quwj.com/project/401)

> 我将提供完整的清单，它包含所有的功能，但你可以根据自己的需求来组装。

PCB 功能介绍
--------

![](https://make.quwj.com/storage/uploads/images/16347/1634723652wri0wgrcwi.jpg)  
我将提供完整的清单，它包含所有的功能，但你可以根据自己的需求来组装。  
如图所示，组件功能介绍如下（参考）：  
S - 1 开关电源模块，用于控制设备开关（必备）。  
S - 2 连续充电模块 ，断开充电器，防止设备重启  
S - 3 升压模， 将电池出来的电压升至 5 V  
S - 4 ESP32-S2 + 复位模块，控制器 （必备）。  
S - 5 振动模块 提供振动反馈（可选）。  
S - 6 时钟模块 断电后保持计时（可选）。  
S - 7 MP6050 / BME280 陀螺仪 / 温湿度（可选）。  
S - 8 扩展电源 设备关闭时可保持显示模块 S - 7 的功能（可选）。  
S - 9 红外模块 红外遥控器 （可选）。

![](https://make.quwj.com/storage/uploads/images/16347/1634723651f7u5l4qhot.jpg)  
我列出两个版本供大家参考，一个精简版和一个完整版。你可以根据自己的需求对组件进行了删减。  
![](https://make.quwj.com/storage/uploads/images/16347/1634710474cp5xylwcei.png)

PCB 与 3D 外壳打印
-------------

![](https://make.quwj.com/storage/uploads/images/16347/1634723652uuesuu7ooc.jpg)  
PCB 选定好功能后就可以开始搭建以及 3D 打印外壳。外壳总有五个部分。  
![](https://make.quwj.com/storage/uploads/images/16347/1634723652rygmooavf3.jpg)  
关于外壳 3D 打印文件可以在本项目文件库中下载：  
[https://make.quwj.com/project/401](https://make.quwj.com/project/401)  
PCB 总有三个部分。  
关于 PCB 3D 打印文件可以在本项目文件库中下载：  
[https://make.quwj.com/project/401](https://make.quwj.com/project/401)

焊接部分
----

![](https://make.quwj.com/storage/uploads/images/16347/16347236523f2v5qyxsu.jpg)  
在 PCB 中一共有八个模块，我采用分模块焊接的方式，先焊接三个模块的零件，这样有助于分步查错。  
![](https://make.quwj.com/storage/uploads/images/16347/1634723652h1wztmepoi.jpg)  
![](https://make.quwj.com/storage/uploads/images/16347/1634723652epawf3ryls.jpg)  
焊接顺序如下：

*   USB C 端口 > S - 2 > 电池连接器 >> 查看电池是否充电。
    
*   S-4 （暂不焊接 FPC 连接器） > 4 x 2 公头排针 >> 测试能否能够上传固件。
    
*   S-1 > S - 3 >> 连接电池并按 S-3 模块中的按钮，查看指示灯是否亮起。
    
*   焊接 FPC 连接器（外接显示器用）和 2 x 20 母头排针 > S-8 >> 键盘 >> 连接树莓派和电池，按住电源按钮查看树莓派是否启动。
    
*   S-6 > S-9 > S-7 >> 查看在 OS 的指导下，整个 PCB 板的是否工作正常。
    

组装部分
----

![](https://make.quwj.com/storage/uploads/images/16347/1634723652q97icblhhk.jpg)  
![](https://make.quwj.com/storage/uploads/images/16347/1634723652y3n1f58ert.jpg)  
![](https://make.quwj.com/storage/uploads/images/16347/1634723652h4lufiez8z.jpg)  
螺丝的型号如图所示。  
3D 打印外壳所需的螺丝的型号为：

*   22 mm x 2
    
*   9 mm x 2
    
*   6 mm x 4
    
*   8 mm x 1
    
*   10 mm x 1
    
*   16 mm x 1
    

尝试将 PCB 放入，毛边的地方需要用砂纸打磨。

线路连接及其他功能的扩展（可选）
----------------

![](https://make.quwj.com/storage/uploads/images/16347/1634723652mpsklfmtws.jpg)  
在 PCB 有一个扩展端口（2 x 10 pin 的排母），接入后可扩展一下六个功能：

*   无线电广播, LoRa 通信
    
*   自定义 Wifi
    
*   GPS
    
*   Micro SD
    
*   蓝牙
    
*   压力 + 湿度 + 温度模块
    
*   自定义专属模块
    

Nurolink / Dock 端口的扩展  
![](https://make.quwj.com/storage/uploads/images/16347/16347236511plnoslg12.jpg)  
该设备的 Nurolink / Dock 对接端口，可以用来供电、连接外部设备或外部电路。

如图所示，可连接到 Nurolink / Dock 端口的 GPS 模块。

同时也可通过 Nurolink / Dock 端口连接两个接口，需要将使用 USB C 和 USB C 的电缆进行改动，需要交换 D+ 和 D- 线，因为 TX 要连接至 RX，没有改动的线是 Tx 连接到 TX。

Nurolink / Dock 端口的 6 个引脚如下：

*   2 个引脚分别为 UART / TTL 引脚：Tx 和 Rx
    
*   2 个引脚分别为电源引脚：3v 和接地
    
*   2 个引脚分别为 I2c 引脚：SDL 和 SCL
    

可升级的地方
------

![](https://make.quwj.com/storage/uploads/images/16347/1634723654euddkqn9dm.jpg)  
DIY PC 的过程不会一帆风顺，仍有许多改进的地方，可以不断的优化项目。  
![](https://make.quwj.com/storage/uploads/images/16347/1634723652805ldyusmd.jpg)  
例如：  
Arduino 编码，可在固件中添加功能。  
Fusion 360，可改进支持且更容易构建。  
Python，可使附件充分发挥作用，例如基于 Lora 的通信应用程序能够与其他 PC 用户进行通信。  
FreeCAD，将其移植到 freeCAD，能够更加开源。  
KiCAD，能够更加开源。