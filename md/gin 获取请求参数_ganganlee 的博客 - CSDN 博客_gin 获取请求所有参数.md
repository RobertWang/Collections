> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_44540711/article/details/105131504)

获取 GET 请求
=========

方法一
---

```
r.GET("/parse/url", func(c *gin.Context) {
		//获取name参数，获取不到返回空
		name := c.Query("name")
		c.JSON(http.StatusOK,gin.H{
			"name":name,
		})
	})

```

方法二
---

```
r.GET("/parse/url", func(c *gin.Context) {
		//获取name参数，获取不到返回默认值
		name := c.DefaultQuery("name","king")
		c.JSON(http.StatusOK,gin.H{
			"name":name,
		})
	})

```

方法三
---

```
r.GET("/parse/url", func(c *gin.Context) {
		//获取name参数，存在返回true反之返回false
		name, ok := c.GetQuery("name")
		if !ok {
			fmt.Println("参数不存在")
			return
		}
		c.JSON(http.StatusOK,gin.H{
			"name":name,
		})
	})

```

获取 POST 请求
==========

方法一
---

```
//获取post请求，不存在返回空
	r.POST("/post", func(c *gin.Context) {
		name := c.PostForm("name")
		passwd := c.PostForm("pass")
		c.JSON(http.StatusOK,gin.H{
			"name":name,
			"passwd":passwd,
		})
	})

```

方法二
---

```
//获取post请求，不存在返回设置的默认值
//注意，是此字段未设置才会返回默认值，字段存在值为空时返回空
	r.POST("/post", func(c *gin.Context) {
		name := c.DefaultPostForm("name","root")
		passwd := c.DefaultPostForm("pass","123")
		c.JSON(http.StatusOK,gin.H{
			"name":name,
			"passwd":passwd,
		})
	})

```

方法三
---

```
//获取post请求，不存在返回false
//注意，是此字段未设置才会返回false，字段存在值为空时返回true
	r.POST("/post", func(c *gin.Context) {
		name, ok := c.GetPostForm("name")
		if !ok {
			fmt.Printf("name is not undefined")
			return
		}
		passwd, ok := c.GetPostForm("pass")
		if !ok {
			fmt.Printf("pass is not undefined")
			return
		}
		c.JSON(http.StatusOK,gin.H{
			"name":name,
			"passwd":passwd,
		})
	})

```

获取地址栏参数
=======

```
//获取地址栏参数
	r.GET("/param/:name/:age", func(c *gin.Context) {
		name := c.Param("name")
		age := c.Param("age")
		c.JSON(http.StatusOK,gin.H{
			"name":name,
			"age":age,
		})
	})

```

通过结构体绑定获取参数
===========

1 定义结构体
-------

```
//主意需要指定tag
type param struct {
	Name string `form:"name"`
	Age  int    `form:"age"`
}

```

2 获取数据
------

```
//通过结构体获取数据
r.POST("/struct", func(c *gin.Context) {
	//声明接收变量
	var arg param

	//传入结构体接受值,这里需要传入地址
	err := c.ShouldBind(&arg)
	if err != nil {
		c.JSON(http.StatusBadRequest,gin.H{
			"msg":err.Error(),
		})
	}else {
		c.JSON(http.StatusOK,arg)
	}
})

```