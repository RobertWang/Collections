> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zicowarn.github.io](https://zicowarn.github.io/2020/09/22/0809-python-hwoto-install-pyevn-virtualenv/)

> 作者: Jim Wang 公众号: 巴博萨船长 摘要：为何需要使用 pyenv 来管理 python 的版本？为何需要使用 virtualenv 来创建 python 虚拟运行环境？pyenv 与 pipenv 和 virtualenv 的关系是什么样的？在自己使用 pyenv 的时候遇见过什么样的问题，又是如何解决所遇问题的。

作者: Jim Wang 公众号: [巴博萨船长](#wechat)

### [](#背景 "背景")背景

Python 的版本管理非常有必要，这种想法在第一次接触到 python3 的时候就开始有的想法。但是工作中所遇项目主要是在 windows 系统下，开发环境并不复杂，又使用 Eclipse 的项目设定结合 windows 的环境变量。所以也一直没有去尝试着安装配置 pyevn，但是自己工作之余学些 aws lambda 函数时，就遇到了需要隔离 python 版本与创建虚拟运行环境的使用场景，最终还是逃不过这部分内容，因此也就有了这篇网文。

### [](#什么是pyenv和为什么要版本隔离 "什么是pyenv和为什么要版本隔离")什么是 pyenv 和为什么要版本隔离

Pyenv 主要是用于 Python 版本隔离，所谓版本隔离，就是同时安装与使用不同版本的 Python 在一个主机里，且彼此之间没有交叉也不相互影响。在没有 Pyenv 之前，在 windows 系统下也可以实现多 python 版本共存，可以使用环境变量进行隔离。在 Linux 系统下则可以使用 update-alternatives 这一系统命令链接符工具来进行版本管理。

那为什么需要版本管理。假设你有两个 Python 项目，一个是 Python2.x 另一个是 Python3.x，这是你就可以考虑版本隔离了，或者需要同时测试，管理很多不同版本的 Python 项目，也可以考虑版本隔离。

### [](#什么是virtualenv和为什么需要虚拟环境 "什么是virtualenv和为什么需要虚拟环境")什么是 virtualenv 和为什么需要虚拟环境

virtualenv 主要是为了创建 Python 运行的虚拟环境。所谓虚拟环境，就是 Python 运行时，库依赖性的隔离。比如项目一需要 A 依赖包，项目二需要 B 依赖包。传统习惯是，将 A 包与 B 包都使用 pip install 命令进行安装，就可以同时完成项目一与项目二的开发工作，但是这在开发时是没问题的，但在部署时就不一定了，如果部署环境与开发环境不一致，就可能需要花时间去从新安装依赖包，如果部署平台与开发平台也不一致，或者部署环境中的存储限制就需要在开发的时候创建一个虚拟环境，创建一个最小依赖的开发环境。

常见的一些定义 virtualenv 和 pyvenv（同 venv）都是用于创建虚拟运行环境，实现库依赖性的隔离。区别是前者支持 Python 不同版本，可以为不同 python 创建虚拟环境，而后者仅支持 python3.x，venv 仅支持 python3.6 以后不的版本。另一点区别是，virtualenv 是 PyPI 软件包，pyvenv（同 venv）是标准库。

### [](#什么是pipenv和什么是依赖管理 "什么是pipenv和什么是依赖管理")什么是 pipenv 和什么是依赖管理

另一些常见的定义还有 pipevn，pipenv 可以视为 pip 命令和 virtualenv 命令的组合命令。virtualenv 创建的虚拟环境仅仅实现了 Python 库的依赖性隔离。但是一个虚拟环境下随着项目功能的增加，依赖包的变化，一段时间之后这个虚拟环境下第三方库的依赖具体什么样，就很难弄明白了。pipenv 就是为了解决这一问题而存在的。这点很像 nodejs 的依赖包管理工具 npm（或 cnpm）。使用 Pipenv 时，Pipenv 会自动管理 python 库依赖。Pipenv 会在创建虚拟环境时自动创建 Pipfile 和 Pipfile.lock 文件，之后在安装和卸载第三方依赖包的时候，这两个文件都会得到更新被维护。Pipfile 记录项目依赖包列表，Pipfile.lock 记录固定版本的详细依赖包列表。

### [](#其他相关定义 "其他相关定义")其他相关定义

virtualenvwrapper 是对 virtualenv 命令的扩展。pyenv-virtualenv 是 pyenv 的插件，主要用于确保 pyenv 和 virtualenv 能够同时使用。如果使用的是 python3.3 以上的版本，则默认 python 自带 venv，pyenv-virtualenv 插件就是可选内容。pyenv-virtualenvwrapper 也是 pyenv 的插件，主要是为了是 pyenv 中能集成 virtualenvwrapper。

### [](#安装pvenv "安装pvenv")安装 pvenv

本文的安装环境为 mac os Catalina 版本为 10.5.5。在终端输入一下命令, pyenv 目前不支持 windows （windows 可以使用 pyenv-win 来代替），只支持 mac 和 linux。官方提供了一个安装脚本，安装起来非常简单，它会自动安装`pyenv`和`pyenv-virtualenv`。

```
curl -L https://github.com/pyenv/pyenv-installer/raw/master/bin/pyenv-installer | bash
```

也可使用 git 命令克隆 pyenv 源代码至本地，命令如下。

```
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
```

完成上述步骤就需要修改环境变量了，打开 bash 配置文件. bash_profile 或者. bashrc，在文档末尾添加如下内容，

```
export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

Mac 环境下可以尝试直接使用下列命令来进行安装，需要注意的是，使用 brew 安装 pyenv 是不需要进行环境配置的，brew 在安装 pyenv 时已经自动设定了相关内容。

```
brew install pyenv
```

### [](#配置pyenv "配置pyenv")配置 pyenv

完成安装知乎，配置部分的内容其实很简单，仅需要在 bash 配置文件中添加如下内容，完成 pyenv 的初始化即可。

```
eval "$(pyenv init -)"
```

### [](#使用pyenv "使用pyenv")使用 pyenv

pyenv 提供了 11 个可用命令，可以参考[文档](https://github.com/pyenv/pyenv/blob/master/COMMANDS.md)来查看具体内容，在这里简单介绍几个常用命令。

##### [](#pyenv-versions "pyenv versions")pyenv versions

用于检查当前系统中已经安装的 python 版本，命令输出如下内容，带星号为当前系统默认使用的 python 版本。

```
username@mac~ % pyenv versions         
* system (set by /Users/username/.pyenv/version)
  2.7.10
  2.7.15
```

##### [](#pyenv-version "pyenv version")pyenv version

用于查看当前系统默认使用的 python 版本，命令输出如下内容，

```
username@mac ~ % pyenv version 
system (set by /Users/username/.pyenv/version)
```

##### [](#pyevn-install-–list "pyevn install –list")pyevn install –list

用于检查，所有可安装的 python 版本，输出内容如下，

```
username@mac ~ % pyenv install --list
Available versions:
  2.1.3
  2.2.3
  2.3.7
  2.4.0
  2.4.1
```

输出列表里纯数字的表示官方版本，其他的版本注释如下：

*   activepython (ActiveState 公司) 主要面向企业用户与数据科学家，可以节省大量精力在 Python 的组装与管理方面
*   anaconda (Anaconda 公司) 主要用例包括数学、统计学、工程、数据分析、机器学习以及其他相关应用
*   ironPython 属于一套立足. Net 运行时——或者 CLR（公共语言运行时）——的 Python 实现方
*   jython 能够将 Python 2.x 编译为 JVM 字节码，并在 JVM 上运行生成的程序，其仅支持 Python 2.x
*   pypy 属于 CPython 解释器的替代品，其利用即时（JIT）编译以加速 Python 程序的执行
*   micropython 是 Python 3 的一个完整软件实现，用 C 语言编写，被优化于运行在微控制器之上， 如 Arduino 等
*   miniconda 只包含最基本的内容：python 与 conda，以及相关的必须依赖项，是 conda 的最小依赖开发环境
*   stackless 是 python 的一个增强版本，是 python 协程的一个实现。 使用它可以避免创建线程所引起的不必要的开销， 同时也可以实现无锁编程。

##### [](#python-install-3-7-0 "python install 3.7.0")python install 3.7.0

该命令意指安装 Python 3.7.0 的官方版本版本，该命令输出如下内容，

```
username@mac ~ % pyenv install 3.7.0  
...
python-build: use openssl@1.1 from homebrew
python-build: use readline from homebrew
Installing Python-3.7.0...
python-build: use readline from homebrew
python-build: use zlib from xcode sdk
Installed Python-3.7.0 to /Users/username/.pyenv/versions/3.7.0
```

##### [](#python-uninstall "python uninstall")python uninstall

该命令用于卸载特定版本的 python

##### [](#python-rehash "python rehash")python rehash

在安装新一个版本的 python 后可以执行此命令，使其 pyenv 更新自己要管理的所有 python 版本的相关信息。

### [](#卸载pyenv "卸载pyenv")卸载 pyenv

pyenv 的卸载非常简单，删除. pyenv 目录与 bash 配置文件的相关内容即可。

### [](#更新pyenv "更新pyenv")更新 pyenv

在 mac 中，可以使用 brew upgrade pyenv 来完成 pyenv 的更新，也可以通过从 github 中克隆新版本的源码到本地安装目录完成相应更新工作。

### [](#安装pyenv-virtualenv "安装pyenv-virtualenv")安装 pyenv-virtualenv

在完成 python 版本的隔离之后，就可以尝试实现项目依赖的隔离，这就需要借助 pyenv-virtualenv 来完成，安装 pyenv-virtualenv 也有两种方法，在 mac 中使用 brew 安装或者，从 github 中克隆源码到本地。

mac 中使用 brew install pyenv-virtualenv 命令则可以完成安装，该命令的输出内容如下，按照输出内容的提示，我们需要在完成安装之后对其进行配置。

```
username@mac ~ % brew install pyenv-virtualenv
==> Downloading https://github.com/pyenv/pyenv-virtualenv/archive/v1.1.5.tar.gz
==> Downloading from https://codeload.github.com/pyenv/pyenv-virtualenv/tar.gz/v

==> ./install.sh
==> Caveats
To enable auto-activation add to your profile:
  if which pyenv-virtualenv-init > /dev/null; then eval "$(pyenv virtualenv-init -)"; fi
==> Summary
 /usr/local/Cellar/pyenv-virtualenv/1.1.5: 22 files, 65.7KB, built in 2 seconds
```

从 github 中克隆源码到本地的命令如下，这里需要注意的是，需要在当前用户的 home 目录运行该命令，因为源码的本地目录为 pyenv 的子目录，如有必要也需要手动创建子目录 plugins。

```
git clone https://github.com/pyenv/pyenv-virtualenv.git .pyenv/plugins/pyenv-virtualenv
```

### [](#配置pyenv-virtualenv "配置pyenv-virtualenv")配置 pyenv-virtualenv

根据 brew install pyenv-virtualenv，完成配置仅需在 bash 配置文件中加入如下内容即可。

```
if which pyenv-virtualenv >/dev/null; then eval "$(pyenv virtualenv-init -)";fi
```

然后可以使用如下命令，更新 bash 配置的修改，

```
source .bash_profile
```

### [](#使用pyenv-virtualenv "使用pyenv-virtualenv")使用 pyenv-virtualenv

pyenv-virtualenv 的具体使用方法，可以参考[文档](https://github.com/pyenv/pyenv-virtualenv/blob/master/README.md)本文简单介绍一下部分命令。

##### [](#pyenv-virtualenv-版本号-虚拟环境名 "pyenv virtualenv 版本号 虚拟环境名")pyenv virtualenv 版本号 虚拟环境名

比如，由于项目中 aws 的 lambda 函数中使用 python 3.6.0 的是最终是需要完成一个 python 3.6.0 的虚拟运行环境，那么就可以使用如下命令，这里版本号为 3.6.0， 虚拟环境名为 env360。

```
username@mac ~ % pyenv virtualenv 3.6.0 env360
Requirement already satisfied: setuptools in /Users/username/.pyenv/versions/3.6.0/envs/env360/lib/python3.6/site-packages
Requirement already satisfied: pip in /Users/username/.pyenv/versions/3.6.0/envs/env360/lib/python3.6/site-packages
```

##### [](#pyenv-virtualenvs "pyenv virtualenvs")pyenv virtualenvs

用于查看当前所有存在的环境名，命令输出内容如下，每个虚拟环境会出现两次，分别为虚拟环境目录的真实目录和目录链接。

```
username@mac ~ % pyenv virtualenvs            
  3.6.0/envs/env360 (created from /Users/username/.pyenv/versions/3.6.0)
  3.8.5/envs/env385 (created from /Users/username/.pyenv/versions/3.8.5)
  env360 (created from /Users/username/.pyenv/versions/3.6.0)
  env385 (created from /Users/username/.pyenv/versions/3.8.5)
```

##### [](#pyenv-activate-虚拟环境名 "pyenv activate 虚拟环境名")pyenv activate 虚拟环境名

用于激活特定名称的虚拟运行环境，命令输出如下内容。激活后有如下提示信息。如提示信息所述，在非虚拟环境下运行 export PYENV_VIRTUALENV_DISABLE_PROMPT=1，则可以在以后激活虚拟环境时，关闭该提示信息。虚拟环境激活之后，命令行提示符部分会有 (env360) 类似字样，说明当前在 env360 的虚拟运行环境下。

```
username@mac ~ % pyenv activate env360
pyenv-virtualenv: prompt changing will be removed from future release. configure `export PYENV_VIRTUALENV_DISABLE_PROMPT=1' to simulate the behavior.
(env360) username@mac ~ %
```

##### [](#pyenv-virtualenv-delete-虚拟环境名-或-pyenv-uninstall-虚拟环境名 "pyenv virtualenv-delete 虚拟环境名 或 pyenv uninstall 虚拟环境名")pyenv virtualenv-delete 虚拟环境名 或 pyenv uninstall 虚拟环境名

这两个命令都是用于删除特定名称的虚拟运行环境，以命令`pyenv virtualenv-delete`为例，输出入如下内容。命令运行后会有提示信息用于确认操作，输入 yes 即可确认操作，完成虚拟运行环境的删除工作。

```
username@mac ~ % pyenv virtualenv-delete env385          
pyenv-virtualenv: remove /Users/username/.pyenv/versions/3.8.5/envs/env385? yes
username@mac ~ %
```

### [](#卸载-pyenv-virtualenv "卸载 pyenv-virtualenv")卸载 pyenv-virtualenv

pyenv-virtualenv 的卸载和 pyenv 的卸载过程类似，删除 bash 配置文件的相关内容，在 pyenv 目录中的删除相对应的 pyenv-virtualenv 子目录即可。

### [](#更新-pyenv-virtualenv "更新 pyenv-virtualenv")更新 pyenv-virtualenv

如图 pyenv 一样，在 mac 中，可以使用 brew upgrade pyenv-virtualenv 来完成 pyenv-virtualenv 的更新，也可以通过从 github 中克隆新版本的源码到本地安装目录完成相应更新工作。

### [](#所遇问题与解决方案 "所遇问题与解决方案")所遇问题与解决方案

#### [](#问题：Ubuntu-下使用pyenv安装3-7-0版本时，遇到的安装错误 "问题：Ubuntu 下使用pyenv安装3.7.0版本时，遇到的安装错误")问题：Ubuntu 下使用 pyenv 安装 3.7.0 版本时，遇到的安装错误

```
root@R021:~
Downloading Python-3.6.12.tar.xz...
-> https://www.python.org/ftp/python/3.6.12/Python-3.6.12.tar.xz
/tmp/python-build.20200916030737.1554 ~
Installing Python-3.6.12...
/tmp/python-build.20200916030737.1554/Python-3.6.12 /tmp/python-build.20200916030737.1554 ~
checking build system type... x86_64-pc-linux-gnu
checking host system type... x86_64-pc-linux-gnu
checking for python3.6... no
checking for python3... python3
checking for --enable-universalsdk... no
checking for --with-universal-archs... no
checking MACHDEP... linux
checking for --without-gcc... no
checking for --with-icc... no
checking for gcc... no
checking for cc... no
checking for cl.exe... cl.exe

BUILD FAILED (Ubuntu 16.04 using python-build 1.2.20-6-gd1ae4a1)

Inspect or clean up the working tree at /tmp/python-build.20200916030737.1554
Results logged to /tmp/python-build.20200916030737.1554.log
```

出现上述问题的原因为，部分 pyenv 所需的内容没有安装，可以尝试使用如下命令来检查和安装 pyenv 的所需组建，然后再使用 pyenv 安装所需 python 版本。

```
sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev
```

#### [](#问题：Mac-下使用pyenv安装3-7-0版本时，遇到的安装错误 "问题：Mac 下使用pyenv安装3.7.0版本时，遇到的安装错误")问题：Mac 下使用 pyenv 安装 3.7.0 版本时，遇到的安装错误

```
username@mac ~ % pyenv install 3.7.0 
Downloading openssl-1.0.2k.tar.gz...
-> https://www.openssl.org/source/openssl-1.0.2k.tar.gz
Installing openssl-1.0.2k...

BUILD FAILED (OS X 10.15.5 using python-build 20180424)


configure: error: C compiler cannot create executables
See `config.log' for more details.
```

上述错误中的关键内容为 C 编译器错误，解决方法。这时候就要检查 xcode comand line 组件是否安装，使用如下命令检测 xcode command line 是否安装，如果有输出内容，则说明已经安装。

```
username@mac ~ % xcode-select -p
/Library/Developer/CommandLineTools
```

如果想检查 xcode command line 的版本，则可以使用如下命令：

```
username@mac ~ % pkgutil --pkg-info=com.apple.pkg.CLTools_Executables
package-id: com.apple.pkg.CLTools_Executables
version: 12.0.0.0.1.1599194153
volume: /
location: /
install-time: 1600823154
groups: com.apple.FindSystemFiles.pkg-group
```

如果没有安装，则需要安装 Xcode command line， 然后重新使用 pyenv 安装所需版本的 python。

#### [](#问题：Mac-终端启动虚拟环境时，遇到错误 "问题：Mac 终端启动虚拟环境时，遇到错误")问题：Mac 终端启动虚拟环境时，遇到错误

在 Mac 上安装完 pyenv 和 pyenv-virtualenv 后，当重新启动终端后，运行 pyenv activate xxx 启动虚拟环境时会遇到如下提示，

```
Failed to deactivate virtualenv.

Perhaps pyenv-virtualenv has not been loaded into your shell properly.
Please restart current shell and try again.
```

检查 bash 的配置`.bash_profile`中内容也没有发现问题，这时候就要查看 Mac 中启动的终端时 bash 还是 zsh。bash 与 zsh 都是 mac 终端自带的 shell 命令解释器，早期 Mac OS 系统默认使用 bash 解释器，在 10.15 之后的系统中官方推荐使用 zsh 解释器。如果当前终端启动的是 zsh，就需要创建 zsh 的配置文件`.zshrc`文件，并在该文件中添加如下相关配置即可，

```
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

当然在 Mac 中 bash 和 zsh 是可以自由切换的，切换方式如下：

*   切换 bash：`chsh -s /bin/bash`
*   切换 zsh：`chsh -s /bin/zsh`
*   **终端**的系统偏好设置里手动设置。

### [](#小结 "小结")小结

以上即为本人在安装和配置 pyenv 和 pyenv-virtualenv 整个过程，也包括自己在不同系统下安装这两个组建所遇问题和找到的解决方案。Python 版本隔离与 Python 开发运行环境隔离，两者哪一个更有必要了解安装？哪种方案最优？个人觉得没必要过多纠结，适合自己需求的就是最合适的。上述方案也仅为一家之言，并非最优，若读者有别的思路也欢迎关注我的个人微信公众号，一起讨论学习。

* * *