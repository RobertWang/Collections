> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/3qxzkgfPXibOFAKMeIrN2g)

> 部署一个 FastAPI 应用到你的服务器是一项复杂的任务。如果你对 NGINX、Gunicorn 和 Uvicorn 这些技术不熟悉，可能会浪费大量的时间。

一、前言

部署一个 **FastAPI** 应用到你的服务器是一项复杂的任务。如果你对 **NGINX**、**Gunicorn** 和 **Uvicorn** 这些技术不熟悉，可能会浪费大量的时间。如果你是刚接触 Python 语言不久或者希望利用 Python 构建自己的 Web 应用程序，本文的内容可能会让你第一次部署时更节省时间。

FastAPI 是用于开发 API 应用最受欢迎的 Python 库之一，用于开发 API。它以其出色的性能和易用性而闻名。如果你在网页应用中使用机器学习模型，那么它很可能是你首选的工具。

NGINX、Gunicorn 和 Uvicorn 都是经过实践验证的技术，常被用作反向代理和 ASGI 服务器来部署 Python 网页应用。如果你熟悉 Django 或 Flask，你可能之前听说过它们中的一些。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/avz34KKHsxrfia7piaibM2dmGcPBZHvf90k6xNLibacia4FPCiaXNX5xPJJTPrE4sdicwuXORHAVGe0X9qhz5Wiax6Pqsw/640?wx_fmt=png)

接下来，我将展示如何结合这些工具来部署一个 FastAPI 网页应用。以下是主要内容：

*   介绍 FastAPI、NGINX、Gunicorn 和 Uvicorn 的基础知识。
    
*   配置 Gunicorn + Uvicorn 作为 ASGI 服务器。
    
*   设置 NGINX 作为反向代理服务器。
    
*   使用 Let's Encrypt 生成免费 SSL 证书。
    

二、技术框架介绍

2.1、FastAPI

**FastAPI** 是一个现代的、高性能的 Web 框架，用于使用 Python 构建 API，并基于标准类型提示。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/avz34KKHsxrfia7piaibM2dmGcPBZHvf90kKk5jBXoNuRV6PfM57v5BwpTDUUmF4x67fccEM85n54V3ic5C04tsV9g/640?wx_fmt=png)

**它具有以下主要特点：**

*   **高效运行**：借助 Starlette 和 pydantic，FastAPI 提供了与 NodeJS 和 Go 类似的出色性能。FastAPI 比 Flask 快得多，它实际上是 Python 最快的 Web 框架之一。唯一比 FastAPI 更快的框架是 Starlette（FastAPI 实际上是基于 Starlette 构建的）。
    
*   **快速开发**：它可以显著提高开发速度。
    
*   **减少错误**：减少了人为错误的可能性。
    
*   **直观易用**：支持强大的编辑器功能、自动补全和更少的调试时间。
    
*   **简单易学**：设计简单明了，您可以花更少时间阅读文档。
    
*   **减少重复代码量**：最大限度地减少了代码重复。
    
*   **健壮可靠**：提供生产就绪的代码和自动生成交互式文档功能。
    
*   **基于标准化**：遵循 API、OpenAPI 和 JSON 模式等开放标准。
    

该框架旨在优化开发人员体验，使您能够以简洁的代码构建具备最佳实践且适合生产环境使用的 API。

2.2、Gunicorn

**Gunicorn** 是用于 Python 应用程序的 WSGI 服务器的一种实现。

**Gunicorn** 是一个符合 **WSGI** 标准的 Web 服务器，用于 Python 应用程序，它接收从**客户端**发送到 Web **服务器**的请求，并将其转发到 Python 应用程序或 **Web 框架**（如 Flask 或 Django）上，以便为请求运行适当的**应用程序**代码。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/avz34KKHsxrfia7piaibM2dmGcPBZHvf90kOuT4FLq1bXhM61vN3Snqv0gGEd1wCicZYDcCReBHicr32XY7jWKUic1Hg/640?wx_fmt=png)

2.3、Uvicorn

与 Flask 框架不同，FastAPI 不包含任何内置的开发服务器。因此，我们需要 **Uvicorn**。它实施 **ASGI** 标准，速度快如闪电。ASGI 代表 **异步服务器网关接口**。

*   Uvicorn 是 Python 的 ASGI Web 服务器实现。
    
*   Uvicorn 目前支持 HTTP / 1.1 和 WebSockets。
    

2.4、Nginx

**Nginx** 是一个异步框架的 Web 服务器，也可以用作反向代理，负载平衡器 和 HTTP 缓存。该软件由 Igor Sysoev 创建，并于 2004 年首次公开发布。同名公司成立于 2011 年，以提供支持。Nginx 是一款免费的开源软件，根据类 BSD 许可证的条款发布。一大部分 Web 服务器使用 Nginx ，通常作为负载均衡器。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/avz34KKHsxrfia7piaibM2dmGcPBZHvf90kTzc3X3Z8FD1kHvJPiaTAicwjSLoMIM1HQNOsiaoStJCV2GMmh5ZFJsuxg/640?wx_fmt=png)

**Nginx 具备以下特点：**

*   **更快**：单次请求会得到更快的响应；在高并发环境下，Nginx 比其他 WEB 服务器有更快的响应。
    
*   **高扩展性**：Nginx 是基于模块化设计，由多个耦合度极低的模块组成，因此具有很高的扩展性。许多高流量的网站都倾向于开发符合自己业务特性的定制模块。
    
*   **高可靠性**：Nginx 的可靠性来自于其核心框架代码的优秀设计，模块设计的简单性。另外，官方提供的常用模块都非常稳定，每个 worker 进程相对独立，master 进程在一个 worker 进程出错时可以快速拉起新的 worker 子进程提供服务。
    
*   **低内存消耗**：一般情况下，10000 个非活跃的 `HTTP Keep-Alive` 连接在 Nginx 中仅消耗 `2.5MB` 的内存，这是 Nginx 支持高并发连接的基础；单机支持 10 万以上的并发连接：理论上，Nginx 支持的并发连接上限取决于内存，10 万远未封顶。
    
*   **热部署**：master 进程与 worker 进程的分离设计，使得 Nginx 能够提供热部署功能，即在 7x24 小时不间断服务的前提下，升级 Nginx 的可执行文件。当然，它也支持不停止服务就更新配置项，更换日志文件等功能。
    
*   **最自由的 BSD 许可协议**：这是 Nginx 可以快速发展的强大动力。BSD 许可协议不只是允许用户免费使用 Nginx ，它还允许用户在自己的项目中直接使用或修改 Nginx 源码，然后发布。
    

![](https://mmbiz.qpic.cn/mmbiz_png/fnGIZJCNaLLjSbe7mU2kuwhib59a8rRHWE8D8NzJiaVGS1JsrnBBQIpqibnVH3HbOqTr0vTTgr48Ndu4By534qwog/640?wx_fmt=png)

三、服务器安全设置

如果你的应用部署在局域网，你可能暂时不需要关注这块，但如果你的应用是部署在云服务器或者 VPS，为了保障你的应用安全性，建议还是需要对服务器做一些必要性的安全设置，避免被攻击。

3.1、启用自动更新

首先，确保您的服务器拥有最新的软件：

```
sudo apt update && sudo apt upgrade -y

```

```
以下是在使用基于Debian的服务器时会看到的常见命令：

```

1.  使用命令 **`sudo apt update`** 来更新系统上的软件包列表索引。
    
2.  使用命令 **`sudo apt upgrade -y`** 将已安装的软件包升级到最新版本。使用**`-y`**标志可以跳过确认步骤，直接进行安装。
    

接下来，设置自动安全更新，这样就不必手动执行更新。为此，您需要安装并启用 **`unnattended-upgrades`**：

```
sudo apt install unattended-upgrades

```

安装完成后，编辑文件 **`/etc/apt/apt.conf.d/20auto-upgrades`** 并按照以下配置进行设置：

```
APT::Periodic::Update-Package-Lists "1";  # 每天自动更新软件包列表
APT::Periodic::Unattended-Upgrade "1";    # 系统将自动升级到最新版本的软件包
APT::Periodic::AutocleanInterval "7";     # 每周运行一次自动清理操作，删除旧的和不必要的包文件

```

最后，在文件 _**`/etc/apt/apt.conf.d/50unattended-upgrades`**_ 中进行编辑，以确保系统在需要内核更新时自动重新启动：

```
Unattended-Upgrade::Automatic-Reboot "true";

```

完成以上步骤后，您的系统将自动执行安全更新，并在需要时重新启动以应用内核更新。

3.2、创建非 root 用户

为了减少黑客攻击造成的损害，创建非 root 用户是必要的。以下是一些命令来创建非 root 用户和设置 SSH 密钥登录：

*   创建一个新的非 root 用户：
    

```
sudo adduser fastapi-user # 将fastapi-user替换为您喜欢的用户名

```

*   将该用户添加到 sudo 组以获得管理员权限：
    

```
sudo usermod -aG sudo fastapi-user # 将fastapi-user替换为您创建的用户名

```

*   使用新创建的用户登录服务器：
    

```
su - fastapi-user # 使用新用户登录，将fastapi-user替换为您创建的用户名

```

*   设置使用 SSH 密钥进行身份验证：
    

如果您还没有 SSH 密钥，请在本地计算机上打开终端并运行以下命令（请注意将 your-email-replace-with-your-email 替换为您的实际电子邮件）：

```
ssh-keygen -t ed25519 -C "your-email-replace-with-your-email"

```

```
这将生成一对SSH密钥。

```

复制公共 SSH 密钥并将其粘贴到远程服务器上（请注意将 your-public-key-replace-with-your-public-key 替换为实际公共密钥）：

```
echo "your-public-key-replace-with-your-public-key" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

```

*   禁用 root 登录和密码身份验证：
    

使用 sudo 权限编辑**`/etc/ssh/sshd_config`**文件

（例如使用**`sudo vim /etc/ssh/sshd_config`**命令）。

将以下行的值更改为所示：

```
PermitRootLogin no
PasswordAuthentication no

```

```
这将禁止root用户登录和使用密码进行SSH身份验证。

```

*   保存并关闭文件后，重新启动 SSH 服务以使更改生效：
    

```
sudo service ssh restart

```

一旦完成上述步骤，请使用新创建的非 root 用户和 SSH 密钥进行登录，并确保禁用了 root 登录和密码身份验证。

3.3、其他安全措施

大多数云提供商都提供防火墙服务。如果您的云提供商不提供防火墙服务，您可以手动配置防火墙来限制传入流量只能到达必要的端口，例如 **80**、**443** 和 **22** 端口。

另外，您还可以安装并配置 **fail2ban** 来预防暴力身份验证攻击。**Fail2ban** 是一个用于保护 Linux 服务器免受恶意登录尝试的工具。它监视系统日志文件，并根据设定的规则自动禁止源 IP 地址。

如果您想了解更多关于保护 Linux 服务器的最佳实践，请查看 Linode 提供的相关指南。这些指南将为您提供有关如何设置和增强服务器安全性的详细信息。

四、安装软件工具

为了安装必要的软件工具，请按照以下步骤执行：

4.1、安装 Python：

```
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.11 python3.11-venv -y

```

4.2、安装 Supervisor 和 NGINX：

```
sudo apt install supervisor nginx -y

```

**Supervisor** 是一个用于类 Unix 操作系统（包括 Linux）的进程控制系统，它用于监视和管理程序的进程。NGINX 是一种常用的多功能软件，通常用作反向代理来部署 Web 应用程序。

4.3、启用并启动 Supervisor：

```
sudo systemctl enable supervisor
sudo systemctl start supervisor

```

使用**`enable`**命令将确保 Supervisor 在启动时自动启动，并使用**`start`**命令立即启动 Supervisor 服务。

完成上述步骤后，您已成功安装 Python、Supervisor 和 NGINX，并已启用并启动了 Supervisor 服务。

五、设置 FastAPI 应用程序

首先，将示例应用程序克隆到**`/home/fastapi-user`**目录下：

```
git clone https://github.com/dylanjcastillo/fastapi-nginx-gunicorn

```

这将从公共存储库克隆代码。如果您要部署私有 GitHub 存储库中的应用程序，请设置 GitHub 部署密钥并使用该密钥克隆存储库。

接下来，创建一个虚拟环境并激活它：

```
cd /home/fastapi-user/fastapi-nginx-gunicorn
python3.11 -m venv .venv
source .venv/bin/activate

```

这些命令将使您进入项目目录，并在其中创建一个虚拟环境并激活它。一旦成功激活，您的命令行提示符前面应显示**`.venv`**。

现在，使用 **requirements.txt**  文件中指定的依赖项安装所需的软件包：

```
pip install -r requirements.txt

```

这将在当前虚拟环境中安装 **fastapi**、**gunicorn** 和 **uvicorn** 等软件包。  

为了验证一切是否顺利，运行应用程序：

```
uvicorn main:app

```

运行此命令时不应出现任何错误。您还可以打开新终端窗口，并连接到服务器，然后使用以下命令发出请求来验证它是否正常工作：

```
curl http://localhost:8000

```

您应该会收到以下响应：

```
{"message":"It's working!"}

```

现在您已经成功运行了 FastAPI 应用程序，接下来将配置 Gunicorn 作为 WSGI 服务器。

六、配置 Gunicorn

配置 **Gunicorn** 有两个步骤。首先，明确指定 **Gunicorn** 的配置要求。其次，设置 Supervisor 程序来运行 Gunicorn。

6.1 设置 Gunicorn

首先，在项目目录中创建一个名为 **gunicorn_start** 的文件：

```
vim gunicorn_start

```

然后，将以下内容添加到文件中：

```
#!/bin/bash

NAME=fastapi-app
DIR=/home/fastapi-user/fastapi-nginx-gunicorn
USER=fastapi-user
GROUP=fastapi-user
WORKERS=3
WORKER_CLASS=uvicorn.workers.UvicornWorker
VENV=$DIR/.venv/bin/activate
BIND=unix:$DIR/run/gunicorn.sock
LOG_LEVEL=error

cd $DIR
source $VENV

exec gunicorn main:app \
  --name $NAME \
  --workers $WORKERS \
  --worker-class $WORKER_CLASS \
  --user=$USER \
  --group=$GROUP \
  --bind=$BIND \
  --log-level=$LOG_LEVEL \
  --log-file=-

```

这是您的设定解释：

*   第 1 行表示此脚本将由 bash shell 执行。
    
*   第 3 行至第 11 行指定您将传递给 Gunicorn 的配置选项。大多数参数都是直观的，除了 **WORKERS**、**WORKER_CLASS** 和 **BIND** ：
    

    - **WORKERS**：定义要使用的工作进程数量，通常建议使用 CPU 核心数 + 1。

   - **WORKER_CLASS**：指定要使用的工作进程类型。在此示例中，您指定 Uvicorn Worker 作为 ASGI 服务器。

    - **BIND**：指定 Gunicorn 绑定到的 **server socket**。

*   第 13 行和第 14 行将当前位置更改为项目目录并激活虚拟环境。
    
*   第 16 行至第 24 行使用指定的参数运行 Gunicorn。
    

保存并关闭文件。然后，通过运行以下命令使其可执行：

```
chmod u+x gunicorn_start

```

最后，在项目目录中创建一个文件夹 **run** ，用于存储您在参数中定义的 Unix 套接字文件 BIND：

```
mkdir run

```

6.2 配置 Supervisor

首先，在项目目录中创建一个名为 **logs** 的目录，用于存储应用程序的错误日志：

```
mkdir logs

```

接下来，通过运行以下命令创建 Supervisor 的配置文件：

```
sudo vim /etc/supervisor/conf.d/fastapi-app.conf

```

```
复制并粘贴以下内容到文件中：

```

```
[program:fastapi-app]
command=/home/fastapi-user/fastapi-nginx-gunicorn/gunicorn_start
user=fastapi-user
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/home/fastapi-user/fastapi-nginx-gunicorn/logs/gunicorn-error.log

```

此配置文件指定了之前创建的 **gunicorn_start** 脚本，并设置了 **fastapi-user** 作为用户。Supervisor 将在服务器启动时启动应用程序，并在应用程序失败时重新启动它。错误日志将记录在项目目录下 **logs/gunicorn-error.log** 文件中。

重新加载 Supervisor 配置并重启服务，通过运行以下命令：

```
sudo supervisorctl reread
sudo supervisorctl update

```

最后，您可以通过运行以下命令检查程序的状态：

```
sudo supervisorctl status fastapi-app

```

```
如果一切顺利，fastapi-app 服务的状态应显示为 RUNNING。

```

您还可以通过打开新的终端窗口，连接到服务器，并使用以下命令发出 GET 请求来测试它：

```
curl --unix-socket /home/fastapi-user/fastapi-nginx-gunicorn/run/gunicorn.sock localhost

```

您应该会看到以下输出：

```
{"message":"It's working!"

```

```
最后，如果您对代码进行了更改，可以通过运行以下命令重新启动服务以应用更改：

```

```
sudo supervisorctl restart fastapi-app

```

现在您已经拥有一个使用 Gunicorn 和 Uvicorn 作为 ASGI 服务器的应用程序。接下来，您将使用 NGINX 设置反向代理服务器。

七、配置 NGINX

为您的项目创建一个新的 **NGINX** 配置文件：

```
sudo vim /etc/nginx/sites-available/fastapi-app

```

```
打开NGINX配置文件并粘贴以下内容：

```

```
upstream app_server {
    server unix:/home/fastapi-user/fastapi-nginx-gunicorn/run/gunicorn.sock fail_timeout=0;
}

server {
    listen 80;

    # add here the ip address of your server
    # or a domain pointing to that ip (like example.com or www.example.com)
    server_name XXXX;

    keepalive_timeout 5;
    client_max_body_size 4G;

    access_log /home/fastapi-user/fastapi-nginx-gunicorn/logs/nginx-access.log;
    error_log /home/fastapi-user/fastapi-nginx-gunicorn/logs/nginx-error.log;

    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;
                        
        if (!-f $request_filename) {
            proxy_pass http://app_server;
            break;
        }
        }
}

```

这是 NGINX 配置文件。它的工作原理如下：

*   **第 1 行到第 3 行****`app_server`**定义了一个称为 NGINX 将代理请求的服务器集群。请求被重定向到位于 的 Unix socket file `/home/fastapi-user/fastapi-nginx-gunicorn/run/gunicorn.sock`。设置**`fail_timeout=0`**告诉 NGINX 不要将服务器视为失败，即使它没有响应。
    
*   **第 1 行到第 5 行**定义了 NGINX 将用于处理请求的虚拟服务器的配置。在本例中，它侦听端口 80。将 XXXX 替换为 IP 或站点名称。
    
*   **第 12 行和第 13 行**指定**`keepalive_timeout`**设置客户端可以保持持久连接打开的最长时间，并**`client_max_body_size`**设置 NGINX 允许的客户端请求正文的大小限制。
    
*   **第 15 行和第 16 行**指定 NGINX 将写入其访问和错误日志的位置。
    
*   **第 18 至 27 行**定义了 NGINX 将如何处理对根目录的请求`/`。您提供一些规范来处理标头，并设置一个指令来将请求代理到您**`app_server`**之前定义的。
    

通过运行以下命令从文件创建符号链接来启用站点配置**`sites-available`**：**`sites-enabled`**

```
sudo ln -s /etc/nginx/sites-available/fastapi-app /etc/nginx/sites-enabled/

```

```
测试配置文件是否正常并重启NGINX：

```

```
sudo nginx -tsudo systemctl restart nginx

```

如果一切顺利，现在您应该能够从浏览器或使用`curl`. 您应该再次看到以下输出：

```
{"message":"It's working!"}

```

您现在应该已经运行了 FastAPI 应用程序，并且 Gunicorn + Uvicorn 作为 ASGI 服务器，NGINX 在它们前面作为反向代理。

7.1、权限错误

如果出现权限错误，表示 NGINX 无法访问 Unix socket，您可以将用户 **www-data**（通常是 NGINX 进程运行的用户）添加到 fastapi-user 组中。您可以使用以下命令：

```
sudo usermod -aG fastapi-user www-data

```

如果您还没有为 API 应用购买域名，则不需要往下阅读了。如果您已经有域名，请继续执行下一步以获取 SSL 证书并启用 HTTPS。

八、使用 Certbot 获取免费 SSL 证书

这仅适用于您希望为其获取 SSL 证书的域名。如果您使用的是 **Ubuntu**，则可以跳过此步骤。否则，您首先需要安装 **snapd**：

```
sudo apt install snapd

```

接下来，确保您拥有最新可用版本：

```
sudo snap install core; sudo snap refresh core

```

安装 Certbot 并确保`certbot`命令可执行：

```
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot

```

接下来，通过以交互方式运行以下命令为您的域名生成证书：

```
sudo certbot --nginx

```

最后，Certbot 将自动处理您证书的续订。要测试它是否有效，请运行以下命令：

```
sudo certbot renew --dry-run

```

如果一切顺利，您应该会看到 **Congratulations, all simulated renewals succeeded...** 消息。

现在，您应该能够通过 HTTPS 成功发送请求。

九、结论

本篇文章介绍了如何使用 **NGINX**、**Gunicorn** 和 **Uvicorn** 来部署 FastAPI 应用程序。FastAPI 是最流行的 Python Web 框架之一。它已成为部署机器学习驱动的 Web 应用程序的首选，因此你希望持续关注 AI 领域，并希望能做一定的应用开发实践，熟悉它还是挺有帮助的。

在这篇文章中您了解到：

*   为什么以及**何时应该使用 FastAPI、NGINX、Gunicorn 和 Uvicorn**。
    
*   如何**设置 Gunicorn+Uvicorn 作为 ASGI 服务器**。
    
*   如何**使用 Supervisor 来运行 Gunicorn****。**
    
*   如何**使用 certbot 配置 NGINX 并生成免费的 SSL 证书****。**