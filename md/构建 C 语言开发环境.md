> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/p_oKh-wsN8pN2H-3YHvXhA)

编译 | 王瑞平

作为程序员，在开发过程中，我只有一个简单的愿望：**在当前的开发环境中将 C 库简化为少数几个文件。**

在本文中，我们将介绍如何为 **C 语言项目**构建容器化开发环境，也将介绍如何使用 CMake 设置构建系统、使用 Unity 设置测试环境以及如何在 **CI 流水线**中构建容器化环境。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQoOJQQprmIGVMDdTdhZl4ib6SlG4gLUichgR8iaYjPK3ZttOJch09cCxSSgq7E1eLAlSa4af2VQFice2g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

接下来，我们将展示如何为 C 项目构建**完整的、容器化**的开发环境：

· 创建 Docker 镜像作为 vscode 的开发容器；

· 基于最小化的 Dummy 库，在容器中设置构建库的工具；

· 设置静态代码分析器 clang-tidy 检查代码是否有常见错误；

·clang-format 维持代码库的格式保持正常和整洁；

· 设置 Unity，通过在主机上执行 Ceedling 测试虚拟函数；

· 最后，我们将设置 GitHub 工作流，使用本地 Docker 镜像**执行、构建**和**测试**项目。

在本文中，我将使用 Docker 命令行接口。如果你不明白为什么需要某些参数，建议参考在线文档。你也可以直接打开 GitHub 上的示例项目。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQoOJQQprmIGVMDdTdhZl4ib6ojD8pHUQOsKpMfyhXp67maIRcFLyAdv0lgYYV3OI2g2zwpRHL6ljYg/640?wx_fmt=png)

有时，使用**嵌入式系统**或 **C/C++** 需要安装大量**专用工具**或**编译器**。如果你正在同时处理不同的项目，版本之间很容易发生冲突。因此，我更倾向于在 Docker 容器中运行所有程序。

你可以使用 **Dockerfiles**，这能避免在本地安装工具，任何人都能通过**预构建镜像**或**本地镜像**加入项目。

### **1. 为什么是 Docker? 存在哪些陷阱？**

如果你是 **Docker 新手**，需注意以下几点：

（1）Dockerfiles 不稳定：昨天构建出的 Dockerfile 今天就可能无法使用，存在太多的外部依赖关系；

（2）Docker 不是平台独立型的，例如 Apple ARM；

（3）某些 Docker 特性仅在 Linux 或专用 Windows 中获得了支持；例如，并非所有平台都支持将 USB 设备安装到 Docker 容器之中，这是自 2016 年以来的限制；

鉴于此，建议你不要将 **“赌注”** 都 **“押”** 在同一项技术上，并且你需要随时做好**切换**的准备。

### **2. 让我们这样做** 

让我们从零开始，构建**空存储库**：

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQrEmxLHMHkk7QvABfN5y1HBMeB11LIIlPiblCAKiaaibJiaxIiaCYlmf8t3iaJDqeKibexEr0N8aGeDOPwvw/640?wx_fmt=png)

确保安装 Dockor 并顺利运行，在项目的根目录中创建如下内容：builder.Dockerfile

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQrEmxLHMHkk7QvABfN5y1HBXbm0qk2VQorICibglJkEZicyYgj7dqwKvCzh5LPwFjDbclSH6FeqoruQ/640?wx_fmt=png)

这个 Dockerifle 指定了基本镜像并安装了一些包，便于在后续步骤中使用。我不会详细介绍每一个**软件包**：在创建镜像时，你很快就能注意到缺少了什么，可以扩展软件包列表，重要的是以下几点：

· 对于基本图像，我强烈建议使用特定标签。apt 包在基本镜像之间变化很大，选择标记可以为你节省更多时间；

· 我倾向于在镜像开发过程中使用特定平台 ，这一点对后续开发步骤的顺利进行很重要。

如果不使用 **vscode**，也可以指定不同的镜像。在本文中，我们将使用 vscode，也将坚持使用 Dev 容器所支持的镜像。

镜像是用如下的命令构建出来的，执行起来可能需要一点时间：

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQrEmxLHMHkk7QvABfN5y1HB763Fk5ibIuIgQzBiaKxXYLlozv8KnDhpGc14dIMiad5cWTNibGYJZghadQ/640?wx_fmt=png)

### **3. 详细命令行调用的快捷方式** 

Docker 命令非常冗长，因此，我通常将常用命令放在 makefile 项目根目录中。假设安装 make 后可进行如下操作：

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQrEmxLHMHkk7QvABfN5y1HBlW7evh276dribpVHqr4CxXu6raWeoOXLgsLCQcxlH48mVibxsSWv6fWg/640?wx_fmt=png)

现在，执行如下操作重建镜像：

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQrEmxLHMHkk7QvABfN5y1HBIZx2S8wIOicGrEXP50pLv7Bd8JUYjoAafm3oQic4OialPIl03NlAtia6Ug/640?wx_fmt=png)

让我们从图像中旋转容器并进行测试：

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQrEmxLHMHkk7QvABfN5y1HB0ImicBhUY86r4ewibtibFdibfNUBVEOg31PzSDeGFCspPZu9XjHRiaE9vzA/640?wx_fmt=png)

当使用 apt 时，已安装的工具版本取决于**基础镜****像**，也取决于**包注册表**。如果需要安装特定的版本，可以通过执行自定义的 RUN 命令。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQoOJQQprmIGVMDdTdhZl4ib6O3T8iahkmTbcEjAuV6rXuzmq4SSibXZ8Tqk3TfvrIP54wMx2PtEYzraw/640?wx_fmt=png)

**没有在本地安装所有工具的缺点是：**你选择的 IDE 无法利用这些工具，例如，当使用 vscode 时，如果没有安装编译器，你将无法正确设置**智能提示**或任何其它的**辅助程序**。

vscode 允许你在开发容器中运行编辑器。这也是我们选择将 mcr.microsoft.com/vscode/devcontainers/base 当作基础图像的原因：我们可以在容器中链接到 vscode，因此所有工具都将被安装在 Docker 镜像中。

值得注意的是，vscode 实例与本地 vscode 安装不匹配，与远程实例非常相似。

通过创建.devcontainer/devcontainer.json 文件，我们可以让 vscode 使用新构建的图像作为开发容器，还可以在 vscode 实例中安装 3 个扩展，通过使用 customizations.extensions 字段中的 devcontainer.json 配置文件：

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQrEmxLHMHkk7QvABfN5y1HByxNALZrIk68VXR8UnC4WL11q5UUXZ9jtUic8ib6IZpnaLKcNLqibbjvPA/640?wx_fmt=png)

如果你从现在开始重新**加载窗口**或重新打开 vscode，vscode 应该会询问你是否需要使用检测到的开发容器。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQrEmxLHMHkk7QvABfN5y1HBOXRUEPxltZM3HjicBAyhjK38SWJrxUYR31yrUJfP0GbeBEaiczibkm6EQ/640?wx_fmt=png)

需要一段时间为 vscode **设置**你的**容器**、**安装扩展**，并用 vscode 连接到 **Linux 容器**。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQrEmxLHMHkk7QvABfN5y1HB5Ciaq5DN6qD1hsDfwJKTSvb7THr51NmibLuhCODVG5yCwWSh9MF6FVlQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQoOJQQprmIGVMDdTdhZl4ib6hstzqmSo5zAnrjSfHCLrHJscT0p587IuvAq9TqRlESXsnuhOSenJVQ/640?wx_fmt=png)

我们将使用 CMake 构建单独的.c 和.h 对。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQrEmxLHMHkk7QvABfN5y1HBIm1XUx0RicOKN3jGzGYmw3XqvjuYp8WzLLgERaMRM3rIUYlYicjBchDA/640?wx_fmt=png)

CMakeLists.txt 简单定义了名为 “Dummy” 的库，并将相应的文件添加到库中。

重要的是：这已在开发容器和 vscode 中被构建出来了! 在你的远程实例中打开集成终端并执行 CMake，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQrEmxLHMHkk7QvABfN5y1HBpac8OO365ziclwuoLqA4EqUshauiaezXJicIj3uzy8XXcFoGOPXiar4oXQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQoOJQQprmIGVMDdTdhZl4ib65QFgOcKJzaCVbBqDotEJPTERiagyywB6MJ1Um666liay6EjnZ1SnecCA/640?wx_fmt=png)

另外，在代码库上进行协作时，格式化器极好；当我们在安装 clang-tidy 时，不妨继续安装 clang-format。

你需要将两个配置文件 clang-format 和.clang-tidy 放置到**项目根目录**中，以便任何 IDE 都能自动拾取它们：

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQrEmxLHMHkk7QvABfN5y1HBJAaujFQm88nia9XehZrq5JbAN4EgP8DEqFr03AzjUKWsh2KyOt1rERg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQoOJQQprmIGVMDdTdhZl4ib6cG9d2kFros6o29JksW0UhoYRX8hv8ibMZXvVoAG5VadxWXH0xObjwSA/640?wx_fmt=png)

我们已经能够构建并分析库，还提供了**格式化**功能。在开发环境中，我们还需要一个**单元测试框架**。

在本文中，我选择了 Unity 测试框架，通过 Ceedling 在主机上执行。顺便说一句，使用这个框架便于**嵌入式系统**在目标硬件上执行任务。

### **1. 安装 Unity 和 Ceedling** 

在构建者镜像的第一步中，我们已经安装了 ruby，所以，安装**单元测试工具**变得更加简单：

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQrEmxLHMHkk7QvABfN5y1HBl02NicicuESblFfXnbzlgftHv7dCjMtvYcKBUbGCn3eNSzjEA42BGSgQ/640?wx_fmt=png)

重建镜像后，我们就可以开始了!

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQrEmxLHMHkk7QvABfN5y1HB5hVIj0iaVt0RSDWPAJPwtUp4I2k3FibprffKPoUsaZ8KnGPribfccQDzA/640?wx_fmt=png)

### **2. 配置 Unity 和运行单元测试** 

简而言之，你需要在一个专门的 unity_config.h 文件中将配置开关设置为 Unity 并配置 Ceedling 与 project.yml. Ceedling，为你生成所有的测试运行程序。

你需要做的就是**添加**你的**测试文件**，然后 **“告诉”**Ceedling 如何检测它们：

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQrEmxLHMHkk7QvABfN5y1HB3BMOYZLJMYhfkMHIy8oElkicJdKdj9D3oHzWbBQN5pXo1fgsdRDfYYA/640?wx_fmt=png)

然后，我们可以创建第一个单元测试 tests/unittest/test/test_dummy.c：

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQrEmxLHMHkk7QvABfN5y1HBRNRI09evSFEzKJQHbfTPSW8pezggIZcAXjTdTWChcQnfMSd68rhbvw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQoOJQQprmIGVMDdTdhZl4ib6KCms1SHtOITOE4iboHhsalSBVXkJJAboy83qDibT2B6SC2WZUI4IrgoA/640?wx_fmt=png)

到这里，所有的工作就都完成了，包括**库的构建**等，报告也可以使用了。神奇的是，所有这些操作都不会使计算机因工具而 **“堵塞”**。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQrEmxLHMHkk7QvABfN5y1HBF9vhGoxKTDtfcicnKsqzGqMrWpfFrzYefNurVx5hxfQwDCLxrCLkHgw/640?wx_fmt=png)

最后，值得一提的是：有了 Docker 桌面，你不仅可以轻松**检查图像漏洞**，还能**检查 Dockerfile** 中的每一步：

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQrEmxLHMHkk7QvABfN5y1HBOjUZ9aWH9sPy9NgJXV1BDzoicZXHd5ib3DHRKqCS1pgU6Ltpib0jmdbRw/640?wx_fmt=png)

现在，你已具备在 GitHub 上用 CI 设置容器化 C/C++ 项目的所有技能。有了这项技能，你能轻松地在文档中添加**特定编译器**或**清理**设置 CI 时出现的所有**错误**。

1.https://code.visualstudio.com/docs/devcontainers/tutorial

2.https://interrupt.memfault.com/blog/a-modern-c-dev-env

3.https://www.throwtheswitch.org/unity

4.https://embeddedartistry.com/blog/2019/02/25/unit-testing-and-reporting-on-a-build-server-using-ceedling-and-unity/

5.https://github.com/features/actions