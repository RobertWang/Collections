> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jb51.net](https://www.jb51.net/article/114552.htm)

> 本篇文章主要介绍了详解 VueJs 前后端分离跨域问题，详细介绍了在项目内设置代理（proxyTable）的方式来解决跨域问题，有兴趣的可以了解一下

 更新时间：2017 年 05 月 24 日 10:28:51   作者：我叫陈小宝 T_T  

本篇文章主要介绍了详解 VueJs 前后端分离跨域问题，详细介绍了在项目内设置代理（proxyTable）的方式来解决跨域问题，有兴趣的可以了解一下

**使用 vue-cli 搭建的 vue 项目**

可以使用在项目内设置代理（proxyTable）的方式来解决跨域问题

设置配置项的目录在 config 下的 index.js，主要通过配置 proxyTable 项，设置代理指向你的后台地址

```
dev: {

  env: require('./dev.env'),

  port: 8085,

  autoOpenBrowser: true,

  assetsSubDirectory: 'static',

  assetsPublicPath: '/',

  proxyTable: {

   '/agent': {

    changeOrigin: true,

    pathRewrite: {

     '^/agent': ''

    }

   }

  },

  // CSS Sourcemaps off by default because relative paths are "buggy"

  // with this option, according to the CSS-Loader README

  // In our experience, they generally work as expected,

  // just be aware of this issue when enabling this option.

  cssSourceMap: false

 }
```

前端使用 vue-resource 来发起请求时

```
Vue.prototype.rootUrl = '/agent/';

that.$http.post(this.rootUrl + 'login', parms).then(function (response) {

     console.log(response);

    }, function (response) {

    });
```

其他方式搭建的前端项目，通过使用 nginx 启动前端服务同时配置代理

下列是我的 nginx 配置文件，不管是通过什么方式搭建的前端项目，构建成功后都会输出一个 dist 文件，我们只需要将 nginx 服务目录指向你的 dist 文件下你项目的入口文件即可

我的文件目录是 root D:\openplatform\portal\webapp\dist; 更改此条配置到你的目录 我入口文件名称是 index.html 使用的是 vue-cli 打包的项目，参考 vue-cli npm run build 的 dist 目录，指向那个目录下

```
worker_processes 4;

events {

  worker_connections 1024;

}

http {

  include    mime.types;

  default_type application/octet-stream;

  log_format main '[$time_iso8601] [$remote_addr] [$request] [$http_user_agent] [$cookie_customerID_cookie_flag] [$args]';

  access_log logs/access.log main;

  sendfile    on;

  keepalive_timeout 65;

  gzip on;

  gzip_min_length 1k;

  gzip_buffers 4 16k;

  gzip_http_version 1.0;

  gzip_comp_level 3;

  gzip_proxied any;

  gzip_types *;

  server {

    listen     80;

    root D:\openplatform\portal\webapp\dist;

    index index.html;

    location / {

       try_files $uri $uri/ @router;

       index index.html;

      }

  location @router {

       rewrite ^.*$ /index.html last;

    }

  location ^~/agent/ {

      proxy_pass  http://127.0.0.1:7105/;

      proxy_redirect  http://127.0.0.1:7105/ /;

      proxy_set_header Host $host;

      proxy_set_header X-Real-IP $remote_addr;

      proxy_set_header REMOTE-HOST $remote_addr;

      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

      proxy_connect_timeout 600s;

      proxy_read_timeout 600s;

      proxy_send_timeout 600s;

    }

  }

}
```

以上就是本文的全部内容，希望对大家的学习有所帮助，也希望大家多多支持脚本之家。

微信公众号搜索 “ 脚本之家 ” ，选择关注

程序猿的那些事、送书等活动等着你

原文链接：http://www.jianshu.com/p/bfe5a43eb95b