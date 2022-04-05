> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NTY1MjY0MQ==&mid=2650846206&idx=5&sn=dad45ad104179799d72ba4f803463f3b&chksm=bd0126b08a76afa6b31199a37096320e433222333ee3d1c923c71097d1dc8c61088566ff9306&mpshare=1&scene=1&srcid=0403b5HyawX2TYDMvF6xLXrO&sharer_sharetime=1648982339087&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

___来源丨网管叨 bi 叨（ID：kevin_tech）___

已获得原公众号授权转载

最近 Go 支持范型的新版本 1.18 已经发布了，那怎么在我们的电脑上安装和配置 Go 1.18 呢，以及假如我有一些非常老的都没有用 Go Modules 管理依赖的项目升级到 1.18 后能兼容吗，今天给大家一一解惑。  

本期的主要涉及的内容有：

*   Go 1.18 的安装和配置
    
*   GOPATH 的推荐设置（是的，虽说不必须，但是我建议你设置）
    
*   还在用 go vendor 的老项目怎么在 Go 1.18 下正常运行
    

注意上面的几个内容并不是文章的目录结构，知识点都在内容里，大家搬好小板凳仔细听讲啦～！

安装演示我是在一台没安装过 Go 的电脑上操作的，偏初学者方向，读者里肯定大部分人都会，也有更 Geek 的安装方式，坐下来当小作文看就行啦～。顺便说一下，诶嘛姨破漏的麦克真香哈。

下载安装 Go
-------

安装包去 Go 的官网下载地址 https://go.dev/dl/ 下载对应系统的包，22 年 3 月 最新的发行版是 1.18

![](https://mmbiz.qpic.cn/mmbiz_png/z4pQ0O5h0f50jtNsRZ6XC4yO3tjDkfPfNr44DQuo9sGtocmWiauWN6zYXCnKbcZh9ojJMFWyOFkEZmXJIpLHwVA/640?wx_fmt=png)

页面上也有其他版本供选择下载。下载完成，打开安装器一路下一步即可：

![](https://mmbiz.qpic.cn/mmbiz_png/z4pQ0O5h0f50jtNsRZ6XC4yO3tjDkfPfsesCRrRthbthzUQm652Dh9fp4cgkmr8JgWibV1WE5JlJqFWbUic7icJLg/640?wx_fmt=png)

安装器会把 Go 安装在 /user/local/go 目录

配置 go env
---------

安装完成后，我们可以在命令行工具执行 go env 命令，看到默认的 go 环境变量：

```
GO111MODULE=""GOARCH="amd64"GOBIN=""GOCACHE=""GOENV=""GOEXE=""GOEXPERIMENT=""GOFLAGS=""GOHOSTARCH="amd64"GOHOSTOS="darwin"GOINSECURE=""GOMODCACHE=""GOOS="darwin"GOPATH="/usrs/local/go"GOROOT="/usr/local/go"....
```

接下来，我们要对几个配置项进行更改，让我们能更好地使用 Go。

*   GOROOT：Go 的安装目录，这个从 1.11 还是 1.12 版本后就不需要我们再自己设置，这里提醒一下。
    
*   GOPATH：
    

*   理论上这个也可以不设置，默认是 /usr/local/go，不过我一般设置成我存放 Go 项目的目录，这样也能兼容一些没有使用 Go Modules 的老项目。虽然 1.12 有了 Go Modules 后，可以把 Go 项目放在任何地方，但我还是习惯统一放在 / Code/Go/src 里，所以在这里执行下面的命令把 GOPATH 设置成 /Users/xxx/Code/Go。
    
*   `go env -w GOPATH="/Users/xxx/Code/Go"`
    
*   那些老项目没有用 Go Modules 的话， 都是按照导入路径约定的目录存放的，比如说导入路径是 example.com/infra/pay 的项目，那么它在本地的目录就是 `$GOPATH/src/example.com/infa/pay` 。
    

另外还有一点要注意，GOMODULECACHE 即 Go Modules 下载在本地的仓库缓存是 $GOPATH/pkg/mod 目录，设置 GOPATH 后会自动随之改变，个人感觉比放在默认的 / usr/local/go/pkg/mod 里让人更舒服些，可能是对自己电脑的洁癖导致的。

*   GOPROXY：Go Modules 的代理地址 ，默认是 https://goproxy.io 但是国内访问较慢，设置成国内代理。
    

```
go env -w GOPROXY="https://goproxy.cn,direct"
```

*   GOPRIVATE：这个初学者一般用不到，如果是引用公司内网的包，可以设置成内网代码仓库的域名，指示 Go Modules 不去代理上下载，而是去这里指定的站点找包。
    

```
go env -w GOPRIVATE="code.inner-company-xxx.com"
```

祖传艺能 -- Hello World
-------------------

安装完后，表演一下祖传艺能，输出个 Hello World。

```
// hello.gopackage mainimport "fmt"func main() { fmt.Println("Hello World!")}
```

命令行切到这个文件的目录，用 `go run ./hello.go`能正常执行后（我还没见过执行不了的情况）就可以开始我们的 Go 搬砖之旅了：）。