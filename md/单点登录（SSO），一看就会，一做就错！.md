> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzkzNzIzNzczMA==&mid=2247488551&idx=1&sn=c315e12789618e4638f1ab4ac5639a36&chksm=c293d9aaf5e450bce0c9931d3dfbe1268ed6e61164256c61c86f2a757d6d313ce683a3f1dc99&mpshare=1&scene=1&srcid=0605Oe6N7HR5piWw8ma9zz70&sharer_sharetime=1622826756683&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

公众号

技术这东西吧，看别人写的好像很简单似的，到自己去写的时候就各种问题，“一看就会，一做就错”。

几经曲折，终于搞定了，决定记录下来，以便后续查看。先来看一下效果：  

![](https://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQwcJSPwoDXGLMVr25039CMwtpw9gAQm10YEXfL5hhuPhxL2hzjzqZnxWMUD0YyH4M0XPf0yJo7XWA/640?wx_fmt=jpeg)  

![](https://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQwcJSPwoDXGLMVr25039CMwTl9GVzC4ERmpCek1icciaxGcfzCGlV0IvRcFdW4InvcYu7QTU4aWZMjA/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQwcJSPwoDXGLMVr25039CMwHNFxEIOIheKD5DgOJDUABfM5h8H2AOCK653PlOJ4Ew0GtgJt8k1icwQ/640?wx_fmt=jpeg)

准备

**①单点登录**

最常见的例子是，我们打开淘宝 APP，首页就会有天猫、聚划算等服务的链接，当你点击以后就直接跳过去了，并没有让你再登录一次。

![](https://mmbiz.qpic.cn/mmbiz_jpg/eZzl4LXykQwcJSPwoDXGLMVr25039CMwkEreE82C1JMx6AfwXdjn0A0BnuALriaY0N4EY3bw5XkFQl8cdfrBSGQ/640?wx_fmt=jpeg)

下面这个图是我在网上找的，我觉得画得比较明白：

![](https://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQwcJSPwoDXGLMVr25039CMwvGBXfysEgooO3bVcUY1kWiavhQhuibiaeTZbSWnmwp6ufciaVeZHwmWDiag/640?wx_fmt=jpeg)

可惜有点儿不清晰，于是我又画了个简版的：

![](https://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQwcJSPwoDXGLMVr25039CMwqzSNgm9Kt7bHOAFmEj3la6W81o7mY8oxBHa3YQSCfQiacMMwc6pR1Yg/640?wx_fmt=jpeg)

重要的是理解：  

*   SSO 服务端和 SSO 客户端直接是通过授权以后发放 Token 的形式来访问受保护的资源。
    
*   相对于浏览器来说，业务系统是服务端，相对于 SSO 服务端来说，业务系统是客户端。
    
*   浏览器和业务系统之间通过会话正常访问。
    
*   不是每次浏览器请求都要去 SSO 服务端去验证，只要浏览器和它所访问的服务端的会话有效它就可以正常访问。  
    

利用 OAuth2 实现单点登录

接下来，只讲跟本例相关的一些配置，不讲原理，不讲为什么。

众所周知，在 OAuth2 在有授权服务器、资源服务器、客户端这样几个角色，当我们用它来实现 SSO 的时候是不需要资源服务器这个角色的，有授权服务器和客户端就够了。

授权服务器当然是用来做认证的，客户端就是各个应用系统，我们只需要登录成功后拿到用户信息以及用户所拥有的权限即可。

之前我一直认为把那些需要权限控制的资源放到资源服务器里保护起来就可以实现权限控制，其实是我想错了，权限控制还得通过 Spring Security 或者自定义拦截器来做。

**①Spring Security 、OAuth2、JWT、SSO**

在本例中，一定要分清楚这几个的作用：

**首先，**SSO 是一种思想，或者说是一种解决方案，是抽象的，我们要做的就是按照它的这种思想去实现它。

**其次，**OAuth2 是用来允许用户授权第三方应用访问他在另一个服务器上的资源的一种协议，它不是用来做单点登录的，但我们可以利用它来实现单点登录。  

在本例实现 SSO 的过程中，受保护的资源就是用户的信息（包括，用户的基本信息，以及用户所具有的权限）。  

而我们想要访问这这一资源就需要用户登录并授权，OAuth2 服务端负责令牌的发放等操作，这令牌的生成我们采用 JWT，也就是说 JWT 是用来承载用户的 Access_Token 的。

最后，Spring Security 是用于安全访问的，这里我们我们用来做访问权限控制。

认证服务器配置

Maven 依赖：  

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.cjs.sso</groupId>
    <artifactId>oauth2-sso-auth-server</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>oauth2-sso-auth-server</name>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.security.oauth.boot</groupId>
            <artifactId>spring-security-oauth2-autoconfigure</artifactId>
            <version>2.1.3.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.session</groupId>
            <artifactId>spring-session-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.8.1</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.56</version>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

这里面最重要的依赖是：spring-security-oauth2-autoconfigure。

application.yml：  

```
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/permission
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
  jpa:
    show-sql: true
  session:
    store-type: redis
  redis:
    host: 127.0.0.1
    password: 123456
    port: 6379
server:
  port: 8080
```

AuthorizationServerConfig（重要）：  

```
package com.cjs.sso.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.security.core.token.DefaultToken;
import org.springframework.security.oauth2.config.annotation.configurers.ClientDetailsServiceConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableAuthorizationServer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerEndpointsConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerSecurityConfigurer;
import org.springframework.security.oauth2.provider.token.DefaultTokenServices;
import org.springframework.security.oauth2.provider.token.TokenStore;
import org.springframework.security.oauth2.provider.token.store.JwtAccessTokenConverter;
import org.springframework.security.oauth2.provider.token.store.JwtTokenStore;

import javax.sql.DataSource;

/**
 * @author ChengJianSheng
 * @date 2019-02-11
 */
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private DataSource dataSource;

    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security.allowFormAuthenticationForClients();
        security.tokenKeyAccess("isAuthenticated()");
    }

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.jdbc(dataSource);
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.accessTokenConverter(jwtAccessTokenConverter());
        endpoints.tokenStore(jwtTokenStore());
//        endpoints.tokenServices(defaultTokenServices());
    }

    /*@Primary
    @Bean
    public DefaultTokenServices defaultTokenServices() {
        DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
        defaultTokenServices.setTokenStore(jwtTokenStore());
        defaultTokenServices.setSupportRefreshToken(true);
        return defaultTokenServices;
    }*/

    @Bean
    public JwtTokenStore jwtTokenStore() {
        return new JwtTokenStore(jwtAccessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter jwtAccessTokenConverter = new JwtAccessTokenConverter();
        jwtAccessTokenConverter.setSigningKey("cjs");   //  Sets the JWT signing key
        return jwtAccessTokenConverter;
    }

}
```

说明：  

*   别忘了 @EnableAuthorizationServer。
    
*   Token 存储采用的是 JWT。
    
*   客户端以及登录用户这些配置存储在数据库，为了减少数据库的查询次数，可以从数据库读出来以后再放到内存中。
    

WebSecurityConfig（重要）：  

```
package com.cjs.sso.config;

import com.cjs.sso.service.MyUserDetailsService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

/**
 * @author ChengJianSheng
 * @date 2019-02-11
 */
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private MyUserDetailsService userDetailsService;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService).passwordEncoder(passwordEncoder());
    }

    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers("/assets/**", "/css/**", "/images/**");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()
                .loginPage("/login")
                .and()
                .authorizeRequests()
                .antMatchers("/login").permitAll()
                .anyRequest()
                .authenticated()
                .and().csrf().disable().cors();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

}
```

自定义登录页面（一般来讲都是要自定义的）：  

```
package com.cjs.sso.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

/**
 * @author ChengJianSheng
 * @date 2019-02-12
 */
@Controller
public class LoginController {

    @GetMapping("/login")
    public String login() {
        return "login";
    }

    @GetMapping("/")
    public String index() {
        return "index";
    }

}
```

自定义登录页面的时候，只需要准备一个登录页面，然后写个 Controller 令其可以访问到即可，登录页面表单提交的时候 method 一定要是 post，最重要的时候 action 要跟访问登录页面的 url 一样。

千万记住了，访问登录页面的时候是 GET 请求，表单提交的时候是 POST 请求，其他的就不用管了。  

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>Ela Admin - HTML5 Admin Template</title>
    <meta >
    <meta >

    <link type="text/css" rel="stylesheet" th:href="@{/assets/css/normalize.css}">
    <link type="text/css" rel="stylesheet" th:href="@{/assets/bootstrap-4.3.1-dist/css/bootstrap.min.css}">
    <link type="text/css" rel="stylesheet" th:href="@{/assets/css/font-awesome.min.css}">
    <link type="text/css" rel="stylesheet" th:href="@{/assets/css/style.css}">

</head>
<body class="bg-dark">

<div class="sufee-login d-flex align-content-center flex-wrap">
    <div class="container">
        <div class="login-content">
            <div class="login-logo">
                <h1 style="color: #57bf95;">欢迎来到王者荣耀</h1>
            </div>
            <div class="login-form">
                <form th:action="@{/login}" method="post">
                    <div class="form-group">
                        <label>Username</label>
                        <input type="text" class="form-control" >
                    </div>
                    <div class="form-group">
                        <label>Password</label>
                        <input type="password" class="form-control" >
                    </div>
                    <div class="checkbox">
                        <label>
                            <input type="checkbox"> Remember Me
                        </label>
                        <label class="pull-right">
                            <a href="#">Forgotten Password?</a>
                        </label>
                    </div>
                    <button type="submit" class="btn btn-success btn-flat m-b-30 m-t-30" style="font-size: 18px;">登录</button>
                </form>
            </div>
        </div>
    </div>
</div>


<script type="text/javascript" th:src="@{/assets/js/jquery-2.1.4.min.js}"></script>
<script type="text/javascript" th:src="@{/assets/bootstrap-4.3.1-dist/js/bootstrap.min.js}"></script>
<script type="text/javascript" th:src="@{/assets/js/main.js}"></script>

</body>
</html>
```

定义客户端，如下图：  

![](https://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQwcJSPwoDXGLMVr25039CMwZUT9P8Q69xiaicxsibbtBtgbpJwLEa6xFOFFy5LPyG7aYl1jmJ11uyK0A/640?wx_fmt=jpeg)  

![](https://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQwcJSPwoDXGLMVr25039CMwp1L9wcKvNWEa1Vv3UV7NUPZcAQxMY0ZbgljQSHFXiaN5XIeibjRbQU5A/640?wx_fmt=jpeg)

加载用户，登录账户：  

```
package com.cjs.sso.domain;

import lombok.Data;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.User;

import java.util.Collection;

/**
 * 大部分时候直接用User即可不必扩展
 * @author ChengJianSheng
 * @date 2019-02-11
 */
@Data
public class MyUser extends User {

    private Integer departmentId;   //  举个例子，部门ID

    private String mobile;  //  举个例子，假设我们想增加一个字段，这里我们增加一个mobile表示手机号

    public MyUser(String username, String password, Collection<? extends GrantedAuthority> authorities) {
        super(username, password, authorities);
    }

    public MyUser(String username, String password, boolean enabled, boolean accountNonExpired, boolean credentialsNonExpired, boolean accountNonLocked, Collection<? extends GrantedAuthority> authorities) {
        super(username, password, enabled, accountNonExpired, credentialsNonExpired, accountNonLocked, authorities);
    }
}
```

加载登录账户：  

```
package com.cjs.sso.service;

import com.alibaba.fastjson.JSON;
import com.cjs.sso.domain.MyUser;
import com.cjs.sso.entity.SysPermission;
import com.cjs.sso.entity.SysUser;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;
import org.springframework.util.CollectionUtils;

import java.util.ArrayList;
import java.util.List;

/**
 * @author ChengJianSheng
 * @date 2019-02-11
 */
@Slf4j
@Service
public class MyUserDetailsService implements UserDetailsService {

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Autowired
    private UserService userService;

    @Autowired
    private PermissionService permissionService;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        SysUser sysUser = userService.getByUsername(username);
        if (null == sysUser) {
            log.warn("用户{}不存在", username);
            throw new UsernameNotFoundException(username);
        }
        List<SysPermission> permissionList = permissionService.findByUserId(sysUser.getId());
        List<SimpleGrantedAuthority> authorityList = new ArrayList<>();
        if (!CollectionUtils.isEmpty(permissionList)) {
            for (SysPermission sysPermission : permissionList) {
                authorityList.add(new SimpleGrantedAuthority(sysPermission.getCode()));
            }
        }

        MyUser myUser = new MyUser(sysUser.getUsername(), passwordEncoder.encode(sysUser.getPassword()), authorityList);

        log.info("登录成功！用户: {}", JSON.toJSONString(myUser));

        return myUser;
    }
}
```

验证：  

![](https://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQwcJSPwoDXGLMVr25039CMwz4GJWdgDIkm17GswNGOTdfRRhsgibJxt3Ez67YFtKQnqDgf51JNLdhQ/640?wx_fmt=jpeg)

当我们看到这个界面的时候，表示认证服务器配置完成。 

两个客户端

Maven 依赖：  

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.cjs.sso</groupId>
    <artifactId>oauth2-sso-client-member</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>oauth2-sso-client-member</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-oauth2-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.security.oauth.boot</groupId>
            <artifactId>spring-security-oauth2-autoconfigure</artifactId>
            <version>2.1.3.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.thymeleaf.extras</groupId>
            <artifactId>thymeleaf-extras-springsecurity5</artifactId>
            <version>3.0.4.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

application.yml：  

```
server:
  port: 8082
  servlet:
    context-path: /memberSystem
security:
  oauth2:
    client:
      client-id: UserManagement
      client-secret: user123
      access-token-uri: http://localhost:8080/oauth/token
      user-authorization-uri: http://localhost:8080/oauth/authorize
    resource:
      jwt:
        key-uri: http://localhost:8080/oauth/token_key
```

![](https://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQwcJSPwoDXGLMVr25039CMwgh9uiaiaGSwNsepCy7zEjfPTXBMCNbibpWvBbDx9KvibcvvWcGvib3BibeLg/640?wx_fmt=jpeg)

这里 context-path 不要设成 /，不然重定向获取 code 的时候回被拦截。

WebSecurityConfig：  

```
package com.cjs.example.config;

import com.cjs.example.util.EnvironmentUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.security.oauth2.client.EnableOAuth2Sso;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;


/**
 * @author ChengJianSheng
 * @date 2019-03-03
 */
@EnableOAuth2Sso
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private EnvironmentUtils environmentUtils;

    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers("/bootstrap/**");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        if ("local".equals(environmentUtils.getActiveProfile())) {
            http.authorizeRequests().anyRequest().permitAll();
        }else {
            http.logout().logoutSuccessUrl("http://localhost:8080/logout")
                    .and()
                    .authorizeRequests()
                    .anyRequest().authenticated()
                    .and()
                    .csrf().disable();
        }
    }
}
```

说明：

*   最重要的注解是 @EnableOAuth2Sso。
    
*   权限控制这里采用 Spring Security 方法级别的访问控制，结合 Thymeleaf 可以很容易做权限控制。
    
*   顺便多提一句，如果是前后端分离的话，前端需求加载用户的权限，然后判断应该显示那些按钮那些菜单。
    

MemberController：  

```
package com.cjs.example.controller;

import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.core.Authentication;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import java.security.Principal;

/**
 * @author ChengJianSheng
 * @date 2019-03-03
 */
@Controller
@RequestMapping("/member")
public class MemberController {

    @GetMapping("/list")
    public String list() {

        return "member/list";
    }

    @GetMapping("/info")
    @ResponseBody
    public Principal info(Principal principal) {
        return principal;
    }

    @GetMapping("/me")
    @ResponseBody
    public Authentication me(Authentication authentication) {
        return authentication;
    }

    @PreAuthorize("hasAuthority('member:save')")
    @ResponseBody
    @PostMapping("/add")
    public String add() {

        return "add";
    }

    @PreAuthorize("hasAuthority('member:detail')")
    @ResponseBody
    @GetMapping("/detail")
    public String detail() {
        return "detail";
    }
}
```

Order 项目跟它是一样的：  

```
server:
  port: 8083
  servlet:
    context-path: /orderSystem
security:
  oauth2:
    client:
      client-id: OrderManagement
      client-secret: order123
      access-token-uri: http://localhost:8080/oauth/token
      user-authorization-uri: http://localhost:8080/oauth/authorize
    resource:
      jwt:
        key-uri: http://localhost:8080/oauth/token_key
```

**关于退出：**退出就是清空用于与 SSO 客户端建立的所有的会话，简单的来说就是使所有端点的 Session 失效，如果想做得更好的话可以令 Token 失效，但是由于我们用的 JWT，故而撤销 Token 就不是那么容易，关于这一点，在官网上也有提到：  

![](https://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQwcJSPwoDXGLMVr25039CMwthGkWf5x3lZzZSHjJqrUQE2ks7Lg6eTUmsojbiaSYCo8x6pCLIFLuSg/640?wx_fmt=jpeg)

本例中采用的方式是在退出的时候先退出业务服务器，成功以后再回调认证服务器，但是这样有一个问题，就是需要主动依次调用各个业务服务器的 logout。

工程结构

![](https://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQwcJSPwoDXGLMVr25039CMwI2TBFpRVQbfqneNw2e6M4w8mtshPK93fQ6icwGtNzXSX3qHHN3BWPaw/640?wx_fmt=jpeg)

附上源码：  

```
https://github.com/chengjiansheng/cjs-oauth2-sso-demo.git
```

演示

![](https://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQwcJSPwoDXGLMVr25039CMwX4BXOv3wUlTfuXprOVUf9vs1EP3Qex98pegLLcHtwue1fzic9iccxeVQ/640?wx_fmt=jpeg)

作者：废物大师兄

出处：cnblogs.com/cjsblog/p/10548022.html