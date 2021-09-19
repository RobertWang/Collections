> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7008806167170252808)

个人记录 参考了很多文章

> 系统环境：win10

首先安装 ffmpeg 地址: [ffmpeg](https://link.juejin.cn?target=http%3A%2F%2Fwww.ffmpeg.org%2Fdownload.html "http://www.ffmpeg.org/download.html")
-----------------------------------------------------------------------------------------------------------------------------------------

根据你的系统选择下载 win linux

下载好后解压到你想安装的目录, 然后打开系统配置环境变量

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8760a8284ace449cb7c7a130bb1679b4~tplv-k3u1fbpfcp-watermark.awebp?)

打开 cmd 窗口 输入 ffmpeg –version 显示下面的版本号 代表安装成功

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ab2539e745c04073966107ee2f3ff692~tplv-k3u1fbpfcp-watermark.awebp?)

安装带有 rtmp 的 nginx 地址: [nginx-rtmp-module](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Farut%2Fnginx-rtmp-module "https://github.com/arut/nginx-rtmp-module")
------------------------------------------------------------------------------------------------------------------------------------------------------------------------

打开 conf 下的 nginx.conf 进行配置 这里是我的配置文件内容

```
#user  nobody;
worker_processes  1;
 
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
 
#pid        logs/nginx.pid;
 
 
events {
    worker_connections  1024;
}
rtmp {
 
    server {
 
        listen 1935; 
 
        chunk_size 4000;
 
        # TV mode: one publisher, many subscribers
        application mylive {
 
            # enable live streaming
            live on;
 
        }
    }
}
 
http {
    include       mime.types;
    default_type  application/octet-stream;
 
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
 
    #access_log  logs/access.log  main;
 
    sendfile        on;
    #tcp_nopush     on;
 
    #keepalive_timeout  0;
    keepalive_timeout  65;
 
    #gzip  on;
 
    server {
        listen       20000;
        server_name  localhost;
 
        #charset koi8-r;
 
        #access_log  logs/host.access.log  main;
 
        location / {
            root   html;
            index  index.html index.htm;
        }
 
        #error_page  404              /404.html;
 
        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
 

 
}

复制代码
```

这里是后续需要用的的参数

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/885420b860854397bf13c568b559b379~tplv-k3u1fbpfcp-watermark.awebp?)

安装 nginx-http-flv-module 地址: [nginx-http-flv-module](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fwinshining%2Fnginx-http-flv-module "https://github.com/winshining/nginx-http-flv-module")
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

通过服务端将上一步的 RTMP 流视频转成 http-flv 流, 转为 web 端可以使用的格式 (由于 chrome 禁止了 Flash 所以 RTMP 并不能直接播放)

```
worker_processes  1;

events {
    worker_connections  1024;
}

rtmp {
    server {
        listen 1985;

        application myapp {
            live on;
        }
    }
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;


    server {
        listen       80;
        server_name  localhost;

        location / {
            add_header 'Access-Control-Allow-Origin' '*';
            root html;
            index  index.html index.htm;
        }

        location /live {
            flv_live on;
        }

        location /flv {
            add_header 'Access-Control-Allow-Origin' '*';
            flv_live on;
            chunked_transfer_encoding on;
        }
        
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
复制代码
```

第一步 打开两个 nginx
==============

双击 nginx-rtmp.exe (rtmp nginx)

双击 nginx.exe (flv nginx)

任务管理器出现图中 4 个进程即可

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1349a300f7f94c6da0cf6a6abf43239c~tplv-k3u1fbpfcp-watermark.awebp?)

第二步 打开两个 cmd 窗口
===============

这一步 是将 rtsp 的原始摄像头数据 通过 nginx 转成 rtmp 格式 (admin:[XXXXXX@192.168.XXX.XXX](https://link.juejin.cn?target=mailto%3AXXXXXX%40192.168.XXX.XXX "mailto:XXXXXX@192.168.XXX.XXX")) 通常说的是摄像头地址 (这个根据厂家摄像头来定义) 后面的 :1935/mylive/test2 对应着 nginx 配置中的本机地址

```
ffmpeg -i "rtsp://admin:XXXXXX@192.168.XXX.XXX"  -vcodec copy -acodec copy -f flv "rtmp://192.168.200.225:1935/mylive/test2"
复制代码
```

在 cmd 输入这串指令后 出现下面即代表成功 这时候用 VLC 访问 rtmp://192.168.200.225:1935/mylive/test2 也能看见视频画面, 但是 由于 chrome 不支持 flash 了, 所以 rtmp 并不能在前端展示.

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1b8c35c5b1624e7786f0ce7c018bed2a~tplv-k3u1fbpfcp-watermark.awebp?)

这时候我们需要使用 nginx-http-flv-module 将 rtmp 流 推成 http-flv 流

在第二个 cmd 窗口中输入

rtmp://192.168.200.225:1935/mylive/test2 是上一个 cmd 推成的地址

rtmp://192.168.200.225:1985/myapp/testv 是转后的地址

```
ffmpeg -re -i rtmp://192.168.200.225:1935/mylive/test2 -c copy -f flv rtmp://192.168.200.225:1985/myapp/testv
复制代码
```

在 cmd 输入这串指令后 出现下面即代表成功

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d81bac70631b46fc8aa997563b485ee9~tplv-k3u1fbpfcp-watermark.awebp?)

配置完成后 其中 myapp 和 testv 代表 nginx 你配置的名字 port 是 nginx 配置的端口号

这里是转播的 http-flv 地址

```
http://xx.xxx.xx.xx/live?port=1985&app=myapp&stream=testv // 原始转播地址
http://xx.xxx.xx.xx/flv?port=1985&app=myapp&stream=testv // flv 转播地址
复制代码
```

这里就可以用 web 端进行访问, 但是需要哔哩哔哩的 flv 进行解析

我这里是使用了 vue + flv.js 实现的

首先引入 flv.js 到项目中

```
<template>
  <div>
    <video autoplay controls width="100%" height="500" id="myVideo"></video>
  </div>
</template>

<script>
import flvjs from "flv.js";

export default {
  props: {
    msg: String,
  },
  mounted() {
    this.$nextTick(() => {
      this.videoPlayer();
    });
  },
  methods: {
    videoPlayer() {
      if (flvjs.isSupported()) {
        var videoElement = document.getElementById("myVideo");
        var flvPlayer = flvjs.createPlayer({
          type: "flv",
          url: "http://192.168.200.225/flv?port=1985&app=myapp&stream=testv", //你的url地址
        });
        flvPlayer.attachMediaElement(videoElement);
        flvPlayer.load();
        flvPlayer.play();
      }
    },
  },
};
</script>
复制代码
```

然后打开网页 就能有摄像头的内容了

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/436911d9936d4f9bac9437fded34ba52~tplv-k3u1fbpfcp-watermark.awebp?)

我这里只是自己折腾了一下子

摄像头延迟问题还需要解决

并且两个 nginx 可以不需要同时打开 可以配置到一起 这里希望大佬们能给我提供一下思路 后续我会继续优化下去的