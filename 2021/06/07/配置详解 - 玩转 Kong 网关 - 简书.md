> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/f3b1699777d6)

配置加载
----

Kong 的默认配置在 `/etc/kong/kong.conf.default` 。如果你通过一个官方的安装包来安装 Kong。您可以复制下面的文件，开始配置 Kong：

```
$ cp /etc/kong/kong.conf.default /etc/kong/kong.conf


```

如果你的配置中的所有值都被考虑，那么 Kong 将使用默认配置运行。在开始时，Kong 可能会查找的几个缺省配置文件位置如下：

```
/etc/kong/kong.conf
/etc/kong.conf


```

您可以通过在 CLI 中使用`-c / -conf`参数自定义配置文件路径来覆盖此默认配置：

```
$ kong start --conf /path/to/kong.conf


```

配置格式很简单：注释由字符 `#` 定义。布尔值可以被指定为 `on/off` 或者`true/false`。

验证您的配置
------

您可以使用`check`命令来验证设置的完整性：

```
$ kong check <path/to/kong.conf>
configuration at <path/to/kong.conf> is valid


```

这个命令，将检测您当前设置的环境变量，并且在您的设置错误时报错。

此外，您还可以在调试模式下使用 CLI，以便更深入地了解正在启动的属性：

```
$ kong start -c <kong.conf> --vv
2016/08/11 14:53:36 [verbose] no config file found at /etc/kong.conf
2016/08/11 14:53:36 [verbose] no config file found at /etc/kong/kong.conf
2016/08/11 14:53:36 [debug] admin_listen = "0.0.0.0:8001"
2016/08/11 14:53:36 [debug] database = "postgres"
2016/08/11 14:53:36 [debug] log_level = "notice"
[...]


```

环境变量
----

当从配置文件中加载属性时，Kong 也会查找相同名称的环境变量。这允许您通过环境变量完全配置 Kong。

所有环境变量的前缀 `KONG_` ，大写并带有与设置相同的名称将覆盖默认配置。  
例如：

```
log_level = debug # in kong.conf


```

会被如下设置覆盖：

```
$ export KONG_LOG_LEVEL=error


```

定制 Nginx 配置和嵌入 Kong
-------------------

调整 Nginx 配置是设置您的 Kong 实例的一个重要部分，因为它允许您优化其性能，或者将 Kong 嵌入到已经运行的 OpenResty 实例中。

*   自定义 Nginx 配置

Kong 可以用 `--nginx-conf` 的参数启动，重新加载和重新启动，该参数必须指定 Nginx 配置模板。这样的模板使用了 Penlight 模板引擎，它是使用给定的 Kong 配置编译的，然后在开始 Nginx 之前被保存到您的 Kong 前缀目录中。

默认模板可以在 [https://github.com/Kong/kong/tree/master/kong/templates](https://github.com/Kong/kong/tree/master/kong/templates)  
找到。它分为两个 Nginx 配置文件：`nginx.lua` 和 `nginx_kong.lua`。前者是运行 Kong 的最低的配置要求，其会包括后者。当 Kong 开始运行时，在开始 Nginx 之前，它将这两个文件复制到前缀目录中，看起来是这样的：

```
/usr/local/kong
├── nginx-kong.conf
├── nginx.conf


```

如果您希望在 Nginx 配置中包含其他服务模块，或者您必须调整不受 Kong 配置影响的全局设置，这里有一个建议：

```
# ---------------------
# custom_nginx.template
# ---------------------

worker_processes ${{NGINX_WORKER_PROCESSES}}; # can be set by kong.conf
daemon ${{NGINX_DAEMON}};                     # can be set by kong.conf

pid pids/nginx.pid;                      # this setting is 强制的
error_log logs/error.log ${{LOG_LEVEL}}; # can be set by kong.conf

events {
    use epoll; # custom setting
    multi_accept on;
}

http {
    # include default Kong Nginx config
    include 'nginx-kong.conf';

    # custom server
    server {
        listen 8888;
        server_name custom_server;

        location / {
          ... # etc
        }
    }
}


```

你可以这样启动 Kong：

```
$ kong start -c kong.conf --nginx-conf custom_nginx.template


```

如果您希望自定义 Kong 的 Nginx 子配置文件，最终添加其他 Lua 处理程序或定制`lua_*`指令，您可以在`custom_nginx.template`内联`nginx_kong.lua`这个配置。模板示例文件如下:

```
# ---------------------
# custom_nginx.template
# ---------------------

worker_processes ${{NGINX_WORKER_PROCESSES}}; # can be set by kong.conf
daemon ${{NGINX_DAEMON}};                     # can be set by kong.conf

pid pids/nginx.pid;                      # this setting is mandatory
error_log logs/error.log ${{LOG_LEVEL}}; # can be set by kong.conf

events {}

http {
  resolver ${{DNS_RESOLVER}} ipv6=off;
  charset UTF-8;
  error_log logs/error.log ${{LOG_LEVEL}};
  access_log logs/access.log;

  ... # etc
}


```

*   在 OpenResty 里嵌入 Kong

如果您正在运行您自己的 OpenResty 服务器，您也可以通过使用`include`指令（类似于上一节的示例）来轻松地嵌入 Kong。如果您有一个有效的顶级 NGINX 配置，那么就可以简单的引入 Kong 的配置：

```
# my_nginx.conf

http {
    include 'nginx-kong.conf';
}


```

你可以像这样开始你的实例：

```
$ nginx -p /usr/local/openresty -c my_nginx.conf


```

这样 Kong 就会运行在你的实例中。（Kong 的配置在`nginx-kong.conf`里）

*   Kong 为网站和你的 api 提供服务

API 提供者的一个常见用例是让 Kong 在代理端口上同时服务于一个网站和 API 本身——在生产中有`80`或`443`。例如：`https://my-api.com`（网站）和`https://my-api.com/api/v1`（API）。

为了实现这一点，我们不能简单地声明一个新的虚拟服务模块，就像我们在上一节中所做的那样。一个好的解决方案是使用一个定制的 Nginx 配置模板，该模板可以内联`nginx_kong.lua`。添加一个新的`location`块，与 Kong 代理`location`块一起服务于网站：

```
# ---------------------
# custom_nginx.template
# ---------------------

worker_processes ${{NGINX_WORKER_PROCESSES}}; # can be set by kong.conf
daemon ${{NGINX_DAEMON}};                     # can be set by kong.conf

pid pids/nginx.pid;                      # this setting is mandatory
error_log logs/error.log ${{LOG_LEVEL}}; # can be set by kong.conf
events {}

http {
  # here, we inline the contents of nginx_kong.lua
  charset UTF-8;

  # any contents until Kong's Proxy server block
  ...

  # Kong's Proxy server block
  server {
    server_name kong;

    # any contents until the location / block
    ...

    # here, we declare our custom location serving our website
    # (or API portal) which we can optimize for serving static assets
    location / {
      root /var/www/my-api.com;
      index index.htm index.html;
      ...
    }

    # Kong's Proxy location / has been changed to /api/v1
    location /api/v1 {
      set $upstream_host nil;
      set $upstream_scheme nil;
      set $upstream_uri nil;

      # Any remaining configuration for the Proxy location
      ...
    }
  }

  # Kong's Admin server block goes below
}


```

配置属性详解
------

可以新建配置文件`/etc/kong/kong.conf`进行添加修改。

### 常规属性

**prefix**

工作目录。相当于 Nginx 的前缀路径，包含临时文件和日志。每个流程必须有一个单独的工作目录。

默认：`/usr/local/kong`

**log_level**

Nginx 服务器的日志级别。可以在`<prefix>/logs/error.log`  
请参阅 [http://nginx.org/en/docs/ngx_core_module.html#error_log](http://nginx.org/en/docs/ngx_core_module.html#error_log)，以获得公认的值列表。

默认：`notice`

**proxy_access_log**

代理端口请求访问日志的路径。设置为`off`以禁用日志代理请求。如果这个值是相对路径，那么它将被放置于前缀路径之下。

默认：`logs/access.log`

**proxy_error_log**

代理端口请求错误日志的路径。这些日志的粒度由`log_level`指令进行调整。

默认：`logs/error.log`

**admin_access_log**

Admin API 的路径请求访问日志。设置为`off`以禁用 Admin API 请求日志。如果这个值是相对路径，那么它将被放置于前缀路径之下。

默认：`logs/admin_access.log`

**admin_error_log**

Admin API 请求错误日志的路径。这些日志的粒度由 log_level 指令进行调整。

默认：`logs/error.log`

**custom_plugins**

这个节点应该加载的附加插件的逗号分隔列表。使用这个属性来加载与 Kong 不捆绑的定制插件。插件将从`kong.plugins.{name}.*`命名空间加载。

默认：`none`  
示例：`my-plugin,hello-world,custom-rate-limiting`

**anonymous_reports**

发送匿名的使用数据，比如错误堆栈跟踪，以帮助改进 Kong。

默认：`on`

### Nginx 属性

**proxy_listen**

代理服务侦听的地址和端口的逗号分隔的列表。代理服务是 Kong 的公共入口点，它代理从您的使用者到您的后端服务的流量。这个值接受 IPv4、IPv6 和主机名。

可以为每一对指定一些后缀：

*   `ssl` 将要求通过启用 TLS 的特定地址 / 端口进行所有连接。
*   `http2` 允许客户端打开 http/2 连接到 Kong 的代理服务
*   最后, `proxy_protocol` 将为给定的地址 / 端口启用代理协议。

这个节点的代理端口，启用了 “控制面板” 模式（没有流量代理功能），可以配置连接到同一数据库的节点集群。

查看 [http://nginx.org/en/docs/http/ngx_http_core_module.html#listen](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen) 用于描述这个和其他`*_listen`值的接受格式。

默认：`0.0.0.0:8000, 0.0.0.0:8443 ssl`  
示例：`0.0.0.0:80, 0.0.0.0:81 http2, 0.0.0.0:443 ssl, 0.0.0.0:444 http2 ssl`  

**admin_listen**

管理接口监听的地址和端口的逗号分隔的列表。Admin 接口是允许您配置和管理 Kong 的 API。对该接口的访问应该仅限于 Kong 管理员。这个值接受 IPv4、IPv6 和主机名。可以为每一对指定一些后缀：

*   `ssl` 将要求通过启用 TLS 的特定地址 / 端口进行所有连接。
*   `http2` 允许客户端打开 http/2 连接到 Kong 的代理服务
*   最后, `proxy_protocol` 将为给定的地址 / 端口启用代理协议。

这个值可以被设置为`off`，从而禁用这个节点的 Admin 接口，从而使 “数据面板” 模式（没有配置功能）从数据库中拉出它的配置更改。

默认：`127.0.0.1:8001, 127.0.0.1:8444 ssl`  
示例：`127.0.0.1:8444 http2 ssl`

**nginx_user**

定义工作进程使用的用户和组凭据。如果省略组，则使用名称与用户名相同的组。

默认：`nobody nobody`  
示例：`nginx www`

**nginx_worker_processes**

确定 Nginx 生成的工作进程的数量。请参阅 [http://nginx.org/en/docs/ngx_core_module.html#worker](http://nginx.org/en/docs/ngx_core_module.html#worker) 流程，以便详细使用该指令和对已接受值的描述。

默认值:`auto`

**nginx_daemon**

确定 Nginx 是否会作为守护进程或前台进程运行。主要用于开发或在 Docker 环境中运行 Kong。

查阅 [http://nginx.org/en/docs/ngx_core_module.html#daemon](http://nginx.org/en/docs/ngx_core_module.html#daemon).

默认：`on`

**mem_cache_size**

数据库实体内存缓存的大小。被接受的单位是 k 和 m，最低推荐值为几个 MBs。

默认：`128m`

**ssl_cipher_suite**

定义 Nginx 提供的 TLS 密码。可接受的值`modern`, `intermediate`, `old`, or `custom`。请参阅 [https://wiki.mozilla.org/Security/Server_Side_TLS](https://wiki.mozilla.org/Security/Server_Side_TLS)  
，了解每个密码套件的详细描述。

默认值：`modern`

**ssl_ciphers**

定义一个由 Nginx 提供的 LTS ciphers 的自定义列表。这个列表必须符合`openssl ciphers`定义的模式。如果`ssl_cipher_suite`不是`custom`，那么这个值就会被忽略。

默认值：`none`

**ssl_cert**

启用 SSL 时，`proxy_listen`的 SSL 证书的绝对路径。

默认值：`none`

**ssl_cert_key**

启用 SSL 时，`proxy_listen`的 SSL key 的绝对路径。

默认值：`none`

**client_ssl**

当代理请求时，确定 Nginx 是否应该发送客户端 SSL 证书。

默认值：`off`

**client_ssl_cert**

如果启用了 client_ssl，用于`proxy_ssl_certificate`配置的客户端 SSL 证书的绝对路径。注意，这个值是静态地在节点上定义的，并且当前不能在每个 api 的基础上配置。

默认值：`none`

**client_ssl_cert_key**

如果启用了 client_ssl，用于`proxy_ssl_certificate_key`配置的客户端 SSL 证书的绝对路径。注意，这个值是静态地在节点上定义的，并且当前不能在每个 api 的基础上配置。

默认值：`none`

**admin_ssl_cert**

启用了 SSL 后， `admin_listen` 的 SSL 证书的绝对路径。

默认值：`none`

**admin_ssl_cert_key**

启用了 SSL 后， `admin_listen` 的 SSL key 的绝对路径。

默认值：`none`

**upstream_keepalive**

在每个工作进程，设置缓存中保存的 upstream 服务的空闲 keepalive 连接的最大数量。当超过这个数字时，会关闭最近最少使用的连接。

默认值：`60`

**server_tokens**

在错误页面，和`Server`或`Via`（如果请求被代理）的响应头字段，启用或禁用展示 Kong 的版本。

默认值：`on`

**latency_tokens**

在`X-Kong-Proxy-Latency`和`X-Kong-Upstream-Latency`响应头字段中，启用或禁用展示 Kong 的潜在信息。

默认值：`on`

**trusted_ips**

定义可信的 IP 地址块，使其知道如何发送正确的 `X-Forwarded-*` 头部信息。来自受信任的 ip 的请求使 Kong 转发他们的 `X-Forwarded-*` headers upstream。不受信任的请求使 Kong 插入自己的 `X-Forwarded-*` headers。

该属性还在 Nginx 配置中设置 `set_real_ip_from` 指令（s）。它接受相同类型的值（CIDR 块），但它是一个逗号分隔的列表。

如果相信 _all_ /!\ IPs，请把这个值设为`0.0.0.0/0,::/0`。

如果特殊值`unix:`被指定了，所有的 unix 域套接字都将被信任。

查阅 [the Nginx docs](http://nginx.org/en/docs/http/ngx_http_realip_module.html#set_real_ip_from) 了解 更详细的`set_real_ip_from`配置资料。

Default: `none`

**real_ip_header**

定义请求头字段，它的值将被用来替换客户端地址。在 Nginx 配置中使用相同名称的指令 [ngx_http_realip_module](http://nginx.org/en/docs/http/ngx_http_realip_module.html) 设置该值。

如果这个值接收到 `proxy_protocol`，那么 `proxy_protocol` 参数将被附加到 Nginx 模板的 `listen` 指令中。

查阅 [the Nginx docs](http://nginx.org/en/docs/http/ngx_http_realip_module.html#real_ip_header) 寻找更详细的描述。

默认值： `X-Real-IP`

**real_ip_recursive**

该值设置了 Nginx 配置中同名的 [ngx_http_realip_module](http://nginx.org/en/docs/http/ngx_http_realip_module.html) 指令。

查阅 [the Nginx docs](http://nginx.org/en/docs/http/ngx_http_realip_module.html#real_ip_recursive) 寻找更详细的描述。

默认值： `off`

**client_max_body_size**

指定在 Content-Length 的请求头中，定义 Kong 代理的请求的最大被允许的请求体大小。如果请求超过这个限度，Kong 将返回 413(请求实体太大)。将该值设置为 0 将禁用检查请求体的大小。

提示: 查阅关于 [the Nginx docs](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_max_body_size) 这个参数的进一步描述。数值可以用`k`或`m`后缀，表示限制是千字节，还是兆字节。

默认值：`0`

**client_body_buffer_size**

定义读取请求主体的缓冲区大小。如果客户端请求体大于这个值，则阀体将被缓冲到磁盘。请注意，当阀体被缓冲到磁盘的时候，访问或操纵请求主体可能无法工作，因此最好将这个值设置为尽可能高的值。（例如，将其设置为`client_max_body_size`，以迫使请求体保持在内存中）。请注意，高并发性环境需要大量的内存分配来处理许多并发的大型请求体。

提示: 查阅关于 [the Nginx docs](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_buffer_size) 这个参数的进一步描述。数值可以用`k`或`m`后缀，表示限制是千字节，还是兆字节。

默认值：`8k`

**error_default_type**

当请求 Accept 标头丢失时，使用默认的 MIME 类型，且 Nginx 为这个请求返回一个错误。可接受的值包括 `text/plain`, `text/html`, `application/json`, 和`application/xml`.

默认值：`text/plain`

### 数据存储属性

Kong 将存储所有的数据（如 api、消费者和插件）到 Cassandra 或 PostgreSQL。

属于同一集群的所有 Kong 节点都必须连接到同一个数据库。

> 从 Kong0.12.0 开始：  
> PostgreSQL 9.4 支持应该被认为是弃用。鼓励用户升级到 9.5+  
> 应该考虑支持 Cassandra 2.1 的支持。鼓励用户升级到 2.2+

**database**

确定这个节点将使用哪个 PostgreSQL 或 Cassandra 作为它的数据存储。可以设置为：`postgres` 和 `cassandra`。

默认值：`postgres`

**Postgres settings**

<table><thead><tr><th>名称</th><th>描述</th></tr></thead><tbody><tr><td>pg_host</td><td>Postgres 服务器的主机</td></tr><tr><td>pg_port</td><td>Postgres 服务器的端口</td></tr><tr><td>pg_user</td><td>Postgres 用户</td></tr><tr><td>pg_password</td><td>Postgres 用户的密码</td></tr><tr><td>pg_database</td><td>数据库连接。必须存在</td></tr><tr><td>pg_ssl</td><td>启用 SSL 连接到服务器</td></tr><tr><td>pg_ssl_verify</td><td>如果启用了<code>pg_ssl</code>，则切换服务器证书验证。看到<code>lua_ssl_trusted_certificate</code>设置。</td></tr></tbody></table>

**Cassandra settings**

<table><thead><tr><th>名称</th><th>描述</th></tr></thead><tbody><tr><td>cassandra_contact_points</td><td>指向您的 Cassandra 集群的链接点列表，使用逗号分割。</td></tr><tr><td>cassandra_port</td><td>你的节点监听的端口</td></tr><tr><td>cassandra_keyspace</td><td>在集群中使用的关键空间。如果不存在，就会被创建。</td></tr><tr><td>cassandra_consistency</td><td>在阅读 / 写作时使用一致性设置。</td></tr><tr><td>cassandra_timeout</td><td>读取 / 写入 超时（ms）时间。</td></tr><tr><td>cassandra_ssl</td><td>启用 SSL 连接到节点。</td></tr><tr><td>cassandra_ssl_verify</td><td>如果启用了<code>cassandra_ssl</code>，则切换服务器证书验证。查看 <code>lua_ssl_trusted_certificate</code> 设置。</td></tr><tr><td>cassandra_username</td><td>使用 PasswordAuthenticator 时的用户名。</td></tr><tr><td>cassandra_password</td><td>在使用 PasswordAuthenticator 时的密码。</td></tr><tr><td>cassandra_consistency</td><td>在读取 / 写入 Cassandra 集群时使用一致性设置。</td></tr><tr><td>cassandra_lb_policy</td><td>当在您的 Cassandra 集群中分布查询时使用负载平衡策略。可设置为 <code>RoundRobin</code> 和 <code>DCAwareRoundRobin</code> 。如果您使用的是多数据中心集群，则后者更好。如果是这样，还要设置 <code>cassandra_local_datacenter</code>。</td></tr><tr><td>cassandra_local_datacenter</td><td>在使用<code>DCAwareRoundRobin</code>政策时，必须指定本地（最近）的集群名称到这个 Kong 节点。</td></tr><tr><td>cassandra_repl_strategy</td><td>如果第一次创建密钥空间，请指定复制策略。</td></tr><tr><td>cassandra_repl_factor</td><td>为<code>SimpleStrategy</code>指定一个复制因子。</td></tr><tr><td>cassandra_data_centers</td><td>为<code>NetworkTopologyStrategy</code>(网络拓扑策略) 指定数据中心。</td></tr><tr><td>cassandra_schema_consensus_timeout</td><td>Cassandra 节点之间同步 scheme 的超时（ ms）时间。这个值只在数据迁移期间使用。</td></tr></tbody></table>

### 数据缓存属性

为了避免与数据存储不必要的通信，Kong 可配置缓存实体（比如 api、消费者、凭证等等）的间隔时间。如果缓存实体被更新，它也会处理也会失效。

本节介绍关于配置 Kong 此类配置实体缓存。

**db_update_frequency**

频率（以秒为单位），用于检查带有数据存储的更新实体。当节点通过 Admin API 创建、更新或删除实体时，其他节点需要等待下一次轮询（由这个值配置），以清除旧的缓存实体并开始使用新的缓存。

默认值：5 seconds

**db_update_propagation**

在数据存储中为实体所花费的时间（以秒为单位）被传播到另一个数据中心的副本节点。当在分布式环境中，比如多数据中心 Cassandra 集群时，这个值应该是 Cassandra 将一行传播到其他数据中心的最大秒数。当设置了该值，该属性将增加 Kong 传播实体变更所花费的时间。单数据中心设置或 PostgreSQL 服务器不应该受到这样的延迟，并且这个值可以安全地设置为 0。

默认值: 0 seconds

**db_cache_ttl**

该节点数据存储实体缓存的生存时间（以秒为单位）。数据库遗漏（没有实体）也会根据这个设置进行缓存。如果设置为 0，那么这种缓存的实体 / 遗漏永远不会过期。

默认值：3600 seconds（1 小时）

### DNS 解析属性

Kong 将把主机名解析为 `SRV` 或 `A` 记录（按照该顺序，`CNAME` 记录将在此过程中被取消）。如果一个名称被解析为`SRV`记录，它会通过从 DNS 服务器接收到端口以覆盖给定的端口号。

DNS 选项`SEARCH`和`NDOTS`（来自 / etc/resolv.conf 文件）将被用于将短名称扩展到完全限定的名称。因此，它将首先尝试完整 `SEARCH` `SRV`类型的列表，如果失败，它将会尝试`SEARCH` `A`记录列表，等等。

在`ttl`的持续时间内，内部 DNS 解析器将对 DNS 记录的条目上做负载均衡请求。对于`SRV`记录，可以设置权重，但是它只会使用记录中最低优先级字段条目。

**dns_resolver**

设置域名服务器列表，使用逗号分隔。格式如： `ip[:port]` 。如果没有制定域名服务器，name 就使用本地 `resolv.conf` 文件。端口默认为 53。可以使用 IPv4 和 IPv6 地址。

默认值： none

**dns_hostsfile**

要使用的主机文件。这个文件只被读取一次，然后会存储在内存中。要在修改后想再次读取该文件，必须重新加载 Kong。

默认值：`/etc/hosts`

**dns_order**

解决不同记录类型的顺序。`LAST`类型指的是最后一次成功的查找的类型（对于指定的名称）。格式是一个（大小写不敏感）逗号分隔的列表。

默认值： `LAST`,`SRV`,`A`,`CNAME`

**dns_stale_ttl**

定义在缓存中保存 DNS 记录的`TTL`时间。当新的 DNS 记录在后台获取时，这个值将被使用。陈旧的数据将从记录的过期时间使用，直到刷新查询完成，或者`dns_stale_ttl`的秒数已经过去。

默认值：`4`

**dns_not_found_ttl**

空 DNS 响应和 "(3) name error" 响应的 TTL 时间（以秒为单位）

默认值：`30`

**dns_error_ttl**

错误响应的 TTL 时间（以秒为单位）

默认值：`1`

**dns_no_sync**

如果启用了，那么在 cache-miss 时，每个请求都会触发自己的 dns 查询。当为相同的名称 / 类型禁用多个请求时，将同步到单个查询。

默认值： off

### 开发与其他属性

从 lua-nginx-module 继承的附加设置，可以更灵活和更高级的使用。  
有关更多信息，请参见 lua-nginx-module 文档：[https://github.com/openresty/lua-nginx-module](https://github.com/openresty/lua-nginx-module)

**lua_ssl_trusted_certificate**

在 PEM 格式的 Lua cosockets 的证书权威文件的绝对路径。该证书将用于验证 Kong 的数据库连接，当启用`pg_ssl_verify`或`cassandra_ssl_verify`时。

详情查阅：[https://github.com/openresty/lua-nginx-module#lua_ssl_trusted_certificate](https://github.com/openresty/lua-nginx-module#lua_ssl_trusted_certificate)

默认值： none

**lua_ssl_verify_depth**

在 Lua cosockets 使用的服务器证书链中设置验证深度，通过`lua_ssl_trusted_certificate` 设置。

这包括为 Kong 的数据库连接配置的证书。

详情查阅： [https://github.com/openresty/lua-nginx-module#lua_ssl_verify_depth](https://github.com/openresty/lua-nginx-module#lua_ssl_verify_depth)

默认值: `1`

**lua_package_path**

设置 Lua 模块搜索路径（LUA_PATH）。在默认搜索路径中，开发或使用不存储的自定义插件时非常有用。

详情查阅：[https://github.com/openresty/lua-nginx-module#lua_package_path](https://github.com/openresty/lua-nginx-module#lua_package_path)

默认值： none

**lua_package_cpath**

设置 Lua C 模块搜索路径（LUA_CPATH）。

详情查阅：[https://github.com/openresty/lua-nginx-module#lua_package_cpath](https://github.com/openresty/lua-nginx-module#lua_package_cpath)

默认值： none

**lua_socket_pool_size**

指定与每个远程服务器相关联的每个 cosocket 连接池的大小限制。

详情查阅：[https://github.com/openresty/lua-nginx-module#lua_socket_pool_size](https://github.com/openresty/lua-nginx-module#lua_socket_pool_size)

默认值：`30`

> 穿梭机：[开源 API 网关系统（Kong 教程）入门到精通](https://www.jianshu.com/p/a68e45bcadb6)