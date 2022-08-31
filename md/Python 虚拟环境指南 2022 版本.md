> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7138022736256843784)*   [Python 虚拟环境指南 2021 版](https://link.juejin.cn?target=https%3A%2F%2Fgame404.github.io%2Fpost%2Fpython-env-2021%2F "https://game404.github.io/post/python-env-2021/")
*   [Python 虚拟环境指南 2020 版](https://link.juejin.cn?target=https%3A%2F%2Fgame404.github.io%2Fpost%2Fpython-env-2020%2F "https://game404.github.io/post/python-env-2020/")
*   [Python 虚拟环境指南 2019 版](https://link.juejin.cn?target=https%3A%2F%2Fgame404.github.io%2Fpost%2Fpython-env-2019%2F "https://game404.github.io/post/python-env-2019/")

去年介绍的 pipenv 和 poetry 发展都还不错，从 github 上看，感觉数据还在伯仲之间。在这两者之外，新出了一个名叫 `PDM(Python Development Master)` 的虚拟环境工具。国人出品，3k 的 star，颜值和文档都还不错，今年就介绍它了。

PDM 介绍
------

mac 上直接使用 `brew install pdm` 安装，其它系统官方提供了一个一键安装的 shell 脚本。安装完成后，创建一个项目目录 _test-pdm_ ，在目录中使用 `pdm init` 命令初始化项目:

```
➜  test-pdm pdm init
Creating a pyproject.toml for PDM...
Please enter the Python interpreter to use
0. /usr/local/opt/python@3.9/bin/python3.9 (3.9)
1. /Library/Developer/CommandLineTools/usr/bin/python3 (3.8)
2. /Library/Frameworks/Python.framework/Versions/Current/bin/python3.8 (3.8)
3. /usr/local/bin/pypy3.7 (3.7)
4. /usr/local/bin/pypy (2.7)
5. /usr/local/Cellar/pdm/2.1.2/libexec/bin/python3.10 (3.10)
Please select (0): 2
Using Python interpreter: /Library/Frameworks/Python.framework/Versions/Current/bin/python3.8 (3.8)
Would you like to create a virtualenv with /Library/Frameworks/Python.framework/Versions/Current/bin/python3.8? [y/n] (y): y
Virtualenv is created successfully at /Users/yoo/tmp/test-pdm/.venv
Is the project a library that will be uploaded to PyPI [y/n] (n): n
License(SPDX name) (MIT):
Author name (game404):
Author email (studyoo@foxmail.com):
Python requires('*' to allow any) (>=3.8):
Changes are written to pyproject.toml.
复制代码

```

`pdm`比较方便的地方是会扫描出系统的 python 解释器，提示用户选择解释器版本, 比如上面日志显示有 5 个 python 版本，有点乱:( 。初始化完成后，生成项目描述文件`pyproject.toml`内容如下:

```
[project]
name = ""
version = ""
description = ""
authors = [
    {name = "game404", email = "studyoo@foxmail.com"},
]
dependencies = []
requires-python = ">=3.8"
license = {text = "MIT"}

[build-system]
requires = ["pdm-pep517>=1.0.0"]
build-backend = "pdm.pep517.api"
复制代码

```

当然在项目目录下，也会创建一个隐藏的 _.venv_ 目录，用来存放解释器，和其它工具一致。

安装包之前先使用 `pdm config pypi.url https://pypi.tuna.tsinghua.edu.cn/simple` 修改 pypi 的国内源，然后使用 `pdm add django`安装包。修改源后，安装包还是挺快的。查看项目的包，表格化展示:

```
pdm list
╭────────────────────┬─────────┬──────────╮
│ Package            │ Version │ Location │
├────────────────────┼─────────┼──────────┤
│ asgiref            │ 3.5.2   │          │
│ backports.zoneinfo │ 0.2.1   │          │
│ django             │ 4.1     │          │
│ sqlparse           │ 0.4.2   │          │
╰────────────────────┴─────────┴──────────╯
复制代码

```

也可以使用下面命令导出标准配置 _requirements.txt_ 文件和其它工具共享。

```
pdm export -o requirements.txt
复制代码

```

另外发现一个 pipx 的工具，也有点意思，以后有时间了体验一下。近期 go 和 rust 使用比较多，顺便也介绍一下这两个语言的开发环境。

go 开发环境
-------

### go 开发环境安装

go 在 mac 环境下的安装没什么好说的，可以直接使用官方提供的安装包安装，这里介绍 linux 下的 go 环境安装。

linux 下需要下载对应的版本，需要根据 cpu 架构选择，一般情况下选择 amd64 架构的，下载完成后解压到 _/usr/local_ 目录即可:

```
curl -LO https://go.dev/dl/go1.17.13.linux-amd64.tar.gz

tar -C /usr/local -xzf go1.19.linux-amd64.tar.gz
复制代码

```

然后修改一下环境变量，增加 go 到系统 path(修改后记得重新登录一下):

```
# /etc/profile

export PATH=$PATH:/usr/local/go/bin
复制代码

```

检测 go 的版本:

```
# go version
go version go1.17.13 linux/amd64
# whereis go
go: /usr/local/go /usr/local/go/bin/go
复制代码

```

go 的开发环境就设置完成了，非常简单, 已经不再需要按照一些古老的文档设置额外的环境变量。

### go 项目依赖

编写一个测试类 _main.go_ :

```
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
复制代码

```

运行一下:

```
$ go run main.go
Hello, World!
复制代码

```

go 使用 mod 命令管理模块，初始化项目 `mod init example/hello`， 其中 `example/hello` 是我们的包名:

```
$ go mod init example/hello
go: creating new go.mod: module example/hello
go: to add module requirements and sums:
	go mod tidy
复制代码

```

完成后会形成 _go.mod_ 文件:

```
module example/hello

go 1.17
复制代码

```

对于项目，可以直接这样运行:

```
$ go run .
Hello, World!
复制代码

```

依赖包的时候，可以直接在代码中添加依赖:

```
...
import "rsc.io/quote"

func main() {
    fmt.Println(quote.Go())
    ...
}
复制代码

```

然后使用`mod tidy` 自动下载和配置依赖项:

```
$ go mod tidy
go: finding module for package rsc.io/quote
go: downloading rsc.io/quote v1.5.2
go: found rsc.io/quote in rsc.io/quote v1.5.2
go: downloading rsc.io/sampler v1.3.0
go: downloading golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c
复制代码

```

再次运行项目，可以得到下面类似 python 之禅的输出:

```
Don't communicate by sharing memory, share memory by communicating.
复制代码

```

### go 版本升级

go 在 1.18 版本开始支持泛型，这是非常重要的一个特性，我们将刚安装好的 go 升级到 1.18 版本。

升级过程和安装类似，但是需要先清理掉旧的 go 版本:

```
$ rm -rf /usr/local/go
复制代码

```

直接修改 _main.go_ 添加一个泛型的实现:

```
package main

import "fmt"
import "rsc.io/quote"

func say[T string | int](a T) {
	fmt.Println(a)
}

func main() {
    fmt.Println(quote.Go())
	fmt.Println("Hello, World!")
	say("hello")
	say(2022)
}
复制代码

```

*   say 函数支持 string 和 int 两种类型的参数

再次运行项目:

```
$ go run .
Don't communicate by sharing memory, share memory by communicating.
Hello, World!
hello
2022
复制代码

```

记得将 _go.mod_ 中的 go 版本修改成 1.18。

rust 开发环境
---------

### rust 开发环境安装

rust 官方文档很详尽，可以直接按照官方文档执行。安装 Rust 的主要方式是通过 Rustup 这一工具，它既是一个 Rust 安装器又是一个版本管理工具，可以使用下面一条命令完成安装:

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
复制代码

```

完成后可以检测一下 rustup 的版本:

```
➜  ~ rustup -V
rustup 1.25.1 (2022-07-12)
The Rust toolchain installer
复制代码

```

还有 rustc 的版本:

```
➜  ~ rustc -V
rustc 1.62.1 (e092d0b6b 2022-07-16)
复制代码

```

编写下面的`hello.rs`程序:

```
➜  rust cat hello.rs
fn main(){
    println!("hello, rust");
}
复制代码

```

rust 是编译程序，所以我们需要先编译再运行:

```
# 编译
➜  rustc hello.rs
# 运行
➜  ./hello
hello, rust
复制代码

```

一般情况下我们不会直接使用 rustc，这样比较难以处理依赖，而是使用 `Cargo` Rust 官方的构建工具和包管理器。rustup 默认会按照 cargo:

```
➜  cargo -V
cargo 1.62.1 (a748cf5a3 2022-06-08)
复制代码

```

使用 `cargo new start_rust` 创建一个名为 start_rust 的项目，或者直接在当前目录使用 `cargo init`， 项目目录结构如下:

```
➜  start_rust git:(master) ✗ tree -L 2
.
├── Cargo.lock
├── Cargo.toml
├── src
│   └── main.rs
复制代码

```

可以看到和 go 的 mod 不一样，cargo 会创建 src 目录，源码都在这个目录下。

`Cargo.toml` 文件描述了项目的信息及依赖，大概如下:

```
[package]
name = "start_rust"
version = "0.1.0"
authors = ["game404 <studyoo@foxmail.com>"]
edition = "2018"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
...
复制代码

```

完成 main.rs 文件中的 main 函数，然后运行:

```
➜  start_rust git:(master) ✗ cargo run
   Compiling start_rust v0.1.0 (/Users/yoo/rust/start_rust)
    Finished dev [unoptimized + debuginfo] target(s) in 0.73s
     Running `target/debug/start_rust`
Hello, world!
复制代码

```

使用 cargo 非常方便，一个命令即完成编译和运行两个动作。

### rust 项目依赖

可以使用 `cargo add` 指令添加项目依赖:

```
# cargo add ferris-says
    Updating crates.io index
      Adding ferris-says v0.2.1 to dependencies.
             Features:
             - clippy
复制代码

```

添加完成后，我们可以在 _Cargo.toml_ 文件中看到下面的内容:

```
...
[dependencies]
ferris-says = "0.2.1"
...
复制代码

```

也可以直接修改这个 toml 文件，使用 `cargo build` 指令时候会自动安装。

修改 _main.rs_ 文件内容:

```
use ferris_says::say; // from the previous step
use std::io::{stdout, BufWriter};

fn main() {
    let stdout = stdout();
    let message = String::from("Hello fellow Rustaceans!");
    let width = message.chars().count();

    let mut writer = BufWriter::new(stdout.lock());
    say(message.as_bytes(), width, &mut writer).unwrap();
}
复制代码

```

运行项目:

```
# cargo run
    Updating crates.io index
  Downloaded smawk v0.3.1
  Downloaded smallvec v0.4.5
  Downloaded unicode-width v0.1.9
  Downloaded textwrap v0.13.4
  Downloaded ferris-says v0.2.1
  Downloaded 5 crates (97.7 KB) in 0.33s
   Compiling unicode-width v0.1.9
   Compiling smawk v0.3.1
   Compiling smallvec v0.4.5
   Compiling textwrap v0.13.4
   Compiling ferris-says v0.2.1
   Compiling rust v0.1.0 (/root/rust)
    Finished dev [unoptimized + debuginfo] target(s) in 2.34s
     Running `target/debug/rust`
 __________________________
< Hello fellow Rustaceans! >
 --------------------------
        \
         \
            _~^~^~_
        \) /  o o  \ (/
          '_   -   _'
          / '-----' \
复制代码

```

可以看到 rust 的吉祥物，一只叫做 ferris 的螃蟹。

### rust 版本升级

当前 rust 最新版本是`1.63.0`, 我们使用 rustup 将 rust 升级到最新版本。升级非常简单, 也只需要执行一条命令：

```
rustup update
复制代码

```

检测一下升级结果:

```
➜  rustup -V
rustup 1.25.1 (2022-07-12)
info: This is the version for the rustup toolchain manager, not the rustc compiler.
info: The currently active `rustc` version is `rustc 1.63.0 (4b91a6ea7 2022-08-08)`
➜  rustc -V
rustc 1.63.0 (4b91a6ea7 2022-08-08)
➜  cargo -V
cargo 1.63.0 (fd9c4297c 2022-07-01)
复制代码

```

> 今年各种事情导致断更了比较久。拖更的原因很多，复更的原因却只有一个，“坚持” 两字而已。也欢迎大家回来:)

参考链接
----

*   [www.rust-lang.org/zh-CN/learn…](https://link.juejin.cn?target=https%3A%2F%2Fwww.rust-lang.org%2Fzh-CN%2Flearn%2Fget-started "https://www.rust-lang.org/zh-CN/learn/get-started")
*   [github.com/pdm-project…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fpdm-project%2Fpdm "https://github.com/pdm-project/pdm")
*   [frostming.com/2020/02-28/…](https://link.juejin.cn?target=https%3A%2F%2Ffrostming.com%2F2020%2F02-28%2Fpdm-introduction%2F "https://frostming.com/2020/02-28/pdm-introduction/")
*   [go.dev/doc/install](https://link.juejin.cn?target=https%3A%2F%2Fgo.dev%2Fdoc%2Finstall "https://go.dev/doc/install")