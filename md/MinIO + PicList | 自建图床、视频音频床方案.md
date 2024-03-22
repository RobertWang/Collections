> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [sspai.com](https://sspai.com/post/86900)

> 在使用 MinIO+PicList 图床方案之前，我也自建过兰空图床、EasyImages2、Chevereto，不过它们更像是公开给别人临时使用的，而不是自己用的，用着不直观。

![](https://cdn.sspai.com/2024/03/05/c390bfeca8b962234bb0deb98e22231c.gif)

介绍 MinIO 如何作为图床
---------------

MinIO 属于对象存储，就是一个网络目录，可以上传文件、下载文件，可以通过网址访问文件。

*   如果我们**上传图片**到 MinIO、并构造正确的网址，就能在网页上显示这张图片了——图床🎉
*   如果**上传视频**，那就能直接在网页上播放视频（前提是浏览器支持播放的格式，比如 mp4）——视频床🎉
*   如果**上传音频**，就是音频床
*   如果**上传普通文件**，可以当作网盘用来分享文件

**安装 MinIO**： [自建 对象存储 - 构建高效可靠的数据存储系统 - 技焉洲 (vfly2.com)](https://sspai.com/link?target=https%3A%2F%2Ftechnique.vfly2.com%2F2024%2F02%2Fself-host-object-storage%2F)

手动上传并构造网址是完全可以的，上传的文件的下载链接格式为（填入 ip 或域名、桶名 vfly2 和文件名 file.name，注意端口是 9000）

```
http://ip:9000/vfly2/file.name

```

比如，在 MinIO 后台创建了一个 imagesbed 桶，在里面上传了一张名为 vfly2technique.png 的图片，那么这张图片的网络地址就是： http://imgbed.ahfei.blog:9000/imagesbed/vfly2technique.png ，用浏览器访问这个网址就能看到图片：

![](https://cdn.sspai.com/2024/03/05/article/d11083cb1c8a2fc4a784937a6451dd7e)

在写 md 格式的文章时，想插入这个图片，使用指定格式就可以了。

```
![](http://imgbed.ahfei.blog:9000/imagesbed/vfly2technique.png)

```

如果借助工具，把上传和构造网址的工作自动化，就更方便了。开源的 PicList 就是专门做这个的。

PicList
-------

PicList 是一款高效的云存储和图床平台管理工具，在 PicGo 的基础上经过深度的二次开发。支持非常丰富的存储方式：WebDAV、S3 API 兼容平台、腾讯 COS V5、Github 等，还有插件功能，通过插件可以支持 MinIO。

GitHub： [Kuingsmile/PicList: An image upload and manage tool, base on PicGo (github.com)](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2FKuingsmile%2FPicList)

### 安装 PicList

下载地址： https://github.com/Kuingsmile/PicList/releases ，找到需要的版本下载安装。

如果访问 GitHub 不畅，也可以到 [UpdateFetch Web (vfly2.com)](https://sspai.com/link?target=https%3A%2F%2Fupdatefetch.vfly2.com%2F) 下载，这是 AhFei 编写并开源的一个自动下载工具。

MacOS 用户现在可以使用 Homebrew 来安装 PicList 了，只需要执行以下命令即可：

```
brew install piclist --cask

```

卸载命令：

```
brew uninstall piclist

```

### 设置 PicList 使用 MinIO 当图床

1.  在【插件】界面，搜索 `MinIO` ，安装作者是 herbertzz 的那个插件。其 GitHub 地址是 https://github.com/Herbertzz/picgo-plugin-minio 。（可能需要先安装 [nodejs](https://sspai.com/link?target=https%3A%2F%2Fnodejs.org%2Fen%2F) ，并且还有重启电脑才能生效）
2.  在【图床】，选择【MinIO 图床】，编辑配置，需要说明的有
    *   endPoint：服务端的域名或者 IP 地址，ib.ahfei.blog
    *   port： 443
    *   userSSL：是否开启 SSL，选择开启
    *   accessKey：安装 Minio Server 时设置的用户名，vfly2
    *   secretKey：之前设置的密码，pass_vfly2_word
    *   bucket：创建的桶名称，imagesbed
3.  保存配置
4.  在【上传】，选择图床，然后随便上传一张图片试试是否成功（实际体验，如果与服务端网络不太好，超过 1 MB 就很容易失败）

### 配合 Obsidian 使用

简单说明作用： **在拖入图片到 Obsidian 时，自动上传到图床**

1.  Obsidian 安装插件：Image Auto Upload Plugin
2.  进入插件设置页面，
    *   修改默认上传器为 `PicGo(app)`
    *   设置 `PicGo server 上传接口` 为 `http://127.0.0.1:36677/upload`
    *   该插件还额外支持通过 PicList 进行云端删除，在删除接口填入 `http://127.0.0.1:36677/delete`
    *   其他设置按需选择
3.  拖入一张图片测试

![](https://cdn.sspai.com/2024/03/05/article/f5049d3f1f4fbfc06e27ad2d22e60383)

效果

![](https://cdn.sspai.com/2024/03/05/fa341f9ba06ba29f3caa64bab0f738d2.gif)

### 视频床

上一节也提到了，其实就是把视频这个文件的网址，按照一定格式构造。`$url` 替换为真实网址：

```
<video controls> <source src="$url"> not support video tag. </video>

```

比如在 Obsidian，是支持 HTML 标签的，因此，放入这样的段落，就是在插入视频

```
<video controls> <source src="https://ib.ahfei.blog/videobed/lightningrod-vid_wg_720p.mp4"> not support video tag. </video>

```

### 音频床

可以用于 Anki 的自制卡片，格式为

```
<audio controls> <source src="$url" type="audio/mpeg"> not support audio tag </audio>

```

比如

```
<audio controls> <source src="https://ib.ahfei.blog/audiobed/indulge-23-09-54.mp3" type="audio/mpeg"> not support audio tag </audio>

```

not support audio tag

> 如果不显示，是因为这个网站平台不支持，可以到博客查看效果。

### 分享普通文件

学会构造网址就行。格式为

```
http://ip:9000/bucket_name/file.name

```

> 原文链接： [https://technique.vfly2.com/2024/03/self-hosted-image-hosting-service/](https://sspai.com/link?target=https%3A%2F%2Ftechnique.vfly2.com%2F2024%2F03%2Fself-hosted-image-hosting-service%2F)

> 版权声明：本博客所有文章除特別声明外，均为 AhFei 原创，采用 [CC BY-NC-SA 4.0](https://sspai.com/link?target=https%3A%2F%2Fcreativecommons.org%2Flicenses%2Fby-nc-sa%2F4.0%2Fdeed.zh) 许可协议。转载请注明来源 [技焉洲 (technique.vfly2.com)](https://sspai.com/link?target=https%3A%2F%2Ftechnique.vfly2.com%2F) 。