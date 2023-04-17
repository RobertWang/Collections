> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [post.smzdm.com](https://post.smzdm.com/p/a4p8625w/?send_by=3547620770&invite_code=zdm4ptnapkinv&zdm_ss=iOS_3547620770_&from=singlemessage)

NAS 原来这么有用 篇九十一：国人大佬开发的中文 docker 管理界面—dockerUI
==============================================

2023-04-10 15:20:11 83 点赞 790 收藏 51 评论

[![](https://am.zdmimg.com/202304/09/64325e2347ad24926.jpg_e1080.jpg)](https://post.smzdm.com/p/a4p8625w/pic_2/)封面

前言
==

说到 Docker GUI 管理界面，可以说业内最出名的便是 Portainer 了，其次 Kitematic 以及没有中文界面，甚至没有图形 UI 的 LazyDocker。而今天介绍的这款是国内大佬开发的 UI，兼备了良好的 UI 界面的同时，也同时具备 Docker [主机](https://www.smzdm.com/ju/sp3rz02/)管理、集群管理以及 docker 的任务编排。

部署
==

docker.UI 的部署很简单，你可以从 docker 下载后再用 ssh 命令启动，也可以直接 docker run 拉取并启动。并且 docker.UI 的容器大小也只有 60 来 M。

[![](https://qnam.smzdm.com/202304/09/64325e2cefe899799.png_e1080.jpg)](https://post.smzdm.com/p/a4p8625w/pic_3/)容器

打开 [NAS](https://www.smzdm.com/ju/sp3qr1k/) 的 ssh 链接，随后依次输入以下命令：

*   sudo -i【获取管理员命令】
    
*   docker run --restart always --name docker.ui -d -v /var/run/docker.sock:/var/run/docker.sock -p 8989:8999 joinsunsoft/docker.ui【拉取并后台启动容器，其中容器端口 8999 不可更改，本地端口 8989 随意更改，不冲突即可】
    

至于为什么只能用命令行启动，那是因为[群晖](https://pinpai.smzdm.com/2315/)

以及大部分 NAS 是不支持映射 docker 的守护进程，也就是

docker.sock

文件，所以我们只能在 ssh 链接 NAS 后再使用管理员命令拉取镜像映射 docker.sock 文件，这样才能使 docker.UI 拥有 docker 权限。

[![](https://qnam.smzdm.com/202304/09/64325e361aa6d5658.png_e1080.jpg)](https://post.smzdm.com/p/a4p8625w/pic_4/)docker.UI

体验
==

浏览器输入 http://nasip + 本地端口即可访问了，初始用户名 / 密码 ginghan/123456。

[![](https://qnam.smzdm.com/202304/09/64325e428a7a71566.png_e1080.jpg)](https://post.smzdm.com/p/a4p8625w/pic_5/)主界面

整个界面很清爽，主界面还会展示资源利用率，右边则是系统信息，最下面是更新日志，可以看到去年八月后就断更了，也不知道作者还会更新不，不过目前看起来完成度已经很高了。

[![](https://qnam.smzdm.com/202304/09/64325e4c86df13324.png_e1080.jpg)](https://post.smzdm.com/p/a4p8625w/pic_6/)镜像管理

进入镜像管理，你可以看到目前 nas 上下载的所有镜像，值得一提的是它还会展示中间镜像以及一些下载中断或者下载出错后残留的镜像，这些是不会在群晖中显示的，也就是所谓的垃圾文件，可以直接进行删除即可。

[![](https://qnam.smzdm.com/202304/09/64325e5910ad81796.png_e1080.jpg)](https://post.smzdm.com/p/a4p8625w/pic_7/)容器管理

除此之外你还能看到下面的容器管理、[存储](https://www.smzdm.com/ju/s2qrjnp/)卷以及集群和任务编排等信息，这些就等大家自行研究了。

总结
==

总的来看还算不错，好看的 UI 加上便捷的操作很适合作为长期使用的 docker 管理，不过我删除或者加载镜像时总觉得有点慢，不知道是不是优化的问题，也不清楚作者还会继续更新不，各位小伙伴觉得还不错可以部署尝试一下。

以上便是本期的全部内容了，如果你觉得还算有趣或者对你有所帮助，不妨点赞收藏，最后也希望能得到你的关注，咱们下期见！  

作者声明本文无利益相关，欢迎值友理性交流，和谐讨论～