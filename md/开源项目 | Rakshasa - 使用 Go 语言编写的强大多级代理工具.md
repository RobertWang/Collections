> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/vNWVb4SZyClYhA_tPSJg9Q)

> 它可以在节点群里面任意两个节点之间转发 TCP 请求和响应，同时支持 socks5 代理，http 代理，并且可以引入外部 http、socks5 代理池，自动切换请求 IP。

rakshasa 是一个使用 Go 语言编写的强大多级代理工具，专为实现**多级代理**，**内网穿透**而设计。它可以在节点群里面任意两个节点之间转发 TCP 请求和响应，同时支持 **socks5 代理**，**http 代理**，并且可以**引入外部 http、socks5 代理池，自动切换请求 IP**。（原文链接跳转到 Github 页面）

节点之间使用内置证书的 TLS 加密 TCP 通讯，再叠加一层自定义秘钥的 AES 加密，可以在所有 Go 支持的平台使用。可以在你所有的的 Windows 和 Linux 服务器上搭建节点并组成节点群网络。

节点分为普通节点 (node) 与控制节点(fullnode)

*   普通节点，无法控制其他节点进行代理、shell 等操作
    
*   控制节点，全功能节点
    

项目结构示例和截图
---------

点击查看更多介绍

win10+Proxifier 实现内网穿透

rakshasa 主被控设计说明

版本迭代
----

*   **v0.1.0**  2023-03-28
    

*   首次发布
    

*   **v0.2.0**  2023-04-02
    

*   更改为 fullnode 版本，fullnode 为全功能版本可以控制别人也能被控
    
*   增加 node 版本，去掉私钥，无法发起代理等关键操作，适合被控
    
*   增加 lite 版本，在上面版本的基础上，精简 cli 交互与 http 代理池，体积缩小 2mb
    
*   优化节点连接逻辑，并且遍历网卡 ip 进行 net.Dail，解决多网卡下，无法连接的问题
    

*   **v0.2.2**  2023-04-08
    

*   增加 http_proxy 重连逻辑，节点掉线后重连 http 代理能够正常重连使用
    
*   优化节点重连逻辑
    
*   增加 uuid 选项，默认 uuid 使用网卡 mac 作为随机数种子，进行生成
    

*   **v0.2.3** 2023-04-22
    

*   优化了连接逻辑，尝试解决连接失败导致 nil 的 bug
    
*   修改一个 map 为 sync.Map，减少代码上 lock 的使用量
    
*   增加了 主控端、被控端说明的 md 文档
    

编译与使用
-----

生成新的证书，编译所有版本节点

```
go run build.go -all


```

编译所有版本节点（不更新证书）

```
go run build.go -all -nocert


```

生成覆盖证书

```
go run build.go -gencert


```

生成控制节点与普通节点

```
go run build.go -fullnode


```

只生成普通节点

```
go run build.go -node


```

证书保存在 cert 目录下，可以使用第三方工具生成，请使用 RSA PKCS1-V1.5

```
private.go     --编译普通节点的时候要删除此文件
private.pem  --与public.pem对应的公钥私钥，普通节点不包含私钥
public.pem
server.crt     --tls通讯证书
server.key    --tls通讯私钥


```

版本区别
----

<table><thead><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); min-width: 85px;"><br></th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); min-width: 85px;">fullnode</th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); min-width: 85px;">node</th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); min-width: 85px;">fullnode_lite</th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); min-width: 85px;">node_lite</th></tr></thead><tbody><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;" class="">连接其他节点</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">√</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">√</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">√</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">√</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">启动本地 socks5 代理</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">√</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">√</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">√</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">√</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">启动本地 http 代理</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">√</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">√</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">√</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">√</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">启动多层代理</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">√</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">×</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">√</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">×</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">远程 shell</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">√</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">×</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">√</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">×</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">其他远程功能</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">√</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">×</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">√</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">×</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">交互式 CLI</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">√</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">√</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">×</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">×</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">check_proxy</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">√</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">√</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">×</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">×</td></tr></tbody></table>

简单来讲

*   fullnode 完全版，能控制别人，也能被控
    
*   node 能连接其他节点，但是不能对其他节点操控，适合作为被控端
    
*   lite 版本，精简掉 cli 和 net/http，与一些 debug 的代码
    

使用图示
----

![](https://mmbiz.qpic.cn/sz_mmbiz_png/akMib3fibarLpVBjTuZRXEcLYSgYf8pXFHkUm8jbOaRNHj9oRjiaY0vc8BlnUiaJgh5LN40aXp33ibDcug3XeIsJYUQ/640?wx_fmt=png)流程

使用方法
----

### 启动一个带 CLI 节点

不带任何参数即可启动：

```
d:\>rakshasa.exe
start on port: 8883
rakshasa>
rakshasa>help

Commands:
  bind              进入bind功能
  clear             clear the screen
  config            配置管理
  connect           进入connect功能
  exit              exit the program
  help              display help
  httpproxy         进入httpProxy功能
  new               与一个或者多个节点连接，使用方法 new ip:端口 多个地址以,间隔 如1080 127.0.0.1:1081,127.0.0.1:1082
  ping              ping 节点
  print             列出所有节点
  remoteshell       远程shell
  remotesocks5      进入remotesocks5功能
  shellcode         执行shellcode
  socks5            进入socks5功能


rakshasa>


```

请查阅 CLI 使用说明了解详细信息

其他启动参数说明
--------

### -nocli

在无法后台执行的情况下，启动一个不带 CLI 的节点:

```
nohup /root/rakshasa -nocli > /root/rakshasa.log 2>&1 &
#Linux下配合nohup后台执行


```

### -p 端口

以指定端口启动:

```
rakshasa -p 8883


```

### -d ip:port,ip:port...

连接下一层代理或更多层代理，多个地址以逗号隔开，生效在最后一个 ip:port：

```
rakshasa -d 192.168.1.1:8883,192.168.1.2:8883,192.168.1.3:8883 -socks5 1080
#从本地1080端口启动一个socks5代理，流量通过三层转发ip最后在192.168.1.3请求目标数据


```

### -socks5 用户名: 密码 @ip: 端口

本地开启 SOCKS5 代理穿透到远程节点，可以不带 - d：

```
rakshasa -socks5 1080 
#不使用-d参数，则表示直接在本机启动一个socks5代理


```

### -remotesocks5 端口

远程开启 SOCKS5 代理流量出口到本地：

```
rakshasa -remotesocks5 1081  -d 192.168.1.2:1080,192.168.1.3:1080
#方向从右往左(加上本机是3个节点)，在192.168.1.3这台机器开启一个socks5端口1081，流量穿透到本地节点出去


```

### -connect ip:port,remote_ip:remote_port

本地监听并转发到指定 IP 端口，使用场景为本机连接 teamserver，隐藏本机 IP：

```
rakshasa -connect 127.0.0.1:50050,192,168,1,2:50050 -d 192.168.1.3:1080,192.168.1.4:1080 
#本机cs连接127.0.0.1:50050实际上通过1.3,1.4节点后，再连接到192.168.1.2:50050 teamserver，teamserver看到你的ip是最后一个节点的ip


```

### -bind  ip:port,remote_ip:remote_port

反向代理模式，必须配合 - d 使用：

```
rakshasa -bind 192.168.1.2:50050,0,0,0.0:50050 -d 192.168.1.3:1080,192.168.1.4:1080
#与上面相反，在最右端节点监听端口50050，流量到本机节点后，最终发往192.168.1.2，最终上线ip为本机ip


```

### -http_proxy 用户名: 密码 @ip: 端口

启动一个 http 代理，可以不使用 - d，建议配合 - http_proxy_pool 使用代理池，自动切换代理 ip：

```
rakshasa -http_proxy 8080 -http_proxy_pool out.txt


```

### -password 密钥

各节点除了证书校验之外，还额外支持密钥连接，建议使用并定期更换密钥，以避免二进制泄露后被别人连上

```
rakshasa -password 123456


```

### -f yaml 文件 详细说明

指定配置文件启动。

### -help

更多启动参数使用帮助

关于开源
----

本作品使用 MPL 2.0 许可证，您可以下载、修改和使用本代码。然而，您必须明确表示，任何此类担保、支持、赔偿或责任义务均由您单独提供，与本作者无关。本人不承担您在使用或修改本程序所造成的任何后果或责任。

在遵循 MPL 2.0 许可证的基础上，您可以自由地对 rakshasa 进行修改和扩展，以满足您的特定需求。同时，您可以将改进和新功能贡献给社区，让更多人受益。但请注意，确保在分享和发布修改后的代码时遵守许可证要求，并尊重原作者的版权。