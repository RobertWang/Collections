> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/myshowtime/p/15596630.html)

![](https://blog-1259586045.cos.ap-shanghai.myqcloud.com/202111213432423432.png)

OAuth 2.1 是 OAuth 2.0 的下一个版本, OAuth 2.1 根据最佳安全实践 (BCP), 目前是第 18 个版本，对 OAuth 2.0 协议进行整合和精简, 移除不安全的授权流程, 并发布了 OAuth 2.1 规范草案, 下面列出了和 OAuth 2.0 相比的主要区别。

⚡ 推荐使用 Authorization Code + PKCE
--------------------------------

[根据 OAuth 2.0 安全最佳实践 (Security Best Current Practices) 2.1.1 章节](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics-18#section-2.1.1)

授权码 (Authorization Code) 模式大家都很熟悉了, 也是最安全的授权流程, 那 PKCE 又是什么呢? PKCE 全称是 Proof Key for Code Exchange, 在 2015 年发布为 RFC 7636, 我们知道, 授权码模式虽好, 但是它不能给公开的客户端用, 因为公开的客户端没有能力保存好秘钥 (client_secret), 所以在此之前, 对于公开的客户端, 只能使用隐式模式和密码模式, PKCE 就是为了解决这个问题而出现的, 另外它也可以防范授权码拦截攻击, 实际上它的原理是客户端提供一个自创建的证明给授权服务器， 授权服务器通过它来验证客户端，把访问令牌 (access_token) 颁发给真实的客户端而不是伪造的，下边是 Authorization Code + PKCE 的授权流程图。

![](https://blog-1259586045.cos.ap-shanghai.myqcloud.com/clipboard_20211113_103034.png)

⚡隐式授权（ Implicit Grant）已弃用
-------------------------

[根据 OAuth 2.0 安全最佳实践 (Security Best Current Practices) 2.1.2 章节](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics-18#section-2.1.2)

在 OAuth 2.1 规范草案中, 授权模式中已经找不到隐式授权 (Implicit Grant), 我们知道, 隐式授权是 OAuth 2.0 中的授权模式, 是授权码模式的简化版本, 用户同意授权后, 直接就能返回访问令牌 access_token, 同时这种也是不安全的。

![](https://blog-1259586045.cos.ap-shanghai.myqcloud.com/clipboard_20211101_054548.png)

现在您可以考虑替换为 Authorization Code + PKCE 的授权模式。

⚡ 密码授权 （Resource Owner Password Credentials Grant）已弃用
-----------------------------------------------------

[根据 OAuth 2.0 安全最佳实践 (Security Best Current Practices) 2.4 章节](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics-18#section-2.4)

在 OAuth 2.1 规范草案中, 密码授权也被移除, 实际上这种授权模式在 OAuth 2.0 中都是不推荐使用的, 密码授权的流程是, 用户把账号密码告诉客户端, 然后客户端再去申请访问令牌, 这种模式只在用户和客户端高度信任的情况下才使用。

试想一下, 在你手机上有一个网易云音乐的 APP, 现在要使用 qq 账号登录, 这时网易云音乐说, 你把 qq 账号密码告诉我就行了, 我拿着你的账号密码去 QQ 那边登录, 这就很离谱了!

正确的做法是, 用户在网易云音乐要使用 qq 登录, 如果用户也安装了 qq 的客户端, 应该唤起 qq 应用, 在 qq 页面完成授权操作, 然后返回到网易云音乐。如果用户没有安装 qq 客户端应用, 唤起浏览器, 引导用户去 qq 的授权页面, 用户授权完成后, 返回到网易云音乐。

请注意, OAuth 是专门为委托授权而设计的，为了让第三方应用使用授权, 它不是为身份验证而设计的, 而 OpenID Connect（建立在 OAuth 之上）是专为身份验证而设计, 所以, 在使用 OAuth 授权协议时, 你需要知道你使用的客户端是第三方应用程序还是第一方应用，这很重要！因为 OAuth 2.1 已经不支持第一方应用授权！

现在您可以考虑使用 Authorization Code + PKCE 替换之前的密码授权模式。

⚡ 使用 access_token 时, 不应该通过 URL 传递 token
---------------------------------------

[根据 OAuth 2.0 安全最佳实践 (Security Best Current Practices) 4.3.2 章节](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics-18#section-4.3.2)

在使用 access_token 时, 您不应该把 token 放到 URL 中, 第一, 浏览器地址栏本来就是暴露的, 第二, 可以查看浏览记录，找到 access_token。

正确的做法是, 把 access_token 放到 Http header 或者是 POST body 中。

⚡ 刷新令牌 (Refresh Token) 应该是一次性的
------------------------------

[根据 OAuth 2.0 安全最佳实践 (Security Best Current Practices) 4.13.2 章节](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics-18#section-4.13.2)

access_token 访问令牌, refresh_token 刷新令牌, 刷新令牌可以在一段时间内获取访问令牌, 平衡了用户体验和安全性, 在 OAuth 2.1 中, refresh_token 应该是一次性的, 用过后失效, 使用 refresh_token 获取 access_token 时, 还可以返回一个新的 refresh_token。

![](https://blog-1259586045.cos.ap-shanghai.myqcloud.com/clipboard_20211121_033808.png)

⚡ 回调地址（Redirect URI）应该精确匹配
--------------------------

[根据 OAuth 2.0 安全最佳实践 (Security Best Current Practices) 4.1.3 章节](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics-18#section-4.1.3)

在 OAuth 2.0 的授权码流程中, 需要设置一个回调地址 redirect_uri, 如下

```
https://www.authorization-server.com/oauth2/authorize?
response_type=code
&client_id=s6BhdRkqt3
&scope=user
&state=8b815ab1d177f5c8e 
&redirect_uri=https://www.client.com/callback
```

假如有三个不同的客户端

*   a.client.com
*   b.client.com
*   c.client.com

这时可能会使用一个通配符的 redirect_uri, 比如 *.client.com, 这样会有什么风险呢? 实际上, 恶意程序有机会篡改 redirect_uri, 假设恶意程序的域名是 [https://attacker.com](https://attacker.com), 然后把 redirect_uri 改成 [https://attacker.com/.client.com](https://attacker.com/.client.com), 这样授权码就发送给了恶意程序。

References
----------

[https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-04](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-04)

[OAuth 2.0 Security Best Current Practice draft-ietf-oauth-security-topics-18](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics-18)

[https://fusionauth.io/learn/expert-advice/oauth/differences-between-oauth-2-oauth-2-1](https://fusionauth.io/learn/expert-advice/oauth/differences-between-oauth-2-oauth-2-1/)

![](https://blog-1259586045.cos.ap-shanghai.myqcloud.com/wechat_logo_s1.png)