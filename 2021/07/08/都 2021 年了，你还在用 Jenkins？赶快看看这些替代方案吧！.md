> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU1Nzg4NjgyMw==&mid=2247491775&idx=1&sn=6f86703d8c04d6674af48e8a9cac5f9b&scene=21#wechat_redirect)

> 为大家介绍几个持续集成中常用的 Jenkins 替代方案。

Jenkins 是目前最常用的持续集成工具，拥有近 50% 的市场份额，它还是很多技术团队的第一个使用的自动化工具。但是随着自动化领域的持续发展，Jenkins 逐渐暴露出了一些问题，例如缺乏功能、维护问题、依赖关系和扩展问题等等。

> 本文将为大家介绍几个持续集成中常用的 Jenkins 替代方案。

1、BuildMaster
-------------

![](https://mmbiz.qpic.cn/mmbiz_png/FE4VibF0SjfNJ4OxL7zyb1UrbCNAK9hIiaNIUIhKgPXuF0JYxic4HpcDmcNLJBmnVdYYOp1eRMYb6N3wqiaJoOiazRQ/640?wx_fmt=png)

图片

> 项目地址：https://inedo.com/buildmaster

Inedo 的 BuildMaster 是 Jenkins 替代方案之一，开发人员能够用它将软件发布到各种环境，为各种平台提供全面的持续集成能力，使团队有能力创建私有的自助发布管理平台，单独处理自己的应用程序并私有部署。更重要的是，避免自动发布未经测试的软件。因为无需精通流水线即可使用，所以用户对它的简洁性都非常满意。

2、Microtica
-----------

![](https://mmbiz.qpic.cn/mmbiz_png/FE4VibF0SjfNJ4OxL7zyb1UrbCNAK9hIiaq9t10pwPQSa4pIS70BEsjTEZJpzxylc8UXSeExSn0xkBYgudCunSuQ/640?wx_fmt=png)

图片

> 项目地址：https://microtica.com/

Microtica 是 DevOps 自动化工具，从创建云基础设施到使用 Kubernetes 交付应用程序和服务，覆盖了整个软件交付过程。Microtica 的开箱即用组件为用户提供可重用的代码片段，无需额外编码即可帮你在几分钟内搭建起底层架构。

通过微服务生成器，开发人员可以自动化地创建微服务。通过已集成的预上线 Kubernetes 和本地 [Kubernetes](http://mp.weixin.qq.com/s?__biz=MzI4Njc5NjM1NQ==&mid=2247504258&idx=1&sn=ae3c5ef6591a48dcab42f45ebc25f448&chksm=ebd5eeaedca267b8544c595845e230ec710a9fd108cc70caeed13c496290740ffcbb7708d6d6&scene=21#wechat_redirect) 仪表板，只要点一点鼠标就能创建出可伸缩的应用程序。

Microtica 流水线定义每个组件和微服务的工作流。用户可以随时自动或手动触发它们，获取整个构建的概览。用户可以在 Microtica 网站内执行所有的操作，每次变更都有 Slack 通知。

最后一点，Microtica 允许开发人员设置自动化的休眠周期，降低 AWS 成本。一旦启动节约模式，Microtica 会自动运行，防止过度消费。而且，节省了多少钱还可在成本仪表板中看到。

3、GitLab
--------

![](https://mmbiz.qpic.cn/mmbiz_png/FE4VibF0SjfNJ4OxL7zyb1UrbCNAK9hIiay1yAvvzRzXwI86fG03C3rKHAwolSWcO16XFWdeO8beHrSK9wmUfB5w/640?wx_fmt=png)

图片

> 项目地址：https://about.gitlab.com/

GitLab 是在线 CI 平台，开发团队可以有效地使用各种开发工具，更快、更安全。通过集中统一的版本控制系统进行规划、构建和管理代码。此外，GitLab 使用户可以使用 Docker 和 Kubernetes 来处理构建输出、容器、应用打包和依赖项。有人表示 GitLab 很容易集成。但是，它有时会有一些令人讨厌的 bug 和限制，也缺少一些完全自动化的特性。

4、CircleCI
----------

![](https://mmbiz.qpic.cn/mmbiz_png/FE4VibF0SjfNJ4OxL7zyb1UrbCNAK9hIiam79qptvsHfYIo8BJJib1J2V0xMWib7HMxicibonCL6SE340wNTkQ6zXMJQ/640?wx_fmt=png)

图片

> 项目地址：https://circleci.com/

CircleCI 是一种可伸缩的 Jenkins 替代方案，它可以在任何环境（如 Python 接口服务或 Docker 集群）中运行。它消除了不稳定性并增强了应用程序的一致性。它支持多种语言，比如 C++、.NET、JavaScript、PHP、Ruby 和 Python。当最近的构建触发后，可自动取消队列中以及正在构建的任务。它可以与 GitHub、GitHub 企业版和 Bitbucket 集成。TrustRadius 用户说，自动构建是 CircleCI 的最大优势，但有时候任务太耗时。

5、Bamboo
--------

![](https://mmbiz.qpic.cn/mmbiz_png/FE4VibF0SjfNJ4OxL7zyb1UrbCNAK9hIiarhtTAQjl7g2MY1Uk8AuoOtAe9ibcF03HOHnxOpmFAicu8vLDickCf8GoA/640?wx_fmt=png)

图片

> 项目地址：https://www.atlassian.com/software/bamboo

Atlassian 的 Bamboo 是持续集成服务，可以自动从一个地方创建、监听和发布应用。它与 JIRA 应用程序和 Bitbucket 集成很方便。此外，Bamboo 集成了 Docker、Git、SVN 和 Amazon S3 存储。基于对仓库中变更的检测，可触发构建并推送来自 Bitbucket 的通知。它既可托管，也可在本地使用。G2 用户 说，Bamboo 构建过程的可视化很棒，但是一些术语和集成还不太容易理解。

6、TravisCI
----------

![](https://mmbiz.qpic.cn/mmbiz_png/FE4VibF0SjfNJ4OxL7zyb1UrbCNAK9hIiaA4auuOlaArjicuTYfHNkP9TrLGDGUHXLohYHibedNkP1wSGxeIiaGvHRg/640?wx_fmt=png)

图片

> 项目地址：https://travis-ci.org/

TravisCI 是持续集成托管服务，开发人员可以使用它来开发和验证 GitHub 和 Bitbucket 托管的应用程序。它可以测试所有 pull 请求，以确保不会发布出去未测试过的代码。用户可以登录 GitHub 来创建项目，包括配置快速激活的预安装数据库和资源。有评论说，TravisCI 非常适合想要快速开始构建的小项目。然而，在意构建的依赖关系、性能和可靠性的大项目，可能会遇到一些问题。

7、Semaphore
-----------

![](https://mmbiz.qpic.cn/mmbiz_png/FE4VibF0SjfNJ4OxL7zyb1UrbCNAK9hIiazhI1V2dew2fpDLf64P7I3XPYuUkmCwwpuIGFcWlJBstHuic2cMbqMkA/640?wx_fmt=png)

图片

> 项目地址：https://semaphoreci.com/product

Semaphore 是 Jenkins 替代方案之一，它覆盖整个 CI/CD 过程，支持 GitHub、Kubernetes、iOS、Docker，并预装了 100 多个工具。它可以自动化任何持续交付流水线，并提供自定义步骤、并行执行、依赖管理等。有人表示，Semaphore 构建非常快速，而且操作简单。然而，有用户表示，界面有时会令人困惑，而且部署流水线的方法有限。

8、Buddy
-------

![](https://mmbiz.qpic.cn/mmbiz_png/FE4VibF0SjfNJ4OxL7zyb1UrbCNAK9hIiatvpTEjaQr2DDjSEZI6cZZqseE2l9LI34lqj3sS52sfRZhS7iaVicZicSQ/640?wx_fmt=png)

图片

> 项目地址：https://buddy.works/

Buddy 是 CI/CD 平台，它通过简单的 UI/UX 来减少配置和维护 Jenkins 的工作量，这使得创建、评估和部署应用程序变得非常简单。

您可以在 15 分钟内通过具有即时 YAML 导出功能的图形化界面完成配置。它可以在云端和本地使用，并提供完整的 Docker 和 Kubernetes 支持。有用户反馈，Buddy 很容易操作，但是价格太贵。

9、Drone.io
----------

![](https://mmbiz.qpic.cn/mmbiz_png/FE4VibF0SjfNJ4OxL7zyb1UrbCNAK9hIiaqBpwzdn7Kd5m71DLUSyfvBxGhcgyOv3YKibHhDanxTO8vT0kVTFKbuA/640?wx_fmt=png)

图片

> 项目地址：https://drone.io/

Drone.io 是自助 CD 平台，它使用简单的 YAML 配置文件和 Dockercompose 的超集在 Docker 容器中创建和执行流水线。运行时会自动下载独立的 Docker，它执行容器中的每个流水线步骤。Drone.io 有 Docker 镜像，可以从 Dockerhub 下载。用户反馈，Drone.io 是 Jenkins 替代品之一，易于操作，是很好的企业解决方案，但是缺少一些特性，需要进一步定制。

10、GoCD
-------

![](https://mmbiz.qpic.cn/mmbiz_png/FE4VibF0SjfNJ4OxL7zyb1UrbCNAK9hIiaeibByVzkGEicVAKSm0yUOqDdHzxic8gmkricHNzxg5DXngdPTO3pBZ1c1A/640?wx_fmt=png)

图片

> 项目地址：https://www.gocd.org/

GoCD 是 ThoughtWorks 的持续集成开源服务。您可以使用它来简化动态工作流的模拟和可视化。它提供持续交付和优雅的设计来构建 CD 流水线，支持并行和顺序执行，可以随时部署任何版本，有活跃的支持社区。用户反馈，GoCD 与跨服务器扩展不兼容，但优点是可以自定义流程。

11、TeamCity
-----------

![](https://mmbiz.qpic.cn/mmbiz_png/FE4VibF0SjfNJ4OxL7zyb1UrbCNAK9hIiazYF70CxxNOqwdbSFsE2ZxVs72NWZpPlBxPvCEOAGKgAVKj7OplYlBA/640?wx_fmt=png)

图片

> 项目地址：https://www.jetbrains.com/teamcity/

TeamCity 是 JetBrains 的 CI/CD 工具。它允许用户在代码提交之前构建、监视和执行自动化测试，从而维护干净的代码库。它提供了全面的 VCS 集成，使 CI 服务器始终保持正常运行，即使没有任何构建。它可以与 Amazon EC2、Microsoft Azure 和 VMware vSphere 集成。用户反馈，TeamCity 是现代化的、健壮的和开放的解决方案，为流水线提供开发人员友好的环境，但是需要仔细对待服务配置。

12、Buildkite
------------

![](https://mmbiz.qpic.cn/mmbiz_png/FE4VibF0SjfNJ4OxL7zyb1UrbCNAK9hIiaMvcziaRUjgic7GFvn9VsocV0LkMJaSoO38QrGkeic8T5vIZdfSfe4CJvQ/640?wx_fmt=png)

图片

> 项目地址：https://buildkite.com/

Buildkite 是开源平台，可以在上面运行 CI 流水线。它提供了源码控制、聊天支持，并且不需要访问源码。你可以将基础设施作为代码系统来进行调度，从而使你可以通过他们的网页平台监视和控制所有流水线。然而，该平台缺少一些 DevOps 流程，比如源码管理和安全测试。

13、Zuul
-------

![](https://mmbiz.qpic.cn/mmbiz_png/FE4VibF0SjfNJ4OxL7zyb1UrbCNAK9hIiaA8iav3sG5pHQAKGGOucdAFVCwNOrQbPHs6v2ln6mwicScibJsbCNPRtQQ/640?wx_fmt=png)

图片

> 项目地址：https://zuul-ci.org/

Zuul 是开源 CI 工具，主要解决 Jenkins 在 CI 测试中的问题，提供以最快的速度测试序列化的未来状态的能力。主要差异是，它可以测试多个仓库的代码，以确保如果某个变更破坏当前项目或其他项目，则不让该变更传递到生产环境中，称为 co-gating。

多年来，Zuul 已经成为自动合并、构建和测试项目变更的工具。对于企业用户来说，它是构建大量必须彼此同步工作的项目的理想选择。

14、结论
-----

很多开发团队仍在使用 Jenkins，然而它不再是唯一的 CI 工具。不断改进工作方式，会有多种方法让你更轻松、更快、更一致地完成工作。固守传统或忽视创新，将失去竞争优势。

> _作者 | Marija Naumovska_ 
> 
> _策划 | 田晓旭_ 
> 
> _原文 | dzone.com/articles/13-jenkins-alternatives-for-continuous-integration_