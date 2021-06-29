> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NzM0MjcyMQ==&mid=2650120894&idx=3&sn=02347ebc7faeb5bd356f46fe2881f72f&chksm=beda79d089adf0c64253f820bbf418a412b8b22d6842b81f0bfeee9578b26dd781b9ecf9270d&mpshare=1&scene=1&srcid=06297AiCpPxcLlAduyvEscxZ&sharer_sharetime=1624940312726&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

文 | 局长  

出品 | OSC 开源社区（ID：oschina2013）

DevEco Studio 2.2 Beta1 已发布，新增低代码开发和远程真机特性。  

**新增特性：**

*   新增支持低代码开发功能，具有丰富的页面编辑功能，遵循 HarmonyOS JS 开发规范，支持通过可视化布局编辑器构建界面，极大地降低了用户的上手成本并且减少了用户构建界面的成本。具体请参考低代码开发。
    

![](https://mmbiz.qpic.cn/mmbiz_gif/dkwuWwLoRK9AWzSn8sp6LkQjHRQoibicBHHiaeicllp144icvSxKUCUP3ASZVT9XXf6HhEJz2xYrwVrODh06svbmSRA/640?wx_fmt=gif)

▲ 创建低代码页面演示图

![](https://mmbiz.qpic.cn/mmbiz_png/dkwuWwLoRK9AWzSn8sp6LkQjHRQoibicBHl7l6o2yexnqhWYHUslQQ5FVWCzeWeqpcQicTjv3t82L36SwPribtkb5g/640?wx_fmt=png)

▲ 低代码页面

![](https://mmbiz.qpic.cn/mmbiz_gif/dkwuWwLoRK9AWzSn8sp6LkQjHRQoibicBH5BbPNbLFLehz1cgAD3hNmKrOC6y4rPWtWaic6Ls6Bt08UGB94Kr2DJw/640?wx_fmt=gif)

▲ 实时预览效果图

*   新增 Remote Device 远程真机功能，支持 Phone 和 Wearable 设备，开发者使用 Remote Device 调试和运行应用时，同本地物理真机设备一样，需要对应用进行签名才能运行。具体请参考使用远程真机运行应用。
    

![](https://mmbiz.qpic.cn/mmbiz_png/dkwuWwLoRK9AWzSn8sp6LkQjHRQoibicBHFjrER0MfMSD6HCgQsqCDulLmAqU6PTmXyo5nTz19lOzoY3UBIzCwjA/640?wx_fmt=png)

▲ 远程真机示意图

**增强特性：**

*   HarmonyOS SDK 新增 API Version 为 6 的接口，Stage 为 Beta。
    
*   分布式模拟器功能增强，默认开启该特性，无需在 DevEco Labs 中手动开启。
    
*   HiTrace 日志跟踪分析能力增强，新增支持 timeline 视图和 events 视图。
    

**解决的问题：**

*   解决了限定词目录设置的屏幕密度与真实设备不一致时，预览界面（文本、图像等）会被缩放的问题。
    
*   解决了使用远程模拟器时，提示需要实名认证，实名认证完成后，仍然提示需要进行实名认证的问题。
    
*   解决了远程模拟器小概率出现列表中无法找到设备的问题。
    
*   解决了使用远程模拟器运行应用时，小概率出现无法找到已运行模拟器的问题。
    

DevEco Studio 2.2 Beta1 下载地址：https://developer.harmonyos.com/cn/develop/deveco-studio

END