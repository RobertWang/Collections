> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/465657840) ![](https://pic3.zhimg.com/v2-af2e77c736a7621a2646a2cd4f1650ae_b.jpg)

大坑：
---

jlesage/nginx-proxy-manager 反代，johngong/calibre-web 部分页面返回不带端口号，其他反代的应用倒是正常

环境：
---

ikuai 软路由 docker 上部署 nginx-proxy-manager，黑群晖 docker 部署 calibre-web

![](https://pic3.zhimg.com/v2-89f06c72550eadcbe5212f3f8a994146_b.jpg)

解决：
---

nginx-proxy-manager 中 修改 /etc/nginx/conf.d/include/proxy.conf 文件内容，命令行界面输入 vi /etc/nginx/conf.d/include/proxy.conf 按 insert 键进入编辑模式，在 proxy_set_header Host $host；这一行中，$host 后添加端口号，我的例子：是 proxy_set_header Host $host:9999；(我使用的是 9999 映射到 nginx 的监听端口 4443，如果你直接使用路由的 4443 映射到 nginx 的 4443 监听端口，此行可用直接写成 proxy_set_header Host $host:$server_port;） 然后按 Esc 输入 wq 回车即可，最后重启下该 docker 实例

参考文章：[Nginx 反向代理端口域名无法访问问题解决_weixin_34233856 的博客 - CSDN 博客](https://link.zhihu.com/?target=https%3A//blog.csdn.net/weixin_34233856/article/details/92246160)

后续坑：
----

不用 nginx-proxy-manager，用群辉的反代功能，calibre-web 就没这个鬼问题，晚点群辉申请 let's encrypt 的泛域名证书方便的话，可以直接用群辉反代了，也是图形化管理，看着也挺方便。用 nginx-proxy-manager 是看中了图形化界面配置和可自动续期证书，持续观察一下。

备注：
---

nginx-proxy-manager 也测试了 jc21 的，一样的错误。没有用 jc21 的原因是他的镜像包比较大