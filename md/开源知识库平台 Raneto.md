> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/uspq9m4gnpAFPsbyyzAQrg)

> Raneto 是一个开源知识库平台，它使用静态 Markdown 文件来支持您的知识库。

**什么是 Raneto ？**

> `Raneto` 是一个开源知识库平台，它使用静态 `Markdown` 文件来支持您的知识库。

官方提供了 `doc & demo` 网站，即是帮助文档，也是个 `demo`，地址：https://docs.raneto.com

准备
==

项目使用`config.js` 做为设置文件，该文件的源码地址：https://raw.githubusercontent.com/ryanlelek/Raneto/master/config/config.js

*   记得用 `UTF-8` 编码格式保存，因为你可能会用中文的标题等；
    
*   每次修改之后，要重启容器才能生效；
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaHnzicdHfePwg1HID0Jiat5P4uibgkG9Gb9pv1h96CscqVpFx0vX8t9tCggxYc6MIOml0vGsqSQneshw/640?wx_fmt=png)

首先有几处需要修改的地方：

1.  修改站点标题
    

例如：将`site_title: 'Raneto Docs'`改成 `site_title: '老苏的测试站点'`

2.  修改用户名和密码
    

默认内置了 `2` 个用户，建议修改

```
  credentials: [
    {
      username: 'admin',
      password: 'password',
    },
    {
      username: 'admin2',
      password: 'password',
    },
  ],


```

3.  修改语言，使之支持中文
    

默认是英文

```
  locale: 'en',

  // Support search with extra languages
  searchExtraLanguages: ['ru'],


```

老苏改为了

```
  locale: 'zh',

  // Support search with extra languages
  searchExtraLanguages: ['zh'],


```

修改后的文件，老苏放在了 https://github.com/wbsu2003/synology/blob/main/Raneto/config.js，方便你需要的时候做对照

安装
==

在群晖上以 Docker 方式安装。

在注册表中搜索 `raneto` ，下翻找到 `raneto/raneto`，版本选择 `latest`。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaHnzicdHfePwg1HID0Jiat5P4WZsbTDzOdicxYMbGNed9icFTMLLcLmXCSZZUIPSAic2zw3giaBMzj5uHaQ/640?wx_fmt=png)

卷
-

在 `docker` 文件夹中，创建一个新文件夹 `raneto`，并在其中建两个子文件夹 `config` 和 `content`，将前面准备的 `config.js` 放入 `config` 目录

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaHnzicdHfePwg1HID0Jiat5P4e9aOQ3hdicpibBOeiaKrXrKcys97zoMcNcqC4CXniavccKMJ9f9Em7eU1Q/640?wx_fmt=png)

<table><thead><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: center;">文件夹</th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: center;" class="">装载路径</th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: center;">说明</th></tr></thead><tbody><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;"><code>docker/raneto/config</code></td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;"><code>/opt/raneto/config</code></td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;">存放设置文件</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;"><code>docker/raneto/content</code></td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;"><code>/opt/raneto/content</code></td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;">存放 <code>markdown</code>文件</td></tr></tbody></table>

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaHnzicdHfePwg1HID0Jiat5P4441F9YT9CwjYvvDjIlXahjvDNG3aPFv8koDcJ1ibphXdMyiczpicN3QVA/640?wx_fmt=png)

端口
--

本地端口不冲突就行，不确定的话可以用命令查一下

```
# 查看端口占用
netstat -tunlp | grep 端口号


```

<table><thead><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: center;">本地端口</th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: center;" class="">容器端口</th></tr></thead><tbody><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;"><code>3844</code></td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;"><code>3000</code></td></tr></tbody></table>

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaHnzicdHfePwg1HID0Jiat5P48axBOicJ2DzSGucohReKZVlDAY7A7xCibiazd1hYSyUSqbjTZiaY6Z7zBA/640?wx_fmt=png)

命令行安装
=====

如果你熟悉命令行，可能用 `docker cli` 更快捷

```
# 新建文件夹 raneto 和 子目录
mkdir -p /volume1/docker/raneto/{config,content/{pages,static}}

# 进入 raneto 目录
cd /volume1/docker/raneto

# 运行容器
docker run -d \
   --restart unless-stopped \
   --name raneto \
   -p 3844:3000 \
   -v $(pwd)/config:/opt/raneto/config \
   -v $(pwd)/content:/opt/raneto/content \
   raneto/raneto:latest


```

也可以用 `docker-compose` 安装，将下面的内容保存为 `docker-compose.yml` 文件

```
version: "2.1"

services:
  raneto:
    image: raneto/raneto:latest
    container_name: raneto
    restart: unless-stopped
    ports:
      - 3844:3000
    volumes:
      - ./config:/opt/raneto/config
      - ./content:/opt/raneto/content


```

然后执行下面的命令

```
# 新建文件夹 raneto 和 子目录
mkdir -p /volume1/docker/raneto/{config,content/{pages,static}}

# 进入 raneto 目录
cd /volume1/docker/raneto

# 将 docker-compose.yml 放入当前目录

# 一键启动
docker-compose up -d


```

运行
==

在浏览器中输入 `http://群晖IP:3844` 就能看到主界面

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaHnzicdHfePwg1HID0Jiat5P4MNibr0xuZjyQgibQFvDpFRyGQr99q6cJ1Pp1yIXS5TG4vN7g9r1bTKQA/640?wx_fmt=png)

点 `login`

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaHnzicdHfePwg1HID0Jiat5P4AqzIeF3XVsyusOsia672MQuj8jcjVF5iciao0siaf3OFnm8kcxdKUwe5rg/640?wx_fmt=png)

用户密码正确的话，会显示

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaHnzicdHfePwg1HID0Jiat5P4H42dXJB2HPiacgN5vJyc1OjhHwXWlP3mz0TOLXDzJzKphNOUkalFpqg/640?wx_fmt=png)

登录成功后的主界面其实还是一样的，只是在进入页面后，会有编辑权限

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaHnzicdHfePwg1HID0Jiat5P4V6dbG1u7xOM4PBK32EtqgC4W4Cn95XOhB6VDApyrIZoLDapoxmQaHQ/640?wx_fmt=png)

接下来在 `/content/pages` 中放入 `markdown` 文件或者目录即可

页面排序
----

每个页面可以包含有关该页面的可选元数据。

*   `Title` - 此变量将覆盖基于文件名的标题；
    
*   `Description` - 此变量将提供要搜索的描述；
    
*   `Sort` - 此变量将影响类别内页面的排序；
    
*   `ShowOnHome` - 可选。如果为 false，页面将不会在主页上列出。可以通过更改`config.show_on_home_default`调整默认行为；
    
*   `Modified` - 此变量将覆盖基于文件名的修改日期。
    

下面是一个示例的 `markdown` 文件，文件名为 `根目录2.md`

```
---
Title: 根目录第二篇
Sort: 1
---

根目录第二篇，但显示为第一位；


```

所以显示效果是下面👇这样的

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaHnzicdHfePwg1HID0Jiat5P4tI7NLNJ1VBKSgUzTLwY9N2mibyqzQZoFw0YwavCIXVtZ7mo0Ovf7trg/640?wx_fmt=png)

目录排序
----

目录显示出来实际上就是分类，你可以在当前文件夹中增加一个名称为 `meta` 的文件

*   `Title` - 此变量将覆盖基于文件夹名称的标题；
    
*   `Sort` - 此变量将影响分类（目录）的排序；
    
*   `ShowOnHome` - 可选。如果为 `false`，将不会显示在主页上。可以通过更改`config.show_on_home_default` 调整默认行为；
    
*   `Description` - 可选。该变量将提供在模板中使用的变量，例如在主页中，以增强和阐明类别的内容。
    

下面是一个示例的 `meta` 文件

```
Title: 安装使用
Sort: 1


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaHnzicdHfePwg1HID0Jiat5P4ryb2cD3yeGicHcicnbmVXwBPxpRNAYMQknQPkupa990PuRrFt4iabv3SQ/640?wx_fmt=png)

如果不需要变更分类名称，可以更简单的提供一个 `sort` 文件，文件中只要写排序就行

```
2


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaHnzicdHfePwg1HID0Jiat5P4ppr1fUaMcSlVat2AREI4bc2Gq5gC9dsXuj181Mu3CbWYibpKmcq3wYw/640?wx_fmt=png)

展示效果
----

重启容器后的显示效果

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaHnzicdHfePwg1HID0Jiat5P4FuZrYqgKQ6icGCicoosbsXQLo2CHCnPSfSWGvbaN3vUJaHRKRj6wuUpw/640?wx_fmt=png)

点开进入

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaHnzicdHfePwg1HID0Jiat5P4jZ2FEZhwJNcE8LOiauibDXEhqkXtic3xutMRCYY3XE3vicThFYLdyYzPMQ/640?wx_fmt=png)

登录之后有编辑权限

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cibq2rFYLAaHnzicdHfePwg1HID0Jiat5P4RRQkPjyzrh1uf3ug4XdIoGBC9VYqT8WZToPf7pcaDZsgrc6DmAx27g/640?wx_fmt=png)

注意事项
----

但凡文件中有中文的，一定要用 `UTF-8` 编码格式保存，否则页面上就会出现乱码，切记切记~

参考文档
====

> ryanlelek/Raneto: Markdown powered Knowledgebase Wiki for Node.js 地址：https://github.com/ryanlelek/Raneto
> 
> Raneto - Markdown Knowledgebase for Node.js 地址：https://raneto.com/