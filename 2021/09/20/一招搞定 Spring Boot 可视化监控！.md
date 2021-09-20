> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7007971438082654245)*   `Spring Actuator`：在应用程序里提供众多 Web 接口，通过它们了解应用程序运行时的内部状况。有关更多信息，请参见 Spring Boot 2.0 中的 Spring Boot Actuator。
*   `Micrometer`：为 Java 平台上的性能数据收集提供了一个通用的 API，它提供了多种度量指标类型（Timers、Guauges、Counters 等），同时支持接入不同的监控系统，例如 Influxdb、Graphite、Prometheus 等。Spring Boot Actuator 对此提供了支持。
*   `Prometheus`：一个时间序列数据库，用于收集指标。
*   `Grafana`：用于显示指标的仪表板。

下面，我们将分别介绍每个组件。本文中使用的代码存档在 GitHub 上。

2、创建示例应用
--------

首先要做的是创建一个可以监控的应用程序。通过`Spring Initializr`，并添加 Spring Boot Actuator，Prometheus 和 Spring Web 依赖项， 我们创建了一个如下所示的 Spring MVC 应用程序。

```
@RestController
public class MetricsController {

@GetMapping("/endPoint1")
public String endPoint1() {
return "Metrics for endPoint1";
}

@GetMapping("/endPoint2")
public String endPoint2() {
return "Metrics for endPoint2";
    }
}
复制代码
```

启动应用程序：

```
$ mvn spring-boot:run
复制代码
```

验证接口是否正常：

```
$ curl http://localhost:8080/endPoint1Metrics for endPoint1$ curl http://localhost:8080/endPoint2Metrics for endPoint2
复制代码
```

验证 Spring Actuator 接口。为了使响应信息方便可读，我们通过 python -mjson.tool 来格式化信息。

```
$ curl http://localhost:8080/actuator | python -mjson.tool
...
{
"_links":{
"self":{
"href":"http://localhost:8080/actuator",
"templated":false
      },
"health":{
"href":"http://localhost:8080/actuator/health",
"templated":false
      },
"health-path":{
"href":"http://localhost:8080/actuator/health/{*path}",
"templated":true
      },
"info":{
"href":"http://localhost:8080/actuator/info",
"templated":false
      }
   }
}
复制代码
```

默认情况下，会显示以上信息。除此之外，Spring Actuator 可以提供更多信息，但是你需要启用它。为了启用 Prometheus，你需要将以下信息添加到 application.properties 文件中。

```
management.endpoints.web.exposure.include=health,info,prometheus
复制代码
```

重启应用程序，访问 [http://localhost:8080/actuator/prometheus 从 Prometheus 拉取数据，返回了大量可用的指标信息。我们这里只显示输出的一小部分，因为它是一个很长的列表。](https://link.juejin.cn?target=http%3A%2F%2Flocalhost%3A8080%2Factuator%2Fprometheus%25E4%25BB%258EPrometheus%25E6%258B%2589%25E5%258F%2596%25E6%2595%25B0%25E6%258D%25AE%25EF%25BC%258C%25E8%25BF%2594%25E5%259B%259E%25E4%25BA%2586%25E5%25A4%25A7%25E9%2587%258F%25E5%258F%25AF%25E7%2594%25A8%25E7%259A%2584%25E6%258C%2587%25E6%25A0%2587%25E4%25BF%25A1%25E6%2581%25AF%25E3%2580%2582%25E6%2588%2591%25E4%25BB%25AC%25E8%25BF%2599%25E9%2587%258C%25E5%258F%25AA%25E6%2598%25BE%25E7%25A4%25BA%25E8%25BE%2593%25E5%2587%25BA%25E7%259A%2584%25E4%25B8%2580%25E5%25B0%258F%25E9%2583%25A8%25E5%2588%2586%25EF%25BC%258C%25E5%259B%25A0%25E4%25B8%25BA%25E5%25AE%2583%25E6%2598%25AF%25E4%25B8%2580%25E4%25B8%25AA%25E5%25BE%2588%25E9%2595%25BF%25E7%259A%2584%25E5%2588%2597%25E8%25A1%25A8%25E3%2580%2582 "http://localhost:8080/actuator/prometheus%E4%BB%8EPrometheus%E6%8B%89%E5%8F%96%E6%95%B0%E6%8D%AE%EF%BC%8C%E8%BF%94%E5%9B%9E%E4%BA%86%E5%A4%A7%E9%87%8F%E5%8F%AF%E7%94%A8%E7%9A%84%E6%8C%87%E6%A0%87%E4%BF%A1%E6%81%AF%E3%80%82%E6%88%91%E4%BB%AC%E8%BF%99%E9%87%8C%E5%8F%AA%E6%98%BE%E7%A4%BA%E8%BE%93%E5%87%BA%E7%9A%84%E4%B8%80%E5%B0%8F%E9%83%A8%E5%88%86%EF%BC%8C%E5%9B%A0%E4%B8%BA%E5%AE%83%E6%98%AF%E4%B8%80%E4%B8%AA%E5%BE%88%E9%95%BF%E7%9A%84%E5%88%97%E8%A1%A8%E3%80%82")

```
$ curl http://localhost:8080/actuator/prometheus
# HELP jvm_gc_pause_seconds Time spent in GC pause
# TYPE jvm_gc_pause_seconds summary
jvm_gc_pause_seconds_count{action="end of minor GC",cause="G1 Evacuation Pause",} 2.0
jvm_gc_pause_seconds_sum{action="end of minor GC",cause="G1 Evacuation Pause",} 0.009
...
复制代码
```

如前所述，还需要 Micrometer。`Micrometer`为最流行的监控系统提供了一个简单的仪表板，允许仪表化 JVM 应用，而无需关心是哪个供应商提供的指标。它的作用和`SLF4J`类似，只不过它关注的不是 Logging（日志），而是 application metrics（应用指标）。简而言之，它就是应用监控界的 SLF4J。

Spring Boot Actuator 为 Micrometer 提供了自动配置。Spring Boot2 在 spring-boot-actuator 中引入了 micrometer，对 1.x 的 metrics 进行了重构，另外支持对接的监控系统也更加丰富 (`Atlas`、`Datadog`、`Ganglia`、`Graphite`、`Influx`、`JMX`、`NewRelic`、`Prometheus`、`SignalFx`、`StatsD`、`Wavefront`)。

更新后的`application.properties`文件如下所示：

```
management.endpoints.web.exposure.include=health,info,metrics,prometheus
复制代码
```

重启应用程序，并从`http://localhost:8080/actuator/metrics`中检索数据。

```
$ curl http://localhost:8080/actuator/metrics | python -mjson.tool
...
{
"names": [
"http.server.requests",
"jvm.buffer.count",
"jvm.buffer.memory.used",
...
复制代码
```

可以直接通过指标名来检索具体信息。例如，如果查询`http.server.requests`指标，可以按以下方式检索：

```
$ curl http://localhost:8080/actuator/metrics/http.server.requests | python -mjson.tool
...
{
"name": "http.server.requests",
"description": null,
"baseUnit": "seconds",
"measurements": [
        {
"statistic": "COUNT",
"value": 3.0
        },
        {
"statistic": "TOTAL_TIME",
"value": 0.08918682
        },
...
复制代码
```

3、添加 Prometheus
---------------

Prometheus 是 Cloud Native Computing Foundation 的一个开源监控系统。由于我们的应用程序中有一个`/actuator/Prometheus`端点来供 Prometheus 抓取数据，因此你现在可以配置 Prometheus 来监控你的 Spring Boot 应用程序。

Prometheus 有几种安装方法，在本文中，我们将在 Docker 容器中运行 Prometheus。

你需要创建一个`prometheus.yml`文件，以添加到 Docker 容器中。

```
global:
scrape_interval:15s

scrape_configs:
- job_name: 'myspringmetricsplanet'
metrics_path: '/actuator/prometheus'
static_configs:
- targets: ['HOST:8080']
复制代码
```

*   `scrape_interval`：Prometheus 多久轮询一次应用程序的指标
*   `job_name`：轮询任务名称
*   `metrics_path`：指标的 URL 的路径
*   `targets`：主机名和端口号。使用时，替换 HOST 为主机的 IP 地址

如果在 Linux 上查找 IP 地址有困难，则可以使用以下命令：

```
$ ip -f inet -o addr show docker0 | awk '{print $4}' | cut -d '/' -f 1
复制代码
```

启动 Docker 容器并将本地`prometheus.yml`文件，映射到 Docker 容器中的文件。

```
$ docker run \
    -p 9090:9090 \
    -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus
复制代码
```

成功启动 Docker 容器后，首先验证 Prometheus 是否能够通过 `http://localhost:9090/targets`收集数据。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/12efc61f189d4f699ec3c71b9faf9f35~tplv-k3u1fbpfcp-watermark.awebp)

如上图所示，我们遇到`context deadline exceeded`错误，造成 Prometheus 无法访问主机上运行的 Spring Boot 应用程序。如何解决呢？

可以通过将 Docker 容器添加到你的主机网络来解决此错误，这将使 Prometheus 能够访问 Spring Boot 应用程序。

```
$ docker run \
    --name prometheus \
    --network host \
    -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
    -d \
    prom/prometheus
复制代码
```

再次验证，状态指示为 UP。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7593619aa406486bbbc1a4d64e639b6e~tplv-k3u1fbpfcp-watermark.awebp)

现在可以显示 Prometheus 指标。通过访问`http://localhost:9090/graph`，在搜索框中输入`http_server_requests_seconds_max`并单击 “执行” 按钮，将为你提供请求期间的最长执行时间。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0b984043f8454b3faf2efac30bd02c94~tplv-k3u1fbpfcp-watermark.awebp)

4、添加 Grafana
------------

最后添加的组件是 Grafana。尽管 Prometheus 可以显示指标，但 Grafana 可以帮助你在更精美的仪表板中显示指标。Grafana 也支持几种安装方式，在本文中，我们也将在 Docker 容器中运行它。

```
$ docker run --name grafana -d -p 3000:3000 grafana/grafana
复制代码
```

点击 `http://localhost:3000/`，就可以访问 Grafana。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc5fe09521914de0b1b4cb986cd0e613~tplv-k3u1fbpfcp-watermark.awebp)

默认的用户名 / 密码为`admin/admin`。单击 “登录” 按钮后，你需要更改默认密码。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d07e02f3e5a644c6888716ddd71f2590~tplv-k3u1fbpfcp-watermark.awebp)

接下来要做的是添加一个数据源。单击左侧边栏中的 “配置” 图标，然后选择“Data Sources(数据源)”。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c408c52d94554a779b03427f932fe98b~tplv-k3u1fbpfcp-watermark.awebp)

单击`Add data source`（添加数据源）按钮。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1286b21ff04f4011baa75e85c32c3efd~tplv-k3u1fbpfcp-watermark.awebp)

Prometheus 在列表的顶部，选择 Prometheus。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e76992811224453a12db0a386f22fbe~tplv-k3u1fbpfcp-watermark.awebp)

填写可访问 Prometheus 的 URL，将`HTTP Access`设置为`Browser`，然后单击页面底部的`Save＆Test`按钮。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/46ed9af89d934807b99b442127166ac7~tplv-k3u1fbpfcp-watermark.awebp)

一切正常后，将显示绿色的通知标语，指示数据源正在工作。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0fa90a1d787548c0910857e3cb96e484~tplv-k3u1fbpfcp-watermark.awebp)

现在该创建仪表板了。你可以自定义一个，但也可以使用开源的仪表板。用于显示 Spring Boot 指标的一种常用仪表板是 JVM 仪表板。

在左侧边栏中，点击 + 号，然后选择导入。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/77ff5b959f91479faf9ea39630299694~tplv-k3u1fbpfcp-watermark.awebp)

输入 JVM 仪表板的 URL [grafana.com/grafana/das…](https://link.juejin.cn?target=https%3A%2F%2Fgrafana.com%2Fgrafana%2Fdashboards%2F4701%25EF%25BC%258C%25E7%2584%25B6%25E5%2590%258E%25E5%258D%2595%25E5%2587%25BB%25E2%2580%259CLoad(%25E5%258A%25A0%25E8%25BD%25BD)%25E2%2580%259D%25E6%258C%2589%25E9%2592%25AE%25E3%2580%2582 "https://grafana.com/grafana/dashboards/4701%EF%BC%8C%E7%84%B6%E5%90%8E%E5%8D%95%E5%87%BB%E2%80%9CLoad(%E5%8A%A0%E8%BD%BD)%E2%80%9D%E6%8C%89%E9%92%AE%E3%80%82")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1206680754d34c7f91efdbe7db518b92~tplv-k3u1fbpfcp-watermark.awebp)

为仪表板输入一个有意义的名称（例如 MySpringMonitoringPlanet），选择 Prometheus 作为数据源，然后单击 Import 按钮。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e378431b8de44e49d52484f4506df49~tplv-k3u1fbpfcp-watermark.awebp)

到目前为止，你就可以使用一个很酷的 Grafana 仪表板。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/60664ac3b2704342a806a0145c08e9e9~tplv-k3u1fbpfcp-watermark.awebp)

也可以将自定义面板添加到仪表板。在仪表板顶部，单击 Add panel（添加面板）图标。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3319bc9c76474c1b97d73427a3d6de0d~tplv-k3u1fbpfcp-watermark.awebp)

单击 Add new panel（添加新面板）。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4bea0974f71d436896984384ec9f7edf~tplv-k3u1fbpfcp-watermark.awebp)

在 Metrics 字段中，输入 http_server_requests_seconds_max，在右侧栏中的 Panel title 字段中，可以输入面板的名称。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e260c33fcbb74aaba5b1811a698b89eb~tplv-k3u1fbpfcp-watermark.awebp)

最后，单击右上角的 Apply 按钮，你的面板将添加到仪表板。不要忘记保存仪表板。

为应用程序设置一些负载，并查看仪表板上的 http_server_requests_seconds_max 指标发生了什么。

```
$ watch -n 5 curl http://localhost:8080/endPoint1$ watch -n 10 curl http://localhost:8080/endPoint2
复制代码
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/90e65a6738ba456fbfb29223dee9a845~tplv-k3u1fbpfcp-watermark.awebp)

5、结论
----

在本文中，我们学习了如何为 Spring Boot 应用程序添加一些基本监控。这非常容易，只需要通过将 Spring Actuator，Micrometer，Prometheus 和 Grafana 组合使用。

当然，这只是一个起点，但是从这里开始，你可以为 Spring Boot 应用程序扩展和配置更多、更具体的指标。

原文：[mydeveloperplanet.com/2021/03/03/…](https://link.juejin.cn?target=https%3A%2F%2Fmydeveloperplanet.com%2F2021%2F03%2F03%2Fhow-to-monitor-a-spring-boot-app%2F "https://mydeveloperplanet.com/2021/03/03/how-to-monitor-a-spring-boot-app/")  
译文：[www.kubernetes.org.cn/9020.html](https://link.juejin.cn?target=https%3A%2F%2Fwww.kubernetes.org.cn%2F9020.html "https://www.kubernetes.org.cn/9020.html")

**近期热文推荐：**

1.[1,000+ 道 Java 面试题及答案整理 (2021 最新版)](https://link.juejin.cn?target=https%3A%2F%2Fwww.javastack.cn%2Fmst%2F "https://www.javastack.cn/mst/")

2. [别在再满屏的 if/ else 了，试试策略模式，真香！！](https://link.juejin.cn?target=https%3A%2F%2Fwww.javastack.cn%2Farticle%2F2021%2Fstrategy-pattern-instead-of-if-else%2F "https://www.javastack.cn/article/2021/strategy-pattern-instead-of-if-else/")

3. [卧槽！Java 中的 xx ≠ null 是什么新语法？](https://link.juejin.cn?target=https%3A%2F%2Fwww.javastack.cn%2Farticle%2F2021%2Fintellij-idea-ligatures-settings%2F "https://www.javastack.cn/article/2021/intellij-idea-ligatures-settings/")

4.[Spring Boot 2.5 重磅发布，黑暗模式太炸了！](https://link.juejin.cn?target=https%3A%2F%2Fwww.javastack.cn%2Farticle%2F2021%2Fspring-boot-2.5-released%2F "https://www.javastack.cn/article/2021/spring-boot-2.5-released/")

5.[《Java 开发手册（嵩山版）》最新发布，速速下载！](https://link.juejin.cn?target=http%3A%2F%2Fwww.javastack.cn%2Farticle%2F2020%2Falibaba-release-java-develop-rules-songshan%2F "http://www.javastack.cn/article/2020/alibaba-release-java-develop-rules-songshan/")

觉得不错，别忘了随手点赞 + 转发哦！