> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2OTQxMTM4OQ==&mid=2247514616&idx=3&sn=ed532e885a4f1f1b9c919d5893dad778&chksm=eae24caadd95c5bcafc38e12cb3ffc03dd36a55be993bc5444952f53fa29e419cdf778cb1d6c&mpshare=1&scene=1&srcid=0724UMIBoXCdlvZ8qyARccxY&sharer_sharetime=1627116667657&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

**项目介绍**  

===========

本项目是通过学习 https://gitee.com/nbsl/idCardCv 后整合 tess4j, 不需要经过训练直接使用的, 当然, 你也可以进行训练后进行使用。该项目修改原有的需要安装 opencv 的过程，全部使用 javaccp 技术重构, 通过 javaccp 引入需要的 c++ 库进行开发。不需要安装 opencv 新增的了前端控制识别区域的功能，新增了后端识别后验证 ，页面样式主要适应 paid，重新修改了后面的识别过程，用户 opencv 进行图片优化和区域 选择，使用 tess4j 进行数字和 x 的识别 配合样式中的区域在后台裁剪相关区域图片 /idCardCv/src/main/resources/static/js/plugins/cropper/cropper.css

**遇到问题**
========

**身份证号码识别**
===========

请求地址 http://localhost:8080/idCard/index 它基于 openCV 这个开源库。这意味着你可以获取全部源代码，并且移植到 opencv 支持的所有平台。它是基于 java 开发。它的识别率较高。图片清晰情况下，号码检测与识别准确率在 90% 以上。

**Required Software**
---------------------

本版本在以下平台测试通过：

*   windows7 64bit
    
*   jdk1.8.0_45
    
*   junit 4
    
*   opencv4.3
    
*   javaccp1.5.3
    
*   tess4j4.5.1
    
*   tesseract4.0.0
    

**项目更新**
========

1、先前使用 base64 进行图片的上传比较缓慢，使用 webuploader 插件进行分片上传，网速慢的时候可以提升速度，尤其是 paid 浏览器使用。原页面改为 idcard_bak.html。

2、原项目中有测试图片保存路径，统一更新到配置文档中。

**项目地址**
========

https://gitee.com/endlesshh/idCardCv