> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247516734&idx=1&sn=966d166d3052f10b9310ff4014733669&chksm=ce0e35bdf979bcabbb75366e1bad32bdbf8f41d1dc80b8be6e39225bd708913ce2c93472f2fc&scene=21#wechat_redirect)

本文我们来看下 SpringSecurity  + JWT 实现单点登录操作，本文 2W 字，预计阅读时间 30 min，文章提供了代码骨架，建议收藏。

一、什么是单点登陆

单点登录（Single Sign On），简称为 SSO，是目前比较流行的企业业务整合的解决方案之一。SSO 的定义是在多个应用系统中，用户只需要登录一次就可以访问所有相互信任的应用系统

二、简单的运行机制

单点登录的机制其实是比较简单的，用一个现实中的例子做比较。某公园内部有许多独立的景点，游客可以在各个景点门口单独买票。

对于需要游玩所有的景点的游客，这种买票方式很不方便，需要在每个景点门口排队买票，钱包拿 进拿出的，容易丢失，很不安全。

于是绝大多数游客选择在大门口买一张通票（也叫套票），就可以玩遍所有的景点而不需要重新再买票。他们只需要在每个景点门 口出示一下刚才买的套票就能够被允许进入每个独立的景点。

单点登录的机制也一样，如下图所示，

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufyg01nHIrpMGoMrsthhZHxecXcSFY3pc3qZ4NYAicDeI9JfKrzHQWgzggDD9LPDub0Kia5ksgQVgyg/640?wx_fmt=png)

**用户认证：**这一环节主要是用户向认证服务器发起认证请求，认证服务器给用户返回一个成功的令牌 token，主要在认证服务器中完成，即图中的认证系统，注意认证系统只能有一个。

**身份校验：** 这一环节是用户携带 token 去访问其他服务器时，在其他服务器中要对 token 的真伪进行检验，主要在资源服务器中完成，即图中的应用系统 2 3

三、JWT 介绍

#### 概念说明

从分布式认证流程中，我们不难发现，这中间起最关键作用的就是 token，token 的安全与否，直接关系到系统的健壮性，这里我们选择使用 JWT 来实现 token 的生成和校验。

JWT，全称 JSON Web Token，官网地址 https://jwt.io，是一款出色的分布式身份校验方案。可以生成 token，也可以解析检验 token。

#### JWT 生成的 token 由三部分组成：

*   头部：主要设置一些规范信息，签名部分的编码格式就在头部中声明。
    
*   载荷：token 中存放有效信息的部分，比如用户名，用户角色，过期时间等，但是不要放密码，会泄露！
    
*   签名：将头部与载荷分别采用 base64 编码后，用 “.” 相连，再加入盐，最后使用头部声明的编码类型进行编码，就得到了签名。
    

#### JWT 生成 token 的安全性分析

从 JWT 生成的 token 组成上来看，要想避免 token 被伪造，主要就得看签名部分了，而签名部分又有三部分组成，其中头部和载荷的 base64 编码，几乎是透明的，毫无安全性可言，那么最终守护 token 安全的重担就落在了加入的盐上面了！

试想：如果生成 token 所用的盐与解析 token 时加入的盐是一样的。岂不是类似于中国人民银行把人民币防伪技术公开了？大家可以用这个盐来解析 token，就能用来伪造 token。这时，我们就需要对盐采用非对称加密的方式进行加密，以达到生成 token 与校验 token 方所用的盐不一致的安全效果！

#### 非对称加密 RSA 介绍

**基本原理：**同时生成两把密钥：私钥和公钥，私钥隐秘保存，公钥可以下发给信任客户端

*   私钥加密，持有私钥或公钥才可以解密
    
*   公钥加密，持有私钥才可解密
    

**优点：** 安全，难以破解

**缺点：** 算法比较耗时，为了安全，可以接受

**历史：** 三位数学家 Rivest、Shamir 和 Adleman 设计了一种算法，可以实现非对称加密。这种算法用他们三个人的名字缩写：RSA。

四、SpringSecurity 整合 JWT

### 1. 认证思路分析

SpringSecurity 主要是通过过滤器来实现功能的！我们要找到 SpringSecurity 实现认证和校验身份的过滤器！

#### 回顾集中式认证流程

**用户认证：**使用`UsernamePasswordAuthenticationFilter`过滤器中`attemptAuthentication`方法实现认证功能，该过滤器父类中`successfulAuthentication`方法实现认证成功后的操作。

**身份校验：**使用`BasicAuthenticationFilter`过滤器中`doFilterInternal`方法验证是否登录，以决定能否进入后续过滤器。

#### 分析分布式认证流程

**用户认证：**

由于分布式项目，多数是前后端分离的架构设计，我们要满足可以接受异步 post 的认证请求参数，需要修改`UsernamePasswordAuthenticationFilter`过滤器中`attemptAuthentication`方法，让其能够接收请求体。

另外，默认`successfulAuthentication`方法在认证通过后，是把用户信息直接放入 session 就完事了，现在我们需要修改这个方法，在认证通过后生成 token 并返回给用户。

**身份校验：**

原来 BasicAuthenticationFilter 过滤器中 doFilterInternal 方法校验用户是否登录，就是看 session 中是否有用户信息，我们要修改为，验证用户携带的 token 是否合法，并解析出用户信息，交给 SpringSecurity，以便于后续的授权功能可以正常使用。

### **2. 具体实现**

为了演示单点登录的效果，我们设计如下项目结构

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufyg01nHIrpMGoMrsthhZHxUibicxibuFhVBPT86K1z9T7xCFdCe7VjpdoSV0yUgZOjuwAbwjjALN2Gw/640?wx_fmt=png)

#### 2.1 父工程创建

因为本案例需要创建多个系统，所以我们使用 maven 聚合工程来实现，首先创建一个父工程，导入 springboot 的父依赖即可

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.3.RELEASE</version>
    <relativePath/>
</parent>

```

#### 2.2 公共工程创建

然后创建一个 common 工程，其他工程依赖此系统

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufyg01nHIrpMGoMrsthhZHxWsEjXT27kibQW1BIzejmnqibMMlgiaicfuwTXV1y1icZ7DibL7I8bVsd3vicA/640?wx_fmt=png)

**导入 JWT 相关的依赖**

```
<dependencies>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
        <version>0.10.7</version>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-impl</artifactId>
        <version>0.10.7</version>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-jackson</artifactId>
        <version>0.10.7</version>
        <scope>runtime</scope>
    </dependency>
    <!--jackson包-->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.9.9</version>
    </dependency>
    <!--日志包-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-logging</artifactId>
    </dependency>
    <dependency>
        <groupId>joda-time</groupId>
        <artifactId>joda-time</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
    </dependency>
</dependencies>

```

创建相关的工具类  

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufyg01nHIrpMGoMrsthhZHxX0sEWYReeFkia7bZgSeygl7wsxvvtNvwMB35V6m0SCzNt9ibcIXibB8Zw/640?wx_fmt=png)

**Payload**

```
@Data
public class Payload <T>{
    private String id;
    private T userInfo;
    private Date expiration;
}

```

**JsonUtils**

```
public class JsonUtils {

    public static final ObjectMapper mapper = new ObjectMapper();

    private static final Logger logger = LoggerFactory.getLogger(JsonUtils.class);

    public static String toString(Object obj) {
        if (obj == null) {
            return null;
        }
        if (obj.getClass() == String.class) {
            return (String) obj;
        }
        try {
            return mapper.writeValueAsString(obj);
        } catch (JsonProcessingException e) {
            logger.error("json序列化出错：" + obj, e);
            return null;
        }
    }

    public static <T> T toBean(String json, Class<T> tClass) {
        try {
            return mapper.readValue(json, tClass);
        } catch (IOException e) {
            logger.error("json解析出错：" + json, e);
            return null;
        }
    }

    public static <E> List<E> toList(String json, Class<E> eClass) {
        try {
            return mapper.readValue(json, mapper.getTypeFactory().constructCollectionType(List.class, eClass));
        } catch (IOException e) {
            logger.error("json解析出错：" + json, e);
            return null;
        }
    }

    public static <K, V> Map<K, V> toMap(String json, Class<K> kClass, Class<V> vClass) {
        try {
            return mapper.readValue(json, mapper.getTypeFactory().constructMapType(Map.class, kClass, vClass));
        } catch (IOException e) {
            logger.error("json解析出错：" + json, e);
            return null;
        }
    }

    public static <T> T nativeRead(String json, TypeReference<T> type) {
        try {
            return mapper.readValue(json, type);
        } catch (IOException e) {
            logger.error("json解析出错：" + json, e);
            return null;
        }
    }
}

```

**JwtUtils**

```
public class JwtUtils {

    private static final String JWT_PAYLOAD_USER_KEY = "user";

    /**
     * 私钥加密token
     *
     * @param userInfo 载荷中的数据
     * @param privateKey 私钥
     * @param expire 过期时间，单位分钟
     * @return JWT
     */
    public static String generateTokenExpireInMinutes(Object userInfo, PrivateKey privateKey, int expire) {
        return Jwts.builder()
                .claim(JWT_PAYLOAD_USER_KEY, JsonUtils.toString(userInfo))
                .setId(createJTI())
                .setExpiration(DateTime.now().plusMinutes(expire).toDate())
                .signWith(privateKey, SignatureAlgorithm.RS256)
                .compact();
    }

    /**
     * 私钥加密token
     *
     * @param userInfo 载荷中的数据
     * @param privateKey 私钥
     * @param expire 过期时间，单位秒
     * @return JWT
     */
    public static String generateTokenExpireInSeconds(Object userInfo, PrivateKey privateKey, int expire) {
        return Jwts.builder()
                .claim(JWT_PAYLOAD_USER_KEY, JsonUtils.toString(userInfo))
                .setId(createJTI())
                .setExpiration(DateTime.now().plusSeconds(expire).toDate())
                .signWith(privateKey, SignatureAlgorithm.RS256)
                .compact();
    }

    /**
     * 公钥解析token
     *
     * @param token 用户请求中的token
     * @param publicKey 公钥
     * @return Jws<Claims>
     */
    private static Jws<Claims> parserToken(String token, PublicKey publicKey) {
        return Jwts.parser().setSigningKey(publicKey).parseClaimsJws(token);
    }

    private static String createJTI() {
        return new String(Base64.getEncoder().encode(UUID.randomUUID().toString().getBytes()));
    }

    /**
     * 获取token中的用户信息
     *
     * @param token 用户请求中的令牌
     * @param publicKey 公钥
     * @return 用户信息
     */
    public static <T> Payload<T> getInfoFromToken(String token, PublicKey publicKey, Class<T> userType) {
        Jws<Claims> claimsJws = parserToken(token, publicKey);
        Claims body = claimsJws.getBody();
        Payload<T> claims = new Payload<>();
        claims.setId(body.getId());
        claims.setUserInfo(JsonUtils.toBean(body.get(JWT_PAYLOAD_USER_KEY).toString(), userType));
        claims.setExpiration(body.getExpiration());
        return claims;
    }

    /**
     * 获取token中的载荷信息
     *
     * @param token 用户请求中的令牌
     * @param publicKey 公钥
     * @return 用户信息
     */
    public static <T> Payload<T> getInfoFromToken(String token, PublicKey publicKey) {
        Jws<Claims> claimsJws = parserToken(token, publicKey);
        Claims body = claimsJws.getBody();
        Payload<T> claims = new Payload<>();
        claims.setId(body.getId());
        claims.setExpiration(body.getExpiration());
        return claims;
    }
}

```

**RsaUtils**

```
public class RsaUtils {

    private static final int DEFAULT_KEY_SIZE = 2048;
    /**
     * 从文件中读取公钥
     *
     * @param filename 公钥保存路径，相对于classpath
     * @return 公钥对象
     * @throws Exception
     */
    public static PublicKey getPublicKey(String filename) throws Exception {
        byte[] bytes = readFile(filename);
        return getPublicKey(bytes);
    }

    /**
     * 从文件中读取密钥
     *
     * @param filename 私钥保存路径，相对于classpath
     * @return 私钥对象
     * @throws Exception
     */
    public static PrivateKey getPrivateKey(String filename) throws Exception {
        byte[] bytes = readFile(filename);
        return getPrivateKey(bytes);
    }

    /**
     * 获取公钥
     *
     * @param bytes 公钥的字节形式
     * @return
     * @throws Exception
     */
    private static PublicKey getPublicKey(byte[] bytes) throws Exception {
        bytes = Base64.getDecoder().decode(bytes);
        X509EncodedKeySpec spec = new X509EncodedKeySpec(bytes);
        KeyFactory factory = KeyFactory.getInstance("RSA");
        return factory.generatePublic(spec);
    }

    /**
     * 获取密钥
     *
     * @param bytes 私钥的字节形式
     * @return
     * @throws Exception
     */
    private static PrivateKey getPrivateKey(byte[] bytes) throws NoSuchAlgorithmException, InvalidKeySpecException {
        bytes = Base64.getDecoder().decode(bytes);
        PKCS8EncodedKeySpec spec = new PKCS8EncodedKeySpec(bytes);
        KeyFactory factory = KeyFactory.getInstance("RSA");
        return factory.generatePrivate(spec);
    }

    /**
     * 根据密文，生存rsa公钥和私钥,并写入指定文件
     *
     * @param publicKeyFilename 公钥文件路径
     * @param privateKeyFilename 私钥文件路径
     * @param secret 生成密钥的密文
     */
    public static void generateKey(String publicKeyFilename, String privateKeyFilename, String secret, int keySize) throws Exception {
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("RSA");
        SecureRandom secureRandom = new SecureRandom(secret.getBytes());
        keyPairGenerator.initialize(Math.max(keySize, DEFAULT_KEY_SIZE), secureRandom);
        KeyPair keyPair = keyPairGenerator.genKeyPair();
        // 获取公钥并写出
        byte[] publicKeyBytes = keyPair.getPublic().getEncoded();
        publicKeyBytes = Base64.getEncoder().encode(publicKeyBytes);
        writeFile(publicKeyFilename, publicKeyBytes);
        // 获取私钥并写出
        byte[] privateKeyBytes = keyPair.getPrivate().getEncoded();
        privateKeyBytes = Base64.getEncoder().encode(privateKeyBytes);
        writeFile(privateKeyFilename, privateKeyBytes);
    }

    private static byte[] readFile(String fileName) throws Exception {
        return Files.readAllBytes(new File(fileName).toPath());
    }

    private static void writeFile(String destPath, byte[] bytes) throws IOException {
        File dest = new File(destPath);
        if (!dest.exists()) {
            dest.createNewFile();
        }
        Files.write(dest.toPath(), bytes);
    }
}

```

**在通用子模块中编写测试类生成 rsa 公钥和私钥**

```
public class JwtTest {
    private String privateKey = "c:/tools/auth_key/id_key_rsa";

    private String publicKey = "c:/tools/auth_key/id_key_rsa.pub";

    @Test
    public void test1() throws Exception{
        RsaUtils.generateKey(publicKey,privateKey,"dpb",1024);
    }

}

```

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufyg01nHIrpMGoMrsthhZHxI3F9TLNYbSW7HATGyOibk7ibCpCar4EXS4miasRtjWZjfribpj3ZicP5HOA/640?wx_fmt=png)

#### 2.3 认证系统创建

接下来我们创建我们的认证服务。

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufyg01nHIrpMGoMrsthhZHxvlgm3CWZEicickr2unCKgMysEFxjpcXDXcYcslHJRL0IIa0g1V4CyWbw/640?wx_fmt=png)

**导入相关的依赖**

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <artifactId>security-jwt-common</artifactId>
        <groupId>com.dpb</groupId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.47</version>
    </dependency>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.1.0</version>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.1.10</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>

```

**创建配置文件**

```
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/srm
    username: root
    password: 123456
    type: com.alibaba.druid.pool.DruidDataSource
mybatis:
  type-aliases-package: com.dpb.domain
  mapper-locations: classpath:mapper/*.xml
logging:
  level:
    com.dpb: debug
rsa:
  key:
    pubKeyFile: c:\tools\auth_key\id_key_rsa.pub
    priKeyFile: c:\tools\auth_key\id_key_rsa

```

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufyg01nHIrpMGoMrsthhZHxp4b0gmPUQRKgMxD1icbp2dJcP2nxTbEEeKtenPEwCjYRoNcdZWUue5A/640?wx_fmt=png)

**提供公钥私钥的配置类**

```
@Data
@ConfigurationProperties(prefix = "rsa.key")
public class RsaKeyProperties {

    private String pubKeyFile;
    private String priKeyFile;

    private PublicKey publicKey;
    private PrivateKey privateKey;

    /**
     * 系统启动的时候触发
     * @throws Exception
     */
    @PostConstruct
    public void createRsaKey() throws Exception {
        publicKey = RsaUtils.getPublicKey(pubKeyFile);
        privateKey = RsaUtils.getPrivateKey(priKeyFile);
    }

}

```

**创建启动类**

```
@SpringBootApplication
@MapperScan("com.dpb.mapper")
@EnableConfigurationProperties(RsaKeyProperties.class)
public class App {

    public static void main(String[] args) {
        SpringApplication.run(App.class,args);
    }
}

```

**完成数据认证的逻辑**

pojo

```
@Data
public class RolePojo implements GrantedAuthority {

    private Integer id;
    private String roleName;
    private String roleDesc;

    @JsonIgnore
    @Override
    public String getAuthority() {
        return roleName;
    }
}

```

```
@Data
public class UserPojo implements UserDetails {

    private Integer id;

    private String username;

    private String password;

    private Integer status;

    private List<RolePojo> roles;

    @JsonIgnore
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        List<SimpleGrantedAuthority> auth = new ArrayList<>();
        auth.add(new SimpleGrantedAuthority("ADMIN"));
        return auth;
    }

    @Override
    public String getPassword() {
        return this.password;
    }

    @Override
    public String getUsername() {
        return this.username;
    }
    @JsonIgnore
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }
    @JsonIgnore
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }
    @JsonIgnore
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }
    @JsonIgnore
    @Override
    public boolean isEnabled() {
        return true;
    }
}

```

**Mapper 接口**

```
public interface UserMapper {
    public UserPojo queryByUserName(@Param("userName") String userName);
}

```

**Mapper 映射文件**

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.dpb.mapper.UserMapper">
    <select id="queryByUserName" resultType="UserPojo">
        select * from t_user where username = #{userName}
    </select>
</mapper>

```

**Service**

```
public interface UserService extends UserDetailsService {

}

```

```
@Service
@Transactional
public class UserServiceImpl implements UserService {

    @Autowired
    private UserMapper mapper;

    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        UserPojo user = mapper.queryByUserName(s);

        return user;
    }
}

```

**自定义认证过滤器**

```
public class TokenLoginFilter extends UsernamePasswordAuthenticationFilter {

    private AuthenticationManager authenticationManager;
    private RsaKeyProperties prop;

    public TokenLoginFilter(AuthenticationManager authenticationManager, RsaKeyProperties prop) {
        this.authenticationManager = authenticationManager;
        this.prop = prop;
    }

    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        try {
            UserPojo sysUser = new ObjectMapper().readValue(request.getInputStream(), UserPojo.class);

            UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(sysUser.getUsername(), sysUser.getPassword());
            return authenticationManager.authenticate(authRequest);
        }catch (Exception e){
            try {
                response.setContentType("application/json;charset=utf-8");
                response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                PrintWriter out = response.getWriter();
                Map resultMap = new HashMap();
                resultMap.put("code", HttpServletResponse.SC_UNAUTHORIZED);
                resultMap.put("msg", "用户名或密码错误！");
                out.write(new ObjectMapper().writeValueAsString(resultMap));
                out.flush();
                out.close();
            }catch (Exception outEx){
                outEx.printStackTrace();
            }
            throw new RuntimeException(e);
        }
    }

    public void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException, ServletException {
        UserPojo user = new UserPojo();
        user.setUsername(authResult.getName());
        user.setRoles((List<RolePojo>)authResult.getAuthorities());
        String token = JwtUtils.generateTokenExpireInMinutes(user, prop.getPrivateKey(), 24 * 60);
        response.addHeader("Authorization", "Bearer "+token);
        try {
            response.setContentType("application/json;charset=utf-8");
            response.setStatus(HttpServletResponse.SC_OK);
            PrintWriter out = response.getWriter();
            Map resultMap = new HashMap();
            resultMap.put("code", HttpServletResponse.SC_OK);
            resultMap.put("msg", "认证通过！");
            out.write(new ObjectMapper().writeValueAsString(resultMap));
            out.flush();
            out.close();
        }catch (Exception outEx){
            outEx.printStackTrace();
        }
    }
}

```

**自定义校验 token 的过滤器**

```
public class TokenVerifyFilter  extends BasicAuthenticationFilter {
    private RsaKeyProperties prop;

    public TokenVerifyFilter(AuthenticationManager authenticationManager, RsaKeyProperties prop) {
        super(authenticationManager);
        this.prop = prop;
    }

    public void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {
        String header = request.getHeader("Authorization");
        if (header == null || !header.startsWith("Bearer ")) {
            //如果携带错误的token，则给用户提示请登录！
            chain.doFilter(request, response);
            response.setContentType("application/json;charset=utf-8");
            response.setStatus(HttpServletResponse.SC_FORBIDDEN);
            PrintWriter out = response.getWriter();
            Map resultMap = new HashMap();
            resultMap.put("code", HttpServletResponse.SC_FORBIDDEN);
            resultMap.put("msg", "请登录！");
            out.write(new ObjectMapper().writeValueAsString(resultMap));
            out.flush();
            out.close();
        } else {
            //如果携带了正确格式的token要先得到token
            String token = header.replace("Bearer ", "");
            //验证tken是否正确
            Payload<UserPojo> payload = JwtUtils.getInfoFromToken(token, prop.getPublicKey(), UserPojo.class);
            UserPojo user = payload.getUserInfo();
            if(user!=null){
                UsernamePasswordAuthenticationToken authResult = new UsernamePasswordAuthenticationToken(user.getUsername(), null, user.getAuthorities());
                SecurityContextHolder.getContext().setAuthentication(authResult);
                chain.doFilter(request, response);
            }
        }
    }

}

```

**编写 SpringSecurity 的配置类**

```
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(securedEnabled=true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private UserService userService;

    @Autowired
    private RsaKeyProperties prop;

    @Bean
    public BCryptPasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    //指定认证对象的来源
    public void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userService).passwordEncoder(passwordEncoder());
    }
    //SpringSecurity配置信息
    public void configure(HttpSecurity http) throws Exception {
        http.csrf()
                .disable()
                .authorizeRequests()
                .antMatchers("/user/query").hasAnyRole("ADMIN")
                .anyRequest()
                .authenticated()
                .and()
                .addFilter(new TokenLoginFilter(super.authenticationManager(), prop))
                .addFilter(new TokenVerifyFilter(super.authenticationManager(), prop))
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);
    }
}

```

**启动服务测试**

启动服务

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufyg01nHIrpMGoMrsthhZHx0ibRulwxVLtuuOp6tepzn0XXiartqnXfpiaxzWlkezTfKyhJXcf3keGhw/640?wx_fmt=png)

通过 Postman 来访问测试

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufyg01nHIrpMGoMrsthhZHxYGftwlv3Pk5TNjaLS1Y2A71VPWBBbc4wYzNwRHibDUU3kJq5gsvLYBw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufyg01nHIrpMGoMrsthhZHxM8qLUHBb42rtR1VuPTTy1TBgpPLKkE7YBuBHrbjTSibNHnWibX6Ka4rQ/640?wx_fmt=png)

根据 token 信息我们访问其他资源

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufyg01nHIrpMGoMrsthhZHxbQZZTLbn4ZVu9n1mnyDuMoatXicKk2T7BfoRb2vXmS9M3alz8wth5eQ/640?wx_fmt=png)

#### 2.4 资源系统创建

**说明**

资源服务可以有很多个，这里只拿产品服务为例，记住，资源服务中只能通过公钥验证认证。不能签发 token！创建产品服务并导入 jar 包根据实际业务导包即可，咱们就暂时和认证服务一样了。

接下来我们再创建一个资源服务

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufyg01nHIrpMGoMrsthhZHxeGXGQQdKVM5ia4Pa45UUJc07TRqaGgxhpjR1O7uguiaBgPrOx1LLWiabA/640?wx_fmt=png)

**导入相关的依赖**

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <artifactId>security-jwt-common</artifactId>
        <groupId>com.dpb</groupId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.47</version>
    </dependency>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.1.0</version>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.1.10</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>

```

**编写产品服务配置文件**  

切记这里只能有公钥地址！

```
server:
  port: 9002
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/srm
    username: root
    password: 123456
    type: com.alibaba.druid.pool.DruidDataSource
mybatis:
  type-aliases-package: com.dpb.domain
  mapper-locations: classpath:mapper/*.xml
logging:
  level:
    com.dpb: debug
rsa:
  key:
    pubKeyFile: c:\tools\auth_key\id_key_rsa.pub

```

**编写读取公钥的配置类**

```
@Data
@ConfigurationProperties(prefix = "rsa.key")
public class RsaKeyProperties {

    private String pubKeyFile;

    private PublicKey publicKey;

    /**
     * 系统启动的时候触发
     * @throws Exception
     */
    @PostConstruct
    public void createRsaKey() throws Exception {
        publicKey = RsaUtils.getPublicKey(pubKeyFile);
    }

}

```

**编写启动类**

```
@SpringBootApplication
@MapperScan("com.dpb.mapper")
@EnableConfigurationProperties(RsaKeyProperties.class)
public class App {

    public static void main(String[] args) {
        SpringApplication.run(App.class,args);
    }
}

```

**复制认证服务中，用户对象，角色对象和校验认证的接口**

复制认证服务中的相关内容即可

**复制认证服务中 SpringSecurity 配置类做修改**

```
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(securedEnabled=true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private UserService userService;

    @Autowired
    private RsaKeyProperties prop;

    @Bean
    public BCryptPasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    //指定认证对象的来源
    public void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userService).passwordEncoder(passwordEncoder());
    }
    //SpringSecurity配置信息
    public void configure(HttpSecurity http) throws Exception {
        http.csrf()
                .disable()
                .authorizeRequests()
                //.antMatchers("/user/query").hasAnyRole("USER")
                .anyRequest()
                .authenticated()
                .and()
                .addFilter(new TokenVerifyFilter(super.authenticationManager(), prop))
                // 禁用掉session
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);
    }
}

```

去掉 “增加自定义认证过滤器” 即可！

**编写产品处理器**

```
@RestController
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/query")
    public String query(){
        return "success";
    }

    @RequestMapping("/update")
    public String update(){
        return "update";
    }
}

```

**测试**

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufyg01nHIrpMGoMrsthhZHxSYXTVDibux4iccyfRJBIPtDHoRPmXyM4XGjwEpBHboRTLzTqCtic6ETNA/640?wx_fmt=png)

作者：波波烤鸭

dpb-bobokaoya-sm.blog.csdn.net/article/details/103409430

推荐：[GitHub 上有哪些好玩的开源项目？](http://mp.weixin.qq.com/s?__biz=MzUxNjg4NDEzNA==&mid=2247498662&idx=1&sn=0087c4f3b79ba3420e917e9b42d45eda&chksm=f9a2286fced5a1794eb9a73d0be7c2e16eaceabf3a0420647c40cb4202bd116d9a15dd57c008&scene=21#wechat_redirect)