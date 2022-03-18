> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651588138&idx=3&sn=aede6605b77b5100fee432d9e075a458&chksm=8022d1ebb75558fdaf22346282754ce8996cefecb3f39a116bba7944ed2022f2a03dbdc3e39d&scene=21#wechat_redirect)

### 通用后台管理系统整体架构方案 (Vue)

#### 项目创建，脚手架的选择 (vite or vue-cli)

*   `vue-cli`基于`webpack`封装，生态非常强大，可配置性也非常高，几乎能够满足前端工程化的所有要求。缺点就是配置复杂，甚至有公司有专门的`webpack工程师`专门做配置，另外就是 webpack 由于开发环境需要打包编译，开发体验实际上不如`vite`。
    
*   `vite`开发模式基于`esbuild`，打包使用的是`rollup`。急速的`冷启动`和无缝的`hmr`在开发模式下获得极大的体验提升。缺点就是该脚手架刚起步，生态上还不及`webpack`。
    

本文主要讲解使用`vite`来作为脚手架开发。(动手能力强的小伙伴完全可以使用`vite`做开发服务器，使用`webpack`做打包编译放到生产环境)

为什么选择 vite 而不是 vue-cli，不论是`webpack`,`parcel`,`rollup`等工具，虽然都极大的提高了前端的开发体验，但是都有一个问题，就是当项目越来越大的时候，需要处理的`js`代码也呈指数级增长，打包过程通常需要很长时间 (甚至是几分钟!) 才能启动开发服务器，体验会随着项目越来越大而变得越来越差。

由于现代浏览器都已经原生支持 es 模块，我们只要使用支持 esm 的浏览器开发，那么是不是我们的代码就不需要打包了？是的，原理就是这么简单。vite 将源码模块的请求会根据`304 Not Modified`进行协商缓存，依赖模块通过`Cache-Control:max-age=31536000,immutable`进行协商缓存，因此一旦被缓存它们将不需要再次请求。

> 软件巨头微软周三 (5 月 19 日) 表示，从 2022 年 6 月 15 日起，公司某些版本的 Windows 软件将不再支持当前版本的 IE 11 桌面应用程序。`所以利用浏览器的最新特性来开发项目是趋势。`

```
$ npm init @vitejs/app <project-name>
$ cd <project-name>
$ npm install
$ npm run dev
```

#### 基础设置，代码规范的支持 (eslint+prettier)

vscode 安装 `eslint`,`prettier`,`vetur`(喜欢用 vue3 setup 语法糖可以使用`volar`，这时要禁用`vetur`)

打开 vscode eslint  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U9lLzsEXB7vonmcCcez44ibPHCgiaxLylMTu8LWpPl9fch8P1jZaXfTXvQ/640?wx_fmt=png)

##### eslint

```
yarn add --dev eslint eslint-plugin-vue @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

##### prettier

```
yarn add --dev prettier eslint-config-prettier eslint-plugin-prettier
```

##### .prettierrc.js

```
module.exports = {
    printWidth: 180, //一行的字符数，如果超过会进行换行，默认为80
    tabWidth: 4, //一个tab代表几个空格数，默认为80
    useTabs: false, //是否使用tab进行缩进，默认为false，表示用空格进行缩减
    singleQuote: true, //字符串是否使用单引号，默认为false，使用双引号
    semi: false, //行位是否使用分号，默认为true
    trailingComma: 'none', //是否使用尾逗号，有三个可选值"<none|es5|all>"
    bracketSpacing: true, //对象大括号直接是否有空格，默认为true，效果：{ foo: bar }
    jsxSingleQuote: true, // jsx语法中使用单引号
    endOfLine: 'auto'
}
```

##### .eslintrc.js

```
//.eslintrc.js
module.exports = {
    parser: 'vue-eslint-parser',
    parserOptions: {
        parser: '@typescript-eslint/parser', // Specifies the ESLint parser
        ecmaVersion: 2020, // Allows for the parsing of modern ECMAScript features
        sourceType: 'module', // Allows for the use of imports
        ecmaFeatures: {
            jsx: true
        }
    },
    extends: [
        'plugin:vue/vue3-recommended',
        'plugin:@typescript-eslint/recommended',
        'prettier',
        'plugin:prettier/recommended'
    ]
}
```

##### .settings.json(工作区)

```
{
    "editor.codeActionsOnSave": {
        "source.fixAll.eslint": true
    },
    "eslint.validate": [
        "javascript",
        "javascriptreact",
        "vue",
        "typescript",
        "typescriptreact",
        "json"
    ]
}
```

#### 目录结构范例

```
├─.vscode           // vscode配置文件
├─public            // 无需编译的静态资源目录
├─src                // 代码源文件目录
│  ├─apis            // apis统一管理
│  │  └─modules        // api模块
│  ├─assets            // 静态资源
│  │  └─images      
│  ├─components     // 项目组件目录
│  │  ├─Form
│  │  ├─Input
│  │  ├─Message
│  │  ├─Search
│  │  ├─Table
│  ├─directives     // 指令目录
│  │  └─print
│  ├─hooks            // hooks目录
│  ├─layouts        // 布局组件
│  │  ├─dashboard
│  │  │  ├─content
│  │  │  ├─header
│  │  │  └─sider
│  │  └─fullpage
│  ├─mock           // mock apu存放地址，和apis对应
│  │  └─modules
│  ├─router            // 路由相关
│  │  └─helpers
│  ├─store            // 状态管理相关
│  ├─styles            // 样式相关(后面降到css架构会涉及具体的目录)
│  ├─types            // 类型定义相关
│  ├─utils            // 工具类相关
│  └─views            // 页面目录地址
│      ├─normal    
│      └─system
└─template            // 模板相关
    ├─apis
    └─page
```

#### CSS 架构之`ITCSS` + `BEM` + `ACSS`

现实开发中，我们经常忽视 CSS 的架构设计。前期对样式架构的忽略，随着项目的增大，导致出现样式污染，覆盖，难以追溯，代码重复等各种问题。因此，CSS 架构设计同样需要重视起来。

*   `ITCSS`  
    ITCSS 是 CSS 设计方法论，它并不是具体的 CSS 约束，他可以让你更好的管理、维护你的项目的 CSS。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U9klnL66CNvNB3U3b3t9iajTHVXI1UAD2aKDL46GEus62gYRlfH5SxBibg/640?wx_fmt=png)

ITCSS 把 CSS 分成了以下的几层

<table data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)"><thead data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)"><tr data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); text-align: center; min-width: 85px;">Layer</th><th data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); text-align: center; min-width: 85px;">作用</th></tr></thead><tbody data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)"><tr data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;">Settings</td><td data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;">项目使用的全局变量</td></tr><tr data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;">Tools</td><td data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;">mixin，function</td></tr><tr data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;">Generic</td><td data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;">最基本的设定 normalize.css，reset</td></tr><tr data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;">Base</td><td data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;">type selector</td></tr><tr data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;">Objects</td><td data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;">不经过装饰 (Cosmetic-free) 的设计模式</td></tr><tr data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;">Components</td><td data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;">UI 组件</td></tr><tr data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;">Trumps</td><td data-darkmode-color-16475466230508="rgb(163, 163, 163)" data-darkmode-original-color-16475466230508="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475466230508="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475466230508="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;">helper 唯一可以使用 important! 的地方</td></tr></tbody></table>

以上是给的范式，我们不一定要完全按照它的方式，可以结合`BEM`和`ACSS`

目前我给出的 CSS 文件目录 (暂定)  
└─styles

```
├───acss
├───generic
├───theme
├───tools
└───transition 
```

*   `BEM`  
    即 Block, Element, Modifier，是`OOCSS`(面向对象 css) 的进阶版， 它是一种基于组件的 web 开发方法。blcok 可以理解成独立的块，在页面中该块的移动并不会影响到内部样式（和组件的概念类似，独立的一块），element 就是块下面的元素，和块有着藕断丝连的关系，modifier 是表示样式大小等。  
    我们来看一下`element-ui`的做法
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U9qO7F1nibSFwyhCIgiccB40ZUYRyHZpZdZ7zoyH9H6ibdV5buxTRjNpYicQ/640?wx_fmt=png)

* * *

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U9qO7F1nibSFwyhCIgiccB40ZUYRyHZpZdZ7zoyH9H6ibdV5buxTRjNpYicQ/640?wx_fmt=png)

我们项目组件的开发或者封装统一使用`BEM`

*   `ACSS`  
    了解`tailwind`的人应该对此设计模式不陌生，即原子级别的 CSS。像. fr,.clearfix 这种都属于 ACSS 的设计思维。此处我们可以用此模式写一些变量等。
    

#### JWT(json web token)

JWT 是一种跨域认证解决方案  
http 请求是无状态的，服务器是不认识前端发送的请求的。比如登录，登录成功之后服务端会生成一个 sessionKey，sessionKey 会写入 Cookie，下次请求的时候会自动带入 sessionKey，现在很多都是把用户 ID 写到 cookie 里面。

这是有问题的，比如要做单点登录，用户登录 A 服务器的时候，服务器生成 sessionKey，登录 B 服务器的时候服务器没有 sessionKey，所以并不知道当前登录的人是谁，所以 sessionKey 做不到单点登录。但是 jwt 由于是服务端生成的 token 给客户端，存在客户端，所以能实现单点登录。

###### 特点

*   由于使用的是 json 传输，所以 JWT 是跨语言的
    
*   便于传输，jwt 的构成非常简单，字节占用很小，所以它是非常便于传输的
    
*   jwt 会生成签名，保证传输安全
    
*   jwt 具有时效性
    
*   jwt 更高效利用集群做好单点登录
    
    ###### 数据结构
    
*   Header.Payload.Signature
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U9gLxWnLPMK14x4bM9MQxR6NWIcaxfwUMxseBu1PADUNXiatoDic5oyy9Q/640?wx_fmt=png)

##### 数据安全

不应该在 jwt 的 payload 部分存放敏感信息，因为该部分是客户端可解密的部分

保护好 secret 私钥，该私钥非常重要

如果可以，请使用 https 协议

##### 使用流程

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U9prCWfwL3dcrUepFazicjchVjRrkLKtiaibmJIIYREjNgqQpa1U0uo9TbA/640?wx_fmt=png)

  

##### 使用方式

*   后端
    
    ```
    const router = require('koa-router')()
    const jwt = require('jsonwebtoken')
    
    router.post('/login', async (ctx) => {
      try {
          const { userName, userPwd } = ctx.request.body
          const res = await User.findOne({
              userName,
              userPwd
          })
          const data = res._doc
          const token = jwt.sign({
              data
          }, 'secret', { expiresIn: '1h' })
          if(res) {
              data.token = token
              ctx.body = data
          }
      } catch(e) {
          
      }
      
    } )
    ```
    
*   前端
    
    ```
    // axios请求拦截器，Cookie写入token,请求头添加:Authorization: Bearer `token`
    service.interceptors.request.use(
      request => {
          const token = Cookies.get('token') // 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ'
          token && (request.headers['Authorization'] = token)
          return request
      },
      error => { 
          Message.error(error)
      }
    )
    ```
    
*   后端验证有效性
    
    ```
    const app = new Koa()
    const router = require('koa-router')()
    const jwt = require('jsonwebtoken')
    const koajwt = require('koa-jwt')
    // 使用koa-jwt中间件不用在接口之前拦截进行校验
    app.use(koajwt({ secret:'secret' }))
    // 验证不通过会将http状态码返回401
    app.use(async (ctx, next) => {
      await next().catch(err => {
          if(err.status === 401) {
              ctx.body.msg = 'token认证失败'
          }
      })
    })
    ```
    

#### 菜单设计

关于菜单的生成方式有很多种，比较传统的是前端维护一个菜单树，根据后端返回的菜单树进行过滤。这种方式实际上提前将路由注册进入到实例中，这种现在其实已经不是最佳实践了。

现在主流的思路是后端通过`XML`来配置菜单，通过配置来生成菜单。前端登录的时候拉取该角色对应的菜单，通过`addroute`方法注册菜单相应的路由地址以及页面在前端项目中的路径等。这是比较主流的，但是我个人觉得不算最完美。

我们菜单和前端代码其实是强耦合的，包括路由地址，页面路径，图标，重定向等。项目初期菜单可能是经常变化的，每次对菜单进行添加或者修改等操作的时候，需要通知后端修改`XML`，并且后端的`XML`实际上就是没有树结构，看起来也不是很方便。

因此我采用如下设计模式，**_`前端`_** 维护一份`menu.json`，所写即所得，json 数是什么样在菜单配置的时候就是什么样。

##### 结构设计

```
string
```

> `注意事项`：同级的 name 要是唯一的，实际使用中，每一级的 name 都是通过上一级的 name 用`-`拼接而来（会通过动态导入章节演示 name 的生成规则），这样可以保证每一个菜单或者按钮项都有唯一的标识。后续不论是做按钮权限控制还是做菜单的缓存，都与此拼接的 name 有关。我们注意此时没有 id，后续会讲到根据 name 全称使用 md5 来生成 id。

示例代码

```
[
    {
        "title": "admin",
        "name": "admin",
        "type": "MODULE",
        "children": [
            {
                "title": "中央控制台",
                "path": "/platform",
                "name": "platform",
                "type": "MENU",
                "component": "/platform/index",
                "icon": "mdi:monitor-dashboard"
            },
            {
                "title": "系统设置",
                "name": "system",
                "type": "MENU",
                "path": "/system",
                "icon": "ri:settings-5-line",
                "children": [
                    {
                        "title": "用户管理",
                        "name": "user",
                        "type": "MENU",
                        "path": "user",
                        "component": "/system/user"
                    },
                    {
                        "title": "角色管理",
                        "name": "role",
                        "type": "MENU",
                        "path": "role",
                        "component": "/system/role"
                    },
                    {
                        "title": "资源管理",
                        "name": "resource",
                        "type": "MENU",
                        "path": "resource",
                        "component": "/system/resource"
                    }
                ]
            },
            {
                "title": "实用功能",
                "name": "function",
                "type": "MENU",
                "path": "/function",
                "icon": "ri:settings-5-line",
                "children": []
            }
        ]
    }
] 
```

生成的菜单树  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U9qSXhx6KfszUOh8jqyqxMqSXAictbYnARicy8PCrhm7DVHCE4KuDicWc4A/640?wx_fmt=png)

> 如果觉得所有页面的路由写在一个页面中太长，难以维护的话，可以把`json`换成 js 用`import`机制，这里涉及到的变动比较多，暂时先不提及

使用时，我们分`development`和`production`两种环境

*   `development`：该模式下，菜单树直接读取 menu.json 文件
    
*   `production`: 该模式下，菜单树通过接口获取数据库的数据
    

##### 如何存到数据库

OK，我们之前提到过，菜单是由前端通过 menu.json 来维护的，那怎么进到数据库中呢？实际上，我的设计是通过`node`读取`menu.json`文件，然后创建 SQL 语句，交给后端放到`liquibase`中，这样不管有多少个数据库环境，后端只要拿到该 SQL 语句，就能在多个环境创建菜单数据。当然，由于`json`是可以跨语言通信的，所以我们可以直接把`json`文件丢给后端，或者把项目`json`路径丢给运维，通过`CI/CD`工具完成自动发布。

nodejs 生成 SQL 示例

```
// createMenu.js
/**
 *
 * =================MENU CONFIG======================
 *
 * this javascript created to genarate SQL for Java
 *
 * ====================================================
 *
 */

const fs = require('fs')
const path = require('path')
const chalk = require('chalk')
const execSync = require('child_process').execSync //同步子进程
const resolve = (dir) => path.join(__dirname, dir)
const moment = require('moment')
// get the Git user name to trace who exported the SQL
const gitName = execSync('git show -s --format=%cn').toString().trim()
const md5 = require('md5')
// use md5 to generate id

/* =========GLOBAL CONFIG=========== */

// 导入路径
const INPUT_PATH = resolve('src/router/menu.json')
// 导出的文件目录位置
const OUTPUT_PATH = resolve('./menu.sql')
// 表名
const TABLE_NAME = 't_sys_menu'

/* =========GLOBAL CONFIG=========== */

function createSQL(data, name = '', pid, arr = []) {
    data.forEach(function (v, d) {
        if (v.children && v.children.length) {
            createSQL(v.children, name + '-' + v.name, v.id, arr)
        }
        arr.push({
            id: v.id || md5(v.name), // name is unique,so we can use name to generate id
            created_at: moment().format('YYYY-MM-DD HH:mm:ss'),
            modified_at: moment().format('YYYY-MM-DD HH:mm:ss'),
            created_by: gitName,
            modified_by: gitName,
            version: 1,
            is_delete: false,
            code: (name + '-' + v.name).slice(1),
            name: v.name,
            title: v.title,
            icon: v.icon,
            path: v.path,
            sort: d + 1,
            parent_id: pid,
            type: v.type,
            component: v.component,
            redirect: v.redirect,
            full_screen: v.fullScreen || false, 
            hidden: v.hidden || false,
            no_cache: v.noCache || false
        })
    })
    return arr
}

fs.readFile(INPUT_PATH, 'utf-8', (err, data) => {
    if (err) chalk.red(err)
    const menuList = createSQL(JSON.parse(data))
    const sql = menuList
        .map((sql) => {
            let value = ''
            for (const v of Object.values(sql)) {
                value += ','
                if (v === true) {
                    value += 1
                } else if (v === false) {
                    value += 0
                } else {
                    value += v ? `'${v}'` : null
                }
            }
            return 'INSERT INTO `' + TABLE_NAME + '` VALUES (' + value.slice(1) + ')' + '\n'
        })
        .join(';')
    const mySQL =
        'DROP TABLE IF EXISTS `' +
        TABLE_NAME +
        '`;' +
        '\n' +
        'CREATE TABLE `' +
        TABLE_NAME +
        '` (' +
        '\n' +
        '`id` varchar(64) NOT NULL,' +
        '\n' +
        "`created_at` timestamp NULL DEFAULT NULL COMMENT '创建时间'," +
        '\n' +
        "`modified_at` timestamp NULL DEFAULT NULL COMMENT '更新时间'," +
        '\n' +
        "`created_by` varchar(64) DEFAULT NULL COMMENT '创建人'," +
        '\n' +
        "`modified_by` varchar(64) DEFAULT NULL COMMENT '更新人'," +
        '\n' +
        "`version` int(11) DEFAULT NULL COMMENT '版本（乐观锁）'," +
        '\n' +
        "`is_delete` int(11) DEFAULT NULL COMMENT '逻辑删除'," +
        '\n' +
        "`code` varchar(150) NOT NULL COMMENT '编码'," +
        '\n' +
        "`name` varchar(50) DEFAULT NULL COMMENT '名称'," +
        '\n' +
        "`title` varchar(50) DEFAULT NULL COMMENT '标题'," +
        '\n' +
        "`icon` varchar(50) DEFAULT NULL COMMENT '图标'," +
        '\n' +
        "`path` varchar(250) DEFAULT NULL COMMENT '路径'," +
        '\n' +
        "`sort` int(11) DEFAULT NULL COMMENT '排序'," +
        '\n' +
        "`parent_id` varchar(64) DEFAULT NULL COMMENT '父id'," +
        '\n' +
        "`type` char(10) DEFAULT NULL COMMENT '类型'," +
        '\n' +
        "`component` varchar(250) DEFAULT NULL COMMENT '组件路径'," +
        '\n' +
        "`redirect` varchar(250) DEFAULT NULL COMMENT '重定向路径'," +
        '\n' +
        "`full_screen` int(11) DEFAULT NULL COMMENT '全屏'," +
        '\n' +
        "`hidden` int(11) DEFAULT NULL COMMENT '隐藏'," +
        '\n' +
        "`no_cache` int(11) DEFAULT NULL COMMENT '缓存'," +
        '\n' +
        'PRIMARY KEY (`id`),' +
        '\n' +
        'UNIQUE KEY `code` (`code`) USING BTREE' +
        '\n' +
        ") ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='资源';" +
        '\n' +
        sql
    fs.writeFile(OUTPUT_PATH, mySQL, (err) => {
        if (err) return chalk.red(err)
        console.log(chalk.cyanBright(`恭喜你，创建sql语句成功，位置：${OUTPUT_PATH}`))
    })
}) 
```

> 注意上面是通过使用`md5`对`name`进行加密生成主键`id`到数据库中

我们尝试用 node 执行该 js

```
node createMenu.js
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U96JibMSTeSRkZyqFTMlIfMWicLL8JiaS5WhibjRRJVb1CoemQt2kYZiaWLoA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U9mNQia0AViadfaFHtia3rLWluic78VX3BG9vw2cKhEmcnevB3ftSkndkhbw/640?wx_fmt=png)

由于生产环境不会直接引入`menu.json`，因此经过打包编译的线上环境不会存在该文件，因此也不会有安全性问题

##### 如何控制到按钮级别

我们知道，按钮 (这里的按钮是广义上的，对于前端来说可能是 button，tab，dropdown 等一切可以控制的内容) 的载体一定是页面，因此按钮可以直接挂在到 menu 树的`MENU`类型的资源下面，没有页面页面权限当然没有该页面下的按钮权限，有页面权限的情况下，我们通过`v-permission`指令来控制按钮的显示  
示例代码

```
// 生成权限按钮表存到store
const createPermissionBtns = router => {
    let btns = []
    const c = (router, name = '') => {
        router.forEach(v => {
            v.type === 'BUTTON' && btns.push((name + '-' + v.name).slice(1))
            return v.children && v.children.length ? c(v.children, name + '-' + v.name) : null
        })
        return btns
    }
    return c(router)
}
```

```
// 权限控制
Vue.directive('permission', {
    // 这里是vue3的写法，vue2请使用inserted生命周期
    mounted(el, binding, vnode) {
        // 获取this
        const { context: vm } = vnode
        // 获取绑定的值
        const name = vm.$options.name + '-' + binding.value
        // 获取权限表
        const {
            state: { permissionBtns }
        } = store
        // 如果没有权限那就移除
        if (permissionBtns.indexOf(name) === -1) {
            el.parentNode.removeChild(el)
        }
    }
})
```

```
<el-button type="text" v-permission="'edit'" @click="edit(row.id)">编辑</el-button>
```

假设当前页面的 name 值是`system-role`，按钮的 name 值是`system-role-edit`，那么通过此指令就可以很方便的控制到按钮的权限

##### 动态导入

我们`json`或者接口配置的路由前端页面地址，在`vue-router`中又是如何注册进去的呢？

> 注意以下 name 的生成规则，以角色菜单为例，name 拼接出的形式大致为：
> 
> *   一级菜单：system
>     
> *   二级菜单：system-role
>     
> *   该二级菜单下的按钮：system-role-edit
>     

*   `vue-cli` vue-cli3 及以上可以直接使用 webpack4 + 引入的`dynamic import`
    
    ```
    // 生成可访问的路由表
    const generateRoutes = (routes, cname = '') => {
      return routes.reduce((prev, { type, uri: path, componentPath, name, title, icon, redirectUri: redirect, hidden, fullScreen, noCache, children = [] }) => {
          // 是菜单项就注册到路由进去
          if (type === 'MENU') {
              prev.push({
                  path,
                  component: () => import(`@/${componentPath}`),
                  name: (cname + '-' + name).slice(1),
                  props: true,
                  redirect,
                  meta: { title, icon, hidden, type, fullScreen, noCache },
                  children: children.length ? createRouter(children, cname + '-' + name) : []
              })
          }
          return prev
      }, [])
    }
    ```
    
*   `vite` vite2 之后可以直接使用 glob-import
    
    ```
    // dynamicImport.ts
    export default function dynamicImport(component: string) {
      const dynamicViewsModules = import.meta.glob('../../views/**/*.{vue,tsx}')
      const keys = Object.keys(dynamicViewsModules)
      const matchKeys = keys.filter((key) => {
          const k = key.replace('../../views', '')
          return k.startsWith(`${component}`) || k.startsWith(`/${component}`)
      })
      if (matchKeys?.length === 1) {
          const matchKey = matchKeys[0]
          return dynamicViewsModules[matchKey]
      }
      if (matchKeys?.length > 1) {
          console.warn(
              'Please do not create `.vue` and `.TSX` files with the same file name in the same hierarchical directory under the views folder. This will cause dynamic introduction failure'
          )
          return
      }
      return null
    }
    ```
    

```
import type { IResource, RouteRecordRaw } from '../types'
import dynamicImport from './dynamicImport'

// 生成可访问的路由表
const generateRoutes = (routes: IResource[], cname = '', level = 1): RouteRecordRaw[] => {
    return routes.reduce((prev: RouteRecordRaw[], curr: IResource) => {
        // 如果是菜单项则注册进来
        const { id, type, path, component, name, title, icon, redirect, hidden, fullscreen, noCache, children } = curr
        if (type === 'MENU') {
            // 如果是一级菜单没有子菜单，则挂在在app路由下面
            if (level === 1 && !(children && children.length)) {
                prev.push({
                    path,
                    component: dynamicImport(component!),
                    name,
                    props: true,
                    meta: { id, title, icon, type, parentName: 'app', hidden: !!hidden, fullscreen: !!fullscreen, noCache: !!noCache }
                })
            } else {
                prev.push({
                    path,
                    component: component ? dynamicImport(component) : () => import('/@/layouts/dashboard'),
                    name: (cname + '-' + name).slice(1),
                    props: true,
                    redirect,
                    meta: { id, title, icon, type, hidden: !!hidden, fullscreen: !!fullscreen, noCache: !!noCache },
                    children: children?.length ? generateRoutes(children, cname + '-' + name, level + 1) : []
                })
            }
        }
        return prev
    }, [])
}

export default generateRoutes
```

##### 动态注册路由

要实现动态添加路由，即只有有权限的路由才会注册到 Vue 实例中。考虑到每次刷新页面的时候由于 vue 的实例会丢失，并且角色的菜单也可能会更新，因此在每次加载页面的时候做菜单的拉取和路由的注入是最合适的时机。因此核心是`vue-router`的`addRoute`和导航守卫`beforeEach`两个方法

要实现动态添加路由，即只有有权限的路由才会注册到 Vue 实例中。考虑到每次刷新页面的时候由于 vue 的实例会丢失，并且角色的菜单也可能会更新，因此在每次加载页面的时候做菜单的拉取和路由的注入是最合适的时机。因此核心是 vue-router 的`addRoute`和导航钩子`beforeEach`两个方法

`vue-router3x`  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U9x7GJxXdVMQf0l8UDibyeSb6ZzARaTfRaUujK0mqWSlJ01OeM7FpficzQ/640?wx_fmt=png)

> `注`：3.5.0API 也更新到了 addRoute，注意区分版本变化

`vue-router4x`  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U9h3NBVYCib0M5ld3Yg9W6M2udmax1R7yxpvaQ4Utoy0sSQoibkN3DomRA/640?wx_fmt=png)

个人更倾向于使用`vue-router4x`的`addRoute`方法，这样可以更精细的控制每一个路由的的定位  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U9hZiaiaentUxfFI075RtIyy74Ro2icLSQ0LaFp7rl0pLYYfmdnmaicPCZxA/640?wx_fmt=png)

大体思路为，在`beforeEach`该导航守卫中 (即每次路由跳转之前做判断)，如果已经授权过 (`authorized`)，就直接进入 next 方法，如果没有，则从后端拉取路由表注册到实例中。(直接在入口文件`main.js`中引入以下文件或代码)

```
// permission.js
router.beforeEach(async (to, from, next) => {
    const token = Cookies.get('token')
    if (token) {
        if (to.path === '/login') {
            next({ path: '/' })
        } else {
            if (!store.state.authorized) {
                // set authority
                await store.dispatch('setAuthority')
                // it's a hack func,avoid bug
                next({ ...to, replace: true })
            } else {
                next()
            }
        }
    } else {
        if (to.path !== '/login') {
            next({ path: '/login' })
        } else {
            next(true)
        }
    }
})
```

由于路由是动态注册的，所以项目的初始路由就会很简洁，只要提供静态的不需要权限的基础路由，其他路由都是从服务器返回之后动态注册进来的

```
// router.js
import { createRouter, createWebHistory } from 'vue-router'
import type { RouteRecordRaw } from './types'

// static modules
import Login from '/@/views/sys/Login.vue'
import NotFound from '/@/views/sys/NotFound.vue'
import Homepage from '/@/views/sys/Homepage.vue'
import Layout from '/@/layouts/dashboard'

const routes: RouteRecordRaw[] = [
    {
        path: '/',
        redirect: '/homepage'
    },
    {
        path: '/login',
        component: Login
    },
    // for 404 page
    {
        path: '/:pathMatch(.*)*',
        component: NotFound
    },
    // to place the route who don't have children
    {
        path: '/app',
        component: Layout,
        name: 'app',
        children: [{ path: '/homepage', component: Homepage, name: 'homepage', meta: { title: '首页' } }]
    }
]

const router = createRouter({
    history: createWebHistory(),
    routes,
    scrollBehavior() {
        // always scroll to top
        return { top: 0 }
    }
})
export default router 
```

##### 左侧菜单树和按钮生成

其实只要递归拿到 type 为`MENU`的资源注册到路由，过滤掉`hidden:true`的菜单在左侧树显示，此处不再赘述。

#### RBAC(Role Based Access Control)

RBAC 是基于角色的访问控制（Role-Based Access Control ）在 RBAC 中，权限与角色相关联，用户通过成为适当角色的成员而得到这些角色的权限。这就极大地简化了权限的管理。这样管理都是层级相互依赖的，权限赋予给角色，而把角色又赋予用户，这样的权限设计很清楚，管理起来很方便。

这样登录的时候只要获取用户

用户选择角色  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U9pSnW908goqGlrPABlmeOAqicuV3sCicFibwGj836mrvVAvTJKqVqc2IgA/640?wx_fmt=png)

角色绑定菜单  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U9yhf3hHMomgXibGEgnQqtgWEj6PyImfQeQahV5FOKhRlzGA5icXxhshDA/640?wx_fmt=png)

菜单  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U95XF6Vb0eCt5zGaRFh54x3hAID6to2L0jZqeb7eq2bzFc0jj8WsGaug/640?wx_fmt=png)

#### 页面缓存控制

页面缓存，听起来无关紧要的功能，却能给客户带来极大的使用体验的提升。  
例如我们有一个分页列表，输入某个查询条件之后筛选出某一条数据，点开详情之后跳转到新的页面，关闭详情返回分页列表的页面，假如之前查询的状态不存在，用户需要重复输入查询条件，这不仅消耗用户的耐心，也增加了服务器不必要的压力。

因此，缓存控制在系统里面很有存在的价值，我们知道`vue`有`keep-alive`组件可以让我们很方便的进行缓存，那么是不是我们直接把根组件直接用`keep-alive`包装起来就好了呢？

实际上这样做是不合适的，比如我有个用户列表，打开小明和小红的详情页都给他缓存起来，由于缓存是写入内存的，用户使用系统久了之后必将导致系统越来越卡。并且类似于详情页这种数据应该是每次打开的时候都从接口获取一次才能保证是最新的数据，将它也缓存起来本身就是不合适的。那么按需缓存就是我们系统迫切需要使用的，好在`keep-alive`给我们提供了`include`这个 api

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U9xlc6QqGDjCUjtibiaX9SKyUhF9hoicCpiaHVdy3G2XvuyRcFlhcKNfzVFA/640?wx_fmt=png)

> 注意这个 include 存的是页面的 name，不是路由的 name

因此，如何定义页面的 name 是很关键的

我的做法是，vue 页面的 name 值与当前的`menu.json`的层级相连的`name`(实际上经过处理就是注册路由的时候的全路径 name) 对应，参考动态导入的介绍，这样做用两个目的：

*   我们知道 vue 的缓存组件`keep-alive`的`include`选项是基于页面的`name`来缓存的，我们使路由的`name`和页面的`name`保持一致，这样我们一旦路由发生变化，我们将所有路由的`name`存到`store`中，也就相当于存了页面的`name`到了`store`中，这样做缓存控制会很方便。当然页面如果不需要缓存，可以在`menu.json`中给这个菜单`noCache`设置为`true`, 这也是我们菜单表结构中该字段的由来。
    
*   我们开发的时候一般都会安装`vue-devtools`进行调试，语义化的`name`值方便进行调试。
    

例如角色管理

对应的 json 位置  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U9dSzzJR8KBHspTm68bSasMlgxTKcsOMpAaKE34OuqvOlQPSLU0HvzJQ/640?wx_fmt=png)

对应的 vue 文件  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U9jzIjsZe5IObVmfSAxXicXY841GFvKgw8KTzXicC8RZdAEqswXsV2wfeg/640?wx_fmt=png)

对应的`vue-devtools`  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U97Jtr2vMJfV8ibwjbzUWSOoBHoDvThKodKdiaUY0DMZibHb5L4WrD1FsUw/640?wx_fmt=png)

为了更好的用户体验，我们在系统里面使用 tag 来记录用户之前点开的页面的状态。其实这也是一个`hack`手段，无非是解决`SPA`项目的一个痛点。

效果图  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U9keBrq7Cg3lHb5s9Wzib6p1nqzmgpzyicflXdN7ON2OiahW45qFH2CGLUQ/640?wx_fmt=png)

大概思路就是监听路由变化，把所有路由的相关信息存到`store`中。根据该路由的`noCache`字段显示不同的小图标，告诉用户这个路由是否是带有缓存的路由。

#### 组件的封装或者基于 UI 库的二次封装

组件的封装原则无非就是复用，可扩展。

我们在最初封装组件的时候不用追求过于完美，满足基础的业务场景即可。后续根据需求变化再去慢慢完善组件。

如果是多人团队的大型项目还是建议使用`Jest`做好单元测试配合`storybook`生成组件文档。

关于组件的封装技巧，网上有很多详细的教程，本人经验有限，这里就不再讨论。

#### 使用 plop 创建模板

基本框架搭建完毕，组件也封装好了之后，剩下的就是码业务功能了。  
对于中后台管理系统，业务部分大部分离不开`CRUD`，我们看到上面的截图，类似用户，角色等菜单，组成部分都大同小异，前端部分只要封装好组件 (列表，表单，弹框等)，页面都可以直接通过模板来生成。甚至现在有很多可视化配置工具 (低代码)，我个人觉得目前不太适合专业前端，因为很多场景下页面的组件都是基于业务封装的，单纯的把 UI 库原生组件搬过来没有意义。当然时间充足的话，可以自己在项目上用 node 开发低代码的工具。

这里我们可以配合 inquirer-directory 来在控制台选择目录

*   plopfile.js
    
    ```
    const promptDirectory = require('inquirer-directory')
    const pageGenerator = require('./template/page/prompt')
    const apisGenerator = require('./template/apis/prompt')
    module.exports = function (plop) {
      plop.setPrompt('directory', promptDirectory)
      plop.setGenerator('page', pageGenerator)
      plop.setGenerator('apis', apisGenerator)
    }
    ```
    

一般情况下， 我们和后台定义好 restful 规范的接口之后，每当有新的业务页面的时候，我们要做两件事情，一个是写好接口配置，一个是写页面，这两个我们可以通过模板来创建了。我们使用`hbs`来创建。

*   api.hbs
    

```
import request from '../request'
{{#if create}}
// Create
export const create{{ properCase name }} = (data: any) => request.post('{{camelCase name}}/', data)
{{/if}}
{{#if delete}}
// Delete
export const remove{{ properCase name }} = (id: string) => request.delete(`{{camelCase name}}/${id}`)
{{/if}}
{{#if update}}
// Update
export const update{{ properCase name }} = (id: string, data: any) => request.put(`{{camelCase name}}/${id}`, data)
{{/if}}
{{#if get}}
// Retrieve
export const get{{ properCase name }} = (id: string) => request.get(`{{camelCase name}}/${id}`)
{{/if}}
{{#if check}}
// Check Unique
export const check{{ properCase name }} = (data: any) => request.post(`{{camelCase name}}/check`, data)
{{/if}}
{{#if fetchList}}
// List query
export const fetch{{ properCase name }}List = (params: any) => request.get('{{camelCase name}}/list', { params })
{{/if}}
{{#if fetchPage}}
// Page query
export const fetch{{ properCase name }}Page = (params: any) => request.get('{{camelCase name}}/page', { params })
{{/if}} 
```

*   prompt.js
    
    ```
    const { notEmpty } = require('../utils.js')
    
    const path = require('path')
    
    // 斜杠转驼峰
    function toCamel(str) {
      return str.replace(/(.*)\/(\w)(.*)/g, function (_, $1, $2, $3) {
          return $1 + $2.toUpperCase() + $3
      })
    }
    // 选项框
    const choices = ['create', 'update', 'get', 'delete', 'check', 'fetchList', 'fetchPage'].map((type) => ({
      name: type,
      value: type,
      checked: true
    }))
    
    module.exports = {
      description: 'generate api template',
      prompts: [
          {
              type: 'directory',
              name: 'from',
              message: 'Please select the file storage address',
              basePath: path.join(__dirname, '../../src/apis')
          },
          {
              type: 'input',
              name: 'name',
              message: 'api name',
              validate: notEmpty('name')
          },
          {
              type: 'checkbox',
              name: 'types',
              message: 'api types',
              choices
          }
      ],
      actions: (data) => {
          const { from, name, types } = data
          const actions = [
              {
                  type: 'add',
                  path: path.join('src/apis', from, toCamel(name) + '.ts'),
                  templateFile: 'template/apis/index.hbs',
                  data: {
                      name,
                      create: types.includes('create'),
                      update: types.includes('update'),
                      get: types.includes('get'),
                      check: types.includes('check'),
                      delete: types.includes('delete'),
                      fetchList: types.includes('fetchList'),
                      fetchPage: types.includes('fetchPage')
                  }
              }
          ]
    
          return actions
      }
    } 
    ```
    

我们来执行 plop

通过`inquirer-directory`，我们可以很方便的选择系统目录  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U9r7ZvXERDib3VOQmXUav4sEcmvJKNZcUjcKMKSEXicFmU3cFXYv1kvorQ/640?wx_fmt=png)

输入 name 名，一般对应后端的 controller 名称  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U98Orjwta5sAjbljTgGCaoI1EjYWd5Lm6hmCBfEpvsgGzwcVibIJNxoOw/640?wx_fmt=png)

使用空格来选择每一项，使用回车来确认  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U90tJMMv7gOU5ibPRGVJUdaVtlqFdnYa72ENRtibWglaND6shVFbuvrPNQ/640?wx_fmt=png)

最终生成的文件  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHoSxo8cdoRQU6hb5FjG31U9iasyibIytRlibth9ALSBXqh4UmvzDtW8zLFImA2Sbz56BIIwkBtkqhrgw/640?wx_fmt=png)

> 生成页面的方式与此类似，我这边也只是抛砖引玉，相信大家能把它玩出花来

##### 项目地址

levi-vue-admin

> 作者：sky124380729
> 
> https://segmentfault.com/a/1190000040096254  

- EOF -

推荐阅读  点击标题可跳转

1、[这几款基于 vue3 和 vite 的开箱即用的中后台管理模版, 让你 yyds !](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651587483&idx=2&sn=a342b5240d6f2d661e7c4e9db0081ba4&chksm=8022d45ab7555d4c3daee8b044c18602fb624c8478d344dcb6b50567bdc0e10c2e601139af2c&scene=21#wechat_redirect)

2、[知乎热议的 Vue3 Ref 语法糖，实现原理是啥？](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651584969&idx=1&sn=266b8800c6a19e08a2e6bc319034c3cd&chksm=80252e08b752a71e1b3d4a36f8d7ea1dff13680fda00b65f30dfc1613ce20cdfe0f41736eacd&scene=21#wechat_redirect)

3、[我曾为 TS 类型编程感到痛不欲生，直到我遇到了这份体操指南](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651581039&idx=3&sn=c22ec7a231ddcd6ba5021ad69e39de6a&chksm=80253daeb752b4b8a5d1c5b05384a228c7f8baee33e86cfaa361560f779d1176f2be48f0bec3&scene=21#wechat_redirect)