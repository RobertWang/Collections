> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/f1224682a425)

> skywalking 是分布式微服务请求链路跟踪的框架，可以实现无侵入的链路跟踪、统计、拓扑架构绘制等，本文介绍如何快速搭建

下载 & 安装
-------

[https://skywalking.apache.org/downloads/](https://links.jianshu.com/go?to=https%3A%2F%2Fskywalking.apache.org%2Fdownloads%2F)

可以选择下载: [https://archive.apache.org/dist/skywalking/8.7.0/apache-skywalking-apm-8.7.0.tar.gz](https://links.jianshu.com/go?to=https%3A%2F%2Farchive.apache.org%2Fdist%2Fskywalking%2F8.7.0%2Fapache-skywalking-apm-8.7.0.tar.gz)

Spring Boot 接入
--------------

对代码没有任何侵入，连 jar 包都不需要引入，只需要修改启动命令就可以了

```
-javaagent:apache-skywalking-apm-bin-es7/agent/skywalking-agent.jar -Dskywalking.agent.service_name=tenmao-mybatis -Dskywalking.collector.backend_service=localhost:8080


```

查看接入效果
------

[http://localhost:8080/](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A8080%2F)

![](http://upload-images.jianshu.io/upload_images/7081994-901017eaca38f241.png) image.png

代码中获取 traceId
-------------

*   添加依赖

```
<dependency>
    <groupId>org.apache.skywalking</groupId>
    <artifactId>apm-toolkit-trace</artifactId>
    <version>8.7.0</version>
    <scope>provided</scope>
</dependency>


```

*   获取 traceId

```
String traceId = TraceContext.traceId();


```

日志中输出 traceId
-------------

*   添加依赖

```
<dependency>
    <groupId>org.apache.skywalking</groupId>
    <artifactId>apm-toolkit-logback-1.x</artifactId>
    <version>8.7.0</version>
</dependency>


```

*   日志配置  
    encoder 使用 ch.qos.logback.core.encoder.LayoutWrappingEncoder  
    layout 使用

```
<appender >
    <!-- encoder 不能使用默认的PatternLayoutEncoder -->
    <encoder>
         <!-- 使用skywalking的TraceIdPatternLogbackLayout-->
        <layout>
            <pattern>${commonPattern}</pattern>
        </layout>
        <charset>UTF-8</charset>
    </encoder>
</appender>


```

常见错误
----

*   `First received frame was not SETTINGS. Hex dump for first 5 bytes: 485454502f`
    *   端口不匹配，可以尝试使用默认端口 11800