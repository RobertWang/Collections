> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/62245660)

如何利用好微信生态的账号体系？

首先我们需要明确几个名词，主体、微信开放平台帐号、微信公众平台、UnionID、OpenID

词条
--

### 主体

微信生态下的主体常指公司主体，维基百科没有这个此条，根据[微信的说法](https://link.zhihu.com/?target=https%3A//kf.qq.com/faq/180110bYRvYZ180110V36j2U.html)则是 “由相关主管部门证明的合法机构”，那么通俗点就是在工商局登记在册的公司。例如 QQ、微信背后的公司主体是 “深圳市腾讯计算机系统有限公司”，抖音、火山小视频、Tik Tok 背后的主体是 “北京微播视界科技有限公司”。当然，除了公司，主体还包括事业单位、民办非企业单位等等。微信进行认证时，只需要提供[主体证件](https://link.zhihu.com/?target=https%3A//kf.qq.com/faq/180110bYRvYZ180110V36j2U.html)即可。

### 微信开放平台

微信开放平台为接入平台的应用模块提供开放接口以及技术支持服务，面向对象包括移动应用（iOS、Android、WP）、网站应用、公众号、小程序以及第三方平台。同一个开放平台账号可以绑定多个应用，同一开放平台账号下的不同应用，对同一用户而言他的 `UnionID` 是唯一的。

同一个开放平台账号[绑定应用限制](https://link.zhihu.com/?target=http%3A//kf.qq.com/faq/180104uY7r2a180104ziqumu.html)：

1.  移动应用
2.  已认证（组织类型）帐号可绑定最多 50 个移动应用。
3.  未认证（个人类型）帐号可绑定最多 10 个移动应用。

2. 网站应用 - 一个帐号可申请最多 10 个网站应用。

3. 公众号 - 已认证（组织类型）帐号可绑定最多 50 个同主体公众号、5 个不同主体公众号及 5 个公众号测试号，一个月最多新增绑定 5 个不同主体的公众号。 - 未认证（个人类型）帐号不支持绑定公众号及公众号测试号。

4. 小程序 - 已认证（组织类型）帐号可绑定最多 50 个同主体小程序、5 个不同主体小程序，一个月最多新增绑定 5 个不同主体的小程序。 - 未认证（个人类型）帐号帐号不支持绑定小程序。

> 备注：以上 “同主体” 指的是：公众号 / 小程序的主体信息与开放平台主体信息相同；“不同主体”指的是公众号 / 小程序的主体信息与开放平台主体信息不相同。  

### 微信公众平台

公众平台的内容就是我们日常接触到的公众号（订阅号、服务号、企业号）

[公众号注册限制](https://link.zhihu.com/?target=http%3A//kf.qq.com/faq/120911VrYVrA140428naUJVv.html)： - 同一个邮箱只能申请 1 个公众号； - 同一个手机号码可绑定 5 个公众号； - 同一身份证注册个人类型公众号数量上限为 1 个； - 同一企业、个体工商户、其他组织资料注册公众号数量上限为 2 个； - 同一政府、媒体类型可注册和认证 50 个公众号。

### OpenID

每个用户在使用不同应用时（公众号、小程序、移动应用、网站等），微信会为每个用户针对不同的应用生成一份 `OpenID`

### UnionID

类似于 `OpenID` 与应用关联，`UnionID` 与开放平台账号关联，同一用户，对同一个微信开放平台下的不同应用，`UnionID` 是相同且唯一的。

获取 OpenID
---------

`OpenID` 的获取方式是静默的，无需用户授权 客户端通过 `wx.login()` 拿到临时登录凭证 code，并且回传到开发者服务器，服务端带着 `code + appid + appsecret` 调用 `code2Session` 接口换取用户 `OpenID` 以及其他数据

[授权流程](https://link.zhihu.com/?target=https%3A//developers.weixin.qq.com/miniprogram/dev/framework/open-ability/login.html)如图：

![](https://pic4.zhimg.com/v2-f9bd8beb60145c4a2c8abc96b167eabb_b.jpg)

获取 UnionID
----------

不同于 `OpenID` 的静默获取方式，获取 `UnionID` 时，必须拿到用户授权，授权方式包括同意获取用户基础信息、关注公众号、授权登录移动应用、授权登录网站等，在拿到用户授权后，开发者通过对应接口拿到用户的 `UnionID`

[UnionID 获取接口](https://link.zhihu.com/?target=https%3A//developers.weixin.qq.com/miniprogram/dev/framework/open-ability/union-id.html)： - 用户已经授权，直接调用 `wx.getUserInfo`，从解密数据中获取 `UnionID`。 - 同一开放平台账号下其他应用已经拿到过授权（**用户关注了公众号、用户已经授权登录过公众号或移动应用等**），可以直接通过 `wx.login + code2Session` 获取到该用户 `UnionID`。(云函数中通过 `cloud.getWXContext` ) - 用户在小程序中支付完成后，可以直接通过 `getPaidUnionId` 接口获取该用户的 `UnionID`，无需用户授权。(注意：接口仅在用户支付完成后的 5 分钟内有效。)

小程序通过 `wx.getSetting` 接口可以获取**小程序已经向用户请求过的权限列表** - 未请求过的权限，除 `userInfo` 均可通过[接口](https://link.zhihu.com/?target=https%3A//developers.weixin.qq.com/miniprogram/dev/framework/open-ability/authorize.html)向用户发起授权请求，而 `userInfo` 则必须使用 `<button open-type="getUserInfo"/>` 方式，由用户点击按钮才能发起。 - 请求过的权限，当用户拒绝权限申请时，如果产品逻辑被阻塞，可调用 `wx.oepnSetting` 接口打开小程序设置界面，引导用户开启权限，当然，`userInfo` 权限被拒绝后仍可以通过 `<button open-type="getUserInfo"/>` 方式继续向用户发起权限请求。

![](https://pic2.zhimg.com/v2-7358da079fc70f5d002b88160f12416d_b.jpg)

UnionID 与内部数据打通
---------------

在获得用户授权，拿到 `UnionID` 后，可以通过验证用户手机号，将 `UnionID` 作为用户信息的一部分，给用户带来的便利是不用每次验证手机号，同时也给公司省了一笔话务的费用，一次打通，多端复用，APP、小程序、H5 活动，可以利用微信生态将所有场景串联。

更重要的是以此积累的数据，通过多端积累的海量 `UnionID`，可以对用户行为数据进行详细分析，进行精准推送，这种数据带来的价值不可估量。

参考资料
----

*   [微信公众平台](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/wiki%3Ft%3Dresource/res_main%26id%3Dmp1445241432)
*   [腾讯客服 - 开放平台](https://link.zhihu.com/?target=http%3A//kf.qq.com/product/wxkfpt.html%23hid%3D2568)
*   [腾讯客服 - 公众平台](https://link.zhihu.com/?target=http%3A//kf.qq.com/faq/120911VrYVrA140428naUJVv.html)
*   [用户信息](https://link.zhihu.com/?target=https%3A//developers.weixin.qq.com/miniprogram/dev/framework/open-ability/login.html)
*   [授权](https://link.zhihu.com/?target=https%3A//developers.weixin.qq.com/miniprogram/dev/framework/open-ability/authorize.html)
*   [AuthSetting](https://link.zhihu.com/?target=https%3A//developers.weixin.qq.com/miniprogram/dev/api/AuthSetting.html)