> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU1Nzg4NjgyMw==&mid=2247492021&idx=1&sn=7f448c8ca94c05f9a19c10702f69cccd&scene=21#wechat_redirect)

> 功能强大，颜值爆表的 API 网关，必须推荐给你！

> 提到 API 网关，大家比较熟悉的有 Spring Cloud 体系中的 Gateway 和 Zuul，这些网关在使用的时候基本都要修改配置文件或自己开发功能。今天给大家介绍一款功能强大的 API 网关`apisix`，自带可视化管理功能，多达三十种插件支持，希望对大家有所帮助！

简介
--

apisix 是一款云原生微服务 API 网关，可以为 API 提供终极性能、安全性、开源和可扩展的平台。apisix 基于 Nginx 和 etcd 实现，与传统 API 网关相比，apisix 具有动态路由和插件热加载，特别适合微服务系统下的 API 管理。

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDY0ebbzCGTbP7H8EhEuuRGiaYnVT9t3tMRQmoVRGWtV9AC2BVR9mIWbuw/640?wx_fmt=png)

核心概念
----

> 我们先来了解下 apisix 的一些核心概念，对我们接下来的使用会很有帮助！

*   上游（Upstream）：可以理解为虚拟主机，对给定的多个目标服务按照配置规则进行负载均衡。
    
*   路由（Route）：通过定义一些规则来匹配客户端的请求，然后对匹配的请求执行配置的插件，并把请求转发给指定的上游。
    
*   消费者（Consumer）：作为 API 网关，有时需要知道 API 的消费方具体是谁，通常可以用来做身份认证。
    
*   服务（Service）：可以理解为一组路由的抽象。它通常与上游是一一对应的，路由与服务之间，通常是多对一的关系。
    
*   插件（Plugin）：API 网关对请求的增强操作，可以对请求增加限流、认证、黑名单等一系列功能。可以配置在消费者、服务和路由之上。
    

安装
--

> 由于官方提供了 Docker Compose 部署方案，只需一个脚本即可安装 apisix 的相关服务，非常方便，这里我们也采用这种方案来部署。

*   首先下载`apisix-docker`项目，其实我们只需要使用其中的`example`目录就行了，下载地址：https://github.com/apache/apisix-docker
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYHI7J8ewgjhPJ7rEtlic0TlQqGwdX7ljCCJK2RXbdI7L7Uz5iahAALMng/640?wx_fmt=png)

*   接下来我们把`example`目录上传到 Linux 服务器上去，来了解下这个目录里面的东西；
    

```
drwxrwxrwx. 2 root root   25 Jun 19 10:12 apisix_conf   # apisix配置文件目录
drwxrwxrwx. 2 root root   71 Jun 24 09:36 apisix_log    # apisix日志文件目录
drwxrwxrwx. 2 root root   23 Jun 23 17:10 dashboard_conf  # 可视化工具apisix-dashboard配置文件目录
-rwxrwxrwx. 1 root root 1304 Jun 19 10:12 docker-compose-alpine.yml # docker-compose 部署脚本（alpine）版本
-rwxrwxrwx. 1 root root 1453 Jun 19 10:12 docker-compose.yml # docker-compose 部署脚本
drwxrwxrwx. 2 root root   27 Jun 19 10:12 etcd_conf # ectd配置文件目录
drwxrwxrwx. 3 root root   31 Jun 23 17:06 etcd_data # ectd数据目录
drwxrwxrwx. 2 root root  107 Jun 19 10:12 mkcert
drwxrwxrwx. 2 root root   40 Jun 19 10:12 upstream # 两个测试用的Nginx服务配置


```

*   从`docker-compose.yml`中我们可以发现，该脚本不仅启动了 apisix、apisix-dashboard、etcd 这三个核心服务，还启动了两个测试用的 Nginx 服务；
    

```
version: "3"

services:
  # 可视化管理工具apisix-dashboard
  apisix-dashboard:
    image: apache/apisix-dashboard:2.7
    restart: always
    volumes:
    - ./dashboard_conf/conf.yaml:/usr/local/apisix-dashboard/conf/conf.yaml
    ports:
    - "9000:9000"
    networks:
      apisix:

    # 网关apisix
  apisix:
    image: apache/apisix:2.6-alpine
    restart: always
    volumes:
      - ./apisix_log:/usr/local/apisix/logs
      - ./apisix_conf/config.yaml:/usr/local/apisix/conf/config.yaml:ro
    depends_on:
      - etcd
    ##network_mode: host
    ports:
      - "9080:9080/tcp"
      - "9443:9443/tcp"
    networks:
      apisix:

    # apisix配置数据存储etcd
  etcd:
    image: bitnami/etcd:3.4.15
    user: root
    restart: always
    volumes:
      - ./etcd_data:/bitnami/etcd
    environment:
      ETCD_ENABLE_V2: "true"
      ALLOW_NONE_AUTHENTICATION: "yes"
      ETCD_ADVERTISE_CLIENT_URLS: "http://0.0.0.0:2379"
      ETCD_LISTEN_CLIENT_URLS: "http://0.0.0.0:2379"
    ports:
      - "2379:2379/tcp"
    networks:
      apisix:

  # 测试用nginx服务web1，调用返回 hello web1
  web1:
    image: nginx:1.19.0-alpine
    restart: always
    volumes:
      - ./upstream/web1.conf:/etc/nginx/nginx.conf
    ports:
      - "9081:80/tcp"
    environment:
      - NGINX_PORT=80
    networks:
      apisix:

  # 测试用nginx服务web2，调用返回 hello web2
  web2:
    image: nginx:1.19.0-alpine
    restart: always
    volumes:
      - ./upstream/web2.conf:/etc/nginx/nginx.conf
    ports:
      - "9082:80/tcp"
    environment:
      - NGINX_PORT=80
    networks:
      apisix:

networks:
  apisix:
    driver: bridge

```

*   在`docker-compose.yml`文件所在目录下，使用如下命令可以一次性启动所有服务；
    

```
docker-compose -p apisix-docker up -d


```

*   启动成功后，使用如下命令可查看所有服务的运行状态；
    

```
docker-compose -p apisix-docker ps


```

```
              Name                            Command               State                       Ports                     
--------------------------------------------------------------------------------------------------------------------------
apisix-docker_apisix-dashboard_1   /usr/local/apisix-dashboar ...   Up      0.0.0.0:9000->9000/tcp                        
apisix-docker_apisix_1             sh -c /usr/bin/apisix init ...   Up      0.0.0.0:9080->9080/tcp, 0.0.0.0:9443->9443/tcp
apisix-docker_etcd_1               /opt/bitnami/scripts/etcd/ ...   Up      0.0.0.0:2379->2379/tcp, 2380/tcp              
apisix-docker_web1_1               /docker-entrypoint.sh ngin ...   Up      0.0.0.0:9081->80/tcp                          
apisix-docker_web2_1               /docker-entrypoint.sh ngin ...   Up      0.0.0.0:9082->80/tcp 


```

*   接下来就可以通过可视化工具来管理 apisix 了，登录账号密码为`admin:admin`，访问地址：http://192.168.5.78:9000/
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYltMhDdI9tXXhE9RKCouiavd7F1CHsksqqmORVI1pehQhvE82rb2uNxA/640?wx_fmt=png)

*   登录之后看下界面，还是挺漂亮的，apisix 搭建非常简单，基本无坑；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYqOKuhmnQSkVf2xgkmibEvbRPsYcuzB5BJpIibmLiaUVdO8ibVeUk2326rw/640?wx_fmt=png)

*   还有两个测试服务，`web1`访问地址：http://192.168.5.78:9081/
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYkmKQVHjMsvh1tGBhyyejT6kh5ib8Sy1x4J4TGxxFaK8Mlx0XJlycTKQ/640?wx_fmt=png)

*   另一个测试服务`web2`访问地址：http://192.168.5.78:9082/
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYZggpJGHB25ym2UoKBnIhdXRkZGiatlSMylfexoJVFRaefF0QU9WLQDw/640?wx_fmt=png)

使用
--

> apisix 作为新一代的网关，不仅支持基本的路由功能，还提供了丰富的插件，功能非常强大。

### 基本使用

> 我们先来体验下 apisix 的基本功能，之前已经启动了两个 Nginx 测试服务`web1`和`web2`，接下来我们将通过 apisix 的路由功能来访问它们。

*   首先我们需要创建上游（Upstream），上游相当于虚拟主机的概念，可以对真实的服务提供负载均衡功能；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYd7Sc41qkpXQhYADS6gJzrNbN0IpPcBXLPEZkQcbgCA59cQP3icoX2pQ/640?wx_fmt=png)

*   创建`web1`的上游，设置好名称、负载均衡算法和目标节点信息；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYuiayF1lU1SfdMxJ0szeK3WOb4pga9oicFC1u28mkGbYDiaj4WsibeGcHRw/640?wx_fmt=png)

*   再按照上述方法创建`web2`的上游，创建完成后上游列表显示如下；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYYvEIpzJQ25CliaEXiaoD82X6KEWtBEn2yibZ5QVDCNeT3Ozr9Ylpn7nBw/640?wx_fmt=png)

*   再创建`web1`的路由（Route），路由可以用于匹配客户端的请求，然后转发到上游；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYunK1HppWPnOSnMCx6uwMIzIbeAic3ibog7ZwK7G3icicatuUvoxTjiak8FA/640?wx_fmt=png)

*   再选择好路由的上游为`web1`；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYnhy9mxCvpeiaVoXlq1X5Mt2hsyb6qTInEWBr5mzqKNoQqgHEmXVLXnA/640?wx_fmt=png)

*   接下来选择需要应用到路由上的插件，apisix 的插件非常丰富，多达三十种，作为基本使用，我们暂时不选插件；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYZPGBAEfF9fHuWB1n57kibLXHhCiazPE7HptIuaKjhlJADcmvpVyQR7Rg/640?wx_fmt=png)

*   再创建`web2`的路由，创建完成后路由列表显示如下；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYlJkawUicCuAicgOQFMSnahEC8nUzbbhYJrqCUthG4WWbkzx3KExPeu4Q/640?wx_fmt=png)

*   接下来我们通过 apisix 网关访问下`web1`服务：http://192.168.5.78:9080/web1/
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYIib4WWBYvbSSdlOQuejialeFN7vFQnTDEjetWvHtwxJicTEwFC7z7yMQA/640?wx_fmt=png)

*   接下来我们通过 apisix 网关访问下`web2`服务：http://192.168.5.78:9080/web2/
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYfWXEJjCia3Kfkns8Nxy5fvFJWWYaCg65ah2kiabXp1sZY91OxAUGtFTw/640?wx_fmt=png)

### 进阶使用

> apisix 通过启用插件，可以实现一系列丰富的功能，下面我们来介绍几个实用的功能。

#### 身份认证

> 使用 JWT 来进行身份认证是一种非常流行的方式，这种方式在 apisix 中也是支持的，可以通过启用`jwt-auth`插件来实现。

*   首先我们需要创建一个消费者对象（Consumer）；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYpGcfPmeuGOibKiam8eQIiahAN0MWeyzq4Gddpj8icpxSoTCmY1kM5fjRYQ/640?wx_fmt=png)

*   然后在插件配置中启用`jwt-auth`插件；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYlt2ezBowPmLAOoPKzYfd94gNqDlDgbU78ahcz5Fj1KmdcOBOG4YYpA/640?wx_fmt=png)

*   启用插件时配置好插件的`key`和`secret`；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYd1US0Ke7INbPJNe9JQZ1Y1Fo3reRl7ia02RdAPsiahPic7bqgcUPjEVibg/640?wx_fmt=png)

*   创建成功后消费者列表时显示如下；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYIF8jCxKiblhgzVQp5ibpics9LpFoVCA0nsmg1JMu6ALibLXtCdrHic1aJDA/640?wx_fmt=png)

*   之后再创建一个路由，路由访问路径匹配`/auth/*`，只需启用`jwt-auth`插件即可；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYuFe0xZVPrILaSExcGDOEicxkU25B1WlLj92qdKUklcxibuIo74xt965Q/640?wx_fmt=png)

*   访问接口获取生成好的 JWT Token，需要添加两个参数，`key`为 JWT 插件中配置的 key，`payload`为 JWT 中存储的自定义负载数据，JWT Token 生成地址：http://192.168.5.78:9080/apisix/plugin/jwt/sign
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDY0OQMkKwBf29DyrAbibRdfFRbPYf33sdw1MRPhLCTKshef1pKMRuwxHQ/640?wx_fmt=png)

*   不添加 JWT Token 访问路由接口，会返回 401，接口地址：http://192.168.5.78:9080/auth/
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYRBeVnfSlM4Pch09nTwQ4wT08nCLgmPF2aTTVlyoqHcAIchxicUVicIuA/640?wx_fmt=png)

*   在请求头`Authorization`中添加 JWT Token 后即可正常访问；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYE0eUtwibNG4IhZTjLkyCHrVehGDeOEmCBrazjQwSsZLibvEYZkwXiaHyg/640?wx_fmt=png)

*   当然 apisix 支持的身份认证并不只这一种，还有下面几种。
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYflzcTOtaYibvoJkC87v8zL9SVC0sxkVWqibDSWRR8sVLsrXMzLgMmXeA/640?wx_fmt=png)

#### 限流功能

> 有时候我们需要对网关进行限流操作，比如每个客户端 IP 在 30 秒内只能访问 2 次接口，可以通过启用`limit-count`插件来实现。

*   我们在创建路由的时候可以选择配置`limit-count`插件；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYuLLotDQr5rlQTzlicEsiahonrkVjV00ru9mSfmf4UfiaiaLYEmjRPJAuPA/640?wx_fmt=png)

*   然后对`limit-count`插件进行配置，根据`remote_addr`进行限流；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYupvjCZQvXybibXib577UpawZdDE1DmNsqC8ov38UwdBDoTiacgbqqPSCw/640?wx_fmt=png)

*   当我们在 30 秒内第 3 次调用接口时，apisix 会返回 503 来限制我们的调用。
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYSIFhzYynmicACyrrHo16LTTWM4Uc7kTaYaicDtsicR7QrlicLMoePKUccw/640?wx_fmt=png)

#### 跨域支持

> 如果你想让网关支持跨域访问的话，可以通过启用`cors`插件来实现。

*   我们在创建路由的时候可以选择配置`cors`插件；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDY4Xic3jDWLGiakk5RibW4ICHjvfXMsmaaa0shOXBXa7PVr4atPgKG0XibmA/640?wx_fmt=png)

*   然后对`cors`插件进行配置，配置好跨域访问策略；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYT9spMevdwNcAwxjGZtyzEqLYQRXXNNAFGfHrVlqnrkiaiao8HM17QJew/640?wx_fmt=png)

*   调用接口测试可以发现接口已经返回了 CORS 相关的请求头。
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYXq7E5WlxNIRdKCYGh48vXKUETtFjPCHWWqAI6mmxic5f5bmJUqWUjvw/640?wx_fmt=png)

总结
--

体验了一把 apisix 这个全新一代的 API 网关，有可视化管理的网关果然不一样，简单易用，功能强大！如果你的微服务是云原生的话，可以试着用它来做网关。

其实 apisix 并不是个小众框架，很多国内外大厂都在使用了，如果你想知道哪些公司在使用，可以参考下面的连接。

https://github.com/apache/apisix/blob/master/powered-by.md

参考资料
----

apisix 的官方文档非常友好，支持中文，简直是业界良心！过一遍官方文档基本就能掌握 apisix 了。

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmOYiaSuh6DPK7BUicx2mhSDYeibYGb4ibBpIg3KHiabohAr4wCn8xUdnlQtzVDWAPZ41PibsXGoPLk5Bkg/640?wx_fmt=png)

官方文档：https://apisix.apache.org/zh/docs/apisix/getting-started

项目源码地址
------

https://github.com/apache/apisix-docker