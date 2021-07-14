> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650517363&idx=2&sn=ffad46cc88a5b6951ae21d5f7e1849e6&chksm=83baca97b4cd438161e58efc18043f7390e5ebf3fc38808078fb6784e25d39ba98649b7a666c&mpshare=1&scene=1&srcid=0713eLGYMhnQup2ul7WoGIUv&sharer_sharetime=1626141916177&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> **文章来****源****：**Hack 学习呀****

*   网站的关于页面／网站地图
    
*   whois 反查
    
*   一些网站里面的跳转请求（也可以关注一下 app）
    
*   还有就是百度，有些会在 title 和 copyright 信息里面出现该公司的信息
    
*   网站 html 源码：主要就是一些图片、js、css 等，也会出现一些域名
    
*   apk 反编译源码里面
    

*   subdomain lijiejie 的子域名收集工具（个人觉得挺好用的）；
    
*   layer：这个工具也不错；
    

搜集 mail：site:xxx.com intext:@xxx.com／intext:@xxx.com

**3、敏感信息收集**  

*   github 源代码：网上有工具（https://github.com/repoog/GitPrey）
    
*   svn 信息泄漏：这个只能用扫描器了
    
*   敏感文件：比如数据库配置文件啦（有案例的）、网站源码啊、数据库备份文件等等
    
*   敏感目录：网站后台目录／一些登录地址／一些接口目录
    
*   email：邮箱命名规则、公司是否具有邮箱默认密码（这个可以采取社工，毕竟我司默认密码就很弱鸡）。
    
*   员工号：很多 oa、um、sso 系统都是采用员工号登录的，所以知道员工号的规则很多时候能帮助我们进行撞库。
    
*   商家信息：如果是一些具有商家系统的，能收集到一些商家账户（自己搞去，可以注册，注册资料请百度）就可以进入很多系统来测试了。
    

**4、小结一下**

*   通过搜索引擎获取系统管理页面，直接越权访问；（说好的没有详细）
    
*   通过 github 直接找到管理后台账号密码；
    
*   通过目录／文件扫描直接得到系统信息（ip、管理员账号密码）连入服务器；
    
*   当然也有很多通过信息收集得到一些东西结合其他手段；
    

**二、漏洞挖掘的中期–信息处理**

对于第三节提到的那些信息收集技术，我们不能收集完了就完了，一定好好整理，会对后期渗透有很大的帮助。这里说一下具体怎么整理。

*   哪些网站功能类似；
    
*   哪些网站可能使用的同一模版；
    
*   哪些网站有 waf（这个一般在 url 中标明就好）；
    
*   哪些网站能登录（注册的账号也一定要记住，最好可以准备两个手机号，两个邮箱方便注册）；
    
*   哪些网站暴露过哪些类型的漏洞（这个只能去乌云上面找）；
    
*   网站目前有哪些功能（这个稍微关注一下网站公告，看最近是否会有业务更迭）；
    

**2、漏洞整理**

第二个就是通过漏洞得到的一些数据：

*   订单信息；遍历、注入
    
*   用户信息：这个可以通过撞库获取、任意密码重置获取、注入
    
*   数据库用户名密码：注入、配置泄漏
    

**三、漏洞挖掘的后期–漏洞挖掘**

首先我们需要对一个网站／app 有一个了解要知道它的功能点有哪些（后期我会更新一个 checklist 介绍一下哪些功能会对应什么样的漏洞）。

*   **首先我们要选择：**
    

1.  筛选涉及查询（是否可以 SQL 注入）
    
2.  加入购物车：商品数量是否可以为负
    

*   **询问商家：**
    

1.  跳转客服系统，跳转 url 中是否含有用户参数
    
2.  xss 打客服 cookie
    
3.  钓鱼 + 社工
    

*   **下单：**
    

1.  填地址，涉及插入（注入）、xss
    
2.  修改单价
    
3.  修改总额（这里说明一下修改总额：情况 1，就是我们可能会遇到可以使用优惠卷的情况，比如我们买了 100 的东西只能使用 5 块的优惠价，但是我有一张 50 的优惠卷是否可以使用；情况 2，打折我们是否可以修改打折的折扣；情况 3，我们是否可以修改运费，将运费改为负数；情况 n）
    
4.  备注：xss，sql 注入
    

*   **电子票据：**  
    

1.  会写抬头
    

*   **支付：**
    

1.  传输过程中是否可以修改，如果是扫描二维码支付，我们可以分析一下二维码中的请求 url 看是否可以修改以后重新生成二维码（这里不讨论具体的支付了，因为微信和支付宝都很安全）
    

*   **订单完成：**  
    

1.  是否可以遍历订单
    

*   **评价：**
    

1.  注入、上传图片、xss
    

*   **退货…………**
    

大家可以无限延伸，这里只是抛砖引玉。

*   大家会觉得就是经验，玩多了就自然会了，教不了什么；
    
*   这种分享经验特别不好写，到现在也不知道我写了什么，其实都是一个思路点；
    
*   懒，不愿意…… 肯定都有一定的原因；
    

**建议：**

**四、安全漏洞相关的概念**

我们经常听到漏洞这个概念，可什么是安全漏洞？想给它一个清晰完整的定义其实是非常困难的。如果你去搜索一下对于漏洞的定义，基本上会发现高大上的学术界和讲求实用的工业界各有各的说法，漏洞相关的各种角色，比如研究者、厂商、用户，对漏洞的认识也是非常不一致的。

这是一个从研究者角度的偏狭义的定义，影响的主体范围限定在了信息系统中，以尽量不把我们所不熟悉的对象扯进来。

定义对安全的影响也只涉及狭义信息安全的三方面：机密性、完整性和可用性。漏洞造成的敏感信息泄露导致机密性的破坏；造成数据库中的信息被非法篡改导致完整性的破坏；造成服务器进程的崩溃导致可用性的丧失。漏洞也可能同时导致多个安全属性的破坏。

漏洞与 Bug 并不等同，他们之间的关系基本可以描述为：大部分的 Bug 影响功能性，并不涉及安全性，也就不构成漏洞；大部分的漏洞来源于 Bug，但并不是全部，它们之间只是有一个很大的交集。可以用如下这个图来展示它们的关系：

各个漏洞数据库和索引收录了大量已知的安全漏洞，下表是一个主流漏洞库的数量的大致估计，漏洞一般最早从 20 世纪 90 年代开始：

事实上，即便把未知的漏洞排除在外，只要订了若干漏洞相关的邮件列表就会知道：并不是所有漏洞数据库都会收录，就算把上面的所列的数据库中的所有条目加起来去重以后也只是收录了一部分的已知漏洞而已，实际的已知漏洞数比总收录的要高得多。

和其他事物一样，安全漏洞具有多方面的属性，也就可以从多个维度对其进行分类，重点关注基于技术的维度。注意，下面提到的所有分类并不是在数学意义上严格的，也就是说并不保证同一抽象层次、穷举和互斥，而是极其简化的出于实用为目的分类。

**本地漏洞**

实例：

**远程漏洞**

实例：

**基于威胁类型的分类**  

可以导致劫持程序执行流程，转向执行攻击者指定的任意指令或命令，控制应用系统或操作系统。威胁最大，同时影响系统的机密性、完整性，甚至在需要的时候可以影响可用性。主要来源：内存破坏类、CGI 类漏洞

可以导致劫持程序访问预期外的资源并泄露给攻击者，影响系统的机密性。

**拒绝服务**

**主要来源：内存破坏类、意外处理错误处理类漏洞。**

基于漏洞成因技术的分类相比上述的两种维度要复杂得多，对于目前我所见过的漏洞大致归纳为以下几类：

*   **内存破坏类**
    
*   **逻辑错误类**
    
*   **输入验证类**
    
*   **设计错误类**
    
*   **配置错误类**
    

**内存破坏类**

*   **栈缓冲区溢出**
    
*   **堆缓冲区溢出**
    
*   **静态数据区溢出**
    
*   **格式串问题**
    
*   **越界内存访问**
    
*   **释放后重用**
    
*   **二次释放**
    

**栈缓冲区溢出**

![](https://mmbiz.qpic.cn/mmbiz/9YBWvqSs4WcxcmfcM5slAxKsXq7QvxJfGIgVtvm8NlyTIKcdpGL7dMdiaVFE5yFN22Q4BJvhgspDbiaTia9rzib3Gg/640?wx_fmt=jpeg)

实例：

*   **暴风影音 stormtray 进程远程栈缓冲区溢出漏洞**
    

![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W17QhOAJ3XGcjtu5wlzwibHtdUicuibLWcfoD6icqoz3yvzSMCeKFDUMFWicA/640?wx_fmt=jpeg)  

*   **Sun Solaris snoop(1M) 工具远程指令执行漏洞**（ CVE-2008-0964 ）
    

![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1FVUOyvsCfNSRibIQ9xbt3bicztfgiamKib3KJRDkvwA6UeFeu45OicGicgow/640?wx_fmt=jpeg)  

*   **Novell eDirectory HTTPSTK Web 服务器栈溢出漏洞无长度检查的 memcpy 调用。**
    

*   **FlashGet FTP PWD 命令超长响应栈溢出漏洞**
    

![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1URprZCOL5mQibBhuDSa1WtKpOFuNl3ic9V9zS5e88tcfEL7zOtTRKTog/640?wx_fmt=jpeg)  

*   **Imatix Xitami If-Modified-Since 头远程栈溢出漏洞。极其危险的 sscanf 类调用。**
    
      
    ![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1hrAsibYfAWqU2vd4605w8zaygR8nj0HccA5Zs3BKiaF1mnG73Jtv09Bw/640?wx_fmt=jpeg)  
    
*   **Borland StarTeam Multicast 服务用户请求解析远程栈溢出漏洞（ CVE-2008-0311 ）**
    

![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1DXqAVK7pribgia8mOoXeEXH2VRAtXQhbtu6n8Emia6ibSib5qJTNY5akGzQ/640?wx_fmt=jpeg)  

*   **Microsoft DirectShow MPEG2TuneRequest 溢出漏洞（ CVE-2008-0015 ）手抖，缓冲区的指针被当做缓冲区本身被数据覆盖溢出。**
    

**堆缓冲区溢出**

实例：

*   **HP OpenView NNM Accept-Language HTTP 头堆溢出漏洞（ CVE-2009-0921）典型的先分配后使用的堆溢出问题。**
    
    ![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1icHTW8fLibefsYZkxFD89hszJiaN8vwp5y2G6mllJwmJTNaibtxxmAibjHw/640?wx_fmt=jpeg)  
    
*   **PHP (phar extension) 堆溢出漏洞堆溢出特有的溢出样式：由于整数溢出引发 Malloc 小缓冲区从而最终导致堆溢出。**
    
    ![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1w4a5RG10T34zyr5plLuXWwrjegYwNbDaSxuFjl1WdARPjz2MhAN0Xg/640?wx_fmt=jpeg)  
    

发生在静态数据区 BSS 段中的溢出，非常少见的溢出类型。

*   **Symantec pcAnyWhere awhost32 远程代码执行漏洞（CVE-2011-3478）**
    
    ![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1z3Nf3jy6BLNpgFDUhMvAADVrQQ1a8tGsNsD9qCWxtkYibVdibOhwcGCg/640?wx_fmt=jpeg)  
    

*   **Qualcomm Qpopper 2.53 格式串处理远程溢出漏洞（CVE-2000-0442）**
    
    ![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1lSFSRZibia4cicdj1KO2VnAa7ib0uBiaMV7aOictgiaXweolfYt8iba1aRNVCA/640?wx_fmt=jpeg)  
    

**越界内存访问**

程序盲目信任来自通信对方传递的数据，并以此作为内存访问的索引，畸形的数值导致越界的内存访问，造成内存破坏或信息泄露。

*   OpenSSL TLS 心跳扩展协议包远程信息泄露漏洞 (CVE-2014-0160) 漏洞是由于进程不加检查地使用通信对端提供的数据区长度值，按指定的长度读取内存返回，导致越界访问到大块的预期以外的内存数据并返回，泄露包括用户名、口令、
    

![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1smibrciatH1RWRxQXYWeRSckq25Q3lPTobib1c24iahvSfibvicODctkuFVQ/640?wx_fmt=jpeg)  

**释放后重用**

这是目前最主流最具威胁的客户端（特别是浏览器）漏洞类型，大多数被发现的利用 0day 漏洞进行的水坑攻击也几乎都是这种类型，每个月各大浏览器厂商都在修复大量的此类漏洞。技术上说，此类漏洞大多来源于对象的引用计数操作不平衡，导致对象被非预期地释放后重用，进程在后续操作那些已经被污染的对象时执行攻击者的指令。与上述几类内存破坏类漏洞的不同之处在于，此类漏洞的触发基于对象的操作异常，而非基于数据的畸形异常（通常是不是符合协议要求的超长或畸形字段值），一般基于协议合规性的异常检测不再能起作用，检测上构成极大的挑战。

实例：

*   **Microsoft IE 非法事件操作内存破坏漏洞**（CVE-2010-0249）
    

著名的 Aurora 攻击，涉嫌入侵包括 Google 在内的许多大互联网公司的行动，就使用了这个 CVE-2010-0249 这个典型的释放后重用漏洞。

![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1icsmx5AB1XbwTT6IrkMz6wiaUgfX9pKQTxMrhokPZ9icHBs3GQ8cKIgfA/640?wx_fmt=jpeg)  

**二次释放**

一般来源于代码中涉及内存使用和释放的操作逻辑，导致同一个堆缓冲区可以被反复地释放，最终导致的后果与操作系统堆管理的实现方式相关，很可能实现执行任意指令。

实例：

*   **CVS 远程非法目录请求导致堆破坏漏洞**（ CVE-2003-0015）
    

![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1zMllXx0YjASA92I6gVia43W5V8MctFhzrQXibtkeF72tHWZ7SRgSMoUw/640?wx_fmt=jpeg)  

**逻辑错误类**

涉及安全检查的实现逻辑上存在的问题，导致设计的安全机制被绕过。

实例：

*   **Real VNC 4.1.1 验证绕过漏洞**（ CVE-2006-2369 ）
    

漏洞允许客户端指定服务端并不声明支持的验证类型，服务端的验证交互代码存在逻辑问题。

*   **Android 应用内购买验证绕过漏洞**
    

Google Play 的应用内购买机制的实现上存在的漏洞，在用户在 Android 应用内购买某些数字资产时会从 Play 市场获取是否已经付费的验证数据，对这块数据的解析验证的代码存在逻辑问题，导致攻击者可以绕过验证不用真的付费就能买到东西。验证相关的代码如下：

![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W125iam3OCvGicGw56zNat9vdwWxrILZpFx45wCIXsMN7tbMV7mnISF4yg/640?wx_fmt=jpeg)  

代码会先检查回来的数据签名是否为空，不空的话检查签名是否正确，如果不对返回失败。问题在于如果签名是空的话并没有对应的 else 逻辑分支来处理，会直接执行最下面的 return true 操作，导致的结果是只要返回的消息中签名为空就会返回验证通过。

**输入验证类**

漏洞来源都是由于对来自用户输入没有做充分的检查过滤就用于后续操作，绝大部分的 CGI 漏洞属于此类。所能导致的后果，经常看到且威胁较大的有以下几类：

*   **SQL 注入**
    
*   **跨站脚本执行**
    
*   **远程或本地文件包含**
    
*   **命令注入**
    
*   **目录遍历**
    

**SQL 注入**

Web 应用对来自用户的输入数据未做充分检查过滤，就用于构造访问后台数据库的 SQL 命令，导致执行非预期的 SQL 操作，最终导致数据泄露或数据库破坏。

实例：

*   **一个网站 Web 应用的数值参数的 SQL 注入漏洞**。
    

![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1orEhhGRXpqU2VDHAMgP1ukgrv9vq9qNzKibVnJ1MZwVPTzmMGdF8iarQ/640?wx_fmt=jpeg)  

**跨站脚本执行（XSS）**

Web 应用对来自用户的输入数据未做充分检查过滤，用于构造返回给用户浏览器的回应数据，导致在用户浏览器中执行任意脚本代码。

实例：

YouTube 上的一个存储式 XSS 漏洞。

![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1NoLTUxWYR1CZqmpPEMficnmHfq5fTjyNqs1n9CfGNy6KYyZicduq2uFQ/640?wx_fmt=jpeg)  

**远程或本地文件包含**

PHP 语言支持在 URL 中包含一个远程服务器上的文件执行其中的代码，这一特性在编码不安全的 Web 应用中很容易被滥用。如果程序员在使用来自客户端的 URL 参数时没有充分地检查过滤，攻击者可以让其包含一个他所控制的服务器上的文件执行其中的代码，导致远程文件包含命令执行。

实例：

*   一个远程文件包含利用的例子
    

![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1Y8Uc9A2NEoEUiaxmLNVynEvbibh8WEB1AMnricicJO21TSLt1BLQZdvcicQ/640?wx_fmt=jpeg)  
![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1VqHgpd8EIkEmIEfVJqEy6gqJCSNTFygt5TeiaTHNoI08WlohaSCyviaA/640?wx_fmt=jpeg)  

如果 Web 应用支持在 URL 参数中指定服务器上的一个文件执行一些处理，对来自客户端 URL 数据及本地资源的访问许可如果未做充分的检查，攻击者可能通过简单的目录遍历串使应用把 Web 主目录以外的系统目录下的文件包含进来，很可能导致信息泄露：

实例：

*   **一个网站存在的本地文件包含的漏洞**
    
    ![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1w5w66ZUDOJXvd6dajDJsgqT4Q84ZlO1oU5mP98vicITJO4bmjerYH1g/640?wx_fmt=jpeg)  
    

**命令注入**

涉及系统命令调用和执行的函数在接收用户的参数输入时未做检查过滤，或者攻击者可以通过编码及其他替换手段绕过安全限制注入命令串，导致执行攻击指定的命令。

实例：

*   **AWStats 6.1 及以下版本 configdir 变量远程执行命令漏洞（ CVE-2005-0116 ）**
    

典型的由于 Perl 语言对文件名特性的支持加入未充分检查用户输入的问题，导致的命令注入漏洞，awstats.pl 的 1082 行：

if(open(CONFIG,”$searchdir$PROG.$SiteConfig.conf”)) 。

![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1HLv7OGZDq5YVhh2BaoBs6SoD7vt0v6kQicJiciaOibO3icYmlTdpty5nRMQ/640?wx_fmt=jpeg)  

**目录遍历**

涉及系统用于生成访问文件路径用户输入数据时未做检查过滤，并且对最终的文件绝对路径的合法性检查存在问题，导致访问允许位置以外的文件。多见于 CGI 类应用，其他服务类型也可能存在此类漏洞。

实例：

*   **Novell Sentinel Log Manager “filename” 参数目录遍历漏洞（ CVE-2011-5028 ）**
    

http://www.example.com/novelllogmanager/FileDownload?filename=/opt/novell/sentinel_log_mgr/3rdparty/tomcat/temp/../../../../../../etc/passwd

*   HP Data Protector Media Operations DBServer.exe 目录遍历漏洞
    

在 HP Data protecetor Media Operations 的客户端连接服务端时，通过私访有的通信协议，客户端会首先检查 [系统分区]:\Documents and Settings\[用户名]\Application Data 下面是否有相应的资源 (如插件等)，如果没有，则会向服务器请求需要的文件，服务器没有验证请求的文件名的合法性，而且这个过程不需要任何验证，攻击者精心构造文件名，可以读取服务端安装目录所在分区的任意文件。

![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1NX889TBibnRydgjLKtciaUBBmEVIpjfZNlBLgIQFXAibYdWB3PiaQmhdKA/640?wx_fmt=jpeg)  

*   **RHINOSOFT SERV-U FTP SERVER 远程目录遍历漏洞**
    

![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1tjYWic78wWibASQktlib7ibPC5ChGRM9ianldcAgxv95Sz6SeXHCetuwkpw/640?wx_fmt=jpeg)  

*   **Caucho Resin 远程目录遍历漏洞**
    

![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W11ZnkyPz7G0V2N4L6V1y5rawZIts8H66kDJcyYDuich4B3U2TcibpI4xA/640?wx_fmt=jpeg)  

**设计错误类**

系统设计上对安全机制的考虑不足导致的在设计阶段就已经引入的安全漏洞。

实例：

*   **LM HASH 算法脆弱性**
    
    ![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1pQW7h2Hhh3ukGYkDWyHms01ibicaKHnL3aPpoPkBAyeXgoF9TzhutDicA/640?wx_fmt=jpeg)  
    

这个算法至少存在以下 3 方面的弱点：

1、口令转换为大写极大地缩小了密钥空间。

2、切分出的两组数据分别是独立加密的，暴力破解时可以完全独立并行。

3、不足 7 字节的口令加密后得到的结果后半部分都是一样的固定串，由此很容易判定口令长度。

这些算法上的弱点导致攻击者得到口令 HASH 后可以非常容易地暴力猜测出等价的明文口令。Microsoft Windows 图形渲染引擎 WMF 格式代码执行漏洞 (MS06-001) (CVE-2005-4560) 如果一个 WMF 文件的 StandardMetaRecord 中，Function 被设置为 META_ESCAPE 而 Parameters[0] 等于 SETABORTPROC，PlayMetaFileRecord()就会调用 Escape()函数，Escape()调用 SetAbortProc()将自己的第四形参设置为一个回调函数，把图像文件中包含的一个数据块象 Shellcode 那样执行。此漏洞从 Windows 3.1 一直影响到 2003，攻击者只要让用户处理恶意的 WMF 文件（通过挂马或邮件）在用户系统上执行任意指令，漏洞实在是太好用影响面太大了，以至有人认为这是一个故意留的后门，其实影响设计的功能是处理打印任务的取消，功能已经被废弃，但废弃的代码并没有移除而导致问题。

*   **搜狐邮箱密码找回功能**
    

密码找回功能在要求用户提供找回密码需要的问题答案时，在返回给用户的页面中就已经包含了答案，只要通过查看页面源码就能看到，使这个找回密码功能的安全验证完全形同虚设，攻击者由此可以控制任意邮箱。之所以这么设计，可能就是为了尽可能地少对数据库的查询，而把用户帐号安全根本不放在心上。

![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1t5vkibMttW4uJqZ05YqvlB3Tl5qCkytSPW2fuTiayb2RbicEKtvtibK6fQ/640?wx_fmt=jpeg)  

*   **紫光输入法用户验证绕过漏洞**
    

这是类似于 2000 年微软输入法漏洞的例子，通过访问输入法设置的某些功能绕过操作系统的用户验证执行某些操作。

![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1k0ZE2ibUuicHpxczof8xoXS4rrnDArMCoSHBXibsLjicldZOs3FWIuMR4A/640?wx_fmt=jpeg)  

**配置错误类**

系统运维过程中默认不安全的配置状态，大多涉及访问验证的方面。

实例：

*   **JBoss 企业应用平台非授权访问漏洞**（ CVE-2010-0738 ）
    

对控制台访问接口的访问控制默认配置只禁止了 HTTP 的两个主要请求方法 GET 和 POST，事实上 HTTP 还支持其他的访问方法，比如 HEAD，虽然无法得到的请求返回的结果，但是提交的命令还是可以正常执行的。

![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1bwbxLUSyic0h4aqruA3Wca62fGvhibv78LqN8d6QibSGuYBDIEMOZDVkQ/640?wx_fmt=jpeg)  

*   **Apache Tomcat 远程目录信息泄露漏洞**
    

Tomcat 的默认配置允许列某些目录的文件列表。

![](https://mmbiz.qpic.cn/mmbiz/VcRPEU1K2ocAVSFfm7XcjYa15M3Iic7W1zdWuuXER9ryEm6CCmYpuwkDHaqzsz0vArFJxfGrkwejqyK0o0vZD7g/640?wx_fmt=jpeg)  

**各种眼花缭乱的安全漏洞其实体现的是人类在做事的各种环节上犯过的错误，通过改进工具流程制度可以得到某些种程度的解决，但有些涉及人性非常不容易解决，而且随着信息系统的日趋复杂，我们可以看到更多的新类型漏洞，这个领域永远都有的玩。**

**挖洞小技巧**

**1. 敢于怀疑漏洞的存在，然后再去验证它**

**2. 从开发设计的角度去看，想开发所想**

**3. 了解业务和情报收集**

**4. 坚持和努力就会有所回报**

**5. 多看师傅们的文章，学会师傅们的骚操作和思路**

**6. 运气也很重要，但是努力的人运气一般不会太差哦**