> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU0OTE4MzYzMw==&mid=2247535737&idx=2&sn=ff07f2db3d15bede4cd067bc61f3c4ba&chksm=fbb1c587ccc64c91271dff3657299b92b89ea6151290ffcca480670f8aba00ee217e469bad76&mpshare=1&scene=1&srcid=02142gVjUVgauD2SGyitUU8x&sharer_sharetime=1644813104799&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

曾经因为一个糟糕的 API 而感到沮丧吗？

  

在这个微服务的世界里，后端 API 的一致性设计是必不可少的。

  

今天，我们将讨论一些可遵循的最佳实践。我们将保持简短和甜蜜——所以系好安全带，出发咯！

1. 对 URL 使用 kebab-case（短横线小写隔开形式）

例如，如果你想要获得订单列表。

  

不应该：

应该：

```
/system-orders
```

  

2. 参数使用 camelCase（驼峰形式）

例如，如果你想从一个特定的商店购买产品。

  

不应该：

```
/system-orders/{order_id}
```

或者：

```
/system-orders/{OrderId}
```

应该：

```
/system-orders/{orderId}
```

  

3. 指向集合的复数名称

如果你想获得系统的所有用户。

  

不应该：

```
GET /user
```

或：

```
GET /User
```

应该：

```
GET /users
```

  

4. URL 以集合开始，以标识符结束

如果要保持概念的单一性和一致性。

  

不应该：

```
GET /shops/:shopId/category/:categoryId/price
```

这很糟糕，因为它指向的是一个属性而不是资源。

  

应该：

```
GET /shops/:shopId/或GET /category/:categoryId
```

  

5. 让动词远离你的资源 URL

不要在 URL 中使用动词来表达你的意图。相反，使用适当的 HTTP 方法来描述操作。

  

不应该：

```
POST /updateuser/{userId}
```

或：

```
GET /getusers
```

应该：

```
PUT /user/{userId}
```

  

6. 对非资源 URL 使用动词

如果你有一个端点，它只返回一个操作。在这种情况下，你可以使用动词。例如，如果你想要向用户重新发送警报。

  

应该：

```
POST /alarm/245743/resend
```

请记住，这些不是我们的 CRUD 操作。相反，它们被认为是在我们的系统中执行特定工作的函数。

7. JSON 属性使用 camelCase 驼峰形式

如果你正在构建一个请求体或响应体为 JSON 的系统，那么属性名应该使用驼峰大小写。

  

不应该：

```
{
   user_name: "Mohammad Faisal"
   user_id: "1"
}
```

应该：

```
{
   userName: "Mohammad Faisal"
   userId: "1"
}
```

  

8. 监控

RESTful HTTP 服务必须实现 / health 和 / version 和 / metricsAPI 端点。他们将提供以下信息。

  

**/health**

  

用 200 OK 状态码响应对 / health 的请求。

  

**/version**

  

用版本号响应对 / version 的请求。

  

**/metrics**

  

这个端点将提供各种指标，如平均响应时间。

  

也强烈推荐使用 / debug 和 / status 端点。

9. 不要使用 table_name 作为资源名

不要只使用表名作为资源名。从长远来看，这种懒惰是有害的。

  

不应该：

```
product_order
```

应该：

```
product-orders
```

这是因为公开底层体系结构不是你的目的。

10. 使用 API 设计工具

有许多好的 API 设计工具用于编写好的文档，例如：

  

*   API 蓝图：https://apiblueprint.org/
    
*   Swagger：https://swagger.io/
    

  

![](https://mmbiz.qpic.cn/mmbiz_png/fEsWkVrSk564lAurSvwmSY5Dz0YldeK3vAGpxQqic9V0WRL3swd3app9S7gfNib1YSiazOgEWADqfo4iaYA9bS3CSQ/640?wx_fmt=png)

  

拥有良好而详细的文档可以为 API 使用者带来良好的用户体验。

11. 使用简单序数作为版本

始终对 API 使用版本控制，并将其向左移动，使其具有最大的作用域。版本号应该是 v1，v2 等等。

  

应该：http://api.domain.com/v1/shops/3/products

  

始终在 API 中使用版本控制，因为如果 API 被外部实体使用，更改端点可能会破坏它们的功能。

12. 在你的响应体中包括总资源数

如果 API 返回一个对象列表，则响应中总是包含资源的总数。你可以为此使用 total 属性。

  

不应该：

```
{
  users: [ 
     ...
  ]
}
```

应该：

```
{
  users: [ 
     ...
  ],
  total: 34
}
```

  

13. 接受 limit 和 offset 参数

在 GET 操作中始终接受 limit 和 offset 参数。

  

应该：

```
GET /shops?offset=5&limit=5
```

这是因为它对于前端的分页是必要的。

14. 获取字段查询参数

返回的数据量也应该考虑在内。添加一个 fields 参数，只公开 API 中必需的字段。

  

例子：

  

只返回商店的名称，地址和联系方式。

```
GET /shops?fields=id,name,address,contact
```

在某些情况下，它还有助于减少响应大小。

15. 不要在 URL 中通过认证令牌

这是一种非常糟糕的做法，因为 url 经常被记录，而身份验证令牌也会被不必要地记录。

  

不应该：

```
GET /shops/123?token=some_kind_of_authenticaiton_token
```

相反，通过头部传递它们：

```
Authorization: Bearer xxxxxx, Extra yyyyy
```

此外，授权令牌应该是短暂有效期的。

16. 验证内容类型

服务器不应该假定内容类型。例如，如果你接受 application/x-www-form-urlencoded，那么攻击者可以创建一个表单并触发一个简单的 POST 请求。

  

因此，始终验证内容类型，如果你想使用默认的内容类型，请使用：

```
content-type: application/json
```

  

17. 对 CRUD 函数使用 HTTP 方法

HTTP 方法用于解释 CRUD 功能。

  

GET：检索资源的表示形式。

  

POST：创建新的资源和子资源。

  

PUT：更新现有资源。

  

PATCH：更新现有资源，它只更新提供的字段，而不更新其他字段。

  

DELETE：删除已存在的资源。

18. 在嵌套资源的 URL 中使用关系

以下是一些实际例子：

  

*   GET /shops/2/products：从 shop 2 获取所有产品的列表。
    
*   GET /shops/2/products/31：获取产品 31 的详细信息，产品 31 属于 shop 2。
    
*   DELETE /shops/2/products/31：应该删除产品 31，它属于商店 2。
    
*   PUT /shops/2/products/31：应该更新产品 31 的信息，只在 resource-URL 上使用 PUT，而不是集合。
    
*   POST /shops：应该创建一个新的商店，并返回创建的新商店的详细信息。在集合 url 上使用 POST。
    

19. CORS（跨源资源共享）

一定要为所有面向公共的 API 支持 CORS（跨源资源共享）头部。

  

考虑支持 CORS 允许的 “*” 来源，并通过有效的 OAuth 令牌强制授权。

  

避免将用户凭证与原始验证相结合。

20. 安全

在所有端点、资源和服务上实施 HTTPS（tls 加密）。

  

强制并要求所有回调 url、推送通知端点和 webhooks 使用 HTTPS。

21. 错误

当客户端向服务发出无效或不正确的请求，或向服务传递无效或不正确的数据，而服务拒绝该请求时，就会出现错误，或者更具体地说，出现服务错误。

  

例子包括无效的身份验证凭证、不正确的参数、未知的版本 id 等。

  

*   当由于一个或多个服务错误而拒绝客户端请求时，一定要返回 4xx HTTP 错误代码。
    
*   考虑处理所有属性，然后在单个响应中返回多个验证问题。
    

22. 黄金法则

如果您对 API 格式的决定有疑问，这些黄金规则可以帮助我们做出正确的决定。

  

*   扁平比嵌套好。
    
*   简单胜于复杂。
    
*   字符串比数字好。
    
*   一致性比定制更好。
    

  

就是这样——如果你已经走到了这一步，恭喜你！希望你学到了一些东西。