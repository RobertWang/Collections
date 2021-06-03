> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/8401582c0ef1)

同域下不同站点的 SSO（跨站点）
=================

两个站点如果在同域下，那么它们之间是可以共享 cookie 的。简单的说就是这种同域下不同站点的 sso 实现可以通过 cookie 来实现，当用户访问这个域下面的任意站点时，浏览器都会将这个 cookie 发送给站点对应的系统。  
举个简单的例子：  
站点 1:[www.ssotest.com/site1](https://link.jianshu.com?t=http%3A%2F%2Fwww.ssotest.com%2Fsite1)  
站点 2:[www.ssotest.com/site2](https://link.jianshu.com?t=http%3A%2F%2Fwww.ssotest.com%2Fsite2)  
从上面可以看出，这两个站点是在相同域名（[www.ssotest.com](https://link.jianshu.com?t=http%3A%2F%2Fwww.ssotest.com)）下，那么从站点 1 登录时，会在浏览器存储一些 cookie，当在站点 1 下做任何操作时都会将这些 cookie 发送给站点 1，同理，当访问站点 2 时，由于站点 2 和站点 1 在同一个域名下面，所以也会将这些 cookie 发送给站点 2，所以这样的话就能够实现 SSO 了。可参考图 1:  

![](http://upload-images.jianshu.io/upload_images/10295193-e9c46afcbb0f3f03.png) 图 1 利用 cookie 实现同域不同站点单点登录图

同域但不同子域名的 SSO（跨子域）
==================

指相同父域，但是不同二级域名的单点登录，例如如下有两个站点：

```
subsite1.ssotest.com
subsite2.ssotest.com


```

从上面可以看出，它们的父域都是. ssotest.com，但是它们的二级域名不相同，所有如果在 subsite1 站点直接创建 cookie，那么 subsite2 站点是不能够获取到 subsite1 的 cookie 的，所以如果直接在 subsite1 站点登录，将无法实现单点登录，那么如何才能实现这种情况的 SSO 呢？  
实现它们之间的 SSO 最关键的一点是：实现 cookie 共享和 session 共享，保持二者的 sessionId 相同。  
解决方案如下：  
1. 如果是使用的 spring boot 框架的后台应用，只需要在 application.properties 配置文件中添加上如下配置即可（因为内嵌 tomcat）：

```
server.session.cookie.path=/
server.session.cookie.domain=.ssotest.com
注意：server.session.cookie.domain不能设置为公共后缀，比如：.com，.cn


```

2. 如果使用外部 tomcat，那么需要在 server.xml 文件中添加如下配置：

```
<Context path="指定访问该web应用的url入口" docBase="指定web应用的文件路径" reloadable="false" useHttpOnly="true" sessionCookiePath="/" sessionCookieDomain=".ssotest.com" />


```

不同域的 SSO（跨域）
============

要实现这种跨域方式的 SSO，有两种方式：  
1. 使用 cookie，在各个应用之间重定向  
2. 使用单独的 SSO 服务器（更优）  
那么下面详细的介绍上面这两种方式，以如下三个站点（应用）来具体描述：

```
www.site1.com
www.site2.com
www.site3.com


```

第一种方式：凭借 cookie，应用间的重定向
-----------------------

这种方式比较简单，当用户在上面三个站点中的任意一个站点登录成功时，必须在浏览器中同时设置其他站点的 cookie 信息。  
例如：当用户登录 site1 站点，并且验证通过之后，浏览器会存储一份 site1 站点的 cookie 信息，这时，为了实现单点登录（为了在 site2 站点和 site3 站点无需登录），那么我们需要在浏览器设置 site2 站点和 site3 站点的 cookie 信息，因此，在用户登录 site1 站点的请求响应之前，需要从 siteId1 站点重定向到 site2 站点和 site3 站点去设置 cookie 信息，这样就可以保证，在任意站点登录成功之后，在浏览器也有其他站点的 cookie 信息。下图 2 可具体展示其中流程：

![](http://upload-images.jianshu.io/upload_images/10295193-3497fdefb1367d98.png) 图 2 cookie 结合重定向实现跨域 SSO

这种方式其实过程比较简单，只需要确保登录其中一个站点在浏览器设置 cookie 其他站点都在浏览器设置对应 cookie，就可以实现单点登录了（单点退出是一样的道理，一个退出清除 cookie，其他也清除）。  
但是这种方式有一个非常明显的缺点是：这里举例是 3 个站点，如果是几十个上百个站点再使用这种方式将非常影响效率。

第二种方式：借助单独的 SSO 服务器
-------------------

这种方式需要借助一个单独的 SSOServer，相对于上一种方式，这种方式就不需要将每个站点的 cookie 信息都保存在浏览器上，浏览器只需要保存 SSOServer 的 cookie 信息。将这个 cookie 信息用于需要做单点登录的所有站点中。  
对于这种方式，在浏览器对于任意一个站点的请求都将会先重定向到 SSOServer 去验证代表当前用户的 cookie 是否存在，如果存在，那么将验证成功后的跳转页面发送给浏览器，否则将跳转到登录页面提示用户登录。可参考如下图 3 模型：

![](http://upload-images.jianshu.io/upload_images/10295193-9763fdea6a1d8ed9.png) 图 3 借助 SSOServer 实现单点登录模型

由于 site1 和 site2 的单点登录与 site1、site2、site3 之间的单点登录是同样道理的（因为登录 site1 后去访问 site2 和 site3 的流程都是相同的），所以，这里借助 site1 和 site2 的单点登录来说明这种方式。  
下面分为三部分（三张图）来说明这种方式的单点登录：  
第一部分：未在 SSOServer 登录，浏览器请求 site1 需要验证的页面，在 SSOServer 获取不到 cookie，被重定向到 site1 登录界面，提示登录，流程如下图 4：

![](http://upload-images.jianshu.io/upload_images/10295193-73fc65dbd9af8515.png) 图 4 SSOServer 未登录拦截请求流程图

第二部分：site1 的 SSO 的登录以及响应流程，如下图 5：

![](http://upload-images.jianshu.io/upload_images/10295193-020248c9c5a7ae4c.png) 图 5 site1 的 SSO 的登录以及响应流程

第三部分：site2 的 SSO 的登录以及响应流程，由于此时 SSOServer 已经登录了，所以不需要再次去 SSOServer 登录，只需要根据 cookie 去拿到 token，去验证 token 有效性，如下图 6：

![](http://upload-images.jianshu.io/upload_images/10295193-194560cc312d183e.png) 图 6 SSOServer 登录之后的其他站点登录流程图

在 SSOServer 登录的情况下，其他所有站点的登录情况都如 site2，图 6 所示的流程，可见使用 SSOServer 的方式会方便许多。  
以上就是 SSO 三种情况的实现方式。