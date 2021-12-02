> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7030585901524713508)*   localStorage 存储的键值采用什么字符编码
*   5M 的单位是什么
*   localStorage 键占不占存储空间
*   localStorage 的键的数量，对写和读性能的影响
*   写个方法统计一个 localStorage 已使用空间

我们挨个解答，之后给各位面试官又多了一个面试题。

**`我们常说localStorage存储空间是5M，请问这个5M的单位是什么？`**

localStorage 存储的键值采用什么字符编码？
---------------------------

打开相对权威的 MDN [localStorage#description](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FWeb%2FAPI%2FWindow%2FlocalStorage%23description "https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage#description")

> The keys and the values stored with `localStorage` are _always_ in the UTF-16 [`DOMString`](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FWeb%2FAPI%2FDOMString "https://developer.mozilla.org/en-US/docs/Web/API/DOMString") format, which uses two bytes per character. As with objects, integer keys are automatically converted to strings.

翻译成中文：

> localStorage 存储的键和值始终采用 UTF-16 DOMString 格式，每个字符使用两个字节。与对象一样，整数键将自动转换为字符串。

**答案： UTF-16**

MDN 这里描述的没有问题，也有问题，因为 UTF-16，每个字符使用两个字节，是有前提条件的，就是码点小于`0xFFFF`(65535)， 大于这个码点的是四个字节。

**`这是全文的关键。`**

5M 的单位是什么
---------

5M 的单位是什么？

选项：

1.  字符的个数
2.  字节数
3.  字符的长度值
4.  bit 数
5.  utf-16 编码单元

以前不知道，现代浏览器，准确的应该是 **选项 3，字符的长度** ，亦或 **选项 5, utf-16 编码单元**

字符的个数，并不等于字符的长度，这一点要知道：

```
"a".length // 1
"人".length // 1
"𠮷".length // 2
"🔴".length // 2
复制代码

```

现代浏览器对字符串的处理是基于 UTF-16 [`DOMString`](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FWeb%2FAPI%2FDOMString "https://developer.mozilla.org/en-US/docs/Web/API/DOMString")。

**但是说 5M 字符串的长度，显然有那么点怪异。**

而根据 UTF-16 编码规则，要么 2 个字节，要么四个字节，**`所以不如说是 10M 的字节数，更为合理。`**

**`当然，2个字节作为一个utf-16的字符编码单元，也可以说是 5M 的utf-16的编码单元。`**

我们先编写一个 utf-16 字符串计算字节数的方法：非常简单，判断码点决定是 2 还是 4

```
function sizeofUtf16Bytes(str) {
    var total = 0,
        charCode,
        i,
        len;
    for (i = 0, len = str.length; i < len; i++) {
        charCode = str.charCodeAt(i);
        if (charCode <= 0xffff) {
            total += 2;
        } else {
            total += 4;
        }
    }
    return total;
}
复制代码

```

我们再根绝 10M 的字节数来存储

我们留下 8 个字节数作为 key，8 个字节可是普通的 4 个字符换，也可是码点大于 65535 的 3 个字符，也可是是组合。

下面的三个组合，都是可以的，

1.  `aaaa`
2.  `aa🔴`
3.  `🔴🔴`

在此基础上增加任意一个字符，都会报错异常异常。

```
const charTxt = "人";
let count = (10 * 1024 * 1024 / 2) - 8 / 2;
let content = new Array(count).fill(charTxt).join("");
const key = "aa🔴";
localStorage.clear();
try {
    localStorage.setItem(key, content);
} catch (err) {
    console.log("err", err);
}

const sizeKey = sizeofUtf16Bytes(key);
const contentSize = sizeofUtf16Bytes(content);
console.log("key size:", sizeKey, content.length);
console.log("content size:", contentSize, content.length);
console.log("total size:", sizeKey + contentSize, content.length + key.length);
复制代码

```

现代浏览器的情况下：

所以，说是 10M 的字节数，更为准确，也更容易让人理解。

如果说 5M，那其单位就是字符串的长度，而不是字符数。

**答案： 字符串的长度值, 或者 utf-16 的编码单元**

**`更合理的答案是 10M字节空间。`**

localStorage 键占不占存储空间
---------------------

我们把 key 和 val 各自设置长 2.5M 的长度

```
const charTxt = "a";
let count = (2.5 * 1024 * 1024);
let content = new Array(count).fill(charTxt).join("");
const key = new Array(count).fill(charTxt).join("");
localStorage.clear();
try {
    console.time("setItem")
    localStorage.setItem(key, content);
    console.timeEnd("setItem")
} catch (err) {
    console.log("err code:", err.code);
    console.log("err message:", err.message)
}
复制代码

```

执行正常。

我们把 content 的长度加`1`， 变为 `2.5 M + 1`, key 的长度依旧是 `2.5M`的长度

```
const charTxt = "a";
let count = (2.5 * 1024 * 1024);
let content = new Array(count).fill(charTxt).join("") + 1;
const key = new Array(count).fill(charTxt).join("");
localStorage.clear();
try {
    console.time("setItem")
    localStorage.setItem(key, content);
    console.timeEnd("setItem")
} catch (err) {
    console.log("err code:", err.code);
    console.log("err message:", err.message)
}
复制代码

```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dff64d929e8e4cc9a65d5447184cd8eb~tplv-k3u1fbpfcp-watermark.awebp?)

产生异常，存储失败。 至于更多异常详情吗，参见 [localstorage_功能检测](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FWeb_Storage_API%2FUsing_the_Web_Storage_API%23localstorage_%25E5%258A%259F%25E8%2583%25BD%25E6%25A3%2580%25E6%25B5%258B "https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Storage_API/Using_the_Web_Storage_API#localstorage_%E5%8A%9F%E8%83%BD%E6%A3%80%E6%B5%8B")：

```
function storageAvailable(type) {
    var storage;
    try {
        storage = window[type];
        var x = '__storage_test__';
        storage.setItem(x, x);
        storage.removeItem(x);
        return true;
    }
    catch(e) {
        return e instanceof DOMException && (
            // everything except Firefox
            e.code === 22 ||
            // Firefox
            e.code === 1014 ||
            // test name field too, because code might not be present
            // everything except Firefox
            e.name === 'QuotaExceededError' ||
            // Firefox
            e.name === 'NS_ERROR_DOM_QUOTA_REACHED') &&
            // acknowledge QuotaExceededError only if there's something already stored
            (storage && storage.length !== 0);
    }
}
复制代码

```

**答案: 占空间**

键的数量，对读写的影响
-----------

我们`500 * 1000`键，如下

```
let keyCount = 500 * 1000;

localStorage.clear();
for (let i = 0; i < keyCount; i++) {
    localStorage.setItem(i, "");
}

setTimeout(() => {
    console.time("save_cost");
    localStorage.setItem("a", "1");
    console.timeEnd("save_cost");
}, 2000)


setTimeout(() => {
    console.time("read_cost");
    localStorage.getItem("a");
    console.timeEnd("read_cost");

}, 2000)

// save_cost: 0.05615234375 ms
// read_cost: 0.008056640625 ms
复制代码

```

你单独执行保存代码：

```
localStorage.clear();    
console.time("save_cost");
localStorage.setItem("a", "1");
console.timeEnd("save_cost");
// save_cost: 0.033203125 ms
复制代码

```

可以多次测试， 影响肯定是有的，也仅仅是数倍，不是特别的大。

反过来，如果是保存的值表较大呢？

```
const charTxt = "a";
const count = 5 * 1024 * 1024  - 1
const val1 = new Array(count).fill(charTxt).join("");

setTimeout(() =>{
    localStorage.clear();
    console.time("save_cost_1");
    localStorage.setItem("a", val1);
    console.timeEnd("save_cost_1");
},1000)


setTimeout(() =>{
    localStorage.clear();
    console.time("save_cost_2");
    localStorage.setItem("a", "a");
    console.timeEnd("save_cost_2");
},1000)

// save_cost_1: 12.276123046875 ms
// save_cost_2: 0.010009765625 ms
复制代码

```

可以多测试很多次，单次值的大小对存的性能影响非常大，读取也一样，合情合理之中。

所以尽量不要保存大的值，因为其是同步读取，纯大数据，用 indexedDB 就好。

**答案：键的数量对读取性能有影响，但是不大。值的大小对性能影响更大，不建议保存大的数据。**

写个方法统计一个 localStorage 已使用空间
---------------------------

现代浏览器的精写版本：

```
function sieOfLS() {
    return Object.entries(localStorage).map(v => v.join('')).join('').length;
}
复制代码

```

测试代码：

```
localStorage.clear();
localStorage.setItem("🔴", 1);
localStorage.setItem("🔴🔴🔴🔴🔴🔴🔴🔴", 1111);
console.log("size:", sieOfLS())   // 23
// 🔴*9 + 1 *5 = 2*9 + 1*5 = 23
复制代码

```

html 的协议标准
----------

WHATWG 超文本应用程序技术工作组 的 [localstorage](https://link.juejin.cn?target=https%3A%2F%2Fhtml.spec.whatwg.org%2Fmultipage%2Fwebstorage.html%23dom-localstorage-dev "https://html.spec.whatwg.org/multipage/webstorage.html#dom-localstorage-dev") 协议定了 localStorage 的方法，属性等等，并没有明确规定其存储空间。也就导致各个浏览器的最大限制不一样。

其并不是 ES 的标准。

页面的 utf-8 编码
------------

我们的 html 页面，经常会出现 `<meta charset="UTF-8">`。 告知浏览器此页面属于什么字符编码格式，下一步浏览器做好解码工作。

```
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta >
    <title>容器</title>
</head>
复制代码

```

这和 localStorage 的存储没有半毛钱的关系。

localStorage 扩容
---------------

localStorage 的空间是 10M 的字节数，一般情况是够用，可是人总是有贪欲。 真达到了空间限制，怎么弄？

localStorage 扩容就是一个话题。

写在最后
----

不忘初衷，有所得，而不为所累，如果你觉得不错，你的一赞一评就是我前行的最大动力。

** 技术交流群请到 [这里来](https://juejin.cn/pin/6994350401550024741 "https://juejin.cn/pin/6994350401550024741")。 或者添加我的微信 dirge-cloud，一起学习。

引用
--

[localStorage](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FWeb%2FAPI%2FWindow%2FlocalStorage "https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage")