> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6997930259953745950)

作者：Shadeed 译者：前端小智 来源：medium

> 有梦想，有干货，微信搜索 **【大迁世界】** 关注这个在凌晨还在刷碗的刷碗智。
> 
> 本文 GitHub [github.com/qq449245884…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fqq449245884%2Fxiaozhi "https://github.com/qq449245884/xiaozhi") 已收录，有一线大厂面试完整考点、资料以及我的系列文章。

JavaScript 可以做很多好玩的事， 从复杂的框架到处理 API，有太多的东西需要学习。但是，它也能让我们只用一行就能做一些了不起的事情。

### 1. 获得一个随机的布尔值（`true`/`false`）

该函数使用`Math.random()`方法返回一个布尔值（`true` 或者 `false`）。`Math.random`创建一个`0`到`1`之间的随机数，我们只要检查它是否高于或低于`0.5`，就有 50% 机会得到`true`或`false`。

```
const randomBoolean = () => Math.random() >= 0.5;
console.log(randomBoolean());
复制代码
```

### 2. 检查所提供的日期是否为工作日

使用这种方法，我们能够检查在函数中提供的日期是否是工作日或周末的日子。

```
const isWeekday = (date) => date.getDay() % 6 !== 0;

console.log(isWeekday(new Date(2021, 7, 6)));
// true  因为是周五

console.log(isWeekday(new Date(2021, 7, 7)));
// false 因为是周六
复制代码
```

### 3. 反转字符串

有几种不同的方法来反转一个字符串。这是最简单的一种，使用`split()`、`reverse()`和`join()`方法。

```
const reverse = str => str.split('').reverse().join('');
reverse('hello world');     
// 'dlrow olleh'
复制代码
```

### 4. 检查当前标签是否隐藏

`Document.hidden` （只读属性）返回布尔值，表示页面是（`true`）否（`false`）隐藏。

```
const isBrowserTabInView = () => document.hidden;
isBrowserTabInView();
复制代码
```

场外：无意间发现爱奇艺广告播放时间居然是在当前标签页激活的时候才会进行倒计时，离开当前标签页的时候，倒计时停止，百度一下发现`document.hidden`这个东东。

`document.hidden`是`h5`新增加`api`使用的时候有兼容性问题。

```
var hidden
if (typeof document.hidden !== "undefined") {
    hidden = "hidden";
} else if (typeof document.mozHidden !== "undefined") {
    hidden = "mozHidden";
} else if (typeof document.msHidden !== "undefined") {
    hidden = "msHidden";
} else if (typeof document.webkitHidden !== "undefined") {
    hidden = "webkitHidden";
}
console.log("当前页面是否被隐藏：" + document[hidden])
复制代码
```

### 5. 检查一个数字是偶数还是奇数

```
const isEven = num => num % 2 === 0;
console.log(isEven(2));
// true
console.log(isEven(3));
// false
复制代码
```

### 6. 从一个日期获取时间

```
const timeFromDate = date => date.toTimeString().slice(0, 8);

console.log(timeFromDate(new Date(2021, 0, 10, 17, 30, 0))); 
// "17:30:00"

console.log(timeFromDate(new Date()));
// 打印当前的时间
复制代码
```

### 7. 保留 n 位小数

```
const toFixed = (n, fixed) => ~~(Math.pow(10, fixed) * n) / Math.pow(10, fixed);
// 事例
toFixed(25.198726354, 1);       // 25.1
toFixed(25.198726354, 2);       // 25.19
toFixed(25.198726354, 3);       // 25.198
toFixed(25.198726354, 4);       // 25.1987
toFixed(25.198726354, 5);       // 25.19872
toFixed(25.198726354, 6);       // 25.198726
复制代码
```

### 8. 检查当前是否有元素处于焦点中

我们可以使用`document.activeElement`属性检查一个元素是否当前处于焦点。

```
const elementIsInFocus = (el) => (el === document.activeElement);
elementIsInFocus(anyElement)
// 如果在焦点中返回true，如果不在焦点中返回 false
复制代码
```

### 9. 检查当前浏览器是否支持触摸事件

```
const touchSupported = () => {
  ('ontouchstart' in window || window.DocumentTouch && document instanceof window.DocumentTouch);
}
console.log(touchSupported());
// 如果支持触摸事件，将返回true，如果不支持则返回false。
复制代码
```

### 10. 检查当前浏览器是否在苹果设备上

```
const isAppleDevice = /Mac|iPod|iPhone|iPad/.test(navigator.platform);
console.log(isAppleDevice);
复制代码
```

### 11. 滚动到页面顶部

```
const goToTop = () => window.scrollTo(0, 0);
goToTop();
复制代码
```

### 12. 获取参数的平均数值

```
const average = (...args) => args.reduce((a, b) => a + b) / args.length;
average(1, 2, 3, 4);
// 2.5
复制代码
```

### 13. 华氏 / 摄氏转换

```
const celsiusToFahrenheit = (celsius) => celsius * 9/5 + 32;
const fahrenheitToCelsius = (fahrenheit) => (fahrenheit - 32) * 5/9;
// 事例
celsiusToFahrenheit(15);    // 59
celsiusToFahrenheit(0);     // 32
celsiusToFahrenheit(-20);   // -4
fahrenheitToCelsius(59);    // 15
fahrenheitToCelsius(32);    // 0
复制代码
```

~ 完，我是刷碗智，会所按摩走起！

原文：[medium.com/dailyjs/13-…](https://link.juejin.cn?target=https%3A%2F%2Fmedium.com%2Fdailyjs%2F13-javascript-one-liners-thatll-make-you-look-like-a-pro-29a27b6f51cb "https://medium.com/dailyjs/13-javascript-one-liners-thatll-make-you-look-like-a-pro-29a27b6f51cb")

交流
--

> 有梦想，有干货，微信搜索 **【大迁世界】** 关注这个在凌晨还在刷碗的刷碗智。
> 
> 本文 GitHub [github.com/qq449245884…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fqq449245884%2Fxiaozhi "https://github.com/qq449245884/xiaozhi") 已收录，有一线大厂面试完整考点、资料以及我的系列文章。