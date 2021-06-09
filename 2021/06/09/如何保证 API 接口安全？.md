> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI4NDY5Mjc1Mg==&mid=2247508670&idx=2&sn=477a46fa89d30e6a2f4af3d4954c490d&chksm=ebf57ac1dc82f3d781f7fdce08ec7be2d965e2357c927e43a6aa1008ac6346da51a30ccab8e2&mpshare=1&scene=1&srcid=0608jdiCyuU1mBPbXzOdoixl&sharer_sharetime=1623131717674&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

### 一、摘要

在实际的业务开发过程中，我们常常会碰到需要与第三方互联网公司进行技术对接，例如支付宝支付对接、微信支付对接、高德地图查询对接等等服务，如果你是一个创业型互联网，大部分可能都是对接别的公司 api 接口。

当你的公司体量上来了时候，这个时候可能有一些公司开始找你进行技术对接了，转变成由你来提供 api 接口，那这个时候，我们应该如何设计并保证 API 接口安全呢？

### 二、方案介绍

最常用的方案，主要有两种：

*   token 方案
    
*   接口签名
    

#### 2.1、token 方案

其中 token 方案，是一种在 web 端使用最广的接口鉴权方案，我记得在之前写过一篇《手把手教你，使用 JWT 实现单点登录》的文章，里面介绍的比较详细，有兴趣的朋友可以看一下，没了解的也没关系，我们在此简单的介绍一下 token 方案。

![](https://mmbiz.qpic.cn/mmbiz_png/tWOhQMr1wdBpCrR8MBf6XIzvFtITWOicROlo7jFsYBJ9meyib0HzVxvqe1ykDkCJo24jYTicpexGYIOBSdwarjEXA/640?wx_fmt=png)

从上图，我们可以很清晰的看到，token 方案的实现主要有以下几个步骤：

*   1、用户登录成功之后，服务端会给用户生成一个唯一有效的凭证，这个有效值被称为 token
    
*   2、当用户每次请求其他的业务接口时，需要在请求头部带上 token
    
*   3、服务端接受到客户端业务接口请求时，会验证 token 的合法性，如果不合法会提示给客户端；如果合法，才会进入业务处理流程。
    

在实际使用过程中，当用户登录成功之后，生成的 token 存放在 redis 中时是有时效的，一般设置为 2 个小时，过了 2 个小时之后会自动失效，这个时候我们就需要重新登录，然后再次获取有效 token。

token 方案，是目前业务类型的项目当中使用最广的方案，而且实用性非常高，可以很有效的防止黑客们进行抓包、爬取数据。

但是 token 方案也有一些缺点！最明显的就是与第三方公司进行接口对接的时候，当你的接口请求量非常大，这个时候 token 突然失效了，会有大量的接口请求失败。

这个我深有体会，我记得在很早的时候，跟一家中、大型互联网公司进行联调的时候，他们提供给我的接口对接方案就是 token 方案，当时我司的流量高峰期时候，请求他们的接口大量报错，原因就是因为 token 失效了，当 token 失效时，我们会调用他们刷新 token 接口，刷新完成之后，在 token 失效与重新刷新 token 这个时间间隔期间，就会出现大量的请求失败的日志，因此在实际 API 对接过程中，我不推荐大家采用 token 方案。

#### 2.2、接口签名

接口签名，顾名思义，就是通过一些签名规则对参数进行签名，然后把签名的信息放入请求头部，服务端收到客户端请求之后，同样的只需要按照已定的规则生产对应的签名串与客户端的签名信息进行对比，如果一致，就进入业务处理流程；如果不通过，就提示签名验证失败。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tWOhQMr1wdBpCrR8MBf6XIzvFtITWOicRDr3IUE6wCVvNfGAibc3X1WTmHWaw1N0iasjwBDZUBYDQwyCYcl2RA6Nw/640?wx_fmt=jpeg)

在接口签名方案中，主要有四个核心参数：

*   1、appid 表示应用 ID，其中与之匹配的还有 appsecret，表示应用密钥，用于数据的签名加密，不同的对接项目分配不同的 appid 和 appsecret，保证数据安全
    
*   2、timestamp 表示时间戳，当请求的时间戳与服务器中的时间戳，差值在 5 分钟之内，属于有效请求，不在此范围内，属于无效请求
    
*   3、nonce 表示临时流水号，用于防止重复提交验证
    
*   4、signature 表示签名字段，用于判断接口请求是否有效。
    

其中签名的生成规则，分两个步骤：

*   第一步：对请求参数进行一次 md5 加密签名
    

```
//步骤一
String 参数1 = 请求方式 + 请求URL相对地址 + 请求Body字符串;
String 参数1加密结果= md5(参数1)


```

*   第二步：对第一步签名结果，再进行一次 md5 加密签名
    

```
//步骤二
String 参数2 = appsecret + timestamp + nonce + 参数1加密结果;
String 参数2加密结果= md5(参数2)


```

参数 2 加密结果，就是我们要的最终签名串。

接口签名方案，尤其是在接口请求量很大的情况下，依然很稳定。

换句话说，你可以将接口签名看作成对 token 方案的一种补充。

但是如果想把接口签名方案，推广到前后端对接，答案是：不适合。

因为签名计算非常复杂，其次，就是容易泄漏 appsecret！

说了这么多，下面我们就一起来用程序实践一下吧！

### 二、程序实践

#### 2.1、token 方案

就像上文所说，token 方案重点在于，当用户登录成功之后，我们只需要生成好对应的 token，然后将其返回给前端，在下次请求业务接口的时候，需要把 token 带上。

具体的实践，也可以分两种：

*   第一种：采用 uuid 生成 token，然后将 token 存放在 redis 中，同时设置有效期 2 哥小时
    
*   第二种：采用 JWT 工具来生成 token，这种 token 是可以跨平台的，天然支持分布式，其实本质也是采用时间戳 + 密钥，来生成一个 token。
    

下面，我们介绍的是第二种实现方式。

首先，编写一个 jwt 工具。

```
public class JwtTokenUtil {
    //定义token返回头部
    public static final String AUTH_HEADER_KEY = "Authorization";
    //token前缀
    public static final String TOKEN_PREFIX = "Bearer ";
    //签名密钥
    public static final String KEY = "q3t6w9z$C&F)J@NcQfTjWnZr4u7x";
    //有效期默认为 2hour
    public static final Long EXPIRATION_TIME = 1000L*60*60*2;
    /**
     * 创建TOKEN
     * @param content
     * @return
     */
    public static String createToken(String content){
        return TOKEN_PREFIX + JWT.create()
                .withSubject(content)
                .withExpiresAt(new Date(System.currentTimeMillis() + EXPIRATION_TIME))
                .sign(Algorithm.HMAC512(KEY));
    }
    /**
     * 验证token
     * @param token
     */
    public static String verifyToken(String token) throws Exception {
        try {
            return JWT.require(Algorithm.HMAC512(KEY))
                    .build()
                    .verify(token.replace(TOKEN_PREFIX, ""))
                    .getSubject();
        } catch (TokenExpiredException e){
            throw new Exception("token已失效，请重新登录",e);
        } catch (JWTVerificationException e) {
            throw new Exception("token验证失败！",e);
        }
    }
}


```

接着，我们在登录的时候，生成一个 token，然后返回给客户端。

```
@RequestMapping(value = "/login", method = RequestMethod.POST, produces = {"application/json;charset=UTF-8"})
public UserVo login(@RequestBody UserDto userDto, HttpServletResponse response){
    //...参数合法性验证
    //从数据库获取用户信息
    User dbUser = userService.selectByUserNo(userDto.getUserNo);
    //....用户、密码验证
    //创建token，并将token放在响应头
    UserToken userToken = new UserToken();
    BeanUtils.copyProperties(dbUser,userToken);
    String token = JwtTokenUtil.createToken(JSONObject.toJSONString(userToken));
    response.setHeader(JwtTokenUtil.AUTH_HEADER_KEY, token);
    //定义返回结果
    UserVo result = new UserVo();
    BeanUtils.copyProperties(dbUser,result);
    return result;
}


```

最后，编写一个统一拦截器，用于验证客户端传入的 token 是否有效。

```
@Slf4j
public class AuthenticationInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 从http请求头中取出token
        final String token = request.getHeader(JwtTokenUtil.AUTH_HEADER_KEY);
        //如果不是映射到方法，直接通过
        if(!(handler instanceof HandlerMethod)){
            return true;
        }
        //如果是方法探测，直接通过
        if (HttpMethod.OPTIONS.equals(request.getMethod())) {
            response.setStatus(HttpServletResponse.SC_OK);
            return true;
        }
        //如果方法有JwtIgnore注解，直接通过
        HandlerMethod handlerMethod = (HandlerMethod) handler;
        Method method=handlerMethod.getMethod();
        if (method.isAnnotationPresent(JwtIgnore.class)) {
            JwtIgnore jwtIgnore = method.getAnnotation(JwtIgnore.class);
            if(jwtIgnore.value()){
                return true;
            }
        }
        LocalAssert.isStringEmpty(token, "token为空，鉴权失败！");
        //验证，并获取token内部信息
        String userToken = JwtTokenUtil.verifyToken(token);
        //将token放入本地缓存
        WebContextUtil.setUserToken(userToken);
        return true;
    }
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        //方法结束后，移除缓存的token
        WebContextUtil.removeUserToken();
    }
}


```

在生成 token 的时候，我们可以将一些基本的用户信息，例如用户 ID、用户姓名，存入 token 中，这样当 token 鉴权通过之后，我们只需要通过解析里面的信息，即可获取对应的用户 ID，可以省下去数据库查询一些基本信息的操作。

同时，使用的过程中，尽量不要存放敏感信息，因为很容易被黑客解析！

#### 2.2、接口签名

同样的思路，站在服务端验证的角度，我们可以先编写一个签名拦截器，验证客户端传入的参数是否合法，只要有一项不合法，就提示错误。

具体代码实践如下：

```
public class SignInterceptor implements HandlerInterceptor {

    @Autowired
    private AppSecretService appSecretService;

    @Autowired
    private RedisUtil redisUtil;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {
        //appId验证
        final String appId = request.getHeader("appid");
        if(StringUtils.isEmpty(appId)){
            throw new CommonException("appid不能为空");
        }
        String appSecret = appSecretService.getAppSecretByAppId(appId);
        if(StringUtils.isEmpty(appSecret)){
            throw new CommonException("appid不合法");
        }
        //时间戳验证
        final String timestamp = request.getHeader("timestamp");
        if(StringUtils.isEmpty(timestamp)){
            throw new CommonException("timestamp不能为空");
        }
        //大于5分钟，非法请求
        long diff = System.currentTimeMillis() - Long.parseLong(timestamp);
        if(Math.abs(diff) > 1000 * 60 * 5){
            throw new CommonException("timestamp已过期");
        }
        //临时流水号，防止重复提交
        final String nonce = request.getHeader("nonce");
        if(StringUtils.isEmpty(nonce)){
            throw new CommonException("nonce不能为空");
        }
        //验证签名
        final String signature = request.getHeader("signature");
        if(StringUtils.isEmpty(nonce)){
            throw new CommonException("signature不能为空");
        }
        final String method = request.getMethod();
        final String url = request.getRequestURI();
        final String body = StreamUtils.copyToString(request.getInputStream(), Charset.forName("UTF-8"));
        String signResult = SignUtil.getSignature(method, url, body, timestamp, nonce, appSecret);
        if(!signature.equals(signResult)){
            throw new CommonException("签名验证失败");
        }
        //检查是否重复请求
        String key = appId + "_" + timestamp + "_" + nonce;
        if(redisUtil.exist(key)){
            throw new CommonException("当前请求正在处理，请不要重复提交");
        }
        //设置5分钟
        redisUtil.save(key, signResult, 5*60);
        request.setAttribute("reidsKey",key);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
            throws Exception {
        //请求处理完毕之后，移除缓存
        String value = request.getAttribute("reidsKey");
        if(!StringUtils.isEmpty(value)){
            redisUtil.remove(value);
        }
    }

}


```

签名工具类`SignUtil`：

```
public class SignUtil {

    /**
     * 签名计算
     * @param method
     * @param url
     * @param body
     * @param timestamp
     * @param nonce
     * @param appSecret
     * @return
     */
    public static String getSignature(String method, String url, String body, String timestamp, String nonce, String appSecret){
        //第一层签名
        String requestStr1 = method + url + body + appSecret;
        String signResult1 = DigestUtils.md5Hex(requestStr1);
        //第二层签名
        String requestStr2 = appSecret + timestamp + nonce + signResult1;
        String signResult2 = DigestUtils.md5Hex(requestStr2);
        return signResult2;
    }
}


```

签名计算，可以换成`hamc`方式进行计算，思路大致一样。

### 三、小结

上面介绍的 token 和接口签名方案，对外都可以对提供的接口起到保护作用，防止别人篡改请求，或者模拟请求。

但是缺少对数据自身的安全保护，即请求的参数和返回的数据都是有可能被别人拦截获取的，而这些数据又是明文的，所以只要被拦截，就能获得相应的业务数据。

对于这种情况，推荐大家对请求参数和返回参数进行加密处理，例如 RSA、AES 等加密工具。

同时，在生产环境，采用`https`方式进行传输，可以起到很好的安全保护作用！