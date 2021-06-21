> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.xampp.cc](https://www.xampp.cc/archives/8182)

> 文中列举了 123 个 Python 渗透测试工具，当然不仅于渗透~   下面我们就开始吧~ 如果你想参与漏洞研究、逆向工程和渗透，我建议你时候用 Python 语言。

文中列举了 123 个 Python 渗透测试工具，当然不仅于渗透~

下面我们就开始吧~

如果你想参与漏洞研究、逆向工程和渗透，我建议你时候用 Python 语言。Python 已经有很多完善可用的库，我将在这里把他们列出来。

这个清单里的工具大部分都是 Python 写成的，一部分是现有 C 库的 Python 绑定，这些库在 Python 中都可以简单使用。

一些强力工具（pentest frameworks、bluetooth smashers、web application vulnerability scanners、war-dialers 等）被排除在外，原因是部分工具在德国法律上有一点争议——就算最高法院曾经认定过。这个清单的主要目的是为了帮助白帽黑客，所以我还是怂一点。

### Network

*   **Scapy, Scapy3k：**发送，嗅探和剖析并伪造网络数据包，可以做交互式应用或单纯的作为库来使用
*   **pypcap, Pcapy and pylibpcap：**几个不同的 libpcap 捆绑 Python 库
*   **libdnet**：****低级别的网络路由器，可用于接口查找和以太网帧转发
*   **dpkt:** 快速、轻量级的数据包创建、解析工具，适用于基本 TCP/IP 协议
*   **Impacket:** 探测和解码网络数据包，支持更高级别协议比如 NMB 和 SMB
*   **pynids**：****libnids 封装提供嗅探，IP 碎片整理，TCP 流重组和端口扫描检测
*   **Dirtbags py-pcap**：****无需 libpcap 即可读取 pcap 文件
*   **flowgrep**：****通过正则表达式查找数据包中的 Payloads
*   **Knock Subdomain Scan：**通过字典枚举目标域上的子域名
*   **SubBrute：** 快速子域枚举工具
*   **Mallory：**可扩展的 TCP / UDP 中间代理，支持即时修改非标准协议
*   **Pytbull****：**灵活的 IDS / IPS 测试框架（配有 300 多个测试用例）
*   **Spoodle****：**大量子域名 + Poodle 漏洞扫描器
*   **SMBMap****：**枚举域中的 Samba 共享驱动器

### 调试和逆向工程

*   **Paimei****：**逆向工程框架，包含 PyDBG, PIDA, pGRAPH
*   **Immunity Debugger****：**可脚本化的 GUI 和命令行调试工具
*   **mona.py****：**Immunity Debugger 中的扩展，用于代替 pvefindaddr
*   **IDAPython****：**DA pro 中的插件，集成 Python 编程语言，允许脚本在 IDA Pro 中执行
*   **PyEMU****：**全脚本实现的英特尔 32 位仿真器，用于恶意软件分析
*   **pefile****：**读取并处理 PE 文件
*   **pydasm****：**ibdasm x86 反汇编库的 Python 接口
*   **PyDbgEng****：**Python 封装的微软视窗操作系统调试引擎
*   **uhooker****：**截获 DLL 或内存中任意地址可执行文件的 API 调用
*   **diStorm****：**AMD64 平台下的反汇编库，通过 BSD 许可
*   **Frida**：****一个动态的工具框架，可以将脚本注入到运行的进程中
*   **python-ptrace****：** Python 语言写成的应用 ptrace 的调试器 (Linux，BSD 和 Darwin 系统调用跟踪进程)
*   **vdb / vtrace**：****vtrace 是一个 Python 实现的跨平台进程调试 API，vdb 是一个应用该 API 的调试器
*   **Androguard****：**安卓应用程序的逆向分析工具
*   **Capstone****：**一个轻量级的多平台多架构支持的反汇编框架。支持包括 ARM,ARM64,MIPS 和 x86/x64 平台
*   **Keystone****：**一个轻量级的多平台多架构支持的汇编框架
*   **PyBFD****：**GNU 二进制文件描述 (BFD) 库的 Python 接口
*   **CHIPSEC****：**分析硬件，系统固件（BIOS / UEFI）和平台组件等 PC 平台安全性的框架。

### 模糊测试

*   **afl-python****：**用于纯 Python 代码的 American fuzzy lop
*   **Sulley****：**一个模糊器开发和模糊测试的框架，由多个可扩展的构件组成的
*   **Peach Fuzzing Platform****：**扩展的模糊测试框架 (v2 版本 是用 Python 语言编写的)
*   antiparser: 模糊测试和故障注入的 API
*   **TAOF：**(The Art of Fuzzing, 模糊的艺术) 包含 ProxyFuzz, 一个中间人网络模糊测试工具
*   **untidy****：**针对 XML 模糊测试工具
*   **Powerfuzzer****：**高度自动化和可完全定制的 Web 模糊测试工具
*   SMUDGE
*   **Mistress****：**基于预设模式，侦测实时文件格式和侦测畸形数据中的协议
*   **Fuzzbox****：**媒体多编码器的模糊测试
*   **Forensic Fuzzing Tools****：**通过生成模糊测试用的文件，文件系统和包含模糊测试文件的文件系统，来测试取证工具的鲁棒性
*   **Windows IPC Fuzzing Tools****：**使用 Windows 进程间通信机制进行模糊测试的工具
*   **WSBang**：****基于 Web 服务自动化测试 SOAP 安全性
*   **Construct****：**用于解析和构建数据格式 (二进制或文本) 的库
*   **fuzzer.py (feliam)****：**由 Felipe Andres Manzano 编写的简单模糊测试工具
*   **Fusil**：****用于编写模糊测试程序的 Python 库

### Web

*   **Requests******：****优雅，简单，人性化的 HTTP 库
*   **lxml******：****便于使用的 XML、HTML 处理库，类似于 Requests
*   **HTTPie******：****人性化的类似 cURL 命令行的 HTTP 客户端
*   **ProxMon******：****处理代理日志和报告发现的问题
*   **WSMap******：****寻找 Web 服务器和发现文件
*   **Twill******：****从命令行界面浏览网页。支持自动化网络测试
*   **Ghost.py******：****Python 写的 WebKit Web 客户端
*   **Windmill******：****Web 测试工具帮助你轻松实现自动化调试 Web 应用
*   **FunkLoad******：****Web 功能和负载测试
*   **spynner******：****Python 写的 Web 浏览模块支持 Javascript/AJAX
*   **python-spidermonkey******：****是 Mozilla JS 引擎在 Python 上的移植，允许调用 Javascript 脚本和函数
*   **mitmproxy******：****支持 SSL 的 HTTP 代理。可以在控制台接口实时检查和编辑网络流量
*   **pathod / pathoc******：****病态的守护程序 / 客户端，用于折磨 HTTP 客户端和服务器
*   **spidy******：****简单的命令行网页抓取器，具有页面下载和单词刮除功能

### 取证

*   **Volatility******：****从 RAM 中提取数据
*   **Rekall：** Google 开发的内存分析框架
*   **LibForensics**：****数字取证应用程序库
*   **TrIDLib：**Python 实现的从二进制签名中识别文件类型
*   **aft******：****安卓取证工具集恶意软件分析

### 恶意软件分析

*   **pyew******：****命令行十六进制编辑器和反汇编工具，主要用于分析恶意软件
*   **Exefilter******：****过滤 E-mail，网页和文件中的特定文件格式。可以检测很多常见文件格式，也可以移除文档内容
*   **pyClamAV******：****增加你 Python 软件的病毒检测能力
*   **jsunpack-n**：通用 JavaScript 解释器，通过模仿浏览器功能来检测针对目标浏览器和浏览器插件的漏洞利用
*   **yara-python******：****对恶意软件样本进行识别和分类
*   **phoneyc******：****纯 Python 实现的蜜罐
*   **CapTipper******：****分析，研究和重放 PCAP 文件中的 HTTP 恶意流量

### PDF

*   **peepdf******：****Python 编写的 PDF 文件分析工具，可以帮助检测恶意的 PDF 文件
*   **Didier Stevens&#x27; PDF tools******：****析，识别和创建 PDF 文件 (包含 PDFiD，pdf-parser，make-pdf 和 mPDF)
*   **Opaf******：****开放 PDF 分析框架，可以将 PDF 转化为 XML 树从而进行分析和修改
*   **Origapy******：****Ruby 工具 Origami 的 Python 接口，用于审查 PDF 文件
*   **pyPDF2******：****Python PDF 工具包包含：信息提取，拆分，合并，制作，加密和解密等等
*   **PDFMiner******：****从 PDF 文件中提取文本
*   **python-poppler-qt4******：****Python 写的 Poppler PDF 库，支持 Qt4

### 杂项

*   **InlineEgg******：****使用 Python 编写的具有一系列小功能的工具箱
*   **Exomind******：****一种旨在通过社交网络提供针对性攻击的工具。（原文是：围绕社交网络服务、搜索引擎和即时消息为中心创建装饰图形、开源智能模块的框架（framework for building decorated graphs and developing open-source intelligence modules and ideas, centered on social network services, search engines and instant messaging）感觉直接翻译原文不太到位，所以我找了一下官网，发现官网描述简单粗暴：）
*   **RevHosts******：****枚举指定 IP 地址包含的虚拟 hosts
*   **simplejson******：****JSON 编码和解码器，例如使用 Google&#x27;s AJAX API
*   **PyMangle******：****命令行工具和一个创建用于渗透测试使用字典的库
*   **Hachoir******：****查看和编辑二进制流
*   **py-mangle******：**** 重复项
*   **wmiexec.py******：****通过 WMI 快速轻松地执行 Powershell 命令
*   **Pentestly******：****Python 和 Powershell 内部渗透测试框架
*   **hacklib：** 黑客爱好者的工具包：词语破解，密码猜测，反向外壳等简单工具

### 其他有用的库或工具

*   **IPython****：** 增强的交互式 Python shell，具有许多功能，用于对象内省，系统 shell 访问以及自己的特殊命令系统
*   **Beautiful Soup****：** 用于抓取的优化版 HTML 解析器
*   **matplotlib****：** 制作 2 维图形
*   **Mayavi****：** 三维科学数据的可视化与绘图
*   **RTGraph3D****：** 在三维空间中创建动态图
*   **Twisted**：**** 事件驱动的网络引擎
*   **Suds****：** 用于 web 服务的轻量级 SOAP 客户端
*   **M2Crypto****：** 最完整的 OpenSSL 包装
*   **NetworkX：** 图像库（边、节点）
*   **Pandas****：**提供高性能，易于使用的数据结构和数据分析工具的库
*   **pyparsing****：**通用解析模块
*   **lxml****：**Python 中用来处理 XML 和 HTML 的功能最多、最宜于使用的库
*   **Whoosh****：**用 Python 实现的快速，有特色的全文索引和搜索库
*   **Pexpect****：** 控制和自动化其他程序，类似于 Don Libes`Expect` 系统
*   **Sikuli：**使用屏幕截图实现搜索和自动化 GUI 的可视化技术，可在 Jython 中运行
*   **PyQt and PySide****：**ython 捆绑的 Qt 应用程序框架和 GUI 库

### 书籍

*   **Violent Python 作者：**TJ O’Connor。一本黑客、取证分析师、渗透测试员和安全工程师的 cookbook
*   **Grey Hat Python 作者：**Justin Seitz。用于黑客和逆向工程的 Python 编程书
*   **Black Hat Python 作者：**Justin Seitz。用于黑客和渗透测试的 Python 编程书
*   **Python Penetration Testing Essentials 作者：**Mohit。使用 Python 的特性实现最佳渗透效果
*   **Python for Secret Agents 作者：**Steven F. Lott。使用 Python 分析，加密和发现智能数据
*   **Python Web Penetration Testing Cookbook 作者：**Cameron Buchanan 等。超过 60 个用于 Web 应用程序测试的 Python 用例
*   **Learning Penetration Testing with Python 作者：**Christopher Duffy。利用 Python 脚本执行有效和高效的渗透测试
*   **Python Forensics 作者：**Chet Hosmer。发明和共享数字取证技术的工作台
*   **The Beginner&#x27;s Guide to IDAPython 作者：**Alexander Hanel

### 演讲、幻灯片和文章

*   **Python &amp; Reverse Engineering Software 作者：**Alexander Hanel
*   **Python Arsenal for Reverse Engineering 作者：**Dmitriy Evdokimov 于 2016 年 RUCTF

### 更多

*   SecurityTube Python Scripting Expert (SPSE) 是一个 Vivek Ramachandran 提供认证的在线课堂
*   SANS 提供了课程 SEC573: Python for Penetration Testers.
*   Python Arsenal for Reverse Engineering 是一个有大量逆向工程工具的合集
*   一篇来源于 SANS 的关于可用于取证分析的 Python 库的文章 (PDF)
*   想要找到更多 Python 库，请查看 PyPI 的 Python 包索引

————终于完了，由于我对 Python 安全领域基本不熟悉，所以翻这个还挺费劲的。。。如果有大牛发现错误请指出来。。。

转载请注明：[XAMPP 中文组官网](https://www.xampp.cc/) » [python 资源：黑客必备 123 个 Python 渗透测试工具](https://www.xampp.cc/archives/8182)