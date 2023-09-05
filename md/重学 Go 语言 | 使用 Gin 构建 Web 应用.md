> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/NjyGuS7W7XZUElQ33d05WQ)

Go 的一个主要使用方向便是`Web`开发，Go 社区中有非常多优秀的`Web`框架，`Gin`便是其中的一员，今天我们来学习如何使用`Gin`框架构建`Web`应用。

Gin 是什么？
--------

`Gin`是一个 Go 语言`Web`框架，类似`Martini`框架，但性能比`Martini`快速了 **40** 倍 (官方文档的说法)。

`Gin`有以下几个特征：

*   快速
    
*   支持中间件
    
*   支持从异常中恢复
    
*   `JSON`数据验证与解析
    
*   支持路由分组
    
*   支持各种响应格式，如`JSON`，`XML`等
    
*   易于扩展
    
*   完整的单元测试支持
    

使用 Gin 构建 Web 应用
----------------

首先，还是先创建并初始化一个 Go 项目：

```
$ mkdir gin-web
$ cd gin-web
$ go mod init ginweb


```

然后再使用`go get`命令安装`Gin`包：

```
$ go get -u github.com/gin-gonic/gin


```

接着，在`main.go`文件输出以下代码：

```
package main

import (
  "net/http"
  "github.com/gin-gonic/gin"
)

func main() {
  router := gin.Default()
  router.GET("/hello", func(c *gin.Context) {
    c.JSON(http.StatusOK, "hello,how are you!")
  })
  router.Run() 
}


```

再来就是调用`go run`命令运行上面的代码：

```
$ go run main.go


```

最后，通过`curl`命令调用：

```
$ curl http://localhost:8080/hello
hello,how are you!


```

从上面的例子可以看出，用`Gin`来开发`Web`应用是一件十分简单的事情。

案例分析
----

下面我们来分析上面的例子，总的来说这个例子由三个部分组成。

首先，是用`gin`的`Default()`函数创建一个`gin.Engine`对象`router`，另一种创建`gin.Engine`的方法是用`New()`函数：

```
router := gin.New()


```

`New()`函数与`Default()`函数的区别在于，`Default()`函数在创建时`gin.Engine`对象时，会添加一个日志输出和异常恢复的中间件，因此使用`Default()`函数就相当于：

```
router := gin.New()
router.Use(Logger(), Recovery())


```

接着，通过`gin.Engine`的`GET`方法设置处理`/hello`路由的请求，从方法名为`GET`我们可以知道这个方法只能用于处理`HTTP`的`get`方法请求。

`Gin`提供与`HTTP`方法相对应的方法用于处理对应的请求：

```
router.POST("/get",function(c *gin.Context){

})
router.POST("/post",function(c *gin.Context){

})
router.PUT("/put",function(c *gin.Context){

})
router.DELETE("/delete",function(c *gin.Context){

})
router.HEAD("/head",function(c *gin.Context){

})
router.PATCH("/patch",function(c *gin.Context){

})
router.OPTIONS("/options",function(c *gin.Context){

})


```

从上面的方法可以看出，处理请求的函数有固定的函数签名：

```
function(c *gin.Context)


```

调用`Handle`方法可以手动指定处理什么请求方法，比如下面我们通过`Handle`函数指定处理`post`请求：

```
router.Handle("post","/handle",func(c *gin.Context){

})


```

如果我们想要在一个函数里处理所有 HTTP 方法的请求，可以调用`Any`方法：

```
router.Any("/any",func(c *gin.Context){

})


```

如果我们想要在一个函数处理多个指定的 HTTP 方法请求，可以用`Match`方法：

```
router.Match(["post","get"],"/match",func(c *gin.Context){

})


```

`Gin`支持路由分组，这样可以将有统一前缀的路由划分在一个分组下：

```
userGroup := router.Group("/user")
userGroup.GET("/add", func(c *gin.Context) {
 c.JSON(200, "添加成功")
})


```

因为添加了路由分组，所以上面路由`add`的完整`url`为：

```
http://localhost:8080/user/add


```

如果一个分组下有多个路由，也可以用花括包含起来，代码结构更加清晰：

```
userGroup := router.Group("/user")
{
  userGroup.GET("/list", func(c *gin.Context) {
   c.JSON(200, "查询成功")
  })
  userGroup.GET("/add", func(c *gin.Context) {
   c.JSON(200, "添加成功")
  })
}


```

在处理请求的函数中，我们可以调用`Context`对象获取请求的数据或者响应用户的请求，比如在上面的例子中，我们用了`JSON`方法给用户发送`JSON`格式响应数据：

```
c.JSON(http.StatusOK, "hello,how are you!")


```

例子的最后，通过`gin.Engine`的`Run()`方法，启动了`Web`服务器，`Gin`框架的`Web`服务默认启动的端口是`8080`，我们也可以通过`Run()`函数的参数修改端口号：

```
router.Run(":8081")


```

另一种启动方式是将`gin.Engine`作为参数传给`net/http`的`ListenAndServe`函数：

```
http.ListenAndServe(":8080", router)


```

之所以可以将类型为`gin.Engine`的`router`传给`net/http`的`ListenAndServe`函数，是因为`gin.Engine`实现了`http.Handler`接口：

```
type Handler interface {
 ServeHTTP(ResponseWriter, *Request)
}


```

小结
--

使用`Gin`框架，我们能轻松创建响应`POST`，`GET`，`PUT`，`DELETE`等各种`HTTP`方法的请求，同时也可以向客户端发送`JSON`,`HTML`模板，`XML`等不同的格式的响应数据。