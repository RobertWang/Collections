> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/xixihahalelehehe/article/details/111373416)

### 文章目录

*   *   [1. Resty 简介](#1_Resty_4)
    *   [2. GET 方法](#2_GET_57)
    *   [3. POST 方法](#3_POST_96)
    *   [4. PUT 方法](#4_PUT_128)
    *   [5. 高级应用](#5__146)
    *   *   [5.1 代理](#51__147)
        *   [5.2 重试](#52__157)
    *   [6. 中间件](#6__178)

1. Resty 简介
-----------

微服务开发中服务间调用的主流方式有两种 HTTP、RPC，HTTP 相对来说比较简单。本文将使用 Resty 包来实现基于 HTTP 的微服务调用。

Resty 是一个简单的 HTTP 和 REST 客户端工具包，简单是指使用上非常简单。Resty 在使用简单的基础上提供了非常强大的功能，涉及到 HTTP 客户端的方方面面，可以满足我们日常开发使用的大部分需求。

go get 安装

```
go get github.com/go-resty/resty/v2

```

使用 Resty 提交 HTTP 请求

```
package main

import (
  "fmt"
  "github.com/go-resty/resty/v2"
)

func main() {
  client := resty.New()
  resp, err := client.R().Get("http://httpbin.org/get")
  fmt.Printf("\nError: %v", err)
  fmt.Printf("\nResponse Status Code: %v", resp.StatusCode())
  fmt.Printf("\nResponse Status: %v", resp.Status())
  fmt.Printf("\nResponse Time: %v", resp.Time())
  fmt.Printf("\nResponse Received At: %v", resp.ReceivedAt())
  fmt.Printf("\nResponse Body: %v", resp)   
}

```

执行输出：

```
Error: <nil>
Response Status Code: 200
Response Status: 200 OK
Response Time: 752.38195ms
Response Received At: 2020-12-18 16:04:16.211559068 +0800 CST m=+0.753544183
Response Body: {
  "args": {}, 
  "headers": {
    "Accept-Encoding": "gzip", 
    "Host": "httpbin.org", 
    "User-Agent": "go-resty/2.3.0 (https://github.com/go-resty/resty)", 
    "X-Amzn-Trace-Id": "Root=1-5fdc6280-1de9ea8f1558a5545c902521"
  }, 
  "origin": "212.129.130.147", 
  "url": "http://httpbin.org/get"
}

```

2. GET 方法
---------

```
client := resty.New()

resp, err := client.R().
    Get("https://httpbin.org/get")

resp, err := client.R().
      SetQueryParams(map[string]string{
          "page_no": "1",
          "limit": "20",
          "sort":"name",
          "order": "asc",
          "random":strconv.FormatInt(time.Now().Unix(), 10),
      }).
      SetHeader("Accept", "application/json").
      SetAuthToken("BC594900518B4F7EAC75BD37F019E08FBC594900518B4F7EAC75BD37F019E08F").
      Get("/search_result")

resp, err := client.R().
      SetQueryString("productId=232&template=fresh-sample&cat=resty&source=google&kw=buy a lot more").
      SetHeader("Accept", "application/json").
      SetAuthToken("BC594900518B4F7EAC75BD37F019E08FBC594900518B4F7EAC75BD37F019E08F").
      Get("/show_product")

resp, err := client.R().
      SetResult(AuthToken{}).
      ForceContentType("application/json").
      Get("v2/alpine/manifests/latest")

```

Resty 提供 SetQueryParams 方法设置请求的查询字符串，使用 SetQueryParams  
方法我们可以动态的修改请求参数。

*   `SetQueryString` 也可以设置请求的查询字符串，如果参数中有变量的话，需要拼接字符串。
*   `SetHeader` 设置请求的 HTTP 头，以上代码设置了 Accept 属性。
*   `SetAuthToken` 设置授权信息，本质上还是设置 HTTP 头，以上例子中 HTTP 头会附加 Authorization: BearerBC594900518B4F7EAC75BD37F019E08FBC594900518B4F7EAC75BD37F019E08F 授权属性。
*   `SetResult` 设置返回值的类型，Resty 自动解析 json 通过 resp.Result().(*AuthToken) 获取。

3. POST 方法
----------

```
client := resty.New()

resp, err := client.R().
      SetBody(User{Username: "testuser", Password: "testpass"}).
      SetResult(&AuthSuccess{}).
      SetError(&AuthError{}).
      Post("https://myapp.com/login")

resp, err := client.R().
      SetBody(map[string]interface{}{"username": "testuser", "password": "testpass"}).
      SetResult(&AuthSuccess{}).
      SetError(&AuthError{}).
      Post("https://myapp.com/login")

resp, err := client.R().
      SetHeader("Content-Type", "application/json").
      SetBody(`{"username":"testuser", "password":"testpass"}`).
      SetResult(&AuthSuccess{}).
      Post("https://myapp.com/login")

resp, err := client.R().
      SetHeader("Content-Type", "application/json").
      SetBody([]byte(`{"username":"testuser", "password":"testpass"}`)).
      SetResult(&AuthSuccess{}).
      Post("https://myapp.com/login")

```

POST 请求的代码和 GET 请求类似，只是最后调用了 Post 方法。POST 请求可以附带 BODY，代码中使用 SetBody 方法设置 POST BODY。  
`SetBody` 参数类型为结构体或 map[string]interface{} 时， Resty 自动附加 HTTP 头 Content-Type: application/json ，当参数为 string 或 []byte 类型时由于很难推断内容的类型，所以需要手动设置 Content-Type 请求头。 SetBody 还支持其他类型的参数，例如上传文件时可能会用到的 io.Reader。 `SetError` 设置 HTTP 状态码为 4XX 或 5XX 等错误时返回的数据类型。

4. PUT 方法
---------

```
client := resty.New()

resp, err := client.R().
      SetBody(Article{
        Title: "go-resty",
        Content: "This is my article content, oh ya!",
        Author: "Jeevanandam M",
        Tags: []string{"article", "sample", "resty"},
      }).
      SetAuthToken("C6A79608-782F-4ED0-A11D-BD82FAD829CD").
      SetError(&Error{}).
      Put("https://myapp.com/article/1234")

```

`PATCH`,`DELETE`,`HEAD`,`OPTIONS`请求也是一样的， Resty 为我们提供了一致的方式发起不同请求。

5. 高级应用
-------

### 5.1 代理

使用 Resty 作为 HTTP 客户端使用的话，添加代理似乎是一个常见的需求。 Resty 提供了 SetProxy 方法为请求添加代理，还可以调用 RemoveProxy 移除代理。代码如下：

```
client := resty.New()

client.SetProxy("http://proxyserver:8888")

client.RemoveProxy()

```

### 5.2 重试

```
client := resty.New()

client.
    SetRetryCount(3).
    SetRetryWaitTime(5 * time.Second).
    SetRetryMaxWaitTime(20 * time.Second).
    SetRetryAfter(func(client *resty.Client, resp *resty.Response) (time.Duration, error) {
        return 0, errors.New("quota exceeded")
    })

client.AddRetryCondition(
    func(r *resty.Response) (bool, error) {
        return r.StatusCode() == http.StatusTooManyRequests
    },
)

```

由于网络抖动带来的接口稳定性的问题 Resty 提供了重试功能来解决。以上代码我们可以看到 `SetRetryCount` 设置重试次数， `SetRetryWaitTime` 和 `SetRetryMaxWaitTime` 设置等待时间。 SetRetryAfter 是一个重试后的回调方法。除此之外还可以调用 AddRetryCondition 设置重试的条件。

6. 中间件
------

Resty 提供了和 Gin 类似的中间件特性。 `OnBeforeRequest` 和 `OnAfterResponse` 回调方法，可以在请求之前和响应之后加入自定义逻辑。参数包含了 resty.Client 和当前请求的 resty.Request 对象。成功时返回 nil ，失败时返回 error 对象。

```
client := resty.New()

client.OnBeforeRequest(func(c *resty.Client, req *resty.Request) error {

    return nil
})

client.OnAfterResponse(func(c *resty.Client, resp *resty.Response) error {

    return nil
})

```

参考链接：  
[https://github.com/go-resty/resty](https://github.com/go-resty/resty)