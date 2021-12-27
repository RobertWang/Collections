> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/eT645choeJACFL-s1IWQkA)

> 在 SpringBoot 中，模板引擎的页面默认是开启缓存的，如果修改了页面的内容，则刷新页面

_2021-12-27 08:44_

**大家好，我是磊哥。**

[在 SpringBoot 中，模板引擎的页面默认是开启缓存的，如果修改了页面的内容，则刷新页面是得不到修改后的页面的，因此我们可以在 application.properties 中关闭模版引擎的缓存，如下：](https://mp.weixin.qq.com/s?__biz=MzI4NTM1NDgwNw==&mid=2247500365&idx=1&sn=7abfd8b30427d4ac15bb351c3dbb640c&scene=21#wechat_redirect)

1、模板热部署
-------

Thymeleaf 的配置：

```
spring.thymeleaf.cache=false


```

FreeMarker 的配置：

```
spring.freemarker.cache=false  


```

Groovy 的配置：

```
spring.groovy.template.cache=false


```

Velocity 的配置：

```
spring.velocity.cache=false


```

2、使用调试模式 Debug 实现热部署
--------------------

[**注 意**  
](https://mp.weixin.qq.com/s?__biz=MzI4NTM1NDgwNw==&mid=2247500365&idx=1&sn=7abfd8b30427d4ac15bb351c3dbb640c&scene=21#wechat_redirect)

 [**文末有：7701 页互联网大厂面试题**](https://mp.weixin.qq.com/s?__biz=MzI4NTM1NDgwNw==&mid=2247500365&idx=1&sn=7abfd8b30427d4ac15bb351c3dbb640c&scene=21#wechat_redirect) 

此种方式为最简单最快速的一种热部署方式，运行系统时使用 Debug 模式，无需装任何插件即可，但是无发对配置文件，方法名称改变，增加类及方法进行热部署，使用范围有限。

3、spring-boot-devtools
----------------------

在 Spring Boot 项目中添加 spring-boot-devtools 依赖即可实现页面和代码的热部署。如下：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>


```

[此种方式的特点是作用范围广，系统的任何变动包括配置文件修改、方法名称变化都能覆盖，但是后遗症也非常明显，它是采用文件变化后重启的策略来实现了，主要是节省了我们手动点击重启的时间，提高了实效性，在体验上回稍差。](https://mp.weixin.qq.com/s?__biz=MzI4NTM1NDgwNw==&mid=2247500365&idx=1&sn=7abfd8b30427d4ac15bb351c3dbb640c&scene=21#wechat_redirect)

[spring-boot-devtools 默认关闭了模版缓存，如果使用这种方式不用单独配置关闭模版缓存。](https://mp.weixin.qq.com/s?__biz=MzI4NTM1NDgwNw==&mid=2247500365&idx=1&sn=7abfd8b30427d4ac15bb351c3dbb640c&scene=21#wechat_redirect)

4、Spring Loaded
---------------

[此种方式与 Debug 模式类似，适用范围有限，但是不依赖于 Debug 模式启动，通过 Spring Loaded 库文件启动，即可在正常模式下进行实时热部署。此种需要在 run confrgration 中进行配置。](https://mp.weixin.qq.com/s?__biz=MzI4NTM1NDgwNw==&mid=2247500365&idx=1&sn=7abfd8b30427d4ac15bb351c3dbb640c&scene=21#wechat_redirect)

5、JRebel
--------

Jrebel 是 Java 开发最好的热部署工具，对 Spring Boot 提供了极佳的支持，JRebel 为收费软件，试用期 14 天。，可直接通过插件安装。