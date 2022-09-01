> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7124258386606030855)

基于 Webassembly + QuickJS 的 Web 安全沙箱技术方案，面向 Web 端建设下一代开放技术。

背景
==

Web 端侧的开放技术长期以来一直在寻找最好的解决方案，从早期基于 Webview + API 管控 的开放形式 ，到目前基于小程序的重容器的架构方案。或多或少都无法全面的解决开发者体验的问题，API 开放形式无法做到安全管控，小程序开放形式的架构必然会给业务带来孤岛效应。如何给开发者带来更好的研发体验、给商家带来更好的产品体验一直是我们淘宝开放技术前端团队的命题。

经过半年的技术演进和业务落地，我们自研了一套基于 Webassembly + QuickJS 的架构方案，来解决上述的问题。

目标
==

三年的小程序形式开放基础已经形成了规模化壁垒，落地新的技术方案必然考虑到成本问题，所以本次架构升级的目标可以描述为 “面向 Web 端建设下一代 PC 开放技术，建设基于 W3C 标准的 Web 技术三方开放方案，与小程序、小部件开放形态互补，构建电商域的完整开放技术生态”。

思考
==

端侧开放到底解决什么问题，我理解开放技术在端侧主要围绕这 2 个点：如何让外部代码安全的可控的执行、用户数据的安全如何保障，做到无端不透；

1.  关于第一个点落实到细节是 JavaScript/CSS/HTML 如何执行的问题。我的解法是：JavaScript 运行在 Webassembly+QuickJS 安全容器里，基于 Webassembly 的安全水位保障 JS 执行的隔离和可控；CSS 基于 Shadow DOM + iframe 做到样式隔离。
2.  数据安全主要依托浏览器容器环境对数据做加签验签加密操作，本文章不展开讲解；

技术细节
====

工欲其事必先利器，我们先对技术底层细节做一个了解。

**WebAssembly**
---------------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1b19ec01257c4aad9eefde9805f31ec9~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

图来源地址（https://medium.com/jspoint/the-anatomy-of-webassembly-writing-your-first-webassembly-module-using-c-c-d9ee18f7ac9b）

这里重点提到 2 个事情：

1.  WebAssembly 二进制码会经过 Liftoff 生成未优化的 ByteCode，再给到 TurboFan 优化代码编译为机器码
2.  WebAssembly 和 JavaScript 的编译后端是共享的，WebAssembly 的尽头是和架构相关的机器码，这里是一个关键点也是 VM 和 HOST 同步调用的关键点。

**WebContainer 架构**
-------------------

本质上我们会以 App 级别的思想来架构方案（多页面、路由、通信），底层运行时基于 QuickJS ，涉及到多页面管理、鉴权、内存分析等等。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9bd42c436de54f18a03a6d5033d08c01~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

**Binding 细节**
--------------

外部代码的 Runtime 会运行到 Quickjs 里，执行环境需要仿真和浏览器环境一致，这个时候涉及到 API Binding 设计，在 HOST JavaScript Runtime 定义 W3C 规范，通过 WebAssembly 的内存 Binding 到 Quickjs Runtime 中，当外部代码调用对应方法时，会映射到 HOST 实现，通过安全管控策略再执行到 HOST 环境中。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6bf2eb09773149e7acf60ebff4e75ec9~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

**研发模式**
--------

WebContainer 的本质要解决开发者体验问题，做到上层框架无关，所以在安全的水位上我们不会约束今天开发者的框架体系。但是我们需要定义 App Export 的接口。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9c78b7cf20724f19b2e75fd96f28e1dd~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

**Benchmark**
-------------

相对于纯小程序的通信效率提升 355 倍，但是受到安全容器的限制 JS 执行相对于底层 V8 引擎降低到 1%，但是也是足够用于生产环境的。

业务落地
====

在商家私域场景，我们已经在旺铺装修表单落地，目前 ISV 反馈非常不错，业务平台未来会全量替换到新开放容器中。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cbdff05f2914427ca585a8d902fd1db4~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

未来
==

继续基于 WebContainer 能力做上层开放体系的建设，解决插件体系。解决启动耗时的问题。技术底层完善 Quickjs Debug 能力。