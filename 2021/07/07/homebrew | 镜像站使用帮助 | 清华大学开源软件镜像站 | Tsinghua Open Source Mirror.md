> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mirrors.tuna.tsinghua.edu.cn](https://mirrors.tuna.tsinghua.edu.cn/help/homebrew/)

> 清华大学开源软件镜像站的 homebrew 使用帮助。

Homebrew / Linuxbrew 镜像使用帮助
---------------------------

**注：该镜像是 Homebrew / Linuxbrew 源程序以及 formula / cask 索引的镜像（即 `brew update` 时所更新内容）。本镜像站同时提供相应的二进制预编译包的镜像，请参考 [Homebrew bottles 镜像使用帮助](https://mirrors.tuna.tsinghua.edu.cn/help/homebrew-bottles/) 和 [Linuxbrew bottles 镜像使用帮助](https://mirrors.tuna.tsinghua.edu.cn/help/linuxbrew-bottles/)。**

### 首次安装 Homebrew / Linuxbrew

首先，需要确保系统中安装了 bash、git 和 curl，对于 macOS 用户需额外要求安装 Command Line Tools (CLT) for Xcode。

*   对于 macOS 用户，系统自带 bash、git 和 curl，在命令行输入 `xcode-select --install` 安装 CLT for Xcode 即可。
*   对于 Linux 用户，系统自带 bash，仅需额外安装 git 和 curl。

接着，在终端输入以下几行命令设置环境变量：

```
if [[ "$(uname -s)" == "Linux" ]]; then BREW_TYPE="linuxbrew"; else BREW_TYPE="homebrew"; fi
export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git"
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/${BREW_TYPE}-core.git"
export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/${BREW_TYPE}-bottles"


```

最后，在终端运行以下命令以安装 Homebrew / Linuxbrew：

```
# 从本镜像下载安装脚本并安装 Homebrew / Linuxbrew
git clone --depth=1 https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/install.git brew-install
/bin/bash brew-install/install.sh
rm -rf brew-install

# 也可从 GitHub 获取官方安装脚本安装 Homebrew / Linuxbrew
/bin/bash -c "$(curl -fsSL https://github.com/Homebrew/install/raw/master/install.sh)"


```

这样在首次安装的时候也可以使用镜像。更多信息请参考 [Homebrew 官方安装文档](https://docs.brew.sh/Installation)。

_* 安装成功后需将 brew 程序的相关路径加入到环境变量中：_

*   _以下针对基于 Apple Silicon CPU 设备上的 macOS 系统（命令行运行 `uname -m` 应输出 `arm64`）上的 Homebrew：_
    
    ```
    test -r ~/.bash_profile && echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.bash_profile
    test -r ~/.zprofile && echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
    
    
    ```
    
    _对基于 Intel CPU 设备上的 macOS 系统（命令行运行 `uname -m` 应输出 `x86_64`）的用户可跳过本步。_
    
*   _以下针对 Linux 系统上的 Linuxbrew：_
    
    ```
    test -d ~/.linuxbrew && eval "$(~/.linuxbrew/bin/brew shellenv)"
    test -d /home/linuxbrew/.linuxbrew && eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
    test -r ~/.bash_profile && echo "eval \"\$($(brew --prefix)/bin/brew shellenv)\"" >> ~/.bash_profile
    test -r ~/.profile && echo "eval \"\$($(brew --prefix)/bin/brew shellenv)\"" >> ~/.profile
    test -r ~/.zprofile && echo "eval \"\$($(brew --prefix)/bin/brew shellenv)\"" >> ~/.zprofile
    
    
    ```
    
    _参考了 [https://docs.brew.sh/Homebrew-on-Linux](https://docs.brew.sh/Homebrew-on-Linux)。_
    

### 替换现有仓库上游

替换 brew 程序本身的源，Homebrew / Linuxbrew 相同：

```
git -C "$(brew --repo)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git


```

以下针对 macOS 系统上的 Homebrew：

```
# 手动设置
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask.git
git -C "$(brew --repo homebrew/cask-fonts)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask-fonts.git
git -C "$(brew --repo homebrew/cask-drivers)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask-drivers.git
git -C "$(brew --repo homebrew/cask-versions)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask-versions.git

# 或使用下面的几行命令自动设置
BREW_TAPS="$(brew tap)"
for tap in core cask{,-fonts,-drivers,-versions}; do
    if echo "$BREW_TAPS" | grep -qE "^homebrew/${tap}\$"; then
        # 将已有 tap 的上游设置为本镜像并设置 auto update
        # 注：原 auto update 只针对托管在 GitHub 上的上游有效
        git -C "$(brew --repo homebrew/${tap})" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-${tap}.git
        git -C "$(brew --repo homebrew/${tap})" config homebrew.forceautoupdate true
    else   # 在 tap 缺失时自动安装（如不需要请删除此行和下面一行）
        brew tap --force-auto-update homebrew/${tap} https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-${tap}.git
    fi
done


```

以下针对 Linux 系统上的 Linuxbrew：

```
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/linuxbrew-core.git


```

更换上游后需重新设置 git 仓库 HEAD：

### 复原仓库上游

_(感谢 Snowonion Lee 提供说明)_

```
# brew 程序本身，Homebrew / Linuxbrew 相同
git -C "$(brew --repo)" remote set-url origin https://github.com/Homebrew/brew.git

# 以下针对 macOS 系统上的 Homebrew
BREW_TAPS="$(brew tap)"
for tap in core cask{,-fonts,-drivers,-versions}; do
    if echo "$BREW_TAPS" | grep -qE "^homebrew/${tap}\$"; then
        git -C "$(brew --repo homebrew/${tap})" remote set-url origin https://github.com/Homebrew/homebrew-${tap}.git
    fi
done

# 以下针对 Linux 系统上的 Linuxbrew
git -C "$(brew --repo homebrew/core)" remote set-url origin https://github.com/Homebrew/linuxbrew-core.git

# 重新设置 git 仓库 HEAD
brew update-reset


```