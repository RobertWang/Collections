> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/52402537)

近期基于 kong 网关开发了几个小插件，感觉非常方便，而且流程简单明了。

**kong 插件项目概要**
---------------

下载官方插件项目开发模版

```
git clone https://github.com/Kong/kong-plugin.git

```

目录结构如下：

```
$ tree
.
├── LICENSE
├── README.md
├── kong
│   └── plugins
│       └── myplugin
│           ├── handler.lua
│           └── schema.lua
├── kong-plugin-myplugin-0.1.0-1.rockspec
└── spec
    └── myplugin
        └── 01-access_spec.lua
5 directories, 6 files

```

kong 插件主要有三个文件：

handler.lua 是包含插件逻辑处理相关代码。 schema.lua 包含插件的配置文件。 rockspec 文件是通过 luarock 安装时用的配置文件。

逻辑处理的代码根据 openresty 的不同处理阶段分成了不同的函数，根据插件的功能只需要在不同的函数中添加自己的业务逻辑。

**kong 插件项目实践**
---------------

下面是开发一个最简单的灰度发布使用的流量分发插件的全部流程。

这个插件的功能非常简单：

根据 http request 头中的 Authorization 的值将流量分发到不同的后端服务器。

插件有两个配置项：pattern 和 upstream，如果 Authorization 的值匹配 pattern，那么当前请求会被代理到相应的 upstream 中。

OK，开始！

**代码开发**
--------

给插件起个温暖的名字，就叫 huidu 吧，修改模版项目中的目录名为 huidu。

```
mv kong-plugin kong-plugin-huidu
cd kong-plugin-huidu/kong/plugins
mv myplugin huidu

```

**修改 schema.lua 文件，添加插件配置代码。**

```
local Errors = require "kong.dao.errors"

return {
  no_consumer = false, 
  -- 添加插件的配置项
  fields = {
    -- Describe your plugin's configuration's schema here.
    pattern = {type = "string", required = true},
    upstream = {type = "string", required = true}
  },
  -- 对输入的配置项进行合法性检查。
  self_check = function(schema, plugin_t, dao, is_updating)
    -- perform any custom verification
    local pattern = plugin_t.pattern
    if #pattern == 0 then
      return false, Errors.schema("pattern must not be null")
    end

    if pattern:sub(1, 1) ~= '^' then
      return false, Errors.schema("pattern must start with ^")
    end

    local upstream = plugin_t.upstream
    if #upstream == 0 then
      return false, Errors.schema("upstream must not be null")
    end

    return true
  end,
}

```

**修改 handler.lua 文件，添加插件处理逻辑。**

```
---[[ runs in the 'access_by_lua_block'
function plugin:access(plugin_conf)
  plugin.super.access(self)

  -- your custom code here
  local pattern = plugin_conf.pattern
  local token = kong.request.get_header("authorization")
  if token == nil then
    return 
  end
  -- 忽略大小写
  local matched = ngx.re.match(token, pattern, "joi")
  if matched then
    -- 设置upstream
    local ok, err = kong.service.set_upstream(plugin_conf.upstream)
    if not ok then
        kong.log.err(err)
        return
    end
    -- 匹配成功添加特定头部方便监控
    ngx.req.set_header("X-Kong-" .. plugin_name .. "-upstream", plugin_conf.upstream)
    ngx.req.set_header("X-Kong-" .. plugin_name .. "-pattern", plugin_conf.pattern)
  end    

end --]]

```

huidu 插件的逻辑只需要在 access 阶段执行就可以，可以把多余的注释和代码删掉。

**Done!**

**安装调试**
--------

假设 kong 的本地开发环境已经搭建好了，在调试阶段插件的安装采用手动指定目录的方式安装。

修改 kong.conf 文件，添加或者修改下面的配置项。

```
# bundled 表示kong内置的所有插件
plugins = bundled,huidu
# 添加huidu插件目录绝对路径
lua_package_path = /Users/youfu/kong-plugin-huidu/?.lua;./?.lua;./?/init.lua;;

```

修改完成后保存启动 kong

```
kong start -c kong.conf --vv

```

在启动的输出日志中可以看到加载了刚刚配置的 huidu 插件。

使用 web 管理界面 konga 来配置插件。创建一个 service，然后在 service 下安装当前插件。

安装后插件的配置如下：

```
{
  "created_at": 1544686913000,
  "config": {
    "upstream": "mocktest",
    "pattern": "^a"
  },
  "id": "87712e00-9ae8-49c0-9436-0d4d431d8523",
  "name": "huidu",
  "service_id": "a6dae27a-5f1a-478d-a0a1-f86475f0be7a",
  "enabled": true
}

```

配置项表示以 a 开头的全部流量走 mocktest，如果没有匹配或者插件代码有错误，那么流量就走 service 本身配置的 upstream。

其中 mocktest 配置的 target 为 [http://mockbin.org:80](https://link.zhihu.com/?target=http%3A//mockbin.org%3A80)。

配置后的数据会保存在 kong 的后端数据库的 plugins 表中。

**测试一把！**

请求匹配的情况

```
curl -X GET 'http://127.0.0.1:8000/request?foo=bar&foo=baz' -H 'authorization: ApwPYjA8J1c9CVaFIBVkRyYV57k2bXfHFH9Jojs8HN2FfyEn39MzV1afG9'

```

响应如下：

```
{
  "headers": {
    "host": "mockbin.org”,
    ...
    "authorization": "ApwPYjA8J1c9CVaFIBVkRyYV57k2bXfHFH9Jojs8HN2FfyEn39MzV1afG9",
    "x-kong-huidu-upstream": "mocktest",
    "x-kong-huidu-pattern": "^a",
    "x-request-id": "e8a4eb07-66c4-434e-aa17-c5815c66820f",
  }
  ...
}

```

请求不匹配的情况

```
curl -X GET 'http://127.0.0.1:8000/request?foo=bar&foo=baz' -H 'authorization: XApwPYjA8J1c9CVaFIBVkRyYV57k2bXfHFH9Jojs8HN2FfyEn39MzV1afG9'

```

响应如下：

```
{"message":"failure to get a peer from the ring-balancer"}

```

可以看到插件已经生效。

**正式环境部署**
----------

正式环境通过 luarocks 来部署。

编辑 kong-plugin-huidu-0.1.0-1.rockspec 文件，修改下面几个配置

```
package = "kong-plugin-huidu”
source = {
  url = "https://github.com/chenyoufu/kong-plugin-huidu.git",
  tag = "0.1.0"
}

```

下面是本地 mac 上安装的情况

```
$ luarocks make —-verbose

...
kong-plugin-huidu 0.1.0-1 is now installed in /usr/local/opt/kong (license: Apache 2.0)

```

mac 下安装完成后 lua 代码会安装到

```
/usr/local/opt/kong/share/lua/5.1/kong/plugins/huidu

```

linux 会安装到这个目录下

```
/usr/local/share/lua/5.1/kong/plugins/huidu

```

这样就不需要在 kong.conf 中配置绝对路径了。

安装完成后需要运行 kong restart 命令才能生效。

**后记**
------

这个示例插件不涉及数据库以及其他复杂的操作，属于最简单的类型，建议在开发之前还是要通读官方的插件开发文档和 kong 本身自带的插件库的源代码。

基于这个流程基本上简单的功能的插件可以做到一天一个。

不建议网关的插件功能逻辑太复杂，简单够用就行。

当前示例插件代码 [https://github.com/chenyoufu/kong-plugin-huidu.git](https://link.zhihu.com/?target=https%3A//github.com/chenyoufu/kong-plugin-huidu.git)

官方插件开发文档 [https://docs.konghq.com/0.14.x/plugin-development/](https://link.zhihu.com/?target=https%3A//docs.konghq.com/0.14.x/plugin-development/)