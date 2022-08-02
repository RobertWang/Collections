> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzkzMDE5MDgwMQ==&mid=2247488707&idx=1&sn=ff14cf5fd541d33a012bbd9ef1a4ff54&chksm=c27f507df508d96b381bbec6314378e2f67d6d184f8cd0d748f9ae92a3a30b1a323a578aaee2&scene=21#wechat_redirect)

```
目录

1. 背景
2. 目标
3. 开源方案
4. 技术设计与实现
5. 稳定性保障
6. 总结


```

> Shopee Games 团队研发了多款电商平台休闲小游戏，比如糖果消消乐游戏 [Shopee Candy](https://mp.weixin.qq.com/s?__biz=MzkzMDE5MDgwMQ==&mid=2247486174&idx=1&sn=9930a6ea7effbe0f1f3faf8aa0f938f9&scene=21#wechat_redirect)，农场经营类游戏 Shopee Farm，支持大促营销的 [Shopee Shake](https://mp.weixin.qq.com/s?__biz=MzkzMDE5MDgwMQ==&mid=2247484004&idx=1&sn=069a5ed04ac1adb86dcb1bb3691a5178&scene=21#wechat_redirect) 等。为用户营造更加愉悦的购物环境，也给商家带来更多的流量曝光。
> 
> 伴随着业务发展，我们的后端服务数量也在不断增加，逐渐演变成了微服务的架构体系。而 Shopee Games 大多项目都需要接受来自用户向的流量，因此我们搭建了一个统一的 API 网关，隔离内外部调用，统一进行系统鉴权、业务监控等。
> 
> 本文将介绍 Shopee Games 团队自研的 API 网关，包括 API 网关如何进行泛化调用、自定义切面功能、稳定性保障、工程化实践等内容。

1. 背景
=====

正如康威定律所言，软件架构反映了组织架构。Shopee Games 团队采用了工作室 + 中台的组织结构，一个工作室负责若干个游戏，服务逻辑高度内聚；而游戏中台（Game Platform）承担一些公共抽象的需求，比如游戏积分服务、生命服务、兑换商店服务等，形成一个简洁高效的系统架构。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn07BGEjvNCEStYPwDa7bIQviay6M6PiaaaFTm3r7yd3QgGcE27PDZVNISqG6vV1AVEuaXqjKmqdO5LjA/640?wx_fmt=png)

图 1 系统架构

然而在某些场景下，后端服务不仅需要处理来自外部用户的流量，也需要接受内部服务的调用。比如部分游戏间可能存在联动，需要调用对方服务的接口，一些游戏项目需要搭建自己的运营平台，暴露接口给内部的 Admin 平台调用。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn07BGEjvNCEStYPwDa7bIQviaicfu580QGVrLpFoJxAq5MxK3UkicAlHcuZPYemsSmibkm9icAdRA8EARzQ/640?wx_fmt=png)

图 2 Web Server 代理转发

*   新增或修改接口时，后端开发需要手动编码 HTTP 协议到 RPC 协议的转换，而这些协议转换的代码是高度相似的，开发需要处理这些枯燥重复的工作；
    
*   每次业务发版时，不仅需要发布 RPC 服务，也需要发布 Web Server，多次发布可能会引入一些额外的风险；
    
*   为了故障隔离，大多项目选择搭建独立的 Web Server 作为前置代理。但某些项目流量并不高，且重要程度稍显弱一些，存在资源冗余的情况；
    
*   各个项目的 Web Server 需要维护相同的功能，比如鉴权、日志统计、监控上报等，不同团队需要处理重复的工作；
    
*   开发需要关注接入层的细节，包括 HTTP 状态码处理、接口格式、超时控制、限流熔断、告警配置等，增加了开发的负担和出问题的风险。
    

2. 目标
=====

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn07BGEjvNCEStYPwDa7bIQviaXbSbF1G1UVmEuhCsP2sE0icibLoiaiaLuSJaS8IqxCJIMMBkQHLicCRyB3A/640?wx_fmt=png)

图 3 API Gateway

**1）屏蔽外部流量的接入细节**

让开发更关注于业务本身，API 网关对业务透明且稳定，解放了每次业务变更需要手动适配协议转换代码的麻烦，也降低了接入层频繁发版的风险。

**2）统一接口规范**

使用配套的工具链来生成统一的接口文档，并约束接口数据格式、HTTP Status Code 语义、错误处理规范。让 Shopee Games 内部不同的团队都按照统一的规则调用接口、处理数据和错误异常，减少沟通成本，提高工作效率。

**3）提供统一的横切能力**

提供诸如鉴权、日志统计、客户端设备信息采集、监控上报等通用的功能，避免基础实现的变更升级，导致所有项目需要适配。减少不同团队之间的重复工作。

**4）提升资源利用率**

后端服务共享 API 网关的容量 Buffer 池，减少整体的资源占用。

3. 开源方案
=======

目前 Shopee Games 内部的 RPC 通信主要使用基于 Protobuf 的 gRPC 协议，而面向前端则大多是 HTTP JSON 的数据格式，API 网关的核心功能是做协议转换，我们调研了业界协议转换的几种实现方式。

3.1 gRPC-Gateway
----------------

gRPC-Gateway 是一款使用 Go 语言编写的软件，通过代码生成的方式，开发在 Protobuf 文件添加批注，编译 Protobuf 文件时，引入 gRPC-Gateway 插件，自动生成 HTTP server 的脚手架代码。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn07BGEjvNCEStYPwDa7bIQvia0iaX5LgatINVZCydjoAP69mV2SsZVhjCtZ1t9r7QvhbL6cpB8ns1oPg/640?wx_fmt=png)

图 4 gRPC gateway

定义 Protobuf 文件，HTTP 映射规则：

```
syntax = "proto3";
package helloworld;
import "google/api/annotations.proto";
option go_package = "gen/pb";
service Hello {
  rpc SayHello(HelloRequest) returns (HelloResponse) {
    option (google.api.http) = {
      post: "/v1/example/echo"
      body: "*"
    };
  };
}
message HelloRequest {
  string str = 1;
}
message HelloResponse {
  string str = 1;
}

```

运行 protoc 命令和 gRPC-Gateway 插件，自动生成 HTTP server 代码。

```
protoc ./proto/hello.proto -I=./proto -I={$GOPATH}/googleapis \    
    --go_out=${shell pwd}/gen/pb --go_opt paths=source_relative  \
    --go-grpc_out=${shell pwd}/gen/pb --go-grpc_opt paths=source_relative \
    --grpc-gateway_out=${shell pwd}/gen/pb --grpc-gateway_opt paths=source_relative


```

3.2 Envoy
---------

Envoy 是基于 C++ 开发的一款反向代理软件，官方提供了 gRPC-JSON transcoder 插件，用于 HTTP RESTful 接口到 gRPC 接口的转换。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn07BGEjvNCEStYPwDa7bIQvia2MxzhKNdDI3Gk8TbzJqsevKRicZqIxcvML6H8XicgflzQDvPsiaREzhtQ/640?wx_fmt=png)

图 5 Envoy with gRPC-JSON transcoder

如 3.1 的 proto 文件，我们使用以下命令来编译 proto 文件。

```
protoc ./proto/hello.proto -I=./proto -I={$GOPATH}/googleapis \
    --go_out=${shell pwd}/gen/pb --go_opt paths=source_relative  \
    --go-grpc_out=${shell pwd}/gen/pb --go-grpc_opt paths=source_relative \
    --include_imports --include_source_info \
    --descriptor_set_out=${shell pwd}/gen/pb/proto.pb


```

`include_imports` 和 `descriptor_set_out` 是 protoc 提供的 option 命令，用来输出完整的协议描述文件，比如上述命令生成了 hello.pb 文件。

3.3 对比
------

<table data-darkmode-color-165943713492210="rgb(163, 163, 163)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)"><thead data-darkmode-color-165943713492210="rgb(163, 163, 163)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)"><tr data-darkmode-color-165943713492210="rgb(163, 163, 163)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)" data-darkmode-bgcolor-165943713492210="rgb(25, 25, 25)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-darkmode-color-165943713492210="rgb(153, 153, 153)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-165943713492210="rgb(31, 31, 31)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(255,255,255)|rgb(249, 249, 249)" data-style="padding: 5px 10px; font-weight: bold; font-size: 14px; border-width: 1px 0.2px; border-right-style: initial; border-left-style: initial; border-right-color: initial; border-left-color: initial; color: rgb(77, 77, 77); min-width: 80px; background-color: rgb(249, 249, 249); border-bottom-style: solid; border-bottom-color: rgb(255, 255, 255); border-top-style: solid; border-top-color: rgb(255, 255, 255); border-top-left-radius: 0px; border-bottom-left-radius: 0px; text-align: center;"><br data-darkmode-color-165943713492210="rgb(153, 153, 153)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-165943713492210="rgb(31, 31, 31)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(255,255,255)|rgb(249, 249, 249)"></th><th data-darkmode-color-165943713492210="rgb(153, 153, 153)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-165943713492210="rgb(31, 31, 31)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(255,255,255)|rgb(249, 249, 249)" data-style="padding: 5px 10px; font-weight: bold; font-size: 14px; border-width: 1px 0.2px; border-right-style: initial; border-left-style: initial; border-right-color: initial; border-left-color: initial; color: rgb(77, 77, 77); min-width: 140px; background-color: rgb(249, 249, 249); border-bottom-style: solid; border-bottom-color: rgb(255, 255, 255); border-top-style: solid; border-top-color: rgb(255, 255, 255); text-align: center;">gRPC-Gateway</th><th data-darkmode-color-165943713492210="rgb(153, 153, 153)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-165943713492210="rgb(31, 31, 31)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(255,255,255)|rgb(249, 249, 249)" data-style="padding: 5px 10px; font-weight: bold; font-size: 14px; border-width: 1px 0.2px; border-right-style: initial; border-left-style: initial; border-right-color: initial; border-left-color: initial; color: rgb(77, 77, 77); min-width: 140px; background-color: rgb(249, 249, 249); border-bottom-style: solid; border-bottom-color: rgb(255, 255, 255); border-top-style: solid; border-top-color: rgb(255, 255, 255); border-top-right-radius: 0px; border-bottom-right-radius: 0px; text-align: center;">Envoy</th></tr></thead><tbody data-darkmode-color-165943713492210="rgb(163, 163, 163)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)"><tr data-darkmode-color-165943713492210="rgb(163, 163, 163)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)" data-darkmode-bgcolor-165943713492210="rgb(25, 25, 25)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-165943713492210="rgb(153, 153, 153)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-165943713492210="rgb(25, 25, 25)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(255,255,255)|rgb(255, 255, 255)" data-style="padding: 5px 10px; font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;">核心原理</td><td data-darkmode-color-165943713492210="rgb(153, 153, 153)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-165943713492210="rgb(25, 25, 25)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(255,255,255)|rgb(255, 255, 255)" data-style="padding: 5px 10px; font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;">工具自动<br data-darkmode-color-165943713492210="rgb(153, 153, 153)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-165943713492210="rgb(25, 25, 25)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(255,255,255)|rgb(255, 255, 255)">生成协议代码</td><td data-darkmode-color-165943713492210="rgb(153, 153, 153)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-165943713492210="rgb(25, 25, 25)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(255,255,255)|rgb(255, 255, 255)" data-style="padding: 5px 10px; font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;">直接使用<br data-darkmode-color-165943713492210="rgb(153, 153, 153)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-165943713492210="rgb(25, 25, 25)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(255,255,255)|rgb(255, 255, 255)">协议描述文件</td></tr><tr data-darkmode-color-165943713492210="rgb(163, 163, 163)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)" data-darkmode-bgcolor-165943713492210="rgb(32, 32, 32)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-165943713492210="rgb(153, 153, 153)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-165943713492210="rgb(25, 25, 25)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)" data-style="padding: 5px 10px; font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;">实现语言</td><td data-darkmode-color-165943713492210="rgb(153, 153, 153)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-165943713492210="rgb(25, 25, 25)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)" data-style="padding: 5px 10px; font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;">Golang</td><td data-darkmode-color-165943713492210="rgb(153, 153, 153)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-165943713492210="rgb(25, 25, 25)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)" data-style="padding: 5px 10px; font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;">C++</td></tr><tr data-darkmode-color-165943713492210="rgb(163, 163, 163)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)" data-darkmode-bgcolor-165943713492210="rgb(25, 25, 25)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-165943713492210="rgb(153, 153, 153)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-165943713492210="rgb(25, 25, 25)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(255,255,255)|rgb(255, 255, 255)" data-style="padding: 5px 10px; font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;">是否支持<br data-darkmode-color-165943713492210="rgb(153, 153, 153)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-165943713492210="rgb(25, 25, 25)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(255,255,255)|rgb(255, 255, 255)">多个服务</td><td data-darkmode-color-165943713492210="rgb(153, 153, 153)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-165943713492210="rgb(25, 25, 25)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(255,255,255)|rgb(255, 255, 255)" data-style="padding: 5px 10px; font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;">不支持</td><td data-darkmode-color-165943713492210="rgb(153, 153, 153)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-165943713492210="rgb(25, 25, 25)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(255,255,255)|rgb(255, 255, 255)" data-style="padding: 5px 10px; font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;">不支持</td></tr><tr data-darkmode-color-165943713492210="rgb(163, 163, 163)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)" data-darkmode-bgcolor-165943713492210="rgb(32, 32, 32)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-165943713492210="rgb(153, 153, 153)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-165943713492210="rgb(25, 25, 25)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)" data-style="padding: 5px 10px; font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;">是否<br data-darkmode-color-165943713492210="rgb(153, 153, 153)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-165943713492210="rgb(25, 25, 25)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)">需要发版</td><td data-darkmode-color-165943713492210="rgb(153, 153, 153)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-165943713492210="rgb(25, 25, 25)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)" data-style="padding: 5px 10px; font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;">需要<br data-darkmode-color-165943713492210="rgb(153, 153, 153)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-165943713492210="rgb(25, 25, 25)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)">随 RPC server 发版</td><td data-darkmode-color-165943713492210="rgb(153, 153, 153)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-165943713492210="rgb(25, 25, 25)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)" data-style="padding: 5px 10px; font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;">不需要<br data-darkmode-color-165943713492210="rgb(153, 153, 153)" data-darkmode-original-color-165943713492210="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-165943713492210="rgb(25, 25, 25)" data-darkmode-original-bgcolor-165943713492210="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)">RPC server 发版后，Envoy reload 协议描述文件</td></tr></tbody></table>

4. 技术设计与实现
==========

4.1 整体架构
--------

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn07BGEjvNCEStYPwDa7bIQviad5L9pORSIfuk7kptpVLD8L43E1XLbHSB0Nrqsj0r3ibdVkrl6HhXgWA/640?wx_fmt=png)

图 6 整体架构

4.2 基于 Protobuf 的控制面
--------------------

业内大多公司都会有自己的 API 网关解决方案，通常来说会搭建一个控制面来管理 API 的生命周期，比如 API 创建、测试、审核、发布、回滚等。

开发新增或修改一个接口时，需要在控制面录入参数，生成 DSL，并配置好 HTTP Request 字段到 RPC Request 字段的映射规则、方法路由规则、校验插件等。这种集中管理 API 的方式有利于整体流程和质量的管控，但在开发阶段也会给开发人员带来额外的使用成本；代码定义和控制规则分开两个地方管理，也会形成较大的割裂感，不利于代码阅读、排查定位问题。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn07BGEjvNCEStYPwDa7bIQvia2iaJ4P3P4ibdbO0Y9dMkU8jfMKLaibz3w1Ay3c5lat3wIQjd66dibkRRQA/640?wx_fmt=png)

图 7 在控制面录入规则

经过内部讨论，我们希望由 Protobuf 来承载这些控制规则，Protobuf 即文档，即规则。将 API 管理流程收敛在业务团队内部，保持和原有一致的开发体验。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn07BGEjvNCEStYPwDa7bIQviavYfJRmjYTAmFylaPQ9lrbT1JIguQG6wNY2icR6w4PESLsBV3kK2Njdg/640?wx_fmt=png)

图 8 Protobuf 定义规则

而 Protobuf 来定义规则也有天然的缺陷，比如修改某些参数时，需要服务发版。我们接下来也会实现一个统一的控制面 Portal，业务方可以在 Portal 上修改参数，其优先级高于 Protobuf 规则，实现控制规则的动态覆盖。

4.3 基础原理
--------

### 4.3.1 Protobuf Descriptor

Protobuf 协议基于 proto 文件提供了一系列的反射机制，程序可以在运行过程中通过 proto 获取 service 定义的方法，也可以动态读取，设置 message 的属性。

Protobuf 为每种实体类型定义了 Descriptor 对象，包含了以下几种类型：

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn07BGEjvNCEStYPwDa7bIQviacH8TqBeicYAhaDquSPRFWiad3mXIV2g1rPofWyTIvTG5ekBQ094S35bw/640?wx_fmt=png)

也就是说，如果我们能获取到 FileDescriptor（下文简称 FD），就能获取到 proto 文件中定义的所有内容，而 3.1 步骤中生成的 hello.pb 文件，其实就是 FD 序列化后的内容，各个支持 Protobuf 的语言，都有提供类似 `FileDescriptorSet.ParseFrom` 的方法来反序列化文件内容，生成 FD 对象，比如 Golang 官方提供的解析库。

```
import 'google.golang.org/protobuf/types/descriptorpb'

var fds descriptorpb.FileDescriptorSet
_ = proto.Unmarshal(fileDescriptor, &fds)
svc := fds.GetFile()[0].GetService()[0]
method := svc.GetMethod()[0]
fmt.Println(method.GetInputType())


```

### 4.3.2 协议感知

开发人员在使用 protoc 命令编译 proto 文件时，需要添加 `include_imports` 和 `descriptor_set_out` 选项，来输出 FD 信息，而 API 网关则需要在运行过程中获取到业务方的 FD。

**方案一**：RPC 进程在启动时，主动上报，存储到一个中心化的存储系统中供 API 网关查询。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn07BGEjvNCEStYPwDa7bIQviaGdmukof3taHxcsh3jPUJ0qy6Cpic8VUHg73q9JKS7UP2YB53Z4Diamgw/640?wx_fmt=png)

图 9 中心化存储协议信息

**方案二**：API 网关主动向业务节点请求 FD 信息，要求 upstream 暴露获取 FD 的接口供 API 网关调用。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn07BGEjvNCEStYPwDa7bIQvianalB3icLNj008TFThnI041DudGcfaFK2834IHvgXLhQtdibakNA4r6LA/640?wx_fmt=png)

图 10 向节点拉取协议信息

方案一类似服务注册、发现的过程。业务方与 API 网关解耦，业务方不再需要提供获取协议的接口。但需要额外引入一个高可用的存储中间件。

方案二则是轮询目标节点，获取 FD 信息，要求目标节点实现暴露协议的接口，对业务代码有一定的侵入。

每增加一个外部依赖都会新增一些未知的风险，且方案一需要我们投入更多的人力去关注存储中间件的稳定性。经过权衡讨论，API 网关初期的版本，我们选择了方案二，定时拉取节点 FD 信息，同时也视为一种健康检查的手段。如果未来需要支持 Serverless 的架构，我们也会重新考虑使用方式一。

### 4.3.3 Protobuf Option

4.2 一节提到，我们希望在 proto 文件中同时定义接口信息和控制规则，而 Protobuf Option 很好地满足了我们的需求。

Protobuf 支持为 Service、Method 和 Field 自定义 option 选项，如下定义：

```
service bookService {
 option (xproto.svcTimeoutMs) = 3000;// service接口超时时间

 rpc GetBook(GetBookRequest) returns (GetBookResponse) {
   option (google.api.http) = {
     get: "/get_book"
   };
 }
}

message GetBookRequest {
 int32 id = 1 ;
 string from = 2 [(xproto.queryKey) = "from"];
}

```

我们在公共的 proto 文件中定义了一系列的 option 选项，业务方的 proto 文件引用 `common.pb` 后，就能为服务、方法或字段添加一些控制规则。

API 网关获取到业务方的 FD 后，就能从 Descriptor 对象中读取到定义的 option 信息，从而做一些转发策略。

### 4.3.4 数据转换

API 网关获取 FD 后，可以从中依次获取到 MethodDescriptor 和 MessageDescriptor。

我们可以很方便地从 Descriptor 中 New 一个 proto 对象，并动态进行赋值：

```
msg := dynamic.NewMessage(methodDescriptor.GetInputType())
msg.SetFieldByName("age", 1)
msg.SetFieldByName("name", "username")


```

API 网关提供了一份公共的定义文件，内置一些 Protobuf Field options，业务方开发人员可以定义每个字段的映射规则，比如 Header、Cookie、Query 参数等。

```
message GetBookRequest {
 int32 id = 1 ;
 string user_ip = 2 [(xproto.headerKey) = "client_ip"];
 string from = 3 [(xproto.queryKey) = "from"];
 string version = 4  [(xproto.cookieKey) = "version"];
}

```

而对于读取 Body 参数，则使用官方的 JSONPB 库，进行同名的字段映射。

默认情况下，会将 Response 按照 JSON 格式同名序列化，并读取 RPC 返回的错误码，将 JSON 按照以下格式渲染：

```
{
  "code":0,
  "msg":"",
  "data":{

  }
}


```

如果业务方希望渲染非 JSON 格式，也可以定义返回约定好的结构体。

```
message RawBody {
  string contentType = 1;
  bytes body = 2;
}

```

通过以上方式，API 网关约束了业务方 HTTP Response 的行为，我们和前端约定好了各个 Status Code 的含义和 Response 返回的格式，统一后端服务错误处理方式，规避了以往不同项目间接口对外输出格式的差异。

4.4 功能设计
--------

### 4.4.1 服务发现

我们在配置中心为每个接入的后端 RPC 服务添加 project_name 和 service_name 的映射对。当用户访问 `/{project_name}/*` 路径时，会检查是否存在对应的 service_name。

获取到 service_name 以后，则向 Name Server 或者内存缓存中查询目标服务的节点信息，用于后续的负载均衡策略。

### 4.4.2 路由匹配

RPC 服务通过定义 Method HTTP Option 的方式来暴露接口，比如：

```
  rpc SayHello(HelloRequest) returns (HelloResponse) {
    option (google.api.http) = {
      post: "/v1/example/echo"
    };
  };

  rpc GetBook(GetBookRequest) returns (GetBookResponse) {
    option (google.api.http) = {
      get: "/v1/example/book/:id"
    };
  };

```

当用户访问 `POST : /{project_name}/v1/example/echo` 路径时，API 网关需要将 HTTP 请求转换为相应服务的 SayHello RPC 调用，如果路径上定义了 Restfull 参数，API 网关也需要能够正确匹配到相应的方法，我们同时使用 Map 和 Trie 树两种数据结构来进行路由匹配。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn07BGEjvNCEStYPwDa7bIQviaQMtGLxrfMSKgPbPOYLibsqfaQylicntejjuQFX1pETibq4f7nWb19icRSA/640?wx_fmt=png)

图 11 Map 存储路由映射

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn07BGEjvNCEStYPwDa7bIQvia1NxYfgiav097aBb3MWshQPC7N4zskzJVYtOXtrplbmCQ2WItHhTjWfA/640?wx_fmt=png)

图 12 Trie 树存储路由映射

### 4.4.3 metadata 传递

业务除了简单的 RPC request/response 之外，往往可能需要额外传递一些额外的附加数据，如控制信息，一些上下文或者服务的通用数据等。为此，我们提供了 metadata 功能。

在上文中，我们提到了业务端在 request 请求体中定义一些 FieldOption 来读取 HTTP 的 header、cookie 等。但如果业务方希望所有请求都读取到某个 header key，这种方式略微有点繁琐，可以通过增加以下定义来传递服务的通用数据：

```
service exampleService {
  option (xproto.forwardHeaderKeys) = "User-Agent";
 
  rpc SayHello(HelloRequest) returns (HelloResponse) {
    option (google.api.http) = {
      post: "/v1/example/echo"
    };
  };
}

```

业务方可以在 RPC 拦截器或者方法入口通过以下方式获取到对应的值：

```
import "google.golang.org/grpc/metadata"

data, ok := metadata.FromIncomingContext(ctx)
if ok {
        fmt.Println(data.Get("User-Agent"))
}


```

同理，当业务方希望设置 HTTP Response Header 时，也可以通过写入 RPC metadata，API 网关读取到相应格式时，会将数据写入响应体中。

### 4.4.4 切面功能

开发人员只需要设置 Protobuf Option，就能直接使用到 API 网关的切面功能。

比如鉴权，API 网关支持接口和服务级别的鉴权，并将用户的上下文信息注入到 Reuqest Message 中，开发人员在 RPC 服务中可以直接获取到上下文传递的 User 对象，不再需要代码中处理鉴权逻辑。

再比如游戏黑名单玩家的过滤，只需在 Protobuf 中定义方法级别的 Proto Option：

```
rpc Begin (BeginRequest) returns (BeginResponse) {
    option (google.api.http) = {
      post: "/begin"
    };
    option (xproto.filterBlockUser) = true;
};

```

基于以上类似的方式，业务方还能实现在 Protobuf 定义请求超时时间、日志打印规则、请求 A/B Test 规则、接口限流规则等等。这种方式对开发人员来说，友好且直观，极大降低了使用成本，并可以持续迭代扩展，满足业务方多样的需求。

4.5 易用性设计
---------

优化开发体验，提高人效也是我们的目标之一。API 网关为了降低业务方接入和使用成本，在易用性方面也提供了一些开箱即用的工具。

### 4.5.1 API 文档托管

我们提供了 protoc 插件，在编译 Protobuf 时，会自动解析所有暴露 HTTP 接口的 RPC 方法，并读取相应的 Protobuf Options，生成 swagger 文档，上传到内部 API 管理平台。

### 4.5.2 Docker 镜像

本地开发联调时，也需要将 RPC 接口转换成 HTTP 接口来调用。我们将 API 网关打包成 Docker 镜像，并适配了一些本地开发环境所需要的特性。只需要在本地环境启动一个 Docker All In One 容器，就能为本机的 RPC 服务提供反向代理的功能。

同时我们也将一些开发环境需要的工具集成到 Docker 镜像中，方便后端同学使用，无需再因为环境不同而烦恼。

### 4.5.3 脚手架

如上文所述，接入 API 网关需要预先实现暴露协议的接口，略微有些繁琐。我们提供了脚手架来帮助后端同学快速创建新的项目，并内置一些胶水代码和开箱即用的 Makefile 命令，本地只需依赖 Docker，即可运行完整的 Proxy + RPC 项目。

5. 稳定性保障
========

API 网关作为业务流量入口，其稳定性是需要优先保证的。我们不仅在代码维度实现多种服务治理策略，也通过一些工作实践来保障其稳定性。

5.1 服务治理
--------

*   **服务限流**：当 API 网关自身负载较高时，快速拒绝一些低优先级的请求；
    
*   **超时控制**：每个请求调用，都会设置合理的超时时间，并通过 metadata 传递到 upstream；
    
*   **熔断降级**：当某个服务接口多次出现异常时，熔断该节点的请求，快速返回错误或降级内容；
    
*   **负载均衡**：业务方可指定使用随机、轮询、一致性哈希等负载均衡策略；
    
*   **健康检查**：对每个节点定时发起健康检查，当节点多次返回内容错误时，标记为非健康状态，从 IP 池中暂时摘除该节点。
    

5.2 立体监控
--------

为了能及时观察服务的运行情况，我们对 API 网关和 upstream 请求进行了全方位、立体的监控，并配置了相应的告警阈值。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn07BGEjvNCEStYPwDa7bIQvia2M7n0Q1MKBzPmgEruqBicFD4zFoo4eoy6LANtV69hWBv5cUNKrsyz3g/640?wx_fmt=png)

5.3 故障预案
--------

突发情况可能会在系统的任何一个环节出现，比如 Nginx 接入层不稳定、服务自身出现 Bug、依赖的中间件异常、服务负载过高、上游节点故障等等。

Shopee Games 团队一贯秉持着凡事预则立，不预则废的原则。多维度的告警机制让我们能够快速感知问题，监控和日志帮助我们定位问题，而事故预案则让我们面对不同的突发情况，能够迅速止血。

我们梳理 API 网关上下游所有的关键路径，对每个线上可能发生的问题都进行了模拟并制定预案，将一个线上事故处理流程拆分为告警表象、排查思路、执行动作、验证动作四个步骤。

同时我们会定期开展故障演练，以低成本的方式发现预案的不足，暴露系统的问题，不断提高人员及系统的能力。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn07BGEjvNCEStYPwDa7bIQviaqPuDiccSnuPj14RuN0fibibcZRN0NEQxl8YIMwBWc6BGjlx9zwsqhwicMQ/640?wx_fmt=png)

图 13 Games 大促演练

5.4 性能优化
--------

我们定时对线上容器进行 pprof，分析项目的性能瓶颈，并进行针对性的优化，最终让 API 网关的性能在原有的基础上提高了 20%。

### 5.4.1 避免 Web 框架中的反射

我们原先内部使用的 Web 框架为了减少用户的使用成本，实现了很多灵活的特性，比如支持多种 Handler 方法形式、在 Handler 调用时自动实现请求体的反序列化、结构体注入。

这些功能都是基于反射实现的。而 API 网关已经实现了泛化调用目标服务，并不需要这些特性，因此我们去掉了框架中的反射功能。

### 5.4.2 使用 Sync.Pool 优化内存分配

API 网关在构造请求体和返回结构体的过程中会创建大量的小对象，内存频繁分配回收，GC 压力上升。我们使用 Golang 提供的 Sync.Pool 来分配，复用内存。

### 5.4.3. string 和 []byte 类型转换

Golang 标准的 string 和 []byte 类型转换会导致内存的拷贝，这导致 API 网关日志打印时有额外的性能损耗。我们在特定的场景下使用 unsafe 进行类型强转，避免内存拷贝。

```
func b2s(b []byte) string {
  //byte数组和string的结构只差一个Cap，直接转换截断
  return *(*string)(unsafe.Pointer(&b))
}


```

6. 总结
=====

我们通过动态获取后端服务集的协议描述文件，来实现 RPC 的泛化调用，并读取 Protobuf 文件定义好的 option，来控制 API 网关的一些行为。Protobuf 已经适配了各种语言，因此无论后端服务集使用什么语言实现，只需要暴露获取协议的 RPC 接口，就能被 API 网关代理转发。

而在易用性方面，我们使用 Docker 镜像打包了开发环境的依赖，包括 protoc 插件依赖、swagger API 文档导出上传等，并提供 CLI 命令来快速搭建 demo，降低了用户的使用门槛。

在稳定性保障方面，我们同样不余遗力，吸取 Shopee Games 多年以来大促的经验和沉淀，建立不同维度指标的监控和告警，对线上可能出现的问题设立预案，开展演练。同时也关注线上的性能瓶颈，针对性的进行优化。

目前 Shopee Games 内部已经有多个游戏接入到 API 网关，并经过了几次大促的考验，其余游戏也会陆续接入。但 API 网关仍然有许多待改进的地方，比如当前并没有按业务的优先级进行网关资源的治理、暂不支持动态配置覆盖等。接下来我们也会继续迭代优化，在丰富 API 网关功能的同时，给业务方带来更稳定的支持。

> **本文作者**
> 
> Dongping，后端工程师，来自 Shopee Games 团队。
> 
> **技术编辑**
> 
> Zhiwang，Shopee 技术委员会 BE 通道委员。
> 
> Hong，来自 Shopee Games 团队。

> **团队介绍**
> 
> 我们不仅会做一些游戏化的营销工具，而且还做真正的游戏！经营、养成、关卡、AR、PVP 等，多达几十种并不断增加。面向东南亚用户群体，挖掘电商产品游戏化的奥秘，精细运营已有用户，解锁不一样的游戏设计和玩法。“好玩” 是我们做游戏的初衷和目标，因为我们相信，游戏能建立情感、拉近距离，给用户带来快乐，为平台创造价值。