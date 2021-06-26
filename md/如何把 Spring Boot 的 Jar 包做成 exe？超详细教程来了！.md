> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI4OTA3NDQ0Nw==&mid=2455551273&idx=2&sn=48427718674d851bc7596b48bd590989&chksm=fb9ca149cceb285fe2929464618b3e3cd9755e9d1a369dce2bb511316f5c217cadd68ad8d37f&mpshare=1&scene=1&srcid=0626We91FoygH75t7owQCHpo&sharer_sharetime=1624717164630&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

作者 | 如漩涡  

准备工作：

1.  一个 jar 包，没有 bug 能正常启动的 jar 包
    
2.  exe4j，一个将 jar 转换成 exe 的工具，链接：https://pan.baidu.com/s/1J30uUMJcYnqWCJSr6gkM5w，提取码：6esr，注册码：L-g782dn2d-1f1yqxx1rv1sqd
    
3.  inno setup，一个将依赖和 exe 一起打成一个安装程序的工具，链接：https://pan.baidu.com/s/1DgFo1ceM_8Bqx_b-veibbQ，提取码：g9jd
    

以我为例子，我将 jar 包放在了桌面

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHJ0YfXHu9lphHTZqLe4zw9yQcOhQnNY04OJdYIykegpNvQ7aQb9tyPA/640?wx_fmt=png)

直接下一步进入界面，选择 JAVA 转 EXE

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHd6qOkjbod071qMyADKoAhJH6NnatBegCPcRPIzMglWndzXuSVicC4Mw/640?wx_fmt=png)

然后点下一步，输入名称和输出路径

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OH7WDXST9BTluNVqcUHbzjBguxibg2G6zAm2NgFI6cpkXeLdwZsbkOr4Q/640?wx_fmt=png)

继续点击下一步，选择启动模式

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHJficQAicELraqznUydqZdkVcZhLUv2FlLSehVB21IUOloueehKicCD66Q/640?wx_fmt=png)

下方有个选项，需要设置打包后的程序兼容 32 和 64 位系统

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OH35lMVOxUGQ6XR54rWWoFaStukV4dKibmicFRBZfAA3xiculybO4mUoahw/640?wx_fmt=png)

进来后勾选上

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHdd3mWofHq8DIfbywQQOTLaqxIuqyM1JkFIj1H0cDjfEKtibQLUvWAjg/640?wx_fmt=png)

然后一直下一步，一直出现如下界面，开始选择 jar 包以及配置

在 VM 参数配置的地方加上：-Dfile.encoding=utf-8

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHLZO5n8PGjBcSn1IOibmJXeKcQUibFzuBnYWJcV7Bx9rYb3yOXiaRdaURw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHfjnMBbctvicwzUauQrXyxTp6QUtA40Kr08sicsPKLGtqEFibBoK4hdKwA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OH8hqmGw7icP1g78pTkw7zVtYBrrRgFYiaicv4Pia8uAt25arV4kOr70HDxw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHJwicUj6OecV9VibFF0icgOAHhBJZ42oAG4MqvF1VZEY2RbPtyrq1NSf4A/640?wx_fmt=png)

点击下一步，配置 JRE

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHVUq5ej9Iiaic6NuSmLspiadtPQMepRrjH9kI6Be0Rh1nwc88aGjT88VuQ/640?wx_fmt=png)

下拉框点击后进入如下界面

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OH7Df8ibEics9Siaup2asq245H71iaERwUicZocjIdPntyjjD767gP5JqZgOA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHspZ4CEiaicjktxJmxZE1bmNRniaJ7YFUOhzZ4m2jL7xzmXdVUKM2dUJZw/640?wx_fmt=png)

照着这个样子写的目的是，最终会把本地 jre 目录和 exe 一起打包，让 exe 文件自己去根据路径去查找一起打包的 jre，可不用再安装 jdk

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHib5zH6WMC5zxCgYibDIVuo2amAibbxk752UwKkLObBUg1EoBplI5NP5sA/640?wx_fmt=png)

接着下一步，选择 Client VM

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHuPZ7MBB1sAzZibc3ThaZMfQUouLwoLcn6sBiaJQLtd0OtNmVic5BU69Fg/640?wx_fmt=png)

然后一直下一步，最终出现如下界面

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHWglGz6xJ5WyvyvrapYj7YbxBpicN4SgEMlVjcMdnrHzAOyT3C6lIMDA/640?wx_fmt=png)

这个时候你会发现桌面多了一个 demo.exe 文件，这个时候先别着急点开，接下来就是将 jre 和 exe 文件再打个包合并，达到在没有 jdk 电脑环境下也能运行

打开 inno setup，左上角 File - New

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHn7iaCKic5frib18hMUX9EtWlYIufXMa6zNvmumXPYicN4oecDfB54cAEbA/640?wx_fmt=png)

直接点下一步，填写配置，应用名称，版本等，随意

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHXhOmkZ8bQEmXZIziadXLLb3WMuePGmicw5Dia22ynSHMBPZzT5yxlxeRw/640?wx_fmt=png)

然后点击下一步，这个地方默认就行，直接下一步

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OH9qQBPUBZEDekGzSOGLpOw2icO9hkqyF0h2UxZE3ib9zts9jA1T6ZKIVQ/640?wx_fmt=png)

接着选择生成好的 exe 文件

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHkmfAicPQfkOymIn6P5yesBhrLtd3FkGcRnSANt68tPzJQefAWrpfOcw/640?wx_fmt=png)

然后下一步，进入这个界面保持默认，直接下一步

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHj7mFgUdFKZbUfm4RDP8Nu1zagl4pGo1dhsDSiclR6xue3RicMak9q8hg/640?wx_fmt=png)

依旧下一步，不用管

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHuCqGDrwjbDFeYuWic4OBwUaBDr7icDVZygLTn5z6JVA68ibByYWKHGmdw/640?wx_fmt=png)

继续下一步，这里是选择语言

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OH6D92Gibia6hx6a5zxYeYjvLcoCwmFf1RIrpeOIZYTT0BCWMFIMcwoFDQ/640?wx_fmt=png)

然后就是选择输出路径和填写安装程序的名字了

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHVBlszWTiabhYPH2fw0MnPMjuNDQ6ER1HvibPLLZic3VkPAYQmUJvFicdsQ/640?wx_fmt=png)

然后下一步，直接点 Next，然后结束

配置到最后一步了，脚本文件，到这里会弹出问你是否马上编译，选择否，先把脚本写好再自己编译

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHyqvpp333mJ8CMvswVNJVSmibwFh5sduibeDlKM5OwnhO5fEfn3zkPksw/640?wx_fmt=png)

然后到了最后一步了，把本地的 JRE 写进脚本

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHHHTicP9s9J0H5Ur3SpNYoypro8U2lfNkFgWC82LtVnL4XfDPtl07V9g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHKyrKaib8WzpMEz09O1hK1yTFwkyZTurX7CEVx4cibibxKXze7ULuzAwLA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHpLpwgqI2PoNdvCEWv6udwVFFh73tYvG1Pe9dkzJD1enGVVGZE4gqrA/640?wx_fmt=png)

Source: "自己本地 JRE 路径 \*"; DestDir: "{app}\{#MyJreName}"; Flags: ignoreversion recursesubdirs createallsubdirs

然后直接编译就好了，会提示保存当前脚本，随便起个名字，下个还可以继续用

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHiaanHDIkshwib0D3vJtaJXL432qVUDDDpCibaJwR3Bpy6d0ag9DibFqpGg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHW1K4MtHyOww1CNYiaYKia8oDbxHqB8TXBJckCE4YKHxR3oW7zPnItxmw/640?wx_fmt=png)

然后等待绿色滚动条结束

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OH9jwHuuTYuR79Rdmwq9icxD7jLN8iagGoLxU4t5025cNRSgEUf6a1k9gw/640?wx_fmt=png)

当绿色滚动条结束后，桌面会多了一个 setup.exe 文件

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OHicaCAOFRjT4Aygfz4OZsLick7SzicekMY8o4Vq8tYeVRZzYYjiakuibgRhQ/640?wx_fmt=png)

也同时会跳出一个安装的，因为程序帮你自动启动生成的安装程序了，安装就可以了，安装的时候记得勾选创建快捷方式

![](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFiaVT9v1Yj3YGBBDMaIL0OH0oa40icOj1I8ebVVCAicHBuCn3GLIClUJsibuW0GzcmSWjYoUU5bqp0wA/640?wx_fmt=png)

这个就是最后的程序了，双击运行就可以看到结果了，把 setup.exe 文件给别人安装，就都可以看到自己的程序了！