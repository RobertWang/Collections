> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/15f71UCbb-9YFwH_fGhz1w)

什么是 Rest
========

*   • 客户端 - 服务器：客户端和服务器是通过标准化接口进行通信的独立实体。
    
*   • 无状态：服务器不存储有关客户端状态的任何信息。来自客户端的每个请求都必须包含服务器处理它所必需的所有信息。**例如，如果要做对请求做用户验证，通常都要在请求头附带用户 token**
    
*   • 可缓存：服务器可以向客户端指示响应是否可以缓存，以提高性能并减少网络流量。
    
*   • 统一接口：服务器公开了一个统一一致的接口，用于访问和操作资源，使用 HTTP 方法（GET、POST、PUT、DELETE 等）和状态码（200 OK、404 Not Found 等）。
    
*   • 分层系统：服务器可以由对客户端透明的多层组件组成。
    
*   • 按需代码（可选）：服务器可以选择性地向客户端发送可执行代码，例如 JavaScript 或 Java applet，以扩展其功能。
    

REST 全称是**表述性状态转移**，那究竟指的是什么的表述? 其实指的就是**资源**。任何事物，只要有被引用到的必要，它就是一个资源。

什么是资源？
======

资源是任何可以通过 URI（统一资源标识符）标识的东西，例如用户、产品、博客文章等。资源可以有不同的表示形式，例如 JSON、XML、HTML 等。例如，资源 https://example.com/users/1 可以表示为 JSON 对象：

```
{
  "id": 1,
  "name": "Alice",
  "email": "alice@example.com"
}

```

或者作为 XML 文档：

```
<user>
  <id>1</id>
  <name>Alice</name>
  <email>alice@example.com</email>
</user>

```

或者作为 HTML 页面：

```
<h1>User Profile</h1>
<p>ID: 1</p>
<p>Name: Alice</p>
<p>Email: alice@example.com</p>

```

**如何使用 PHP 创建 RESTful Web 服务？**
===============================

任何一个 RESTful Web 服务，创建时都至少需要做以下 4 部分工作：

*   • 定义您的资源及其 URI
    
*   • 为每个资源实现 HTTP 方法
    
*   • 处理请求和响应格式
    
*   • 处理错误和异常
    

定义您的资源及其 URI
------------

第一步是确定您的 Web 服务将提供的资源并为它们分配有意义的 URI。例如，假设我们要创建一个用于管理博客文章的 Web 服务。您可以定义以下资源：

*   • /posts ：所有文章的集合
    
*   • /posts/{id} ：由其 ID 标识的单个文章
    
*   • /posts/{id}/comments ：一篇文章的评论集合
    
*   • /posts/{id}/comments/{id} ：博客文章的单个评论
    

**这里我们应该使用名词来命名资源并避免使用动词。还应该对集合使用复数形式**，对单个资源使用单数形式。应该使用路径参数 ({id} ) 来标识集合中的特定资源。

为每个资源实现 HTTP 方法
---------------

下一步是使用适当的 HTTP 方法为每个资源实现逻辑。 HTTP 方法是：

*   • GET：用于检索资源或资源集合的表示。
    
*   • POST：用于创建新资源或对资源或资源集合执行操作。
    
*   • PUT：用于更新或替换现有资源或资源集合。
    
*   • DELETE：用于删除现有资源或资源集合。
    

例如，可以为 /posts 资源实现以下方法：

*   • GET /posts：返回 JSON 格式的所有博客文章列表。
    
*   • POST /posts：使用请求正文中提供的 JSON 格式数据创建新博客文章，并以 JSON 格式返回创建的文章。
    
*   • PUT /posts：使用请求正文中提供的 JSON 格式数据更新或替换所有博客文章，并以 JSON 格式返回更新后的文章。
    
*   • DELETE /posts：删除所有博客文章并返回空响应。
    

*   • 200 OK：请求成功，响应中包含请求的数据。
    
*   • 201 Created：请求成功，创建了新的资源。响应包含创建的资源及其位置标头。
    
*   • 204 No Content：请求成功但没有数据返回。
    
*   • 400 Bad Request：请求无效或格式错误，服务器无法处理。
    
*   • 401 Unauthorized：请求需要身份验证但未提供或无效。
    
*   • 403 Forbidden：请求已通过身份验证，但无权访问或修改资源。
    
*   • 404 Not Found：请求的资源不存在或服务器未找到。
    
*   • 405 Method Not Allowed：请求的方法不受资源支持或服务器配置不允许。
    
*   • 500 Internal Server Error：服务器在处理请求时遇到意外错误。
    

处理请求和响应格式
---------

下一步是处理 Web 服务的请求和响应格式。应该根据客户的需要支持不同的格式。例如，可以支持 JSON、XML、HTML 等。也应该使用内容协商来根据请求的 Accept 标头和响应的 Content-Type 标头来确定要使用的格式。  
例如，如果你想为你的网络服务支持 JSON 和 XML 格式，你可以这样写：

```
// Get the accept header from the request
$accept = $_SERVER['HTTP_ACCEPT'];

// Set default format to JSON
$format = 'json';

// Check if XML format is requested
if (strpos($accept,'application/xml') !== false) {
  // Set format to XML
  $format = 'xml';
}

// Set content type header according to format
header('Content-Type: application/' . $format);

// Encode data according to format
if ($format == 'json') {
  // Encode data as JSON
  echo json_encode($data);
} else if ($format == 'xml') {
  // Encode data as XML
  echo xml_encode($data); // You need to implement this function yourself
}

```

还应该根据客户将数据发送到 Web 服务的方式来处理不同的请求格式。例如，您可以支持 JSON、XML、表单数据等。您应该再次使用内容协商来根据请求的 Content-Type 标头来确定使用哪种格式。

```
// Get content type header from request
$content_type = $_SERVER['CONTENT_TYPE'];

// Set default format to JSON
$format = 'json';

// Check if XML format is sent
if (strpos($content_type,'application/xml') !== false) {
    // Set format to XML
    $format = 'xml';
}

// Decode data according to format
if ($format == 'json') {
    // Decode data from JSON
    $data = json_decode(file_get_contents('php://input'), true);
} else if ($format == 'xml') {
    // Decode data from XML
    $data = xml_decode(file_get_contents('php://input')); // You need to implement this function yourself
}

```

当然，现代 php 开发都基于如 Laravel 这样的知名框架来开发，在框架里面，这些请求格式，请求头都非常容易获取，一般都是通过使用中间件技术来实现

处理错误和异常
-------

最后一步是处理处理请求时可能出现的任何错误和异常。您应该使用 try-catch 块来捕获您的代码或外部库可能抛出的任何异常。您还应该使用错误处理程序来捕获可能由 PHP 本身触发的任何错误。您应该在响应中返回有意义的错误消息和状态代码。  
例如，您可以使用这样的东西：

```
// Set error handler function
set_error_handler('error_handler');

// Define error handler function
function error_handler($errno, $errstr) {
    // Log error message
    error_log($errstr);

    // Set status code header to 500 Internal Server Error
    http_response_code(500);

    // Return error message in JSON format
    echo json_encode(['error' => $errstr]);
}

// Use try-catch block around your code
try {
    // Your code here ...
} catch (Exception $e) {
    // Log exception message
    error_log($e->getMessage());

    // Set status code header according to exception type
    switch (get_class($e)) {
        case 'InvalidArgumentException':
            http_response_code(400);
            break;
        case 'UnauthorizedException':
            http_response_code(401);
            break;
        case 'ForbiddenException':
            http_response_code(403);
            break;
        case 'NotFoundException':
            http_response_code(404);
            break;
        default:
            http_response_code(500);
            break;
    }

    // Return exception message in JSON format
    echo json_encode(['error' => $e->getMessage()]);
}

```

在现代 php 开发中，通过使用框架，**使用框架中的自动异常捕获技术，只需做一些简单的配置，就可以轻松处理程序错误和运行时异常**

总结
==

在本文中，我们学习了如何使用 PHP 创建简单的 RESTful Web 服务。学习了如何定义资源及其 URI、如何为每个资源实现 HTTP 方法、如何处理请求和响应格式以及如何处理错误和异常。还看到了一些说明这些概念的代码示例。  
使用 PHP 创建 RESTful Web 服务并不难，但需要注意细节和一些最佳实践。在设计和实现 Web 服务时，**应该始终遵循 REST 原则和 HTTP 约定**。还应该做好 Web 服务的测试，感谢你的阅读，希望本文能够帮助到你！