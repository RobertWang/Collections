> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/351958447)

这是关于油猴脚本的第三篇文章，最近都在研究一些关于使用 JavaScript 来编写一些脚本从而完成一些简单但是繁琐的工作。

前景回顾：

*   [强大的油猴 Tampermonkey：简单的脚本制作](https://link.zhihu.com/?target=https%3A//www.cclliang.com/2020/05/28/%25E5%25BC%25BA%25E5%25A4%25A7%25E7%259A%2584%25E6%25B2%25B9%25E7%258C%25B4Tampermonkey%25EF%25BC%259A%25E7%25AE%2580%25E5%258D%2595%25E7%259A%2584%25E8%2584%259A%25E6%259C%25AC%25E5%2588%25B6%25E4%25BD%259C/)。
*   [制作油猴 Tampermonkey 脚本需要哪些知识](https://link.zhihu.com/?target=https%3A//www.cclliang.com/2020/07/27/%25E5%2588%25B6%25E4%25BD%259C%25E6%25B2%25B9%25E7%258C%25B4Tampermonkey%25E8%2584%259A%25E6%259C%25AC%25E9%259C%2580%25E8%25A6%2581%25E5%2593%25AA%25E4%25BA%259B%25E7%259F%25A5%25E8%25AF%2586/)。

**本篇文章适合对于 webpack 和 React 有一定了解的开发者，不适合新手开发者阅读，因为本文章主要讲解的是如何搭建开发环境。**

最近在学习 webpack 打包技术的时候，突然想到，是否能够使用`babel-loader`将更高级的语法转换为 ES5 语法，然后运行在油猴上面，我试了一下，发现是可行的。

虽然油猴自带 ES6 模板，但是它自带的 ES6 模板居然和它本身的语法检测冲突了？

咱也不知道是什么情况，于是就使用 webpack 自己搭建一个开发环境吧。

**为什么要使用 webpack 打包？**

*   因为 ES6 新增了非常多的新特性，弥补了 JavaScript 诞生以来的一些缺陷，比如 this 指针、回调地狱、没有 class 关键字等等。
*   它可以让 ES7 的 async，await 变得可用，甚至连最新的 ES11 语法都可以使用，也就是说你可以使用任何 JavaScript 的新特性。
*   它支持模块化编写，如果一个庞大的脚本几千行，将所有内容都写在一个文件中显然后期的维护也会变得非常麻烦，使用 webpack 打包编写就可以将这些功能分布到不同的文件中，从而使脚本维护变得简单。
*   可以随心所欲的引入其它第三方库，因为都是运行在浏览器上面，所以也不会像小程序那样有很多限制和兼容性问题。

**1. 开始**
---------

使用 webpack 将代码转换成 ES5 语法其实非常简单，只需要先安装一下包：

```
npm install webpack webpack-cli babel-loader @babel/core @babel/preset-env --save-dev

```

在`webpack.config.js`中进行配置处理 js 的 loader。

```
{
  test: /\.js$/,
  exclude: /node_modules/,
  use: [
    {
      loader: "babel-loader",
      options: {
        // 预设：指示babel做怎么样的兼容性处理。
        presets: [
          [
            "@babel/preset-env",
            {
              corejs: {
                version: 3,
              },
              // 按需加载
              useBuiltIns: "usage",
            },
          ],
        ],
      },
    },
  ],
},


```

配置完毕后再到`package.json`文件中添加浏览器版本适配信息：

```
"browserslist": {
  "production": [
    ">0.2%",
    "not dead",
    "not op_mini all"
  ],
  "development": [
    "last 1 chrome version",
    "last 1 firefox version",
    "last 1 safari version"
  ]
},

```

到这里，你可以随意的在你的 JavaScript 文件中使用最新的 js 语法，babel 会自动将它进行转换。

**2. 使用 React 语法**
------------------

当一个 JavaScript 脚本需要涉及到创建比较复杂的界面时，使用第三方框架来编写界面是非常不错的选择，比如：React，Vue。

这些框架编写界面远远比使用原生 JavaScript 创建界面或者使用 jQuery 创建界面要简单太多。

并且使用 React 还可以很方便的给 DOM 编写 CSS 样式。当然前提是你已经对 React 有一定的了解。

首先需要安装几个包：

```
npm install react react-dom
npm install @babel/preset-react -D

```

*   **react、react-dom**：这两个包是 React 相关包。
*   **@babel/preset-react**：转换 jsx 语法。

接着在 webpack 中进行配置：

```
{
  test: /\.(js|jsx|mjs)$/,
  exclude: /node_modules/,
  use: [
    {
      loader: "babel-loader",
      options: {
        // 预设：指示babel做怎么样的兼容性处理。
        presets: [
          [
            "@babel/preset-env",
            {
              corejs: {
                version: 3,
              },
              // 按需加载
              useBuiltIns: "usage",
            },
          ],
          "@babel/preset-react",
        ],
      },
    },
  ],
},


```

到这里就配置完毕，可以在脚本中使用 React 相关的语法。

入口文件：

```
import React from "react";
import App from "./components/App";
import $ from "jquery";
import { render } from "react-dom";

// 添加一个div作为React的入口文件
$("body").append(`<div/>`);

render(<App />, document.getElementById("ccll-app"));


```

**3. 编写脚本**
-----------

逐步摸索了几个小时，我发现不仅可以使用 **Bootstrap** 这类的 UI 库，甚至连 React 的 **Material-UI** 都能够使用，也就是说你**通过 webpack 打包来进行开发脚本的话，你可以往网页上面嵌入任意的内容，就和你平时开发 React 一样。**

光说不练假把式，这里就演示一下我练手的 Instagram 脚本，实现了点击图片查看原图、图片批量下载。

该脚本在页面上面新增两个按钮，分别是：

*   图片查看开关
*   图片批量下载

点击**图片批量下载**后，会弹出一个模态框，如下图所示，查看需要下载的图片，不过经过我的测试，最多貌似只能同时下载 12 张图片。

![](https://pic2.zhimg.com/v2-8f27d2d6b9ea0e23161c0bc180ac8bd9_r.jpg)

能够引入 UI 框架还有一个好处，你不用再去为样式而发愁，直接引用框架，随便写写感觉就很高大上。

使用 jQuery 来操作网页上已经存在的元素，使用 React 配合 UI 来搭建辅助界面，可以让你在当前网页上面随意发挥，看不惯哪儿还可以直接修改元素的样式、内容等。

**4. 最后**
---------

因为该文章是我先搭好了开发环境后又隔了一天，自己尝试制作完成一个脚本的情况下又跑回来完善了这篇文章。

你如果要用 React 或者 Vue 这种框架 webpack 环境已经帮你搭好了并且经过很多个版本的迭代和修复，几乎已经不需要你再去修改什么。

但是像上面这种油猴开发环境就仁者见仁智者见智，需要什么就添加什么，比如我需要使用 sass，我就增加一个 **sass-loader**，我需要使用 ts，我就增加一个 **ts-loader**，这些都是看自己心情。

在没有学习 webpack 之前，我是肯定不会想到 js 的脚本可以如此强大，学习了 webpack 后让我能够随心所欲的修改任意的网页（当然仅本地有效），不得不说 webpack 还是没有白学。

因为是编写脚本的关系，其实大多数情况下根本不需要去创建界面，所以仅使用 jQuery 就已经够用了，因为在脚本中引入 React 会导致脚本的体积变得异常的庞大，什么都不写打包出来的脚本都有 100 多 KB。

不过脚本是存在于本地然后运行，所以理论上脚本的大小其实没有太大的影响，实现功能就可以了，这一点和 Web 开发不大一样。

在 Web 开发中，如果一个文件太大，客户的网速又不好的情况下，会极大的影响用户体验，而脚本的大小就不被这些限制。