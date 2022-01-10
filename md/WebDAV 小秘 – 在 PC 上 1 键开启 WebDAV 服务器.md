> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.appinn.com](https://www.appinn.com/webdav-xiaomi/)

**WebDAV 小秘** 是一款简洁、易用的 Windows 工具，它能让你在 PC 上 1 键开启 WebDAV 服务器，用于传输文件或者为支持 WebDAV 协议的软件（如 Jolpin 笔记、Enpass 等）提供存储服务。其底层基于的 [webdav-cli](https://github.com/svtslv/webdav-cli) 项目也支持 Mac 与 Linux 系统。@[Appinn](https://www.appinn.com/webdav-xiaomi/)

![](https://img3.appinn.net/images/202105/webdav-xiaomi.jpg!o)

在讨论组的一个帖子：[我开始使用 Joplin 了，再顺便求问一下 webdav 的服务端](https://meta.appinn.net/t/topic/23263) 中，@[live9999](https://meta.appinn.net/u/live9999) 同学推荐了 WebDAV 小秘 这款小工具。

WebDAV 是什么
----------

**基于 Web 的分布式编写和版本控制**（**WebDAV**）是超文本传输协议（HTTP）的扩展，有利于用户间协同编辑和管理存储在万维网服务器文档。WebDAV 由互联网工程任务组的工作组在 RFC 4918 中定义。

WebDAV 协议为用户在服务器上创建，更改和移动文档提供了一个框架。WebDAV 协议最重要的功能包括维护作者或修改日期的属性、名字空间管理、集合和覆盖保护。维护属性包括创建、删除和查询文件信息等。**名字空间管理**处理在服务器名称空间内复制和移动网页的能力。**集合**（Collections）处理各种资源的创建、删除和列举。**覆盖保护**处理与锁定文件相关的方面。许多现代操作系统为 WebDAV 提供了内置的客户端支持。

大致可以理解为使用 WebDAV 协议创建了一个网盘空间，支持 WebDAV 协议的软件就可以连接到这个空间上，用来当作云存储服务。

WebDAV 小秘
---------

WebDAV 小秘 的整个界面就是这样了，非常干净清爽，只需要适当修改，就能完成设置了。

![](https://img3.appinn.net/images/202105/webdavj2jsmsxjci.jpg!o)

其中比较重要的，就是共享目录，以及主机地址了。至于 SSL 证书，更推荐在公网使用，局域网内，IP 即可。

点击**启动**按钮，就能在浏览器中测试了，比如上图中，只需要在浏览器中输入地址： http://172.16.0.189:1900 就能弹出用户名、密码，输入后，即可看到 C 盘中的 Tools 文件夹内容，并且可下载。

**WebDAV 小秘** 所允许的访问权限十分详细，包括允许创建、删除、移动、重命名、追加、写入、读取、属性等等，自己用的话，就选允许所有操作即可。

然后，就能在支持 WebDAV 的客户端中设置了，其中 **WebDav URL** 就是你在浏览器中打开的地址，包括后面的端口号。

**WebDAV 小秘**下载
---------------

你可以[在这里](https://lightzhan.xyz/index.php/webdavhelper/)访问 WebDAV 小秘 官网，以及[在这里下载](https://lightzhan.lanzoui.com/b015wjsri)到由 WebDAV 小秘 提供的网盘地址。

#### 几个问题

对于 Mac 与 Linux 用户，可以使用 [webdav-cli](https://github.com/svtslv/webdav-cli) 项目，命令行工具。

至于在外网访问你在家中的 WebDAV 服务器，那需要[内网穿透](https://www.appinn.com/tag/%E5%86%85%E7%BD%91%E7%A9%BF%E9%80%8F/)，这就是另外一个问题了。那么最后问题来了，你都知道哪些软件可以使用 WebDAV 吗？