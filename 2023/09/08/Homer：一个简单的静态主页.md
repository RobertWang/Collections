> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/RklEWr3DWEvhNWHFABY0UA)

> Homer 是一个完全静态的 html/js 仪表板。

**什么是 Homer ?**

> `Homer` 是一个完全静态的 `html/js` 仪表板，基于一个简单的 `yaml` 配置文件。它旨在由 `HTTP` 服务器提供服务，如果您直接通过 `file://` 协议打开 `index.html`，它将无法工作。

安装
==

在群晖上以 Docker 方式安装。

在注册表中搜索 `homer` ，选择第一个 `b4bz/homer`，版本选择 `latest` 或者 `v23.02.02`。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaEq4YZd9VULe1ptmeC99j2n10Z2p1Vmbm0ic8Y2xsHm4QzmbhrhWV2YbIDxc26v7H7BS6pVwcQaSVA/640?wx_fmt=png)

卷
-

在 `docker` 文件夹中，创建一个新文件夹 `homer`，并在其中建一个子文件夹`assets`, 需要给 `assets` 增加 `Everyone` 的读写权限

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaEq4YZd9VULe1ptmeC99j2ney0ekKIesVLhQEX5006DPYL8Nuvic5LNGH5J4aT2MyAeyBjr7BVy3lg/640?wx_fmt=png)

<table><thead><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: center;">文件夹</th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: center;" class="">装载路径</th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: center;">说明</th></tr></thead><tbody><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;"><code>docker/homer/assets</code></td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;"><code>/www/assets</code></td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;">存放设置文件</td></tr></tbody></table>

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaEq4YZd9VULe1ptmeC99j2nknHGFzEhl8qRc8ty6pzNXJFY83h6xcAsuPUlno3iaTyZarvYibeZpkNg/640?wx_fmt=png)

端口
--

本地端口不冲突就行，不确定的话可以用命令查一下

```
# 查看端口占用
netstat -tunlp | grep 端口号


```

<table><thead><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: center;">本地端口</th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: center;">容器端口</th></tr></thead><tbody><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;"><code>8058</code></td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;"><code>8080</code></td></tr></tbody></table>

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaEq4YZd9VULe1ptmeC99j2nwwrdb0JZWY2F85zU1Om0ZQDjcjVKoO9V1JFYSp0XgyXJI5oTEEpd4Q/640?wx_fmt=png)

环境
--

<table><thead><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: center;">可变</th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: center;">值</th></tr></thead><tbody><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;"><code>INIT_ASSETS</code></td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;">缺省值为 <code>1</code></td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;"><code>SUBFOLDER</code></td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;">缺省值为 <code>null</code></td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;"><code>PORT</code></td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;">缺省值为 <code>8080</code></td></tr></tbody></table>

*   `INIT_ASSETS`：默认为 `1`，会安装示例配置文件来帮助您入门。
    
*   `SUBFOLDER`：如果您想在子文件夹中托管 `Homer`，（例如：`http://my-domain/homer`），将 `SUBFOLDER`设置为子文件夹路径（`/homer`）。
    
*   `PORT`：如果您想将 `Homer` 的内部端口从默认的 `8080`更改为您选择的端口。
    

以上几个参数，老苏都采用的默认值

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaEq4YZd9VULe1ptmeC99j2n1qdoBZrdEtbEKxLib0eugPmopwFBraytkRyGsxMQWj6hTZlxwjLx0Bw/640?wx_fmt=png)

命令行安装
=====

如果你熟悉命令行，可能用 `docker cli` 更快捷

```
# 新建文件夹 homer 和 子目录
mkdir -p /volume2/docker/homer/assets

# 进入 homer 目录
cd /volume2/docker/homer

# 修改目录权限
chmod 777 assets

# 运行容器
docker run -d \
   --restart unless-stopped \
   --name homer \
   -p 8058:8080 \
   -v $(pwd)/assets:/www/assets \
   b4bz/homer:latest


```

也可以用 `docker-compose` 安装，将下面的内容保存为 `docker-compose.yml` 文件

```
version: "2"

services:
  homer:
    image: b4bz/homer
    container_name: homer
    volumes:
      - ./assets/:/www/assets
    ports:
      - 8058:8080
    user: 1000:1000 
    environment:
      - INIT_ASSETS=1 # default


```

然后执行下面的命令

```
# 新建文件夹 homer 和 子目录
mkdir -p /volume2/docker/homer/assets

# 进入 homer 目录
cd /volume2/docker/homer

# 修改目录权限
chmod 777 assets

# 将 docker-compose.yml 放入当前目录

# 一键启动
docker-compose up -d


```

运行
==

在浏览器中输入 `http://群晖IP:8058` 就能看到主界面

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaEq4YZd9VULe1ptmeC99j2nMrmPKIjiaSE5olWxV3Gjics1u8U7x3nVbwdyph8C0p3XYldIoNlcDfBw/640?wx_fmt=png)

因为使用了默认的环境变量，所以安装了示例，进入 `assets`，找到 `config.yml` 文件

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaEq4YZd9VULe1ptmeC99j2nib5zMEfzNWekaAMaw9rnTgX4thSvQ9pibg5ezPut1gZJ6kic4QHROVCbQ/640?wx_fmt=png)

其中 `links` 节对应于网页的导航条部分，而 `services` 节对应于我们要设置的书签

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaEq4YZd9VULe1ptmeC99j2neVA9nMkj9mcbxic2Iy9GSu0JsvaUKlGhxicc5yVreAPiata2aHbLP2EXQ/640?wx_fmt=png)

老苏将 `links` 部分改成了下面这样

> 记得将 `config.yml` 的编码改为 `UTF-8`，否则中文会显示成乱码

```
links:
  - name: "老苏的博客"
    icon: "fab fa-github"
    url: "https://laosu.cf"
    target: "_blank" # optional html a tag target attribute
  - name: "CSDN博客"
    icon: "fas fa-book"
    url: "https://blog.csdn.net/wbsu2004"


```

保存之后，只要刷新页面就可以了，不需要重启容器

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaEq4YZd9VULe1ptmeC99j2nCwz0lRmQQPLn4SWStJXNn5P1Ffe671U7gTic518MUzQtkAnI7IjqJEw/640?wx_fmt=png)

接下来改改 `services` 节

> `logo` 可以保存到 `assets/icons` 目录中，也可以直接用 `url`

```
services:
  - name: "Applications"
    icon: "fas fa-cloud"
    items:
      - name: "国内可用ChatGPT"
        logo: "https://www.sunboy.cf/favicon.svg"
        subtitle: "不用填API Key可直接用"
        tag: "chatgpt"
        keywords: "chatgpt"
        url: "https://www.sunboy.cf/"
        target: "_blank" # optional html a tag target attribute
      - name: "poe"
        logo: "https://poe.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2FchatGPTAvatar.04ed8443.png&w=48&q=75"
        subtitle: "需要科学上网"
        tag: "chatgpt"
        keywords: "chatgpt"
        url: "https://poe.com/ChatGPT"
        target: "_blank"
      - name: "DoGPT"
        logo: "https://pbs.twimg.com/profile_images/1604437547836248066/7RscimSD_400x400.png"
        subtitle: "自动化的GPT助手，点击试用，无需API。"
        tag: "chatgpt"
        keywords: "chatgpt"
        url: "https://www.dogpt.ai/"
        target: "_blank"


```

> 本文完成于 `4` 月，上面的网址，有些已不能使用；

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaEq4YZd9VULe1ptmeC99j2nOkbTHvZTM2CluIhuWJYmzaYgJv5c7K4L8oUTHlPfSPfSRGAneMOunA/640?wx_fmt=png)

如果要增加一个分组也很简单，只要增加一组

```
  - name: "Applications"
    icon: "fas fa-cloud"
    items:


```

还是举个栗子吧，这样看起来会更容易理解

```
services:
  - name: "人工智能"
    icon: "fas fa-cloud"
    items:
      - name: "国内可用ChatGPT"
        logo: "https://www.sunboy.cf/favicon.svg"
        subtitle: "不用填API Key可直接用"
        tag: "chatgpt"
        keywords: "chatgpt"
        url: "https://www.sunboy.cf/"
        target: "_blank" # optional html a tag target attribute
        
  - name: "老苏的博客"
    icon: "fa-solid fa-blog"
    items:
      - name: "老苏的博客"
        logo: "https://laosu.cf/images/laosu_wx.jpg"
        subtitle: "各种折腾"
        tag: "blog"
        keywords: "nas,docker"
        url: "https://laosu.cf"
        target: "_blank" 


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaEq4YZd9VULe1ptmeC99j2n9DhSs98X3TuM8UMdzj6zBSic6Aj9PO9AKGvjFyXWMTPJnHAUKInGibEA/640?wx_fmt=png)

中间的 `demo` 是消息，放在 `message` 节，不需要可以删掉，也可以设置需要提醒的内容

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaEq4YZd9VULe1ptmeC99j2nPWibPA6iclsQYIYdV52aiaeo8Lp06AiaLkMq0KdR4S0jh2abkEm2Go324w/640?wx_fmt=png)

是不是挺简单的？

参考文档
====

> bastienwirtz/homer: A very simple static homepage for your server.  
> 地址：https://github.com/bastienwirtz/homer

@所有人：写文不易，如果你都看到了这里，请点个`赞`和`在看`，分享给更多的朋友；为确保你能收到每一篇文章，请主页右上角设置星标。