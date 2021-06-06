> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5OTA1MDUyMA==&mid=2655463072&idx=2&sn=5cd23d946868b91362c378fa24401ceb&chksm=bd72ead78a0563c1a8fcfb8228f5d2a1b7fe47beea36a70bbdce5285d3b409efecdbeceabb3a&mpshare=1&scene=1&srcid=0605vacggqjYZjkJSLXjqpa0&sharer_sharetime=1622826664861&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

最近，**Spring Boot 2.5** 正式发布了，支持 **Java 16**，增强了 **Docker** 镜像构建功能，而且提供了初始化数据源的新机制。

**基于 Spring Boot 2.4 的变动**
--------------------------

### Sql 脚本初始化数据源

在 **Spring Boot 2.5** 中已经重新设计了用于支持`schema.sql`和`data.sql`编写脚本的基础方法。`spring.datasource.*`中和数据源初始化相关的配置已经过时，会被`spring.sql.init.*`系列配置所代替，而且新的配置对 **R2DBC** 也适用。需要注意的是目前不支持分离许可证（`separate credentials`），目的是降低复杂度并同 **Flyway** 和 **Liquibase** 保持一致性兼容。当然你可以通过自行实现

```
org.springframework.jdbc.datasource.init.DataSourceInitializer
```

来扩展。

### 环境变量前缀

现在可以为系统环境变量`SystemEnvironmentPropertySource`指定前缀，以便您可以在同一环境中运行多个不同的 Spring Boot 应用程序时使用

```
SpringApplication.setEnvironmentPrefix("PRIFIX")
```

例如：

```
SpringApplication application = new SpringApplication(MyApp.class);
application.setEnvironmentPrefix("myapp");
application.run(args);
```

当你需要针对特定的应用改变系统变量时，如`OS`, 就可以声明为`MYAPP_OS`、`MYAPP-OS`或者`MYAPP.OS`。

❝注意不是`application.yaml`中的配置。

### HTTP/2 支持

现在 **Spring Boot** 内置的四种 Web 容器已经在不需要任何自定义的情况下，支持 **HTTP/2 over TCP**。设置`server.http2.enabled`为 `true`，`server.ssl.enabled`为`false`即可生效。

### Docker 镜像

#### War 分层镜像

现在 **Spring Boot** 也能打成 **war** 包装进 Docker 镜像了，而且支持分层构建。

#### buildpacks

如果你使用 buildpacks 构建镜像，你可以将其配置属性文件放到一个目录下或者`tar.gz`文件中。卷（`volume` ）绑定现在也支持 buildpacks 构建器了。

### 度量指标

现在 **Spring Boot** 支持 **OpenMetrics for Prometheus**、**Spring Data Repositories**、**WebFlux**、**MongoDB** 、**Quartz** 的度量指标监控。

### 依赖升级

以下依赖升级到新版本

*   Spring Data 2021.0
    
*   Spring Integration 5.5
    
*   Spring Security 5.5
    
*   Spring Session 2021.0
    
*   Spring HATEOAS 1.3
    
*   Spring Kafka 2.7.0
    

### 过期依赖移除

**Spring Boot 2.5** 已删除了 **Spring Boot 2.3** 中不推荐使用的代码。**Spring Boot 2.4** 不推荐使用的代码目前保留，并计划在 **Spring Boot 2.6** 中将其删除。

❝不推荐使用的代码即`@Deprecated`标记的 API。

### 文档优化

**Spring Boot** 文档史诗级优化，界面更新颖漂亮，字体更加清晰，暗黑主题，代码折叠，代码剪切板都有了！

### 其它

其实还有其它一些细节改动和优化，基于篇幅就不多介绍了，有兴趣可以查看官方文档了解。

推荐阅读  点击标题可跳转

1、[Spring 官宣，干掉原生 JVM！](http://mp.weixin.qq.com/s?__biz=MjM5OTA1MDUyMA==&mid=2655462637&idx=2&sn=a4af77fc20b471763c8a89334b3a4ab6&chksm=bd72e89a8a05618c60560d0f12ef60dd24c17a5aa66b5b8775d700981a2a09d8316592c0ad57&scene=21#wechat_redirect)

2、[Spring 的 Controller 是单例还是多例？怎么保证并发的安全](http://mp.weixin.qq.com/s?__biz=MjM5OTA1MDUyMA==&mid=2655457631&idx=4&sn=3670052bca1132b1dc41f8157ce506b3&chksm=bd72dd288a05543e549968175bcbd992ce707450966139da21e489d38c497e3b1050943c6414&scene=21#wechat_redirect)

3、[25 张图，一万字，拆解 Linux 网络包发送过程](http://mp.weixin.qq.com/s?__biz=MjM5OTA1MDUyMA==&mid=2655463003&idx=2&sn=400efe1717a9d1585a6e09a8f62130c1&chksm=bd72ea2c8a05633a1ca2000c0cd85a3f660447c5cf43a49c4f9347e9f36905045a96890fa8b6&scene=21#wechat_redirect)

  

公众号

点赞和在看就是最大的支持❤️