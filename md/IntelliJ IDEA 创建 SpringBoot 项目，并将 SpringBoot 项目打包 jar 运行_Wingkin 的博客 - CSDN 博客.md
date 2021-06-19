> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/Wing_kin666/article/details/108485112)

一、创建 SpringBoot 项目  
1. 选择 spring Initializr 创建项目，并点击 next  
![](https://img-blog.csdnimg.cn/20200909110043100.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dpbmdfa2luNjY2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200909110259824.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dpbmdfa2luNjY2,size_16,color_FFFFFF,t_70#pic_center)  
2. 选择 web - spring web  
![](https://img-blog.csdnimg.cn/20200909110424286.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dpbmdfa2luNjY2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200909110551144.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dpbmdfa2luNjY2,size_16,color_FFFFFF,t_70#pic_center)  
3. 创建好的项目目录结构如下  
![](https://img-blog.csdnimg.cn/20200909111240709.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dpbmdfa2luNjY2,size_16,color_FFFFFF,t_70#pic_center)  
4. 在 hello 文件夹下新建一个 controller 包，然后创建 HelloController.java  
![](https://img-blog.csdnimg.cn/20200909111527585.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dpbmdfa2luNjY2,size_16,color_FFFFFF,t_70#pic_center)  
代码如下：

```
package com.example.hello.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    @RequestMapping("/hello")
    public String HelloWorld() {
        return "Hello World!";
    }
}
```

5. 右键运行 HelloApplication  
![](https://img-blog.csdnimg.cn/20200909111920173.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dpbmdfa2luNjY2,size_16,color_FFFFFF,t_70#pic_center)  
启动成功，默认端口 8080，这个可以在 application.properties 中修改  
![](https://img-blog.csdnimg.cn/20200909112046204.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dpbmdfa2luNjY2,size_16,color_FFFFFF,t_70#pic_center)  
打开浏览器，输入 localhost:8080/hello  
![](https://img-blog.csdnimg.cn/20200909112117948.png#pic_)  
二、Springboot 项目打包成 jar 运行  
1. 按 Alt+F12 打开 Terminal  
![](https://img-blog.csdnimg.cn/20200909112340982.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dpbmdfa2luNjY2,size_16,color_FFFFFF,t_70#pic_center)  
2. 输入 mvn clean package；该命令首先清理了 target 文件夹，然后进行打包。  
![](https://img-blog.csdnimg.cn/2020090911250612.png#pic_center)  
等待一段时间后打包成功，target 目录下生成 jar 文件：  
![](https://img-blog.csdnimg.cn/20200909112553807.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dpbmdfa2luNjY2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200909112742841.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dpbmdfa2luNjY2,size_16,color_FFFFFF,t_70#pic_)  
3.win+R 打开 cmd，进入项目中的 target 文件夹  
![](https://img-blog.csdnimg.cn/20200909113300546.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dpbmdfa2luNjY2,size_16,color_FFFFFF,t_70#pic_cnter)

![](https://img-blog.csdnimg.cn/20200909113232141.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dpbmdfa2luNjY2,size_16,color_FFFFFF,t_70#pic_cer)  
4. 输入 java -jar xxxxx.jar 运行 jar 包  
![](https://img-blog.csdnimg.cn/20200909113540432.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dpbmdfa2luNjY2,size_16,color_FFFFFF,t_70#pic_center)  
打开浏览器，输入 localhost:8080/hello，运行成功。  
![](https://img-blog.csdnimg.cn/20200909113656320.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dpbmdfa2luNjY2,size_16,color_FFFFFF,t_70#pic_ceter)