> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1622997718&ver=3114&signature=gZDO3pJT4NAQ9*46zfjAW18VGHhWsSwiXT64ExBwpX0UlX*x9OrEKWSuNWI07cC*8stecFvTZP8kIcddDmXJPzYa7pzvBs7ojeNmeaTjUb9OIgl7BjzeBA0MUGjsGONw&new=1)

#### ![](https://mmbiz.qpic.cn/mmbiz_png/tO7NEN7wjr5S0EIHqsWAxW2M3K8NSSwfzVbLsQ6rb3OoicbYr0dMmvOKcxu41Eols8sk3EqmwnJ8rETuuia054PQ/640?wx_fmt=png)  

#### 1、Spring Boot 应用暴露监控指标【版本 1.5.7.RELEASE】  

首先，添加依赖如下依赖：

```
<dependency>       
   <groupId>org.springframework.boot</groupId>     
   <artifactId>spring-boot-starter-actuator</artifactId>  
</dependency>   
```

采集应用的指标信息，我们使用的是 prometheus, 相应的我们引入包：

```
<dependency>          
  <groupId>io.prometheus</groupId>    
  <artifactId>simpleclient_spring_boot</artifactId>                     <version>0.0.26</version>    
</dependency>
```

然后，在启动类 `Application.java` 添加如下注解：

```
@SpringBootApplication
@EnablePrometheusEndpoint
@EnableSpringBootMetricsCollector
public class Application {  
    public static void main(String[] args) {        SpringApplication.run(Application.class, args); 
 }
}
```

最后，配置默认的登录账号和密码，在 `application.yml` 中：

```
security:
  user:
    name:user
    password: pwd
```

启动应用程序后，会看到如下一系列的 `Mappings`

![](https://mmbiz.qpic.cn/mmbiz_jpg/tO7NEN7wjr5S0EIHqsWAxW2M3K8NSSwfL8pVM1C9sH98iaXicdMvcsaLGvqwUWn4tzGFtENw9icccoYyg1vAtXxdw/640?wx_fmt=jpeg)

img

利用账号密码访问 http://localhost:8080/application/prometheus ，可以看到 Prometheus 格式的指标数据 

  
![](https://mmbiz.qpic.cn/mmbiz_jpg/tO7NEN7wjr5S0EIHqsWAxW2M3K8NSSwfanMmJJoaKN7HYrpa8GzP6sAnd2uVcFVPMQBQKNIwDYP5ZrcN9iaqYYA/640?wx_fmt=jpeg)

#### 2、Prometheus 采集 Spring Boot 指标数据

首先，获取 Prometheus 的 Docker 镜像：

```
$ docker pull prom/prometheus
```

然后，编写配置文件 `prometheus.yml` ：

```
global:
  scrape_interval: 10s
  scrape_timeout: 10s
  evaluation_interval: 10m
scrape_configs:
  - job_name: spring-boot
    scrape_interval: 5s
    scrape_timeout: 5s
    metrics_path: /application/prometheus
    scheme: http
    basic_auth:
      username: admin
      password: 123456
    static_configs:
      - targets:
        - 192.168.11.54:8099 #此处填写 Spring Boot 应用的 IP + 端口号
```

接着，启动 Prometheus :

```
docker run -d --name prometheus -p 9090:9090
-v D:\test\actuator\prometheus\prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

> 请注意，`D:\test\actuator\prometheus\prometheus.yml` ，是我的配置文件存放地址，我们需要将它放到容器里面去，所以用了`-v`来做文件映射。`/etc/prometheus/prometheus.yml`这个是容器启动的时候去取的默认配置，这里我是直接覆盖掉了它。`prom/prometheus`这是镜像，如果本地没有，就回去你设置好的镜像仓库去取。

启动完成后用`docker ps`看下是否已经启动成功，之后打开浏览器输入：  
`http://localhost:9090/targets`, 检查 Spring Boot 采集状态是否正常, 如果看到下图就是成功了。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tO7NEN7wjr5S0EIHqsWAxW2M3K8NSSwf1ianOv1icwEiaKVeKzibLUvQiaPiance4cxxqZl2tVjibxkp3ibOu0tceXNgPw/640?wx_fmt=jpeg)

img

#### 3、Grafana 可视化监控数据

首先，获取 Grafana 的 Docker 镜像：

```
$ docker pull grafana/grafana1
```

然后，启动 Grafana：

```
$ docker run --name grafana -d -p 3000:3000 grafana/grafana1
```

接着，访问 http://localhost:3000/ 配置 Prometheus 数据源：

> Grafana 登录账号 admin 密码 admin

1.  先配置数据源.
    

![](https://mmbiz.qpic.cn/mmbiz_png/tO7NEN7wjr5S0EIHqsWAxW2M3K8NSSwfVSOXsrqOWwNIBibIrxkcYUQKRCD57bybubN2tf5ZvAUWzBmlwXQK28A/640?wx_fmt=png)

img

2. 配置单个指标的可视化监控面板：

![](https://mmbiz.qpic.cn/mmbiz_jpg/tO7NEN7wjr5S0EIHqsWAxW2M3K8NSSwfuhTR1EOpRE7snzMhzGLyBe5KPWglDldFDYDiagP4iahhAIx5rsLmCUow/640?wx_fmt=jpeg)

img

![](https://mmbiz.qpic.cn/mmbiz_jpg/tO7NEN7wjr5S0EIHqsWAxW2M3K8NSSwfwia5b2lUrZby3shZIjXnAW3X490WTgVNVvPLGHKyibGam6Vlbty938pg/640?wx_fmt=jpeg)

img

![](https://mmbiz.qpic.cn/mmbiz_jpg/tO7NEN7wjr5S0EIHqsWAxW2M3K8NSSwft5Uv0VdtlYGJiaNfoJW5oL7yibIYlcWYhsU3rp13fZQ9228VG2lHOpUQ/640?wx_fmt=jpeg)

img

`prometh`采集的数据

![](https://mmbiz.qpic.cn/mmbiz_png/tO7NEN7wjr5S0EIHqsWAxW2M3K8NSSwfxiafibjSAhfCylXseDtYSyNUXBG8bUWmGCQXDXUT8iabUaRrtgLVogdicA/640?wx_fmt=png)

img

![](https://mmbiz.qpic.cn/mmbiz_jpg/tO7NEN7wjr5S0EIHqsWAxW2M3K8NSSwfFrJgm1lpSY6b0bM6roicYu3nic7qCoewJzeku1kibMQnZ16icXu6UAOjGA/640?wx_fmt=jpeg)

**明天见 (｡･ω･｡)ﾉ♡**