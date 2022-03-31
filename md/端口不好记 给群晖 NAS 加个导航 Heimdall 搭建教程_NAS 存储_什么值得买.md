> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [post.m.smzdm.com](https://post.m.smzdm.com/p/aoxnw8vm/?send_by=3547620770&invite_code=zdm4ptnapkinv&zdm_ss=iOS_3547620770_&from=singlemessage)

> Heimdall 搭建 1、登陆群晖，打开 dokcer 注册表搜索 heimdall，选择 linuxserver/heimdall 下载。2、稍等片刻，在映像

Heimdall 搭建
-----------

1、登陆[群晖](https://pinpai.m.smzdm.com/2315/)，打开 dokcer 注册表搜索 heimdall，选择 linuxserver/heimdall 下载。

![](https://qnam.smzdm.com/202203/11/622ae4695e1f52089.png_e600.jpg)选择最新版本（lasetr）下载即可

2、稍等片刻，在映像中找到下载好的镜像，点击启动，勾选高级执行容器，然后点击设置

![](https://qnam.smzdm.com/202203/11/622ae49f8b7cd2752.png_e600.jpg)

3.、高级设置：勾选自动重启。

![](https://qnam.smzdm.com/202203/11/622ae63e0cff27893.png_e600.jpg)

4.、存储空间：建立如下映射关系，< **文件 / [文件夹](https://m.smzdm.com/fenlei/wenjianjia/) > 为自建目录，请自行在 docker 目录下创建，**装载路填写 **/config**

![](https://qnam.smzdm.com/202203/11/622ae65ea66472866.png_e600.jpg)

5、网络：保持默认无需更改。

6、端口设置：本地端口请填写未使用的端口，容器端口按照如下图填写。

![](https://qnam.smzdm.com/202203/11/622b09d8b8b692682.png_e600.jpg)

7、链接：保持默认，无需更改。

8、环境：增加两条 **PUID = 1000 PGID =1000,** 如下图所示。

![](https://qnam.smzdm.com/202203/11/622b09f182fec8816.png_e600.jpg)

9、以上步骤完成以后点击应用→下一步，然后在[摘要](https://pinpai.m.smzdm.com/234669/)中再次检查配置是否正确，如无问题请点击完成。

![](https://qnam.smzdm.com/202203/11/622b0d1c7f8117109.png_e600.jpg)

10、返回到 docker 首页，点击容器，可以看到服务已经正确运行。

![](https://qnam.smzdm.com/202203/11/622b0d46d0be32526.png_e600.jpg)

11、因为第一次运行，需要加载文件，请等待 1-2 分钟之后，在浏览器地址栏中输入群晖 NASip 地址：端口号进入 heimdall 页面, 如果无法进入请重[启一](https://pinpai.m.smzdm.com/274317/)下容器（后面的滑块关闭在打开一下）。

![](https://qnam.smzdm.com/202203/11/622b0e4e547e52748.png_e600.jpg)

12、搭建还是很简单，接下来是汉化过程。

Heimdall 汉化
-----------

本教程采用的是 heimdall 原版包，默认无中文，下面是汉化步骤。  

![](https://qnam.smzdm.com/202203/11/622b6509ab5c32314.png_e600.jpg)原版镜像包无中文

1、汉化过程需要使用到的工具有 FinalShell SSH 工具、WinScp 文件传输工具（这两款工具为教程演示工具，也可以使用其他具有相同功能的软件代替）。

*   FinalShell SSH 工具 下载地址：不让放链接百度搜索第一个就是
    
*   WinScp 文件传输工具 下载地址：不让放链接百度搜索第一个就是  
    

2、打开群晖控制面板→终端机和 SNMP→勾选启 SSH 功能。

![](https://qnam.smzdm.com/202203/11/622b1063aee299300.png_e600.jpg)

3、打开 FinalShell SSH 工具，登陆群晖，输入 sudo -i 在输入密码（密码不显示）切换到 root 模式。

![](https://qnam.smzdm.com/202203/11/622b1155bbd206546.png_e600.jpg)

4、输入以下命令查找 heidmall 的路径（此路径非安装过程中映射的文件夹）。

find / -name heimdall

![](https://qnam.smzdm.com/202203/11/622b12960beb55433.png_e600.jpg)

5、查找 heimdall 的安装路径，可以看到我安装的路径为：（找我加粗标记的那一行，前面的可能不相同）

/volume1/@docker/btrfs/subvolumes/11c1a1168afa4ff3c692048eb4cfe67739def550eb47691aa9818586a83d8383**/var/www/localhost/heimdall**

6、复制好这串路径，保存备用，然后打开 WinScp，用 root 用户登陆，如果没有 root 权限的请看我这一篇获取 root 权限的教程，群晖 DSM7.0 获取 ROOT 用户登陆权限 [https://zwbcc.cn:54321/archives/dsm70root](https://zwbcc.cn:54321/archives/dsm70root)

![](https://qnam.smzdm.com/202203/11/622b14d1004592563.png_e600.jpg)点击新建站点输入信息登陆，需要使用 root 用户否则进行修改文件的时候会提示权限不足

![](https://qnam.smzdm.com/202203/11/622b14de8a1465039.png_e600.jpg)登陆成功

7、点击 打开目录 ，将步骤五复制的路径粘贴到打开目录中，然后点击确定，﻿  

![](https://qnam.smzdm.com/202203/11/622b158774830198.png_e600.jpg)

8、按照第 7 步骤操作完成以后即进入了 heimdall 的安装目录。

![](https://qnam.smzdm.com/202203/11/622b617bc16339853.png_e600.jpg)

9、依次点击 resources→lang，lang 里面的便是各种语言目录，咱们只需要修改其中的一种语言为中文，然后再 heimdall 设置中将这种语言改成默认语言既可以，这里咱们就修改 en 也就英语。打开 en 文件夹。（汉化原理：heimdall 采用了专门的语言包，找到这个语言包，加上自己的翻译，然后替换即可。）

![](https://qnam.smzdm.com/202203/11/622b6263ab9ff6015.png_e600.jpg)

10、en 文件夹下的 app.php 便是语言文件，双击打开 app.php 文件，只需要将 类似'settings.system' => 'System' 修改为'settings.system' => '系统'，这样便完成了系统的汉化版，为了方便大家，可以直接下载已经汉化好的 app.php 文件替换原文件即可。

app.php 汉化文件下载 链接：https://pan.baidu.com/s/1LEJFl7sZHYxCorPgtMxycQ 提取码：w6vl  

11、下载好 app.php 直接拖动到 en 文件夹下，覆盖原有文件，如下图。

![](https://qnam.smzdm.com/202203/11/622b65bb01ad5122.png_e600.jpg)

12、无需任何重启操作，回到浏览器刷新页面，发现已经变为中文。

![](https://qnam.smzdm.com/202203/11/622b66118cbab3094.png_e600.jpg)

Heimdall 细节优化
-------------

1、如果开启了搜索框（设置中主页搜索框更改为 yes）我们会发现搜索框还是英语，且有相关搜索也不是我们所使用的，本次节教程将对搜索框进行优化，修改为 “百度”“[谷歌](https://pinpai.m.smzdm.com/2047/)”“必应”。  

![](https://qnam.smzdm.com/202203/11/622b66c9a3cfe341.png_e600.jpg)

2、按照 heimdall 中的 5-8 步骤，使用 winscp 继续定位到 heimdall 的安装目录，这次我们需要修改几个其他的文件来达到汉化目的。

3、进入目录 app 中双击 Search.php，找到如下文件修改，修改完成以后 ctrl+s 保存。

![](https://qnam.smzdm.com/202203/11/622b689e83ae36303.png_e600.jpg)修改为的文件如↓

![](https://qnam.smzdm.com/202203/11/622b68f50e77e3666.png_e600.jpg)

4、如果和我的需求相同的话也可以下载我修改好的文件直接替换 Search.php。（步骤按照 heimdall 汉化中的步骤 11 操作）。

Search.php 修改文件下载 [https://pan.baidu.com/s/1yutkt_k1Vw4olSDxbftmcQ 提取码：9qqp](https://post.m.smzdm.com/%22%20target=)

5、回到浏览器刷新效果，如图，搜索引擎已经变少。

![](https://qnam.smzdm.com/202203/11/622b6a42bfbb17600.png_e600.jpg)

6、搜索引擎的名字还有问题，我们需要进入 resources→lang→en→app.phh 文件夹下，修改搜索引擎的汉化文件。

![](https://qnam.smzdm.com/202203/11/622b6ba16ea336703.png_e600.jpg)

7、修改效果如下，修改好以后按 ctrl+s 保存退出关闭 winscp 工具。

![](https://qnam.smzdm.com/202203/11/622b6bd1af6741672.png_e600.jpg)

8、回到浏览器点击刷新，发现已经正常，到此 heimdall 的安装、汉化、细节优化基本完成，下一节将是使用上的介绍。  

![](https://qnam.smzdm.com/202203/11/622b6c593d77e9005.png_e600.jpg)

Heimdall 使用
-----------

这一节主要介绍卡片的添加和一些增强型应用。

1、首先对应用列表进行更新。

![](https://qnam.smzdm.com/202203/11/622b6d67347575207.png_e600.jpg)

2、点击添加，弹出如下界面。  

![](https://qnam.smzdm.com/202203/11/622b6d9b83ec21872.png_e600.jpg)

3、这里我将添加一个群辉登录页面，其他情况类似。  

![](https://qnam.smzdm.com/202203/11/622b6e57b53851675.png_e600.jpg)

4、添加好以后这样。

![](https://qnam.smzdm.com/202203/11/622b6e66452dd7622.png_e600.jpg)

5、heimdall 还能显示增强型应用的实时数据，这里有 qb 为例子.

![](https://qnam.smzdm.com/202203/11/622b7115456556949.png_e600.jpg)

6、配置好以后显示如下图。

![](https://qnam.smzdm.com/202203/11/622b714d29a259452.png_e600.jpg)

7、对于有些外网使用的话可以增加密码，这样防止别人篡改你的数据。

![](https://qnam.smzdm.com/202203/11/622b71c5175cd6060.png_e600.jpg)

8、点击编辑，更改用户名，添加密码，勾选‘允许通过公共链接登录 - 仅当密码已经设置的情况下.’，这样在外网访问的时候，如果进行修改需要输入密码。

![](https://qnam.smzdm.com/202203/12/622b729af1fbf4156.png_e600.jpg)配置好以后，再外网修改信息的时候需要输入密码↓

![](https://qnam.smzdm.com/202203/12/622b7375e9a59272.png_e600.jpg)

9、附上自己目前实际使用的。

![](https://qnam.smzdm.com/202203/12/622b7535ef9659884.png_e600.jpg)

总结
--

这篇教程真的写的特别累，博客上的好多图片都丢失了，然后从新部署从新截图上传的，虽然现在 docker 仓库里面有人编译好的镜像直接拉取能省去很多麻烦，但这是后话了，我最开始折腾这个镜像的时候就是这么一步一步来的，本篇教程纯属纪念折腾精神，接下来我陆续会把我博客内容分享出来，类似都是这种服务的搭建，大家还有什么想知道的，想了解的也可以留言，只要我会弄得就一定会写教程，下一篇教程应该写 halo 博客搭建。凌晨了 睡了睡了 安![](https://res.smzdm.com/images/emotions/189.gif)

作者声明本文无利益相关，欢迎值友理性交流，和谐讨论～