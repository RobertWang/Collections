> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1623031534&ver=3115&signature=8WQJb9ufUotRA3gkDcXVtyDmnnL4vXYbMEw1cYvyWTKqbLS4QjwKLC-ReFFrRWfVLZYAvYQn0xMk6THX7yNja7BMuX7Tu7cNafrgPogUO9627PZMQR36j5oJozITxanj&new=1)

点击上方 “朱小厮的博客”，选择 “设为星标”

后台回复 " 加群 "，加入新技术群

Kong 是由 Mashape 开发的并于 2015 年开源的一款 API 网关，它是基于 OpenResty（Nginx + Lua 模块）和 Apache Cassandra/PostgreSQL 构建的，能提供易于使用的 RESTful API 来操作和配置 API 管理系统。Kong 可以水平扩展多个 Kong Server，通过前置的负载均衡配置把请求均匀地分发到各个 Server，来应对大批量的网络请求。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbKghBkXvy50mrw5Udu85tWib4y5d6nToOPib311aLudDtiaicCbP5b5JYZm28xCZ84eXzJPOVbiaZbJfrQ/640?wx_fmt=png)

Kong 的扩展是通过插件机制进行的，并且也提供了插件的定制示例方法。插件定义了一个请求从进入到最后反馈到客户端的整个生命周期，所以可以满足大部分的定制需求，本身 Kong 也已经集成了相当多的插件，包括密钥认证、CORS、文件日志、API 请求限流、请求转发、健康检查、熔断等。官网地址：https://konghq.com/，代码托管地址：https://github.com/Kong/kong。

Nginx、Openresty 和 Kong 三者紧密相连：

*   Nginx = Http Server + Reversed Proxy + Load Balancer
    
*   Openresty = Nginx + Lua-nginx-module，Openresty 是寄生在 Nginx 上，暴露 Nginx 处理的各个阶段的钩子， 使用 Lua 扩展 Nginx
    
*   Kong = Openresty + Customized Framework，Kong 作为 OpenResty 的一个应用程序
    

> 在使用 Kong 之前，最好新了解一下 OpenResty 和 Nginx，可以参考微信公众号「朱小厮博客」中的《[Nginx 架构原理科普](https://mp.weixin.qq.com/s?__biz=MzU0MzQ5MDA0Mw==&mid=2247489921&idx=1&sn=0137f8212d94062987d83767836514ac&scene=21#wechat_redirect)》和《[OpenResty 概要及原理科普](https://mp.weixin.qq.com/s?__biz=MzU0MzQ5MDA0Mw==&mid=2247490049&idx=1&sn=47e297afd0cb938ab74936e8cc4cc905&scene=21#wechat_redirect)》这两篇文章。

Kong 网关具有以下的特性：

*   可扩展性: 通过简单地添加更多的服务器，可以轻松地进行横向扩展，这意味着您的平台可以在一个较低负载的情况下处理任何请求。
    
*   模块化: 可以通过添加新的插件进行扩展，这些插件可以通过 RESTful Admin API 轻松配置。
    
*   在任何基础架构上运行: Kong 网关可以在任何地方都能运行。可以在云或内部网络环境中部署 Kong，包括单个或多个数据中心设置，以及 public，private 或 invite-only APIs。
    

Kong 的整体架构如下所示：![](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbKghBkXvy50mrw5Udu85tWibTIMAwOwrwThB06Jk0I12LwcGia3Q5WCDwQy0lrpFCvzc0c4s0iaJXw8w/640?wx_fmt=png)

*   Kong Restful 管理 API 提供了 API、API 消费者、插件、upstreams、证书等管理。
    
*   Kong 插件拦截请求 / 响应，相当于 Servlet 中的拦截器，实现请求的 AOP 处理。
    
*   数据中心用于存储 Kong 集群节点信息、API、消费者、插件等信息，目前提供了 PostgreSQL 和 Cassandra 支持，如果需要高可用建议使用 Cassandra。
    
*   Kong 集群中的节点通过 Gossip 协议自动发现其他节点，当通过一个 Kong 节点的管理 API 进行一些变更时也会通知其他节点。每个 Kong 节点的配置信息是会缓存的，如插件，那么当在某一个 Kong 节点修改了插件配置时，需要通知其他节点配置的变更。
    
*   Kong 核心基于 OpenResty，实现了请求 / 响应的 Lua 处理化。
    

Kong 网关的 API 接口的典型请求工作流程如下图所示：![](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbKghBkXvy50mrw5Udu85tWib7QdyJOibQ3XYH3nJXbM3ZFZibTSjhibJxynqR8KOh1EN1iaPx13JZMq8xg/640?wx_fmt=png)

当 Kong 运行时，每个对 API 的请求将先被 Kong 命中，然后这个请求将会被代理转发到最终的 API 接口。在请求（Requests）和响应（Responses）之间，Kong 将会执行已经事先安装和配置好的任何插件，授权 API 访问操作。Kong 是每个 API 请求的入口点（Endpoint）。

Install
-------

Kong 可运行在某些 Linux 发行版、Mac OS X 和 Docker 中，无论是本地机还是云端服务器皆可运行。除了免费的开源版本，Mashape 还提供了付费的企业版 [1]，其中包括技术支持、使用培训服务以及 API 分析插件。![](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbKghBkXvy50mrw5Udu85tWib9xuTQbjF9iaOkQYiboN8ng9CVRTlvCibgPhrzANrPj7IZlAXVT9YAM2sA/640?wx_fmt=png)

为了演示方便，下面就以 Docker 环境中部署 Kong 为例来做相关讲解，内容参考官网：https://docs.konghq.com/install/docker/。Kong 安装有两种方式，一种是没有数据库依赖的 DB-less 模式，另一种是 with a Database 模式。我们这里使用第二种带 Database 的模式，因为这种模式功能更全。

**1. 构建 Kong 的容器网络**

首先我们创建一个 docker 自定义网络，以允许容器相互发现和通信。在下面的创建命令中 kong-net 是我们创建的 Docker 网络名称。

```
 $ docker network create kong-net


```

**2. 搭建数据库环境**

Kong 目前使用 Cassandra 或者 PostgreSQL，你可以执行以下命令中的一个来选择你的 Database。请注意定义网络 --network=kong-net 。

使用 Cassandra：

```
 docker run -d --name kong-database \
               --network=kong-net \
               -p 9042:9042 \
               cassandra:3


```

使用 PostgreSQL:

```
 $ docker run -d --name kong-database \
               --network=kong-net \
               -p 5432:5432 \
               -e "POSTGRES_USER=kong" \
               -e "POSTGRES_DB=kong" \
               -e "POSTGRES_PASSWORD=kong" \
               postgres:9.6


```

**3. 初始化或者迁移数据库**

我们使用 docker run --rm 来初始化数据库，该命令执行后会退出容器而保留内部的数据卷（volume）。这个命令我们还是要注意的，一定要跟你声明的网络，数据库类型、host 名称一致。同时注意 Kong 的版本号，注：当前 Kong 最新版本为 2.x，不过目前的 kong-dashboard (Kong Admin UI) 尚未支持 2.x 版的 Kong，为了方便后面的演示，这里以最新的 1.x 版的 Kong 作为演示。（截止 2020-04-24 时，Kong 最新版为 1.5.1）

下面指定的数据库是 PostgreSQL，如果连接的是 Cassandra，可以将下面的 KONG_DATABASE 配置为 cassandra。

```
$ docker run --rm \
     --network=kong-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_HOST=kong-database" \
     -e "KONG_PG_PASSWORD=kong" \
     -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
     kong:1.5.1 kong migrations bootstrap


```

**4. 启动 Kong 容器**

完成初始化或者迁移数据库后，我们就可以启动一个连接到数据库容器的 Kong 容器，请务必保证你的数据库容器启动状态，同时检查所有的环境参数 -e 是否是你定义的环境。

```
 $ docker run -d --name kong \
     --network=kong-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_HOST=kong-database" \
     -e "KONG_PG_PASSWORD=kong" \
     -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
     -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
     -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
     -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
     -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
     -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
     -p 8000:8000 \
     -p 8443:8443 \
     -p 8001:8001 \
     -p 8444:8444 \
     kong:1.5.1


```

Kong 默认绑定 4 个端口：

*   8000：用来接收客户端的 HTTP 请求，并转发到 upstream。
    
*   8443：用来接收客户端的 HTTPS 请求，并转发到 upstream。
    
*   8001：HTTP 监听的 API 管理接口。
    
*   8444：HTTPS 监听的 API 管理接口。
    

到这里，Kong 已经安装完毕，我们可以使用 `docker ps`命令查看当前运行容器，正常情况下可以看到 Kong 和  PostgreSQL 的两个容器：

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                NAMES
a28160da4a9d        kong:latest         "/docker-entrypoint.…"   10 seconds ago      Up 9 seconds        0.0.0.0:8000-8001->8000-8001/tcp, 0.0.0.0:8443-8444->8443-8444/tcp   kong
6c85a2e5491f        postgres:9.6        "docker-entrypoint.s…"   31 minutes ago      Up 31 minutes       0.0.0.0:5432->5432/tcp                                               kong-database


```

我们可以通过 `curl -i http://localhost:8001/` 来查看 Kong 是否运行完好。

Kong UI
-------

Kong 企业版提供了管理 UI，开源版本是没有的。但是有很多的开源的管理 UI ，其中比较 Fashion 的有 Kong Dashboard 和 Konga。Kong Dashboard 当前最新版本（3.6.x）并不支持最新版本的 Kong，最后一次更新也要追溯到 1 年多以前了，选择 Konga 会更好一点。这里简单介绍一下 Kong Dashboard 和 Konga。

### Kong Dashboard

Kong Dashboard 的 Github 地址为：https://github.com/PGBI/kong-dashboard。docker 环境中安装运行如下：

```
$ docker run --rm  \
--network=kong-net \
-p 8080:8080 \
pgbi/kong-dashboard start \
--kong-url http://kong:8001


```

启动之后，可以在浏览器中输入 `http://localhost:8080`来访问 Kong Dashboard 管理界面。![](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbKghBkXvy50mrw5Udu85tWib16wq1hoPubPMzd284T2x8ZKhxx1OWPGR8ibgw2lEOwmLrMKpJLibtIeA/640?wx_fmt=png)

### Konga

Konga （官网地址：https://pantsel.github.io/konga/，Github 地址：https://github.com/pantsel/konga）可以很好地通过 UI 观察到现在 Kong 的所有的配置，并且可以对于管理 Kong 节点情况进行查看、监控和预警。Konga 主要是用 AngularJS 写的，运行于 nodejs 服务端。具有以下特性：

*   管理所有 Kong Admin API 对象。
    
*   支持从远程源（数据库，文件，API 等）导入使用者。
    
*   管理多个 Kong 节点。使用快照备份，还原和迁移 Kong 节点。
    
*   使用运行状况检查监视节点和 API 状态。
    
*   支持电子邮件和闲置通知。
    
*   支持多用户。
    
*   易于数据库集成（MySQL，PostgresSQL，MongoDB，SQL Server）。
    

下面使用的 PostgresSQL 是和上面在 docker 环境中安装 Kong 时的是一致的，注意用户名、密码、数据库名称等配置，docker 环境安装启动 Konga:

```
$ docker run  -d -p 1337:1337 \
        --network kong-net \
        --name konga \
        -e "DB_ADAPTER=postgres" \
        -e "DB_URI=postgresql://kong:kong@kong-database/kong" \
        pantsel/konga


```

如果 Konga 容器启动成功，可以通过 http://localhost:1337 / 访问管理界面。通过注册后进入，然后在 CONNECTIONS 中添加 Kong 服务的管理路径`http://xxx.xxx.xxx.xxx:8001`。Konga 管理界面示例如下：![](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbKghBkXvy50mrw5Udu85tWibYc70ZRvevNfwkjseB5OvM2hs1DXtxJ53e0227icNPtTJqQd8c5FxA7w/640?wx_fmt=png)

Kong Admin API
--------------

部署好 Kong 之后，则需要将我们自己的接口加入到 Kong 的中管理，Kong 提供了比较全面的 RESTful API，每个版本会有所不同，详细可以参考官网：https://docs.konghq.com/2.0.x/admin-api/。Kong 管理 API 的端口是 8001（8044），服务、路由、配置都是通过这个端口进行管理，所以部署好之后页面可以直接访问 `http://localhost:8001`。

这里我们先来了解一下如何使用 RESTful 管理接口来管理 Service (服务)、Route（路由）。

**1. 添加一个 Service**

```
$ curl -i -X POST http://localhost:8001/services \
--data name=hello-service \
--data url='http://xxx.xxx.xxx.xxx:8081/hello'


```

> 这里的'http://xxx.xxx.xxx.xxx:8081/hello' 是在《[网关 Zuul 科普](https://mp.weixin.qq.com/s?__biz=MzU0MzQ5MDA0Mw==&mid=2247489653&idx=1&sn=b2ed7b657b67c147483571ae01cb9aae&scene=21#wechat_redirect)》中提及的一个简单的基础服务接口，调用这个接口会返回 `Hello！`。

客户端调用 Service 名称 hello-service 访问'http://xxx.xxx.xxx.xxx:8081/hello'。添加成功后，系统将返回：

```
{
  "host": "xxx.xxx.xxx.xxx",
  "created_at": 1587959433,
  "connect_timeout": 60000,
  "id": "d96f418a-8158-4b1d-844d-ed994fdbcc2c",
  "protocol": "http",
  "name": "hello-service",
  "read_timeout": 60000,
  "port": 8081,
  "path": "\/hello",
  "updated_at": 1587959433,
  "retries": 5,
  "write_timeout": 60000,
  "tags": null,
  "client_certificate": null
}


```

**2. 为 Service 添加一个 Route**

```
$ curl -i -X POST \
--url http://localhost:8001/services/hello-service/routes \
--data 'paths[]=/hello' \
--data name=hello-route


```

添加成功后，系统将返回：

```
{
  "id": "667bafde-7ca4-4fc4-b4f1-15c3cbec0b09",
  "path_handling": "v1",
  "paths": [
    "\/hello"
  ],
  "destinations": null,
  "headers": null,
  "protocols": [
    "http",
    "https"
  ],
  "methods": null,
  "snis": null,
  "service": {
    "id": "d96f418a-8158-4b1d-844d-ed994fdbcc2c"
  },
  "name": hello-route,
  "strip_path": true,
  "preserve_host": false,
  "regex_priority": 0,
  "updated_at": 1587959468,
  "sources": null,
  "hosts": null,
  "https_redirect_status_code": 426,
  "tags": null,
  "created_at": 1587959468
}


```

**3. 验证**

我们可以通过访问 http://localhost:8000/hello 来验证一下配置是否正确。

前面的操作就等效于配置 nginx.conf:

```
server {
  listen 8000;
  location /hello {
    proxy_pass http://xxx.xxx.xxx.xxx8081/hello;
  }
}


```

不过，前面的配置操作都是动态的，无需像 Nginx 一样需要重启。

Service 是抽象层面的服务，它可以直接映射到一个物理服务，也可以指向一个 Upstream（同 Nginx 中的 Upstream，是对上游服务器的抽象）。Route 是路由的抽象，它负责将实际的请求映射到 Service。除了 Serivce、Route 之外，还有 Tag、Consumer、Plugin、Certificate、SNI、Upstream、Target 等，读者可以从官网的介绍文档 [2] 中了解全貌。

下面在演示一个例子，修改 Service，将其映射到一个 Upstream：

```
# 添加 name为 hello-upstream 的 Upstream
$ curl -i -X POST http://localhost:8001/upstreams \
--data name=hello-upstream

# 为 mock-upstream 添加 Target，Target 代表了一个物理服务（IP地址/hostname + port的抽象），一个Upstream可以包含多个Targets
$ curl -i -X POST http://localhost:8001/upstreams/hello-upstream/targets \
--data target="xxx.xxx.xxx.xxx:8081"

# 修改  hello-service，为其配置
$ curl -i -X PATCH http://localhost:8001/services/hello-service \
--data url='http://hello-upstream/hello'


```

上面的配置等同于 Nginx 中的 nginx.conf 配置 :

```
upstream hello-upstream{
  server xxx.xxx.xxx.xxx:8081;
}

server {
  listen 8000;
  location /hello {
    proxy_pass http://hello-upstream/hello;
  }
}


```

当然，这里的配置我们也可以通过管理界面来操作。上面操作完之后，在 Konga 中也有相关信息展示出来：![](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbKghBkXvy50mrw5Udu85tWib3AehCwRZXIpiaDr4KPrIoXrQoVRJAOVZDOb7AIuvtqj5FLmAppdCdhw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbKghBkXvy50mrw5Udu85tWibjHHUv60svU5QJiaia3aPzsLKqVWOUhVLuT8GVw8N8fkC4uWjUSSez5gQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbKghBkXvy50mrw5Udu85tWiboyrYOWialaOaRRco2rHMKjGticKuTueb1ofzVgatkK7HYCTN5icDR8iaxw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbKghBkXvy50mrw5Udu85tWibJGKUeUHekZpjibdCcDT2UWIOC02a28fpGEFvYXwClu3H78adlNlPialw/640?wx_fmt=png)

Kong Plugins
------------

Kong 通过插件 Plugins 实现日志记录、安全检测、性能监控和负载均衡等功能。下面我将演示一个例子，通过启动 apikey 实现简单网关安全检验。

**1. 配置 key-auth 插件**

```
$ curl -i -X POST http://localhost:8001/routes/hello-route/plugins \
--data name=key-auth


```

这个插件接收 config.key_names 定义参数，默认参数名称 ['apikey']。在 HTTP 请求中 header 和 params 参数中包含 apikey 参数，参数值必须 apikey 密钥，Kong 网关将坚持密钥，验证通过才可以访问后续服务。

此时我们使用 `curl -i http://localhost:8000/hello` 来验证一下是否生效，如果如下所示，访问失败（HTTP/1.1 401 Unauthorized，"No API key found in request" ），说明 Kong 安全机制生效了。

```
HTTP/1.1 401 Unauthorized
Date: Mon, 27 Apr 2020 06:44:58 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
WWW-Authenticate: Key realm="kong"
Content-Length: 41
X-Kong-Response-Latency: 2
Server: kong/1.5.1

{"message":"No API key found in request"}


```

在 Konga 中我们也可以看到相关记录：![](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbKghBkXvy50mrw5Udu85tWibCicYBicdXZ9nz0qAzRNoxAWp7gnqdKuY8n1mK7Hn5hCFdL5CqMByMTWQ/640?wx_fmt=png)

**2. 为 Service 添加服务消费者（Consumer）**，定义消费者访问 API Key, 让他拥有访问 hello-service 的权限。

创建消费者 Hidden：

```
$ curl -i -X POST http://localhost:8001/consumers/ \
--data username=Hidden


```

创建成功之后，返回：

```
{
  "custom_id": null,
  "created_at": 1587970751,
  "id": "95546c8f-248c-45c7-bce5-d972d3d9291a",
  "tags": null,
  "username": "Hidden"
}


```

之后为消费者 Hidden 创建一个 api key，输入如下命令：

```
$ curl -i -X POST http://localhost:8001/consumers/Hidden/key-auth/ \
--data key=ENTER_KEY_HERE


```

现在我们再来验证一下`http://localhost:8000/hello`：

```
$ curl -i -X GET http://localhost:8000/hello \
--header "apikey:ENTER_KEY_HERE"


```

返回：

```
HTTP/1.1 200
Content-Type: text/plain;charset=UTF-8
Content-Length: 7
Connection: keep-alive
Date: Mon, 27 Apr 2020 07:08:38 GMT
X-Kong-Upstream-Latency: 116
X-Kong-Proxy-Latency: 71
Via: kong/1.5.1

Hello!


```

Well done.

Kong 官网（https://docs.konghq.com/hub/）列出了已有的所有插件，如下图所示：![](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbKghBkXvy50mrw5Udu85tWib2PUby0CGGn3dlVIOPaibaWZUMbtEa34iaXTDm3cfny4gkHibpWXpjN07g/640?wx_fmt=png)

Kong 网关插件概括为如下：

*   身份认证插件：Kong 提供了 Basic Authentication、Key authentication、OAuth2.0 authentication、HMAC authentication、JWT、LDAP authentication 认证实现。
    
*   安全控制插件：ACL（访问控制）、CORS（跨域资源共享）、动态 SSL、IP 限制、爬虫检测实现。
    
*   流量控制插件：请求限流（基于请求计数限流）、上游响应限流（根据 upstream 响应计数限流）、请求大小限制。限流支持本地、Redis 和集群限流模式。
    
*   分析监控插件：Galileo（记录请求和响应数据，实现 API 分析）、Datadog（记录 API Metric 如请求次数、请求大小、响应状态和延迟，可视化 API Metric）、Runscope（记录请求和响应数据，实现 API 性能测试和监控）。
    
*   协议转换插件：请求转换（在转发到 upstream 之前修改请求）、响应转换（在 upstream 响应返回给客户端之前修改响应）。
    
*   日志应用插件：TCP、UDP、HTTP、File、Syslog、StatsD、Loggly 等。
    

总结
--

Kong 作为 API 网关提供了 API 管理功能及围绕 API 管理实现了一些默认的插件，另外还具备集群水平扩展能力，从而提升整体吞吐量。Kong 本身是基于 OpenResty，可以在现有 Kong 的基础上进行一些扩展，从而实现更复杂的特性。虽然有一些特性 Kong 默认是缺失的，如 API 级别的超时、重试、fallback 策略、缓存、API 聚合、AB 测试等，这些功能插件需要企业开发人员通过 Lua 语言进行定制和扩展。综上所述，Kong API 网关默认提供的插件比较丰富， 适应针对企业级的 API 网关定位。

### References

1.  https://github.com/Kong/kong
    
2.  https://www.jianshu.com/p/a2e0bc8f4bfb
    
3.  https://docs.konghq.com/install/docker
    
4.  https://www.cnblogs.com/duanxz/p/9770645.html
    
5.  https://docs.konghq.com/2.0.x/admin-api/
    
6.  https://docs.konghq.com/hub/
    

### 参考资料

[1]

企业版: _http://getkong.org/enterprise/_

[2]

官网的介绍文档: _https://docs.konghq.com/2.0.x/admin-api/_