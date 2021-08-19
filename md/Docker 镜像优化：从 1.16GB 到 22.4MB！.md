> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA4NzQ0Njc4Ng==&mid=2247501874&idx=2&sn=5fe6cd6c510d67ff8ebd49abaf3077e0&chksm=903bcc5fa74c4549fda45c20eb89c6c22486b5592092fc90338749fe7afd5483350fb083df32&mpshare=1&scene=1&srcid=0819MQmtumeGLIW1QCLajree&sharer_sharetime=1629351564848&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> 作者：ai52learn
> 
> 来源：update.blog.csdn.net/article/details/112816434

Docker 简介
---------

Docker 是一个供软件开发人员和系统管理员使用容器构建、运行和与分享应用程序的平台。容器是在独立环境中运行的进程，它运行在自己的文件系统上，该文件系统是使用 docker 镜像构建的。镜像中包含运行应用程序所需的一切（编译后的代码、依赖项、库等等）。镜像使用 Dockerfile 文件定义。

术语 dockerization 或 containerization 通常用于定义创建 Docker 容器的过程。

因为容器具备如下优点，所以很受欢迎：

*   灵活性：即使是最复杂的应用程序也可以容器化。
    
*   轻量化：容器共享主机内核，使得它们远比虚拟机高效。
    
*   便携性：可以做到本地编译，到处运行。
    
*   松耦合：容器自我封装，一个容器被替换或升级不会打断别的容器。
    
*   安全性：容器对进程进行了严格的限制和隔离，而无需用户进行任何配置。
    

在这篇文章中，我将重点讨论如何优化 Docker 镜像以使其轻量化。

优化过程
----

让我们从一个示例开始，在该示例中，我们构建了一个 React 应用程序并将其容器化。运行 npx 命令并创建 Dockerfile 之后，我们得到了如图 1 所示的文件结构。

```
npx create-react-app app --template typescript
```

![](https://mmbiz.qpic.cn/mmbiz_png/FE4VibF0SjfNhVTJicyGKy6EqEvNvvuU1DUK7kLSZ5q48pwMC9mXWicciboNPVXQO57ncic1a6fZNtOczLDZDlw5niaA/640?wx_fmt=png)

图 1：文件结构

如果我们构建一个基础的 Dockerfile（如下所示），我们最终会得到一个 1.16 GB 的镜像：

```
FROM node:10 
WORKDIR /app
COPY app /app
RUN npm install -g webserver.local
RUN npm install && npm run build 
EXPOSE 3000
CMD webserver.local -d ./build
```

![](https://mmbiz.qpic.cn/mmbiz_png/FE4VibF0SjfNhVTJicyGKy6EqEvNvvuU1Dv9FefxBogabibZpwR0DEfnmQ9kRTEzSGVSRBDx3REKoR2gbSBzJUIxg/640?wx_fmt=png)

图 2：镜像的初始大小为 1.16GB

### 第一步优化：使用轻量化基础镜像

在 Docker Hub（公共 Docker 仓库）中，有一些镜像可供下载，每个镜像都有不同的特征和大小。

通常，相较于基于其他 Linux 发行版（例如 Ubuntu）的镜像，基于 Alpine 或 BusyBox 的镜像非常小。这是因为 Alpine 镜像和类似的其他镜像都经过了优化，其中仅包含最少的必须的软件包。在下面的图片中，你可以看到 Ubuntu、Alpine、Node 和基于 Alpine 的 Node 镜像之间的大小比较。

![](https://mmbiz.qpic.cn/mmbiz_png/FE4VibF0SjfNhVTJicyGKy6EqEvNvvuU1DyPEvO56AAVfr0EtZZEDRfm3SfrfUnk0ibGnNGToUY3xmON9bjX4lW7Q/640?wx_fmt=png)

图 3：基础镜像的不同大小

通过修改 Dockerfile 并使用 Alpine 作为基础镜像，我们的镜像最终大小为 330MB：

```
FROM node:10-alpine 
WORKDIR /app
COPY app /app
RUN npm install -g webserver.local
RUN npm install && npm run build 
EXPOSE 3000
CMD webserver.local -d ./build
```

![](https://mmbiz.qpic.cn/mmbiz_png/FE4VibF0SjfNhVTJicyGKy6EqEvNvvuU1DPAmvleurGS40GKxvwVDHJyyibSFry0TRib8SAEB60PScUic28cxcffOWg/640?wx_fmt=png)

图 4：经过第一步优化后镜像大小为 330MB

### 第二步优化：多阶段构建

通过多阶段构建，我们可以在 Dockerfile 中使用多个基础镜像，并将编译成品、配置文件等从一个阶段复制到另一个阶段，这样我们就可以丢弃不需要的东西。

在本例中，我们部署 React 应用程序需要的是编译后的代码，我们不需要源文件，也不需要 node_modules 目录和 package.json 文件等。

通过将 Dockerfile 修改为如下内容，我们最终得到的镜像大小为 91.5MB。请记住，来自第一阶段（第 1-4 行）的镜像不会被自动删除，Docker 将它保存在 cache 中，如果我们在另一个构建镜像过程中执行了相同的阶段，就可以使镜像构建更快。所以你必须手动删除第一阶段镜像。

```
FROM node:10-alpine AS build
WORKDIR /app
COPY app /app
RUN npm install && npm run build  
FROM node:10-alpineWORKDIR /app
RUN npm install -g webserver.local
COPY --from=build /app/build ./build
EXPOSE 3000
CMD webserver.local -d ./build
```

![](https://mmbiz.qpic.cn/mmbiz_png/FE4VibF0SjfNhVTJicyGKy6EqEvNvvuU1D332AWw1XrXW5FGQX9wb3EticY6qtlZO5u2cwzj7aEDVfNfLm2oNHVPg/640?wx_fmt=png)

图 5：第二步优化后的镜像大小为 91.5MB

现在我们有了一个 Dockerfile，它有两个阶段：在第一个阶段中，我们编译项目，在第二个阶段中，我们在 web 服务器上部署应用程序。然而，Node 容器并不是提供网页（HTML、CSS 和 JavaScript 文件、图片等）服务的最佳选择，最好的选择是使用像 Nginx 或 Apache 这样的服务。在本例中，我将使用 Nginx。

通过将 Dockerfile 修改为如下内容，我们的镜像最终大小是 22.4MB，如果我们运行这个容器，我们可以看到网页可以正常工作，没有任何问题（图 7）。

```
FROM node:10-alpine AS build
WORKDIR /app
COPY app /app
RUN npm install && npm run build  
FROM nginx:stable-alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

![](https://mmbiz.qpic.cn/mmbiz_png/FE4VibF0SjfNhVTJicyGKy6EqEvNvvuU1DvnhQ4ViaYaiaZNUPwODZx51m9pVW6kO9sflrTwD57TuWMgvvN1ibCNlNg/640?wx_fmt=png)

图 6：第三步优化后的镜像大小为 22.4MB