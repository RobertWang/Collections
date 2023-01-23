> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [post.smzdm.com](https://post.smzdm.com/p/akxko548/?send_by=3547620770&invite_code=zdm4ptnapkinv&zdm_ss=iOS_3547620770_&from=singlemessage)

NAS 教程 篇一： 硬链接利器 hlink 使用教程（Docker）
===================================

2022-08-29 16:26:06 111 点赞 1251 收藏 56 评论

**未经允许，禁止转载**  

1. 什么需要硬链接
----------

对于 NAS 玩家来说，硬链接这个名词再熟悉不过了，硬链接简单来说就是可以将系统的一个文件链接到系统的另外一个地方，对一个文件设置硬链接后可以在指定的另外一个路径看到相同的一个文件。这个功能对于想要同时实现种子做种和影音库的 NAS 用户来说是一个很实用的功能。

接下来介绍一个近期发现的硬链接的神器：hlink。看一眼官网的对这个工具的介绍：

[![](https://qnam.smzdm.com/202208/29/630c6e8593b233623.jpg_e1080.jpg)](https://post.smzdm.com/p/akxko548/pic_2/)hlink 的特点

这个工具对于小白来说非常友好，同时具有 webui，可以使用 webui 来配置硬链接任务，设置定时执行硬链接任务，开发者将硬链接的使用难度降到了很低，几乎没有上手难度，官网的文档写的也十分清[楚明](https://pinpai.smzdm.com/287962/)

[](https://pinpai.smzdm.com/287962/)

了，看一眼官网文档的截图：

[![](https://am.zdmimg.com/202208/28/630b76dee786a2293.jpg_e1080.jpg)](https://post.smzdm.com/p/akxko548/pic_3/)hlink 官网文档

hlink 文档链接：[为什么是 hlink | hlink (likun.me)](https://hlink.likun.me/guide/why.html)

### 2. 如何使用 hlink

接下来介绍一下如何使用 hlink，这里介绍的是 docker 版本的 hlink，docker 的 hlink 已经配置好了环境，可以做到开箱即用，非常方便，所以建议大家也是用 docker 版本的 hlink。  

首先看到官网文档上介绍的如何使用 docker 安装 hlink：

[![](https://qnam.smzdm.com/202208/28/630b787f937c88563.jpg_e1080.jpg)](https://post.smzdm.com/p/akxko548/pic_4/)使用 docker 安装 hlink

可以看到作者在文档中写的非常明确如何使用 docker 安装 hlink，将其中的 $YOUR_HLINK_HOME_DIR、$YOUR_NAS_VOLUME_PATH、$DOCKER_VOLUME_PATH 三个变量根据自己的情况设置。接下来以我自己的情况为例，示例如何设置这三个变量。

我的[文件夹](https://www.smzdm.com/fenlei/wenjianjia/)结构如图：  

[![](https://qnam.smzdm.com/202208/28/630b7d2d0dd371275.jpg_e1080.jpg)](https://post.smzdm.com/p/akxko548/pic_5/)

可以看到我的文件夹结构分为两大部分，第一部分是下载文件夹 “未整理”，这个文件夹是 tr 或者其他下载器的文件夹，"未整理" 下面有三个文件夹，第一个 incomplete 文件夹，下载器未下载完成的文件夹放置在此，第二个和第三个为 Film 和 Episode 两个文件夹，用来分别放置下载完成的电影和剧集；第二部分是 Video 文件夹，这个文件夹就是 Jellyfin 或者 Emby 这类媒体[服务器](https://www.smzdm.com/fenlei/fuwuqi/)读取的影音文件夹。

接下设置三个变量 $YOUR_HLINK_HOME_DIR、$YOUR_NAS_VOLUME_PATH、$DOCKER_VOLUME_PATH。

1. $YOUR_HLINK_HOME_DIR 是环境变量，我这里设置为：/mnt/user/appdata/hlink。

2. $YOUR_NAS_VOLUME_PATH 变量设置，这个变量是将 NAS 的文件夹映射到 docker 中，看官网对于使用 docker 安装 hlink 的文档：[Hlink 硬链接 群晖 Docker WebUI 配置 | hlink (likun.me)](https://hlink.likun.me/other/HlinkDockerWebUI.html)

[![](https://qnam.smzdm.com/202208/28/630b7c2dc71711196.jpg_e1080.jpg)](https://post.smzdm.com/p/akxko548/pic_6/)官网对于文件夹结构的说明

可以看到官网建议直接映射母文件夹，看到文件夹结构图，这里我们设置为最上层的文件夹："downloads"，这里填写到命令的路径要为绝对路径，对我来说就是填写 "/mnt/user/downloads"，每个人根据自己系统来填写这个变量。

3. $DOCKER_VOLUME_PATH，这个变量为 $YOUR_NAS_VOLUME_PATH 这个变量映射到 docker 对用的路径，这里我设置为："/data"，这个路径可以自定义。

完整的命令如下：

[![](https://qnam.smzdm.com/202208/28/630b7faec561f5304.jpg_e1080.jpg)](https://post.smzdm.com/p/akxko548/pic_7/)

等待系统拉取镜像并设置完成后，在浏览器中输入 p:9090 打开 hlink 的 webui：  

[![](https://qnam.smzdm.com/202208/28/630b7fe81894c5837.jpg_e1080.jpg)](https://post.smzdm.com/p/akxko548/pic_8/)hlink 的 webui 界面

可以看到 hlink 的 webui 界面非常简洁，分为两个部分：任务列表和配置列表。  

首先设置配置文件，点击配置列表右侧的创建配置：  

[![](https://qnam.smzdm.com/202208/28/630b808eda7831496.jpg_e1080.jpg)](https://post.smzdm.com/p/akxko548/pic_9/)配置文件

可以看到配置文件如何填写，作者做了很详细的说明，每一个部分的功能以及设置方法都写得非常清楚。我们主要设置的就是 pathMapping 部分，可以看到我将下载文件夹下的 Film 和 Episode 文件夹映射到了 Jellyfin 文件夹下的 Film 和 Episode 文件夹，后面的 include 以及 exclude 还有其他部分保持默认即可，如果有需要参考官方文档进行修改。  

将配置文件保存后，可以看到配置列表下已经显示了我们刚才编辑的配置文件。接下来设置任务列表来实现自动定时执行硬链接任务：

[![](https://qnam.smzdm.com/202208/28/630b81a12b90377.jpg_e1080.jpg)](https://post.smzdm.com/p/akxko548/pic_10/)任务列表

可以看到任务列表的设置非常简单，只有三个部分需要设置，第一个是任务名称，这个根据自己的需求设置即可；第二个是任务类型，选择为 “硬链 (hlink)”；第三个是配置文件，选择为刚才我们编写的配置文件即可。设置好后可以在任务列表出现了一个任务：

[![](https://qnam.smzdm.com/202208/28/630b828daa63b243.jpg_e1080.jpg)](https://post.smzdm.com/p/akxko548/pic_11/)任务列表

点击定时设置选项（图[中红](https://pinpai.smzdm.com/286129/)

[](https://pinpai.smzdm.com/286129/)[关注](https://pinpai.smzdm.com/286129/) [](javascript:;)品牌 ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)   粉丝：

*   商品百科
    
*   好价
    
*   社区文章
    

箭头标识），看到 hlink 提供了两个选项，新手选项和 cron 定时任务。对于不懂 cron 规则的小伙伴可以使用新手规则：

[![](https://qnam.smzdm.com/202208/28/630b82debef8d9662.jpg_e1080.jpg)](https://post.smzdm.com/p/akxko548/pic_12/)定时任务选项

[![](https://am.zdmimg.com/202208/28/630b82eb303bc2733.jpg_e1080.jpg)](https://post.smzdm.com/p/akxko548/pic_13/)定时任务设置

这里可以设置每隔多少秒执行一次硬链接任务，根据自己的需求设置一个值就好，对于更灵活的定时设置，hlink 还提供了 cron 规则设置：  

[![](https://am.zdmimg.com/202208/28/630b8340b40fe3816.jpg_e1080.jpg)](https://post.smzdm.com/p/akxko548/pic_14/)cron 设置

根据自己的需求设置定时任务即可。  

到这里 hlink 的设置就完成了，将鼠标移到刚配置的任务：

[![](https://qnam.smzdm.com/202208/28/630b8385994108191.jpg_e1080.jpg)](https://post.smzdm.com/p/akxko548/pic_15/)硬链接任务

点击开始按钮，hlink 就开始工作了：  

[![](https://qnam.smzdm.com/202208/28/630b83b764e3d256.jpg_e1080.jpg)](https://post.smzdm.com/p/akxko548/pic_16/)hlink 执行结果

可以看到 hlink 执行硬链接任务成功。然后到设置的目标文件夹查看是否链接成功：

[![](https://qnam.smzdm.com/202208/28/630b8479e6ab57887.jpg_e1080.jpg)](https://post.smzdm.com/p/akxko548/pic_17/)硬链接结果

看到截图的左边为源文件，右侧为硬链接的目标文件夹，由于我们在配置文件中设置了 include 规则，hlink 只将包含在 include 内的文件类型映射到目标文件夹。(这[一点](https://pinpai.smzdm.com/281186/)

[](https://pinpai.smzdm.com/281186/)[关注](https://pinpai.smzdm.com/281186/) [](javascript:;)品牌 ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)   粉丝：

*   商品百科
    
*   好价
    
*   社区文章
    

对于需要刮削的用户来说非常友好，因为刮削只需要影音文件，那些图片文件夹或者 txt 文件并不需要，值得一提的是，如果源文件有字幕文件，应该在 include 规则内添加常见字幕文件的后缀)

到这里，已经完成了将下载好的 Film 文件夹和 Episode 文件夹映射到我们希望的目标文件夹，接下来就交给刮削器进行刮削。

PS：对于一个新工具的使用，用户首先做的应该是去看官方文档，而不是去百度或者 google 别人如何做的，官方文档是教你如何使用这个工具的，每一个参数怎么设置，有什么意义都写得非常清楚，所以部署一个工具出现了问题，应该去查阅官方文档，仔细阅读官方文档，解决不了再搜索别人是否出现相同的问题，进而找出自己的问题所在最后再找出解决办法。

**参考链接：**

[Home | hlink (likun.me)](https://hlink.likun.me/)

作者声明本文无利益相关，欢迎值友理性交流，和谐讨论～

![](https://res.smzdm.com/pc/pc_shequ/dist/img/the-end.png)