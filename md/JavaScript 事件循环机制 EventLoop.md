> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7007449778615222303) 即事件循环，是指浏览器或 Node 的一种解决 javaScript 单线程运行时不会阻塞的一种机制，也就是我们经常使用异步的原理。 在 JavaScript 中，任务被分为两种，一种`宏任务`（MacroTask）也叫 Task，一种叫`微任务`（MicroTask）。

宏任务
---

**宏任务**包括 `script (整体代码)`、`setTimeout`、`setInterval`、 `setlmmediate`、 `l/O`、 `UI rendering`

微任务
---

**微任务**包括 `promise`、`Qbject observe`、`Mutation Observer` 微任务是跟屁虫，一直跟着`当前`宏任务后面代码执行到一个微任务就跟上，一个接着一个 ![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8505f4fdd813499092500793cd2a6fa1~tplv-k3u1fbpfcp-watermark.awebp?)

实例展示
----

我们来看以下代码

```
console.log('script start' );
 // 宏任务
setTimeout( function () {
  console.log('setTimeout' );
}, 0);
  // 微任务跟在当前宏在务屁股后面
Promise. resolve().then(function () {
  //微任务跟在当前宏任务配服石面
  console.log('promise1');
}).then(function () {
  // 微任务跟在当前宏任务屈股后面
  console.log('promise2');
});
console.log('script end');
复制代码
```

在以上代码中第一次洪任务为`script (整体代码)`，所以优先输出`script start`和`script end` 在本次`宏任务`中有两个`.then`的`微任务`，所以其次输出`promise1`和`promise2` 在第二次事件循环中我们遇到了宏任务`setTimeout` 所以再输出`setTimeout` 所以最终输出为

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/51df6db7ee924edd9b60fe05b772e624~tplv-k3u1fbpfcp-watermark.awebp?)

以下我画了张图方便大家更好理解

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c7233fa695054975999ec27740355d26~tplv-k3u1fbpfcp-watermark.awebp?)

小练习
---

以下给大家带来了两个小练习，大家可以手动敲下，深刻的理解其中的原理

### 题目 1

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/77c7c5f630064d5c89442064cee267b4~tplv-k3u1fbpfcp-watermark.awebp?)

输出为 3 4 8 5 2

### 题目 2

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7486080a34eb4b838a225f10a918912e~tplv-k3u1fbpfcp-watermark.awebp?)

输出为

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c263a864f4054fcd931bc3722cb60ec1~tplv-k3u1fbpfcp-watermark.awebp?)