> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/465925053)

背景：
---

家里之前有个 QNAP 的 NAS，2022 年过年时候 QNAP 发布了漏洞声明，建议纯域名访问 NAS 规避漏洞，这算是个契机，外加自己爱折腾，就将之前采用的、在路由器上使用不同端口映射各类服务，改为 nginx 反代——统一访问端口，但通过不同子域名访问。这避免了 IP 直接访问 NAS 的问题。做这个反代，还有个想法是把 bitwarden 在 docker 中也做起来，之前测试部署这个好像也需要反代。

搭建前提：
-----

1.  动态公网 IP：电信宽带可以申请 ip v4 的动态公网 IP，打 10000 号申请，移动和联通是不行的；
2.  DDNS（动态域名解析）：我从阿里万网上买的 tech 后缀的便宜域名，199 买了 10 年的，配合软路由上 aliDDNS 解析，或者部署 aliddns 的 docker 都可以实现动态域名解析。
3.  docker 环境：群辉 NAS，QNAP NAS，或者 linux 系统，ikuai/openwrt 软路由系统等，都可以安装 docker 环境。推荐 NAS 或者软路由上安装 docker，图形化界面方便部署和管理镜像；

安装步骤：
-----

*   docker 中，下载镜像：jlesage/nginx-proxy-manager
*   创建容器时，参考说明 [jlesage/nginx-proxy-manager - Docker Image | Docker Hub](https://link.zhihu.com/?target=https%3A//hub.docker.com/r/jlesage/nginx-proxy-manager) ，映射下文件夹就好了。一般使用，这个文件夹映射都可以不做。

![](https://pic4.zhimg.com/v2-1370ba7d9ca8db46d3883529d1260813_b.jpg)

*   部署完毕后，访问管理端口默认部署的话 8181 是管理端口，路由器上将外网映射到这个 docker 的 4443 端口，外网均采用 https 前缀访问

![](https://pic2.zhimg.com/v2-eb93b38514bc98114a6de1e9cb1ffe91_b.jpg)![](https://pic4.zhimg.com/v2-5cba38392b0bb043d686ba92adfc3d0f_b.jpg)

*   访问 nginx 管理地址进行反代配置，访问地址是 [http://dockerIP:8181](https://link.zhihu.com/?target=http%3A//dockerIP%3A8181)。默认账号密码是 admin@[example.com](https://link.zhihu.com/?target=http%3A//example.com/) 密码是 changeme 登陆进去后修改邮箱和密码，后续用新的邮箱和密码登陆即可

![](https://pic3.zhimg.com/v2-1b6239326b190478a71c4a298ddb51a6_b.jpg)

*   进去后，先通过 Let's Encrypt 申请泛域名的证书，假设我的域名是 [http://abc.com](https://link.zhihu.com/?target=http%3A//abc.com)，那申请证书就使用 *.[http://abc.com](https://link.zhihu.com/?target=http%3A//abc.com) 来申请，这样后续任何子域名 https 访问都是没问题的了，例如 [book.abc.com](https://link.zhihu.com/?target=http%3A//book.abc.com/)

![](https://pic1.zhimg.com/v2-2c98ddd54ce95e5a45d079ba7feda504_b.jpg)![](https://pic2.zhimg.com/v2-65f2687f0a2ed3fec88552002b72da25_b.jpg)

*   这里有个小 bug，申请界面会一直转圈转几分钟到超时，实际已经申请成功了，F5 刷新下界面，就可以考到类似上图 “证书申请 1” 中的泛域名证书了，这个证书给后面所有的子域名配置。docker 作者的文档提到证书在到期一个月前会自动续期，到时继续观察下这个流程是否丝滑。。
*   基础准备完后，就开始配置子域名的解析，在阿里云（或你的域名商）域名管理界面，将所有子域名都配置 CNAME 到根域名，例如 [http://book.abc.com](https://link.zhihu.com/?target=http%3A//book.abc.com) 就 CNAME 到 [http://abc.com](https://link.zhihu.com/?target=http%3A//abc.com)
*   然后在 nginx 界面配置所需的子域名反代，就 OK 了

![](https://pic1.zhimg.com/v2-8e26da696a7c60c61dc6eced6c1e8600_b.jpg)

参考和其他说明：
--------

1.  除了阿里云买域名实现 DDNS，腾讯云 DNSPod 买域名也可以实现，只是我习惯了阿里云的 DDNS，腾讯云详见 [使用 docker 搭建 nginx proxy manager 实现反向代理和 SSL 证书申请 - 哔哩哔哩 (bilibili.com)](https://link.zhihu.com/?target=https%3A//www.bilibili.com/read/cv14665485)
2.  B 站有 Up 主介绍 nginx-proxy=manager 这个应用，有兴趣可以听听前面的功能介绍，后面的要买 VPS 什么的没必要，是另外个套路了，另外不建议在阿里云或腾讯云之外买域名，没有现成的插件或者 docker，DDNS 估计都会很头疼。[【Docker 系列】一个反向代理神器——Nginx Proxy Manager_哔哩哔哩_bilibili](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV1Gg411w7kQ%3Fp%3D1)
3.  这个应用图形化管理很方便，有个小坑建议填填，无需 ssh 工具，docker 命令行界面进去修改 proxy.conf 文件即可，详见：[nginx-proxy-manager 填坑](https://zhuanlan.zhihu.com/p/465657840)