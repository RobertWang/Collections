> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [post.m.smzdm.com](https://post.m.smzdm.com/p/aoxqd82m/?send_by=3547620770&invite_code=zdm4ptnapkinv&zdm_ss=iOS_3547620770_&from=other)

> #我家 NAS 装了啥 #有奖活动正在火热进行中，分享你家 NAS 装了啥 App，赢取 TS-453D mini！→戳此了解←一、看片哒 EMBY 是我目前看片的首

> [**#我家 NAS 装了啥#**](https://post.smzdm.com/p/aen50394/)有奖活动正在火热进行中，分享你家 NAS 装了啥 App，赢取 TS-453D mini！[**→戳此了解←**](https://post.smzdm.com/p/aen50394/)

一、看片哒
-----

[**EMBY**](https://emby.media/) 是我目前看片的首选，它的安装相当方便，只需要无脑下一步即可，配置也是十分轻松，媒体库可以挂载网络位置，客户端和服务端支持多平台（你能想到的，客户端都支持）

![](https://am.zdmimg.com/202107/17/60f231ac80cd53882.png_e600.jpg)EMBY 媒体库设置

二、下载哒
-----

[Transmission](https://transmissionbt.com/) 是我主要的下载工具，主要用来下载 PT，也可以下载 BT，不过不能很方便的过滤吸血雷

![](https://am.zdmimg.com/202107/17/60f2331609ecb1732.png_e600.jpg)Transmission Remote GUI

![](https://qnam.smzdm.com/202107/17/60f23382b0a6d5088.png_e600.jpg)Transmission

[aria2c](https://github.com/) 我只用来下载网盘内的东西，不过也可以下载 BT

![](https://am.zdmimg.com/202107/17/60f233e14ab7e1133.png_e600.jpg)AriaNg

三、备份哒
-----

### 1、Syncthing  

支持多平台的备份工具，我主要用它在 Windows 上备份，它会在备份目录下创建 “.stfolder” [文件夹](https://m.smzdm.com/fenlei/wenjianjia/)

![](https://qnam.smzdm.com/202107/17/60f237b89b7e13772.png_e600.jpg)Syncthing 界面

### 2、FolderSync

这个工具只有[手机](https://m.smzdm.com/fenlei/zhinengshouji/)端，但是支持多种备份方式（FTP、SMB、WebDAV 及一堆国外网盘）  

![](https://am.zdmimg.com/202107/17/60f238739f1398286.jpg_e600.jpg)FolderSync 主界面及部分支持的备份方式

四、搜素哒
-----

Everything 支持 Web 访问的搜索工具  

![](https://qnam.smzdm.com/202107/17/60f23ad851ea37965.png_e600.jpg)Everything Web 界面

五、去重哒
-----

DuplicateCleaner 可以搜索出相同或相似的文件并进行删除或移动等操作（不过我主要用它找图片）

![](https://qnam.smzdm.com/202107/17/60f23c7f949f5224.png_e600.jpg)图片模式 精确匹配 可以看到有一组图片大小不同

这个程序没有 Web 端，但是可以使用远程桌面连接来操作（也可以搭配 [remote app tool](https://github.com/kimmknight/remoteapptool) 生成单独运行的快捷方式）

六、其他
----

可以使用 Nginx 将上述所有含 Web 界面的工具整合到一个端口

![](https://qnam.smzdm.com/202107/17/60f2412d089928121.png_e600.jpg)将 Transmission、Everything、Emby 整合到一个端口（域名已打码）

配置文件示例：

> server {
> 
> listen 9999 ssl;
> 
> listen [::]:9999 ssl;
> 
> ssl_certificate /etc/nginx/cert / 你的域名 / cert.pem;
> 
> ssl_certificate_key /etc/nginx/cert / 你的域名 / key.pem ;
> 
> server_name 你的域名;
> 
> root 网站位置;
> 
> index index.html index.htm index.php /h5ai/_h5ai/public/index.php;
> 
> #transmission
> 
> location ^~ /transmission {
> 
> proxy_set_[head](https://pinpai.m.smzdm.com/783/)er X-Real-IP $remote_addr;
> 
> proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
> 
> proxy_set_header Host $http_host;
> 
> proxy_pass_header X-Transmission-Session-Id;
> 
> }
> 
> location /transmission/rpc {
> 
> proxy_pass http://127.0.0.1:9091;
> 
> }
> 
> location /transmission/web/ {
> 
> proxy_pass http://127.0.0.1:9091;
> 
> }
> 
> location /transmission/upload {
> 
> proxy_pass http://127.0.0.1:9091;
> 
> }
> 
> #Emby
> 
> location /web {
> 
> proxy_pass http://127.0.0.1:8096;
> 
> }
> 
> location /emby {
> 
> proxy_pass http://127.0.0.1:8096;
> 
> }
> 
> #Everything
> 
> location / {
> 
> proxy_pass http://127.0.0.1:88;
> 
> }
> 
> }