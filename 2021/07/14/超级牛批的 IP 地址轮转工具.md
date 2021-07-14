> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI4ODU5NzY1MQ==&mid=2247497381&idx=3&sn=ae001505d5286f83671ab3781b03e496&chksm=ec394559db4ecc4f97a1f7aa77d2044fddd580aee1b714d36a494f11d8bad483056efa35f669&mpshare=1&scene=1&srcid=0714ZMdwCDg5ap1rt6za4zs2&sharer_sharetime=1626229160414&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

关注「Github 大本营」，选择 “设为星标”  

回复「资源」，赠送 价值 12800 元 编程资料一份~

公众号

关于 mubeng
---------

> 代理 IP 轮换：在每次发送请求之后变更你的 IP 地址。
> 
> 代理检测：检测你的代理 IP 是否仍然可用。
> 
> 支持所有的 HTTP/S 方法。
> 
> 支持传递所有的参数和 URI。
> 
> 支持 HTTP&Socksv5 代理协议。
> 
> 易于使用：你可以直接使用自己的代理文件来配置和运行 mubeng，并选择需要执行的操作。
> 
> 跨平台特性：无论你使用的是 Windows、Linux、macOS 或是树莓派，你都可以正常使用 mubeng。

工具安装
----

### 预编译源码安装

### Docker 安装

### 源码安装

```
▶ GO111MODULE=on go get -u ktbs.dev/mubeng/cmd/mubeng
```

工具使用
----

```
▶ mubeng [-c|-a :8080] -f file.txt [options...]
```

### 工具选项

```
▶ mubeng -h
```

<table width="677"><thead><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important;"><th data-darkmode-bgcolor-16262570495684="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16262570495684="#fff|rgb(240, 240, 240)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); font-size: 0.8em; box-sizing: border-box !important;"><strong data-darkmode-bgcolor-16262570495684="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16262570495684="#fff|rgb(240, 240, 240)">参数选项</strong></th><th data-darkmode-bgcolor-16262570495684="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16262570495684="#fff|rgb(240, 240, 240)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); font-size: 0.8em; box-sizing: border-box !important;"><strong data-darkmode-bgcolor-16262570495684="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16262570495684="#fff|rgb(240, 240, 240)">描述</strong></th></tr></thead><tbody><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important;"><td data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); font-size: 0.8em; box-sizing: border-box !important;">-f, —file</td><td data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); font-size: 0.8em; box-sizing: border-box !important;">代理文件</td></tr><tr data-darkmode-bgcolor-16262570495684="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16262570495684="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248); box-sizing: border-box !important;"><td data-darkmode-bgcolor-16262570495684="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16262570495684="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); font-size: 0.8em; box-sizing: border-box !important;">-a, —address<addr data-darkmode-bgcolor-16262570495684="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16262570495684="#fff|rgb(248, 248, 248)">:</addr></td><td data-darkmode-bgcolor-16262570495684="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16262570495684="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); font-size: 0.8em; box-sizing: border-box !important;">运行代理服务器</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important;"><td data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); font-size: 0.8em; box-sizing: border-box !important;">-d, —daemon</td><td data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); font-size: 0.8em; box-sizing: border-box !important;">代理服务器守护程序</td></tr><tr data-darkmode-bgcolor-16262570495684="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16262570495684="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248); box-sizing: border-box !important;"><td data-darkmode-bgcolor-16262570495684="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16262570495684="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); font-size: 0.8em; box-sizing: border-box !important;">-c, —check</td><td data-darkmode-bgcolor-16262570495684="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16262570495684="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); font-size: 0.8em; box-sizing: border-box !important;">执行代理状态检测</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important;"><td data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); font-size: 0.8em; box-sizing: border-box !important;">-t, —timeout</td><td data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); font-size: 0.8em; box-sizing: border-box !important;">代理服务器检测最大超时（默认为 30s）</td></tr><tr data-darkmode-bgcolor-16262570495684="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16262570495684="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248); box-sizing: border-box !important;"><td data-darkmode-bgcolor-16262570495684="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16262570495684="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); font-size: 0.8em; box-sizing: border-box !important;">-r, —rotate</td><td data-darkmode-bgcolor-16262570495684="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16262570495684="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); font-size: 0.8em; box-sizing: border-box !important;">每次请求后轮转代理 IP 地址（默认为 1）</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important;"><td data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); font-size: 0.8em; box-sizing: border-box !important;">-v, —verbose</td><td data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); font-size: 0.8em; box-sizing: border-box !important;">导出 HTTP 请求 / 响应，或显示无响应的代理服务器</td></tr><tr data-darkmode-bgcolor-16262570495684="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16262570495684="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248); box-sizing: border-box !important;"><td data-darkmode-bgcolor-16262570495684="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16262570495684="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); font-size: 0.8em; box-sizing: border-box !important;">-o, —output</td><td data-darkmode-bgcolor-16262570495684="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16262570495684="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); font-size: 0.8em; box-sizing: border-box !important;">日志输出</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important;"><td data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); font-size: 0.8em; box-sizing: border-box !important;">-u, —update</td><td data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); font-size: 0.8em; box-sizing: border-box !important;">更新 mubeng 至最新稳定版本</td></tr><tr data-darkmode-bgcolor-16262570495684="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16262570495684="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248); box-sizing: border-box !important;"><td data-darkmode-bgcolor-16262570495684="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16262570495684="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); font-size: 0.8em; box-sizing: border-box !important;">-V, —version</td><td data-darkmode-bgcolor-16262570495684="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16262570495684="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; word-break: break-all; hyphens: auto; border-color: rgb(204, 204, 204); font-size: 0.8em; box-sizing: border-box !important;">显示当前 mubeng 版本</td></tr></tbody></table>

工具使用样例
------

```
http://127.0.0.1:8080

https://127.0.0.1:3128

socks5://127.0.0.1:2121

...

...

```

### 代理检测

```
▶ mubeng -f proxies.txt --check --output live.txt
```

### 代理 IP 轮转

```
▶ mubeng -a localhost:8089 -f live.txt -r 10
```

### BurpSuite 代理

项目地址
----

https://github.com/kitabisa/mubeng

许可证协议
-----

参考资料
----

https://golang.org/doc/install

https://portswigger.net/burp/documentation/desktop/getting-started/installing-burp

https://github.com/kitabisa/mubeng/blob/master/.github/CONTRIBUTING.md