> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU0OTU5OTI4MA==&mid=2247500941&idx=1&sn=c69c9fcdabae5ab66da3e0d7c42e8081&chksm=fbafe5d2ccd86cc4c334c3d3dc3e777b6caa143303a4524940acf5f9d24a4fcdeab26d4c1171&mpshare=1&scene=1&srcid=0727X2IZZCJ6GRZsVbYKY0wK&sharer_sharetime=1627373626511&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

文 | 豆豆

来源：Python 技术「ID: pythonall」

一般我们创建 Python 项目的时候都会创建一个虚拟环境，这样做的好处就是会把项目环境和操作系统环境区分开来，避免把操作系统环境弄乱。  

还有一个痛点就是我们在开发环境开发完成之后，需要把代码复制到生产环境上线，这时候我不想将开发环境的所有包重新在 pip install 一次了，怎么办？

于是 pipenv 应运而生。见名识意，pipenv 就是 pip 和 virtualenv 的结合体。

安装
--

直接使用 pip3 进行安装即可。

```
pip3 install pipenv
```

创建虚拟环境
------

```
$ mkdir demo
$ cd demo
$ pipenv install
```

![](https://mmbiz.qpic.cn/mmbiz_png/SAy0yVjKWyxUia7HHSqsMoo0hh39NRFiaFQZElJWsjqVmvXHQ4nicfygnQ3GSibJ1JG4vJTXdrZlkLuSyxrz6RIuicw/640?wx_fmt=png)

安装完成之后会在你的项目目录自动生成 Pipfile 和 Pipfile.lock 两个文件，他们主要是用来管理包的。不信，我们用 pipenv 安装下 requests 库试一下。

```
$ pipenv install requests
```

咱们来看下 Pipfile 和 Pipfile.lock 的内容。

```
# Pipfile

[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[packages]
requests = "*"

[dev-packages]

[requires]
python_version = "3.8"
```

```
# Pipfile.lock

{
省略部分信息
"default": {
    "requests": {
        "hashes": [
            "sha256:27973dd4a...",
            "sha256:c210084e3..."
        ],
        "index": "pypi",
        "version": "==2.25.1"
    },
省略部分信息
},
"develop": {}
}
```

Pipfile 列出了 requests 库的信息和 Python 版本信息，细心的你可能发现了，该文件中还有一个 dev-packages 的信息，安装时如果指定 -dev 参数，那么就会记录在 dev-packages 下面。而 Pipfile.lock 则保存了库的哈希值，这是确保生产环境和开发环境库信息一致的关键。

当你把项目从开发环境复制到生产环境之后，只需要执行 `pipenv install` 就可以了，无需在重新安装之前在开发环境安装的包了，是不是很省心。

其他命令
----

进入虚拟环境：

```
$ pipenv shell
```

退出虚拟环境：

```
$ exit
```

安装库：

```
$ pipenv install xxx
```

删除库：

```
# 删除指定库
$ pipenv uninstall xxx

# 删除所有库
$ pipenv uninstall --all
```

升级库：

```
$ pipenv update
```

查看库的具体信息：

```
$ pipenv open xxx
```

获取本地工程路径：

```
$ pipenv --where
```

获取虚拟环境路径：

```
$ pipenv --venv
```

检查库的依赖关系，这个非常有用。

```
$ pipenv graph
```

检查库的安全性：

```
$ pipenv check
```

删除虚拟环境：

```
$ pipenv --rm
```

总结
--

今天我们介绍了 Python 虚拟环境 pipenv 的使用，好的工具可以事半功倍，希望对小伙伴们有所帮助。

——END——