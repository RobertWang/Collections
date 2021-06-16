> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [arduino.nxez.com](https://arduino.nxez.com/2021/06/04/program-the-raspberry-pi-pico-with-the-arduino-ide.html)

> Arduino 中文站，提供丰富的 Arduino 教程和 DIY 资讯。

![](https://arduino.nxez.com/wp-content/uploads/2021/06/20210604220725355.jpg)

[树莓派 Pico](https://shumeipai.nxez.com/2021/01/21/raspberry-pi-silicon-pico-now-on-sale.html) 是今年树莓派基金会推出的微控制器（MCU）产品，其功能丰富且成本低，非常适合我们的项目。我们很多人都使用 Arduino IDE 来对微控制器进行编程。Python 和 C/C++ 都非常适合来进行 Pico 的编程。

而下面的要介绍的是使用 Arduino IDE 来作为树莓派 Pico 的开发环境，就和开发 Arduino 程序一样去做 Pico 上的开发，将树莓派 Pico 集成到 Arduino 的生态中。这样做的原因之一是可以直接使用丰富的 Arduino 底层库、允许集成模块、传感器和其他复杂的东西，而无需从头开始编写所有代码。

所以事不宜迟，让我们开始吧！  

### 准备好 Arduino IDE

通过菜单 Tools>Boards>Boards Manager 来搜索开发板，输入关键词 ‘pico’ 找到并下载安装到 Arduino IDE。

### 上传示例程序

将 Pico 连上电脑，菜单 Tools>Port 中选择 Pico 对应的 COM 端口。

![](https://arduino.nxez.com/wp-content/uploads/2021/06/20210604220725255.jpg)

在菜单 Files>Examples>Basics>Blink 中打开 Blink 示例程序，点击 Upload 上传到 Pico。

当 IDE 显示已经上传完成，就可以看到 Pico 板载的 LED 开始闪烁。

![](https://arduino.nxez.com/wp-content/uploads/2021/06/20210604220725386.jpg)

[via](https://www.instructables.com/Program-the-Raspberry-Pi-Pico-With-the-Arduino-IDE/)

[![](https://arduino.nxez.com/wp-content/uploads/2020/10/talk_quwj_banner_767.png)](https://talk.quwj.com/node/arduino "趣小组-Arduino")