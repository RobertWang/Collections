> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [aws.amazon.com](https://aws.amazon.com/cn/blogs/china/demystifying-entrypoint-cmd-docker/)

> 在开始创建 Docker 容器的过程中，您可能会发现自己面临一个十分困惑的问题：Dockerfile 是应包含一条 ENTRYPOINT 说明、一条 CMD 说明，还是两者？ 在本博文中，我将详细讨论这两者的差异，解释如何在您可能遇到的不同使用案例中用好它们。

在开始创建 Docker 容器的过程中，您可能会发现自己面临一个十分困惑的问题：Dockerfile 是应包含一条 `ENTRYPOINT` 说明、一条 `CMD` 说明，还是两者？ 在本博文中，我将详细讨论这两者的差异，解释如何在您可能遇到的不同使用案例中用好它们。

一般原则
----

如果说今天您可以学到_一个经验_ ，那就是要遵循以下**一般原则**：

`ENTRYPOINT` + `CMD` = 默认容器命令参数

但须满足下列条件：

1.  必须都可在运行时上单独覆盖；
2.  任何一个或两者都可为空；并且
3.  其中的加号 (+) 是指 `ENTRYPOINT` 与 `CMD` **在阵列环境中分别**_串联_。

Chamber 简介
----------

为便于说明 `ENTRYPOINT` 的优势，我们推出了 [Chamber](https://github.com/segmentio/chamber)，这是一种开源实用工具，可以使用在 [AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-paramstore.html) 中找到的值来填充容器环境。一种典型的调用是 `chamber exec production -- program`，该函数会获取键前缀为 `/production` 的所有 Parameter Store 值，将键中的斜线转换为下划线，然后使用返回的键和值填充环境。例如，如果找到键 `/production/mysql/password`，则 Chamber 会将 `MYSQL_PASSWORD` 环境变量设置为其中的安全值。

简单示例
----

下面我们来看一个示例。假设 `Dockerfile` 代码段含有 `ENTRYPOINT` 和 `CMD` 并且这两个参数都指定为阵列：

```
ENTRYPOINT ["/bin/chamber", "exec", "production", "--"]
CMD ["/bin/service", "-d"]

```

将这两个参数组合起来，则容器的默认参数将为 `["/bin/chamber", "exec", "production", "--", "/bin/service", "-d"]`。

此列表大致相当于壳命令 `/bin/chamber exec production -- /bin/service -d`。（实际上，这涉及了壳的主要作用：它们提取提示屏幕上以空格分隔的 “命令”，最终将这些命令转换为参数整列，以便发送到 [exec 系统调用](https://linux.die.net/man/2/execve)。）

参数始终为阵列
-------

需要注意的是，在 `Dockerfile` 中，`ENTRYPOINT` 和 `CMD` **始终**将转换为阵列 — 即使您声明它们为字符串。（但为避免歧义，我始终建议将它们声明为阵列。）

假设我们声明一个用于启动 Web 服务器的 `CMD` 如下：

```
CMD /usr/bin/httpd -DFOREGROUND

```

Docker 会自动将 `CMD` 转换为阵列，如下所示：

```
["/bin/sh", "-c", "/usr/bin/httpd -DFOREGROUND"]

```

这同样适用于 `ENTRYPOINT` 参数。

因此，当我们声明 `ENTRYPOINT` 和 `CMD` 时，`ENTRYPOINT` 将是一个列表，这两个参数将组合成为一个默认的参数列表 — 即使我们声明 `CMD` 为字符串。

请看下面的示例。如果我们声明如下：

```
ENTRYPOINT ["/bin/chamber", "exec", "production", "--"]
CMD "/bin/service -d"

```

默认的参数列表将为 `["/bin/chamber", "exec", "production", "--", "/bin/sh", "-c", "/bin/service -d"]`。

**注意：**`**ENTRYPOINT**` **和** `**CMD**` **不能同时为字符串值。** 它们可以都是阵列值，或者 `ENTRYPOINT` 为一个阵列值而 `CMD` 为一个字符串值；但如果 `ENTRYPOINT` 为一个字符串值，则 `CMD` 将被忽略。这是将参数字符串转换为阵列后无法避免的不幸后果。这也是我始终建议尽可能指定阵列的原因之一。

CMD 仅为默认值
---------

指定 `Dockerfile` 中的 `CMD` 仅会创建一个默认值：如果我们将无选项的参数提交到 `docker run`，则它们将被 `CMD` 的值覆盖。

为方便演示，假设我们拥有如下的 `Dockerfile`，并从它创建了一个叫做 `myservice` 的映像：

```
ENTRYPOINT ["/bin/chamber", "exec", "production", "--"]
CMD ["/bin/service", "-d"]

```

如果我们调用 `docker run myservice`，则将创建包含下列参数的容器：

```
["/bin/chamber", "exec", "production", "--", "/bin/service", "-d"]

```

如果我们改为调用 `docker run myservice /bin/debug`，则将创建包含下列参数的容器：

```
["/bin/chamber", "exec", "production", "--", "/bin/debug"]

```

请注意 `CMD` 将被_完全替换_ — 不能在其开头或结尾处添加任何字符。

ENTRYPOINT 也可覆盖
---------------

我可以轻松覆盖在 `Dockerfile` 中声明的 `ENTRYPOINT`。为此我们将指定 `docker run` 的 `--entrypoint` 选项参数。

如前面一样，假设我们拥有如下的 `Dockerfile`，并从它创建了一个叫做 `myservice` 的映像：

```
ENTRYPOINT ["/bin/chamber", "exec", "production", "--"]
CMD ["/bin/service", "-d"]

```

然后让我们通过运行如下命令来修改 `ENTRYPOINT`：

```
docker run --entrypoint /bin/logwrap myservice

```

根据我们的一般原则，将会构建如下参数列表：

```
["/bin/logwrap", "/bin/service", "-d"]

```

同时覆盖 ENTRYPOINT 和 CMD
---------------------

我们能否同时覆盖 `ENTRYPOINT` **和** `CMD`？ 当然可以：

```
docker run --entrypoint /bin/logwrap myservice /bin/service -e

```

对应的参数列表如下 — 到这时应该不会发生任何意外了：

```
["/bin/logwrap", "/bin/service", "-e"]

```

我们何时应该使用 ENTRYPOINT？ CMD 呢？
---------------------------

假设我们为某个项目构建自己的 `Dockerfile`。我们已经了解了 `ENTRYPOINT` 和 `CMD` 如何结合起来构建容器的默认参数列表的_原理_。但现在我们需要知道_如何选择_：何时建议使用 `ENTRYPOINT`，何时又建议使用 `CMD`？

您所做的选择基本上属于一种艺术，它严重依赖您的使用案例。但我的经验是，`ENTRYPOINT` 几乎适合我遇到的所有案例。可以考虑下列使用案例：

### 封套

一些映像包含所谓的 “封套”，将原有的程序进行装饰或者进行其他准备，以便于在容器化环境中应用。例如，假设您的服务设计为从文件读取配置，而非从环境变量读取。则在这种情况下，您可以包含一个将利用环境变量生成配置文件的封套脚本，然后在最后调用 `exec /path/to/app` 以启动应用程序。

声明指向封套的 `ENTRYPOINT` 就是确保封套始终运行的一个极佳方法，而不论将何参数发送到 `docker run`。

### 单用途映像

如果您的映像仅用于执行一个操作（ 例如运行 Web 服务器），则使用 `ENTRYPOINT` 来指定服务器二进制代码和任何强制参数的路径。一个典型示例是 `nginx` 映像，它的唯一用途是运行 nginx Web 服务器。这使它本身拥有一个天然的命令行调用：`docker run nginx`。然后您可以顺理成章地在命令行后添加程序命令，例如 `docker run nginx -c /test.conf`，就好比您在运行无 Docker 的 nginx。

### 多模式映像

支持多种 “模式” 的映像在 `docker run <image>` 上使用第一个参数来指定映射到模式的谓语（例如 `shell`、`migrate` 或 `debug`），也十分常见。对于此类使用案例，我建议 `ENTRYPOINT` 的设置应指向一个将解释动作参数并根据其值执行相关操作的脚本，例如：

```
ENTRYPOINT ["/bin/parse_container_args"]

```

这些参数将在调用时通过 `ARGV[1..n]` 或 `$1`、`$2` 等发送到入口点。

小结
--

Docker 拥有极其强大与灵活的映像构建功能，但在确定如何构建容器的默认运行时参数方面可能极具挑战。但愿本文可以帮助您更加清楚参数 - 汇编机制，以及如何在您的环境中发挥它们的最佳作用。

本篇作者
----