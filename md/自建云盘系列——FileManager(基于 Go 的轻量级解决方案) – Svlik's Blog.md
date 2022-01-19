> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.svlik.com](https://www.svlik.com/2436.html)

> 一. 介绍 Caddy，用 Go 写的一款相当优秀的 Web 服务器软件，它有不少很有特色的功能，国内目前来说用的不多，不过也逐渐有越来越多的人知道了，它有个特色的插件功能，其中一款插件是 FileManager，......

[![](https://cdn.svlik.com/wp-content/uploads/2020/09/1600521189-J4VEE4RQ%408ILUJL5G7Z1G.png)](https://cdn.svlik.com/wp-content/uploads/2020/09/1600521189-J4VEE4RQ@8ILUJL5G7Z1G.png)

一. 介绍
-----

Caddy，用 Go 写的一款相当优秀的 Web [服务器](https://www.svlik.com/tag/%e6%9c%8d%e5%8a%a1%e5%99%a8 "查看所有包含 服务器 的帖子")软件，它有不少很有特色的功能，国内目前来说用的不多，不过也逐渐有越来越多的人知道了，它有个特色的插件功能，其中一款插件是 FileManager，可以类似 H5ai 一样提供一个美化的 Index 目录列表，但是功能更多，不仅能下载，还能上传。但是，我一直不知道这玩意竟然还有个独立版本，最近 Loc 有人提到，我才发现这个确实不错。所以另外介绍一下，当然，之后也可能顺便介绍一下 FileRun，不过这个免费版我感觉功能限制的有点多，还是需要考虑下。

二. 安装
-----

简单到极致，看过我博客以前那些介绍用 Go 写的程序的文章的同学肯定对某个特点印象深刻，那就是安装贼鸡儿方便，特别是在官方提供现成的二进制文件的情况下，那就是下载 -- 解压 --done。FileManager 秉承了这个优点，官方甚至不需要你自己下载对应的二进制文件，全是一键脚本，自动判断环境一步到位。

```
#Linux下, 提供curl和wget两种
curl -fsSL https://henriquedias.com/filemanager/get.sh | bash
 
wget -qO- https://henriquedias.com/filemanager/get.sh | bash
 
#Windows下, 需要以管理员运行powershell然后安装
iwr -useb https://henriquedias.com/filemanager/get.ps1 | iex
```

刺激不刺激，当然，你在安装了 Caddy 的情况下只需要打开 http.filemanager 插件就行了

此外，现在 Docker 这么火当然也少不了它

```
docker run \
 -v /path/to/sites/root:/srv \
 -v /path/to/config.json:/config.json \
 -v /path/to/database.db:/database.db \
 -p 80:80 \
 hacdias/filemanager
```

这个采用的是默认配置，如下

```
{
 "port": 80,
 "address": "",
 "database": "/etc/database.db",
 "scope": "/srv",
 "allowCommands": true,
 "allowEdit": true,
 "allowNew": true,
 "commands": []
}
```

你也可以把配置写到命令里

```
docker run \
 -v /path/to/sites/root:/srv \
 -v /path/to/database.db:/database.db \
 -p 80:80 \
 hacdias/filemanager
 --port 80
 --database /database.db
 --scope /srv
 --other-flag other-value
```

当然，有一点需要注意，那就是 FileManager **不支持 SSL**，所以如果需要 SSL 或者说想用 HTTP/2 加速，请换 Caddy 配合插件

下面介绍一下，FileManager 的命令行参数以及配置文件

```
-a, --address 监听地址，默认为空，即为监听所有地址
-b, --baseurl 根地址，即为访问的入口地址
--prefixurl 前缀地址
-c, --config 指定配置文件
-d, --databaseis 数据库文件地址，默认为 “./filemanager.db”
-l, --log 显示错误日志记录器，可为 ‘stdout’, ‘stderr’ 或者是一个文件路径，默认为“stdout”
-p, --port 监听端口，默认为0，即为随机的可用端口
--staticgen 指定你是否需要启用静态网站生成器，可用 jekyll 以及 hugo
-v, --version 打印程序版本
--recaptcha-key ReCAPTCHA 站点 key ，配置后可在登陆处启用ReCAPTCHA
--recaptcha-secret ReCAPTCHA 站点 secret key ，配置后可在登陆处启用ReCAPTCHA
 
#下面的选项用于为新用户配置默认设置
--allow-commands 允许使用命令的默认值，默认为 true
--allow-edit 允许修改设置的默认值，默认为 true
--allow-new 允许新建设置的默认值，默认为 true
--allow-publish 允许发布功能，默认为 true
--view-mode 新用户默认查看视图，默认为 mosaic
--locale 新用户默认语言，可选 en, pt, jp, zh-cn, zh-tw (英语, 葡萄牙语, 日语, 简体中文, 繁体中文)
--commands 新用户可用命令，以空格分隔，默认为 “git svn hg”
--no-auth 禁用认证，启用这个选项会将权限设置为默认
-s, --scope 新用户的默认目录，默认为工作目录
```

下面举个栗子方便理解，在监听所有地址的 80 端口，数据库指定为 / etc/fm.db，新用户默认可访问 / data 目录

```
filemanager --port 80 --database /etc/fm.db --scope /data
```

FileManager 的配置文件支持多种写法，分别为 JSON，YAML 以及 TOML

IN JSON:

```
{
 "port": 80,
 "noAuth": false,
 "baseURL": "/admin",
 "address": "127.0.0.1",
 "reCaptchaKey": "",
 "reCaptchaSecret": "",
 "database": "/path/to/database.db",
 "log": "stdout",
 "plugin": "",
 "scope": "/path/to/my/files",
 "allowCommands": true,
 "allowEdit": true,
 "allowNew": true,
 "commands": [
 "git",
 "svn"
 ]
}
```

In YAML:

```
port: 80
baseURL: /admin
noAuth: false
address: 127.0.0.1
reCaptchaKey: ''
reCaptchaSecret: ''
database: "/path/to/database.db"
log: stdout
plugin: ''
scope: "/path/to/my/files"
allowCommands: true
allowEdit: true
allowNew: true
commands:
- git
- svn
```

In TOML:

```
port = 80
baseURL = /admin
address = 127.0.0.1
noAuth = false
reCaptchaKey = ''
reCaptchaSecret = ''
database = "/path/to/database.db"
log = stdout
plugin = ''
scope = "/path/to/my/files"
allowCommands = true
allowEdit = true
allowNew = true
commands = ["git", "svn"]
```

建议看哪种顺眼选哪种，没必要纠结太多，反正也不是天天改

对了，默认用户名密码均为 admin，其他看下图

这是登陆界面，所谓自建云盘嘛，虽然不一定有啥见不得人的东西，但是还是要上个锁的

[![](https://cdn.svlik.com/wp-content/uploads/2020/09/1600521327-NB7C1IAAI3S6MF01F.png)](https://cdn.svlik.com/wp-content/uploads/2020/09/1600521327-NB7C1IAAI3S6MF01F.png)

在用户设置中，可以配置 ACL 规则以便多人使用的情况下防止搞事，当然自定义 CSS 这种东西提供了更多的可能性

[![](https://cdn.svlik.com/wp-content/uploads/2020/09/1600521350-69P3ORM%40LKBKYSOZT26YOMC.png)](https://cdn.svlik.com/wp-content/uploads/2020/09/1600521350-69P3ORM@LKBKYSOZT26YOMC.png)

支持命令操作，是不是很刺激，这样就能玩出更多花样了

[![](https://cdn.svlik.com/wp-content/uploads/2020/09/1600521367-KYYW09GBWXF%40CFKQ%40U.png)](https://cdn.svlik.com/wp-content/uploads/2020/09/1600521367-KYYW09GBWXF@CFKQ@U.png)