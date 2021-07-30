> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzkyODIxMDA3Ng==&mid=2247484512&idx=3&sn=52bc96e7278f5ae4a251314fe2a24cca&chksm=c21d0993f56a808537862d880a3574607879301b4d9f541ce8d59bb1d5a6feb51667c7d694b9&mpshare=1&scene=1&srcid=0727qInWG3p2lQbl47DxR7wp&sharer_sharetime=1627375064566&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

1、输入地址
------

当我们开始在浏览器中输入网址的时候，浏览器其实就已经在智能的匹配可能得 url 了，他会从历史记录，书签等地方，找到已经输入的字符串可能对应的 url，然后给出智能提示，让你可以补全 url 地址。对于 google 的 chrome 的浏览器，他甚至会直接从缓存中把网页展示出来，就是说，你还没有按下 enter，页面就出来了。

2、浏览器查找域名的 IP 地址　　
------------------

2、如果在本地的 hosts 文件没有能够找到对应的 ip 地址，浏览器会发出一个 DNS 请求到本地 DNS 服务器 。本地 DNS 服务器一般都是你的网络接入服务器商提供，比如中国电信，中国移动。

5、本地 DNS 服务器继续向域服务器发出请求，在这个例子中，请求的对象是. com 域服务器。.com 域服务器收到请求之后，也不会直接返回域名和 IP 地址的对应关系，而是告诉本地 DNS 服务器，你的域名的解析服务器的地址。

6、最后，本地 DNS 服务器向域名的解析服务器发出请求，这时就能收到一个域名和 IP 地址对应关系，本地 DNS 服务器不仅要把 IP 地址返回给用户电脑，还要把这个对应关系保存在缓存中，以备下次别的用户查询时，可以直接返回结果，加快网络访问。

下面这张图很完美的解释了这一过程：

![](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhrveNibsGNAYwFicicqoHsYTcLQYBT4V7eR1ibVPA1bQhTTNkpntXX0gv5Jxu4wwzQviaYdy9tWl6VQ46Q/640?wx_fmt=png)

### —知识扩展—

#### 1. 什么是 DNS？

DNS（Domain Name System，域名系统），因特网上作为域名和 IP 地址相互映射的一个分布式数据库，能够使用户更方便的访问互联网，而不用去记住能够被机器直接读取的 IP 数串。通过主机名，最终得到该主机名对应的 IP 地址的过程叫做域名解析（或主机名解析）。  
　　  
通俗的讲，我们更习惯于记住一个网站的名字，比如 www.baidu.com, 而不是记住它的 ip 地址，比如：167.23.10.2。而计算机更擅长记住网站的 ip 地址，而不是像 www.baidu.com 等链接。因为，DNS 就相当于一个电话本，比如你要找 www.baidu.com 这个域名，那我翻一翻我的电话本，我就知道，哦，它的电话（ip）是 167.23.10.2。

#### 2.DNS 查询的两种方式：递归查询和迭代查询

**1、递归解析**

当局部 DNS 服务器自己不能回答客户机的 DNS 查询时，它就需要向其他 DNS 服务器进行查询。此时有两种方式，如图所示的是递归方式。局部 DNS 服务器自己负责向其他 DNS 服务器进行查询，一般是先向该域名的根域服务器查询，再由根域名服务器一级级向下查询。最后得到的查询结果返回给局部 DNS 服务器，再由局部 DNS 服务器返回给客户端。

![](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhrveNibsGNAYwFicicqoHsYTcLJEqhad3CiaBGT1Jn9qhtzoqfc54ukxa6nf0PvYLNRShdbcRg2HsLZoQ/640?wx_fmt=png)

**2、迭代解析**

当局部 DNS 服务器自己不能回答客户机的 DNS 查询时，也可以通过迭代查询的方式进行解析，如图所示。局部 DNS 服务器不是自己向其他 DNS 服务器进行查询，而是把能解析该域名的其他 DNS 服务器的 IP 地址返回给客户端 DNS 程序，客户端 DNS 程序再继续向这些 DNS 服务器进行查询，直到得到查询结果为止。也就是说，迭代解析只是帮你找到相关的服务器而已，而不会帮你去查。比如说：baidu.com 的服务器 ip 地址在 192.168.4.5 这里，你自己去查吧，本人比较忙，只能帮你到这里了。

![](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhrveNibsGNAYwFicicqoHsYTcLU4x8jadafZYeZIZf6c8sGjOWZwDgiadqeD6qZgYqH3ZqxZDAoCCesJA/640?wx_fmt=png)

#### 3.DNS 域名称空间的组织方式

我们在前面有说到根 DNS 服务器，域 DNS 服务器，这些都是 DNS 域名称空间的组织方式。按其功能命名空间中用来描述 DNS 域名称的五个类别的介绍详见下表中，以及与每个名称类型的示例

![](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhrveNibsGNAYwFicicqoHsYTcLTxJMSQTA3uF4ia1icUdVfrkOTrAOGuVnEFCDHibT7SUJyloYOm03YClDg/640?wx_fmt=png)

#### 4.DNS 负载均衡

当一个网站有足够多的用户的时候，假如每次请求的资源都位于同一台机器上面，那么这台机器随时可能会蹦掉。处理办法就是用 DNS 负载均衡技术，它的原理是在 DNS 服务器中为同一个主机名配置多个 IP 地址, 在应答 DNS 查询时, DNS 服务器对每个查询将以 DNS 文件中主机记录的 IP 地址按顺序返回不同的解析结果, 将客户端的访问引导到不同的机器上去, 使得不同的客户端访问不同的服务器, 从而达到负载均衡的目的｡例如可以根据每台机器的负载量，该机器离用户地理位置的距离等等。

3、浏览器向 web 服务器发送一个 HTTP 请求
--------------------------

拿到域名对应的 IP 地址之后，浏览器会以一个随机端口（1024 < 端口 < 65535）向服务器的 WEB 程序（常用的有 httpd,nginx 等）80 端口发起 TCP 的连接请求。这个连接请求到达服务器端后（这中间通过各种路由设备，局域网内除外），进入到网卡，然后是进入到内核的 TCP/IP 协议栈（用于识别该连接请求，解封包，一层一层的剥开），还有可能要经过 Netfilter 防火墙（属于内核的模块）的过滤，最终到达 WEB 程序，最终建立了 TCP/IP 的连接。  
　　  
TCP 连接如图所示: 　　

![](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhrveNibsGNAYwFicicqoHsYTcLVyuxia2r0Libkc63JFwPKUPJSwTfEQYYe737gklXP69vJ3CumZicsjOIA/640?wx_fmt=png)

建立了 [TCP 连接](http://mp.weixin.qq.com/s?__biz=MzU2MTI4MjI0MQ==&mid=2247484376&idx=1&sn=f5654ddc27e9998c98a646c89b178765&chksm=fc7a6e76cb0de760b5e0e22ac9911649da3a54f9608c75ca6087c6cf73e3564d0e0dc6280efd&scene=21#wechat_redirect)之后，发起一个 http 请求。一个典型的 http request header 一般需要包括请求的方法，例如 GET 或者 POST 等，不常用的还有 PUT 和 DELETE 、HEAD、OPTION 以及 TRACE 方法，一般的浏览器只能发起 GET 或者 POST 请求。  
　　  
客户端向服务器发起 http 请求的时候，会有一些请求信息，请求信息包含三个部分：

*   请求方法 URI 协议 / 版本
    
*   请求头 (Request Header)
    
*   请求正文
    

下面是一个完整的 HTTP 请求例子：

```
GET/sample.jspHTTP/1.1
Accept:image/gif.image/jpeg,*/*
Accept-Language:zh-cn
Connection:Keep-Alive
Host:localhost
User-Agent:Mozila/4.0(compatible;MSIE5.01;Window NT5.0)
Accept-Encoding:gzip,deflate

username=jinqiao&password=1234
```

> 注意：最后一个请求头之后是一个空行，发送回车符和换行符，通知服务器以下不再有请求头。

1. 请求的第一行是 “方法 URL 议 / 版本”：GET/sample.jsp HTTP/1.1  
2. 请求头 (Request Header)

请求头包含许多有关的客户端环境和请求正文的有用信息。例如，请求头可以声明浏览器所用的语言，请求正文的长度等。

```
Accept:image/gif.image/jpeg.*/*
Accept-Language:zh-cn
Connection:Keep-Alive
Host:localhost
User-Agent:Mozila/4.0(compatible:MSIE5.01:Windows NT5.0)
Accept-Encoding:gzip,deflate.
```

3. 请求正文  
请求头和请求正文之间是一个空行，这个行非常重要，它表示请求头已经结束，接下来的是请求正文。请求正文中可以包含客户提交的查询字符串信息：

```
username=jinqiao&password=1234
```

### — 知识扩展—

#### 1.TCP 三次握手

**第一次握手：**客户端 A 将标志位 SYN 置为 1, 随机产生一个值为 seq=J（J 的取值范围为 = 1234567）的数据包到服务器，客户端 A 进入 SYN_SENT 状态，等待服务端 B 确认；

**第二次握手：**服务端 B 收到数据包后由标志位 SYN=1 知道客户端 A 请求建立连接，服务端 B 将标志位 SYN 和 ACK 都置为 1，ack=J+1，随机产生一个值 seq=K，并将该数据包发送给客户端 A 以确认连接请求，服务端 B 进入 SYN_RCVD 状态。

**第三次握手：**客户端 A 收到确认后，检查 ack 是否为 J+1，ACK 是否为 1，如果正确则将标志位 ACK 置为 1，ack=K+1，并将该数据包发送给服务端 B，服务端 B 检查 ack 是否为 K+1，ACK 是否为 1，如果正确则连接建立成功，客户端 A 和服务端 B 进入 ESTABLISHED 状态，完成三次握手，随后客户端 A 与服务端 B 之间可以开始传输数据了。

如图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhrveNibsGNAYwFicicqoHsYTcLIiamPTviaaM2tn5UjaSR5TFn9y49l4maPLWFFvLuYxf2BUKWBB2cbuzw/640?wx_fmt=png)

#### 2. 为什需要三次握手？

《计算机网络》第四版中讲 “三次握手” 的目的是“为了防止已失效的连接请求报文段突然又传送到了服务端，因而产生错误”

书中的例子是这样的，“已失效的连接请求报文段” 的产生在这样一种情况下：client 发出的第一个连接请求报文段并没有丢失，而是在某个网络结点长时间的滞留了，以致延误到连接释放以后的某个时间才到达 server。本来这是一个早已失效的报文段。但 server 收到此失效的连接请求报文段后，就误认为是 client 再次发出的一个新的连接请求。于是就向 client 发出确认报文段，同意建立连接。

假设不采用 “三次握手”，那么只要 server 发出确认，新的连接就建立了。由于现在 client 并没有发出建立连接的请求，因此不会理睬 server 的确认，也不会向 server 发送数据。但 server 却以为新的运输连接已经建立，并一直等待 client 发来数据。这样，server 的很多资源就白白浪费掉了。采用“三次握手” 的办法可以防止上述现象发生。例如刚才那种情况，client 不会向 server 的确认发出确认。server 由于收不到确认，就知道 client 并没有要求建立连接。”。主要目的防止 server 端一直等待，浪费资源。

#### 3.TCP 四次挥手

**第一次挥手：**Client 发送一个 FIN，用来关闭 Client 到 Server 的数据传送，Client 进入 FIN_WAIT_1 状态。

**第二次挥手：**Server 收到 FIN 后，发送一个 ACK 给 Client，确认序号为收到序号 + 1（与 - SYN 相同，一个 FIN 占用一个序号），Server 进入 CLOSE_WAIT 状态。

**第三次挥手：**Server 发送一个 FIN，用来关闭 Server 到 Client 的数据传送，Server 进入 LAST_ACK 状态。

**第四次挥手：**Client 收到 FIN 后，Client 进入 TIME_WAIT 状态，接着发送一个 ACK 给 Server，确认序号为收到序号 + 1，Server 进入 CLOSED 状态，完成四次挥手。

![](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhrveNibsGNAYwFicicqoHsYTcLhmJK7IiakWRpiahgjPBWrTFI1qYohiapFrkCEkxzJodOEjkV8cv9Xb1JA/640?wx_fmt=png)

#### 4. 为什么建立连接是三次握手，而关闭连接却是四次挥手呢？

这是因为服务端在 LISTEN 状态下，收到建立连接请求的 SYN 报文后，把 ACK 和 SYN 放在一个报文里发送给客户端。而关闭连接时，当收到对方的 FIN 报文时，仅仅表示对方不再发送数据了但是还能接收数据，己方也未必全部数据都发送给对方了，所以己方可以立即 close，也可以发送一些数据给对方后，再发送 FIN 报文给对方来表示同意现在关闭连接，因此，己方 ACK 和 FIN 一般都会分开发送。

4、服务器的永久重定向响应
-------------

服务器给浏览器响应一个 301 永久重定向响应，这样浏览器就会访问`http://www.google.com/`而非`http://google.com/`。

为什么服务器一定要重定向而不是直接发送用户想看的网页内容呢？其中一个原因跟搜索引擎排名有关。如果一个页面有两个地址，就像 http://www.yy.com / 和 http://yy.com/，搜索引擎会认为它们是两个网站，结果造成每个搜索链接都减少从而降低排名。而搜索引擎知道 301 永久重定向是什么意思，这样就会把访问带 www 的和不带 www 的地址归到同一个网站排名下。还有就是用不同的地址会造成缓存友好性变差，当一个页面有好几个名字时，它可能会在缓存里出现好几次。

### —- 扩展知识—-

#### 1.301 和 302 的区别。

301 和 302 状态码都表示重定向，就是说浏览器在拿到服务器返回的这个状态码后会自动跳转到一个新的 URL 地址，这个地址可以从响应的 Location 首部中获取（用户看到的效果就是他输入的地址 A 瞬间变成了另一个地址 B）——这是它们的共同点。

他们的不同在于。301 表示旧地址 A 的资源已经被永久地移除了（这个资源不可访问了），搜索引擎在抓取新内容的同时也将旧的网址交换为重定向之后的网址；

302 表示旧地址 A 的资源还在（仍然可以访问），这个重定向只是临时地从旧地址 A 跳转到地址 B，搜索引擎会抓取新的内容而保存旧的网址。SEO302 好于 301

#### 2. 重定向原因：

*   网站调整（如改变网页目录结构）；
    
*   网页被移到一个新地址；
    
*   网页扩展名改变 (如应用需要把. php 改成. Html 或. shtml)。
    

这种情况下，如果不做重定向，则用户收藏夹或搜索引擎数据库中旧地址只能让访问客户得到一个 404 页面错误信息，访问流量白白丧失；再者某些注册了多个域名的网站，也需要通过重定向让访问这些域名的用户自动跳转到主站点等。

#### 3. 什么时候进行 301 或者 302 跳转呢？

当一个网站或者网页 24—48 小时内临时移动到一个新的位置，这时候就要进行 302 跳转，而使用 301 跳转的场景就是之前的网站因为某种原因需要移除掉，然后要到新的地址访问，是永久性的。

清晰明确而言：使用 301 跳转的大概场景如下：

*   域名到期不想续费（或者发现了更适合网站的域名），想换个域名。
    
*   在搜索引擎的搜索结果中出现了不带 www 的域名，而带 www 的域名却没有收录，这个时候可以用 301 重定向来告诉搜索引擎我们目标的域名是哪一个。
    
*   空间服务器不稳定，换空间的时候。
    

#### 5、浏览器跟踪重定向地址

现在浏览器知道了 `http://www.google.com/` 才是要访问的正确地址，所以它会发送另一个 http 请求。

6、服务器处理请求
---------

经过前面的重重步骤，我们终于将我们的 [http 请求](http://mp.weixin.qq.com/s?__biz=MzU2MTI4MjI0MQ==&mid=2247484360&idx=1&sn=2f0e3fd8caa4855bde7e16280cbbe5a4&chksm=fc7a6e66cb0de770f0c1235e2f9afd22364b6383f1a7f7188a01ee68e9b08baea539fdb237b7&scene=21#wechat_redirect)发送到了服务器这里，其实前面的重定向已经是到达服务器了，那么，服务器是如何处理我们的请求的呢？

后端从在固定的端口接收到 TCP 报文开始，它会对 TCP 连接进行处理，对 HTTP 协议进行解析，并按照报文格式进一步封装成 HTTP Request 对象，供上层使用。

一些大一点的网站会将你的请求到反向代理服务器中，因为当网站访问量非常大，网站越来越慢，一台服务器已经不够用了。于是将同一个应用部署在多台服务器上，将大量用户的请求分配给多台机器处理。

此时，客户端不是直接通过 HTTP 协议访问某网站应用服务器，而是先请求到 Nginx，Nginx 再请求应用服务器，然后将结果返回给客户端，这里 Nginx 的作用是反向代理服务器。同时也带来了一个好处，其中一台服务器万一挂了，只要还有其他服务器正常运行，就不会影响用户使用。

如图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhrveNibsGNAYwFicicqoHsYTcLCA1v8XWgHDOEURuS4suw0njG3DIahcVKPpRb0wt9PqhNmP9miaPA82w/640?wx_fmt=png)

通过 [Nginx 的反向代理](http://mp.weixin.qq.com/s?__biz=MzU2MTI4MjI0MQ==&mid=2247485395&idx=1&sn=e0263e2a8e2b4f84068817cb817e75cc&chksm=fc7a6a7dcb0de36bb2d3dc4b359887fcf524647af770a340a65ed4394f236345c34009501e11&scene=21#wechat_redirect)，我们到达了 web 服务器，服务端脚本处理我们的请求，访问我们的数据库，获取需要获取的内容等等，当然，这个过程涉及很多后端脚本的复杂操作。由于对这一块不熟，所以这一块只能介绍这么多了。

### —- 扩展阅读—-

#### 1. 什么是反向代理？

客户端本来可以直接通过 HTTP 协议访问某网站应用服务器，网站管理员可以在中间加上一个 Nginx，客户端请求 Nginx，Nginx 请求应用服务器，然后将结果返回给客户端，此时 Nginx 就是反向代理服务器。

7、服务器返回一个 HTTP 响应　
------------------

经过前面的 6 个步骤，服务器收到了我们的请求，也处理我们的请求，到这一步，它会把它的处理结果返回，也就是返回一个 HTPP 响应。

HTTP 响应与 HTTP 请求相似，HTTP 响应也由 3 个部分构成，分别是：

*   状态行
    
*   响应头 (Response Header)
    
*   响应正文
    

```
HTTP/1.1 200 OK
Date: Sat, 31 Dec 2005 23:59:59 GMT
Content-Type: text/html;charset=ISO-8859-1
Content-Length: 122

＜html＞
＜head＞
＜title＞http＜/title＞
＜/head＞
＜body＞
＜!-- body goes here --＞
＜/body＞
＜/html＞
```

#### 状态行：

状态行由协议版本、数字形式的状态代码、及相应的状态描述，各元素之间以空格分隔。

> 格式: `HTTP-Version Status-Code Reason-Phrase CRLF`  
> 例如: `HTTP/1.1 200 OK \r\n`

**协议版本：**是用 http1.0 还是其他版本

1xx：信息性状态码，表示服务器已接收了客户端请求，客户端可继续发送请求。

*   100 Continue
    
*   101 Switching Protocols
    
*   2xx：成功状态码，表示服务器已成功接收到请求并进行处理。
    

200 OK 表示客户端请求成功

*   204 No Content 成功，但不返回任何实体的主体部分
    
*   206 Partial Content 成功执行了一个范围（Range）请求
    

3xx：重定向状态码，表示服务器要求客户端重定向。

*   301 Moved Permanently 永久性重定向，响应报文的 Location 首部应该有该资源的新 URL
    
*   302 Found 临时性重定向，响应报文的 Location 首部给出的 URL 用来临时定位资源
    
*   303 See Other 请求的资源存在着另一个 URI，客户端应使用 GET 方法定向获取请求的资源
    
*   304 Not Modified 服务器内容没有更新，可以直接读取浏览器缓存
    
*   307 Temporary Redirect 临时重定向。与 302 Found 含义一样。302 禁止 POST 变换为 GET，但实际使用时并不一定，307 则更多浏览器可能会遵循这一标准，但也依赖于浏览器具体实现
    

4xx：客户端错误状态码，表示客户端的请求有非法内容。

*   400 Bad Request 表示客户端请求有语法错误，不能被服务器所理解
    
*   401 Unauthonzed 表示请求未经授权，该状态代码必须与 WWW-Authenticate 报头域一起使用
    
*   403 Forbidden 表示服务器收到请求，但是拒绝提供服务，通常会在响应正文中给出不提供服务的原因
    
*   404 Not Found 请求的资源不存在，例如，输入了错误的 URL
    

5xx：服务器错误状态码，表示服务器未能正常处理客户端的请求而出现意外错误。

*   500 Internel Server Error 表示服务器发生不可预期的错误，导致无法完成客户端的请求
    
*   503 Service Unavailable 表示服务器当前不能够处理客户端的请求，在一段时间之后，服务器可能会恢复正常
    

#### 响应头：

响应头部：由关键字 / 值对组成，每行一对，关键字和值用英文冒号”:” 分隔，典型的响应头有：

![](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhrveNibsGNAYwFicicqoHsYTcLGyWgxf93RLCvicKjITxfflogZlOYteOroTzbPJoCut3Nia7KibLs2YTDQ/640?wx_fmt=png)

#### 响应正文

包含着我们需要的一些具体信息，比如 cookie，html，image，后端返回的请求数据等等。这里需要注意，响应正文和响应头之间有一行空格，表示响应头的信息到空格为止，下图是 fiddler 抓到的请求正文，红色框中的：响应正文：

![](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhrveNibsGNAYwFicicqoHsYTcL6iaD4IFZnDpuvDzQW21aqOgJvibTXHf5JHibAEbMcHZPY99iaGGUfMNBHw/640?wx_fmt=png)

8、浏览器显示 HTML
------------

在浏览器没有完整接受全部 HTML 文档时，它就已经开始显示这个页面了，浏览器是如何把页面呈现在屏幕上的呢？不同浏览器可能解析的过程不太一样，这里我们只介绍 webkit 的渲染过程，下图对应的就是 WebKit 渲染的过程，这个过程包括：

> 解析 html 以构建 dom 树 -> 构建 render 树 -> 布局 render 树 -> 绘制 render 树

**浏览器在解析 html 文件时，会” 自上而下 “加载，并在加载过程中进行解析渲染**。在解析过程中，如果遇到请求外部资源时，如图片、外链的 CSS、iconfont 等，请求过程是异步的，并不会影响 html 文档进行加载。

解析过程中，浏览器**首先会解析 HTML 文件构建 DOM 树，然后解析 CSS 文件构建渲染树，等到渲染树构建完成后，浏览器开始布局渲染树并将其绘制到屏幕上**。这个过程比较复杂，涉及到两个概念: reflow(回流) 和 repain(重绘)。

DOM 节点中的各个元素都是以盒模型的形式存在，这些都需要浏览器去计算其位置和大小等，这个过程称为 relow; 当盒模型的位置, 大小以及其他属性，如颜色, 字体, 等确定下来之后，浏览器便开始绘制内容，这个过程称为 repain。

页面在首次加载时必然会经历 reflow 和 repain。reflow 和 repain 过程是非常消耗性能的，尤其是在移动设备上，它会破坏用户体验，有时会造成页面卡顿。所以**我们应该尽可能少的减少 reflow 和 repain**。

当文档加载过程中遇到 js 文件，html 文档会挂起渲染（加载解析渲染同步）的线程，不仅要等待文档中 js 文件加载完毕，还要等待解析执行完毕，才可以恢复 html 文档的渲染线程。因为 JS 有可能会修改 DOM，最为经典的 document.write，这意味着，在 JS 执行完成前，后续所有资源的下载可能是没有必要的，这是 js 阻塞后续资源下载的根本原因。所以我明平时的代码中，js 是放在 html 文档末尾的。

**JS 的解析是由浏览器中的 JS 解析引擎完成的**，比如谷歌的是 V8。JS 是单线程运行，也就是说，在同一个时间内只能做一件事，所有的任务都需要排队，前一个任务结束，后一个任务才能开始。但是又存在某些任务比较耗时，如 IO 读写等，所以需要一种机制可以先执行排在后面的任务，这就是：同步任务 (synchronous) 和异步任务(asynchronous)。

**JS 的执行机制就可以看做是一个主线程加上一个任务队列 (task queue)**。同步任务就是放在主线程上执行的任务，异步任务是放在任务队列中的任务。所有的同步任务在主线程上执行，形成一个执行栈; 异步任务有了运行结果就会在任务队列中放置一个事件；脚本运行时先依次运行执行栈，然后会从任务队列里提取事件，运行任务队列中的任务，这个过程是不断重复的，所以又叫做事件循环 (Event loop)。具体的过程可以看这篇文章：

> http://www.cnblogs.com/xianyulaodi/p/6414805.html

9、浏览器发送请求获取嵌入在 HTML 中的资源（如图片、音频、视频、CSS、JS 等等）
---------------------------------------------

其实这个步骤可以并列在步骤 8 中，在浏览器显示 HTML 时，它会注意到需要获取其他地址内容的标签。这时，浏览器会发送一个获取请求来重新获得这些文件。比如我要获取外图片，CSS，JS 文件等，类似于下面的链接：

> 图片：http://static.ak.fbcdn.net/rsrc.php/z12E0/hash/8q2anwu7.gif  
> CSS 式样表：http://static.ak.fbcdn.net/rsrc.php/z448Z/hash/2plh8s4n.css  
> JavaScript 文件：http://static.ak.fbcdn.net/rsrc.php/zEMOA/hash/c8yzb6ub.js

这些地址都要经历一个和 HTML 读取类似的过程。所以浏览器会在 DNS 中查找这些域名，发送请求，重定向等等…

不像动态页面，静态文件会允许浏览器对其进行缓存。有的文件可能会不需要与服务器通讯，而从缓存中直接读取，或者可以放到 CDN 中

#### 参考文献：

> https://segmentfault.com/a/1190000006879700  
> http://igoro.com/archive/what-really-happens-when-you-navigate-to-a-url/  
> http://zrj.me/archives/589