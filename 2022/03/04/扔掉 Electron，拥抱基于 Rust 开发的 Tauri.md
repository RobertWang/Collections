> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7067342513920540686) ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/649d31573bcb443db5b33d19769996a6~tplv-k3u1fbpfcp-watermark.awebp)

Tauri 是什么
---------

Tauri 是一个跨平台 `GUI` 框架，与 `Electron` 的思想基本类似。Tauri 的前端实现也是基于 Web 系列语言，Tauri 的后端使用 `Rust`。Tauri 可以创建体积更小、运行更快、更加安全的跨平台桌面应用。

为什么选择 Rust？

`Rust` 是一门赋予每个人构建可靠且高效软件能力的语言。它在高性能、可靠性、生产力方面表现尤为出色。Rust 速度惊人且内存利用率极高，由于没有运行时和垃圾回收，它能够胜任对性能要求特别高的服务，可以在嵌入式设备上运行，还能轻松和其他语言集成。Rust 丰富的类型系统和所有权模型保证了内存安全和线程安全，让您在编译期就能够消除各种各样的错误。Rust 也拥有出色的文档、友好的编译器和清晰的错误提示信息，还集成了一流的工具——包管理器和构建工具……

基于此，让 Rust 成为不二之选，开发人员可以很容易的使用 Rust 扩展 Tauri 默认的 `Api` 以实现定制化功能。

Tauri VS Electron
-----------------

<table><thead><tr><th>Detail</th><th>Tauri</th><th>Electron</th></tr></thead><tbody><tr><td>Installer Size Linux</td><td>3.1 MB</td><td>52.1 MB</td></tr><tr><td>Memory Consumption Linux</td><td>180 MB</td><td>462 MB</td></tr><tr><td>Launch Time Linux</td><td>0.39s</td><td>0.80s</td></tr><tr><td>Interface Service Provider</td><td>WRY</td><td>Chromium</td></tr><tr><td>Backend Binding</td><td>Rust</td><td>Node.js (ECMAScript)</td></tr><tr><td>Underlying Engine</td><td>Rust</td><td>V8 (C/C++)</td></tr><tr><td>FLOSS</td><td>Yes</td><td>No</td></tr><tr><td>Multithreading</td><td>Yes</td><td>Yes</td></tr><tr><td>Bytecode Delivery</td><td>Yes</td><td>No</td></tr><tr><td>Multiple Windows</td><td>Yes</td><td>Yes</td></tr><tr><td>Auto Updater</td><td>Yes</td><td>Yes</td></tr><tr><td>Custom App Icon</td><td>Yes</td><td>Yes</td></tr><tr><td>Windows Binary</td><td>Yes</td><td>Yes</td></tr><tr><td>MacOS Binary</td><td>Yes</td><td>Yes</td></tr><tr><td>Linux Binary</td><td>Yes</td><td>Yes</td></tr><tr><td>iOS Binary</td><td>Soon</td><td>No</td></tr><tr><td>Android Binary</td><td>Soon</td><td>No</td></tr><tr><td>Desktop Tray</td><td>Yes</td><td>Yes</td></tr><tr><td>Sidecar Binaries</td><td>Yes</td><td>No</td></tr></tbody></table>

环境安装
----

### macOS

由于安装过程比较简单，作者使用的是 macOS，本文只介绍 macOS 安装步骤， Windows 安装步骤可自行查看官网。

**1. 确保 Xcode 已经安装**

```
$ xcode-select --install
复制代码
```

**2. Node.js**

建议使用 `nvm` 进行 node 版本管理：

```
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.2/install.sh | bash
复制代码
```

```
$ nvm install node --latest-npm
$ nvm use node
复制代码
```

强烈推荐安装 `Yarn`，用来替代 npm。

**3.Rust 环境**

安装 `rustup`：

```
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
复制代码
```

验证 `Rust` 是否安装成功：

```
$ rustc --version

rustc 1.58.1 (db9d1b20b 2022-01-20)
复制代码
```

_tips：如果 `rustc` 命令执行失败，可以重启一下终端。_

至此，Tauri 开发环境已安装完毕。

项目搭建
----

**1. 创建一个 Tauri 项目**

```
$ yarn create tauri-app
复制代码
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3f121f2b8e544d9d8b516a644dc6e9a4~tplv-k3u1fbpfcp-watermark.awebp)

按一下回车键，继续……

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/945c6d7a0e12423ba0c44ca25f6f0176~tplv-k3u1fbpfcp-watermark.awebp)

可以看出，目前主流的 Web 框架 Tauri 都支持， 我们选择 `create-vite`……

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e93ec96659364860977fd7fb0d049d89~tplv-k3u1fbpfcp-watermark.awebp)

此处选择 `Y`，将 `@tauri-apps/api` 安装进来， 然后选择 `vue-ts`……

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/537e1b1d06b5452993b0469c4d1e6311~tplv-k3u1fbpfcp-watermark.awebp)

检查 Tauri 相关的设置，确保一切就绪……

```
$ yarn tauri info
复制代码
```

```
yarn run v1.22.17
$ tauri info

Operating System - Mac OS, version 12.2.0 X64

Node.js environment
  Node.js - 14.17.0
  @tauri-apps/cli - 1.0.0-rc.2
  @tauri-apps/api - 1.0.0-rc.0

Global packages
  npm - 6.14.13
  pnpm - Not installed
  yarn - 1.22.17

Rust environment
  rustc - 1.58.1
  cargo - 1.58.0

Rust environment
  rustup - 1.24.3
  rustc - 1.58.1
  cargo - 1.58.0
  toolchain - stable-x86_64-apple-darwin

App directory structure
/dist
/node_modules
/public
/src-tauri
/.vscode
/src

App
  tauri.rs - 1.0.0-rc.1
  build-type - bundle
  CSP - default-src 'self'
  distDir - ../dist
  devPath - http://localhost:3000/
  framework - Vue.js
✨  Done in 20.72s.
复制代码
```

至此，一个新的 Tauri 项目已创建完成。

_tips：Tauri 也支持基于已存在的前端项目进行集成，具体流程可查看官网，本文不做介绍。_

项目目录介绍
------

```
├── README.md
├── dist                 - web 项目打包编译目录
│   ├── assets
│   ├── favicon.ico
│   └── index.html
├── index.html         
├── node_modules
├── package.json
├── public
│   └── favicon.ico
├── src                  - vue 项目目录（页面开发）
│   ├── App.vue
│   ├── assets
│   ├── components
│   ├── env.d.ts
│   └── main.ts
├── src-tauri            - rust 相关目录（tauri-api 相关配置）
│   ├── Cargo.lock
│   ├── Cargo.toml       - rust 配置文件
│   ├── build.rs
│   ├── icons            - 应用相关的 icons
│   ├── src              - rust 入口
│   ├── target           - rust 编译目录
│   └── tauri.conf.json  - tauri 相关配置文件
├── tsconfig.json
├── tsconfig.node.json
├── vite.config.ts
└── yarn.lock
复制代码
```

运行
--

运行项目：

```
$ cd tauri-demo1
$ yarn tauri dev
复制代码
```

等待项目 run 起来……

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bdd3fa8c131445b798ab0f09967920f2~tplv-k3u1fbpfcp-watermark.awebp)

可以看到，一个基于 `Vue 3 + TypeScript + Vite` 的桌面端应用已经运行起来了。

API 调用及功能配置
-----------

Tauri 的 Api 有 `JavaScript Api` 和 `Rust Api` 两种 ，本文主要选择一些 `Rust Api` 来进行讲解（Rust 相关知识可自行学习），JavaScript 相关的 Api 相对简单一些，可按照官方文档进行学习。

### 1.Splashscreen（启动画面）

添加启动画面对于初始化耗时的应用来说是非常有必要的，可以提升用户体验。

大致原理是在应用初始化阶段先隐藏主应用视图，展示启动画面视图，等待初始化完成以后动态关闭启动画面视图，展示主视图。

首先在项目根目录创建一个 `splashscreen.html` 文件作为启动画面视图，具体展示内容可自行配置，代码如下：

```
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta >
  <title>Loading</title>
</head>

<body style="background-color: aquamarine;">
  <h1>Loading...</h1>
</body>

</html>
复制代码
```

其次更改 `tauri.conf.json` 配置项：

```
"windows": [
  {
    "title": "Tauri App",
    "width": 800,
    "height": 600,
    "resizable": true,
    "fullscreen": false,
+   "visible": false // 默认隐藏主视图
  },
  // 添加启动视图
+ {
+   "width": 400,
+   "height": 200,
+   "decorations": false,
+   "url": "splashscreen.html",
+   "label": "splashscreen"
+ }
]
复制代码
```

将 `windows` 配置项下的主视图 `visible` 属性设置为 `false`，这样初始化阶段，主视图就会隐藏；

在 `windows` 配置项下新建一个启动视图，视图大小可以自定义配置。

接下来就是动态控制两个视图的显示和隐藏了。

打开 `src-tauri/main.rs` 文件，添加以下 Rust 代码：

```
use tauri::Manager;

// 创建一个 Rust 命令
#[tauri::command]
fn close_splashscreen(window: tauri::Window) {
  // 关闭启动视图
  if let Some(splashscreen) = window.get_window("splashscreen") {
    splashscreen.close().unwrap();
  }
  // 展示主视图
  window.get_window("main").unwrap().show().unwrap();
}

fn main() {
  tauri::Builder::default()
    // 注册命令
    .invoke_handler(tauri::generate_handler![close_splashscreen])
    .run(tauri::generate_context!())
    .expect("error while running tauri application");
}
复制代码
```

以上 Rust 代码的执行逻辑是创建一个 `close_splashscreen` 函数用来关闭启动视图并展示主视图，并将这个函数注册为一个 Rust 命令，在应用初始化时进行注册，以便在 JavaScript 中可以动态调用该命令。

接下来，在 `src/App.vue` 中添加以下代码：

```
// 导入 invoke 方法
import { invoke } from '@tauri-apps/api/tauri'

// 添加监听函数，监听 DOM 内容加载完成事件
document.addEventListener('DOMContentLoaded', () => {
  // DOM 内容加载完成之后，通过 invoke 调用 在 Rust 中已经注册的命令
  invoke('close_splashscreen')
})
复制代码
```

我们可以看一下 `invoke` 方法的源码：

```
/**
 * Sends a message to the backend.
 *
 * @param cmd The command name.
 * @param args The optional arguments to pass to the command.
 * @return A promise resolving or rejecting to the backend response.
 */
async function invoke<T>(cmd: string, args: InvokeArgs = {}): Promise<T> {
  return new Promise((resolve, reject) => {
    const callback = transformCallback((e) => {
      resolve(e)
      Reflect.deleteProperty(window, error)
    }, true)
    const error = transformCallback((e) => {
      reject(e)
      Reflect.deleteProperty(window, callback)
    }, true)

    window.rpc.notify(cmd, {
      __invokeKey: __TAURI_INVOKE_KEY__,
      callback,
      error,
      ...args
    })
  })
}
复制代码
```

`invoke` 方法是用来和后端（Rust）进行通信，第一个参数 `cmd` 就是在 Rust 中定义的命令，第二个参数 `args` 是可选的配合第一个参数的额外信息。方法内部通过 `window.rpc.notify` 来进行通信，返回值是一个 Promise。

至此，添加启动视图的相关逻辑已全部完成，我们可以运行查看一下效果。

由于我们的 demo 项目初始化很快，不容易看到启动视图，因此可通过 `setTimeout` 延迟 `invoke('close_splashscreen')` 的执行，方便调试查看：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4be7438f0cf431ab676d2d58f4a6a6f~tplv-k3u1fbpfcp-watermark.awebp)

可以看到，在项目运行起来之后，首先展示的是启动视图，其次启动视图消失，主视图展示出来。

### 2.Window Menu（应用菜单）

为应用添加菜单是很基础的功能，同时也很重要。

打开 `src-tauri/main.rs` 文件，添加以下 Rust 代码：

```
use tauri::{ Menu, Submenu, MenuItem, CustomMenuItem };

fn main() {
  let submenu_gear = Submenu::new(
    "Gear",
    Menu::new()
      .add_native_item(MenuItem::Copy)
      .add_native_item(MenuItem::Paste)
      .add_native_item(MenuItem::Separator)
      .add_native_item(MenuItem::Zoom)
      .add_native_item(MenuItem::Separator)
      .add_native_item(MenuItem::Hide)
      .add_native_item(MenuItem::CloseWindow)
      .add_native_item(MenuItem::Quit),
  );
  let close = CustomMenuItem::new("close".to_string(), "Close");
  let quit = CustomMenuItem::new("quit".to_string(), "Quit");
  let submenu_customer = Submenu::new(
    "Customer", 
    Menu::new()
      .add_item(close)
      .add_item(quit)
    );
  let menus = Menu::new()
    .add_submenu(submenu_gear)
    .add_submenu(submenu_customer);

  tauri::Builder::default()
    // 添加菜单
    .menu(menus)
    // 监听自定义菜单事件
    .on_menu_event(|event| match event.menu_item_id() {
      "quit" => {
        std::process::exit(0);
      }
      "close" => {
        event.window().close().unwrap();
      }
      _ => {}
    })
    // 注册命令
    .invoke_handler(tauri::generate_handler![close_splashscreen])
    .run(tauri::generate_context!())
    .expect("error while running tauri application");
}
复制代码
```

首先我们引入 `Menu`、`Submenu`、`MenuItem`、`CustomMenuItem`。

查看 `Menu` 以及 `Submenu` 源码：

```
/// A window menu.
#[derive(Debug, Clone)]
#[non_exhaustive]
pub struct Menu {
  pub items: Vec<MenuEntry>,
}

impl Default for Menu {
  fn default() -> Self {
    Self { items: Vec::new() }
  }
}

#[derive(Debug, Clone)]
#[non_exhaustive]
pub struct Submenu {
  pub title: String,
  pub enabled: bool,
  pub inner: Menu,
}

impl Submenu {
  /// Creates a new submenu with the given title and menu items.
  pub fn new<S: Into<String>>(title: S, menu: Menu) -> Self {
    Self {
      title: title.into(),
      enabled: true,
      inner: menu,
    }
  }
}

impl Menu {
  /// Creates a new window menu.
  pub fn new() -> Self {
    Default::default()
  }

  /// Adds the custom menu item to the menu.
  pub fn add_item(mut self, item: CustomMenuItem) -> Self {
    self.items.push(MenuEntry::CustomItem(item));
    self
  }

  /// Adds a native item to the menu.
  pub fn add_native_item(mut self, item: MenuItem) -> Self {
    self.items.push(MenuEntry::NativeItem(item));
    self
  }

  /// Adds an entry with submenu.
  pub fn add_submenu(mut self, submenu: Submenu) -> Self {
    self.items.push(MenuEntry::Submenu(submenu));
    self
  }
}
复制代码
```

`Menu` 这个结构体就是用来实现应用菜单的，它内置的 `new` 关联函数用来创建 `menu`，`add_item` 方法用来添加自定义菜单项，`add_native_item` 方法用来添加 Tauri 原生实现的菜单项，`add_submenu` 用来添加菜单入口。

`Submenu` 这个结构体用来创建菜单项的入口。

如图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/72a72f99019e4674856a022797b2cbe2~tplv-k3u1fbpfcp-watermark.awebp)

箭头所指的 `Gear` 和 `Customer` 就是 `Submenu`，红框里是 `Submenu` 下所包含的 `MenuItem` 项。

我们创建了一个命名为 `Gear` 的 `Submenu`，并添加了一些 Tauri 原生支持的 `MenuItem` 项进去。

我们也创建了一个命名为 `Customer` 的 `Submenu`，并添加了两个自定义的 `CustomMenuItem` 项，`CustomMenuItem` 的事件需要开发者自己定义：

```
// 监听自定义菜单事件
on_menu_event(|event| match event.menu_item_id() {
  "quit" => {
    // 逻辑自定义
    std::process::exit(0);
  }
  "close" => {
    // 逻辑自定义
    event.window().close().unwrap();
  }
  _ => {}
})
复制代码
```

通过 `on_menu_event` 方法监听自定义菜单项的触发事件，它接收的参数是一个 `闭包`，用 `match` 对菜单项的 `事件 id` 进行匹配，并添加自定义逻辑。

**注意事项**

Tauri 原生支持的 MenuItem 菜单项存在兼容性问题，可以看源码：

```
/// A menu item, bound to a pre-defined action or `Custom` emit an event. Note that status bar only
/// supports `Custom` menu item variants. And on the menu bar, some platforms might not support some
/// of the variants. Unsupported variant will be no-op on such platform.
#[derive(Debug, Clone)]
#[non_exhaustive]
pub enum MenuItem {

  /// A menu item for enabling cutting (often text) from responders.
  ///
  /// ## Platform-specific
  ///
  /// - **Windows / Android / iOS:** Unsupported
  ///
  Cut,

  /// A menu item for pasting (often text) into responders.
  ///
  /// ## Platform-specific
  ///
  /// - **Windows / Android / iOS:** Unsupported
  ///
  Paste,

  /// Represents a Separator
  ///
  /// ## Platform-specific
  ///
  /// - **Windows / Android / iOS:** Unsupported
  ///
  Separator,
  ...
}
复制代码
```

可以看出内置的这些菜单项在 `Windows`、`Android`、`iOS` 平台都还不支持，但是随着稳定版的发布，相信这些兼容性问题应该能得到很好的解决。

调试
--

在开发模式下，调试相对容易。以下来看在开发模式下如何分别调试 `Rust` 和 `JavaScript` 代码。

### Rust Console

调试 Rust 代码，我们可以使用 `println!` 宏，来进行调试信息打印：

```
let msg = String::from("Debug Infos.")
println!("Hello Tauri! {}", msg);
复制代码
```

调试信息会在终端打印出来：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d6d4fc8e9e634afdae46c812e0a75a67~tplv-k3u1fbpfcp-watermark.awebp)

### WebView JS Console

JavaScript 代码的调试，我们可使用 `console` 相关的函数来进行。在应用窗口右键单击，选择 `Inspect Element` 即 审查元素，就可以打开 WebView 控制台。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/854d04416610419094d3dd455064a066~tplv-k3u1fbpfcp-watermark.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6424725e7002440cab04ed108dc5552c~tplv-k3u1fbpfcp-watermark.awebp)

控制台相关的操作就不再赘述了。

_tips：在一些情况下，我们可能也需要在最终包查看 WebView 控制台，因此 Tauri 提供了一个简单的命令用来创建 `调试包`：_

```
yarn tauri build --debug
复制代码
```

通过该命令打包的应用程序将放置在 `src-tauri/target/debug/bundle` 目录下。

应用打包
----

```
yarn tauri build
复制代码
```

该命令会将 Web 资源 与 Rust 代码一起嵌入到单个二进制文件中。二进制文件本身将位于 `src-tauri/target/release/[app name]`，安装程序将位于 `src-tauri/target/release/bundle/`。

Roadmap
-------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cba3dbea18cd437aa935bcde595f5acf~tplv-k3u1fbpfcp-watermark.awebp)

从 Tauri 的 `Roadmap` 可以看出，稳定版会在 `2022 Q1` 发布，包括后续对 `Deno` 的支持，以及打包到移动设备的支持。因此 Tauri 的发展还是很值得期待的。

总结
--

Tauri 主打的 更小、更快、更安全，相较于 `Electron` 让人诟病的包太大、内存消耗过大等问题来看，的确是一个很有潜力的桌面端应用开发框架，同时在 `Rust` 的加持下如有神助，让这款桌面端应用开发框架极具魅力。不过由于 Tauri 到目前为止还没发布稳定版，以及一些功能还存在多平台兼容性等问题，致使目前还不能在生产环境进行大面积应用。相信随着 Tauri 的发展，这些问题都会得到解决，以后的桌面端应用开发市场中也会有很大一部分份额会被 Tauri 所占有。作为开发者的我们，此刻正是学习 `Tauri` 以及 `Rust` 的最佳时机，行动起来吧~

#### **更多精彩请关注我们的公众号「百瓶技术」，有不定期福利呦！**