> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ka15iwaXZf4NnrEYqScjXA)

> ​ Server-Sent Events 服务器推送事件，简称 SSE，是一种服务端实时主动向浏览器推送消息的技术。

**背景**

小盟 AI 助手项目中需要服务端把 AI 模型回调回来内容，实时推送到客户端，展示给用户；整个流程需要一个能够快速支持上线的服务端推送方案。经过对现有的一些服务端推送方案进行调研，并结合项目的周期、实现成本、用户体验等多方综合考量，最终选择了 Server-Sent Events（SSE）方案进行实践。

首先服务端推送，是一种允许应用服务器主动将信息发送到客户端的能力，为客户端提供了实时的信息更新和通知，增强了用户体验。

服务端推送主要基于以下几个诉求：

（1）实时通知：在很多情况下，用户期望实时接收到应用的通知，如新消息提醒、活动提醒等。

（2）节省资源：如果没有服务端推送，客户端需要通过轮询的方式来获取新信息，会造成客户端、服务端的资源损耗。通过服务端推送，客户端只需要在收到通知时做出响应，大大减少了资源的消耗。

（3）增强用户体验：通过服务端推送，应用可以针对特定用户或用户群发送有针对性的内容，如优惠活动、个性化推荐等。这有助于提高用户对应用的满意度和黏性。

**方案对比**

**轮询：**是一种较为传统的方式，客户端定时地向服务端发送请求，询问是否有新数据。服务端只需要检查数据状态，然后将结果返回给客户端。轮询的优点是实现简单，兼容性好；缺点是可能产生较大的延迟，且对服务端资源消耗较高。

**长轮询（Long Polling）：**轮询的改进版。客户端向服务器发送请求，服务器收到请求后，如果有新的数据，立即返回给客户端；如果没有新数据，服务器会等待一定时间（比如 30 秒超时时间），在这段时间内，如果有新数据，就返回给客户端，否则返回空数据。客户端处理完服务器返回的响应后，再次发起新的请求，如此反复。长轮询相较于传统的轮询方式减少了请求次数，但仍然存在一定的延迟。   

**WebSocket：**一种双向通信协议，同时支持服务端和客户端之间的实时交互。WebSocket 是基于 TCP 的长连接，和 HTTP 协议相比，它能实现轻量级的、低延迟的数据传输，非常适合实时通信场景，主要用于交互性强的双向通信。

**SSE：**是一种基于 HTTP 协议的推送技术。服务端可以使用 SSE 来向客户端推送数据，但客户端不能通过 SSE 向服务端发送数据。相较于 WebSocket，SSE 更简单、更轻量级，但只能实现单向通信。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/RYp9bfCzzialOCfCJqZuEFDxsncepY7qNYZtnoFIicYqTpUIVT70rIh1BiaFnz4J5l622MgZNfmA1oYibNNQUMMbng/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/RYp9bfCzzialOCfCJqZuEFDxsncepY7qNDEtTJHEEWuSaGicIeAOT7lTCEuQaqR3SRg9DQYGYUfSz5chFHDlnEZw/640?wx_fmt=png)

小盟 AI 助手项目需要快速上线且保证要用户较好的使用体验。鉴于 SSE 技术的轻量、实现简单、不增加额外的资源成本；当前业务场景也只需要服务端到用户端的单向的字符推送。非常适合项目需要。所以决定使用 SSE 来实现内容推送。 

**深入 SSE**

SSE 服务端推送，它基于 HTTP 协议，易于实现和部署，特别适合那些需要服务器主动推送信息、客户端只需接收数据的场景。有以下特点：

1、简单：基于 HTTP，无须额外的协议或者类库支持。主流浏览器都支持。

2、事件流：使用 "事件流"（Event Stream）将数据从服务器发送到客户端。每个事件都可以包含一个事件标识符、事件类型和数据字段。客户端可以根据这些信息来解析和处理接收到的数据。

3、自动重连：意外断开时会自动尝试重新连接。可以确保了在网络故障或连接中断后能够及时恢复通信，为用户提供连续的数据流。重连时会在 HTTP 头中的 Last_Event_ID 带上上一次的数据 ID，便于服务端返回后续数据。

4、单向推送：只能从服务端推送数据到客户端。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/RYp9bfCzzialOCfCJqZuEFDxsncepY7qNtHtTQDMXEBIiaUqLGEjKN6Sic2Etvu1vTuCWDZON3EkiaMIicWDy37vLqA/640?wx_fmt=png)

**SSE 消息体介绍：** 

![](https://mmbiz.qpic.cn/sz_mmbiz_png/RYp9bfCzzialOCfCJqZuEFDxsncepY7qNcxJzvBL8TV5kibFZuxoNZbxYKNcoRHtS6CmZRCh5aLic84CcdF1IgsWw/640?wx_fmt=png)

**SSE 消息体示例：**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/RYp9bfCzzialOCfCJqZuEFDxsncepY7qN6CicPbQibwBGBLlUwhJFu3icpBBRUwftvgic4ic7ZgaRmhP3LxaRJIcIdhQ/640?wx_fmt=png)

服务端主要使用 Spring，其对 SSE 主要提供了两种支持：

*   Spring WebMVC：传统的基于 Servlet 的同步阻塞编程模型，即 同步模型 Web 框架。
    
*   Spring WebFlux：异步非阻塞的响应式编程模型，即 异步模型 Web 框架。          
    

项目基于 springboot，所以选择使用前者实现。SseEmitter emitter = new SseEmitter(); 一句代码就可以建立一个 SSE 连接。

**实践**

#### **后端实现** 

建立一个 SseEmitterManger，统一管理当前服务 SSE 连接的创建、释放以及数据推送。结合 Redis 缓存可实现集群环境 SSE 连接的管理。

**核心逻辑如下：** 

*   连接池维护，设定一个上限，避免过大，导致内存问题。
    

```
static final Map<String, SseEmitter> sseCache = 
    new ConcurrentHashMap<>(300)          

```

*   建立 SSE 连接，为每个连接建立一个唯一的 MsgId，用来维护 SSE 连接与客户端的关系了；在 Redis 缓存中存入 MsgId 和当前机器节点的 IP 和 Port，这样可以找到 SSE 连接所在的服务结点，然后通过 HTTP 请求转发需要发送的数据到对应的服务节点上进行处理。
    

```
sse = new SseEmitter()
sseId = "sse_xxx";
redisKey= "aisse:" + bosId + "_" + wid 
ipPort = "10.10.10.10:8080"
redis.hset(redisKey, msgId, ipPort)
sseCache.put(msgId, sseEmitter);

```

*   获取持有连接的 pod ipPort；根据 IP 发起请求。
    

```
ipPort = redisUtil.hashGet(redisKey, msgId)

```

*   获取当前服务结点的 SSE 连接，发送数据。
    

```
sseEmitter = sseCache.get(msgId)sseEmitter.send(msgJson)          

```

*   释放 SSE 连接
    

```
SseEmitter sseEmitter = sseCache.get(msgId);
sseEmitter.complete();
sseCache.remove(msgId);
redisUtil.hashDel(redisKey, msgId);

```

**核心流程图如下：**  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/RYp9bfCzzialOCfCJqZuEFDxsncepY7qN3mIGl3IajRkq2Hh5ZmGtGvoVLhDf7O9FLLI5Zbcs11ktXibp0iafgmdw/640?wx_fmt=png)

需要注意的是开启 SSE 连接接口的整个链路都要支持长连接。例如使用 Nginx 则要开启长连接的配置：

*   keepalive 用于控制可连接整个 upstream servers 的 HTTP 长连接的个数，即控制总数。
    
*   proxy_http_verion 用于控制代理后端链接时使用的 HTTP 版本，默认为 1.0。要想使用长连接，必须配置为 1.1。
    
*   proxy_set_header 需要设置为 Connection ""，否则则发往 upstream servers 的请求中，Connection header 的值将为 close，导致无法建立长连接。   
    

```
http {
        upstream keepAliveService {
            server 10.10.131.149:8080;
            keepalive 20;
        }    
        server {
            listen 80;
            server_name keepAliveService;
            location /keep-alive/hello {
                proxy_http_version 1.1;
                proxy_set_header Connection "";
                proxy_pass http://keepAliveService;
            }
        }
}

```

**前端实现**  

前端可以使用组件 @microsoft/fetch-event-source 来实现。

```
npm i @microsoft/fetch-event-source
import { fetchEventSource } from '@microsoft/fetch-event-source';
let controller = new AbortController();  
let eventSource = fetchEventSource('apiUrl', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'token': '....'
  },
  signal: controller.signal,
  body: JSON.stringify({
    ... // 传参
  }),
  onopen() {
    // 建立连接
  },
  onmessage(event) {
    // 接收信息
    // 成功之后满足某些条件可以使用AbortController关闭连接
        controller.abort()
        eventSource?.close && eventSource.close();
  },
  onerror() {
    // 服务异常
        controller.abort()
        eventSource?.close && eventSource.close();
  },
  onclose() {
    // 服务关闭
  },
})

```

**总结**

SSE 轻量级的服务端单向推送技术；具有支持跨域、使用简单、支持自动重连等特点。相对于 WebSocket 更加轻量级，如果需求场景客户端和服务端单向通信，那么 SSE 是一个不错的选择。
==============================================================================================