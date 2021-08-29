> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/o7evv4Br9ocDLclMLcDM5w)

**1. 前言**

**1.1 GraphQL 作为网关层**

如果你对 GraphQL 还不了解，可以通过我们团队的讲座和文章进一布了解：

*   官方网站
    

*   GraphQL 官网（https://graphql.cn/）
    

*   Apollo 官网（https://apollographql.com/）
    

*   讲座与 Demo（第 3 场）
    

*   用 GraphQL 控制你的特斯拉真车
    

*   https://www.yuque.com/zaotalk/posts/s9
    

*   技术文章
    

*   聊聊 GraphQL 和 Apollo 的工作流
    

*   https://zhuanlan.zhihu.com/p/115068436
    

*   还在用 Redux，要不要试试 GraphQL 和 Apollo？
    

*   https://zhuanlan.zhihu.com/p/34238617
    

*   TypeScript + GraphQL = TypeGraphQL
    

*   https://zhuanlan.zhihu.com/p/56516614
    

*   基于 GraphQL 的数据导出
    

*   https://zhuanlan.zhihu.com/p/46141806
    

通过我们团队 4 年的持续努力，现如今在 CCO 技术部，GraphQL 已经成为了 API 对内对外描述、暴露及调用的唯一标准。而在国外，Facebook、Netflix、Github、Paypal、微软、大众、沃尔玛等企业也在大规模使用 GraphQL 中，甚至让以 GraphQL 为生的 Apollo 公司成功拿下了 1.3 亿美元的 D 轮融资。在面向全球前端开发者调研问卷中，GraphQL 也成为最受关注的技术和最想学习的技术。Github 上有一份持续更新的 GraphQL 公开服务列表。

![](https://mmbiz.qpic.cn/mmbiz_jpg/QRibyjewM1IDTTjMlrc9NLiccx10iahqsib1LFQJ1TGW9qdV9mlZvYjQqZtUXgmMHe84GskicUicoF1M5iadA4swL73ibQ/640?wx_fmt=jpeg)

我们认为 GraphQL 最适合的场景莫过于作为 BFF（Backend for Frontend）的网关层，即根据客户端的实际需要，将后端的原始 HSF 接口、第三方 RESTful 接口进行整合和封装形成自己的 Service Façade 层。GraphQL 自身的特性由、使得其非常容易与 RESTful、MTOP/MOPEN 等基于 HTTP 的现有网关进行集成，而另一方面，在国外很多文章中都提到 GraphQL 非常适合作为 Serverless/FaaS 的网关层，你甚至只需要唯一一个 HTTP Trigger 就能实现代理所有背后的 API。

**1.2 GraphQL 网关与 CDN 边缘计算**

EdgeRoutine 边缘计算 是阿里云 CDN 团队推出的新一代 Serverless 计算平台，它提供了一个类似 W3C 标准的 ServiceWorker 容器，可以充分利用 CDN 遍布全球的节点空闲计算资源以及强大的加速与缓存能力，实现高可用性、高性能的分布式弹性计算，更重要的是目前对于弹内用户来说它是**完全免费**的，当然截止至笔者发稿时 EdgeRoutine 还处在试用阶段。EdgeRoutine 将在 8 月底 9 月初正式对外发布！

在 1.1 节中我们提到 GraphQL 非常适合作为 BFF 网关层，而结合电商后台业务的特点我们发现：

> Query 类的请求占了大量的比例，而这些只读类查询请求，通常响应结果在相当长的时间范围甚至是永远都不会发生变化，尽管如此，每一次 API 调用时我们还是将请求发送到了后端的应用 / 服务器上。

这让我们产生了一个全新的思路：

![](https://mmbiz.qpic.cn/mmbiz_png/QRibyjewM1IDTTjMlrc9NLiccx10iahqsib1OqeswlZSibebJVWEdSSTUnxUjbghakXibTJRuvUbh9bFMWhGQhXYHVtg/640?wx_fmt=png)

如上图所示，将 CDN EdgeRoutine 作为 GraphQL Query 类请求的代理层，首次执行 Query 时，我们将请求先从 CDN 代理到 GraphQL 网关层，再通过网关层代理到实际的应用服务（例如通过 HSF 调用），然后将获得的返回结果缓存在 CDN 上，之后的请求可以根据 TTL 业务规则动态决定走缓存还是去 GraphQL 网关层。这样我们可以充分利用 CDN 的特性，将查询类请求分散到遍布全球的节点中，显著降低主应用程序的 QPS。

**2. 移植 Apollo GraphQL Server**

Apollo GraphQL Server 是目前使用最广泛的开源 GraphQL 服务，它的 Node.js 版本 更是被 BFF 类应用广为使用。但是遗憾的是 apollo-server 是一个面向 Node.js 技术栈开发的项目，而前文中提到 EdgeRoutine 提供的是一个类似 Service Worker 的 Serverless 容器，因此我们首先需要做的就是将 apollo-server-core 移植到 EdgeRoutine 中。为此，我开发了 apollo-server-edge-routine，本章节将简述设计和实现思路。

### 2.1 构建 TypeScript 开发环境和脚手架

首先，我们需要构建一个 EdgeRoutine 容器的 TypeScript 环境，此前我已经开发了 EdgeRoutine TypeScript 描述和 EdgeRoutine TypeScript 脚手架及本地模拟器（在 EdgeRoutine 正式上线后，我会开源到 Github 上），因此可以快速构建一个本地开发环境。这里简单解释一下，我实际上是用 Service Worker 的 TypeScript 库来模拟编译时环境，同时将 Webpack 作为本地调试服务器，并用浏览器的 Service Worker 来模拟运行 edge.js 脚本，用 Webpack 的 socket 通讯实现 Hot Reload 效果。

### 2.2 为 EdgeRoutine 环境实现自己的 ApolloServer

Apollo 官方似乎并没有给出如何移植 Apollo Server 的文档，不过简单研究了一下 ApolloServerBase 的代码，不难发现其实它已经是一个功能完备的服务器了，只是缺少与 HTTP 服务器的连接。因此，我们只要集成该类，并实现一个自己的 `listen(path: string)` 方法即可，这里的 `listen()` 方法与传统 HTTP 服务器不同，我们需要指定的不是 `port` 而是一个 `path`，也就是需要侦听 GraphQL 请求的路径。下面是我实现的一个简单版本：

```
import { ApolloServerBase } from 'apollo-server-core';
import { handleGraphQLRequest } from './handlers';
/**
 * Apollo GraphQL Server 在 EdgeRoutine 上的实现。
 */
export class ApolloServer extends ApolloServerBase {
  /**
   * 在指定的路径上，侦听 GraphQL Post 请求。
   * @param path 指定要侦听的路径。
   */
  async listen(path = '/graphql') {
    // 如果在未调用 `start()` 方法前，错误的先使用了 `listen()` 方法，则抛出异常。
    this.assertStarted('listen');
    // addEventListenr('fetch', (FetchEvent) => void) 由 EdgeRoutine 提供。
    addEventListener('fetch', async (event: FetchEvent) => {
      // 侦听 EdgeRoutine 的所有请求。
      const { request } = event;
      if (request.method === 'POST') {
        // 只处理 POST 请求
        const url = new URL(request.url);
        if (url.pathname === path) {
          // 当路径相符合时，将请求交给 `handleGraphQLRequest()` 处理
          const options = await this.graphQLServerOptions();
          event.respondWith(handleGraphQLRequest(this, request, options));
        }
      }
    });
  }
}
```

接下来，我们需要实现核心的 `handleGraphQLRequest()` 方法，该方法实际上是一个通道模式，负责将 HTTP 请求转换成 GraphQL 请求发送到 Apollo Server，并将其返回的 GraphQL 响应转换回 HTTP 响应。Apollo 官方其实是有一个名为 `runHttpQuery()` 的类似方法，但是该方法用到了 `buffer` 等 Node.js 环境内置的模块，因此无法在 Service Worker 环境中编译通过。这里给出一个我自己的简单实现：

```
import { GraphQLOptions, GraphQLRequest } from 'apollo-server-core';
import { ApolloServer } from './ApolloServer';
/**
 * 从 HTTP 请求中解析出 GraphQL 查询并执行，再将执行的结果返回。
 */
export async function handleGraphQLRequest(
  server: ApolloServer,
  request: Request,
  options: GraphQLOptions,
): Promise<Response> {
  let gqlReq: GraphQLRequest;
  try {
    // 从 HTTP request body 中解析出 JSON 格式的请求。
    // 该请求是一个 GraphQLRequest 类型，包含 query、variables、operationName 等。
    gqlReq = await request.json();
  } catch (e) {
    throw new Error('Error occurred when parsing request body to JSON.');
  }
  // 执行 GraphQL 操作请求。
  // 当执行失败时不会抛出异常，而是返回一个包含 `errors` 的响应。
  const gqlRes = await server.executeOperation(gqlReq);
  const response = new Response(JSON.stringify({ data: gqlRes.data, errors: gqlRes.errors }), {
    // 永远确保 content-type 为 JSON 格式。
    headers: { 'content-type': 'application/json' },
  });
  // 将 GraphQLResponse 中的消息头复制到 HTTP Response 中。
  for (const [key, value] of Object.entries(gqlRes.http.headers)) {
    response.headers.set(key, value);
  }
  return response;
}
```

**3. 一个简单的天气查询 GraphQL CDN 代理网关示例**

### 3.1 我们要做什么

在这个 Demo 里，我们假设要对第三方天气服务进行二次封装。我们将会为天气 API 网（tianqiapi.com）开发一个 GraphQL CDN 代理网关。天气 API 网对免费用户的 QPS 有一定的限制，每天只能 300 次查询，既然天气预报一般变化频率较低，我们假设希望在首次查询某一个城市天气的时候，将会真正访问到天气 API 网的服务，而此后的同一城市天气查询将走 CDN 缓存。

### 3.2 天气 API 网接口简介

天气 API 网（tianqiapi.com）对外提供商业级的天气预报服务，据说每天有千万级的 QPS。这里也可以设想一下如果它们使用 GraphQL 来定义、暴露 API 接口将会带来多大的便利性，并且都没有必要写 API 接口文档了。

根据它的官方 API 文档，我们可以通过下面的 API 获得当前某一个城市的天气（这里以笔者所在城市南京为例）：

**HTTP 请求**

```
Request URL: https://www.tianqiapi.com/free/day?appid={APP_ID}&appsecret={APP_SECRET}&city=%E5%8D%97%E4%BA%AC
Request Method: GET
Status Code: 200 OK
Remote Address: 127.0.0.1:7890
Referrer Policy: strict-origin-when-cross-origin
```

> 其中 {APP_ID} 和 {APP_SECRET} 为你申请的 API 账号。

**HTTP 响应**

```
HTTP/1.1 200 OK
Server: nginx
Date: Thu, 19 Aug 2021 06:21:45 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
Vary: Accept-Encoding
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
Content-Encoding: gzip
{
  air: "94",
  city: "南京",
  cityid: "101190101",
  tem: "31",
  tem_day: "31",
  tem_night: "24",
  update_time: "14:12",
  wea: "多云",
  wea_img: "yun",
  win: "东南风",
  win_meter: "9km/h",
  win_speed: "2级"
}
```

> 这里的命名和大小写实在要吐槽一下。

这里给出一份最简单的 API 客户端实现：

```
export async function fetchWeatherOfCity(city: string) {
  // URL 类在 EdgeRoutine 中有对应的实现。
  const url = new URL('http://www.tianqiapi.com/free/day');
  // 这里我们直接采用官方示例中的免费账户。
  url.searchParams.set('appid', '23035354');
  url.searchParams.set('appsecret', '8YvlPNrz');
  url.searchParams.set('city', city);
  const response = await fetch(url.toString);
  return response;
}
```

### 3.3 定义我们的 GraphQL SDL

让我们用 GraphQL SDL 语言定接下来要实现接口的 Schema：

```
type Query {
    "查询当前 API 的版本信息。"
  versions: Versions!
    "查询指定城市的实时天气数据。"
  weatherOfCity(name: String!): Weather!
}
"""
城市信息
"""
type City {
  """
  城市的唯一标识
  """
  id: ID!
  """
  城市的名称
  """
  name: String!
}
"""
版本信息
"""
type Versions {
  """
  API 版本号。
  """
  api: String!
  """
  `graphql` NPM 版本号。
  """
  graphql: String!
}
"""
天气数据
"""
type Weather {
  "当前城市"
  city: City!
  "最后更新时间"
  updateTime: String!
  "天气状况代码"
  code: String!
  "本地化（中文）的天气状态"
  localized: String!
  "白天气温"
  tempOfDay: Float!
  "夜晚气温"
  tempOfNight: Float!
}
```

### 3.4 实现 GraphQL Resolvers

Resolvers 的实现思路很简单，详见注释：

```
import { version as graphqlVersion } from 'graphql';
import { apiVersion } from '../api-version';
import { fetchWeatherOfCity } from '../tianqi-api';
export function versions() {
  return {
    // EdgeRoutine 的部署不像 FaaS 那么及时。
    // 因此每次部署前，我都会手工的修改 `api-version.ts` 中的版本号，
    // 查询时看到 api 版本号变了，就说明 CDN 端已经部署成功了。
    api: apiVersion,
    graphql: graphqlVersion,
  };
}
export async function weatherOfCity(parent: any, args: { name: string }) {
  // 调用 API 并将返回的格式转换为 JSON。
  const raw = await fetchWeatherOfCity(args.name).then((res) => res.json());
  // 将原始的返回结果映射到我们定义的接口对象中。
  return {
    city: {
      id: raw.cityid,
      name: raw.city,
    },
    updateTime: raw.update_time,
    code: raw.wea_img,
    localized: raw.wea,
    tempOfDay: raw.tem_day,
    tempOfNight: raw.tem_night,
  };
}
```

### 3.5 创建并启动服务器

现在我们已经有了 GraphQL 的接口大纲和 Resolvers，接下来就可以像 Node.js 里那样创建和启动我们的 Server 了。

```
// 注意这里不再是 `import { ApolloServer } from 'apollo-server'` 了。
import { ApolloServer } from '@ali/apollo-server-edge-routine';
import { default as typeDefs } from '../graphql/schema.graphql';
import * as resolvers from '../resolvers';
// 创建我们的服务器
const server = new ApolloServer({
  // `typeDefs` 是一个 GraphQL 的 `DocumentNode` 对象。
  // `*.graphql` 文件被 `webpack-graphql-loader` 加载后就变成了 `DocumentNode` 对象。
  typeDefs,
  // 即 3.4 章节中的 Resolvers
  resolvers,
});
// 先启动服务器，然后监听，一行代码全部搞定！
server.start().then(() => server.listen());
```

是的，就是这么简单，创建一个 `server` 对象，然后将它启动并使其侦听指定的路径（在本例中没有传递 `path` 参数，使用的是默认的 `/graphql`）。

到目前为止，主要的 TypeScript 和 GraphQL 代码已经全部完成了！

### 3.6 工程化配置

为了让 TypeScript 明白我们在 EdgeRoutine 环境中写代码，我们需要在 `tsconfig.json` 中交代 `lib` 和 `types`：

```
{
  "compilerOptions": {
    "alwaysStrict": true,
    "esModuleInterop": true,
    "lib": ["esnext", "webworker"],
    "module": "esnext",
    "moduleResolution": "node",
    "outDir": "./dist",
    "preserveConstEnums": true,
    "removeComments": true,
    "sourceMap": true,
    "strict": true,
    "target": "esnext",
    "types": ["@ali/edge-routine-types"]
  },
  "include": ["src"],
  "exclude": ["node_modules"]
}
```

再次强调一遍，与 Serverless / FaaS 不同，我们的程序并不是跑在 Node.js 环境中，而是跑在类似 ServiceWorker 环境 中。从 Webpack 5 开始，在 `browser` 目标环境中不再会自动注入 Node.js 内置模块的 polyfills，因此在 Webpack 的配置中我们需要手工加上：

```
{
  ...
  resolve: {
      fallback: {
      assert: require.resolve('assert/'),
      buffer: require.resolve('buffer/'),
      crypto: require.resolve('crypto-browserify'),
      os: require.resolve('os-browserify/browser'),
      stream: require.resolve('stream-browserify'),
      zlib: require.resolve('browserify-zlib'),
      util: require.resolve('util/'),
    },
    ...
  }
  ...
}
```

当然，你还需要手工安装包括 `assert`、`buffer`、`crypto-browserify`、`os-browserify`、`stream-browserify`、`browserify-zlib` 及 `util` 等在内的 polyfills 包。

### 3.7 添加 CDN 缓存

最后，让我们把 CDN 缓存加上，由于 EdgeRoutine 在笔者截稿前还处于 beta 阶段，因此我们只能用 Experimental 的 API 来实现缓存，让我们重新实现一下 `fetchWeatherOfCity()` 方法。

```
export async function fetchWeatherOfCity(city: string) {
  const url = new URL('http://www.tianqiapi.com/free/day');
  url.searchParams.set('appid', '23035354');
  url.searchParams.set('appsecret', '8YvlPNrz');
  url.searchParams.set('city', city);
  const urlString = url.toString();
  if (isCacheSupported()) {
    const cachedResponse = await cache.get(urlString);
    if (cachedResponse) {
      return cachedResponse;
    }
  }
  const response = await fetch(urlString);
  if (isCacheSupported()) {
    cache.put(urlString, response);
  }
  return response;
}
```

在全局（`globalThis`）中提供的 cache 对象，本质上是一个通过 Swift 实现的缓存器，它的键必须是一个 HTTP Request 对象或一个 HTTP 协议（非 HTTPS）的 URL 字符串，而值必须是一个 HTTP Response 对象（可以来自 `fetch()` 方法）。虽然 EdgeRoutine 的 Serverless 程序每隔几分钟或者 1 小时就会重启，我们的全局变量会随之销毁，但是有了 `cahce` 对象的帮助，可以帮我们实现 CDN 级别的缓存。

在阿里云的 EdgeRoutine KV 数据库上线后，我们会更新这个示例，实现更强大的缓存。遗憾的是，截止至笔者发稿时该功能暂未上线，本人十分期待！

### 3.8 添加 Playground 调试器

为了更好的调试 GraphQL 我们还可以添加一个官方的 Playground 调试器，它是一个单页面应用，因此我们可以通过 Webpack 的 `html-loader` 加载进来。

```
addEventListener('fetch', (event) => {
  const response = handleRequest(event.request);
  if (response) {
    event.respondWith(response);
  }
});
function handleRequest(request: Request): Promise<Response> | void {
  const url = new URL(request.url);
  const path = url.pathname;
  // 为了方便调试，我们把所有对 `/graphql` 的 GET 请求都处理为返回 playground。
  // 而 POST 请求则为实际的 GraphQL 调用
  if (request.method === 'GET' && path === '/graphql') {
    return Promise.resolve(new Response(rawPlaygroundHTML, { status: 200, headers: { 'content-type': 'text/html' } }));
  }
}
```

最后让我们在浏览器中访问 `/graphql`，看到的就是下面的界面：

在其中输入一段查询语句：

```
query CityWeater($name: String!) {
  versions {
    api
    graphql
  }
  weatherOfCity(name: $name) {
    city {
      id
      name
    }
    code
    updateTime
    localized
    tempOfDay
    tempOfNight
  }
}
```

将 `Variables` 设置为 `{ "name": "杭州" }`，点击中间的 Play 按钮即可。

### 3.9 完整的项目代码

在 EdgeRoutine 正式发布后，我会将上述 NPM 包和 Demo 在 我的 Github 上开源。

**4. 面向未来**

在这个简单的公开示例中，我们没有办法完整的演示如何将 EdgeRoutine 作为 GraphQL 网关的二级代理网关，你可以访问 graphcdn.io 通过视频了解更多关于 GrpahQL CDN 网关的信息。在可预见的将来，我们将利用 CDN 的边缘 KV 数据库实现对 Query 结果的缓存，并通过对 GraphQL 的语法解析和单类型中 ID 唯一的特性实现当发生 Mutations 时，自动使相关数据实体的缓存失效。

作者所在团队 CCO 技术部坐落在**杭州西溪 B 园区**和**南京建邺奥体新城科技园区**，前端技术栈是 TypeScript + GraphQL + React + AntDesign，致力于努力改善淘系及非淘系的消费者、商家及服务小二的用户体验，并将中台化的服务 OS 产品推向千万商家市场！

欢迎大家在与我交流讨论（https://www.zhihu.com/people/henry-li-03）。