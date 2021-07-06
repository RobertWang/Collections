> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI4Njk5OTg1MA==&mid=2247489147&idx=1&sn=371a4073b3a95adce12b83d883c1f1cc&chksm=ebd5023edca28b2898f5a657c01307fa2dde40fd4c4424cd479a8c2af50a930de70d81c21862&mpshare=1&scene=1&srcid=0706XNuBx4OppvH29CQXBb8r&sharer_sharetime=1625557681184&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

1. 前言  
2. 瘦身前的 Jar 包  
3. 解决方案

**一、前言**

[Spring Boot](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247497986&idx=1&sn=6584affae12fe5293617a65090d7d734&chksm=eb507c34dc27f522589ab0e05b40bd8e3b77b9ce8ca7688c2fda896f242d8d1ddb3e7d33f353&scene=21#wechat_redirect) 部署起来虽然简单，如果服务器部署在公司内网，速度还行，但是如果部署在公网，部署起来实在头疼：编译出来的 Jar 包很大，如果工程引入了许多开源组件（[Spring Cloud](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247493585&idx=2&sn=4bafea16b54f2e3470e923c36dbf6c54&chksm=eb5062e7dc27ebf13d4a5189cfcfcc09c62048eec8e232887b21d4bf3516d74bae343befc879&scene=21#wechat_redirect) 等），那就更大了。

这个时候如果想要对线上运行工程有一些微调，则非常痛苦。

**二、瘦身前的 Jar 包**

[Tomcat](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247499387&idx=1&sn=e5463879ace24489d05940abb526a351&chksm=eb507b4ddc27f25bf17c1e3f98781b76a8a0180218c7e56461c7024768fd7667d719dcd68b01&scene=21#wechat_redirect) 在部署 Web 工程的时候，可以进行增量更新，[Spring Boot](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247497986&idx=1&sn=6584affae12fe5293617a65090d7d734&chksm=eb507c34dc27f522589ab0e05b40bd8e3b77b9ce8ca7688c2fda896f242d8d1ddb3e7d33f353&scene=21#wechat_redirect) 也是可以的～

[Spring Boot](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247497986&idx=1&sn=6584affae12fe5293617a65090d7d734&chksm=eb507c34dc27f522589ab0e05b40bd8e3b77b9ce8ca7688c2fda896f242d8d1ddb3e7d33f353&scene=21#wechat_redirect) 编译出来的 Jar 包中，磁盘占用大的，是一些外部依赖库（jar 包），例如：进入项目工程根目录，执行 [mvn clean install](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247486164&idx=2&sn=808a383535bb32caa0087529348e604f&chksm=eb538fe2dc2406f465558f96285727b6584637bda752e5813eeb5cfd1758cf235e81265bf094&scene=21#wechat_redirect) 命令，得到的 Jar 包，用压缩软件打开，目录结构如下：

![](https://mmbiz.qpic.cn/mmbiz_png/TNUwKhV0JpSuBiaRDn5SgyFAjib8auYGu5yknTEvDh8vHyTnFarlpSVpZJhFmpwg30PSia2BYicyL8lTPsb93YfOcw/640?wx_fmt=png)

整个 Jar 包 18.18 MB, 但是 BOOT-INF/lib 就占用了将近 18 MB：

![](https://mmbiz.qpic.cn/mmbiz_png/TNUwKhV0JpSuBiaRDn5SgyFAjib8auYGu55vkZRsD2icyBacwMMOWICemsKu1FmaS2H90yfNAX13uzAmlbNBI5ZvA/640?wx_fmt=png)

**三、解决方法**

**步骤 1: 正常编译 JAR 包，解压出 lib 文件夹**

POM 文件如下：

```
<build>  
    <plugins>  
        <plugin>  
            <groupId>org.springframework.boot</groupId>   
            <artifactId>spring-boot-maven-plugin</artifactId>  
            <configuration>  
                <mainClass>com.johnnian.App</mainClass>  
                <layout>ZIP</layout>  
            </configuration>  
            <executions>  
            <execution>  
                 <goals>  
                     <goal>repackage</goal>  
                 </goals>  
             </execution>  
           </executions>  
        </plugin>  
     <plugins>  
<build>


```

进入项目根目录，执行命令：mvn clean install

将编译后的 Jar 包解压，拷贝 BOOT-INF 目录下的 lib 文件夹 到目标路径；

**步骤 2: 修改 pom.xml 配置，编译出不带 lib 文件夹的 Jar 包**

```
<build>  
    <plugins>  
        <plugin>  
            <groupId>org.springframework.boot</groupId>   
            <artifactId>spring-boot-maven-plugin</artifactId>  
            <configuration>  
                <mainClass>com.johnnian.App</mainClass>  
                <layout>ZIP</layout>  
                <includes>   
                    <include>  
                        <groupId>nothing</groupId>  
                        <artifactId>nothing</artifactId>  
                    </include>    
                </includes>  
            </configuration>  
            <executions>  
                <execution>  
                    <goals>  
                        <goal>repackage</goal>  
                    </goals>  
                </execution>  
            </executions>  
        </plugin>  
     <plugins>  
<build>


```

配置完成后，再次执行编译：mvn clean install

生成的 Jar 包体积明显变小，如下所示， 外部的 jar 包已经不会被引入了：

![](https://mmbiz.qpic.cn/mmbiz_png/TNUwKhV0JpSuBiaRDn5SgyFAjib8auYGu5DEwwJDZo4qFM8uNyyVVJJazlCf6Fs7Kicleq4dHbrG4gKencaGayzYg/640?wx_fmt=png)

**步骤 3: 运行编译后的 Jar 包**

将 步骤 1 解压出来的 lib 文件夹、步骤 2 编译的 jar 包放在同一个目录, 运行下面命令：

```
java -Dloader.path=/path/to/lib -jar /path/to/springboot-jsp-0.0.1-SNAPSHOT.jar


```

或者在 maven 中输入一下命令导出需要用到的 jar 包

```
mvn dependency:copy-dependencies -DoutputDirectory=F:\\ideaWorkPlace\\AnalysisEngine\\lib -DincludeScope=runtime


```

**备注：**

将 / path/to / 改成实际的路径。

-Dloader.path=lib 文件夹路径

最终目录文件结构是：

```
├── lib   #lib文件夹  
└── springboot-jsp-0.0.1-SNAPSHOT.jar


```

**说明：**

1、通常，一个工程项目架构确定后，引入的 jar 包基本上不会变，改变的大部分是业务逻辑；

2、后面如果需要变更业务逻辑，只需要轻量地编译工程，大大提高项目部署的效率。

版权声明：本文为 CSDN 博主「yjgithub」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。原文链接：https://blog.csdn.net/yjgithub/article/details/80475521