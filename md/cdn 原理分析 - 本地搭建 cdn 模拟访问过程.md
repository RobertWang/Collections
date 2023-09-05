> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/u-VWrrdlkRzKs7u04EPV-g)

> ❝
> 
> 这一生听过许多道理，但还是过不好这一生，这是因为缺少真正的动手实践，光听道理，缺少动手实践的过程，学习难免会让人觉得味同嚼蜡，所以我的分享都比较倾向于实践，在一次次动手实践的过程中感受知识原本纯真的模样。
> 
> ❞

大家好，我是蓝胖子，往往从事互联网开发的同学都听过 cdn 这个词，不过对于刚入行的同学可能会对这个概念比较模糊，今天我们就来聊聊它，并且我会在原理的基础上在本地搭建一个 cdn 环境，模拟域名配置，回源，以及缓存的过程。

本节源码已经上传到 github

```
https://github.com/HobbyBear/cdndemo


```

现在，让我们开始吧。

cdn 原理介绍
--------

首先，我们来看下为什么要用 cdn，比如一个专注做视频播放或者图片阅览的网站，当用户浏览网站时，需要从网站拉取图片或者视频资源，这将产生流量费用，并且，如果用户里网站服务器越远，产生的流量费用将越高。而 cdn 的原理则是将视频或者图片资源缓存在离用户比较近的服务器上，这样既提升了响应速率，又节约了流量费。

来看下使用 cdn 后，用户访问网站的过程。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/81178hAUFMicze0mKCb4LY1jctCXDsBPI6vRThWiboagpKhcUkaPicsrlLxS8qJfwAQX5e8D0jPncdV0ayulRFQQQ/640?wx_fmt=png)img.png

如上图所示，假设用户自己的想要加速的域名是 web.cdn.test，如果用户想对这个域名进行加速，首先要去 cdn 服务商那里配置上这个**「加速域名」**和**「源站服务地址」**，接着 cdn 服务商会生成一个域名地址, 这里假设为 web.cdn.test.c.lanpangzi，而用户自己需要去自己的 dns 服务商那里将加速域名 web.cdn.test 指向这个新的域名地址，这种将一个域名指向另一个域名的记录被称作**「cname」**记录。

经过上述步骤后，对域名 web.cdn.test 的访问会被指向 web.cdn.test.c.lanpangzi, 但目前还有一个问题，新的域名 web.cdn.test.c.lanpangzi 应该由 cdn 服务商自己的 dns 调度系统去解析，这样 cdn 服务商才能将边缘节点的 ip 返回给用户，那本地 dns 服务器怎么知道 web.cdn.test.c.lanpangzi 这个域名要交给哪台机器去解析呢？

原因是这样的，cdn 服务商在注册自己的主域名时 (这里的主域名是 langpangzi.) ，向 dns 服务器注册了一条**「ns」**记录，ns 记录可以指定将某个域名的解析交给哪一台 dns 服务器解析，比如这里，cdn 服务商就把**「c.langpangzi.」** 域名的解析指向了 cdn 服务商自己的 dns 服务器。

完成了这一步，用户自己的加速域名最终就会被 cdn 服务商的 dns 服务器去解析了，cdn 服务商一般在全球各地都有自己的节点，所以它会根据用户的 ip 去筛选一个离用户比较近节点 ip 返回，这些节点被称作**「边缘节点」** ，这样用户的请求就能就近访问了。

本地搭建一个 cdn
----------

我们把上面的请求过程与 cdn 架构在本地模拟下，把自己想象成一个 cdn 服务商，我们将会搭建 cdn 的域名调度中心，和边缘节点，然后允许用户提供加速域名和回源服务器给我进行配置。

### 搭建 dns 服务器

我们先来搭建一个 dns 服务器，因为无论是 cdn 还是用户本身都需要进行一些 dns 域名配置，你可以把当前这个 dns 服务器想象成第三方 dns 运营商。

我们用 dnsmasq 在本地进行 dns 服务器的搭建，我本地机器用的 mac，安装命令如下:

```
brew install  dnsmasq

==> Dependencies
Build: pkg-config ✔
==> Caveats
To start dnsmasq now and restart at startup:
  sudo brew services start dnsmasq
Or, if you don't want/need a background service you can just run:
  /opt/homebrew/opt/dnsmasq/sbin/dnsmasq --keep-in-foreground -C /opt/homebrew/etc/dnsmasq.conf -7 /opt/homebrew/etc/dnsmasq.d,*.conf
==> Analytics
install: 0 (30 days), 0 (90 days), 0 (365 days)
install-on-request: 0 (30 days), 0 (90 days), 0 (365 days


```

接着需要对其配置文件进行修改，配置上游服务器，以及配置当前 dns 服务器能够解析的域名

从安装的信息可以看出，其配置文件是在 / opt/homebrew/etc/dnsmasq.conf 这个位置，这里我直接给出我的配置信息

```
# 配置上行DNS，对应no-resolv
resolv-file=/Users/xiongchuanhong/dnsmasq.conf
# 运行进程以哪个用户身份运行，直接用root，因为dnsmasq的配置涉及到权限问题
user=root
# 配置dnsmqsq运行日志，对排错很重要
log-facility=/Users/xiongchuanhong/logs/dnsmasq.log
# resolv.conf内的DNS寻址严格按照从上到下顺序执行，直到成功为止
strict-order
# DNS解析hosts时对应的hosts文件，对应no-hosts
addn-hosts=/etc/hosts
cache-size=1024
# 多个IP用逗号分隔，192.168.x.x表示本机的ip地址，只有127.0.0.1的时候表示只有本机可以访问。
# 通过这个设置就可以实现同一局域网内的设备，通过把网络DNS设置为本机IP从而实现局域网范围内的DNS泛解析(注：无效IP有可能导至服务无法启动）192.168.17.150 是我本地机器内网ip
listen-address=127.0.0.1,192.168.17.150
# 相当于ns记录，c.lanpangzi的域名以及其子域名都会由本地1053端口的进程去进行解析
server=/c.lanpangzi/127.0.0.1#1053
# cname记录，访问web.cdn.test 的域名都会指向web.cdn.test.c.lanpangzi 
cname=web.cdn.test,web.cdn.test.c.lanpangzi
# 重要！！这一行就是你想要泛解析的域名配置.
#address=/hello.me/127.0.0.1


```

上面 resolv-file 所指向的文件是要配置的上游 dns 服务器,/Users/xiongchuanhong/dnsmasq.conf 配置如下:

```
(base) ➜  ~ cat /Users/xiongchuanhong/dnsmasq.conf
 nameserver 8.8.8.8


```

当本地解析不了域名，那么 dnsmasq 会询问它的上游 dns 服务器，所以我们把它配置成谷歌的域名解析系统。

> ❝
> 
> 📢📢注意 Users/xiongchuanhong/dnsmasq.conf 和 Users/xiongchuanhong/logs/dnsmasq.log 文件都需要用 root 用户去创建，不然到时候运行会报权限错误。
> 
> ❞

接着启动 dnsmasq 服务

```
sudo brew services start  dnsmasq


```

然后再把机器的 dns 服务器 ip 改成 127.0.0.1

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/81178hAUFMicze0mKCb4LY1jctCXDsBPIOSZG3ML9Sc59ZRVK2AnBkq82ngVicrPQVBQWK14GDUHXVOSzBRECHQw/640?wx_fmt=jpeg)1791686894116_.pic.jpg![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/81178hAUFMicze0mKCb4LY1jctCXDsBPIj13HADTmP6m8Fxe5VRGBVliaicmmP9e6wsmpzho2FcNicgL1KwsibQAqgg/640?wx_fmt=jpeg)1801686894126_.pic.jpg

接着测试下修改后的网络访问有没有问题

```
(base) ➜  ~ ping www.baidu.com
PING www.a.shifen.com (14.119.104.189): 56 data bytes
64 bytes from 14.119.104.189: icmp_seq=0 ttl=54 time=36.413 ms
64 bytes from 14.119.104.189: icmp_seq=1 ttl=54 time=11.351 ms
64 bytes from 14.119.104.189: icmp_seq=2 ttl=54 time=11.339 ms
64 bytes from 14.119.104.189: icmp_seq=3 ttl=54 time=16.660 ms


```

域名解析正常，接着进行下面的步骤。

### 搭建回源服务器和 cdn 边缘节点

接着我们来快速搭建下回源服务器和 cdn 的边缘节点，这里我是直接借用了 nginx 来进行搭建，因为 nginx 可以用作静态资源服务器，所以可以直接启动一个 nginx 容器把它作为源服务器，提供图片资源的访问。 并且 nginx 也能提供缓存服务，这也刚好符合边缘节点缓存资源的需求。由于边缘节点需要回源到源服务器，但是容器的 ip 又是不固定的，所以这里我直接用 docker-compose 对这两个容器进行了编排，这样在写 nginx 配置文件的时候，便可以用容器名代替 ip 了。

#### docker-compose.yaml 配置文件

```
version: "3"  
services:  
  sourceserver:  
    image: nginx:latest  
    container_name: source-server  
    hostname: source-server  
    ports:  
      - '8080:80'  
    volumes:  
      - ./sourceserver/nginx.conf:/etc/nginx/nginx.conf  
      - ./sourceserver/logs:/var/log/nginx  
      - ./sourceserver/imgs:/imgs  
  
  edgenode:  
    image: nginx:latest  
    container_name: edgenode  
    hostname: edgenode  
    ports:  
      - '80:80'  
    volumes:  
      - ./edgenode/nginx.conf:/etc/nginx/nginx.conf  
      - ./edgenode/logs:/var/log/nginx  
      - ./edgenode/cache:/cache


```

#### 源站服务器的 nginx 配置

```
  
worker_processes  1;  
error_log   /var/log/nginx/error.log;  
events {  
    worker_connections  1024;  
}  
http {  
    default_type  application/octet-stream;  
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '  
                      '$status $body_bytes_sent "$http_referer" '  
                      '"$http_user_agent" "$http_x_forwarded_for"';  
    # 静态资源配置  
    server {  
        listen       80;  
        access_log  /var/log/nginx/access.log  main ;  
        location /static {  
            alias   /imgs;  
        }  
    }  
}


```

#### 边缘节点的 nginx 配置

在 server 模块里，我们配置 proxy_pass 转发路径时，直接用源服务器的容器名代替了 ip，http://sourceserver。当本地没有缓存时，就会通过 http://sourceserver 访问源站服务器。

> ❝
> 
> 📢📢注意下，在真实环境里，边缘节点的源站服务器名和加速域名肯定不是写死的，是用户配置在 cdn 服务商的数据库里，然后边缘节点再从服务器里读取的。
> 
> 并且边缘节点要求只有用户的加速域名才能访问，所以在转发请求前还要判断下域名是不是用户的加速域名。
> 
> ❞

```
worker_processes  1;  
error_log   /var/log/nginx/error.log;  
events {  
    worker_connections  1024;  
}  
http {  
    log_format  main  '$host- $remote_addr - $remote_user [$time_local] "$request" '  
                      '$status $body_bytes_sent "$http_referer" '  
                      '"$http_user_agent" "$http_x_forwarded_for"';  
      proxy_cache_path  /cache levels=1:2 keys_zone=lanpangzi-cache:20m max_size=50g inactive=168h;  
      proxy_cache lanpangzi-cache;  
      proxy_cache_valid 168h;  
  server {  
      root /static;  
      listen       80;  
      location ~* \.(css|js|png|jpg|jpeg|gif|gz|svg|mp4|ogg|ogv|webm|htc|xml|woff)$ {  
        if ($host != 'web.cdn.test')  {  
             return 403;  
           }  
      access_log  /var/log/nginx/access.log  main;  
      proxy_pass http://sourceserver;  
    }  
  }  
}


```

这样就快速搭建好了源服务器和边缘节点，我们到时候直接用 docker-compose up 命令便可以启动容器，边缘节点监听了本地 80 端口。

### 搭建 cdn 的域名调度系统

最后，我们来搭建下 cdn 的域名调度系统，防止遗忘，我们再来串联下配置 cdn 的过程，用户拥有自己的加速域名和源站服务器，并且将这两个告诉了 cdn 服务商，cdn 服务商就产生一个新域名，这个新域名需要配置为 cname 记录，与用户的加速域名关联起来，这条 cname 记录作我们在搭建 dnsmasq 时已经写死在了配置文件里。

```
cname=web.cdn.test,web.cdn.test.c.lanpangzi


```

> ❝
> 
> 📢📢注意，现实中，cdn 服务商新生成的域名需要用户自己去其 dns 服务商那里去进行配置
> 
> ❞

同时，cdn 服务商生成的新域名解析因为配置的**「ns」**记录，已经将新的这个域名解析指向了自己的 dns 域名调度系统，到时候通过这个系统便可以返回给用户最近的边缘节点的 ip。

这条 ns 记录我们也写死在了 dnsmasq 的配置文件里。

```
server=/c.lanpangzi/127.0.0.1#1053


```

到时候我们本地会在 1053 端口启动一个进程用于对 c.lanpangzi 域名进行解析。

接着来思考🤔下如何写一个域名调度系统，我们的服务通信需要遵循 dns 协议，由于我们的边缘节点是部署到本地，ip 是 127.0.0.1，所以当解析到请求时要询问 web.cdn.test.c.lanpangzi 这个域名时，直接返回本地这个 ip 即可。 那对于其他域名如何处理，当然是直接丢掉，因为 cdn 的域名调度系统，只提供对自己 cdn 服务商生成的域名进行解析。

用 golang 简单实现代码如下:

这里我直接在代码里将 cname 域名以及边缘节点的 ip 写死在代码里了 (重在模拟 cdn 请求过程)，只要是 web.cdn.test.c.lanpangzi. 这个域名就返回边缘节点 ip，由于我们都是部署到本地，所以边缘节点 ip 就是 127.0.0.1 了，dns 的解析用了现成的 golang 库。

```
import (  
   "github.com/miekg/dns"  
   "log"   "net")  
  
var cdnConfig = map[string]string{  
   "web.cdn.test.c.lanpangzi.": "127.0.0.1",  
}  
  
// 处理到来的请求  
func handler(writer dns.ResponseWriter, req *dns.Msg) {  
   var resp dns.Msg  
   resp.SetReply(req) // 创建应答  
   for _, question := range req.Question {  
      ip := cdnConfig[question.Name]  
      if len(ip) == 0 {  
         return  
      }  
      recordA := dns.A{  
         Hdr: dns.RR_Header{  
            Name:   question.Name,  
            Rrtype: dns.TypeA,  
            Class:  dns.ClassINET,  
            Ttl:    100,  
         },  
         A: net.ParseIP(ip).To4(), // 全部解析为127.0.0.1  
      }  
      resp.Answer = append(resp.Answer, &recordA) // 写入应答  
   }  
   err := writer.WriteMsg(&resp) // 回写信息  
   if err != nil {  
      return  
   }  
}  
  
func main() {  
   dns.HandleFunc(".", handler)                   // 绑定函数  
   err := dns.ListenAndServe(":1053", "udp", nil) // 启动  
   if err != nil {  
      log.Println(err)  
   }  
}


```

### 测试

接着，我们就可以测试下现在搭建的这个 cdn 系统了，虽然我们将很多配置信息都写死在了代码里，但你可以假装他们是自动配置上的🐶。

来看下现在的代码框架.

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/81178hAUFMicze0mKCb4LY1jctCXDsBPIxYdRmU4BDgWtYw1FPHqYR81NsKeiaw8EBAlhiaSmk696CnsHhYWzvLnA/640?wx_fmt=jpeg)Pasted image 20230616164830.png

cnddns 就是模拟的 cdn 调度中心了，edgenode 和 sourceserver 放的都是 nginx 的配置文件，docker-compose.yaml 到时候会启动他们。

#### 启动源站服务器，边缘节点

```
(base) ➜  cdndemo git:(master) ✗ docker-compose up 
Starting edgenode      ... done
Starting source-server ... done
Attaching to edgenode, source-server
source-server   | /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
source-server   | /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
source-server   | /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh



```

#### 启动 cdn 调度中心

```
(base) ➜  cdndemo git:(master) ✗ cd cdndns            
(base) ➜  cdndns git:(master) ✗ go run main.go  


```

#### 测试访问

我们已经把本地机器的 dns 服务器改为 dnsmasq 了，接着就是最后测试下访问是否生效。

我在浏览器连续进行了两次访问

```
http://web.cdn.test/static/1781686486866_.pic.jpg


```

接着查看下 nginx 的 access.log

边缘节点的日志

```
web.cdn.test- 172.22.0.1 - - [16/Jun/2023:08:53:12 +0000] "GET /static/1781686486866_.pic.jpg HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36" "-"  
web.cdn.test- 172.22.0.1 - - [16/Jun/2023:08:53:31 +0000] "GET /static/1781686486866_.pic.jpg HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36" "-"


```

源站服务器日志

```
sourceserver- 172.22.0.3 - - [16/Jun/2023:08:53:12 +0000] "GET /static/1781686486866_.pic.jpg HTTP/1.0" 200 265084 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36" "-"


```

可以看见当第二次访问的时候，源站服务器上没有日志了，边缘节点仍然有访问日志，说明第二次访问已经直接从边缘节点读取到了图片信息。

还是强烈建议把源码下下来亲自实验一遍，你会更加深刻👍🏻。