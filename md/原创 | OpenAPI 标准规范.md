> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU0MzQ5MDA0Mw==&mid=2247492414&idx=1&sn=77e0ebdb4007c33d3460cb841a9b4a20&chksm=fb0809aacc7f80bc79c64dbaf4885f87820f58d247ec94062bd4c5ab43eb4a5f1bf53b9bcffb&scene=21#wechat_redirect)

> API 是模块或者子系统之间交互的接口定义。好的系统架构离不开好的 API 设计，而一个设计不够完善的 API 则注定会导致系统的后续发展和维护非常困难。

什么是 API 规范
----------

API 是模块或者子系统之间交互的接口定义。好的系统架构离不开好的 API 设计，而一个设计不够完善的 API 则注定会导致系统的后续发展和维护非常困难。在关键环节制定明确的 API 规范有助于 Service 对内提高产品间互通的效率，对外提供一致的使用体验，也有助于更好地被集成。

对于 API 规范，比较知名的是 OpenAPI Specfication[1] 和 Google API Design Guide[2]。前者针对 RESTful API 设计在细节层面给出了非常具体的规定，已经成为 RESTful API 设计领域的事实标准，而后者则主要从云厂商的角度提出许多最佳实践性质的规范与建议，这些原则不仅仅适用于 RESTful API，也适合其他类型 API 设计。

虽然 RESTful 设计风格曝光率很高，但并不是所有云服务商都选择了完全遵循 RESTful，例如 AWS 和 阿里云 [3] RPC 风格反而占了大多数，Google 和 Azure 则 RESTful 居多。

RESTful API 的优势是 HTTP 具备更好的易用性，让异构系统更容易集成，且开发执行效率比较高，面向资源要求也比较高。而 RPC API 可以使用更广泛的框架和方案，技术层面更底层也更为灵活，设计起来相对简单，掌握起来有一定门槛，架构上更加复杂。RESTful 与 RPC 模式对比如下：

<table data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)"><thead data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)"><tr data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); font-size: 15px; min-width: 85px;"><br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)|rgb(240, 240, 240)"></th><th data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); font-size: 15px; min-width: 85px;">RESTful API</th><th data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); font-size: 15px; min-width: 85px;">RPC API</th></tr></thead><tbody data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)"><tr data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">是否有统一规范</td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">HTTP</td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">无</td></tr><tr data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">面向资源</td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">是</td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">不确定</td></tr><tr data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">性能</td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">中</td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">高</td></tr><tr data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">通用性</td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">高</td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">弱</td></tr><tr data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">复杂度</td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">中</td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">高</td></tr></tbody></table>

如果强制统一风格，有些适合 RESTful 风格的服务非要使用 RPC 的话，看起来就会比较丑陋，如果只是一个过程化的服务调用，往 RESTful 资源化设计方向去靠会比较困难。但如果不强制使用统一风格，会造成针对 API 的体系化支持会更加复杂，例如为兼容两种风格 SDK 的自动化支持需要两套代码。

选择 API 风格时要考虑几个问题：

*   选择支持哪种风格，才能更好地体现业务特性，让客户操作起来更加方便；
    
*   设计 API 时能否面向资源设计，相应的工程人员是否具备做这种设计的能力；
    
*   针对这种风格工具链的支持是否到位，投入产出比如何；
    
*   业界流行的趋势如何，是否需要考虑与其他系统体系的互操作。
    

用户使用 API 来访问 Service，本质上是想通过对某种资源执行特定的操作来完成一个业务动作。对于资源有两个关键点：一是要有统一的资源模型；二是要明确资源关系。统一的资源模型对 Service 的帮助是巨大的：

*   它可以使 API 具有更清晰的结构，帮助用户理解；
    
*   它可以帮助对比 API 与后台实体关系模型，更容易提供更完整的 API 服务；
    
*   它可以使产品协作更加顺畅，对资源的操作也更加规范化；
    
*   它可以使云服务底层平台实现起来更统一、更方便；
    
*   它可以使围绕 API 的生态集成起来更加简单、高效。
    

确定了设计模式和资源模型后，就需要考虑 API 的设计细节了，诸如 API 名称、参数名、属性名称、数据格式、错误码之类的信息。除此之外，还要考虑以下一些问题：

*   在 API 命名的时候，遵循什么样的范式来确保大体风格相似？动词、名词、介词如何组合才能保持 API 风格看起来比较统一，降低理解成本？
    
*   对于类似的操作，有没有使用规范？有哪些公共的标准词汇使得同类型的操作可以比较容易理解，避免使用晦涩奇怪的词汇（例如读操作，Read/Query/Describe/List/Get 中都在什么场合使用什么动词）？
    
*   被广泛使用的参数如何尽可能保持一致，避免不同产品的表达混乱的情况（例如分页参数用 PageNumber 还是 PageNum）？
    
*   对于常用的场景，例如幂等、分页、异步 API 的设计有没有统一的规范，避免使用体验不一致？
    
*   错误码应该怎么设计？公共错误码怎么统一，业务错误码怎么表达？
    

上述问题都是实际研发过程中要注意的，要全部罗列的话远不止这些。API 的用词描述是 Service 展现给外部用户的第一印象，绝非随意写就。对人员有一定规模，内部有多条产品线的组织来说，如何协调组织的各个部分对外具有统一的体验是个很大挑战。

Service 在管理 API 时应该考虑一些具体的规范，对命名规则、标准词汇、最佳实践模式、错误码等信息都有明确的规定，同时用系统化、平台化的手段来管理 API，确保不走偏。设计风格不是云服务 API 设计中致命的问题，但是它关乎云服务外表形象，不可不察。

API 是后端服务的外部表达，是服务就有可能出现问题，无论这个问题是可预期的还是不可预期的。如果只考虑功能本身功能特性，而忽视对异常情况的设计，当问题出现的时候业务本身可能无法感知造成服务异常，更重要的是站在客户角度去看，不能有效获取错误原因是非常痛苦的，很多时候只能束手无策，降低云服务提供商的整体口碑，甚至损害营收。

OpenAPI 规范
----------

本规范基于 RESTful 风格的架构设计准则，广泛参考 GitHub、Azure、Google API Design Guide、腾讯云、阿里云等公开资料，兼顾现有实际情况和未来发展做一个概括性记录总结。

### 一、协议

API 与用户的通信协议，总是使用 HTTPS 协议。这个和 RESTful API 本身没有很大的关系，但是对于增加网站的安全是非常重要的。特别如果你提供的是公开 API，用户的信息泄露或者被攻击会严重影响网站的信誉。

### 二、版本（Version）

关于版本的设计有 3 种形式：

1.  将 API 的版本号放入 URL 中，如：`http://api.example.com/v1`，这样方便和直观；
    
2.  将版本号记录在 url query 中，如：`http://api.example.com?param1=val&version=1.0`中的 version 参数。
    
3.  将版本号放在 HTTP 头信息中，基于的准则是：不同的版本，可以理解成同一种资源的不同形式，所以应该采用同一个 URL。如：`Accept: application/json; version=1.0`，可以参考 Github API Design[4] 和 Versioning REST Services[5]；
    

根据现有的实际情况，如果是为了兼容已存在的服务接口，可以采用对应的形式。如果是新构建的体系结构，建议采用第三种。

### 三、Schema

URI 的格式定义如下：`URI = scheme "://" authority "/" path \[ "?" query \] \[ "#" fragment \]`

URL 是 URI 的一个子集 (一种具体实现)，对于 REST API 来说一个资源一般对应一个唯一的 URI（URL）。在 URL 的设计中，我们会遵循一些规则，使接口看起透明易读，方便使用者调用。

"/" 分隔符一般用来对资源层级的划分。对于 RESTful API 来说，"/" 只是一个分隔符，并无其他含义。为了避免混淆，"/" 不应该出现在 URL 的末尾。

**URL 中尽量使用连字符 "-" 代替下划线 "_" 的使用。** 连字符 "-" 一般用来分割 URL 中出现的字符串 (单词)，来提高 URL 的可读性，例如：`http://api.example.restapi.org/blogs/mark-masse/entries/this-is-my-first-post`。使用下划线 "_" 来分割字符串 (单词) 可能会和链接的样式冲突重叠，而影响阅读性。但实际上，"-" 和 "_" 对 URL 中字符串的分割语意上还是有些差异的："-" 分割的字符串 (单词) 一般各自都具有独立的含义，可参见上面的例子。而 "_" 一般用于对一个整体含义的字符串做了层级的分割，方便阅读，例如你想在 URL 中体现一个 IP 地址的信息：210_110_25_88 . (欢迎关注：朱小厮的博客)

**URL 应该统一使用小写字母**。

**URL 中不要包含文件 (脚本) 的扩展名**。例如 `.json` 之内的就不要出现了，对于接口来说没有任何实际的意义。如果是想对返回的数据内容格式标示的话，通过 HTTP Header 中的 Content-Type 字段更好一些。

对于响应返回的格式，JSON 因为它的可读性、紧凑性以及多种语言支持等优点，成为了 HTTP API 最常用的返回格式。因此，最好采用 JSON 作为返回内容的格式。如果用户需要其他格式，比如 `xml`，应该在请求头部 `Accept` 中指定。对于不支持的格式，服务端需要返回正确的 `status code`，并给出详细的说明。

**JSON 中的所有字段都应该用小写的蛇形命名形式，而不是采用驼峰命名。**

### 四、以资源为中心的 URL 设计

资源是 `Restful API` 的核心元素，所有的操作都是针对特定资源进行的。而资源就是 `URL`（Uniform Resoure Locator）表示的，所以简洁、清晰、结构化的 URL 设计是至关重要的。Github 可以说是这方面的典范，下面我们就拿 `repository` 来说明：

```
/users/:username/repos
/users/:org/repos
/repos/:owner/:repo
/repos/:owner/:repo/tags
/repos/:owner/:repo/branches/:branch
```

我们可以看到几个特性：

*   资源分为单个文档和集合，尽量使用复数来表示资源，单个资源通过添加 id 或者 name 等来表示
    
*   一个资源可以有多个不同的 URL
    
*   资源可以嵌套，通过类似目录路径的方式来表示，以体现它们之间的关系
    

**最常见的一种设计错误，就是 URL 包含动词。** 因为 "资源" 表示一种实体，所以应该是名词，URL 不应该有动词，动词应该放在 HTTP Method （参考下一条）中。举例来说，某个 URL 是 `/users/show/1`，其中 `show`是动词，这个 URL 就设计错了，正确的写法应该是 `/users/1`，然后用 HTTP GET 方法表示 show。

如果某些动作是 HTTP 动词表示不了的，你可以把动作看成是一种资源。比如网上汇款，从账户 1 向账户 2 汇款 500 元，错误的 URL 是：

```
POST /accounts/1/transfer/500/to/2
```

正确的写法是把动词 transfer 改成 transaction，资源不能是动词，但是可以是一种服务：

```
POST  /transactoin HTTP/1.1
HOST: 127.0.0.1

from=1&to=2&amount=500
```

### 五、正确使用 HTTP Method

有了资源的 URI 设计，所有针对资源的操作都是使用 HTTP 方法指定的。比较常用的 HTTP/1.1 动词有下面 5 个：

*   GET：从服务器取出资源（一项或多项）。
    
*   POST：在服务器新建一个资源。
    
*   PUT：在服务器更新资源（客户端提供改变后的完整资源）。
    
*   PATCH：在服务器更新资源（更新资源的部分属性）。
    
*   DELETE：从服务器删除资源。
    

还有 4 个不常用的 HTTP/1.1 动词：

*   HEAD：只获取某个资源的头部信息。比如只想了解某个文件的大小，某个资源的修改日期等
    
*   OPTIONS：获取信息，关于资源的哪些属性是客户端可以改变的。
    
*   TRACE：追踪路径。不建议使用。
    
*   CONNECT：要求用隧道协议连接代理。不建议使用。
    

举例：

```
GET /repos/:owner/:repo/issues
GET /repos/:owner/:repo/issues/:number
POST /repos/:owner/:repo/issues
PATCH /repos/:owner/:repo/issues/:number
DELETE /repos/:owner/:repo
```

这里顺带探讨一下，HTTP 协议涉及到的一种重要性质：幂等性 (Idempotence)。在 HTTP/1.1 规范中幂等性的定义是：

> Methods can also have the property of "idempotence" in that (aside from error or expiration issues) the side-effects of N > 0 identical requests is the same as for a single request.

从定义上看，HTTP 方法的幂等性是指一次和多次请求某一个资源应该具有同样的副作用。幂等性属于语义范畴，正如编译器只能帮助检查语法错误一样，HTTP 规范也没有办法通过消息格式等语法手段来定义它，这可能是它不太受到重视的原因之一。但实际上，幂等性是分布式系统设计中十分重要的概念，而 HTTP 的分布式本质也决定了它在 HTTP 中具有重要地位。

安全方法是指不修改资源的 HTTP 方法。譬如，当使用 GET 或者 HEAD 作为资源 URL，都必须不去改变资源。然而，这并不全准确。意思是：它不改变资源的表示形式。对于安全方法，它仍然可能改变服务器上的内容或资源，但这必须不导致不同的表现形式。

<table data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)"><thead data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)"><tr data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); font-size: 15px; min-width: 85px;"><br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)|rgb(240, 240, 240)"></th><th data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); font-size: 15px; min-width: 85px;">HTTP Method<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)|rgb(240, 240, 240)"></th><th data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); font-size: 15px; min-width: 85px;">幂等<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)|rgb(240, 240, 240)"></th><th data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); font-size: 15px; min-width: 85px;">安全<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)|rgb(240, 240, 240)"></th></tr></thead><tbody data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)"><tr data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">1<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)"></td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">OPTIONS<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)"></td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">yes<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)"></td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">yes<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)"></td></tr><tr data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">2<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)"></td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">GET<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)"></td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">yes<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)"></td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">yes<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)"></td></tr><tr data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">3<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)"></td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">HEAD<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)"></td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">yes<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)"></td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">yes<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)"></td></tr><tr data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">4<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)"></td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">PUT<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)"></td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">yes<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)"></td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">no<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)"></td></tr><tr data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">5<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)"></td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">POST<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)"></td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">no<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)"></td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">no<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)"></td></tr><tr data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">6<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)"></td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">DELETE<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)"></td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">yes<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)"></td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">no<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(248, 248, 248)"></td></tr><tr data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">7<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)"></td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">PATCH<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)"></td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">no<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)"></td><td data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); font-size: 15px; min-width: 85px;">no<br data-darkmode-color-16271990298745="rgb(163, 163, 163)" data-darkmode-original-color-16271990298745="#fff|rgb(0,0,0)|rgb(0,0,0)" data-darkmode-bgcolor-16271990298745="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16271990298745="#fff|rgb(255,255,255)"></td></tr></tbody></table>

实际上接口的幂等或者安全与否取决于接口的实现，只是 HTTP Method 语义上我们约定俗成地认为实现的过程会参照上表所示。对于幂等的接口，客户端就可以放心地多次调用，网关层面也可以重试。

### 六、状态码 （Status Code）

HTTP 应答中，需要带一个很重要的字段：`status code`。它说明了请求的大致情况，是否正常完成、需要进一步处理、出现了什么错误，对于客户端非常重要。状态码都是三位的整数，大概分成了几个区间：

*   `2XX`：请求正常处理并返回
    
*   `3XX`：重定向，请求的资源位置发生变化
    
*   `4XX`：客户端发送的请求有错误
    
*   `5XX`：服务器端错误
    

在 HTTP API 设计中，经常用到的状态码以及它们的意义如下表：

```
OK
```

上面这些状态码覆盖了 API 设计中大部分的情况，如果对某个状态码不清楚或者希望查看更完整的列表，可以参考 HTTP Status Code[6] 、维基百科 - 状态码 [7] 或者 RFC7231 Response Status Codes[8] 的内容。

### 七、错误处理（Error Handling）

如果出错，应该在 response body 中通过 `message` 给出明确的错误信息（一般来说，返回的信息中将 message 作为键名，出错详情作为键值即可）。比如客户端发送的请求有错误，一般会返回 `4XX Bad Request` 结果。这个结果很模糊，给出错误 `message` 的话，能更好地让客户端知道具体哪里有问题，进行快速修改。

```
{
 "message":"错误详情"
}
```

错误详情应该可以帮助用户轻松快捷地理解和解决 API 错误。通常，在编写错误详情时请考虑以下准则：

*   不要假设用户是您 API 的专家用户。用户可能是客户端开发人员、操作人员、IT 人员或应用的最终用户。
    
*   不要假设用户了解有关服务实现的任何信息，或者熟悉错误的上下文（例如日志分析）。
    
*   如果可能，应构建错误详情，以便技术用户（但不一定是 API 开发人员）可以响应错误并改正。
    
*   确保错误详情内容简洁。如果需要，请提供一个链接，便于有疑问的读者提问、提供反馈或详细了解错误详情中不方便说明的信息。此外，可使用详细信息字段来提供更多信息。
    

### 八、命名规则

为了能够长时间在众多 API 中为开发者提供一致的体验，API 使用的所有名称都应该具有以下特点：

*   简单
    
*   直观
    
*   一致
    

这包括接口、资源、集合、方法和消息的名称。

由于很多开发者不是以英语为母语，所以这些命名惯例的目标之一是确保大多数开发者可以轻松理解 API。对于方法和资源，我们鼓励使用简单、直观和少量的词汇来命名。

*   API 名称 **应该** 使用正确的美式英语。例如，使用美式英语的 license、color，而非英式英语的 licence、colour。
    
*   为了简化命名， **可以** 使用已被广泛接受的简写形式或缩写。例如，API 优于 Application Programming Interface。
    
*   尽量使用直观、熟悉的术语。例如，如果描述移除（和销毁）一个资源，则删除优于擦除。
    
*   使用相同的名称或术语命名同样的概念，包括跨 API 共享的概念。
    
*   避免名称过载。使用不同的名称命名不同的概念。
    
*   避免在 API 的上下文以及更大的 API 生态系统中使用含糊不清且过于笼统的名称。这些名称可能导致对 API 概念的误解。相反，应选择能准确描述 API 概念的名称。这对定义一阶 API 元素（例如资源）的名称尤其重要。没有要避免使用的名称的明确列表，因为每个名称都必须放在其他名称的上下文中进行评估。实例、信息和服务是过去有问题的名称的示例。所选择的名称应清楚地描述 API 概念（例如：什么的实例？），并将其与其他相关概念区分开（例如：“alert” 是指规则、信号还是通知？）。
    
*   仔细考虑是否使用可能与常用编程语言中的关键字相冲突的名称。您可以使用这些名称，但在 API 审核期间可能会触发额外的审查。因此应谨慎使用。
    

### 九、认证和授权（Authentication & Authorization）

一般来说，让任何人随意访问公开的 API 是不好的做法。验证和授权是两件事情：

*   验证（Authentication）是为了确定用户是其申明的身份，比如提供账户的密码。不然的话，任何人伪造成其他身份（比如其他用户或者管理员）是非常危险的
    
*   授权（Authorization）是为了保证用户有对请求资源特定操作的权限。比如用户的私人信息只能自己能访问，其他人无法看到；有些特殊的操作只能管理员可以操作，其他用户有只读的权限等等
    

如果没有通过验证（提供的用户名和密码不匹配，token 不正确等），需要返回 401 Unauthorized[9] 状态码，并在 body 中说明具体的错误信息；而没有被授权访问的资源操作，需要返回 403 Forbidden[10] 状态码，还有详细的错误信息。

> NOTES: 借鉴于 Github，它对某些用户未被授权访问的资源操作返回 404 Not Found[11]，目的是为了防止私有资源的泄露（比如黑客可以自动化试探用户的私有资源，返回 403 的话，就等于告诉黑客用户有这些私有的资源）。

### 十、限流（RateLimit）

如果对访问的次数不加控制，很可能会造成 API 被滥用，甚至被 DDOS 攻击 [12]。根据使用者不同的身份对其进行限流，可以防止这些情况，减少服务器的压力。

对用户的请求限流之后，要有方法告诉用户它的请求使用情况，本文档推荐使用的三个相关的头部：

*   `X-RateLimit-Limit`: 用户每个小时允许发送请求的最大值
    
*   `X-RateLimit-Remaining`：当前时间窗口剩下的可用请求数目
    
*   `X-RateLimit-Rest`: 时间窗口重置的时候，到这个时间点可用的请求数量就会变成 `X-RateLimit-Limit` 的值
    

举例：

```
X-Ratelimit-Limit: 18000
X-Ratelimit-Remaining: 17995
X-Ratelimit-Reset: 1590570990
```

如果允许没有登录的用户使用 API（可以让用户试用），可以把 `X-RateLimit-Limit` 的值设置得很小，比如 `60`。没有登录的用户是按照请求的 IP 来确定的，而登录的用户按照认证后的信息来确定身份。

对于超过流量的请求，可以返回 429 Too many requests[13] 状态码，并附带错误信息。

### 十一、编写优秀的文档

API 最终是给人使用的，不管是公司内部，还是公开的 API 都是一样。即使我们遵循了上面提到的所有规范，设计的 API 非常优雅，用户还是不知道怎么使用我们的 API。最后一步，但非常重要的一步是：为你的 API 编写优秀的文档。

对每个请求以及返回的参数给出说明，最好给出一个详细而完整地示例，提醒用户需要注意的地方…… 反正目标就是用户可以根据你的文档就能直接使用 API，而不是要发邮件给你，或者跑到你的座位上问你一堆问题。

### 参考资料

[1]

OpenAPI Specfication: _https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md_

[2]

Google API Design Guide: _https://cloud.google.com/apis/design_

[3]

阿里云: _https://bytedance.feishu.cn/docs/doccnrP6Ud3YTavonOlxds9lzad#I1tZbK_

[4]

Github API Design: _https://developer.github.com/v3/#current-version_

[5]

Versioning REST Services: _https://www.informit.com/articles/article.aspx?p=1566460_

[6]

HTTP Status Code: _https://httpstatuses.com/_

[7]

维基百科 - 状态码: _https://zh.wikipedia.org/wiki/HTTP%E7%8A%B6%E6%80%81%E7%A0%81_

[8]

RFC7231 Response Status Codes: _https://tools.ietf.org/html/rfc7231#section-6_

[9]

401 Unauthorized: _https://httpstatuses.com/401_

[10]

403 Forbidden: _https://httpstatuses.com/403_

[11]

404 Not Found: _https://httpstatuses.com/404_

[12]

DDOS 攻击: _https://en.wikipedia.org/wiki/Denial-of-service_attack_

[13]

429 Too many requests: _https://httpstatuses.com/429_