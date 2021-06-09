> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651577309&idx=1&sn=be4edf660920eab626043f40b6c3ebf5&chksm=80250c1cb752850a8b878131e9c7559f61031bed0a588108525879ae6cb703fd5f2f0fe4561c&mpshare=1&scene=1&srcid=0609WhzkkgAf1JSHdBhPPutb&sharer_sharetime=1623220882000&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

↓推荐关注↓

公众号

JavaScript 可以做很多神奇的事情！

从复杂的框架到处理 API，有太多的东西需要学习。

但是，它也能让你只用一行代码就能做一些了不起的事情。

看看这 13 句 JavaScript 单行代码，会让你看起来像个专家!

1. 获取一个随机布尔值 (true/false)
-------------------------

这个函数使用 `Math.random()` 方法返回一个布尔值（true 或 false）。`Math.random` 将在 0 和 1 之间创建一个随机数，之后我们检查它是否高于或低于 0.5。这意味着得到真或假的几率是 50%/50%。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tibUxowsg9P1OlLy24Rv35V2KA8SNB4X4zpABLaiahV9AHVLnSzS7QQhxOb1iaeeooMQmzeePCG9ytmf8RVUBrT7A/640?wx_fmt=jpeg)

```
const randomBoolean = () => Math.random() >= 0.5;
console.log(randomBoolean());
// Result: a 50/50 change on returning true of false


```

2. 检查日期是否为工作日
-------------

使用这个方法，你就可以检查函数参数是工作日还是周末。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tibUxowsg9P1OlLy24Rv35V2KA8SNB4X4aibZpDBHbUTgOLxWhVGhfEicM7jTKFgcPJ3F6nHbOqTQnG0pOsyn3VHA/640?wx_fmt=jpeg)

```
const isWeekday = (date) => date.getDay() % 6 !== 0;
console.log(isWeekday(new Date(2021, 0, 11)));
// Result: true (Monday)
console.log(isWeekday(new Date(2021, 0, 10)));
// Result: false (Sunday)


```

3. 反转字符串
--------

有几种不同的方法来反转一个字符串。以下代码是最简单的方式之一。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tibUxowsg9P1OlLy24Rv35V2KA8SNB4X4zHtD1Qj13OB5MkGvoOo5LuXSdacvYTXNGtYzU7LUhA7ysVrPJppfPA/640?wx_fmt=jpeg)

```
const reverse = str => str.split('').reverse().join('');
reverse('hello world');     
// Result: 'dlrow olleh'


```

4. 检查当前 Tab 页是否在前台
------------------

我们可以通过使用 `document.hidden`属性来检查当前标签页是否在前台中。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tibUxowsg9P1OlLy24Rv35V2KA8SNB4X4lic0ENSRkbSlmeyUhggpc8RogeJibslVPHBUD4asdicN4sXjILMa3b1Cg/640?wx_fmt=jpeg)

```
const isBrowserTabInView = () => document.hidden;
isBrowserTabInView();
// Result: returns true or false depending on if tab is in view / focus


```

5. 检查数字是否为奇数
------------

最简单的方式是通过使用模数运算符（%）来解决。如果你对它不太熟悉，这里是 Stack Overflow 上的一个很好的图解。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tibUxowsg9P1OlLy24Rv35V2KA8SNB4X4G8OF3cChNArtFbgWfu7pXEHmic2t816wDOXNUUic2MMEuIhHkGWebJOA/640?wx_fmt=jpeg)

```
const isEven = num => num % 2 === 0;
console.log(isEven(2));
// Result: true
console.log(isEven(3));
// Result: false


```

6. 从日期中获取时间
-----------

通过使用 `toTimeString()` 方法，在正确的位置对字符串进行切片，我们可以从提供的日期中获取时间或者当前时间。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tibUxowsg9P1OlLy24Rv35V2KA8SNB4X4dSPpXc0zfbIJ1ohJjAORtPicYt4bhicg5GSN77Kqf9ITeWcickA9ib3HJg/640?wx_fmt=jpeg)

```
const timeFromDate = date => date.toTimeString().slice(0, 8);
console.log(timeFromDate(new Date(2021, 0, 10, 17, 30, 0))); 
// Result: "17:30:00"
console.log(timeFromDate(new Date()));
// Result: will log the current time


```

7. 保留小数点（非四舍五入）
---------------

使用 `Math.pow()` 方法，我们可以将一个数字截断到某个小数点。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tibUxowsg9P1OlLy24Rv35V2KA8SNB4X4713VkEZggXgXjHPNEHibA17DAIJQwUuLnAia1UBB9fCoaDcSfNKon3Ug/640?wx_fmt=jpeg)

```
const toFixed = (n, fixed) => ~~(Math.pow(10, fixed) * n) / Math.pow(10, fixed);
// Examples
toFixed(25.198726354, 1);       // 25.1
toFixed(25.198726354, 2);       // 25.19
toFixed(25.198726354, 3);       // 25.198
toFixed(25.198726354, 4);       // 25.1987
toFixed(25.198726354, 5);       // 25.19872
toFixed(25.198726354, 6);       // 25.198726


```

8. 检查元素当前是否为聚焦状态
----------------

我们可以使用 `document.activeElement` 属性检查一个元素当前是否处于聚焦状态。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tibUxowsg9P1OlLy24Rv35V2KA8SNB4X4ea3xkohOZzjD3vBdet67YicQpSKWJ98ZmL18yAKj9lFPVjhgKQ5uVxw/640?wx_fmt=jpeg)

```
const elementIsInFocus = (el) => (el === document.activeElement);
elementIsInFocus(anyElement)
// Result: will return true if in focus, false if not in focus


```

9. 检查浏览器是否支持触摸事件
----------------

```
const touchSupported = () => {
  ('ontouchstart' in window || window.DocumentTouch && document instanceof window.DocumentTouch);
}
console.log(touchSupported());
// Result: will return true if touch events are supported, false if not


```

10. 检查当前用户是否为苹果设备
-----------------

我们可以使用 `navigator.platform`来检查当前用户是否为苹果设备。

```
const isAppleDevice = /Mac|iPod|iPhone|iPad/.test(navigator.platform);
console.log(isAppleDevice);
// Result: will return true if user is on an Apple device


```

11. 滚动到页面顶部
-----------

`window.scrollTo()` 方法会取一个 x 和 y 坐标来进行滚动。如果我们将这些坐标设置为零，就可以滚动到页面的顶部。

**注意：IE 不支持 `scrollTo()` 方法。**

![](https://mmbiz.qpic.cn/mmbiz_jpg/tibUxowsg9P1OlLy24Rv35V2KA8SNB4X4LIlz4nDJIkMPOv0Ija9ibRzlMcib4om3dqgkliaXVpAicsnxQjL6WzSPmA/640?wx_fmt=jpeg)

```
const goToTop = () => window.scrollTo(0, 0);
goToTop();
// Result: will scroll the browser to the top of the page


```

12. 获取所有参数平均值
-------------

我们可以使用 `reduce` 方法来获得函数参数的平均值。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tibUxowsg9P1OlLy24Rv35V2KA8SNB4X46mlKBTSpkHuDmuaiavKf65g8uE8BmeQYmBSlW7Omb4p9bP4JKVE3lWw/640?wx_fmt=jpeg)

```
const average = (...args) => args.reduce((a, b) => a + b) / args.length;
average(1, 2, 3, 4);
// Result: 2.5


```

13. 转换华氏度 / 摄氏度。（这个应该很少在国内用到吧）
------------------------------

处理温度有时会让人感到困惑。这 2 个功能将帮助你将华氏温度转换为摄氏温度，反之亦然。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tibUxowsg9P1OlLy24Rv35V2KA8SNB4X4oYJyUa1kgogGEhp8YAhZic4EXlTF5orK0eG6pNtLiaiauGGAdDZiaaCfrA/640?wx_fmt=jpeg)

```
const celsiusToFahrenheit = (celsius) => celsius * 9/5 + 32;
const fahrenheitToCelsius = (fahrenheit) => (fahrenheit - 32) * 5/9;
// Examples
celsiusToFahrenheit(15);    // 59
celsiusToFahrenheit(0);     // 32
celsiusToFahrenheit(-20);   // -4
fahrenheitToCelsius(59);    // 15
fahrenheitToCelsius(32);    // 0


```

> 文章为翻译，老外也很会写标题，标题可能有 XX 党嫌疑，但是部分内容还是挺有用的。
> 
> 原文链接：https://medium.com/dailyjs/13-javascript-one-liners-thatll-make-you-look-like-a-pro-29a27b6f51cb

> 译者：掘金 - yck
> 
> https://juejin.cn/post/6921509748785283086

- EOF -

推荐阅读  点击标题可跳转

1、[用 three.js 写一个下雨动画](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651574231&idx=2&sn=cbf8d400cec5b16d52855b9dfd85b1ed&chksm=80251816b7529100cc472f75d44f98ec8d6ea59cd97825a0bf09890c920556010237651fce9d&scene=21#wechat_redirect)

2、[JS 语法 ES6、ES7、ES8、ES9、ES10、ES11、ES12 新特性](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651573707&idx=1&sn=2601b3e36c2476bfcfd842585764a15b&chksm=80251a0ab752931c485c432463632f94d0cc52fe3f9debea91fa25f57861159575a60d7ccff6&scene=21#wechat_redirect)

3、[关于从入门 three.js 到做出 3d 地球这件事 (通俗易懂的入门)](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651574123&idx=2&sn=52a2852d0105d69e69b3027f90f34b42&chksm=802518aab75291bc237a2b0ea3dfa3e2d36def01fc200ceedc65f79c63f50622b6bced485f74&scene=21#wechat_redirect)

觉得本文对你有帮助？请分享给更多人

推荐关注「前端大全」，提升前端技能

公众号

点赞和在看就是最大的支持❤️