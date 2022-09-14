> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.yyzq.cf](https://www.yyzq.cf/?id=140)

[![](https://www.yyzq.cf/zb_users/upload/2022/03/20220312191949164708398970641.png)](https://www.yyzq.cf/zb_users/upload/2022/03/20220312191949164708398970641.png)

 **NginxProxyManager 是一款使用超级简单的 Nginx 反向代理管理器 + SSL 证书申请神器。**

**开源项目：**[**https://github.com/NginxProxyManager/nginx-proxy-manager**](https://github.com/NginxProxyManager/nginx-proxy-manager) **目前（2022.03.12）已经** [**6.3k stars**](https://github.com/NginxProxyManager/nginx-proxy-manager/stargazers)

**官方网站：**[**https://nginxproxymanager.com**](https://nginxproxymanager.com/)

**主要特点：**

1. 漂亮的用户界面：基于 Tabler，界面使用起来很愉快。配置服务器从未如此有趣。
-------------------------------------------

2. 代理主机：公开您的专用网络 Web 服务并随时随地连接。
-------------------------------

3. 免费 SSL：内置的 Let's Encrypt 支持允许您免费保护您的 Web 服务。证书甚至可以自我更新！
----------------------------------------------------------

4. 部署简单：Nginx 代理管理器构建为 Docker 映像，而无需对 Nginx 或 Letsencrypt 有太多了解
---------------------------------------------------------------

5. 多个用户：配置其他用户查看或管理他们自己的主机。完全访问权限可用。
------------------------------------

**开始安装：**  
**一、安装 Docker**  

**1.1****(以下安装 docker 步骤适用于 Centos，其他系统安装请参考** [**Docker 官方文档**](https://docs.docker.com/engine/install/centos/)**。)**

```
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo    
sudo yum-config-manager --enable docker-ce-nightly
sudo yum install docker-ce docker-ce-cli containerd.io

```

### **1.2 容器 docker 管理：**

```
systemctl start docker  #启动容器
systemctl enable docker #开机自启
systemctl status docker #查看状态
docker --version #查看docker版本

```

**二、**安装**** **Docker Compose**

**2.1. 下载安装**

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

```

**2.2. 给执行权限**

```
sudo chmod +x /usr/local/bin/docker-compose

```

**2.3. 创建链接**  

```
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

```

**2.4. 查看 docker-compose 版本**

```
docker-compose --version

```

**三、部署 Nginx Proxy Manager  服务**

**3.1 创建一个与此类似的 docker-compose.yml 文件**

```
mkdir ~/npm #创建一个目录用来安装此服务
cd ~/npm #进入目录

```

```
vim docker-compose.yml  #将以下代码粘贴到里面然后保存退出

```

```
version: '3'
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt

```

**3.2 在当前目录运行以下命令安装此服务  
**

```
docker-compose up -d   #部署服务

```

**3.3 登录到管理 UI**

用你的 vps 的 ip 加 81 端口进行访问

默认管理员用户：

```
Email:    admin@example.com
Password: changeme

```

[![](https://www.yyzq.cf/zb_users/upload/2022/03/20220312200007164708640761722.png)](https://www.yyzq.cf/zb_users/upload/2022/03/20220312200007164708640761722.png)

  四、登录使用

...........

[视频教程](https://www.ixigua.com/7074536443745567245?logTag=3611c79a8854b8c38ed8)：

over

[](##read_time)[![](https://www.yyzq.cf/zb_users/plugin/zharry_Reading_time/images/tishi.gif)](https://www.yyzq.cf/zb_users/plugin/zharry_Reading_time/images/tishi.gif)  您阅读本篇文章共花了： 0 小时 00 分 03 秒

分享到：

_阅读剩余的 74%_