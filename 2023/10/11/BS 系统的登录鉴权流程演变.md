> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/jann8/p/17744851.html)

1 基础知识
------

用户登录是使用指定用户名和密码登录到系统，以对用户的私密数据进行访问和操作。在一个有登录鉴权的 BS 系统中，通常用户访问数据时，后端拦截请求，对用户进行鉴权，以验证用户身份和权限。用户名、密码等身份信息只需要在登录时输入一次，然后通过前后端的配合，在之后的每次访问都不用再输入了，通常的方案是将身份标识存在 cookie 中。

实际的登录方案通常较为复杂。一方面需要了解系统的整体架构，包括前端的架构，然后按需设计不同的登录方案；二是需要考虑安全漏洞。我接触过几个系统，从简单的系统到复杂的，在这里把它们的登录方案介绍一下。

在介绍具体的登录方案前，先介绍下登录相关的基础知识。

### 1.1 Http Cookie

Cookie 是由 Web 服务器向 Web 浏览器发送的一小段字符串，此后的所有浏览器对服务端的访问都会携带这个字符串。Cookie 由 Netscape 发明，它使得保持 HTTP 请求的状态（Http 协议是无状态协议）变得容易，服务端可以向 Cookie 中存入任意的信息。Cookie 最常见用于已登录用户的鉴权，用户不用每次请求访问时都在页面进行登录。Cookie 也有其它用途，比如用于存储购物车列表 [3]。

Cookie 的使用是通过 Http 头 set-cookie 和 cookie 实现的。在接收到 Http 请求后，服务端可以向浏览器发送一个或多个 Set-Cookie 应答头。浏览器会自动存储 cookie 并在此后的浏览器对服务端的请求中携带 Cookie 请求头 [1]。

服务端向浏览器发送的应答头：

```
HTTP/2.0 200 OK
Content-Type: text/html
Set-Cookie: yummy_cookie=choco
Set-Cookie: tasty_cookie=strawberry


```

浏览器自动将 cookie 保存，在此后每次浏览器向服务的请求都会自动携带该 cookie：

```
GET /sample_page.html HTTP/2.0
Host: www.example.org
Cookie: yummy_cookie=choco; tasty_cookie=strawberry


```

服务端向浏览器发送应答头的 java 代码实现如 Demo1。你可以为 cookie 设置存活时间，指定 Cookie 的域名和路径。Cookie 的默认存活时间为浏览器会话结束; 设置存活时间后，会话结束不会影响 cookie 的存活；浏览器会话结束的场景比如关闭浏览器窗口。Cookie 的默认所属路径是 “/ 项目路径 / 相对路径” 的上一层路径；当请求路径为 Cookie 所属的路径及其子路径时，才会携带 cookie。还可以为 Cookie 设置 Same-Site、HttpOnly 等属性 [2]，它们与 Cookie 使用的安全性有关。

```
public class CookieTestServlet extends HttpServlet {
	protected void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		// 创建cookie对象
		Cookie yummy = new Cookie("yummy_cookie", "choco");
		Cookie tasty = new Cookie("tasty_cookie", "strawberry");

		//默认存活时间为浏览器会话结束;设置存活时间后，会话结束不会影响cookie的存活；浏览器会话结束的场景比如关闭浏览器窗口
		tasty.setMaxAge(3600);

		//默认路径是“/项目路径/相对路径”的上一层路径；当请求路径为设置的路径及其子路径时，才会携带cookie；
		tasty.setPath("/test");

		//不设置Domain时，Domain的默认值为当前请求的域名(比如localhost、www.example.org等)

		//将cookie返回给浏览器，通过应答头set-cookie
		response.addCookie(yummy);
		response.addCookie(tasty);
	}
}


```

Demo1 创建 Cookie 并设置其常用属性的 java 代码实现

在服务端设置了 Cookie 的存活时间和路径后，服务端向浏览器发送的应答头如下：

```
HTTP/2.0 200 OK
Content-Type: text/html
Set-Cookie:tasty_cookie=strawberry; Max-Age=3600; Expires=Thu, 21-Sep-2023 08:33:24 GMT; Path=/test
Set-Cookie:yummy_cookie=choco


```

### 1.2 重定向与前端路由 Vue-router

#### 1.2.1 后端重定向 [4]

早期的系统是前后端不分离的。一个典型的系统使用 SSM+JSP 的架构。未登录的用户访问系统页面时，会通过后端重定向到登录页，如 Demo2。登录时通常将用户名、密码通过 form 表单提交到后台，后台登录校验不通过时，也会重定向到登录页。

```
//权限过滤器
public class PermissionFilter implements Filter {
    public void doFilter(ServletRequest _request, ServletResponse _response,
			FilterChain chain) throws IOException, ServletException {
        //如果未登录，重定向到登录页
        if (!checkLogin(request, response)){
            response.sendRedirect(request.getContextPath() + "/login.jsp")
        }
        chain.doFilter(request, response);
    }
}


```

Demo2 权限过滤器的简单实现

#### 1.2.2 Vue-router

在系统的前后端分离后，一个比较常见 Web 系统，前端使用 Vue+Nodejs，后端使用 SpringBoot+Spring+Mybatis。在用户认证鉴权业务中，前端应用可独立地提供页面的访问和实现页面间的跳转，后端实现用户认证鉴权的逻辑并提供接口。用户未登录访问系统页面时，页面跳转通常是通过 Vue-router 实现的。如 Demo3，在 router 的全局前置守卫 (router.berforeEach) 中，判断用户是否已登录，如果未登录，则跳转（通过路由导航）到登录页。

```
router.beforeEach((to, from, next) => {
	//判断是否已登录
    if (getToken()) {
        next()  //进入管道中的下一个钩子
    }else{
        next(`/login`) //跳转到登录页
    }
}


```

Demo3 使用 Vue-router.beforeEach 进行用户是否登录的判断

对于大多数单页应用，Vue 都推荐使用官方的 Vue-router。这是由于前端应用的业务功能越来越复杂，单页应用（SPA）成为前端应用的主流形式。Vue-router 通过管理 URL，实现 URL 和组件的对应，以及通过 URL 进行组件之间的切换。可以参考相关博客中的案例，进行 Vue-router 的安装和简单使用 [5]。使用 Vue-router，通过改变 URL，在不重新请求页面的情况下，就可以更新页面视图。“更新视图但不重新请求页面” 是前端路由原理的核心之一，目前在浏览器环境中这一功能的实现主要有两种方式 [6]:

*   利用 URL 中的 hash（“#”）
*   利用 History interface 在 HTML5 中新增的方法。

如 Demo4，在 vue-router 中是通过 mode 这一参数控制路由的实现模式的，mode 值为 “hash“表示第一种方式，值为”history“表示第二种方式。可以使用 router.beforeEach 注册一个全局前置守卫。当一个路由导航触发时，全局前置守卫按照创建顺序调用。每个守卫方法接收三个参数 [16]：

*   to: Route: 即将要进入的目标 路由对象
*   from: Route: 当前导航正要离开的路由
*   next: Function: 一定要调用该方法来 resolve 这个钩子。执行效果依赖 next 方法的调用参数。

1.  next(): 进行管道中的下一个钩子。如果全部钩子执行完了，则导航的状态就是 confirmed (确认的)。
2.  next(false): 中断当前的导航。如果浏览器的 URL 改变了 (可能是用户手动或者浏览器后退按钮)，那么 URL 地址会重置到 from 路由对应的地址。
3.  next('/') 或者 next({ path: '/' }): 跳转到一个不同的地址。当前的导航被中断，然后进行一个新的导航。你可以向 next 传递任意位置对象，且允许设置诸如 replace: true、name: 'home' 之类的选项以及任何用在 router-link 的 to prop 或 router.push 中的选项。
4.  next(error): (2.4.0+) 如果传入 next 的参数是一个 Error 实例，则导航会被终止且该错误会被传递给 router.onError() 注册过的回调。

确保要调用 next 方法，否则钩子就不会被 resolved。

```
export default new Router({
  // mode: 'hash',
  mode: 'history',
  scrollBehavior: () => ({ y: 0 }),
  routes: constantRouterMap
})


```

Demo4 创建 Vue-router 时指定 mode 为 history

### 1.3 JWT

#### 1.3.1 JWT 简介 [7]

JSON Web Token (JWT) 是一个开放的标准（RFC 7519[8]）。它定义了严谨且独立的方式，在多方服务间以 JSON 对象的格式安全地传递信息。数字签名使传递的信息可验证可信任。JWT 签名方式有使用一个 secret 的 HMAC 算法和使用公钥 / 私钥的 RSA 或 ECDSA 算法等。

JWT 包含 head、payload 和 signature 三个部分。比如 signature 的签名方式使用的是公钥 / 私钥的 RSA 算法。payload 中一般包含用户名和权限等信息，可以直接从 token 中获取这些信息。token 还有一些其它特性，signature 使用公钥解密后的值与 head 和 playload 的值做对比是否一致，可以判断 token 是否被篡改。从系统层面讲，只有对 token 的签名方拥有私钥。

JWT 是基于 token 鉴权标准之一，常用的标准还有 OAuth[9]。token 是身份验证过程中用到的令牌，他是验证用户身份和资源权限的临时密钥。一个有效的 token 允许用户对在线的服务和 web 应用进行访问直至 token 过期。这提供了便利，用户不用每次都重新进行登录认证，就可以继续访问资源。这与 cookie 中的 sessionId 有类似之处。

#### 1.3.2 JWT 的构成

JSON Web Token(JWT)包含 Header、Payload 和 Signature3 个部分，3 个部分以点号 (.) 隔开，他的格式可以表示为 xxxxx.yyyyy.zzzzz。

##### 1.3.2.1 Header

Header 通常包含 2 部分，一是 token 的类型，比如 JWT 或 OAuth; 二是签名所用的算法，比如 HMAC SHA256 或 RSA。以使用公钥私钥加密解密的 RSA 算法为例，它的 Header 内容如下：

```
{
"alg": "RS256", //使用RSA签名算法
"typ": "JWT"//使用JWT类型的token
}


```

Header 的 json 内容使用 Base64Url 编码后构成 JWT 的第一部分。

##### 1.4.2.2 Payload

Token 的第二部分是 payload，在 payload 中包含 claims。claims 是对实体信息（比如用户）的陈述以及其它数据。claims 分为 registered、public 和 private 等 3 种类型。

*   Registered claims：指一些预定义的 claims，这些 claims 不是强制的但推荐使用，它们实用性强、交互性好。比如 iss(issuer)、exp(expiration time)、sub(subject) 和 aud(audience) 等。Registered claims 的名称都是 3 个字符的，很紧凑。
*   Public claims：可由 JWTs 的使用者自定义。但为了防止命名冲突，这些 claims 应在 IANA JSON WebToken Registry[10] 中定义，或者定义为包含防冲突命名空间的 URI(Uniform Resource Identifier)。
*   Private claims：用户创建的 claims，用于在约定好的多方服务间交换信息。它们既不是 Registerd cliams, 也不是 Public claims。

使用 RS256 签名的 Payload 如下：

```
{
"sub": "RS256InOTA",
"name": "John Doe"
}


```

Payload 的 json 内容使用 Base64Url 编码后构成 JWT 的第二部分。

##### 1.4.2.3 Signature

将 Base64Url 编码后的 Header、Payload 和私钥，使用 header 中的算法 (比如 RS256) 进行处理后，得到 Signature。比如，使用 sha256 进行加密的 Signature 的创建方式如下, 其中私钥是自动生成的 [11]：

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  jwtRSA256-private.pem)


```

Signature 构成 JWT 的第三部分。

#### 1.4.3 JWT 在 WEB 开发中的使用

在 WEB 项目中，基于 JWT 进行鉴权时，通常将 token 以请求头 Authorization 的方式发送。

Authorization 的格式如下：

```
HTTP
Authorization: <type> <credentials>


```

HTTP 提供了一个访问控制和身份验证的框架（见 RFC 7235[12]）。它可以用于服务端质询客户端请求，以及客户端向服务端提供用户认证的信息。Authorization 是该框架中的一个请求头，用于浏览器向服务端提供用户认证的信息。该框架定义了用户认证的多种机制，其中包含 “Bearer” 这种机制 [13]。Bearer(见 RFC 6750[14]) 机制的官方定义是使用 bearer tokens 获取 OAuth 2.0 保护的资源。JWT 与 OAuth 是 token 生成的 2 种不同方式。使用 JWT 的 web 项目 Authorization 的格式如下：

```
HTTP
Authorization: Bearer xxxxx.yyyyy.zzzzz


```

token 放在请求头 Authorization 中，而不放在 cookie 中，主要是第三方服务的 cookie 已经被很多浏览器禁用，要支持第三方服务的 cookie 只能使用 Authorization。Authorization 和 cookie 的区别主要涉及安全性层面，这里不详述。

查看 JWT 解析的源码 [15]，JWT 解析时会经过一系列步骤，其中包含以下步骤：

*   检查 token 是否被篡改。将 token 中的 signature 使用公钥进行解密，与 Header、payload 做比对是否一致，如果不一致说明被篡改，直接抛出异常；
*   检查 token 的 payload 中的过期时间（exp），如果 token 已经过期，直接抛出异常。

### 1.4 认证、鉴权和授权的含义

#### 1.4.1 认证（Authentication）[17]

*   当服务端需要知道谁在访问它们的信息或网站时，会对用户进行认证；
*   在认证中，用户需要向服务端证明自己的身份；
*   通常，服务端认证需要使用用户名和密码。其它的身份验证方式包括（银行）卡、视网膜扫描、语言识别和指纹等；
*   认证不决定用户可以访问哪些资源。认证仅仅识别和验证用户是谁。

#### 1.4.2 授权（Authorization)

##### 1.4.2.1 系统对用户授权

服务端通过授权这个过程决定用户是否有访问资源的权限 [17]。用户授权确保用户在访问敏感数据前拥有适当的权限，敏感数据包括个人信息、安全数据库和私密数据等。通常授权与访问控制（access control）或客户端特权（client priviledge）可互换 [18]。

不要将授权和认证混淆，认证和授权是管理员保护系统和信息的两个至关重要的过程。授权和认证通常成对出现，服务端需要先认证以确认用户身份。授权通常分解为以下步骤 [18]：

*   身份识别：在赋予任何权限前，系统需要识别用户的身份。通常通过用户名、邮件名或其它唯一标识。
*   认证：在识别用户身份后，通常通过输入密码、生物扫描或多重身份验证的方法对用户进行认证。
*   分配权限：在认证成功后，系统获取用户所拥有的权限和角色。
*   确保只有授权用户可以访问：根据获取的用户权限和角色，系统决定用户可以访问哪些资源或函数。常用方法有检查访问控制列表（Access Control Lists，简称 ACLs）、基于角色的权限或其它的授权方法；
*   审计和监控：系统持续的输出日志并监控用户的行为。这帮助识别未授权和可疑的行为，同时也可以周期性回顾用户的权限，以确保符合他们的角色和职责。
*   会话终止：在会话到期或用户登出后，会话会终止，确保会话终止后出现未授权的访问。

有些情况下，没有授权这个过程，任意用户都通过请求来使用资源或访问文件。大多数的 Web 页面都是不需要认证和授权的。

##### 1.4.2.2 其它含义 [21]

在信息安全领域，授权是指资源所有者委派执行者，赋予执行者指定范围的资源操作权限，以便执行者代理执行对资源的相关操作。授权的实现方式非常多也很广泛，我们常见的银行卡、门禁卡、钥匙、公证书，这些都是现实生活中授权的实现方式。

在互联网应用开发领域，授权所用到的授信媒介主要包括如下几种：

*   通过 web 服务器的 session 机制，一个访问会话保持着用户的授权信息
*   通过 web 浏览器的 cookie 机制，一个网站的 cookie 保持着用户的授权信息
*   颁发授权令牌（token），一个合法有效的令牌中保持着用户的授权信息

简单理解，授权指系统为用户颁发用户凭证的过程。

#### 1.4.2 鉴权

鉴权是通信行业中的术语 [19][20]。 鉴权含义之一是通过评估可应用的访问控制信息来确定是否允许某主体具有接入到特定资源所规定的类型的过程。通常情况下，鉴权在认证上下文中进行。一旦某主体被认证，它可以被授权执行不同类型的接入。简单理解是验证用户身份，判断是否有权限。有一篇文章赞同这个观点 [21]，而另一篇文章则没有引入鉴权这个概念，直接将鉴权的过程包含在 1.5.2.1 系统对用户授权中 [18]。

#### 1.4.3 本文用语约定

因为对于授权、鉴权的含义有不同看法，现对本文用语做以下约定：

1. 认证：用户第一次登录，服务端进行验证用户名密码，并应答鉴权凭据的过程；

2. 鉴权：用户每次访问接口时，对用户的鉴权凭据进行校验，并判断用户是否有权限访问该接口或资源；

3. 授权：根据鉴权结果，确保用户只能访问有权限的接口或资源。

4. 访问控制：包含鉴权和授权。

这样的约定可简化后文的表述。比如用户登录校验可表述为用户认证，用户访问数据时的访问控制也可表述为鉴权授权，SpringSecurity 安全框架可描述为包含用户认证（身份认证）、鉴权授权（访问控制）两个部分。一篇关于微服务架构的文章中有使用用户认证、鉴权授权的表述 [22]。

### 1.5 Spring-Security

Spring Security 框架提供了身份认证、权限控制和安全漏洞防护等功能。它在保护 spring 应用方面是实际上的标准，为保护命令式和反应式应用程序提供一流的支持。

#### 1.5.1 框架的结构

##### 1.5.1.1 过滤器链 [24]

Spring Security 的 servlet 实现是基于 servlet 过滤器的。如图 1 中的图①，当客户端向应用发送请求后，servlet 容器依据请求的 URL 创建 FilterChain，FilterChain 中包含 Filter 实例和处理 HttpServetRequest 的 Servlet。

Spring 提供了 DelegatingFilterProxy 这个过滤器，它将 Servlet 容器的生命周期和 Spring 中的 ApplicationContext 连接起来。Servlet 容器允许使用自己的标准注册 Filter 实例，但无法识别 Spring 中定义的 beans。你可以通过 Servlet 容器的机制注册 DelegatingFilterProxy 并将所有的工作委托给实现 Filter 接口的 Spring Bean。图 1 中的②展示了 DelegatingFilterProxy 与 FilterChain 和 Spring 的 Filter 实例的关系。DelegatingFilterProxy 从 ApplicationContext 中搜寻 Bean Filter0 并调用该 Bean Filter0，DelegatingFilterProxy 的伪代码实现如 Demo5。

```
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
	Filter delegate = getFilterBean(someBeanName); 
	delegate.doFilter(request, response); 
}


```

Demo5 DelegatingFilterProxy 将工作委托给 Spring Beans 的伪代码

Spring Security 对 Servlet 的支持都包含在 FilterChainProxy 中。FilterChainProxy 是一个由 Spring Security 提供的特殊的 Filter，它允许通过 SecurityFilterChain 将工作委托给许多 Filter 实例。FilterChainProxy 是一个 Bean，通常包装在 DelegatingFilterProxy 中。图 1 中的③展示了 FilterChainProxy 的角色。

FilterChainProxy 依据 SecurityFilterChain 决定哪个 Spring Security 过滤器实例在当前请求被调用。图 1 中的④展示了 SecurityFilterChain 的角色。SecurityFilterChain 中的 Security 过滤器都注册在 FilterChainProxy，FilterChainProxy 提供了所有 Spring Security 过滤器的入口。

图 1 中的⑤展示了多 SecurityFilterChain 实例的情况。如图，FilterChainProxy 决定哪个 SecurityFilterChain 被使用。第一个匹配到的 SecurityFilterChain 被调用。比如一个 URL 为 / api/message 的请求，它首先匹配到 SecurityFilterChain0 的模式 / api/**，尽管它也匹配 SecurityFilterChainn 的模式，但只有 SecurityFilterChain0 被调用。

[![](https://img2023.cnblogs.com/blog/1389306/202310/1389306-20231006184256125-1388035046.png)](https://img2023.cnblogs.com/blog/1389306/202310/1389306-20231006184256125-1388035046.png)

图 1 过滤器链的结构：①Servlet 容器中的过滤器链; ②Spring 中的 DelegatingFilterProxy; ③FilterChainProxy; ④SecurityFilterChain; ⑤多 SecurityFilterChain。

##### 1.5.1.2 DelegationFilterProxy 的实例化和拦截配置

DelagtingFilterProxy 的初始化和拦截配置在容器启动的时候就完成了 [23]。SpringServletContainerInitializer 是 ServletContainerInitializer 的实现类，且使用 @HandlesTypes 注解。当容器启动后，会调用 SpringServletContainerInitializer 的 onStartup 方法，收集 WebApplicationInitializer 的子类，并循环调用这些子类的 onStartup 方法。AbstractSecurityWebApplicationInitializer 是 WebApplicationInitializer 的子类，它的 onStartup 方法被调用，对 DelegationFilterProxy 进行实例化并配置拦截路径。图 2 展示了从容器启动到 DelegationFilterProxy 完成实例化和拦截配置的流程。

[![](https://img2023.cnblogs.com/blog/1389306/202310/1389306-20231006184329850-479152505.png)](https://img2023.cnblogs.com/blog/1389306/202310/1389306-20231006184329850-479152505.png)

图 2 从容器启动到 DelegationFilterProxy 完成实例化和拦截配置的流程

##### 1.5.1.3 过滤器日志

一个特定请求到达服务器，知道哪些过滤器会被调用是有帮助的，比如你想确认你添加的过滤器是否被调用。在应用启动时将应用的日志级别设置为 INFO, 你就可以在控制台看到如 Log1 的日志。

```
2023-06-14T08:55:22.321-03:00  INFO 76975 --- [           main] o.s.s.web.DefaultSecurityFilterChain     : Will secure any request with [
org.springframework.security.web.session.DisableEncodeUrlFilter@404db674,
org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@50f097b5,
org.springframework.security.web.context.SecurityContextHolderFilter@6fc6deb7,
org.springframework.security.web.header.HeaderWriterFilter@6f76c2cc,
org.springframework.security.web.csrf.CsrfFilter@c29fe36,
org.springframework.security.web.authentication.logout.LogoutFilter@ef60710,
org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter@7c2dfa2,
org.springframework.security.web.authentication.ui.DefaultLoginPageGeneratingFilter@4397a639,
org.springframework.security.web.authentication.ui.DefaultLogoutPageGeneratingFilter@7add838c,
org.springframework.security.web.authentication.www.BasicAuthenticationFilter@5cc9d3d0,
org.springframework.security.web.savedrequest.RequestCacheAwareFilter@7da39774,
org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@32b0876c,
org.springframework.security.web.authentication.AnonymousAuthenticationFilter@3662bdff,
org.springframework.security.web.access.ExceptionTranslationFilter@77681ce4,
org.springframework.security.web.access.intercept.AuthorizationFilter@169268a7]


```

Log1 当应用日志级别设置为 INFO 时，应用启动时应用日志记录了配置的所有过滤器

Spring Security 在 security 日志级别为 DEBUG 和 TRACE 级别时提供了 security 相关事件的全面记录。这对你调试应用很有帮助，Spring Security 为确保安全，当一个请求被拒绝时应答体并未包含错误信息。但你遇到 401 或 403 的错误时，日志信息将会帮助你定位问题。举个例子，当你发送 POST 请求获取有 CSRF 保护的资源，但并未携带 CSRF token。你将会看到 403 的错误且没有任何错误说明。可以通过如 Config1 的配置设置 Secuirty 的日志级别为 TRACE。配置后，出现如上的情况时你除了看到 403 的错误，还可以看到如 Log2 所示的日志。

```
#application.properties in Spring Boot
logging.level.org.springframework.security=TRACE


```

Config1 在 spring Boot 项目中配置 Security 的日志级别为 TRACE

```
2023-06-14T09:44:25.797-03:00 DEBUG 76975 --- [nio-8080-exec-1] o.s.security.web.FilterChainProxy        : Securing POST /hello
2023-06-14T09:44:25.797-03:00 TRACE 76975 --- [nio-8080-exec-1] o.s.security.web.FilterChainProxy        : Invoking DisableEncodeUrlFilter (1/15)
2023-06-14T09:44:25.798-03:00 TRACE 76975 --- [nio-8080-exec-1] o.s.security.web.FilterChainProxy        : Invoking WebAsyncManagerIntegrationFilter (2/15)
2023-06-14T09:44:25.800-03:00 TRACE 76975 --- [nio-8080-exec-1] o.s.security.web.FilterChainProxy        : Invoking SecurityContextHolderFilter (3/15)
2023-06-14T09:44:25.801-03:00 TRACE 76975 --- [nio-8080-exec-1] o.s.security.web.FilterChainProxy        : Invoking HeaderWriterFilter (4/15)
2023-06-14T09:44:25.802-03:00 TRACE 76975 --- [nio-8080-exec-1] o.s.security.web.FilterChainProxy        : Invoking CsrfFilter (5/15)
2023-06-14T09:44:25.814-03:00 DEBUG 76975 --- [nio-8080-exec-1] o.s.security.web.csrf.CsrfFilter         : Invalid CSRF token found for http://localhost:8080/hello
2023-06-14T09:44:25.814-03:00 DEBUG 76975 --- [nio-8080-exec-1] o.s.s.w.access.AccessDeniedHandlerImpl   : Responding with 403 status code
2023-06-14T09:44:25.814-03:00 TRACE 76975 --- [nio-8080-exec-1] o.s.s.w.header.writers.HstsHeaderWriter  : Not injecting HSTS header since it did not match request to [Is Secure]


```

Log2 Security 的日志级别设置为 DEBUG 或 TRACE 后，日志对 security 相关事件进行了全面记录

##### 1.5.1.4 过滤器配置

Security 过滤器通过 SecurityFilterChain 的 API 插入到 FilterChainProxy 中。这些过滤器有不同的用途，比如身份认证、访问控制和漏洞防护等。Demo6 展示了通过 SecurityFilterChain 配置了一系列过滤器。.csrf() 表示配置 csrfFilter，.authorizeHttpRequests() 表示配置 UsernamePasswordAuthenticationFilter，.httpBasic 表示配置 BasicAuthenticationFilter，.formLogin 表示配置 AuthorizationFilter。

如果未进行过滤器的配置，则会使用默认配置。在 springSecurityFilterChain 初始化时，判断当前是否配置了 webSecurityConfigurers，如果没有，则会生成一个默认的：new WebSecurityConfigurerAdapter()。[25]WebSecurityConfigurerAdapter 类的 init 方法中进行了过滤器的默认配置。默认配置相关代码如 Demo7。

```
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(Customizer.withDefaults())
            .authorizeHttpRequests(authorize -> authorize
                .anyRequest().authenticated()
            )
            .httpBasic(Customizer.withDefaults())
            .formLogin(Customizer.withDefaults());
        return http.build();
    }

}


```

Demo6 通过 SecurityFilterChain 配置 seucrity 中的过滤器链

```
public abstract class WebSecurityConfigurerAdapter implements
		WebSecurityConfigurer<WebSecurity> {
		
	public void init(final WebSecurity web) throws Exception {
		final HttpSecurity http = getHttp();
    }
    
    protected final HttpSecurity getHttp() throws Exception {
		if (!disableDefaults) {
			// @formatter:off
			http
				.csrf().and()
				.addFilter(new WebAsyncManagerIntegrationFilter())
				.exceptionHandling().and()
				.headers().and()
				.sessionManagement().and()
				.securityContext().and()
				.requestCache().and()
				.anonymous().and()
				.servletApi().and()
				.apply(new DefaultLoginPageConfigurer<>()).and()
				.logout();
			// @formatter:on
			ClassLoader classLoader = this.context.getClassLoader();
			List<AbstractHttpConfigurer> defaultHttpConfigurers =
					SpringFactoriesLoader.loadFactories(AbstractHttpConfigurer.class, classLoader);

			for (AbstractHttpConfigurer configurer : defaultHttpConfigurers) {
				http.apply(configurer);
			}
		}
		configure(http);
		return http;
	}
}


```

Demo7 security 过滤器链的默认配置

#### 1.5.2 在项目中使用 Spring Security

现在的项目大多都是前后端分离的。以开源项目 eladmin[26] 为例进行说明。该项目前端使用 Vue+Nodejs，后端使用 SpringBoot+Spring+Mybatis，基于 Spring Security 进行用户认证、鉴权授权。项目中引入 Spring Security 后，进行了用户认证、鉴权授权的功能开发。功能开发主要分为两块内容。

##### 1.5.2.1 用户认证

用户在登录时进行用户认证。如图 3，用户登录时，前端发送登录请求到后端的登录接口。登录接口的逻辑实现如 Demo8，其中调用了 Spring Security 中的方法。实际用户认证逻辑是在 Spring Security 框架中实现。Spring Security 会调用接口 UserDetailService 的 loadUserByUsername 方法获取系统中的用户，需要在系统中添加接口 UserDetailService 的实现类 UserDetailServiceImpl, 如 Demo9。在获取系统的用户后，会调用 Spring Security 中的 DaoAuthenticationProvider 的 additionalAuthenticationChecks 方法，该方法对比登录密码与系统中的密码是否一致，如果一致则用户认证成功，否则认证失败。需要开发者实现的是后台登录接口和获取用户信息的实现类 UserDetailServiceImpl，用户认证的逻辑是由 Spring Security 框架实现的。用户登录接口的 url 为 “/login”，该接口在过滤器配置中配置了所有用户（包括未登录用户）都可以访问。

[![](https://img2023.cnblogs.com/blog/1389306/202310/1389306-20231006184432596-2000113038.jpg)](https://img2023.cnblogs.com/blog/1389306/202310/1389306-20231006184432596-2000113038.jpg)

图 3 eladmin 项目中用户登录时，后台基于 Spring Security 进行用户认证

```
@AnonymousPostMapping(value = "/login")
public ResponseEntity<Object> login(@Validated @RequestBody AuthUserDto authUser, HttpServletRequest request) throws Exception {
    // 密码解密
    String password = RsaUtils.decryptByPrivateKey(RsaProperties.privateKey, authUser.getPassword());
    UsernamePasswordAuthenticationToken authenticationToken =
        new UsernamePasswordAuthenticationToken(authUser.getUsername(), password);
    
    //调用spring-security框架中的方法，进行用户认证
    Authentication authentication = 
    		authenticationManagerBuilder.getObject().authenticate(authenticationToken);

    // 生成令牌
    String token = tokenProvider.createToken(authentication);

    // 将令牌token存入redis中
    
    // 将令牌token应答到前端
}


```

Demo8 用户登录接口代码实现

```
@Service("userDetailsService")
public class UserDetailsServiceImpl implements UserDetailsService {
    @Override
    public JwtUserDto loadUserByUsername(String username) {
        user = userService.findByName(username);
          
        jwtUserDto = new JwtUserDto(
            user,
            dataService.getDeptIds(user),
            roleService.mapToGrantedAuthorities(user)
        );
                
        return jwtUserDto;
    }
}


```

Demo9 系统中添加了 UserDetailsService 的实现类 UserDetailsServiceImpl

##### 1.5.2.2 鉴权授权

如图 4，当前端向后端发送请求后，后端的 spring-security 会进行鉴权授权，判断用户是否有请求该资源的权限，如果没有权限则拒绝。如果用户已登录，会携带请求头 Authrozation，它的值是 token。如 Demo10，自定义过滤器 TokenFilter 继承 GenericFilterBean 类，在 doFilter 方法中根据请求头中的 token 判断用户是否已登录，如果已登录，将 token 中的用户及权限信息存入 SecurityContextHolder.getContext() 中。然后请求进入 Spring Security 的过滤器链，在过滤器 FilterSecurityInterceptor 中通过 AccessDecisionManager 的 decide 方法进行鉴权。decide 方法有 3 个参数，第一个参数 authenticated 是用户信息和权限信息，来源于 SecurityContextHolder.getContext()；第二个参数 objcet 是 spring-security 过滤器链相关的对象，第三个参数 attributes 的来源是 SecurityConfig 中的接口权限配置或默认配置。一个典型的 SecurityConfig 的配置如 Demo11，其中配置了访问接口需要的权限，比如. antMatchers(HttpMethod.OPTIONS, "/**").permitAll() 表示对所有 OPTIONS 的请求进行无条件放行；.anyRequest().authenticated() 表示对所有接口需要认证成功后才能访问。鉴权通过后，请求进入到后台接口的 Servlet 中，Servlet 处理该请求并向前台发送应答。总的来说，需要开发者实现的是自定义 TokenFilter，在 TokenFilter 中判断用户是否已登录，如果已登录，将 token 存在 SecurityContextHolder.getContext() 中；还需要自定义 SecurityConfig 继承 WebSecurityConfigurerAdapter，在 SecrityConfig 中进行过滤器配置，接口权限配置等。具体的鉴权授权是 Spring Security 框架实现的，它获取在项目启动时加载的 SecurityConfig 中的接口权限配置数据，并获取 SecurityContextHolder.getContext() 中的用户权限数据，然后通过 SPEL 表达式判断是否鉴权通过。

[![](https://img2023.cnblogs.com/blog/1389306/202310/1389306-20231006184452894-1097270162.jpg)](https://img2023.cnblogs.com/blog/1389306/202310/1389306-20231006184452894-1097270162.jpg)

图 4 eladmin 项目中前台向后台发送请求时，后台基于 Spring Security 进行鉴权授权的流程

```
public class TokenFilter extends GenericFilterBean {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
        String token = resolveToken(httpServletRequest);
        // 对于 Token 为空的不需要去查 Redis
        if (StrUtil.isNotBlank(token)) {
            OnlineUserDto onlineUserDto = null;
            boolean cleanUserCache = false;
            
            onlineUserDto = onlineUserService.getOne("online-token-" + token);
           	
            //如果redis中包含该用户，则说明已登录，将登录信息设置到SecurityContextHolder.getContext()中；
            if (onlineUserDto != null && StringUtils.hasText(token)) {
                Authentication authentication = tokenProvider.getAuthentication(token);
                SecurityContextHolder.getContext().setAuthentication(authentication);
                // Token 续期
                tokenProvider.checkRenewal(token);
            }
       		filterChain.doFilter(servletRequest, servletResponse);
    	}
    }
}


```

Demo10 自定义 TokenFilter 继承 GenericFilterBean

```
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity httpSecurity) throws Exception {
        // 搜寻匿名标记 url： @AnonymousAccess
        Map<RequestMappingInfo, HandlerMethod> handlerMethodMap = applicationContext.getBean(RequestMappingHandlerMapping.class).getHandlerMethods();
        // 获取匿名标记
        Map<String, Set<String>> anonymousUrls = getAnonymousUrl(handlerMethodMap);
        httpSecurity
                // 禁用 CSRF
                .csrf().disable()
                .addFilterBefore(corsFilter, UsernamePasswordAuthenticationFilter.class)
                // 授权异常
                .exceptionHandling()
                .authenticationEntryPoint(authenticationErrorHandler)
                .accessDeniedHandler(jwtAccessDeniedHandler)
                // 防止iframe 造成跨域
                .and()
                .headers()
                .frameOptions()
                .disable()
                // 不创建会话
                .and()
                .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
                // 静态资源等等
                .antMatchers(
                        HttpMethod.GET,
                        "/*.html",
                        "/**/*.html",
                        "/**/*.css",
                        "/**/*.js",
                        "/webSocket/**"
                ).permitAll()
                // swagger 文档
                .antMatchers("/swagger-ui.html").permitAll()
                .antMatchers("/swagger-resources/**").permitAll()
                .antMatchers("/webjars/**").permitAll()
                .antMatchers("/*/api-docs").permitAll()
                // 文件
                .antMatchers("/avatar/**").permitAll()
                .antMatchers("/file/**").permitAll()
                // 阿里巴巴 druid
                .antMatchers("/druid/**").permitAll()
                // 放行OPTIONS请求
                .antMatchers(HttpMethod.OPTIONS, "/**").permitAll()
                // 自定义匿名访问所有url放行：允许匿名和带Token访问，细腻化到每个 Request 类型
                // GET
                .antMatchers(HttpMethod.POST,"/**").permitAll()
                .antMatchers(HttpMethod.GET, anonymousUrls.get(RequestMethodEnum.GET.getType()).toArray(new String[0])).permitAll()
                // POST
                .antMatchers(HttpMethod.POST, anonymousUrls.get(RequestMethodEnum.POST.getType()).toArray(new String[0])).permitAll()
                // PUT
                .antMatchers(HttpMethod.PUT, anonymousUrls.get(RequestMethodEnum.PUT.getType()).toArray(new String[0])).permitAll()
                // PATCH
                .antMatchers(HttpMethod.PATCH, anonymousUrls.get(RequestMethodEnum.PATCH.getType()).toArray(new String[0])).permitAll()
                // DELETE
                .antMatchers(HttpMethod.DELETE, anonymousUrls.get(RequestMethodEnum.DELETE.getType()).toArray(new String[0])).permitAll()
                // 所有类型的接口都放行
                .antMatchers(anonymousUrls.get(RequestMethodEnum.ALL.getType()).toArray(new String[0])).permitAll()
                // 所有请求都需要认证
                .anyRequest().authenticated()
                .and().apply(securityConfigurerAdapter());
    }
}


```

Demo11 自定义 SecurityConfig 继承 WebSecurityConfigurerAdapter

总来来说，spring-security 是一个用户认证、鉴权授权的框架。Spring Security 的结构是一个过滤器链，当请求进来时会经过一系列的过滤器。本文侧重的是用户认证，鉴权授权的前后端交互流程，虽然后台用户认证、鉴权授权具体是由 Spring Security 实现的，实现细节也较为复杂，但这不影响对前后端交互流程的理解，本文只需对 Spring Security 有个初步认识即可。

2. 登录鉴权方式演变
-----------

登录鉴权方式是随着前后端架构的变化而变化的。早期的系统是前后端不分离的。通常前端是 freemaker/velocity/jsp+html。后端是 SSH 或 SSM。

后来 Vue 等前端框架的兴起，使得前后端得以分离。前端是 Vue+nodejs，后端是 SSM 或 SpirngBoot。SpringBoot 大大简化了应用的配置。

再后来微服务 SpringCloud 兴起，它包含网关、配置中心、注册中心等组件。多个微服务的登录鉴权实现和单应用系统又略有差异。

### 2.1 前后端不分离的登录鉴权流程

早期的系统是前后端不分离的。一个典型的系统使用 SSM+JSP 的架构，技术栈为 SpringMVC + Spring + Mybatis + JSP+ Apache + Weblogic。系统的架构图如图 5 所示。系统的登录鉴权流程如图 6 所示。系统未登录时，访问系统的请求将被用户权限过滤器拦截，首次进入权限过滤器会生成会话 Session，然后通过后端重定向跳转到登录页。在登录页中，用户填写好用户名密码后，提交 form 表单，请求后台登录接口，后台进行登录校验，登录成功后将用户名、密码等用户信息存入 Session 中。在后面的每次访问后端接口时，根据携带 cookie 的 jessionId 就可以从 Session 中获取用户信息，如果用户名不为空，就说明已认证过，然后可正常访问后端接口，不用每次访问都进行用户认证。用户登录认证失败后，会跳转到错误页。

[![](https://img2023.cnblogs.com/blog/1389306/202310/1389306-20231006184523985-127537681.jpg)](https://img2023.cnblogs.com/blog/1389306/202310/1389306-20231006184523985-127537681.jpg)

图 5 一个前后端不分离的典型系统的架构图

[![](https://img2023.cnblogs.com/blog/1389306/202310/1389306-20231006184547088-1357284742.jpg)](https://img2023.cnblogs.com/blog/1389306/202310/1389306-20231006184547088-1357284742.jpg)

图 6 一个前后端不分离的典型系统的登录鉴权流程

### 2.2 前后端分离后的登录鉴权流程

#### 2.2.1 单应用系统

在系统的前后端分离后，一个比较常见 Web 系统，前端使用 Vue+Nodejs，后端使用 SpringBoot+Spring+Mybatis。以开源项目 eladmin[26] 为例进行说明。系统的架构图如图 7 所示。系统的用户认证流程如图 8 所示，在用户认证流程中，前端应用可独立地提供页面的访问和实现页面间的跳转，后端实现用户认证的逻辑并提供接口。访问系统的请求通过 Vue-Router 导航到相应页面，路由导航会触发全局前置守卫的调用。在全局守卫的逻辑中，如果用户未登录，路由会导航到登录页。用户填好用户名和密码进行登录，向后台发起登录的 ajax 请求，后台会校验用户名密码，校验逻辑是基于 SpringSecurity 实现的，如果校验通过，则生成 JWT 类型的 token，应答到浏览器。浏览器端会保存 token 到 cookie 中，并创建后端接口请求拦截器，拦截请求并将 cookie 中的 token 放到请求头 Authentication 中。请求到达后端服务后，后端的用户权限过滤器会判断 Authentication 中的 token 是否已登录过（判断在 redis 中是否存在），并基于 SpringSecurity 进行鉴权，如果鉴权通过，则正常访问接口。不用每次访问后端接口都进行用户认证。如果登录失败，则应答错误状态码和错误信息，浏览器会在页面进行错误提示。

[![](https://img2023.cnblogs.com/blog/1389306/202310/1389306-20231006184710990-188619528.png)](https://img2023.cnblogs.com/blog/1389306/202310/1389306-20231006184710990-188619528.png)

图 7 前后端分离的典型单应用系统的架构图

[![](https://img2023.cnblogs.com/blog/1389306/202310/1389306-20231006184745529-727326527.jpg)](https://img2023.cnblogs.com/blog/1389306/202310/1389306-20231006184745529-727326527.jpg)

图 8 前后端分离的典型单应用系统的登录鉴权流程

#### 2.2.2 多个微服务的系统

后来微服务逐渐兴起，SpringCloud 是热门的技术之一。SpringCloud 包含一系列组件，包括 Eureka、Ribbon、Zuul、Feign 和 Config Server 等，方便进行微服务的管理、调用和配置。一个比较常见的 Web 系统，前端使用 Vue+Nodejs，后端使用 SpringBoot+SpringCloud+Spring+Mybatis。系统的架构图如图 9 所示。系统的用户认证流程如图 10 所示。与单应用系统的用户认证流程相比，主要有 2 点不同。一是用户认证逻辑会放在独立的鉴权微服务中。二是不是每个包含业务接口的微服务都放一个用户权限过滤器，而将过滤器放在网关微服务中。如图 10 的架构图，每个后端请求都会经过网关，在网关中放入用户权限过滤器是合适的。在实际业务中，网关作为后端微服务的唯一入口，后端微服务则放在内网中，不能不通过网关直接访问后端微服务的接口。用户认证成功后，后端会应答 set-cookie:token=aaaaa.bbbb.ccccc 到浏览器，浏览器将 token 存入 cookie 中，后续的接口访问请求都会携带该 cookie，请求经过权限过滤器时，过滤器将 cookie 中的 token 取出，判断该 token 是否已登录过（判断在 redis 中是否存在），并基于 SpringSecurity 进行鉴权，如果鉴权通过，则正常访问接口，不用每次都进行用户认证。不过，token 放在 Cookie 中是不建议的，建议放在请求头 Authrization 中，详细可参考 1.4.3 节。

[![](https://img2023.cnblogs.com/blog/1389306/202310/1389306-20231006184821661-481186955.jpg)](https://img2023.cnblogs.com/blog/1389306/202310/1389306-20231006184821661-481186955.jpg)

图 9 前后端分离的包含多个微服务的系统架构图

[![](https://img2023.cnblogs.com/blog/1389306/202310/1389306-20231006184840068-121104796.jpg)](https://img2023.cnblogs.com/blog/1389306/202310/1389306-20231006184840068-121104796.jpg)

图 10 前后端分离的包含多个微服务的系统登录鉴权流程

引用：
---

[1] [https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)

[2] [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)

[3] [https://foldoc.org/Dictionary](https://foldoc.org/Dictionary)

[4] [https://www.cnblogs.com/jann8/p/17472129.html](https://www.cnblogs.com/jann8/p/17472129.html)

[5] [https://www.cnblogs.com/xiaohuochai/p/7527273.html](https://www.cnblogs.com/xiaohuochai/p/7527273.html)

[6] [https://zhuanlan.zhihu.com/p/27588422](https://zhuanlan.zhihu.com/p/27588422)

[7] [https://jwt.io/introduction](https://jwt.io/introduction)

[8] [https://datatracker.ietf.org/doc/html/rfc7519](https://datatracker.ietf.org/doc/html/rfc7519)

[9] [https://www.strongdm.com/blog/token-based-authentication](https://www.strongdm.com/blog/token-based-authentication)

[10] [https://www.iana.org/assignments/jwt/jwt.xhtml](https://www.iana.org/assignments/jwt/jwt.xhtml)

[11] [https://techdocs.akamai.com/iot-token-access-control/docs/generate-rsa-keys](https://techdocs.akamai.com/iot-token-access-control/docs/generate-rsa-keys)

[12] [https://datatracker.ietf.org/doc/html/rfc7235](https://datatracker.ietf.org/doc/html/rfc7235)

[13] [https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication#authentication_schemes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication#authentication_schemes)

[14] [https://datatracker.ietf.org/doc/html/rfc6750](https://datatracker.ietf.org/doc/html/rfc6750)

[15] Maven 依赖 io.jsonwebtoken:jjwt:0.9.0 中 DefaultJwtParser 的源码

[16] [https://zhuanlan.zhihu.com/p/204946145](https://zhuanlan.zhihu.com/p/204946145)

[17] [https://www.bu.edu/tech/about/security-resources/bestpractice/auth/](https://www.bu.edu/tech/about/security-resources/bestpractice/auth/)

[18] [https://frontegg.com/blog/user-authorization](https://frontegg.com/blog/user-authorization)

[19] [https://baike.c114.com.cn/view.asp?id=14-32137803](https://baike.c114.com.cn/view.asp?id=14-32137803)

[20] [http://oldmh.ccsa.org.cn/article_new/dic_search.php](http://oldmh.ccsa.org.cn/article_new/dic_search.php?selectField=cname&categories_id=d33955da-cf2e-87b3-416e-4b552f05e5ef&selectStr=%BC%F8%C8%A8)

[21] [https://blog.csdn.net/Amelie123/article/details/125362070](https://blog.csdn.net/Amelie123/article/details/125362070)

[22] [https://wenku.baidu.com/view/e30e5d02a717866fb84ae45c3b3567ec102ddccb.html](https://wenku.baidu.com/view/e30e5d02a717866fb84ae45c3b3567ec102ddccb.html?_wkts_=1695607783665&bdQuery=%E9%80%9A%E4%BF%A1+%E8%AE%A4%E8%AF%81%E9%89%B4%E6%9D%83%E6%8E%88%E6%9D%83)

[23] [https://blog.csdn.net/qq_31063463/article/details/106359804?spm=1001.2014.3001.5502](https://blog.csdn.net/qq_31063463/article/details/106359804?spm=1001.2014.3001.5502)

[24] [https://docs.spring.io/spring-security/reference/servlet/architecture.html](https://docs.spring.io/spring-security/reference/servlet/architecture.html)

[25] [https://www.cnblogs.com/vincentren/p/15685730.html](https://www.cnblogs.com/vincentren/p/15685730.html)

[26] [https://github.com/elunez/](https://github.com/elunez/)