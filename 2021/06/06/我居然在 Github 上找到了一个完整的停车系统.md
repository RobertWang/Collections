> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI4MjI1MTI0Mw==&mid=2247494582&idx=1&sn=d080a97f914711733ef24cb7b4550637&chksm=eb9e7132dce9f8242b1e20a71b21025f6166eb7c5b72a0bc96e86f2ab501a65671a01682e7d6&mpshare=1&scene=1&srcid=0606IkKQ0YrpOJAjFYXMDsP1&sharer_sharetime=1622991481862&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

开源最前线（ID：OpenSourceTop） 猿妹整编

大家好，我是 boy 哥。

最近，Github 热榜冲上来一个名叫 **--** 的项目，这应该是我见过的取名最随意的项目，也是目前看过的最完整的停车场系统。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/kOTNkic5gVBEqKHMSBmk59EdNDqBBtgMZkZ1ZkU04nsESEkABibbuPCA61Jria0GqlYEwGBFI5ZT0MSnDlVlMjRLw/640?wx_fmt=png)

停车场系统的运行流程也是比较直观的，具体如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/kOTNkic5gVBEqKHMSBmk59EdNDqBBtgMZ2ae4fJOnxy17beKxUgtzUxkNZv1Ex2HPADkiazDka7zxgEVfGYqpdSA/640?wx_fmt=jpeg)

这个停车系统具有以下功能特性：

*   兼容市面上主流的多家相机，理论上兼容所有硬件，可灵活扩展，②相机识别后数据自动上传到云端并记录，校验相机唯一 id 和硬件序列号，防止非法数据录入
    
*   用户手机查询停车记录详情可自主缴费 (支持微信，支付宝，银行接口支付，支持每个停车场指定不同的商户进行收款)，支付后出场在免费时间内会自动抬杆。
    
*   支持 app 上查询附近停车场 (导航，可用车位数，停车场费用，优惠券，评分，评论等)，可预约车位。
    
*   断电断网支持岗亭人员使用 app 可接管硬件进行停车记录的录入。
    

主要用到的技术架构如下：

*   后端开发语言 java，框架 oauth2+springboot2+doubble2.7.3
    
*   数据库 mysql/mongodb/redis
    
*   即时通讯底层框架 netty4，安卓和 ios 均为原生开发
    
*   后台管理模板 vue-typescript-admin-template
    
*   文件服务 fastDFS
    
*   短信目前仅集成阿里云短信服务
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/kOTNkic5gVBEqKHMSBmk59EdNDqBBtgMZQu8JFyvo5DxR32ic5L4P6ib1lYJFd4t3oribWE9PKQ5YcZibtzCGR3jWwg/640?wx_fmt=png)

**关于创建者**

创建者 4 年前曾就职于开发停车场系统的公司，发现目前国内该领域垄断，技术过于陈旧，没有一个规范，故个人用来接近 1 年的时间在业余时间开发出这种系统，现代化标准的互联网应用，定位大型物联网大数据云平台系统。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/kOTNkic5gVBEqKHMSBmk59EdNDqBBtgMZY8uUCwXy7RbPbpNibeoDFueKvgL1tTRMNuNRm3mGFQvT3QcvZibu6dIA/640?wx_fmt=png)

该项目代码完全开源，完全自主原创，创建者已经在 Linux 环境中测试过，而且出了详细的教程文档。

如果你不仅仅是想要学习系统代码，那你自行购置摄像头、道闸，再部署上这个系统，就能将这个停车系统付诸实践了。

参考来源：  

https://github.com/981011512/

https://www.showdoc.com.cn/902821218399318?page_id=4807644925521516