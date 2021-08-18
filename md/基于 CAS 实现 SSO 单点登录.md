> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI4Njk5OTg1MA==&mid=2247489559&idx=1&sn=c467efe274a8969242b989c6e4124188&chksm=ebd50c52dca285446222222d34d9db0df59a859ba56ac7756c4b3a2bfb2cad33203c5cb0e8ba&mpshare=1&scene=1&srcid=0817Ct5WXRjyXNm8i8Bhb1OM&sharer_sharetime=1629168768488&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

1. 概述  

--------

### 1.1. 什么是 SSO?

单点登录（ Single Sign-On , 简称 SSO ）是目前比较流行的服务于企业业务整合的解决方案之一， SSO 使得在多个应用系统中，用户只需要 登录一次 就可以访问所有相互信任的应用系统。

### 1.2. 什么是 CAS?

随着 SSO 技术的流行，相关产品也比较多，其中 CAS 就是一套解决方案，CAS（Central Authentication Service）中文翻译为统一身份认证服务或中央身份服务，它由服务端和客户端组成，实现 SSO, 并且容易进行企业应用的集成。

CAS 是 Yale 大学（耶鲁）发起的一个开源项目，旨在为 web 应用系统提供一种可靠的单点登录方法，CAS 在 2004 年 12 月正式成为 JA-SIG 的一个项目。

> 官网：https://www.apereo.org/projects/cas

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbueHDRjBHHrG3diaXz9Qhd3D5xdVw5LKkoo2wOWE2znkngYR4b8nmbTpFfl7KSWv92joJW7gOTD1WRA/640?wx_fmt=png)

CAS 具有以下的特点：

*   开源的企业级单点登录解决方案
    
*   CAS Server 为需要独立部署的 web 应用
    
*   CAS Client 支持非常多的客户端（这里指单点登录系统中的各个 web 应用），包括 Java、.Net 、ISAPI、Php、Perl、uPortal、Acegi、Ruby、VBScript 等客户端
    

有了 CAS, 我们的系统架构就演变成下面这样的：

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbueHDRjBHHrG3diaXz9Qhd3D5xwj7nFH7a8mSCAeDHSIhd9A97QsLWkQd3D9P0D8n2jLVEyQjR2iatLA/640?wx_fmt=png)

从架构上可以看出，CAS 包含两个部分：CAS Server 和 CAS Client.

*   CAS Server 需要独立部署，主要负责对用户的认证工作，CAS Client 负责处理
    
*   对客户端受保护资源的访问请求，需要登录，重定向到 CAS Server。
    

> 下面，我们一步步搭建 CAS 实现 SSO.

### 1.3. 开发环境要求

Jdk1.8+ maven3.6 idea tomcat9.0+ windows10

2. CAS Server 服务器端
------------------

#### 2.1. CAS 服务器端软件包下载

*   下载版本为 5.3
    

> 下载服务器的 overlay 地址: https://github.com/apereo/cas-overlay-template/tree/5.3

压缩包：`cas-overlay-template-5.3.zip`

解压好后用命令：`build.cmd package`

然后用编译的目录查看 war 包:

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbueHDRjBHHrG3diaXz9Qhd3D5rOh7oZMJZM3PNNPTuPjl8bcb0yhcicIRq7IFq7EBU89ezTmAAWPoyicw/640?wx_fmt=png)

### 2.2. 服务器端的基本部署和测试

将 war 包放到 tomcat 的 webapp 中，然后启动 tomcat

访问地址：`http://localhost:8080/cas` 或者 `http://localhost:8080/cas/login`

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbueHDRjBHHrG3diaXz9Qhd3D5Htibkg3KhHJerw5Q977pxKic7nWiad7mhZgK3JJVflQWzSJd7ylcGUmIA/640?wx_fmt=png)

默认用户名和密码在`\webapps\cas\WEB-INF\classes\application.properties`里面 用户名：casuser 密码：Mellon

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbueHDRjBHHrG3diaXz9Qhd3D5CyYzHWRgV3icZAs8wIRzUC8faLdebia9deTGd6lpxEJeNhXlhJgFtnEA/640?wx_fmt=png)

CAS 服务端启动成功

### 2.3. CAS Server 服务器配置

#### 2.3.1 去除 https 认证

CAS 默认使用的是 HTTPS 协议，如果使用 HTTPS 协议需要 SSL 安全证书（需向特定的机构申请和购买）。如果对安全要求不高或是在开发测试阶段，可使用 HTTP 协议。我们这里讲解通过修改配置，让 CAS 使用 HTTP 协议。

修改 CAS 服务端配置文件：

`\cas\WEB-INF\classes\application.properties`里添加如下内容:

```
cas.tgc.secure=false
cas.serviceRegistry.initFromJson=true
```

`\cas\WEB-INF\classes\services`目录下的 HTTPSandIMAPS-10000001.json 修改内容如下:

```
"serviceId" : "^(https|http|imaps)://.*"
```

3. CAS Client 客户端配置（自己项目）
-------------------------

Pom 文件的依赖即 pom.xml

```
<dependency>
<groupId>net.unicon.cas</groupId>
<artifactId>cas-client-autoconfig-support</artifactId>
<version>2.1.0-GA</version>
</dependency>
```

application.yml 配置文件

客户端 1

```
server:
  port: 9010
cas:
  server-url-prefix: http://localhost:8080/cas
  server-login-url: http://localhost:8080/cas/login
  client-host-url: http://localhost:9010
  validation-type: cas3
```

> 注：启动类追加开启 CAS 的注解 @EnableCasClient

项目中新建一个测试类

```
iimport io.swagger.annotations.Api;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@Api(description = "SSO-CAS的测试")
public class TestController {

    @GetMapping("/test1")
    public String test1(){
        return "test1....";
    }
}
```

客户端 2

```
server:
  port: 9011
cas:
  server-url-prefix: http://localhost:8080/cas
  server-login-url: http://localhost:8080/cas/login
  client-host-url: http://localhost:9011
  validation-type: cas3
```

> 注：启动类追加开启 CAS 的注解 @EnableCasClient

项目中新建一个测试类

```
import io.swagger.annotations.Api;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@Api(description = "SSO-CAS的测试")
public class TestController {

    @GetMapping("/test2")
    public String test1(){
        return "test2....";
    }
}
```

客户端 1，客户端 2 和 cas 服务器搭建好之后，接下来我们进行测试：

1. 首先启动 tomcat 服务器中的 CAS Server。

2. 分别启动客户端 1 和客户端 2，然后在浏览器地址栏输入客户端 1 的地址`http://localhost:9010/test1`

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbueHDRjBHHrG3diaXz9Qhd3D5gPbweb402qoIa1NM85wPRRv3CURRDicnC6vfVTdcIZickWotnIBVoAxA/640?wx_fmt=png)

在不登录的状态，在浏览器的地址栏继续输入客户端 2 的地址：`http://localhost:9011/test2`

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbueHDRjBHHrG3diaXz9Qhd3D5KqeGeGyeZQrLA6QkjrZCKBmbxicMXjgPHT1JImXJRyYgk3pTFF84EAw/640?wx_fmt=png)

当我们在其中一个登录界面登录账号后（假设登录客户端 2）就会跳转到登陆后的界面，如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbueHDRjBHHrG3diaXz9Qhd3D5PJO6D5ibkd9YQVBicZSQiawonkw6cibicXmo80twbYmKFoj5XdwVMNFY4fw/640?wx_fmt=png)

我们再次在浏览器窗口重新输入客户端 1，`http://localhost:9010/test1`，或者在刚刚输入客户端页面重新刷新, 不用登录即可进入页面，如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbueHDRjBHHrG3diaXz9Qhd3D5ymSlu5RVcIiaL7cgq8tBcP1Yo4z8fvuaEXVSvLqKibZSSRTN5a7h4Ukw/640?wx_fmt=png)

以上就是单点登录的测试。