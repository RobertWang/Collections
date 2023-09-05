> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/HBB4s85cfi8jAXlOkxl3WA)

`Web`是 Go 语言的一个主要应用方向，虽然 Go 语言的标准库提供了诸如`net/http`，`database/sql`等多个非常好用的库，以供我们开发`Web`应用，但是对于大型的`Web`应用来说，标准库还是显得有些力不从心。

Go 并不像其他编程语言一样推崇大型的应用框架 (比如 Java 的`Spring`)，Go 语言的编程哲学是一个库只做一件事，因此我们可以挑选 Go 社区中优秀的第三方库，像搭积木一样构建我们的`Web`程序。

在`Go`社区中，有许多非常优秀的第三方库，在这篇文章中，我们来学其中五个在`Web`开发中常用的库，分别是：`Cobra`，`Gin`，`Gorm`，`Viper`以及`Logrus`。

开始
--

我们通过一个完整的示例程序来演示上述几个 Go 第三方库的使用。

这个示例程序最终会被编译为一个命令行可执行命令`ms`，该程序支持两个子命令：

```
$ ms migrate //在数据库生成创建数据表
$ ms server // 启动Web服务器


```

让我们先从创建一个示例程序目录开始吧：

```
$ mkdir ms
$ cd ms
$ go mod init ms


```

接着，创建程序的入口`main.go`文件：

```
package main 

func main(){
 //do something
}


```

此时项目的目录结构为：

```
.
├── go.mod
└── main.go


```

Cobra
-----

`Cobra`是一个用于创建现代命令行程序的库，被许多非常出名的开源项目所使用，比如`Kubernetes`，`Docker`等。

安装`cobra`库，在项目根目录下执行`go get`命令：

```
$ go get -u github.com/spf13/cobra@latest


```

在`ms`目录创建`cmd`子目录，这个目录我们用来放置子命令代码：

```
$ mkdir cmd


```

由于`Cobra`的子命令有统一的代码格式，因此`Cobra`提供了`cobra-cli`工具来生成代码，以节省开发时间。

执行下面的`go install`命令将`cobra-cli`命令安装到`GOPATH/bin`目录下：

> `GOPATH/bin`要配置到环境变量`PATH`中，这样才可以在命令行下执行`cobra-cli`命令。

```
$ go install github.com/spf13/cobra-cli@latest


```

创建`cmd/root.go`文件，这是主命令的入口，其代码如下：

```
package cmd

import (
 "fmt"
 "os"

 "github.com/spf13/cobra"
)

var rootCmd = &cobra.Command{
 Use:   "ms",
 Short: "This is my example project",
 Long:  "This is my example project for praticing my golang programs skill",
 Run: nil,
}

func Execute() {
 if err := rootCmd.Execute(); err != nil {
  fmt.Fprintln(os.Stderr, err)
  os.Exit(1)
 }
}


```

在上面的代码中，我们调用`cobra.Command`创建一个主命令，其中`Run`字段表示命令的执行逻辑，由于我们把功能放在子命令中，因此这里`Run`设置为`nil`。

接着，在项目根目录下执行`cobra-cli`命令：

```
$ cobra-cli add server
$ cobra-cli add migrate


```

上面命令执行成功后会`cmd`目录生面子命令文件`server.go`和`migrate.go`，其代码如下：

`cmd/server.go`

```
package cmd

import (
 "fmt"

 "github.com/spf13/cobra"
)

func init() {
 rootCmd.AddCommand(serverCmd)
}

var serverCmd = &cobra.Command{
 Use:   "server",
 Short: "启动一个Web服务器",
 Long:  `用于启动一个接收HTTP请求的Web服务器`,
 Run: func(cmd *cobra.Command, args []string) {
        //do something
 },
}


```

`cmd/migrate.go`

```
package cmd

import (
 "fmt"

 "github.com/spf13/cobra"
)

func init() {
 rootCmd.AddCommand(migrateCmd)
}

var migrateCmd = &cobra.Command{
 Use:   "migrate",
 Short: "数据表迁移工具",
 Long:  `主要用于将在项目创建时初始化数据表`,
 Run: func(cmd *cobra.Command, args []string) {
  //do something
 },
}



```

上面两个子命令的`Run`就是后面要开发的主逻辑，这里暂时为空，不过此时我们的项目结构已经逐渐完善起来了：

```
.
├── cmd
│   ├── migrate.go
│   ├── root.go
│   └── server.go
├── go.mod
├── go.sum
└── main.go


```

如果此时我们编译后运行`ms --help`，会输出命令行帮助信息，这就是`Cobra`帮我们做到的：

```
$ ./ms --help
This is my example project for praticing my golang programs skill

Usage:
  ms [command]

Available Commands:
  completion  Generate the autocompletion script for the specified shell
  help        Help about any command
  migrate     数据表迁移工具
  server      启动一个Web服务器


```

Viper
-----

我们会将很多的参数放在配置文件中，比如`Web`服务器运行的端口号，访问数据库的主机、用户名等信息，在我们示例项目中，将配置保存在`yaml`格式文件中，而读取这些配置则`viper`库。

`viper`是 Go 语言一个非常完整的应用配置解决方案，支持多种不同数据格式配置文件的读取与写入。

首先还是用`go get`命令下载安装`viper`：

```
$ go get github.com/spf13/viper


```

在项目根目录下创建`config.yaml`文件：

```
Server: 
  port: 8080
Log:
  path: ./logs/app.log
  level: 6 //对应logrus的日志等级
database:
  host: localhost
  port: 3306
  user: root
  password: root
  dbName: test


```

在`ms`目录下创建`internal`目录：

```
$ mkdir internal


```

在`internal`目录下创建配置项对应的结构体：

`internal/config.go`

```
package internal

//总配置
type Config struct {
 Database *Database
 Server   *Server
 Log      *Log
}

//数据库配置
type Database struct {
 Host     string
 Port     int
 User     string
 Password string
 DbName   string
}

//服务器配置
type Server struct {
 Port string
}

//日志
type Log struct {
 Path  string
 Level uint8
}


```

在`internal`目录下创建`app.go`存放全局变量：

```
package internal

var AppConfig Config


```

我们将读取配置的代码单独放在项目`internal`包的`util.go`文件中：

```
package internal

import (
 "ms/app"
 "fmt"

 "github.com/spf13/viper"
)

func InitConfig() {
 //配置文件名称
 viper.SetConfigName("config")
 //配置文件类型
 viper.SetConfigType("yaml")
 //读取配置文件的范围
 viper.AddConfigPath("/etc/appname/")
 viper.AddConfigPath("$HOME/.appname")
 viper.AddConfigPath(".")
 //开始读取
 err := viper.ReadInConfig()
 if err != nil {
  panic(fmt.Errorf("fatal error config file: %w", err))
 }
 //将配置文件设置到结构体中
 viper.Unmarshal(&app.AppConfig)
}


```

Gin
---

获得配置参数之后，我们可以通过参数来初始我们的`Web`服务器了，`Web`框架选择`Gin`，因为`Gin`是一个极其优秀的`Web`框架，简单易用，支持中间件，路由分组，`JSON`处理等功能。

同样用`go get`命令安装`Gin`库：

```
$ go get -u github.com/gin-gonic/gin


```

接下来我们修改`cmd/server.go`文件中的源码：

```
package cmd

import (
 "ms/app"
 "ms/internal"

 "github.com/gin-gonic/gin"
 "github.com/spf13/cobra"
)

func init() {
 rootCmd.AddCommand(serverCmd)
}

var serverCmd = &cobra.Command{
 Use:   "server",
 Short: "启动一个Web服务器",
 Long:  `用于启动一个接收HTTP请求的Web服务器`,
 Run: func(cmd *cobra.Command, args []string) {
  internal.InitConfig()
  internal.InitLogger()
  internal.InitDb()
  server := gin.Default()
  app.InitRouter(server)
  err := server.Run(":" + internal.AppConfig.Server.Port)
  if err != nil {
   panic(fmt.Errorf("fatal to start a web server: %w", err))
  }
 },
}


```

创建`app/controllers`目录，用于存储控制器：

```
$ mkdir app
$ cd app
$ mkdir controllers
$ cd controllers


```

在`controllers`目录下创建`user`目录，在这个目录下有三个文件存放控制器代码：

`ms/app/controllers/user/add.go`

```
package user

import "github.com/gin-gonic/gin"

func Add(c *gin.Context) {

}


```

`ms/app/controllers/user/detail.go`

```
package user

import "github.com/gin-gonic/gin"

func Detail(c *gin.Context) {

}



```

`ms/app/controllers/user/list.go`

```
package user

import "github.com/gin-gonic/gin"

func List(c *gin.Context) {

}


```

在`app`目录创建`router.go`文件，在该文件添加一个初始化路由的函数：

```
package app

import (
 "ms/app/controllers/user"

 "github.com/gin-gonic/gin"
)

//注册路由
func InitRouter(server *gin.Engine) {
 server.GET("/user/list", user.List)
 server.POST("/user/add", user.Add)
 server.GET("/user/:id", user.Detail)
    //...
}


```

在`internal`包增加一个`response.go`文件存放格式响应的代码：

```
package internal

import (
 "net/http"

 "github.com/gin-gonic/gin"
)

type response struct {
 Code    int
 Message string
 Data    interface{}
}

func Success(c *gin.Context, message string, data interface{}) {
 c.JSON(http.StatusOK, response{Code: http.StatusOK, Message: message, Data: data})
}

func ReturnData(c *gin.Context, data interface{}) {
 c.JSON(http.StatusOK, response{Code: http.StatusOK, Message: "成功", Data: data})
}

func SimpleSuccess(c *gin.Context) {
 c.JSON(http.StatusOK, response{Code: http.StatusOK, Message: "执行成功", Data: nil})
}

func InternalError(c *gin.Context) {
 c.JSON(http.StatusInternalServerError, response{Code: http.StatusInternalServerError, Message: "服务内部错误"})
}

func BadRequest(c *gin.Context) {
 c.JSON(http.StatusBadRequest, response{Code: http.StatusBadRequest, Message: "请求数据错误"})
}


```

Logrus
------

日志也是一个应用程序不可或缺的一部分，尤其是调试`bug`或者定位程序运行的问题的时候。

Go 应用程序的日志库推荐使用`logrus`，`logrus`是一个非常优秀的日志库，可以完美兼容 Go 标准库`log`，支持多种日志格式输出以及自定义字段。

同样用`go get`命令进行安装：

```
$ github.com/sirupsen/logrus


```

接着，在`internal/util.go`文件添加以下函数用于初始化日志：

```
func InitLogger() {
 internal.Logger = logrus.New()
 file, err := os.Create(app.AppConfig.Log.Path)
 if err != nil {
  panic("could not open log file in path " + internal.AppConfig.Log.Path)
 }
 internal.Logger.SetOutput(file)
 internal.Logger.SetLevel(logrus.Level(internal.AppConfig.Log.Level))
}


```

在`InitLogger`函数中，我们创建一个`Logrus.Logger`对象，并将日志输出改为配置中的文件路径，再者就是设置了日志的级别，这样在程序的其他地方就可以调用以下的代码进行各个级别的日志输出了：

```
internal.Logger.Info("logging message")
internal.Logger.Eroor("logging message")
internal.Logger.Trace("logging message")
internal.Logger.Fatal("logging message")
internal.Logger.Debug("logging message")
internal.Logger.Warn("logging message")
internal.Logger.Panic("logging message")


```

Gorm
----

`Gorm`是全功能型的`ORM`，支持`MySQL`, `PostgreSQL`, `SQLite`, `SQL Server` 和 `TiDB`等不同的数据库。

使用`ORM`可让我们免于手写 SQL 语句，因为`ORM`会自动将对结构体的操作转化为对应的`SQL`语句。

使用`go get`安装`Gorm`以及连接`MySQL`数据库的驱动库：

```
$ go get -u gorm.io/gorm
$ go get gorm.io/driver/mysql


```

在`internal/util.go`文件中增加一个方法，用于初始化数据：

```
func InitDb() {
 dsn := "%s:%s@tcp(%s:%d)/%s?charset=utf8mb4&parseTime=True&loc=Local"

 dbConf := app.AppConfig.Database

 dsn = fmt.Sprintf(dsn, dbConf.User, dbConf.Password, dbConf.Host, dbConf.Port, dbConf.DbName)

 db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})

 if err != nil {
  panic("could not connect mysql server")
 }
 app.Db = db
}


```

在数据迁移时，需要数据库操作，所以我们将`cmd/migrate.go`文件修改为：

```
package cmd

import (
 "ms/app/models"
 "ms/internal"

 "github.com/spf13/cobra"
)

func init() {
 rootCmd.AddCommand(migrateCmd)
}

var migrateCmd = &cobra.Command{
 Use:   "migrate",
 Short: "数据表迁移工具",
 Long:  `主要用于将在项目创建时初始化数据表`,
 Run: func(cmd *cobra.Command, args []string) {
  internal.InitConfig()

  internal.InitDb()

  internal.Db.AutoMigrate(&models.User{})
 },
}


```

迁移时，需要为数据表创建对应的数据模型，我们放其放在`models/user.go`源文件中：

```
package models

type User struct {
 ID     int    `form:"id"`
 Name   string `form:"name"`
 Email  string `form:"email"`
 Gender uint8  `form:"gender"`
}


```

编译程序后，执行以下命令便可以创建`users`数据表了：

```
$ ./ms migrate


```

`cmd/server.go`文件的`server`命令也应该添加下面的代码：

```
util.InitDb()


```

初始化了数据库连接后，接下来我们通过用户新增，用户列表，用户详情等几个逻辑来讲解通过`Gorm`如何操作数据库：

**新增用户 (app/controller/user/add.go)：**

```
package user

import (
 "ms/app/models"
 "ms/internal"

 "github.com/gin-gonic/gin"
)

func Add(c *gin.Context) {

 var user models.User

 err := c.Bind(&user)

 if err != nil {
  internal.BadRequest(c)
 }

 err = internal.Db.Create(&user).Error

 if err != nil {
  internal.Logger.Error("添加用户出错：" + err.Error())
  internal.InternalError(c)
  return
 }

 if user.ID <= 0 {
  internal.Logger.Error("无法添加新用户：" + err.Error())
  internal.InternalError(c)
  return
 }
 internal.SimpleSuccess(c)
}


```

使用`curl`工具调用添加用户接口：

```
curl --location 'http://localhost:8080/user/add' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'name=test' \
--data-urlencode 'gender=1' \
--data-urlencode 'email=test@163.com'


```

调用结果：

```
{"Code":200,"Message":"执行成功","Data":null}


```

**用户列表 (app/controller/user/list.go)：**

```
package user

import (
 "ms/app/models"
 "ms/internal"

 "github.com/gin-gonic/gin"
)

func List(c *gin.Context) {
 var users []models.User
 internal.Db.Order("id Desc").Find(&users, nil)
 internal.ReturnData(c, users)
}



```

使用`curl`工具调用用户列表接口：

```
curl --location 'http://localhost:8080/user/list'


```

调用结果：

```
{"Code":200,"Message":"成功","Data":{"ID":1,"Name":"test","Email":"test@163.com","Gender":1}]}


```

**用户详情 (app/controller/user/detail.go)：**

```
package user

import (
 "ms/app/models"
 "ms/internal"

 "github.com/gin-gonic/gin"
)

func Detail(c *gin.Context) {
 id := c.Param("id")
 var user models.User
 internal.Db.Where("id = ?", id).First(&user)
 internal.ReturnData(c, user)
}


```

使用`curl`工具调用详情接口：

```
curl --location 'http://localhost:8080/user/1'


```

调用结果：

```
{"Code":200,"Message":"成功","Data":{"ID":1,"Name":"test","Email":"test@163.com","Gender":1}}


```

到这里我们的示例程序就完成了，最终项目的结构如下：

```
├── app
│   ├── controllers
│   │   └── user
│   │       ├── add.go
│   │       ├── detail.go
│   │       └── list.go
│   ├── models
│   │   └── user.go
│   └── router.go
├── cmd
│   ├── migrate.go
│   ├── root.go
│   └── server.go
├── config.yaml
├── go.mod
├── go.sum
├── internal
│   ├── app.go
│   ├── config.go
│   ├── response.go
│   └── util.go
└── main.go


```

当然，我们的示例程序还有很多改进的空间，如果你感兴趣的，可以查看示例源码，其`Github`地址为：

```
https://github.com/jinhongtl/ms


```

小结
--

好了，到这里，通过一个完整的示例程序，相信你应该对`Cobra`,`Viper`,`Gin`,`Gorm`,`Logrus`这几个库有所了解了吧，我们再总结一下这几个库的用途：

*   `Cobra`：构建命令行应用程序
    
*   `Viper`：一个强大的应用配置解决方案
    
*   `Gin`：非常快速的 Web 框架
    
*   `Gorm`：全功能 ORM
    
*   `Logrus`：非常流行的日志框架