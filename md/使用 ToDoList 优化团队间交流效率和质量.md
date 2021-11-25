> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7003356101966053413) 。 通过该方法结合`钉钉文档`的形式，我想到了既可以一次性整合所修改的内容又可以实时看到修改进度，还可以进行评论和回复的方案。

**理论原型是这样的：**

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/00f42eb5874a43a7a0d5b25f8267b50f~tplv-k3u1fbpfcp-watermark.awebp)

解决方案
----

经过多次优化后我完善了我的方案，总体如下：

第一步：建立钉钉文档（当然其他软件应该也有对应功能）

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f9b78cd945bb4a0bbffe616b1b210518~tplv-k3u1fbpfcp-watermark.awebp)

第二步：开发者修改并将无法修改的备注

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/47ae47b4c0714f45a24ad85b1cb9351c~tplv-k3u1fbpfcp-watermark.awebp)

第三步：开发修改完成后在关闭文档时，通知到发起人

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a63538d4623540ffa015ad2b339fad30~tplv-k3u1fbpfcp-watermark.awebp)

总结
--

使用这个方法不仅可以准确的记录修改的内容和进度，打包的形式发送也可以减少对方工作的打断。为开发者明确要修改的内容，为提出者了解目前修改的进度。当然与后端的对接接口时产生的问题，或者在开发过程中需要多次请求别人援助的情况下我们也可以这么做，来减少对方打断耗时和增加自己跟进的粘合度。