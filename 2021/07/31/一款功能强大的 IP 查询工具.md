> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666556038&idx=3&sn=f010402dc2ce2ad81c812123d93842f7&chksm=80dcae2db7ab273bb9c01793389b59da6052f858c1c09a66d188600d52fbc2b76e84c2515ba5&scene=21#wechat_redirect)

### Fav-up

Fav-up 是一款功能强大的 IP 查询工具，该工具可以通过 Shodan 和 Favicon（网站图标）来帮助研究人员查询目标服务或设备的真实 IP 地址。

### 工具安装

首先，该工具需要本地设备安装并部署好 Python 3 环境。然后广大研究人员需要使用下列命令将该项目源码克隆至本地：

```
git clone https://github.com/pielco11/fav-up.git
```

接下来， 运行下列命令安装好 Fav-up 所需的依赖组件：

```
pip3 install -r requirements.txt
```

除此之外，你还需要一个 Shodan API 密钥！

### 工具使用

#### 命令行接口

首先，你需要确定如何传递你的 API 密钥：

-k 或—key：向 stdin 传递密钥；  
-kf 或—key-file：传递获取密钥的目标文件名；  
-sc 或—shodan-cli：从 Shodan 命令行接口获取密钥；

配置好密钥之后，我们就能够以下列几种不同方式使用 Fav-up 了：

-f 或—favicon-file：在本地存储的需要查询的 Favicon 网站图标文件；  
-fu 或—favicon-url：无需在本地存储 Favicon 网站图标，但是需要知道目标图标的实际 URL 地址；  
-w 或—web：如果你不知道 Favicon 网站图标的实际 URL，可以直接传递目标站点地址；  
-fh 或—favicon-hash：在全网搜索 Favicon 网站图标哈希；

你可以指定包含了 Favicon 网站图标的 URL 和域名的输入文件，或者直接提供 Favicon 网站图标的本地存储路径：

-fl 或—favicon-list：文件包含所有待查询 Favicon 网站图标的完整路径；  
-ul 或—url-list：文件包含所有待查询 Favicon 网站图标的完整 URL 地址；  
-wl 或—web-list：

当然了，你也可以将搜索结果存储至一个 CSV/JSON 文件中：

-o 或—output：指定数据输出文件和格式，比如说 csv，它会将存储结果存储至一个 CSV 文件中；

### 工具使用样例

#### Favicon-file

```
python3 favUp.py --favicon-file favicon.ico -sc
```

#### Favicon-url

```
python3 favUp.py --favicon-url https://domain.behind.cloudflare/assets/favicon.ico -sc
```

#### Web

```
python3 favUp.py --web domain.behind.cloudflare -sc
```

### 作为模块导入使用

```
from favUp import FavUp

f = FavUp()          

f.shodanCLI = True

f.web = "domain.behind.cloudflare"

f.show = True

f.run()

for result in f.faviconsList:

    print(f"Real-IP: {result['found_ips']}")

print(f"Hash: {result['favhash']}")
```

### 所有属性

<table width="677" data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)"><thead data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)"><tr data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="max-width: 100%; font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important; overflow-wrap: break-word !important;"><th data-darkmode-bgcolor-16277438317027="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)|rgb(240, 240, 240)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); max-width: 100%; color: inherit; line-height: inherit; font-size: 1em; text-align: left; overflow-wrap: break-word !important; box-sizing: border-box !important;">变量</th><th data-darkmode-bgcolor-16277438317027="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)|rgb(240, 240, 240)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); max-width: 100%; color: inherit; line-height: inherit; font-size: 1em; text-align: left; overflow-wrap: break-word !important; box-sizing: border-box !important;">类型</th></tr></thead><tbody data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)"><tr data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="max-width: 100%; font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); max-width: 100%; color: inherit; line-height: inherit; font-size: 1em; overflow-wrap: break-word !important; box-sizing: border-box !important;">FavUp.show</td><td data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); max-width: 100%; color: inherit; line-height: inherit; font-size: 1em; overflow-wrap: break-word !important; box-sizing: border-box !important;">布尔</td></tr><tr data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="max-width: 100%; font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); max-width: 100%; color: inherit; line-height: inherit; font-size: 1em; overflow-wrap: break-word !important; box-sizing: border-box !important;">FavUp.key</td><td data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); max-width: 100%; color: inherit; line-height: inherit; font-size: 1em; overflow-wrap: break-word !important; box-sizing: border-box !important;">字符串</td></tr><tr data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="max-width: 100%; font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); max-width: 100%; color: inherit; line-height: inherit; font-size: 1em; overflow-wrap: break-word !important; box-sizing: border-box !important;">FavUp.keyFile</td><td data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); max-width: 100%; color: inherit; line-height: inherit; font-size: 1em; overflow-wrap: break-word !important; box-sizing: border-box !important;">字符串</td></tr><tr data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="max-width: 100%; font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); max-width: 100%; color: inherit; line-height: inherit; font-size: 1em; overflow-wrap: break-word !important; box-sizing: border-box !important;">FavUp.shodanCLI</td><td data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); max-width: 100%; color: inherit; line-height: inherit; font-size: 1em; overflow-wrap: break-word !important; box-sizing: border-box !important;">布尔</td></tr><tr data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="max-width: 100%; font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); max-width: 100%; color: inherit; line-height: inherit; font-size: 1em; overflow-wrap: break-word !important; box-sizing: border-box !important;" class="">FavUp.faviconFile</td><td data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); max-width: 100%; color: inherit; line-height: inherit; font-size: 1em; overflow-wrap: break-word !important; box-sizing: border-box !important;">字符串</td></tr><tr data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="max-width: 100%; font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); max-width: 100%; color: inherit; line-height: inherit; font-size: 1em; overflow-wrap: break-word !important; box-sizing: border-box !important;">FavUp.faviconURL</td><td data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); max-width: 100%; color: inherit; line-height: inherit; font-size: 1em; overflow-wrap: break-word !important; box-sizing: border-box !important;">字符串</td></tr><tr data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="max-width: 100%; font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); max-width: 100%; color: inherit; line-height: inherit; font-size: 1em; overflow-wrap: break-word !important; box-sizing: border-box !important;">FavUp.web</td><td data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); max-width: 100%; color: inherit; line-height: inherit; font-size: 1em; overflow-wrap: break-word !important; box-sizing: border-box !important;">字符串</td></tr><tr data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="max-width: 100%; font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); max-width: 100%; color: inherit; line-height: inherit; font-size: 1em; overflow-wrap: break-word !important; box-sizing: border-box !important;">FavUp.shodan</td><td data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); max-width: 100%; color: inherit; line-height: inherit; font-size: 1em; overflow-wrap: break-word !important; box-sizing: border-box !important;">Shodan 类</td></tr><tr data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="max-width: 100%; font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); max-width: 100%; color: inherit; line-height: inherit; font-size: 1em; overflow-wrap: break-word !important; box-sizing: border-box !important;">FavUp.faviconsList</td><td data-darkmode-bgcolor-16277438317027="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277438317027="#fff|rgb(255, 255, 255)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); max-width: 100%; color: inherit; line-height: inherit; font-size: 1em; overflow-wrap: break-word !important; box-sizing: border-box !important;">列表 [字典]</td></tr></tbody></table>

### 工具使用截图

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibdSSiaa0zZNbMhfZtt1oOAOQx1FSbLLjzT1CG4YO9TLkmhtFYVyWxUAYPj8lg6XafExTrLqzhB0Gw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibdSSiaa0zZNbMhfZtt1oOAOXjyialFkylu7sk8eic40YPE0F56LicKUibMRub6qKtDX5cDLjIZcu1Lr0Q/640?wx_fmt=jpeg)

### 项目地址

Fav-up：https://github.com/pielco11/fav-up

> 转自：FreeBuf

- EOF -