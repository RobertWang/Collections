> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7125382825229942814) // mac 的话可以直接 brew install node

Go v1.17+  
// 可以参考 [Go 语言安装和配置](https://link.juejin.cn?target=https%3A%2F%2Fgo.dev%2Fdoc%2Finstall "https://go.dev/doc/install")

首先执行这个命令。会远程拉一个脚本并执行 Bud 的安装

```
curl -sf https://raw.githubusercontent.com/livebud/bud/main/install.sh | sh
复制代码

```

然后使用命令 `bud -h` 验证安装是否成功。

### 创建项目

用 `bud create` 来创建一个项目，参数是指定的当前路径下的项目目录。

```
# 在 blog/ 目录下创建一个新的bud项目
bud create blog
复制代码

```

会生成这样的目录和文件

```
$ ls
go.mod  node_modules/  package-lock.json  package.json
复制代码

```

### 目录结构

Bud 的标准目录结构如下所示：

```
$YOUR_APP
├─ bud
├─ controller
├─ internal
├─ public
└─ view
复制代码

```

`bud` 目录包含框架生成的代码。偶尔会需要从这个目录导入生成的包，但在大多数情况下可以忽略它的内容。 建议将此目录置于源代码控制之外。

`controller` 目录用于放置响应请求的代码。可用于为视图获取动态数据或提供 API。

`internal` 目录用于放置项目自身的代码，bud 框架永远不会用到这个目录。

`public` 目录用于放置静态资源，比如图片和视频。

`view` 目录用于存储 Svelte 文件以显示给您的访问者。视图通常与控制器配对。

运行`bud run`可以启动项目。

```
# 在默认的 3000 端口启动 bud server
bud run

# 在指定端口启动 bud server
bud run --port=8080

# 以非热加载模式启动 bud server
bud run --hot=false
复制代码

```

三、功能与使用
-------

###1. 网络路由

后端模块的请求处理逻辑，都是从 controller 开始。

当我们需要创建 controller 的时候，可以直接通过命令的方式创建。

```
# 创建一个叫作post的controller
bud new controller post

# 创建一个叫作post的controller，有index和show的function
bud new controller post index show

# 创建一个指定路由路径的controller
bud new controller post:/
复制代码

```

Bud 框架的 controller 目录结构示例如下所示：

```
$YOUR_APP
└─ controller
    ├─ controller.go        -> root controller
    ├─ posts
    │  ├─ posts.go          -> posts controller
    │  └─ comments
    │     └─ comments.go    -> comments controller
    └─ users
       └─ users.go          -> users controller
复制代码

```

users controller 的示例如下，Bud 框架对于 controller 的方法，都有对应的默认 url 路径。

```
package users

// Controller for /users
type Controller struct {}

// User type
type User struct {
   ID int
   Name string
   Age int
}

// Index shows a list of users
// GET /users
func (c *Controller) Index() ([]*User, error) {}

// New user page
// GET /users/new
func (c *Controller) New() {}

// Create a new user
// POST /users
func (c *Controller) Create(name string, age int) (*User, error) {}

// Show a user
// GET /users/:id
func (c *Controller) Show(id int) (*User, error) {}

// Update a user
// PATCH /users/:id
func (c *Controller) Update(id int, name string, age int) error {}

// Delete a user
// DELETE /users/:id
func (c *Controller) Delete(id int) error {}

// Edit user page
// GET /users/:id/edit
func (c *Controller) Edit(id int) (*User, error) {}
复制代码

```

例如一个这样的请求

对于路径 /posts/:id/comments?order=xx&author=xx

```
GET /posts/10/comments?order=asc&author=Alice
复制代码

```

那么实际的 GET 请求参数是这样的：

```
{
  "id": 10,
  "order": "asc",
  "author": "Alice"
}
复制代码

```

```
POST /posts/10/comments?author=Alice
{
  "email": "alice@livebud.com"
}
复制代码

```

那么实际的 POST 请求参数是这样的：

```
{
  "id": 10,
  "author": "Alice",
  "email": "alice@livebud.com"
}
复制代码

```

对于 path 参数和 query 参数有冲突的情况，例如：

```
POST /posts/10/comments?id=20&author=Alice
{
   "id": 30,
   "author": "Bob"
}
复制代码

```

那这时候，参数生效的优先级是这样的

1. 路径值（最高优先级）  
2.query 参数  
3. 请求正文（最低优先级）

那么，最后生效的参数是这样的：

```
{
   "id": 10,
   "author": "Bob"
}
复制代码

```

bud 框架对这一类的功能，已经有处理的逻辑了。并且对于错误参数的处理也已经有一套逻辑，不需要用户再去写这类逻辑。

### 2. 前端框架

bud 使用的前端框架是 Svelte.JS，当如 1 中所示创建 controller 和 action 之后，可以生成对应的 svelte 文件

```
$YOUR_APP
└─ view
   ├─ index.svelte     -> Root index page
   ├─ posts
   │  ├─ edit.svelte   -> Edit post page
   │  ├─ index.svelte  -> Post index page
   │  ├─ new.svelte    -> New post page
   │  ├─ show.svelte   -> Show post page
   │  └─ Post.svelte   -> Post component (not rendered)
   └─ users
      ├─ edit.svelte   -> Edit user page
      ├─ index.svelte  -> User index page
      └─ show.svelte   -> Show user page
复制代码

```

例如可以这样让 controller 和 view 交互：

controller/users/users.go

```
package users

type Controller struct {}

type User struct {
  Name string
  Age int
}

func (c *Controller) Index() []*User {
  return []*User{
    {"Matt", 32},
    {"Mia", 31},
    {"Mike", 26},
  }
}
复制代码

```

view/users/index.svelte

```
<script>
  export let users = []
</script>

<h1>Users</h1>

<ul>
   {#each users as user}
   <li>{user.name} is {user.age} years old</li>
   {/each}
</ul>
复制代码

```

### 3. 扩展插件

插件以 bud- 前缀开头，看起来像任何其他 Bud 应用程序。例如，让我们创建一个插件来服务 Tailwind 的 preflight.css 文件

```
bud-tailwind/
├── public
│   └── preflight.css
├── go.mod
└── go.sum
复制代码

```

如果 Bud 项目的目录如下所示

```
$YOUR_APP
├── controller
│   └── post.go
├── public
│   └── favicon.ico
├── view
│   ├── index.svelte
│   └── show.svelte
├── go.mod
└── go.sum
复制代码

```

那通过执行命令 `go get github.com/livebud/bud-tailwind`之后，preflight.css 就会安装到该 bud 项目中。

```
  $YOUR_APP
  ├── controller
  │   └── post.go
  ├── public
  │   ├── favicon.ico
+ │   └── preflight.css
  ├── view
  │   ├── index.svelte
  │   └── show.svelte
  ├── go.mod
  └── go.sum
复制代码

```

四、总结
----

### Bud 的特点：

*   Bud 使用静态分析生成代码。这允许 Bud 提供更高级别的 API，这些 API 可以被编译成低级的高性能 Go 代码。
*   越来越多的捆绑代码生成器提供了特定于框架的功能。这些代码生成器与您可以自己编写的代码生成器没有什么不同。通过添加、自定义甚至丢弃这些代码生成器，让 Bud 成为您自己的。
*   Bud 带来了它自己的依赖注入框架，它减少了你需要做的手动连接，并标准化了你开发模块化、可测试和可交换依赖项的方式。

但是目前 Bud 框架并没有自己的 ORM，用户需要自行选择 orm 框架，通过依赖注入在控制器中使用。

另外，还有全栈框架包含的特性，如错误处理、依赖注入、数据验证及安全、文件与存储等都可以在官方文档上找到。

本文正在参加[技术专题 18 期 - 聊聊 Go 语言框架](https://juejin.cn/post/7117898969866305566 "https://juejin.cn/post/7117898969866305566")