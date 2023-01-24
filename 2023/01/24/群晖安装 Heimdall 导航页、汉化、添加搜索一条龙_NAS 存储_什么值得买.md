> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [post.smzdm.com](https://post.smzdm.com/p/arqe5q3x/?send_by=3547620770&invite_code=zdm4ptnapkinv&zdm_ss=iOS_3547620770_&from=singlemessage)

群晖安装 Heimdall 导航页、汉化、添加搜索一条龙
============================

2022-08-24 18:31:23 45 点赞 598 收藏 20 评论

写在前面
----

随着[群晖](https://pinpai.smzdm.com/2315/)

[](https://pinpai.smzdm.com/2315/)

中使用的功能越不来越多，功能入口记忆就成了一大难题，所以就有了使用导航页的想法。最开始使用的导航页是使用别人的模板修改之后使用的，但网页是静态的，每次修改都要去修改网页后台，比较麻烦。后来又使用了 wordpress 建站，使用一段时间后发现相当麻烦，门槛也比较高。最后发现 heimdall，在 docker 中运行，使用比较方便，修改和用户管理都很方便。折腾了一段时间，分享下我的过程供大家参考。

安装
--

### 1、下载镜像

打开 docker，在册表中搜索 heimdall，然后选择如下图中第一个下载最新版本。

[![](https://am.zdmimg.com/202208/24/63059b25ae3067841.png_e1080.jpg)](https://post.smzdm.com/p/arqe5q3x/pic_2/)

### 2、配置参数

下载完成后，在映像中点击启动，然后可以参考以下设置。

[![](https://am.zdmimg.com/202208/24/63059c6f56f7e3726.png_e1080.jpg)](https://post.smzdm.com/p/arqe5q3x/pic_3/)

[![](https://am.zdmimg.com/202208/24/63059c7e055287020.png_e1080.jpg)](https://post.smzdm.com/p/arqe5q3x/pic_4/)

[![](https://qnam.smzdm.com/202208/24/63059c8d655224860.png_e1080.jpg)](https://post.smzdm.com/p/arqe5q3x/pic_5/)

为了方便配置，[文件夹](https://www.smzdm.com/fenlei/wenjianjia/)映射如下：  

[![](https://qnam.smzdm.com/202208/24/63059ce467f705898.png_e1080.jpg)](https://post.smzdm.com/p/arqe5q3x/pic_6/)

最后启动。然后就可以通过群晖 IP + 端口号的方式访问。

汉化
--

[推荐使用这位朋友的方法。](https://post.smzdm.com/p/a99v2nqo/)

 [![](https://qna.smzdm.com/202002/29/5e59d265571d21040.jpg_a200.jpg) 文章 打造你自己的专属浏览器首页：heimdall 进阶玩法，汉化、增加百度搜索项以增应用 ![](http://avatarimg.smzdm.com/default/2944043377/62cb7850ad3419404-small.jpg) 灰色会 20-02-29  97](https://post.smzdm.com/p/a99v2nqo/) 

本地化搜索
-----

搜索按上面的方法弄[好之](https://pinpai.smzdm.com/32463/)

[](https://pinpai.smzdm.com/32463/)[关注](https://pinpai.smzdm.com/32463/) [](javascript:;)品牌 ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)   粉丝：

*   商品百科
    
*   好价
    
*   社区文章
    

后，其实就可以使用了。但有两个问题：里面有无用的搜索引擎、自己添加的百度不显示为百度。

1、删除无用的搜索引擎。打开 Search.php 文件，删除以下内容，这里以删除 google 为例。

[![](https://qnam.smzdm.com/202208/24/63059f0a724bc3133.png_e1080.jpg)](https://post.smzdm.com/p/arqe5q3x/pic_7/)

2、自己添加的搜索引擎正常显示名称。首先按上面的方法添加搜索引擎，名称可以按自己需要更改，但最好不要用中文，可以用英文和拼音。  

[![](https://qnam.smzdm.com/202208/24/63059fb279c355088.png_e1080.jpg)](https://post.smzdm.com/p/arqe5q3x/pic_8/)

修改完这一步后，主面显示的名称依旧不是我们想要的，还需要到语[言](https://pinpai.smzdm.com/61543/)

[](https://pinpai.smzdm.com/61543/)[关注](https://pinpai.smzdm.com/61543/) [](javascript:;)品牌 ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)   粉丝：

*   商品百科
    
*   好价
    
*   社区文章
    

包中添加相对应的代码，才能正常显示我们想要的结果 。打开我们使用的汉化语言包添加如下语句：

[![](https://qnam.smzdm.com/202208/24/6305a0319271e5141.png_e1080.jpg)](https://post.smzdm.com/p/arqe5q3x/pic_9/)

完成以上步骤就可以[得到](https://pinpai.smzdm.com/271508/)

[](https://pinpai.smzdm.com/271508/)[关注](https://pinpai.smzdm.com/271508/) [](javascript:;)品牌 ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)   粉丝：

*   商品百科
    
*   好价
    
*   社区文章
    

如下的效果了。

[![](https://am.zdmimg.com/202208/24/6305a0b239b851079.png_e1080.jpg)](https://post.smzdm.com/p/arqe5q3x/pic_10/)最终效果

作者声明本文无利益相关，欢迎值友理性交流，和谐讨论～

![](https://res.smzdm.com/pc/pc_shequ/dist/img/the-end.png)