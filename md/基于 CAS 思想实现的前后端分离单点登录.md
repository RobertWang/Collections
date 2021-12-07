> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7004716677300486174)*   保护重点：CAS 是重点保障客户端的用户资源安全，Oauth2 则是保障服务端用户资源的安全。
*   使用场景：Oauth2.0 解决的是不同公司不同产品之间的实现登录的一种简单授权方式。CAS 通常处理同一个公司的不同应用间的应用访问登录问题。如企业有多个子系统，只需要登录一个系统，就可以实现不同系统之间的跳转，避免重复登录问题。

下面看看 CAS 的认证流程图

摘录自 CAS 官方文档 (CAS 文档)[[apereo.github.io/cas/6.2.x/p…](https://link.juejin.cn?target=https%3A%2F%2Fapereo.github.io%2Fcas%2F6.2.x%2Fprotocol%2FCAS-Protocol.html "https://apereo.github.io/cas/6.2.x/protocol/CAS-Protocol.html")]

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a20db016f4d4dcdb8b0ba4b5a59de96~tplv-k3u1fbpfcp-watermark.awebp)

各位认真了阅读一下上面的流程图，我相信一定已经将 CAS 的流程了然于胸了，不过在下还是会用几句话简单的描述一下。

*   浏览器发起请求到后端服务，后端服务判断当前用户是否登录
    *   如果没有登录过，带上 cookie（TGC）重定向到中央认证服务器
        *   如果在中央认证服务器中根据 TGC 的值，获取到了对应的 Session 数据，说明当前用户已经登录过，生成一个 ST 重定向回到浏览器开始请求的 URL 地址
        *   反之展示登录页面，让用户输入用户名和密码后提交登录，如果用户名和密码正确，缓存 Session 信息后生成一个 TGC ，同时生成一个 ticket 票据 ST，重定向回到浏览器开始请求的 URL 地址
    *   如果登录过了，生成一个 ST 重定向回到浏览器开始请求的 URL 地址

在下的遇到的问题
--------

目前我们公司有许多的产品，这些产品都有一套记录的用户管理系统。导致平台用户去使用各个产品的时候，需要在不同的子系统之间进行登录交互，用户反馈操作不够友好。所以公司需要实现**单点登录**功能。同时需要兼容原有子系统的授权、鉴权流程，也就是说单点登录服务值提供认证服务即可。

在互联网上查找了很多关于这方面的资料。无外乎两种声音，Oauth2.0 和 CAS，而且绝大多数都是在介绍在单体应用的情况下如何实现，很少有资料介绍如何在前后端分离的情况下介绍的很清楚（当然也可能是在下愚昧，不能理解罢了），而且在 Spring 项目中，基于`Spring security Oauth2.0`实现，在关于如何衔接现有子系统的授权流程的时候，显得非常的笨重不够灵活。所以在和领导商量下，决定基于`CAS和Oauth2.0`的思想自己开发一套认证服务。

前后端分离场景
-------

#### 重定向问题

在前后端分离的场景里面，遇到的一大问题就是重定向的问题。前端请求通过 ajax 的方式请求后端服务，这时候是不能由后端服务直接重定向到认证服务器的，只能返回对应的 json 数据，提示前端自己去重定向到中央认证服务器。

#### 安全问题

浏览器端需要能够直接访问中央认证服务器，那么为了限制部分浏览器端能访问，部分不能访问。我们需要设置两个参数`clientId`(表明客户端是谁) 和`secrect`（密钥），在浏览器重定向到认证服务器的时候需要带上这两个参数，用户认证服务器判定当前浏览器客户端是否可以接入单点登录。

流程设计
----

在下目前实现的也只是一个基本可使用版，只是分享出来供各位参考。其中还存在种种的细节问题，**欢迎评论留言交流**

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/95a0283c683d47d18553295d6cdceca1~tplv-k3u1fbpfcp-watermark.awebp)

代码实现
----

### 服务端

`AuthEndpoint.java`

```
package com.pkit;

import cn.hutool.core.util.IdUtil;
import cn.hutool.core.util.StrUtil;
import cn.hutool.json.JSONUtil;
import com.pkit.service.ISsoClientConfigService;
import com.pkit.service.IUserService;
import com.pkit.vo.SsoUserVO;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;
import java.util.Objects;
import java.util.concurrent.TimeUnit;

/**
 * 认证相关的端点
 *
 * @author zhuxy  zhuxy@pukkasoft.cn
 * @date 2021-08-26 11:05
 */
@Controller
@RequestMapping("/pkit")
public class AuthEndpoint {


    private final static String TGC = "CASTGC";
    private final static String TGC_REDIS_PREFIX = "CASTGC-";
    private final static String SSO_TOKEN_REDIS_PREFIX = "SSO-TOKEN-";
    private final static String CODE_REDIS_PREFIX = "CODE-";

    private final static int COOKIE_EXPIRE_TIME = 60 * 60 * 24;
    private final static int TOKEN_EXPIRE_TIME = 60 * 60 * 8;

    private final StringRedisTemplate redisTemplate;

    private final IUserService userService;
    private final ISsoClientConfigService clientConfigService;

    public AuthController(StringRedisTemplate redisTemplate, IUserService userService,
                          ISsoClientConfigService clientConfigService) {
        this.redisTemplate = redisTemplate;
        this.userService = userService;
        this.clientConfigService = clientConfigService;
    }

    /**
     * 跳转到登录页面
     *
     * @return 登录页
     */
    @GetMapping("/login")
    public ModelAndView loginPage(HttpServletRequest request, HttpServletResponse response, ModelAndView modelAndView) throws IOException {

        String clientId = request.getParameter("clientId");
        String secrete = request.getParameter("secret");

        modelAndView.setViewName("login");

        if (!clientConfigService.isLegalClient(clientId, secrete, modelAndView)) {

            return modelAndView;
        }

        Cookie[] cookies = request.getCookies();
        Cookie cookie = findCookieByName(cookies);
        if (cookie == null) {
            return modelAndView;
        }
        String value = cookie.getValue();
        String userInfo;
        // cookie是否失效
        if ((userInfo = redisTemplate.opsForValue().get(TGC_REDIS_PREFIX + value)) == null) {
            return modelAndView;
        }

        String redirectUrl = request.getParameter("redirectUrl");

        if (StrUtil.isBlank(redirectUrl)) {
            modelAndView.setViewName("index");
            return modelAndView;
        }

        String code = IdUtil.simpleUUID().toLowerCase();
        setResponseWithAuthorization(code, userInfo);

        response.sendRedirect(redirectUrl + "?code=" + code);
       
        return modelAndView;
    }

    @PostMapping("/action/login")
    public String login(String userName, String password, String redirectUrl,
                        String clientId, HttpServletResponse response) throws IOException {

        SsoUserVO ssoUserVO = userService.login(userName, password, clientId);

        if (StrUtil.isBlank(redirectUrl)) {
            return "index";
        }
        String userInfo = JSONUtil.toJsonStr(ssoUserVO);
        String cookieValue = IdUtil.simpleUUID().toLowerCase();

        String code = IdUtil.simpleUUID().toLowerCase();

        setResponseWithAuthorizationAndCookie(response, code, userInfo,
                                              TGC, cookieValue, "/", COOKIE_EXPIRE_TIME);


        response.sendRedirect(redirectUrl + "?code=" + code);
        return null;
    }

    /**
     * 使用授权码兑换token
     *
     * @param code 授权码
     * @return accessToken
     */
    @GetMapping("/accessToken")
    public ResponseEntity<Map<String, Object>> accessToken(@RequestParam String code) {
        String userInfo;

        if ((userInfo = redisTemplate.opsForValue().get(CODE_REDIS_PREFIX + code)) == null) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
        }

        String accessToken = IdUtil.simpleUUID().toLowerCase();
        //颁发令牌
        redisTemplate.opsForValue().set(SSO_TOKEN_REDIS_PREFIX + accessToken, userInfo, TOKEN_EXPIRE_TIME, TimeUnit.SECONDS);

        HashMap<String, Object> result = new HashMap<>();
        result.put("accessToken", accessToken);
        result.put("code", 1000);
        return ResponseEntity.ok(result);
    }
    
	/**
	* SSO-Client调用该接口检验accessToken是否合法
	*
	*/
    @GetMapping("/token/validate")
    public ResponseEntity<String> validate(@RequestParam String accessToken) {
        String userInfo;
        if ((userInfo = redisTemplate.opsForValue().get(SSO_TOKEN_REDIS_PREFIX + accessToken)) == null) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
        }
        Long expire = redisTemplate.opsForValue().getOperations().getExpire(SSO_TOKEN_REDIS_PREFIX + accessToken, TimeUnit.SECONDS);
        SsoUserVO userVO = JSONUtil.toBean(userInfo, SsoUserVO.class);
        userVO.setExpireTime(expire == null ? 10 : expire);
        return ResponseEntity.ok(JSONUtil.toJsonStr(userVO));
    }


    /**
     * 查找TGC Cookie
     *
     * @param cookies Request中的Cookies
     * @return 对应的Cookie
     */
    private Cookie findCookieByName(Cookie[] cookies) {
        if (Objects.isNull(cookies)) {
            return null;
        }
        for (Cookie cookie : cookies) {
            if (AuthController.TGC.equals(cookie.getName())) {
                return cookie;
            }
        }
        return null;
    }

    private void setResponseWithAuthorization(String code, String userInfo) {
        redisTemplate.opsForValue().set(CODE_REDIS_PREFIX + code, userInfo, 5, TimeUnit.SECONDS);

    }
	
    private void setResponseWithAuthorizationAndCookie(HttpServletResponse response, String code, String userInfo,String cookieName, String cookieValue,String path, int cookieExpireTime) {
        setResponseWithAuthorization(code, userInfo);
        Cookie cookie = new Cookie(cookieName, cookieValue);
        cookie.setMaxAge(cookieExpireTime);
        cookie.setPath(path);
        response.addCookie(cookie);
        redisTemplate.opsForValue().set(TGC_REDIS_PREFIX + cookieValue, userInfo, cookieExpireTime, TimeUnit.SECONDS);
    }

复制代码
```

上面暴露出了四个端点

*   `/login`用户访问跳转到平台登录页
*   `/action/login`登录页提交登录请求的端点，处理用户登录逻辑 (只是认证)
*   `/accesstoken`客户端使用授权码兑换 `accessToken`
*   `/token/validate`客户端校验 `accessToken`，目的是与服务端实现`accessToken`同步

各位看到这里的时候肯定会有如下疑问：

*   为什么没有退出登录的端点呢?
    *   答: 哦，没来得及做。聪明的各位可自行实现（很简单的）
*   为什么在重定向回客户端的时候，不直接返回 accessToken？而是先返回 code，在用 code 去获取 accessToken？
    *   答：主要是出于安全的考虑，因为在重定向回去的时候，只能将数据带在 URL 里面（PS: 如果各位有更好的重定向带数据的方式请评论留言，么么哒）这样做的话 accessToken 泄露的概率很大。

这就是服务端的核心代码，其实不难的。文末有完整代码

### 客户端

在客户端我们需要配置服务端的端点地址，同时需要对全局接口进行拦截，统一判断每一个访问的请求是否被认证过。所以显而易见我们需要一个全局 Interceptor。

`pom.xml`

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <optional>true</optional>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <optional>true</optional>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>

<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.7.9</version>
</dependency>

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
复制代码
```

`application.yml`

```
pkit:
  sso:
    client:
      clientId: client1
      secrete: 123456
    server:
      loginUrl: http://192.168.0.1:8888/pkit/login # 单点登录地址
      tokenUrl: http://192.168.0.1:8888/pkit/accesstoken # 获取token地址
      tokenValidate: http://192.168.0.1:8888/pkit/token/validate #校验token地址
# 这里默认使用的Redis作为本地Session缓存管理
spring:
  redis:
    password: PUKKA028
    host: 192.168.0.1
    database: 1

复制代码
```

`RestAuthenticationInterceptor.java`

```
package com.pkit.framework.interceptor;

import cn.hutool.core.bean.BeanUtil;
import cn.hutool.core.util.StrUtil;
import cn.hutool.http.HttpRequest;
import cn.hutool.http.HttpResponse;
import cn.hutool.http.HttpUtil;
import com.pkit.framework.config.SSOProperties;
import com.pkit.framework.exception.BizException;
import com.pkit.framework.exception.BizExceptionEnum;
import com.pkit.framework.vo.SsoUserVO;
import lombok.Getter;
import lombok.Setter;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.concurrent.TimeUnit;

/**
 * 认证拦截器
 *
 * @author zhuxy  zhuxy@pukkasoft.cn
 * @date 2021-08-30 14:10:25
 */
@Component
public class RestAuthenticationInterceptor implements HandlerInterceptor {

    private final SSOProperties ssoProperties;
    private final StringRedisTemplate redisTemplate;
    public RestAuthenticationInterceptor(SSOProperties ssoProperties, StringRedisTemplate redisTemplate) {
        this.ssoProperties = ssoProperties;
        this.redisTemplate = redisTemplate;
    }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String accessToken = request.getHeader("Authorization");

        String clientId = ssoProperties.getClient().getClientId();
        String secret = ssoProperties.getClient().getSecret();
        String loginUrl = ssoProperties.getServer().getLoginUrl();
        String tokenValidateUrl = ssoProperties.getServer().getTokenValidate();

        //没有登录过
        if (StrUtil.isBlank(accessToken)) {
            //返回重定向，通知客户端重定向到 loginUrl
            RedirectUrlInfo redirectUrlInfo = new RedirectUrlInfo(loginUrl, clientId, secret);
            throw new BizException(BizExceptionEnum.REDIRECT,redirectUrlInfo);
        }
        //查询本地缓存是否有Session信息
        String sessionInfo = redisTemplate.opsForValue().get("Authorization-" + accessToken);
        SsoUserVO userVO ;
        if (StrUtil.isBlank(sessionInfo)) {
            //查询在认证服务器中是否登录过
            HttpRequest getRequest = HttpUtil.createGet(tokenValidateUrl);
            getRequest.header("accessToken",accessToken);
            HttpResponse httpResponse = getRequest.execute();
            //在认证服务器登录过了
            if (httpResponse.isOk()){
                String body = httpResponse.body();
                userVO = BeanUtil.toBean(body, SsoUserVO.class);
                redisTemplate.opsForValue().set("Authorization-" + accessToken,body,userVO.getExpireTime(), TimeUnit.MILLISECONDS);
            }else{
                //返回重定向，通知客户端重定向到 loginUrl
                RedirectUrlInfo redirectUrlInfo = new RedirectUrlInfo(loginUrl, clientId, secret);
                throw new BizException(BizExceptionEnum.REDIRECT,redirectUrlInfo);
            }
        }else{
            userVO = BeanUtil.toBean(sessionInfo,SsoUserVO.class);
        }
        request.setAttribute("clientUserName",userVO.getClientName());
        request.setAttribute("accessToken",accessToken);
        return true;
    }


    @Getter
    @Setter
    static class RedirectUrlInfo {
        private String loginUrl;
        private String clientId;
        private String secrete;

        public RedirectUrlInfo(String loginUrl, String clientId, String secrete) {
            this.loginUrl = loginUrl;
            this.clientId = clientId;
            this.secrete = secrete;
        }
    }
}

复制代码
```

上面代码中的注释已经非常的详细，在下还是在简单的描述一下流程：

判断用户请求头中是否存在约定的请求头`Authorization`

1.  如果没有，说明用户没有登录过，直接返回 1302 状态码, 重定向 Url，clientId，secret，提示前端使用`window.location.href`重定向到 Server 端的登录页面
    
2.  如果存在该请求头参数，先从本地 Session 缓存中查询是否有对应的会话
    
    1.  如果本地 Session 缓存中存在当前会话，取出会话信息，封装为`SsoUser`对象将该对象往下一个拦截器传递，进行权限校验。
    2.  如果本地 Session 缓存中不存在当前会话，发起 Rest 请求 Server 端的`/token/validate`端点。
        *   如果响应 200 状态码并且返回了 accessToken 参数，将该 accessToken 存储在本地缓存，并执行 2.1。
        *   如果响应了 401 状态码，则执行 1

当然上面的很多步骤都涉及到前端开发 XD，所以各位有必要为他们讲解一下其中的流程。一个简单的单点服务就这样搭建起来了，欢迎各位在下方留言校验指正。

[代码地址](https://link.juejin.cn?target=https%3A%2F%2Fwww.aliyundrive.com%2Fs%2F1Q2D9Ee3crQ "https://www.aliyundrive.com/s/1Q2D9Ee3crQ")