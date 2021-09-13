> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5ODI5Njc2MA==&mid=2655852313&idx=2&sn=2847f8e57d73041d479cb5b90171be5c&chksm=bd755ace8a02d3d852d3bf3844e4fd565c7d1b57a545b87250726f0346fcf31fe17f64a30bd4&mpshare=1&scene=1&srcid=0913uZ5BQCQnRtadeVpndILW&sharer_sharetime=1631530210673&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

话说很久以前，我将 MicroPython 的解释器给 “挖” 了出来，然后做了适配，成功运行于鸿蒙设备（Hi3861）之上。

详见前一篇帖子：《[使用 Python 开发鸿蒙设备程序（0 - 初体验）](http://mp.weixin.qq.com/s?__biz=MzI3OTE3MTE2Mg==&mid=2247484173&idx=3&sn=988380d5cbff3106166dded560b12329&chksm=eb4a9a38dc3d132e4a04d6e81a53bc3953d4204965d1b693ad95d2b06eb36dfd08d49b544e1f&scene=21#wechat_redirect)》

然而，这在本质上也就只是一件装酷的事，除了写个 Hello World 体验资源受限设备上的 Python 语言程序设计之外，基本一无是处......

为了实现最初的梦想：通过 Python 降低鸿蒙设备开发的门槛。最近我又开始躁动起来，大刀阔斧的在之前工作的基础上做了 Python SDK 的设计和开发。终于，现在可以直接用 Python 来控制鸿蒙开发板外设了。

我之前的帖子《[<<鸿蒙开发板外设控制>> 直播图文版（2020.10.28）](http://mp.weixin.qq.com/s?__biz=MzkzNzE0MTA4OQ==&mid=2247484856&idx=1&sn=9f9f3b2094e0230b6fa374b65446d0ff&chksm=c292b3b6f5e53aa0f722f8832c85c8c508f23fe4b0bdfc35a36f6a7cb932b86272da7e01d37c&scene=21#wechat_redirect)》中涉及的案例都可以用 Python 完成！

大家看完这篇帖子后，可以尝试使用 C 和 Python 来实现相同的功能，体会一下不同。

OK！我们进入正题，直接上 Python 代码学习！

![](https://mmbiz.qpic.cn/mmbiz_jpg/bmc8z80Q8fBlDVk5ibOamEavRRaG2ewDZVtxpc7XpaSlI0jctL9ibTJvoNOQntzMqZjP1gagJdKNkfwOAvpsksoA/640?wx_fmt=jpeg)

在这里给大家做一点点概念上的科普，帮助大家更好的理解代码。GPIO（General Purpose Input/Output）即：通用型输入输出的简称。

其物理表现形式为：可接收或输出电信号的引脚，使用者可根据需要将其作为输入（GPI）或输出（GPO）使用。并且， 一般情况下，开发板上都有多个 GPIO 引脚供使用。

当 GPIO 作为输出使用时，输出的电信号为高电平（1）或者低电平（0），因此，只要在电路上稍加设计就可以接入外设（如：LED 灯，电动机，等），并通过程序控制外设的状态。

有了这些概念之后，上面的示例的代码理解起来就简单了！无非就是将 LED 灯（一种外设）接入第 7 号 GPIO 引脚，并通过程序设置第 7 号引脚输出高电平，点亮 LED 灯。

如果只是单纯的通过代码点亮一个 LED 灯，是真的不难，但也是真的挺无聊。所以，再给大家一个稍微复杂一点的示例：通过开关控制 LED 灯的状态。

“Show me the code!”

![](https://mmbiz.qpic.cn/mmbiz_jpg/bmc8z80Q8fBlDVk5ibOamEavRRaG2ewDZK9KbPybjeJ9VqwibhyqRxib5aKGthXNVmwVr1a7VINiafhOgOfn4IOJ4w/640?wx_fmt=jpeg)

这个示例看起来挺吓人的，比上一个示例复杂了一些。然而，本质却依旧是 GPIO 外设控制。

在硬件连接上，第 11 号 GPIO 引脚接入了一个按键，其目的是接收按键的信号，既然是接收信号那么显然 GPIO 基本功能应该设置为 “输入”（与连接 LED 的第 7 号 GPIO 基本功能设置相反）。

并且，将初始状态设置为高电平（pull up）态，当引脚电平从高电平转为低电平（按键被按下）时触发中断。

最后，设置中断触发后调用的函数为 button_callback，当这个函数被调用时会通过 GPIO_7 改变外接 LED 灯的状态。  

整个过程如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_jpg/bmc8z80Q8fBlDVk5ibOamEavRRaG2ewDZh6DUXD4kKMRgHhWypSicJAVmShyTglaopfVZ5wibgiaFBiaaeiaDNJiaMr4Q/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_gif/bmc8z80Q8fBlDVk5ibOamEavRRaG2ewDZ3FzRG41YyggBDWhTE2KqSToBs4oUu0SuEK1xCibBvBuSvlTmXqpQvIQ/640?wx_fmt=gif)

相信大家已经迫不及待想要动手实战，体验一下 Python 操作外设的快感了。

OK！方法如下：

*   下载附件中的 libdtpython.a 并存储到 /code/vendor/hisi/hi3861/hi3861/build/libs。
    
*   编写 Python 代码并使用工具 Txt2CStr.exe 转换为 C 数组。
    
*   将转换后的代码加入附件中的 demo 工程中编译并执行。
    

注意：

*   由于在 Python 中提供了 i2c 相关接口，因此，需要改动文件 user_config.mk
    
*   路径：/code/vendor/hisi/hi3861/hi3861/build/config/usr_config.mk
    
*   配置：CONFIG_I2C_SUPPORT=y
    

代码已经开源，记得给个星星哦！

```
https://gitee.com/delphi-tang/python-for-hos
```

Enjoy it!