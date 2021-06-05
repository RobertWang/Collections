> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [faq.fydeos.com](https://faq.fydeos.com/#why-does-the-usb-flash-drive-prompt-to-be-formatted-in-windows-after-burning)

> FydeOS 的常见问题、帮助手册及知识库

**FydeOS（Fyde 发音类似 “fide” `/faɪd/`) 是一款携带 Linux 内核 + 浏览器平台 + 容器技术驱动的轻量级操作系统；使用和 Google Chrome OS 十分类似。**

**FydeOS 能在大部分主流硬件上平稳运行。安装有 FydeOS 的硬件设备可以完美支持最新的网页应用标准、兼容安卓程序和 Linux 环境并且可以毫无障碍地在中国地区网络环境下使用，提供类似 Google Chromebook 的使用体验**

FydeOS 基于开源项目 [Chromium Project](https://www.chromium.org/chromium-os) 二次开发，我们对其核心代码进行了修改和优化，以最优化的浏览器平台为基础，加入更多符合中国地区用户习惯和提高用户体验的本地化增强功能，是一款真正符合互联网时代需求的操作系统。经过长时间的技术积累，我们可以让 FydeOS 运行在更多品类的硬件设备当中，提供围绕 FydeOS 的整体解决方案。

和 Chromium OS 一样，FydeOS 的诞生是缘于这样一种信念：今天我们所使用的应用和服务都将逐渐移向云端。随着浏览器平台技术和网页前端技术的不断成熟，我们深信，今天你通过网络所完成的绝大部分工作未来都可以通过一个简单的浏览器窗口而实现。我们正在步入一个全新的时代，需要安装的应用将成为历史。

目前我们有如下三种主要的 FydeOS 发行版，根据不同的场景，你可以选择最适合你的发行版下载体验：

*   **FydeOS for PC** 是最普适的 FydeOS 发行版，意在适配绝大多数带有 Intel CPU 及 Intel 集成显卡的的 PC 设备。首次启动需要使用 USB 存储介质引导，具体的安装指南可以查看[以下页面](https://fydeos.com/instructions-pc/)。如果你拥有一台 2010 年之后出厂的 携带有 Intel 集成显卡的 PC 设备，很大概率上 FydeOS for PC 是能够兼容的。
    
*   **FydeOS for You** 是我们为部分指定设备专门定制的版本，具有更好的适配性，并且支持一些根据具体设备功能。如果你恰好拥有已适配的这些设备，那么推荐你使用相应的版本。安装方式与 FydeOS for PC 基本无异。FydeOS for You 支持的设备列表将不定期更新，详情请查看[下载](https://fydeos.com/download/)页面。
    
*   **FydeOS VM for VMWare** 是我们针对 VMWare 虚拟机客户端专门适配的 FydeOS 虚拟机镜像文件。使用该发行版可以以最简便的体验方式体验 FydeOS，只需要[下载](https://fydeos.com/download/)最新的 FydeOS VM for VMWare 客户端的镜像文件，按照[指南](https://fydeos.com/instructions-vmware/)导入即可在安装有 VMware 客户端的所有主流操作系统上体验 FydeOS 的大部分功能。值得注意的是，由于 FydeOS 及其子系统设计的特殊性，FydeOS VM 里的部分功能是受到限制或者无法使用的，详情请认真阅读 FydeOS VM 的发布通知。
    

与此同时，我们还把制作 FydeOS 的一些核心技术开源，维护了能在嵌入式设备以及单板机开发套件上运行的 Chromium OS，包括：

*   [Chromium OS for Raspberry Pi](https://github.com/FydeOS/chromium_os_for_raspberry_pi): 适配著名单板机开发板树莓派及套装（3B / 3B+ / 4B / Pi400）。
*   [Chromium OS for Asus Tinker Board](https://github.com/FydeOS/chromium_os_for_tinker_board): 适配华硕出品的与树莓派同规格的开发板 Tinker Board。
*   [Chromium OS VM for VMWare](https://github.com/FydeOS/chromium_os-vm-vmware): 适配 VMWware 虚拟机客户端的 Chromium OS 虚拟机，也是 FydeOS VM for VMWare 的技术基础。

**相比较其它更为传统的 Linux 发行版，FydeOS 对于图形驱动的要求比较特殊。这意味着 FydeOS 只能在一部分符合要求的 PC 设备上运行。**

目前，FydeOS for PC 只能在使用 Intel x86 架构处理器的 PC 设备上启动。如需使用安卓子系统和 Linux 子系统的附加功能，设备必须带有第四代及以后的（约 2010 年之后出厂的）Intel 显卡芯片。

FydeOS（及其上游项目 Chromium OS）不同于传统的其它 Linux 发行版，其高度优化并集成进浏览器的图形栈（包含底层图形驱动、硬件加速模块及用户态的窗口管理器模块）为专有技术，为 FydeOS 提供更高的使用性能和用户体验之同时也限制了对硬件的兼容性。

如果如果你发现 FydeOS 由于种种原因未能在你的计算机上运行并需要帮助，请在我们的[中文社区](https://community.fydeos.com/)发帖求助并提供尽量详细的关于你的配置信息，我们以及社区里的爱好者们会尽快帮助你解答。

如果想要体验适配性最佳的 FydeOS，请尝试 FydeOS for You。

FydeOS 致力于适配并且兼容更多的硬件设备和外接设备。由于 Chromium OS 系统结构的特殊性，FydeOS 目前对非主流、未提供主线 Linux 内核驱动以及对主流 Linux 发行版支持不完善的硬件设备兼容性较弱。

这里的「某个设备」可能包括但不限于：

*   无线网卡
*   以太网卡
*   声音
*   打印机
*   耳机插孔自动识别
*   触控板
*   触屏
*   手写笔
*   指纹识别器
*   摄像头
*   重力感应模块
*   电池余量显示
*   电源管理策略
*   其它特有的外接设备

如果你发现你的「某个设备」无法在 FydeOS 上被识别导致功能缺失，可以在你的设备上尝试运行 [Ubuntu Desktop](https://cn.ubuntu.com/desktop/) 来检测该设备对 Linux 系统的兼容性。若出现在 Ubuntu 下能被识别及能正常工作而在 FydeOS 下失灵的情况，欢迎至我们的[中文社区](https://community.fydeos.com/)发帖求助并提供详细的硬件信息以帮助我们的开发团队进一步完善硬件兼容性。

另外，FydeOS for PC 在 x86 体系的 Chromebook 设备上也会出现不同程度的适配问题，因为 FydeOS 在构建的时候试图加入对更多 PC 设备的支持，为此去掉了上游项目对 Chromebook 的特殊优化。对于还在支持期间的 Chromebook 设备，我们仍然推荐使用 Google Chrome OS 已获得最佳体验。

FydeOS for PC 和 FydeOS VM 的使用是免费且不受时间限制的。但并不包含 FydeOS 的增值服务。

适用于 FydeOS for PC / FydeOS VM 的增值服务包括但不限于：

*   系统 OTA 升级服务
*   带 SLA 保障的工单服务
*   由 FydeOS 支持团队远程接入并帮助你解决问题的服务

针对 FydeOS for PC 的增值服务收费方案仍在制定中，敬请期待。

自 2020 年 9 月起，我们对安装 FydeOS for You 的设备开放「FydeOS 订阅服务」。关于 FydeOS 订阅服务的详细信息请查阅[这个问题](https://faq.fydeos.com/#what-is-fydeos-subscription-what-are-the-benefits)。

FydeOS 致力于提供更完整的行业解决方案，目前的商业模式围绕在对公的解决方案定制、技术咨询和系统授权。如果你是企业客户有意向采用 FydeOS 作为解决方案，欢迎与我们[联系](mailto:hi@fydeos.io)。

#### 什么是 FydeOS 订阅服务？

FydeOS 订阅服务是我们在 2020 年 9 月针对 FydeOS for You 推出的增值服务。

每一台安装 FydeOS for You 的设备自首次启动联网激活开始会获得为期 120 天的试用期。若你觉得 FydeOS for You 能帮助到你，在试用期内，你可以选择支付一笔 FydeOS 订阅费用。支付成功之后，你的 FydeOS for You 设备的订阅有效期会在支付时的基础上增加 365 天。

#### FydeOS 订阅服务怎么收费？

目前，FydeOS 订阅计划的费用是 120 元人民币 / 年。

#### FydeOS 订阅服务能给我带来什么？

*   继续不受限制地使用 FydeOS for You 及其所有附属云服务。
*   持续的系统更新，以 OTA 的形式下发到 FydeOS 设备上。
*   在 [FydeOS 社区](http://community.fydeos.com/)可以绑定已订阅的设备 ID，解锁「Patron」徽章之后即可向 `support` 小组提出关于 FydeOS 的技术支持工单。FydeOS 技术支持团队将在一个工作日之内给予回复。

#### 我如果不选择支付 FydeOS 订阅服务，我的 FydeOS for You 设备会怎样？

当 FydeOS for You 试用期结束之后，若系统检测不到有效的订阅，系统会做出一些限制使用的行为，并且给予充足的时间提示你保存现有的工作或备份数据。

**如果你觉得 FydeOS for You 能帮助到你，请考虑支付 FydeOS 订阅**。

发布和维护针对不同设备单独优化的 FydeOS for You 所需要的投入比维护 FydeOS for PC 要显著很多；完整的 FydeOS 体验还需要一系列云服务的支持，然而这些云服务的运维需要昂贵的支出来维系。

我们希望能将最好的 FydeOS 使用体验带给你；**这意味着我们对插入广告、转售你的隐私以及其它各种不优雅的变现手段一概不感兴趣**。

与此同时，我们认为「捐赠」是一种不太可控的且不甚公平的收入方式。针对 FydeOS for You 这个产品和其使用场景，支付订阅的费用来换取增值服务无疑是最直接也是最公平的方式。

我们希望 FydeOS for You 能给你带去帮助，如果你认可 FydeOS，请考虑支付 FydeOS 订阅计划。

通过 USB 存储介质启动后，确认 FydeOS 在你的硬件设备上的兼容性足够满足需求。由于 U 盘介质的读写速度受限制，若要使用 FydeOS 的完整功能你需要将 FydeOS 安装到硬盘。

目前我们提供了两种安装方案，一种是[「全盘安装」](https://faq.fydeos.com/getting-started/install-fydeos-to-hdd/)，一种是[「与其他操作系统多启动安装」](https://faq.fydeos.com/recipes/dual-boot/)。需要强调的是，这些教程假设你对多种操作系统的日常操作并不陌生且具备最基本的熟悉程度。若你觉得这些教程难以理解，请寻求身边朋友的帮助或者到 [FydeOS 中文社区](https://community.fydeos.com/)寻求帮助。如果你不确定，请不要继续。

从 FydeOS v11.3 开始，在首次启动 FydeOS 的第一个画面里即可呼出「FydeOS Installer」，你可以根据屏幕的提示将 FydeOS 安装到设备的硬盘里。

Google Play Store 需要 Google 官方授权才能进行分发。通过 Google Play Store 分发的应用通常依赖于 Google 的服务框架，一旦失去 Google 的官方授权将应用安装到操作系统中，这些应用往往会无法正常运行。

另外由于众所周知的原因，Google Play Store 也无法在中国大陆地区正常使用。

FydeOS 目前在系统内预置了「FydeOS 应用商店」和「酷安」（仅限安卓子系统），你可以在其中获取数以万计的扩展程序、 Chrome 应用，以及安卓应用。

如果你有一定的动手能力和使用安卓版 Google 服务的必要，可以在「FydeOS 应用商店」内获取相关配置程序进行配置以通过开源项目「Open GApps」满足你的需求。需要注意的是，该功能目前还属于测试阶段，无法保证可用性。

请注意：安卓子系统及安卓相关功能目前仅能适配 Intel 系列显卡。

首次登陆系统之后，系统会自动从 FydeOS 服务器上下载最新版本的支持插件和所需文件，请耐心等待。下载完成之后这些程序会自动安装，即可以在「应用程序启动器」中找到它们。

要运行安卓程序，需要激活启动 FydeOS 携带的安卓子系统。首次运行「安卓设置」即可开始激活过程。安卓子系统拥有一套独立的「最终用户使用协议」，请仔细阅读。你只有勾选同意之后点击「确认启动」才能激活启用安卓子系统。

安卓子系统被成功激活之后，安卓子系统的设置界面会随即开启。该「安卓设置」程序是之后访问安卓设置界面的入口。

目前，FydeOS 预置了「酷安」作为默认的安卓应用商店。你可以在其中下载安装你所需要的应用程序。当然，你也可以自行下载 apk 文件，双击即可通过「旁加载」的方式安装该 apk 程序；或者自行安装一个你喜欢的安卓市场程序，通过该市场程序安装新的安卓软件。

我们在 FydeOS 上测试了由 [Codeweavers](https://www.codeweavers.com/) 提供的 [CrossOver Chrome OS](https://www.codeweavers.com/crossover/download#chromeos)，可以在 FydeOS 上运行一部分常用的 Windows 软件。

值得注意的是，CrossOver Chrome OS 需要运行在 FydeOS 的 Linux 子系统下。该软件提供 14 天的免费试用期，目前的售价是 $39.95。

在 FydeOS for PC 版本中，您可以使用用户名 `chronos` 登录系统的命令行（shell）环境，该账户默认不设置密码。此外，该账户还可通过 `sudo` 命令提权。

基于安全性的考量，我们建议您尽早修改该密码。

关于如何修改 FydeOS 的密码，请参考[这篇知识库文档](https://faq.fydeos.com/recipes/chronos-password/)。

因为 FydeOS 所采用的分区格式在 Windows 下无法识别，因此会出现这样的提示，但千万记住，务必不要格式化，否则会造成 FydeOS 无法正常使用。

目前已推出了适配 VMware 的虚拟机版本，你可以移步[下载页面](https://fydeos.com/download/)下载尝鲜。需要注意的是，该虚拟机镜像仅能适配 VMWare Workstation Pro 15, Workstation Player 15 以及 WMWare Fusion 11。如果想要获得最佳体验，我们还是建议你使用 FydeOS for PC 或 FydeOS for You 版本。

非常遗憾 FydeOS 不能完整地适配你现在使用的计算机设备。如果你希望得到我们的帮助，请：

*   尝试在你的设备上运行 Ubuntu 等传统 Linux 发行版，对比并记录适配情况；
*   详细描述不能适配的硬件相关信息，包括（该硬件的）品牌型号、能否在其它 Linux 发行版下得到支持、相关驱动信息等；
*   将上述内容详细地发至我们的[中文社区](https://community.fydeos.com/)，等待我们技术支持给与你反馈和帮助。

十分抱歉，我们无法对个人用户做出任何关于适配的承诺，但若收到足够详尽的适配申请我们一定会尽力支持。

你可以使用磁盘管理工具（如 Disk Genius 等）删除 U 盘上的所有分区并新建一个分区，格式化成你想要的文件系统，请一定要谨慎操作。

这是一个已知的 bug，但我们发现在 Windows 10 上以管理员身份运行 Etcher 即可解决大部分不稳定的问题（这里感谢 Greg 同学提供该提示）。当然，你也可以尝试使用没那么高颜值但老牌更稳定的烧写程序，比如 [Rufus](https://rufus.akeo.ie/) 或者 [Win32 Disk Imager](https://sourceforge.net/projects/win32diskimager/)。

**如果你想要获得 FydeOS for PC 的自动更新，请确保系统设置中「自动保持我的 FydeOS 运行最新版本」的选项处于打开状态。**

FydeOS for PC 的更新有两种情况：

*   小版本更新，例如从 FydeOS for PC `v12.0` 更新到 `v12.1`。FydeOS 会自动检测此类更新，自动下载并进行 OTA 更新。当一切完成后，你只需重启 FydeOS 即可自动切换到最新版本，当然，你可以在任何你觉得方便的时候进行重启。
*   大版本更新，例如从 FydeOS for PC `v11.4` 更新到 `v12.0`。在 FydeOS for PC 中，此类更新涵盖了 Linux 内核以及其他系统层的更新。PC 版并不针对某一特定型号的设备，无法保证每一次大版本的更新都能在你的 PC 上流畅运行。因此，我们决定暂停 FydeOS for PC 自动进行大版本更新。你有以下 2 个选择：
    *   你可以自行下载最新版本的 FydeOS for PC，重新进行安装。由此，你也可以判断最新版本对你的设备的适配程度。但是，你之前的使用数据和所安装的程序都将被清空。
    *   对于想要保留使用数据的用户，也可以选择进行 OTA 更新，只需支付**一次性的小额费用**。当然，为了确保最新版的适配程度，建议你先在 U 盘中进行测试后再决定是否需要更新到最新版本。

以下是对 FydeOS for PC 提供收费大版本更新的 FAQ，以解释这一略微复杂的情况。

#### Q: 为什么要对大版本的 OTA 更新进行收费？

> 随着 FydeOS for PC 的用户量激增，每一次大版本更新背后的云服务费用大幅增加，这也是我们决定暂停大版本 OTA 更新的原因之一。我们希望维持 FydeOS 带给你的最佳体验，这也就意味着，我们拒绝一切不优雅的赚钱方式，例如，在 FydeOS 中插入广告或者销售你的隐私数据等。对于一些用户建议开通「捐赠」渠道，我们始终认为，这是不可控且有失公平的方式。因此，我们决定采用一次性收费的方式提供大版本的 OTA 更新，供有该需求的用户进行选择，这是目前我们所想到的最直接且公平的方式。

#### Q: 多少钱？

> 每次大版本更新的费用是 9.9 元人民币。

#### Q: 我支付的这笔钱到底花在哪里？

> 你支付的费用，大多数是 CDN 流量费用，包括从我们的服务器传输到你本地设备的 2GB 左右的数据，以及我们后台支持云服务的运维费用。

#### Q: 如果我不想付费，仍然可以使用 FydeOS for PC 吗？

> 当然可以。你可以关闭「自动保持我的 FydeOS 运行最新版本」选项，持续使用当前的版本。你也可以重新下载安装最新版的 FydeOS for PC。

#### Q: 每一次大版本的更新，我都需要进行付费才能获得 OTA 更新吗？

> 对的。

#### Q: 如果我不喜欢此次更新，希望退回到之前的版本，可以获得退款吗？

> 不可以。你已经消耗了更新的流量费用，我们无法对此进行退款。因此，请确保对新版本的满意度后再进行付费更新。

#### Q: 如果新版本在我的设备上无法运行，我该怎么办？

> 很抱歉，这多半是由于你的设备比较老旧而新版本无法兼容的缘故。你可以回退到之前的版本（在 grub 界面选择另一个启动项）。为了避免这种情况，建议你先在 U 盘中测试最新版本的适配程度后再进行 OTA 更新。

如你使用的是 FydeOS for You，请确保订阅在有效期内，否则是无法进行 OTA 更新的。

除此之外，请确保你是使用 FydeOS 应用商店中的「安装程序」将 FydeOS 安装进硬盘的。使用 etcher 等程序直接将 FydeOS 烧录进 U 盘或硬盘无法正常创建根分区，也就无法进行 OTA 更新。如你想要在移动硬盘上使用 FydeOS，也请使用「安装程序」将 FydeOS 安装到该移动硬盘，这样才能确保正常接收更新。

通过测试，我们发现通过移动存储设备引导启动的 FydeOS 将会受到较大的性能影响。若希望获得最佳使用体验，可以尝试将 FydeOS 装进硬盘。

另外，不同的存储介质对于用户体验有着巨大的影响。

我们推荐使用以下两种品牌的 SD 卡和 USB 盘烧写 FydeOS，当然，如果你有更好的选择也可以推荐给我们。

*   SD 卡：三星 EVO Plus 32GB SD 卡是一个性价比较高的好选择
*   USB 盘：SanDisk Extreme Pro USB 3.0

FydeOS 默认携带了名为「FydeOS 系统控制」的浏览器插件（id: `mofiofjpikncjaigmdlblhojbnkabako`）。此插件为 FydeOS 提供了不可或缺的系统功能，是 FydeOS 系统的重要组成部分。

由于该插件的特殊性，故在插件详细页面中显示该插件申请的权限内容非常多。但请勿担心，该插件仅为 FydeOS 及其附属功能提供服务，涉及系统底层的接口由`minijail`等方案包裹保护（见[设计文档](https://www.chromium.org/chromium-os/developer-guide/chromium-os-sandboxing)），最大程度保障了系统的整体安全性。

原则上 FydeOS 的核心版本会和上游项目 Chromium OS 以及 Google Chrome OS 的稳定版保持不超过 2 个大版本的间隔。比如，若 Google Chrome OS 目前的稳定版是`70.0.*.*`，则在最差情况下，FydeOS 会保持在`68.0.*.*`。

版本内的小规模修复将不定期发布，如遇到重要安全性相关的修补团队将第一时间发出。

Adobe Flash Player 由于其封闭的设计和安全性的缺失，正在被主流浏览器淘汰并停止更新。从 FydeOS v12 开始，Adobe Flash Player 不再默认提供。如果你仍在使用重度依赖 Flash 技术的网站或程序，建议你尽早替换。

我们仅能保证从 FydeOS 官方途径发布的带有 FydeOS 标志的操作系统镜像是安全的、无害的、符合我们《隐私保护政策》以及其它在中华人民共和国互联网安全相关法律条款的。

我们无法对第三方途径发布的类似 FydeOS 的操作系统产品、基于 FydeOS 技术开发的衍生产品、基于我们维护的开源项目二次开发的产品做出任何安全性保障。

FydeOS 所采用的 Chromium OS 开源项目对安全性极其重视。如果你发现 Chromium OS（及其附带的任何底层组件）中含有未被或已经披露的重大安全隐患，欢迎及时与我们取得联系。

不用担心，这是正常现象，务必不要格式化这些分区，否则会造成 FydeOS 无法正常使用。

FydeOS 应用商店中已有数千个应用供用户下载使用。由于这些应用全部来源于 Chrome Web Store，应用的自动更新在中国大陆地区可能无法正常进行。因此我们在应用商店中新增了「检查更新」功能以帮助用户将已安装的 Chrome 应用迁移成 FydeOS 应用。两者的区别仅有一处，即应用的自动更新链接。迁移完成之后，您即可享受应用自动更新的便利。

请注意，与您应用相关的本地文件和偏好设置不会在迁移后同步。请事先做好备份，以免造成损失。

会。具体收集的内容和我们使用及保护用户隐私信息的法律边界已在我们的[隐私保护政策](https://fydeos.com/privacy/)中详细指出，建议你尽早阅读。

在你开始使用 FydeOS 之前我们请你阅读并同意过我们的《软件许可协议》。继续使用 FydeOS 及其附属产品表示你同意我们的《隐私保护政策》。

可信平台模块 (TPM) 通常是一种植于计算机内部为计算机提供可信根的芯片，为了保障系统安全性，FydeOS 有不少功能依赖于 TPM 芯片提供的功能。市面上存在许多不同的 TPM 芯片及对应的规格，有些时候因为不兼容的原因 FydeOS for PC 无法加载你设备上的 TPM 芯片模块，会导致如下功能不可用：

*   <chrome://flags> 页面空白
*   浏览器无法导入自定义安全证书
*   「关于 FydeOS」-「更多详情」-「变更版本」按键不可用

若出现上述情况，可以开启此开关促使系统跳过一些原本设定好的对 TPM 芯片的检查以保障这些功能可用。

该功能目前为实验性功能，存在一些使用体验上的小缺陷，我们仍在不断完善，开放公测旨在让你第一时间体验新功能。

### 如何在桌面模式下启用 Android 输入法？

1.  打开「设置」中的「FydeOS 设置」，开启「桌面模式下 Android 输入法支持 (beta)」的开关，然后点击「重新启动」。 ![](https://fydeos.com/wp-content/uploads/2021/05/android-ime1.jpeg)
    
2.  重启后，在「FydeOS 商店」中安装安卓应用「搜狗拼音」。 ![](https://fydeos.com/wp-content/uploads/2021/05/android-ime2.jpeg)
    
3.  打开「搜狗拼音」，点击「第一步 启用搜狗输入法」，随后同意相关权限。 ![](https://fydeos.com/wp-content/uploads/2021/05/android-ime5.jpeg)
    
4.  打开输入法设置，点击「添加输入法」。 ![](https://fydeos.com/wp-content/uploads/2021/05/android-ime3.jpeg)
    
5.  勾选「搜狗输入法」后选择「添加」。 ![](https://fydeos.com/wp-content/uploads/2021/05/android-ime4.jpeg)
    
6.  在输入法中选择「搜狗输入法」，即可在桌面模式下启用搜狗输入法。如果出现问题，打开「搜狗拼音」应用查看是否有其他权限申请。 ![](https://fydeos.com/wp-content/uploads/2021/05/android-ime6.jpeg)