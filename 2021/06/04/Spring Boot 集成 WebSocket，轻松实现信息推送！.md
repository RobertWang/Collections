> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA4NzQ0Njc4Ng==&mid=2247499143&idx=2&sn=d6657cb578f0384bb13c7c51e35530e8&chksm=903bf9eaa74c70fc0b228cd693e1991371b464c48e3b8881bbc567e2fc514628caac497ff7b9&mpshare=1&scene=1&srcid=0507kUu8CukoHLxhaoclJrsY&sharer_sharetime=1620326311557&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

在一次项目开发中，使用到了 Netty 网络应用框架，以及 MQTT 进行消息数据的收发，这其中需要后台来将获取到的消息主动推送给前端，于是就使用到了 MQTT，特此记录一下。

一、什么是 websocket？
----------------

WebSocket 协议是基于 TCP 的一种新的网络协议。

它实现了客户端与服务器之间的全双工通信，学过计算机网络都知道，既然是全双工，就说明了**服务器可以主动发送信息给客户端**。

这与我们的推送技术或者是多人在线聊天的功能不谋而合。![](https://mmbiz.qpic.cn/mmbiz_png/TNUwKhV0JpTWRmhiaKnEKYia9VIsp3DjhKUe97picgd1OeY4WHXpVdn4MPQgRZBFPEHtKjQN9Dy6Q8jTibxMCNykYg/640?wx_fmt=png)

为什么不使用 HTTP 协议呢？

这是因为 HTTP 是单工通信，通信只能由客户端发起，客户端请求一下，服务器处理一下，这就太麻烦了。

于是 websocket 应运而生。

![](https://mmbiz.qpic.cn/mmbiz_png/TNUwKhV0JpTWRmhiaKnEKYia9VIsp3DjhK8kYTCicFS8c1DkhTIGQ3mts42UO5EflQERjet8pNeUiccu5iaSC2m9aicg/640?wx_fmt=png)

下面我们就直接开始使用 Spring Boot 开始整合。以下案例都在我自己的电脑上测试成功，你可以根据自己的功能进行修改即可。Spring Boot 学习笔记，分享给你了。

我的项目结构如下：

![](https://mmbiz.qpic.cn/mmbiz_png/TNUwKhV0JpTWRmhiaKnEKYia9VIsp3DjhK6YMRDjyficUPqyhhJ7sXB56ly9qjGmsFtrFbx2VvmxlawVazFtJZMEw/640?wx_fmt=png)

二、使用步骤
------

#### 1. 添加依赖

Maven 依赖：

```
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-websocket</artifactId>  
</dependency> 


```

#### 2. 启用 Springboot 对 WebSocket 的支持

启用 WebSocket 的支持也是很简单，几句代码搞定。Spring Boot 最新教程推荐看这个：https://github.com/javastacks/spring-boot-best-practice

```
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.server.standard.ServerEndpointExporter;
/**
 * @ Auther: 马超伟
 * @ Date: 2020/06/16/14:35
 * @ Description: 开启WebSocket支持
 */
@Configuration
public class WebSocketConfig {
    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }
}


```

#### 3. 核心配置：WebSocketServer

因为 Web Socket 是类似客户端服务端的形式 (采用 ws 协议)，那么这里的 WebSocketServer 其实就相当于一个 ws 协议的 Controller。

下面是具体业务代码：

```
package cc.mrbird.febs.external.webScoket;

import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.baomidou.mybatisplus.core.conditions.update.UpdateWrapper;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Service;

import javax.websocket.*;
import javax.websocket.server.PathParam;
import javax.websocket.server.ServerEndpoint;
import java.io.IOException;
import java.time.LocalDateTime;
import java.util.List;
import java.util.concurrent.CopyOnWriteArraySet;

/**
 * Created with IntelliJ IDEA.
 * @ Auther: 马超伟
 * @ Date: 2020/06/16/14:35
 * @ Description:
 * @ ServerEndpoint 注解是一个类层次的注解，它的功能主要是将目前的类定义成一个websocket服务器端,
 * 注解的值将被用于监听用户连接的终端访问URL地址,客户端可以通过这个URL来连接到WebSocket服务器端
 */
@Component
@Slf4j
@Service
@ServerEndpoint("/api/websocket/{sid}")
public class WebSocketServer {
    //当前在线连接数
    private static int onlineCount = 0;
    //存放每个客户端对应的MyWebSocket对象
    private static CopyOnWriteArraySet<WebSocketServer> webSocketSet = new CopyOnWriteArraySet<WebSocketServer>();

    private Session session;

    //接收sid
    private String sid = "";

    /**
     * 连接建立成功调用的方法
     */
    @OnOpen
    public void onOpen(Session session, @PathParam("sid") String sid) {
        this.session = session;
        webSocketSet.add(this);     //加入set中
        this.sid = sid;
        addOnlineCount();           //在线数加1
        try {
            sendMessage("conn_success");
            log.info("有新窗口开始监听:" + sid + ",当前在线人数为:" + getOnlineCount());
        } catch (IOException e) {
            log.error("websocket IO Exception");
        }
    }

    /**
     * 连接关闭调用的方法
     */
    @OnClose
    public void onClose() {
        webSocketSet.remove(this);  //从set中删除
        subOnlineCount();           //在线数减1
       
        log.info("释放的sid为："+sid);
        
        log.info("有一连接关闭！当前在线人数为" + getOnlineCount());

    }

    /**
     * 收到客户端消息后调用的方法
     * @ Param message 客户端发送过来的消息
     */
    @OnMessage
    public void onMessage(String message, Session session) {
        log.info("收到来自窗口" + sid + "的信息:" + message);
        //群发消息
        for (WebSocketServer item : webSocketSet) {
            try {
                item.sendMessage(message);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * @ Param session
     * @ Param error
     */
    @OnError
    public void onError(Session session, Throwable error) {
        log.error("发生错误");
        error.printStackTrace();
    }

    /**
     * 实现服务器主动推送
     */
    public void sendMessage(String message) throws IOException {
        this.session.getBasicRemote().sendText(message);
    }

    /**
     * 群发自定义消息
     */
    public static void sendInfo(String message, @PathParam("sid") String sid) throws IOException {
        log.info("推送消息到窗口" + sid + "，推送内容:" + message);

        for (WebSocketServer item : webSocketSet) {
            try {
                //为null则全部推送
                if (sid == null) {
//                    item.sendMessage(message);
                } else if (item.sid.equals(sid)) {
                    item.sendMessage(message);
                }
            } catch (IOException e) {
                continue;
            }
        }
    }

    public static synchronized int getOnlineCount() {
        return onlineCount;
    }

    public static synchronized void addOnlineCount() {
        WebSocketServer.onlineCount++;
    }

    public static synchronized void subOnlineCount() {
        WebSocketServer.onlineCount--;
    }

    public static CopyOnWriteArraySet<WebSocketServer> getWebSocketSet() {
        return webSocketSet;
    }
}


```

#### 4. 测试 Controller

```
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.servlet.ModelAndView;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

/**
 * Created with IntelliJ IDEA.
 *
 * @ Auther: 马超伟
 * @ Date: 2020/06/16/14:38
 * @ Description:
 */
@Controller("web_Scoket_system")
@RequestMapping("/api/socket")
public class SystemController {
    //页面请求
    @GetMapping("/index/{userId}")
    public ModelAndView socket(@PathVariable String userId) {
        ModelAndView mav = new ModelAndView("/socket1");
        mav.addObject("userId", userId);
        return mav;
    }

    //推送数据接口
    @ResponseBody
    @RequestMapping("/socket/push/{cid}")
    public Map pushToWeb(@PathVariable String cid, String message) {
        Map<String,Object> result = new HashMap<>();
        try {
            WebSocketServer.sendInfo(message, cid);
            result.put("code", cid);
            result.put("msg", message);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return result;
    }
}


```

#### 5. 测试页面 index.html

```
<!DOCTYPE html>
<html>

 <head>
  <meta charset="utf-8">
  <title>Java 后端 WebSocket 的 Tomcat 实现</title>
  <script type="text/javascript" src="js/jquery.min.js"></script>
 </head>

 <body>
  <div id="main" style="width: 1200px;height:800px;"></div>
  Welcome<br/><input id="text" type="text" />
  <button onclick="send()">发送消息</button>
  <hr/>
  <button onclick="closeWebSocket()">关闭WebSocket连接</button>
  <hr/>
  <div id="message"></div>
 </body>
 <script type="text/javascript">
  var websocket = null;
  //判断当前浏览器是否支持WebSocket
  if('WebSocket' in window) {
   websocket = new WebSocket("ws://192.168.100.196:8082/api/websocket/100");
  } else {
   alert('当前浏览器 Not support websocket')
  }

  //连接发生错误回调方法
  websocket.onerror = function() {
   setMessageInnerHTML("WebSocket连接发生错误");
  };

  //连接成功建立回调方法
  websocket.onopen = function() {
   setMessageInnerHTML("WebSocket连接成功");
  }
  var U01data, Uidata, Usdata
  //接收消息回调方法
  websocket.onmessage = function(event) {
   console.log(event);
   setMessageInnerHTML(event);
   setechart()
  }

  //连接关闭回调方法
  websocket.onclose = function() {
   setMessageInnerHTML("WebSocket连接关闭");
  }

  //监听窗口关闭事件
  window.onbeforeunload = function() {
   closeWebSocket();
  }

  //将消息显示在网页上
  function setMessageInnerHTML(innerHTML) {
   document.getElementById('message').innerHTML += innerHTML + '<br/>';
  }

  //关闭WebSocket连接
  function closeWebSocket() {
   websocket.close();
  }

  //发送消息
  function send() {
   var message = document.getElementById('text').value;
   websocket.send('{"msg":"' + message + '"}');
   setMessageInnerHTML(message + "
");
  }
 </script>

</html>


```

#### 6. 结果展示

后台：

如果有连接请求

![](https://mmbiz.qpic.cn/mmbiz_png/TNUwKhV0JpTWRmhiaKnEKYia9VIsp3DjhKLQ8O9afy8LJnl6FJ1GrZXre05mzvhVguRsvy7VnkiaiajHDzoA4bIJBg/640?wx_fmt=png)

前台显示：

![](https://mmbiz.qpic.cn/mmbiz_png/TNUwKhV0JpTWRmhiaKnEKYia9VIsp3DjhK87Yibyd19mM1Dx5C1slBjEpUBo0nRHrzg9icztMqwFSzuQ5RONJBsSmQ/640?wx_fmt=png)

总结
==

这中间我遇到一个问题，就是说 WebSocket 启动的时候优先于 spring 容器，从而导致在 WebSocketServer 中调用业务 Service 会报空指针异常。

所以需要在 WebSocketServer 中将所需要用到的 service 给静态初始化一下：

如图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/TNUwKhV0JpTWRmhiaKnEKYia9VIsp3DjhK1yu8CEXVfcGPSBVibRUCupN2dp3EqGgkKJ4IpB0ribjAibM59jENgUy6Q/640?wx_fmt=png)

还需要做如下配置：

![](https://mmbiz.qpic.cn/mmbiz_png/TNUwKhV0JpTWRmhiaKnEKYia9VIsp3DjhK1yu8CEXVfcGPSBVibRUCupN2dp3EqGgkKJ4IpB0ribjAibM59jENgUy6Q/640?wx_fmt=png)

_原文：blog.csdn.net/MacWx/article/details/111319558_

_版权声明：本文为 CSDN 博主「大树先生.」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。_

```
往期资源  需要请自取

Java项目分享 最新整理全集，找项目不累啦 03版


卧槽！字节跳动《算法中文手册》火了，完整版 PDF 开放下载！

字节跳动总结的设计模式 PDF 火了，完整版开放下载！

堪称神级的Spring Boot手册，从基础入门到实战进阶


卧槽！阿里大佬总结的《图解Java》火了，完整版PDF开放下载！

喜欢就"在看"呗^_^

```