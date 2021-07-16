> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [ykloveyxk.github.io](https://ykloveyxk.github.io/2017/03/21/vue-login-%E9%80%9A%E8%BF%87%E4%B8%80%E4%B8%AA%E9%A1%B9%E7%9B%AE%E5%B8%A6%E4%BD%A0%E8%B5%B0%E8%BF%9Bvue%E5%85%A8%E6%A0%88%E5%BC%80%E5%8F%91/)

> 这是一个基于 vue & axios & nodejs(express) & mongodb(mongoose) 的登录／注册 demo，面向 vue 初学者，场景虽简单，但五脏俱全。

发表于 2017-03-21 | [](https://ykloveyxk.github.io/2017/03/21/vue-login-%E9%80%9A%E8%BF%87%E4%B8%80%E4%B8%AA%E9%A1%B9%E7%9B%AE%E5%B8%A6%E4%BD%A0%E8%B5%B0%E8%BF%9Bvue%E5%85%A8%E6%A0%88%E5%BC%80%E5%8F%91/#comments)| 阅读次数

> 这是一个基于 vue & axios & nodejs(express) & mongodb(mongoose) 的登录／注册 demo，面向 vue 初学者，场景虽简单，但五脏俱全。有前后台，涵盖非常多的 vue 及其相关技术的基本操作。有详细的注释，帮助大家快速上手 vue 。且我整理了一些在 vue 全栈开发过程中，有可能会用到的技术文章，希望大家能在这些前辈们身上有所收获。
> 
> 当然如果您觉得这篇文章 or 这个项目对您的学习有所帮助，请不吝点个 star 鼓励一下，当然如果存在问题，也非常希望您能提交 issues 或者在 [我的博客](https://ykloveyxk.github.io/)任意文章下留言，我会及时处理回复，和大家一起进步。

[](#项目Github地址 "项目Github地址")项目 Github 地址
----------------------------------------

* * *

*   [vue-login](https://github.com/ykloveyxk/vue-login)

[](#项目技术栈 "项目技术栈")项目技术栈
-----------------------

* * *

*   前台：vue & vue-router & vuex & vue-cli(webpack) & element-ui
*   后台： nodejs (express)
*   前后台交互： axios
*   单点登录： jsonwebtoken

[](#Build-Setup "Build Setup")Build Setup
-----------------------------------------

* * *

```
# install dependencies

npm install

# serve with hot reload at localhost:8080

npm run dev

# build for production with minification

npm run build

# build for production and view the bundle analyzer report

npm run build --report

# start server

node server.js

# start mongodb

mongod
```

[](#项目开发推荐阅读 "项目开发推荐阅读")项目开发推荐阅读
--------------------------------

* * *

> 因为项目难度并不是很高，且我也在代码中写了较多注释，所以我不会细节到行去解释代码，而是会搜集、分享一些前辈们书写的相关技术文章，相信大家踩在巨人的肩膀上，能学到更多。

### [](#1-vue-cli-生成项目主体框架 "1. vue-cli 生成项目主体框架")1. vue-cli 生成项目主体框架

使用 vue-cli 的优点是方便快捷，能快速生成项目的主体结构。但不能一味依赖这种开发方式，还是要了解其中的技术细节。此处推荐几篇文章：

*   [vue-cli 官方文档](https://github.com/vuejs/vue-cli)
*   [webpack2 中文文档](https://doc.webpack-china.org/)
*   [vue-cli#2.0 webpack 配置分析 - 掘金](https://juejin.im/post/584e48b2ac502e006c74a120)（强烈推荐）

### [](#2-vue-全家桶 "2. vue 全家桶")2. vue 全家桶

顾名思义就是我们熟知的 vue + vue-router + vuex + … ，  
虽然还有很多的组件，但是基础都是 前三个。这块儿首推官方文档，我个人认为 vue 的成功除开自身素质过硬外，最大的优势就是文档写的非常的浅显易懂！所以学 vue 一定要多读官方文档。此处放出连接：

*   [Vue.js](https://cn.vuejs.org/)
*   [Vue-router](https://router.vuejs.org/zh-cn/)
*   [Vuex](https://vuex.vuejs.org/zh-cn/)

当然掘金上也有许多的详解文章，也推荐大家去看看。

### [](#3-后台服务端 "3. 后台服务端")3. 后台服务端

后台主要作用是接收前台请求，处理完成后返回一个含有所需数据或状态的 api 接口，供前台去调用。这需要你了解熟悉 nodejs 或任意一种后端语言，以 nodejs 为例，有以下文章推荐你去阅读：

*   [阮一峰老师的 js 教程（含 node）](http://javascript.ruanyifeng.com/) （强烈建议把 js 部分也看看）
*   [Express 4.x API - 作业部落 Cmd Markdown 编辑阅读器](https://www.zybuluo.com/XiangZhou/note/208532)（express 4.x 的中文文档）
*   [koa](http://koa.bootcss.com/) （最近常出现的一个 node 框架，有兴趣的可以去了解一下 ）

### [](#4-axios前后台交互 "4. axios前后台交互")4. axios 前后台交互

vue 和 node 的交互还是主要采用 ajax 来进行，此处就介绍一个主流交互工具 axios，当然别的工具例如 vue-resource、jquery 都可以。但是 vue-resource 不维护了，jquery 如果只是为了 ajax 就引入又太庞大，所以我个人是比较推荐 axios。此处久推荐这几篇文章吧：

*   [axios 全攻略](https://juejin.im/entry/58b2532f2f301e006c0a2d80/detail) （我写的，羞射😳，但我个人觉得很值得阅读）
*   [Vuex2 和 Axios 的开发 | Hope’s Blog](https://blog.ygxdxx.com/2017/02/01/Vuex2&Axios-Develop/) （也是掘金作者，让理论照进现实）

### [](#5-jsonwebtoken "5. jsonwebtoken")5. jsonwebtoken

此项目使用 jsonwebtoken 进行用户认证，其实 jsonwebtoken 也可以用来做权限控制或者向 Web 应用传递信息。关于 jsonwebtoken 除了它的官方文档外，还有这几篇文章可以看看：

*   [JSON Web Token - 在 Web 应用间安全地传递信息](http://blog.leapoahead.com/2015/09/06/understanding-jwt/)
*   [八幅漫画理解使用 JSON Web Token](http://www.tuicool.com/articles/uuAzAbU)

### [](#6-组件库 "6. 组件库")6. 组件库

随着 vue 的不断发展，社区越来越活跃，因此产生了许多组件库，此处我就推荐一个我个人使用的最多的由饿了么团队开发的组件库 element-ui。

*   [element-ui](http://element.eleme.io/#/zh-CN)

### [](#7-demo "7. demo")7. demo

开发其实除了冥思苦想外，很多时候要多读别人的源码，从中才能有所启发。放出几个 demo：

*   [一个使用 github api 写的登录程序](https://github.com/superman66/vue-axios-github) （掘金作者，我的这篇文章也是受他启发）
*   [vue + koa + mysql 的 todos demo](https://molunerfinn.com/Vue+Koa/) （想了解 koa + mysql 如何在 vue 中运用可以看看）

[](#后记 "后记")后记
--------------

当然纸上学来终觉浅，绝知此事要躬行。学完理论就需要去实践，所以希望大家多多去看看别人的代码，然后写写小 demo。一定会事半功倍。希望和大家共同进步。

然后我无意发现有人未经过我同意就转发我的文章。而且未出现任何我的相关信息。我的观点是我写文章是为了方便大家，督促自己，所以转就转了，不需要我的同意，但是哪怕不出现我的名字，也请务必注一个 **转** 字，不要把他当作自己的文章来用。