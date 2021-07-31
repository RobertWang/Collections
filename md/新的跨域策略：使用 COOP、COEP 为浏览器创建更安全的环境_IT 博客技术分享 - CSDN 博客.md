> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_41646249/article/details/107760122)

Web 资源
------

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9hRG9ZdmVwRTV4MkdiNmRZYzl1RWJBZHJ2eVNCTElPSTR0SVNpYUJoUDhQbzBzeVR3anFRblRwZ0Z6THFBYlllTUJBSEFOUDFUYkZJa3hkb2liT29namtBLzY0MA?x-oss-process=image/format,png)

这些服务非常强大，也很方便，但是这样的策略同样会加大信息泄漏的风险，攻击者可以利用某些手段泄漏你的用户信息。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9hRG9ZdmVwRTV4MkdiNmRZYzl1RWJBZHJ2eVNCTElPSUpWT3NwOGs0cDdjaWFpY0FQN3N3bHBsSnVBYzlRaWFwcGhlQUFyVktxR2lhV0w3djh5RGZaZXRKY3cvNjQw?x-oss-process=image/format,png)

浏览器在阻止这些攻击上做的也很好。同源策略我们已经很熟悉了，它用于限制不同源的站点的资源访问。详细可以戳浏览器的同源策略，这里不再过多介绍。

但是同源策略也有一些例外，任何网站都可以不受限制的加载下面的资源：

*   嵌入跨域 `iframe`
    
*   `image、script` 等资源
    
*   使用 DOM 打开跨域弹出窗口
    

对于这些资源，浏览器可以将各个站点的跨域资源分隔在不同的 `Context Group` 下，不同的 `Context Group` 下资源无法相互访问。

浏览器 `Context Group` 是一组共享相同上下文的 tab、window 或 iframe。例如，如果网站（`https://a.example`）打开弹出窗口（`https://b.example`），则打开器窗口和弹出窗口共享相同的浏览上下文，并且它们可以通过 `DOM API`相互访问，例如 `window.opener`。

Spectre 漏洞
----------

长久以来，这些安全策略一直保护着网站的隐私数据，直到 `Spectre` 漏洞出现。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9hRG9ZdmVwRTV4MkdiNmRZYzl1RWJBZHJ2eVNCTElPSXFER0xlZ1paOVpyWTRlR1NDU2lhNzdNYlc2RFBBVmpyQjM1OVZ6aWJyYTZVSFFZakFJNHRnOHlBLzY0MA?x-oss-process=image/format,png)

`Spectre` 是一个在 `CPU` 中被发现的漏洞，利用 `Spectre` ，攻击者可以读取到在统一浏览器下任意 `Context Group` 下的资源。

特别是在使用一些需要和计算机硬件进行交互的 `API` 时：

*   `SharedArrayBuffer (required for WebAssembly Threads)`
    
*   `performance.measureMemory()`
    
*   `JS Self-Profiling API`
    

为此，浏览器一度禁用了 `SharedArrayBuffer` 等高风险的 `API`。

跨域隔离
----

为了能够使用这些强大的功能，并且保证我们的网站资源更加安全，我们需要为浏览器创建一个跨域隔离环境。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9hRG9ZdmVwRTV4MkdiNmRZYzl1RWJBZHJ2eVNCTElPSWIzZkxUelM3ZUluN0pIYXBiYTJDcEFLQ3VJdDNta0lVM1JFdEhUSDd4aWI1Y0luRDhOcmljWUx3LzY0MA?x-oss-process=image/format,png)

下文会提到很多专有术语，我们先把所有跨域相关的名词列出来，以防后面搞混：

*   `COEP: Cross Origin Embedder Policy`：跨源嵌入程序策略
    
*   `COOP: Cross Origin Opener Policy`：跨源开放者政策
    
*   `CORP: Cross Origin Resource Policy`：跨源资源策略
    
*   `CORS: Cross Origin Resource Sharing`：跨源资源共享
    
*   `CORB: Cross Origin Read Blocking`：跨源读取阻止
    

我们可以通过 `COOP、COEP` 来创建隔离环境。

```
Cross-Origin-Embedder-Policy: require-corp
Cross-Origin-Opener-Policy: same-origin
```

下面我们来看一下，这两个 `Hedaer` 的意义，以及如何进行配置。

COOP：Cross Origin Resource Policy
---------------------------------

COOP：跨源开放者政策，对应的 `HTTP Header` 是 `Cross-Origin-Opener-Policy`。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9hRG9ZdmVwRTV4MkdiNmRZYzl1RWJBZHJ2eVNCTElPSUJmMm45aWFySlllZ0RKak1XV1ZIdFNsWTVENFU5R2RrZlJyUjE0aWFZUTIybEg1UGxKaEU5aWE3US82NDA?x-oss-process=image/format,png)

通过将 `COOP` 设置为 `Cross-Origin-Opener-Policy: same-origin`，将把从该网站打开的其他不同源的窗口隔离在不同的浏览器 `Context Group`，这样就创建的资源的隔离环境。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9hRG9ZdmVwRTV4MkdiNmRZYzl1RWJBZHJ2eVNCTElPSXkySjdWZ3dHT1ZuaDI4aWJ4MWthU295dmljbGxNekVhUnBrcncwSTlMZTZWVFdxZHpRSUU3aWFHQS82NDA?x-oss-process=image/format,png)

例如，如果带有 `COOP` 的网站打开一个新的跨域弹出页面，则其 `window.opener` 属性将为 `null` 。

除了 `same-origin` 、 `COOP` 还有另外两个不同的值：

```
Cross-Origin-Opener-Policy: same-origin-allow-popups
```

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9hRG9ZdmVwRTV4MkdiNmRZYzl1RWJBZHJ2eVNCTElPSXdsaGVJWHE4d3UyVE1UYTFOc0lQQk9sVWljcld4NjFPMXZTcUxKdGFFQXFPQ3FGeWdycGVjUmcvNjQw?x-oss-process=image/format,png)

带有 `same-origin-allow-popups` 的顶级页面会保留一些弹出窗口的引用，这些弹出窗口要么没有设置 `COOP` ，要么通过将 `COOP` 设置为 `unsafe-none` 来选择脱离隔离。

```
Cross-Origin-Opener-Policy: unsafe-none
```

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9hRG9ZdmVwRTV4MkdiNmRZYzl1RWJBZHJ2eVNCTElPSVBjSHdRd2hXOUowSm1qUjRkREhJd2hMVUJKSVpZWmVvWWlhaWJ3b1NqUDRoSWliYTFrNmVFR0ZyZy82NDA?x-oss-process=image/format,png)

`unsafe-none` 是默认设置，允许当前页面和弹出页面共享 `Context Group`。

CORP、CORS
---------

要启用跨域隔离，你还首先需要明确所有跨域资源明确被允许加载。这有两种实现方式，一种是`CORP`，另一种是 `CORS`。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9hRG9ZdmVwRTV4MkdiNmRZYzl1RWJBZHJ2eVNCTElPSUswaWE5cUMyU2liZng5OWU4ZjF2WXhpYm1aVjZoaWJGaWFXbTNYMXJoeWlhY1U3SVBweHJITENWTTJQdy82NDA?x-oss-process=image/format,png)

`CORS`(跨域资源共享) 在我么日常解决跨域问题时经常会使用，这个我们已经非常熟悉了，我们再来看看 `CORP`：

```
Cross-Origin-Resource-Policy: same-site
```

标记 `same-site` 的资源只能从同一站点加载。

```
Cross-Origin-Resource-Policy: same-origin
```

标记 `same-origin` 的资源只能从相同的来源加载。

```
Cross-Origin-Resource-Policy: cross-origin
```

标记 `cross-origin` 的资源可以由任何网站加载。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9hRG9ZdmVwRTV4MkdiNmRZYzl1RWJBZHJ2eVNCTElPSWYybndEQWRQVXdqV0p5SjhzY2ljSGhqZ25ISUZScEI5a2oxc0wwV1JyZWlhY3d2TnVpY1drU2dvdy82NDA?x-oss-process=image/format,png)

注意，如果是一些通用的 `CDN` 资源，例如 `image、font、video`、等，一定要设置成 `cross-origin` ，否则可能会导致资源无法被正常加载。

> 对于你无法控制的跨域资源，可以手动在 html 标签中添加 `crossorigin` 属性。

COEP：Cross Origin Embedder Policy
---------------------------------

COOP：跨源嵌入程序政策，对应的 `HTTP Header` 是 `Cross-Origin-Embedder -Policy`。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9hRG9ZdmVwRTV4MkdiNmRZYzl1RWJBZHJ2eVNCTElPSVdaU3FzVEp1cW04YWV4ZUlRb3hwNkR6cXVqcTVRN0c3TXNMVWhvVTh5RXpKTWhpY3dpYmZ6VHVBLzY0MA?x-oss-process=image/format,png)

启用 `Cross-Origin-Embedder-Policy: require-corp`，你可以让你的站点仅加载明确标记为可共享的跨域资源，也就是我们上面刚刚提到的配置，或者是同域资源。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9hRG9ZdmVwRTV4MkdiNmRZYzl1RWJBZHJ2eVNCTElPSUc1ZU16M0FGYmFDVUJSMzV0VkJOT3lVM200TEpacXRTblppYjJ1M0p1eWlhMk1zcUhCck5ncllnLzY0MA?x-oss-process=image/format,png)

例如，上面的图片资源如果没有设置 `Cross-Origin-Resource-Policy` 将会被阻止加载。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9hRG9ZdmVwRTV4MkdiNmRZYzl1RWJBZHJ2eVNCTElPSWRnazUwckZZTTFEckZIRFdZNjkwTUxuc2JkWXFudEJldzNpYTVYd0JpYUY1SE54R0VPaWN2TDlDZy82NDA?x-oss-process=image/format,png)

在完全启用 `COEP` 之前，可以通过使用 `Cross-Origin-Embedder-Policy-Report-Only` 检查策略是否能够正常运行。如果有不符合规范的资源，将不会被禁止加载，而是上报到你的服务器日志中。

测试跨域隔离是否正常
----------

当你的 `COOP、COEP` 都配置完成之后，现在你的站点应该处于跨域隔离状态了，你可以通过使用 `self.crossOriginIsolated` 来判断隔离状态是否正常。

```
if(self.crossOriginIsolated){
  // 跨域隔离成功
}
```

好了，你现在可以愉快的使用 `haredArrayBuffer`, `performance.measureMemory` 或者 `JS Self-Profiling API` 这些强大的 API 了～

### 参考

*   https://web.dev/why-coop-coep/
    
*   https://web.dev/coop-coep/