> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIyNTY4NjU0OQ==&mid=2247506638&idx=3&sn=a8d2870e061e32d6f911044c7891cf8a&chksm=e8797fb4df0ef6a255d5d5a3049b5d2399f62e141d7b839b62747f2adaaf8819300f8ade1734&mpshare=1&scene=1&srcid=0605rF95Wfifv7O0Euzgz2Dd&sharer_sharetime=1622826526146&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

  

**背景**  

Web 开发中，通过 Session 在服务端记录用户状态是很常见的操作。对于 Web 开发中 Session、Cookie 等概念请参考《Session 机制详解》。但是 Session 的机制对于单机应用是没问题的，但是对于集群环境，由于在将请求分配到另一台服务器时，新的服务器无法通过浏览器传入的 Cookie 值取到 Session，所以导致所有基于 Session 的操作都会失败，如：登录状态。

 本文通过搭建一个非常简易的集群环境，来演示 Session 机制在集群环境中存在的问题，并通过 Redis 进行 Session 共享来解决该问题。

**一、问题再现**

1、测试环境

（1）App Server

使用 Spring Boot 2 写一个简单的 Web 应用，提供两个链接：

```
@RestController
public class TestController {

@GetMapping("/set-session")
public Object writeSession(String sessionVal, HttpSession httpSession) {
System.out.println("Param 'sessionVal' = " + sessionVal);
httpSession.setAttribute("sessionVal", sessionVal);
return sessionVal;
}

@GetMapping("/get-session")
public Object readSession(HttpSession httpSession) {
Object obj = httpSession.getAttribute("sessionVal");
System.out.println("'sessionVal' in Session = " + obj);
return obj;
}

}
```

单机测试通过。

（2）通过 Nginx 做负载均衡  
分别在 9001 和 9002 两个端口启动 App Server，然后通过 Nginx 配置负载均衡，配置如下：

```
http {

upstream app_server {
server 127.0.0.1:9001;
server 127.0.0.1:9002;
}

server {
listen 9000;
location / {
proxy_pass http://app_server;
}
}
}
```

测试失败。  

**二、原因分析**

主要是因为原来 A 服务器将其 Session 的标识 Cookie_for_Session_A 放入浏览器 Cookie，当下一次请求被分配到 B 服务器，B 服务器无法通过  Cookie_for_Session_A 获取到对应的 Session，导致失败。

解决的思路，主要是引入三方服务器，将 Session 保存到三方服务器，A、B 服务器共享三方服务器中的 Session 数据。

**三、解决方案**

引入 Redis 作为三方服务器存储 Session 数据。

1、引入 Redis 相关库

```
dependencies {
implementation('org.springframework.boot:spring-boot-starter-web')
implementation('org.springframework.boot:spring-boot-starter-data-redis')

// https://mvnrepository.com/artifact/org.springframework.session/spring-session-data-redis
compile group: 'org.springframework.session', name: 'spring-session-data-redis', version: '2.1.2.RELEASE'

}
```

  

2、配置 Redis 连接  

application.yml，这里为了演示清晰，只做了最简配置，正式使用请调整相关参数

```
spring:
redis:
host: 127.0.0.1
port: 6379
```

  

3、开启配置  

创建一个配置类 SessionConfig，类名随意。

关键是两个注解：

```
@Configuration
@EnableRedisHttpSession



@Configuration
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 86400*30)
public class SessionConfig {

}
```

  

4、打包、运行测试  

执行 Gradle 的 bootJar 任务，然后按照前面的方式，分别在 9001 和 9002 端口运行 jar 包：

```
java -jar redis-session.jar
java -jar redis-session.jar
```

  

测试通过。

 来源：https://blog.51cto.com/13981400/2367775