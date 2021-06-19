> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/d1348b2f9121)

> 首先打开 spring boot pom.xml

```
spring-boot-maven-plugin已经包含了我们需简要打包的插件 (必须存在)

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    
    只需要修改清单选项的packaging 就可以为我们生成不同的可执行文件 (必须存在)
    
    <packaging>jar</packaging>
```

> 开始使用 maven 插件进行打包

```
打开 Maven Projects
```

![](http://upload-images.jianshu.io/upload_images/16639461-1abee7869bd37b8b.png) 1.png

```
打开 Execute Maven Goal
输入命令 clean package
```

![](http://upload-images.jianshu.io/upload_images/16639461-0d20f6bba7e94a26.png) 2.png

```
打开 target 找到生成的jar包(例：demo-0.0.1-SNAPSHOT.jar)
说明打包成功
```

![](http://upload-images.jianshu.io/upload_images/16639461-84916e10b86c33c0.png) 3.png

```
找到当前jar目录 执行 java -jar demo-0.0.1-SNAPSHOT.jar 命令
启动成功 说明项目打包成功
```

![](http://upload-images.jianshu.io/upload_images/16639461-3a3eddee17783c5f.png) 4.png