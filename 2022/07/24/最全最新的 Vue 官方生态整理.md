> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA5NjUxMTM2MQ==&mid=2247492874&idx=1&sn=8b54a17d98267bc139e9cf2c641617c5&chksm=90ac51e4a7dbd8f263041e7481a45d33e6fa34398336f7d5cd49cad7b55adcadf2490c14cc90&mpshare=1&scene=1&srcid=0724gSG8NMkD70I78AznO46q&sharer_sharetime=1658631815782&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

> 今天，咱们来整理下最新的 Vue 相关的「官方」生态都有哪些，考察你是不是都听过或用过？

按照尤雨溪之前的官宣，Vue 3 已经在 **「2022 年 2 月 7 日」**成为新的默认版本

> 官宣全文：https://gist.github.com/yyx990803/bf9a625eeff8b471bf0701afb8e3fe75

其中有一部分关于仓库的迁移：**「`vuejs`」** **「组织下的所有 GitHub 仓库将把默认分支切换到」** **「Vue」** **「3 对应的版本。」** 此外，还伴随着 Vue 周边生态仓库的重命名。

今天，咱们来整理下最新的 Vue 相关的**「官方」**生态都有哪些，考察你是不是都听过或用过？

首先，是众所周知的 Vue 全家桶：Vue 框架 + 路由 + 状态管理。

Vue
---

### vuejs/vue[1]

Vue2 的代码仓库

### vuejs/core[2]

Vue3 的代码仓库

路由 Router
---------

### vuejs/vue-router[3]

Vue2 的官方前端路由解决方案

### vuejs/router[4]

Vue3 的官方前端路由解决方案

状态管理 State Management
---------------------

### Vuex[5]

Vue 官方的状态管理器，分别在 v3 和 v4 两个大版本支持 Vue2 和 Vue3。

### Pinia[6]

又一个官方的 Vue 状态管理器，比 Vuex 更轻量，基于 Vue 的`组合式 API`（composition API），同时支持 Vue2 和 3，被认为是下一代的 Vuex。

* * *

由于 Vue3 引入了 Hook 的概念，这边也介绍下与 Hook 有关的：

Hook
----

### vue/composition-api[7]

用于提供`组合式 API`（composition API）的 Vue 2 插件。能在 Vue2 项目中使用 Vue3 的部分 API，帮助项目顺利迁移过渡到 Vue3。

### vueuse[8]

能够用于 Vue 2 和 Vue 3 项目的`组合式 API`（composition API）实用 utils 工具集合。有 useMouse、useScroll 等上百种 hook 函数。据说目前的 Vuer，`vueuse` 是除了 `lodash` 以外必装的工集。

### vue-demi[9]

Vue Demi （demi 在法语中有`半`的含义）是一个给开发 Vue 第三方包的开发者使用库，支持你开发出同时支持 Vue2 和 3 的通用 Vue 库。`vueuse`，`vuelidate` 和 `vue-echarts` 都使用了该工具。

* * *

接下来看看 Vue 项目工程化相关：

构建工具 Build Tool
---------------

### Vite[10]

Vite(法语单词，表示 “快速”，发音为 / vit/，就像 “veet”)，被称为下一代前端开发与构建工具，极大地改善了前端的开发体验。特点就是快!

脚手架 Scaffold
------------

### create-vue[11]

快速搭建基于 Vite 的 Vue 项目的官方脚手架，只需执行命令`npm init vue@3`。

### vue-cli[12]

快速搭建基于 webpack 的 Vue 项目的开发者工具，目前官方更**「推荐使用」** `create-vue`。

编辑器 Editor
----------

### Vetur[13]

官方的 VSCode Vue 工具扩展，包括语法高亮，代码片段（snippet），Lint 校验，代码格式化等特性。Vue 开发者必装。

### eslint-plugin-vue[14]

Vue 官方的 ESLint 插件。

### vue-eslint-parser[15]

自定义 ESLint 解析 `.vue` 单文件的解析器。

调试套件 Devtools
-------------

### Devtools[16]

用于调试 Vue.js 项目的浏览器 devtools 扩展插件。

测试 Test
-------

### Test Utils[17]

服务于 Vue 3 的组件测试工具集。

### vue-jest[18]

Vue 单文件组件的 Jest 转换器。

构建 Build
--------

### vue-loader[19]

是 webpack 的一个 loader，允许你用单文件组件 (sfc) 的格式创建 Vue 组件并打包。

### babel-plugin-jsx[20]

Vue 3 的 Babel JSX 插件。以 JSX 的方式来编写 Vue 代码。

### rollup-plugin-vue[21]

使用 rollup 构建 Vue 单文件的插件，已经不再维护，推荐使用 Vite 及 @vitejs/plugin-vue。

* * *

除了项目开发相关，还有一些其他的官方仓库：

站点生成 Static Site Generator
--------------------------

### Vuepress[22]

基于 Vue 的，极简的静态站点生成器。通常用于生成`个人博客`和`文档`站点。

### Vitepress[23]

基于 Vue 和 Vite 的静态站点生成器。是 VuePress 的 vite 版本。

优质内容收录 Collection
-----------------

### Awesome Vue[24]

收录了很多与 Vue 相关的一切优质内容。包括面试题，课程，开源项目，UI 库，第三方包，工具集，开发者工具等。

> 如有遗漏，欢迎评论留言补充。

### 

参考资料

  

_[1]

vuejs/vue:https://github.com/vuejs/vue

[2]

vuejs/core:https://github.com/vuejs/core

[3]

vuejs/vue-router:https://github.com/vuejs/vue-router

[4]

vuejs/router:https://github.com/vuejs/router

[5]

Vuex:https://github.com/vuejs/vuex

[6]

Pinia:https://github.com/vuejs/pinia

[7]

vue/composition-api:https://github.com/vuejs/composition-api

[8]

vueuse:https://github.com/vueuse/vueuse

[9]

vue-demi:https://github.com/vueuse/vue-demi

[10]

Vite:https://github.com/vitejs/vite

[11]

create-vue:https://github.com/vuejs/create-vue

[12]

vue-cli:https://github.com/vuejs/vue-cli

[13]

Vetur:https://github.com/vuejs/vetur

[14]

eslint-plugin-vue:https://github.com/vuejs/eslint-plugin-vue

[15]

vue-eslint-parser:https://github.com/vuejs/vue-eslint-parser

[16]

Devtools:https://github.com/vuejs/devtools

[17]

Test Utils:https://github.com/vuejs/test-utils

[18]

vue-jest:https://github.com/vuejs/vue-jest

[19]

vue-loader:https://github.com/vuejs/vue-loader

[20]

babel-plugin-jsx:https://github.com/vuejs/babel-plugin-jsx

[21]

rollup-plugin-vue:https://github.com/vuejs/rollup-plugin-vue

[22]

Vuepress:https://github.com/vuejs/vuepress

[23]

Vitepress:https://github.com/vuejs/vitepress

[24]

Awesome Vue:https://github.com/vuejs/awesome-vue

_

向下滑动查看

- EOF -

推荐阅读  点击标题可跳转

1、[Vue 2.7 正式发布，代号为 Naruto](http://mp.weixin.qq.com/s?__biz=MzA5NjUxMTM2MQ==&mid=2247492757&idx=1&sn=b8a9353e65fc588f16c74c7a72763974&chksm=90ac507ba7dbd96da8c43b27b193215a1c663ec7e701625c461c62a0a388aeefc730cb0952b1&scene=21#wechat_redirect)

2、[Vue 想要抛弃虚拟 DOM 了？！](http://mp.weixin.qq.com/s?__biz=MzA5NjUxMTM2MQ==&mid=2247492745&idx=1&sn=dd1501eb2004680b98c28e75be8b390f&chksm=90ac5067a7dbd971e11d801123babcec0f95ddcef3e7e233b79372b024c849bbb0a6c2632093&scene=21#wechat_redirect)

3、[基于 Vue3 和 TypeScript 项目大量实践后的思考](http://mp.weixin.qq.com/s?__biz=MzA5NjUxMTM2MQ==&mid=2247492718&idx=2&sn=8992a42b311fb472b8d6f4b7886de5ce&chksm=90ac5080a7dbd996164becd75fd0e5870415abe2b400709509b8c1db1e242ff1895baee57ce3&scene=21#wechat_redirect)

觉得本文对你有帮助？请分享给更多人

关注「大前端技术之路」加星标，提升前端技能

 ![](http://mmbiz.qpic.cn/mmbiz_png/mshqAkialV7FvgHwjwFbEbT0nNZU2AVPsmAuuZfXfp0Yc8H3FNkbiaKZYUJNichcJ3lZj4HiceOTXAeTnPHJrLFklA/0?wx_fmt=png) ** 大前端技术之路 ** 分享 Web 前端，Node.js、React Native 等大前端技术栈精选 19 篇原创内容  公众号

点赞和在看就是最大的支持❤️