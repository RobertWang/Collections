> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6844903725908099079) 今天看到一篇很有意思的一篇文章，如何打印多彩的 console.log? 前端的小伙伴对 console.log 再熟悉不过了，但是至今为止，我都是一直在用其最普通的用法，控制台中打印一条 message。没想到，还能给 console.log 应用样式呢？不知道你们是否知道呢？

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/11/27/167557f4f44d11b3~tplv-t2oaga2asx-watermark.awebp)

```
console.log('%cHello', 'color: green; background: yellow: font-size: 30px');
复制代码
```

可以看出，上面的 log 语句由三部分组成： **%c** + message + **style** 其中标识符后紧跟 message, 第二个参数为样式 最后输出的 message 的效果就如样式所定义的一致。

优点
==

当遇到一个具有大量 log 输出的大型应用时，如果在一些比较重要的地方输出带样式的 log 时，你可以在控制台中快速发现它，不至于淹没在一堆的 log 中难以发现查找。

栗子 2
====

```
console.log(
  'Nothing here %cHi Cat %cHey Bear',  // Console Message
  'color: blue', 'color: red' // CSS Style
);
复制代码
```

效果如下图，标识符前面的文本不受影响。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/11/27/167558a807a924d7~tplv-t2oaga2asx-watermark.awebp)

五种类型的 console message 都可以添加样式
=============================

*   console.log
*   [console.info](https://link.juejin.cn?target=http%3A%2F%2Fconsole.info "http://console.info")
*   console.debug
*   console.warn
*   console.error

```
console.log('%cconsole.log', 'color: green;');
console.info('%cconsole.info', 'color: green;');
console.debug('%cconsole.debug', 'color: green;');
console.warn('%cconsole.warn', 'color: green;');
console.error('%cconsole.error', 'color: green;');
复制代码
```

优雅的传递样式之实践
==========

```
// 1. 将css样式内容放入数组
const styles = [
  'color: green', 
  'background: yellow', 
  'font-size: 30px',
  'border: 1px solid red',
  'text-shadow: 2px 2px black',
  'padding: 10px',
].join(';'); 
// 2. 利用join方法讲各项以分号连接成一串字符串
// 3. 传入styles变量
console.log('%cHello There', styles);
复制代码
```

甚至，你还能把需要输出的 message 也抽离出来，保存在变量中

```
const styles = ['color: green', 'background: yellow'].join(';');
const message = 'Some Important Message Here';
// 3. 传入styles和message变量
console.log('%c%s', styles, message);
复制代码
```

参考文章 [medium.com/@samanthami…](https://link.juejin.cn?target=https%3A%2F%2Fmedium.com%2F%40samanthaming%2Fcolorful-console-message-2e8203786838 "https://medium.com/@samanthaming/colorful-console-message-2e8203786838")

喜欢的点赞支持下哦！