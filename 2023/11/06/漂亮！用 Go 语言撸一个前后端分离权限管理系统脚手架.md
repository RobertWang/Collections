> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIwNDgzMzI3Mg==&mid=2247493004&idx=1&sn=beb8e374472320c46ba06992e1f7344c&chksm=97388d1ba04f040da0cf3d648273ab8a6d408ebe1ae1bab460d17c1140a7ad682fe64fd2eb98&scene=178&cur_album_id=1571213952619954180#rd)

你好，我是小金。熟悉小金的小伙伴都清楚，最近小金在学习 Go 语言，以后有考虑从前端转到 Go 语言或者当一个全栈开发。

今天要给大家推荐的项目叫做 **go-admin**，这是我在周末学习的一个项目，感觉挺不错的，非常适合拿来学习，也可以用来作为搭建自己项目的脚手架。

目前，go-admin 这个项目在 Github 上收获了 7.6k star，算得上是小有名气的 Go 项目了。

![](https://mmbiz.qpic.cn/mmbiz_png/BcyAypujBVYrhR8cC2k4Vnu5v1yiao4SZc1rt7Beh4C5mKYymA5Kz2RxV3fhticDOBianibNdkKKSuM0uHlhziaslpQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/BcyAypujBVYrhR8cC2k4Vnu5v1yiao4SZqibNNyQCGjTvaNEChEx23vGnUnoqVYfapwd8KBBcHDVfYqnhzRaxJJg/640?wx_fmt=png)

go-admin 是一个基于 Gin + Vue + Element UI OR Arco Design OR Ant Design 的前后端分离权限管理系统，提供了很多开箱即用的功能比如多租户、用户管理、部门管理、岗位管理、菜单管理、代码生成、字典管理。

![](https://mmbiz.qpic.cn/mmbiz_png/BcyAypujBVYrhR8cC2k4Vnu5v1yiao4SZOtBDggxibqsIrl3BFR6lUhot45KQOV3TcutDrbK0OqzYns8UNHJKPwg/640?wx_fmt=png)

*   项目地址：https://github.com/go-admin-team/go-admin。
    
*   官方文档：https://www.go-admin.pro/ 。
    

效果概览
----

Element UI vue demo：https://vue2.go-admin.dev

> 账号 / 密码：admin / 123456

Arco Design vue3 demo：https://vue3.go-admin.dev

> 账号 / 密码：admin / 123456

antd demo：https://antd.go-admin.pro

> 账号 / 密码：admin / 123456

这里以 Element UI vue demo 为例。

### 后台登录

![](https://mmbiz.qpic.cn/mmbiz_png/BcyAypujBVYrhR8cC2k4Vnu5v1yiao4SZp0FIsYAXBxTjHesiczeNRfeaKYgagvwc2Crt3v7u0uqFa7zloW1q6Ug/640?wx_fmt=png)

### 后台首页

![](https://mmbiz.qpic.cn/mmbiz_png/BcyAypujBVYrhR8cC2k4Vnu5v1yiao4SZDBd0n8NCgSA4yic7SEJatzOUpd7Jn6sCJKcvR4kCXvy2Y8eoib6DprOg/640?wx_fmt=png)

### 用户管理

![](https://mmbiz.qpic.cn/mmbiz_png/BcyAypujBVYrhR8cC2k4Vnu5v1yiao4SZDNRVpcIHAbknUjR9Mo5NHNMiaNwtfGMuq34DL5bzEjb6ErHbyzXFibOg/640?wx_fmt=png)

### 部门管理

![](https://mmbiz.qpic.cn/mmbiz_png/BcyAypujBVYrhR8cC2k4Vnu5v1yiao4SZkBTYibe9e3KtxwTMT3eG3FjCG075fT51tERgpbOVgkEOl8B5cacHvmg/640?wx_fmt=png)

### 字典管理

![](https://mmbiz.qpic.cn/mmbiz_png/BcyAypujBVYrhR8cC2k4Vnu5v1yiao4SZviaMZ2jj8e82XloF5zW91nrwEE7yk3lOBiabBia4gRiahZF0BOic2BTj8xA/640?wx_fmt=png)

本地开发
----

**环境要求：**

*   go 1.18
    
*   node 版本: v14.16.0
    
*   npm 版本: 6.14.11
    

**代码获取：**

> 重点注意：两个项目必须放在同一文件夹下；

```
# 获取后端代码
git clone https://github.com/go-admin-team/go-admin.git


# 获取前端代码
git clone https://github.com/go-admin-team/go-admin-ui.git


```

**启动后端项目：**

```
# 进入 go-admin 后端项目
cd ./go-admin


# 更新整理依赖
go mod tidy


# 编译项目
go build


# 修改配置
# 文件路径  go-admin/config/settings.yml
vi ./config/setting.yml


# 1. 配置文件中修改数据库信息
# 注意: settings.database 下对应的配置数据
# 2. 确认log路径


```

**初始化数据库：**

```
# 首次配置需要初始化数据库资源信息
# macOS or linux 下使用
$ ./go-admin migrate -c config/settings.dev.yml


# ⚠️注意:windows 下使用
$ go-admin.exe migrate -c config/settings.dev.yml




# 启动项目，也可以用IDE进行调试
# macOS or linux 下使用
$ ./go-admin server -c config/settings.yml




# ⚠️注意:windows 下使用
$ go-admin.exe server -c config/settings.yml


```

**启动前端项目：**

```
# 进入 go-admin 前端项目
cd go-admin-ui
# 安装依赖
npm install


# 建议不要直接使用 cnpm 安装依赖，会有各种诡异的 bug。可以通过如下操作解决 npm 下载速度慢的问题
npm install --registry=https://registry.npmmirror.com


# 启动服务
npm run dev


```

总结
--

go-admin 是一个基于 Gin + Vue + Element UI OR Arco Design OR Ant Design 的前后端分离权限管理系统，提供了很多开箱即用的功能比如多租户、用户管理、部门管理、岗位管理、菜单管理、代码生成、字典管理。

go-admin 非常适合拿来学习，也可以用来作为搭建自己项目的脚手架。

推荐
--

用心发掘优质开源项目，欢迎关注，欢迎点赞分享！

历史优质开源项目推荐地址：[Github 掘金计划](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzIwNDgzMzI3Mg==&action=getalbum&album_id=1571213952619954180#wechat_redirect)。

*   [计算机基础](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1635325633234780161&__biz=MzIwNDgzMzI3Mg==#wechat_redirect)：精选计算机基础（操作系统、计算机网络、算法、数据结构）相关的开源项目。
    
*   [神器工具](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzIwNDgzMzI3Mg==&action=getalbum&album_id=1692140336665378820#wechat_redirect) : 一些好用的插件、软件、网站。
    
*   [程序人生](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzIwNDgzMzI3Mg==&action=getalbum&album_id=2084343476975878144#wechat_redirect)：编程经历、英语学习、延寿指南。
    
*   [项目实战](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1632590550748938241&__biz=MzIwNDgzMzI3Mg==#wechat_redirect) ：精选实战类型的开源项目。