> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/PP9cme3v5eRna3G5eWGlEg)

  

<table width="676" class=""><tbody><tr><td width="557" valign="top" height="62" data-style="margin: 0px; padding: 5px 10px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(221, 221, 221); border-style: solid; border-width: 1px; max-width: 100%; overflow-wrap: break-word !important; box-sizing: border-box !important;" class=""><section><strong>声明：</strong>该公众号大部分文章来自作者日常学习笔记，也有部分文章是经过作者授权和其他公众号白名单转载，未经授权，严禁转载，如需转载，联系开白。</section><section>请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者和本<br>公众号无关。</section></td></tr></tbody></table>  

### 购买 IP 地址池

### 获取 API 接口

购买套餐后，选择》API 提取》直接提取，推荐配置如下：

*   1. 流量提取。
    
*   2. 使用时长按需选择，建议选择 25 分钟 - 180 分钟。
    
*   3. 提取数量建议为 5-10，土豪随意。
    
*   4. 建议省份混拨，并选择自己所在省份或临近省份，提高访问速度。
    
*   5. 目前该代理协议仅支持 SOKCS5 连接。
    
*   6. 数据格式选择 Json 格式，方便脚本解析。
    
*   7. 选择属性全部勾选，否则会发生错误。
    
*   8.IP 去重 365 天。
    

### 部署说明

```
将Auto_proxy代码（Auto_proxy_example.yaml, Auto_proxy.py, proxyIgnoreList.plist ）拷贝到Clash配置文件目录下。
Windows默认：Clash\Data\profiles\
Mac默认：~/.config/clash/

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/icdRm5bGXSHcich661L1UPxtdDpYicN4fqWoc1xE4AGRGC0QgCdRqe4h7n7glYxQHgsfLDssGeRSG5DbcSg6o5Aug/640?wx_fmt=jpeg)

修改 Auto_proxy.py 相关配置，主要参数如下。

*   test_url：需要监控测试的 IP 地址。
    
*   py_api：上一步获取的品易 API 接口。
    
*   max_connect_error：错误连接次数，连续连接错误 N 次，重新获取代理。
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/icdRm5bGXSHcich661L1UPxtdDpYicN4fqW0RqXwRY3kXc4Q8xgou3czg1JKTnuX6q9PQGNn3DSpmAnXBvCnseOWg/640?wx_fmt=jpeg)

白名单配置，可参考 https://www.cnblogs.com/PowerTips/p/14775956.html

*   Windows：在 Auto_proxy_example.yaml 添加 cfw-bypass 配置。
    
*   Mac: 直接使用项目中 proxyIgnoreList.plist 即可，需重启生效。
    

注：务必将 *.taolop.com 加入白名单中，不然可能会导致代理失效一直重复获取代理。

### **使用说明**

```
在Clash目录下执行python3 Auto_proxy.py，同时Clash将配置选为Auto_proxy。

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/icdRm5bGXSHcich661L1UPxtdDpYicN4fqWGKEeBMjQtiabkHLfbpJWlNDicicTBumVicwxMaYRdQyPGxa9U7NuLe31ng/640?wx_fmt=jpeg)

需将 Clash 配置为全局模式，同时设置系统代理，目前脚本设置两种规则：

*   加速模式：根据监控网站选择延迟最低的代理。
    
*   负载模式：每次请求都会随机一条代理进行连接。
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/icdRm5bGXSHcich661L1UPxtdDpYicN4fqWmMANSIzGYfMnH6d7LF1SsAnmkrluKClSbMeyeESdrI3zwZOfDAtWBg/640?wx_fmt=jpeg)

负载模式运行效果：

![](https://mmbiz.qpic.cn/mmbiz_jpg/icdRm5bGXSHcich661L1UPxtdDpYicN4fqWicrOV0kmhich8UBEs61v2JZ6dbKh2CDfUXW4WfiaQsOpSGWNAKNDgfRng/640?wx_fmt=jpeg)

当运行错误超出设置阀值，会进行提示 “IP 已被封禁，重新获取代理”，此时 Clash 提示 “重载配置文件”，需手动点击更新。

![](https://mmbiz.qpic.cn/mmbiz_jpg/icdRm5bGXSHcich661L1UPxtdDpYicN4fqW8yCO75jnOgWia1ZMVIicTTjiaGs0yP2QzgQXicPlxAWyccJMgf1rg6s0QA/640?wx_fmt=jpeg)

### 使用效果

该效果模式为负载模式，测试 Dirsearch, 其它工具请自行测试。

*   ```
    靶机端：python3 -m http.server 8000
    攻击端：python3 dirsearch.py -u  http://X.X.X.X:8000 --proxy=http://127.0.0.1:7890
    
    ```
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/icdRm5bGXSHcich661L1UPxtdDpYicN4fqWzlxgRHLtJnLgicTUtvNfIfibeGcQ2Qb1K398MqzYiam8FfiaIJYRTrWFAA/640?wx_fmt=jpeg)

🤣 🤣 🤣  同时 10 个 IP 爆破目录，就问你慌不慌！