> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651578023&idx=2&sn=2932be198b2dfc753a5c62882c2ea90e&chksm=80250966b7528070940dcb13f7ae91cd5e021bb8faf5c4a177828bb0b5ae82dbd693f7228a0d&mpshare=1&scene=1&srcid=0618yL5OuMppQczuuh0IiZlh&sharer_sharetime=1623989252839&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

前言
==

Vue Router 是 Vue.js[1] 官方的路由管理器。

它和 Vue.js 的核心深度集成，让构建单页面应用变得易如反掌。

包含的功能有：

*   嵌套的路由 / 视图表
    
*   模块化的、基于组件的路由配置
    
*   路由参数、查询、通配符
    
*   基于 Vue.js 过渡系统的视图过渡效果
    
*   细粒度的导航控制
    
*   带有自动激活的 CSS class 的链接
    
*   HTML5 历史模式或 hash 模式，在 IE9 中自动降级
    
*   自定义的滚动条行为
    

本文是作者是实际项目中遇到的一些总结，希望对你有所帮助。

正文
==

1. 响应路由参数变化
-----------

针对复用组件（只是路由参数发生改变），生命周期函数钩子不会被调用，如何能刷新组件了？

*   watch 监听
    
    ```
    watch: {
      '$route' (to, from) {
      // 对路由变化作出响应...
      }
    }
    ```
    
*   beforeRouteUpdate
    
    ```
    beforeRouteUpdate (to, from, next) {
    // react to route changes...
    / / don't forget to call next()
    }
    ```
    
    2. 路由匹配
    -------
    
    ```
    {
    // 会匹配所有路径
    path: '*'
    }
    {
    // 会匹配以 `/user-` 开头的任意路径
    path: '/user-*'
    }
    ```
    
    注意：当使用通配符路由时，请确保路由的顺序是正确的，也就是说含有通配符的路由应该放在最后。路由 {path: '*'} 通常用于客户端 404 错误。
    

如果你使用了 History 模式，请确保正确配置你的服务器。

当使用一个通配符时，$route.params 内会自动添加一个名为 pathMatch 参数。

它包含了 URL 通过通配符被匹配的部分：

3. 高级匹配模式
---------

```
// 命名参数必须有"单个字符"[A-Za-z09]组成
 
// ?可选参数
{ path: '/optional-params/:foo?' }
// 路由跳转是可以设置或者不设置foo参数，可选
<router-link to="/optional-params">/optional-params</router-link>
<router-link to="/optional-params/foo">/optional-params/foo</router-link>
 
// 零个或多个参数
{ path: '/optional-params/*' }
<router-link to="/number">没有参数</router-link>
<router-link to="/number/foo000">一个参数</router-link>
<router-link to="/number/foo111/fff222">多个参数</router-link>
 
 
// 一个或多个参数
{ path: '/optional-params/:foo+' }
<router-link to="/number/foo">一个参数</router-link>
<router-link to="/number/foo/foo111/fff222">多个参数</router-link>
 
// 自定义匹配参数
// 可以为所有参数提供一个自定义的regexp，它将覆盖默认值（[^\/]+）
{ path: '/optional-params/:id(\\d+)' }
{ path: '/optional-params/(foo/)?bar' }
```

4. 匹配优先级
--------

有时候一个路径可能匹配多个路由。

此时，匹配的优先级就是按照路由的定义顺序：先定义，优先级最高。

5. push 和 replace 的第二个第三个参数
---------------------------

在 2.2.0 + 版本，可选的在 router.push 或 router.replace 中提供 onComplete 和 onAbort 回调作为第二个和第三个参数。

这些回调将会在导航成功完成 (在所有的异步钩子被解析之后) 或终止 (导航到相同的路由、或在当前导航完成之前导航到另一个不同的路由) 的时候进行相应的调用。在 3.1.0+，可以省略第二个和第三个参数，此时如果支持 Promise，router.push 或 router.replace 将返回一个 Promise。

接下来看几个例子来看看第二个第三个参数的调用时机：

#### 1. 组件 1 跳转组件 2

公众号

点赞和在看就是最大的支持❤️