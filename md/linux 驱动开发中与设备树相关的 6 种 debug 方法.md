> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666556104&idx=3&sn=485b0d3aa8fa04cf2580cf8fcb205b18&chksm=80dcae63b7ab2775e81724a4980cc95055e4abf884c15f58b26312daf9017076cf8eaea6fc1c&mpshare=1&scene=1&srcid=0725aFQYKB1QCNXcz0yrTQq0&sharer_sharetime=1627209962238&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

整理出了 6 种驱动开发时与设备注册、设备树相关的调试方法，彼此间没有优先级之分，每种方法不一定是最优解，但可以作为一种 debug 查找问题的手段，快速定位问题原因。例如在芯片验证时，不同时钟频率下系统启动情况摸底时，U-Boot fdt 命令可以方便快捷的帮助我们完成这个实验。

### 1. dtdiff 工具

这个文件需要在宿主机安装，在对比二进制的 dtb 文件时比较方便。文本格式的 dts 文件对比并不需要这个工具。

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VpkHxggFcicH8POAbTxey6vpGLfJDCofCicWuDsjiaWFyhaO7JDQ11W8gA/640?wx_fmt=png)

对比以下两个 dtb 文件的结果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VcTQZicB3zDmCwf6ewqyIrFEtdbt4WFPHlL31aHHofyIbWjIhUjjI4Hw/640?wx_fmt=png)

### 2. kernel device-tree base

系统启动后进入到 / sys/firmware/devicetree/base 目录可以看到当前已注册设备的设备树信息，通过相关命令可以查看当前设备的结点信息、状态等。

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VIrRDv4Rknl8PSjr9icec5lMdYDLVa0sUqJL3ueq6tIo69t7fCeXYgpg/640?wx_fmt=png)

上面各个子目录里显示的信息和设备树 dts 文件中定义的条目数是一样的。

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VickNl3w3eWRqyKmH4tutvkbjxSYEd2xiafK56Un27XR4q1ztJd576viag/640?wx_fmt=png)

### 3. U-Boot fdt command

驱动代码在 debug 期间，若希望更改外设模块的设备树属性时，在不改变存储设备中 dtb 文件的前提下，进入到 U-Boot 的命令行界面，通过 U-Boot 的 fdt 命令来实现。例如修改外设时钟源、修改外设时钟名、status 属性等。为了使 U-Boot 支持 fdt 命令需要打开 CONFIG_OF_LIBFDT。

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VZhReibqCVykbBYNRQKaibc9vWv3uBvWvfAd3pYr2ySgHCB4yDicXK2cHQ/640?wx_fmt=png)

U-Boot 提供的 fdt 命令是针对内存中的 FDT 而言的，因此，需要将存储设备中的 dtb 文件加载到内存 RAM 中。然后再告知 FDT 设备树在内存中的地址。

将 dtb 文件从 mmc 中加载到 DDR 的 0x61000000 地址处，并告知 U-Boot FDT 文件在内存中所在的位置为 0x61000000。

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VtgmGIT9ADpib32DyEqueNxtYr0rQGiaRrGTXFaAmRNg3bWKFuVmUctVQ/640?wx_fmt=png)

通过 fdt print 查看测试驱动 driver-test 的设备树信息，当查看某一个设备树结点的信息时，需要使用绝对路径进行设备树结点的索引。

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VOtiaibmuQU56uXPleFaXxG2HWNgPmuboR6bYkiaToRlhub7TaWWFib4jdg/640?wx_fmt=png)

driver-test 的设备树定义在源文件中 dts 如下图，dtb 内的信息是完全展开的，实际上和 dts 中信息完全一致。clocks = <0x00000005> 是 dtc 编译时对结点引用 label 重新插入的 phandle 值。

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VcZWxv3GOt05q02AjtvzEOqQzIB0ic6Y4bwdM40KtxdvxGyNJxWMEtiaw/640?wx_fmt=png)

#### 3. 修改设备时钟

设备树文件中 driver_test 的时钟源为 oscclk2，时钟名为 apb_clk。现在将 driver_test 时钟源设置为 oscclk1，时钟名改为 ahb_clk。oscclk1 在 dtc 编译后的 label 编号时 0x00000012。

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VYyOKvXGNxvnleYK6aIWUdOUuK2L04vGGhNfC1OzUg3SvnxvD1R9eyA/640?wx_fmt=png)

修改后如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8Vw7u9SgYDBNLeJRqbHeznKYuVpsgBXe3Iq2E3icdLJXpeDibguWicrPJ0Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VIbibjdoNI1x5Xxvjajqslwag3xET2q1X5KCowib1F9wN2Je9QfyDh4uA/640?wx_fmt=png)

修改完之后，手动加载 kernel 镜像来启动系统。系统启动后查看设备树信息是否修改成功。可以看见 clock-names 已经由原来的 apb_clk 更改为 ahb_clk。

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VkTa3uhVNcewajB5uVA4LAvjC5c52EBvkeedFE9wpZOgaW71VzAlU8g/640?wx_fmt=png)

#### 3. 修改设备 status 状态

设备树里 status 可以决定设备使能状态，status 状态支持以下几种格式，若设置了 status 为 disable，那么设备是不可用的。若不设置 status，默认设备可用。

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VvPF4YqGpy4NlnkBoNb6Gmmyxd271eibcvoMG9IMwAmNtTUspRdPSAwQ/640?wx_fmt=png)

在 platform_device 创建时会检查设备的可用性，若设备不可用，那么是不会创建 platform_device 的。of_device_is_available 用于检查 status 属性。

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VSXPPRAwRmSpeuww89ia7bAz6ynFpjlqDfEIWY14xTKRAGbmKNF95RdA/640?wx_fmt=png)

driver-test 的设备树里定义了 status = “disable”，查看设备结点的 status 信息也显示为 disable。

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VcZWxv3GOt05q02AjtvzEOqQzIB0ic6Y4bwdM40KtxdvxGyNJxWMEtiaw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VCBiaNdrBYNvVkJv78494LNKS7qzIrQhkPduP9MOFSuqa7DfJYviak8wQ/640?wx_fmt=png)

加载 driver-test 驱动以后设备未创建成功，当然也就无法执行驱动的 probe 函数。这是除 compatible 不匹配之外的另一个无法执行驱动 probe 函数的原因。

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VZiaI8N2FJaWez5HQUkgNworfVISXXou5iaoVIKaemx95nyNcTicF1QHUg/640?wx_fmt=png)

现在重启系统进入到 U-Boot 的命令行模式，通过 fdt 修改 status 的值为 okay。

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VL1ibnkxmTyUllNmn8icQ13sSHYjFx2aso1QoOZCA8engbHgPZapGuSjA/640?wx_fmt=png)

启动系统，再次确认设备树结点信息是否修改成功以及驱动是否执行了 probe 函数。通过系统启动的 log 信息可以看到，当修改完 status 状态值之后，driver_test 的 probe 函数得到了执行。

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VolyvEUOxPT2dWic1vnDXzRnapGr0WdCjPqLe7bIvVVOfSqia5QZ1kxnA/640?wx_fmt=png)

driver_test 设备也正常的注册进 platform 设备中。

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VP2qZX6DvCT8Me1p455KMWley8YOBIwsvuC2icuHB7XbjsmZ6lNe4zJQ/640?wx_fmt=png)

#### 3. fdt 其他功能

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VA6WbG6SRiaEEIukoEasDAZ2r4QyIn0uGPKibS7oy53GiaBN5Dk5Al3aEQ/640?wx_fmt=png)

fdt print 可以打印整个的 dtb FDT 信息

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8V7f28LAh92icXIhL9TVx56GHBR323mphB8p0RtpGHZmD39X43H0fsHag/640?wx_fmt=png)

fdt header 查看 dtb 的头部信息，通过 size 大小也可以间接的判断当前加载的设备树文件是否为所需的设备树。

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VicNeWhqg79eiaTZgIBfA6O0Zic6IHEUrFu1UKV93xc6s2SgXXQp8ATUMA/640?wx_fmt=png)

### 4. dtc 工具

dtc 可以使用宿主机提供的亦可以使用 kernel 提供的。这个工具是将已编译的 dtb 文件反汇编。

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8V0oqQfulHTvEUULWIBrclfN9Xv9icib4fyveU7g2CBWd0IkPTZBwk8Xuw/640?wx_fmt=png)

### 5. 查看 kernel fdt 文件

这个 fdt 是未解压缩的 dtb 文件，里面的内容和 dtb 完全一样。在 kernel 系统中执行 hexdump 查看：

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VsT5fOs7RpYLmZFWwjwHsibTDicRs3uomK9ehCSVxuibH73EkJqbcGp3Bw/640?wx_fmt=png)

通过 UE 查看原始的 dtb 文件，与 fdt 文件内容完全一致。

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VibibspEXS8f11e2gIMIYf9nFmhfYCPXbibjB2P0yJxictJBSia3koLFuRxQ/640?wx_fmt=png)

### 6. of_property_xxx

在代码中可以调用 of.h 中提供的 API 来检查或这获取 device node 的信息。

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8VGM690Ryln2D47CHJE4eMWj84GzLnRpvkXNPwViaYlFnIWk4fXtlbcjA/640?wx_fmt=png)

例如下面的调用方式

![](https://mmbiz.qpic.cn/mmbiz_png/icSw2jOnyJ9bTgEtwT7b1y9KNcNV8kc8V521AGrlXeshP9u9gtjXoQX63nFRticE1JJJic0cn2Libdmq7wvjRvicjqg/640?wx_fmt=png)

点赞和在看就是最大的支持❤️