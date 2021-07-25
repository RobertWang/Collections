> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg3MDYwMzkwMw==&mid=2247484481&idx=1&sn=01aeee2d6b04e694f8f797bd20194c05&chksm=ce8a0951f9fd804776654110d9d385ee70484da3388dcadcb5ffbe6ea0ea6acfffd712688b0b&mpshare=1&scene=1&srcid=0724GXj2xfxqFd3NIdr8b2gg&sharer_sharetime=1627116504791&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

Kona 是由腾讯专业 JVM 技术团队维护开发的，基于 OpenJDK 的，提供长期支持并按季度更新的 JDK 发行版本。Kona 目前作为默认 JDK 应用于腾讯云业务场景及其他 Java 应用场景。Kona 基于 OpenJDK，同时提供了更多的功能拓展及维护。通过腾讯 Kona，用户可以获得更为先进的功能及性能优化，提高用户和开发者的使用体验。目前，腾讯 Kona 仅支持 Linux x86-64 位系统.

使用说明
----

### 简介

TencentKona-8 支持以下特性:

*   Default CDS Archive 提高启动速度.
    
*   Java Flight Recorder 采集 java 应用程序的诊断信息.
    

### Default CDS Archive

Tencent Kona 默认打开 Default CDS Archive 功能， 用户可以通过以下启动标志关闭该功能:

### Java Flight Recorder (JFR)

Tencent Kona 默认关闭 JFR 功能， 用户可通过以下步骤使用 "

### JFR 使用步骤

*   使用以下标志启动 java
    

```
java -XX:+FlightRecorder
```

*   当应用程序运行时，使用以下命令采集 JFR 数据
    

```
jcmd <your_pid> JFR.start name=<record_name> filename=<dump_file_name>.jfr 
```

*   使用以下命令停止 JFR 采集:
    

```
jcmd <your_pid> JFR.stop
```

### JFR 数据处理

请使用 java mission control (jmc) 7.0 以上版本打开 *.jfr 文件

安装说明
----

### 安装腾讯 Kona

从此处下载腾讯 Kona 二进制文件 Releases, 例如: TencentKona-8.0.0-232.x86_64.tar.gz

```
cd <Install_Path>
tar -xvf TencentKona-8.0.0-232.x86_64.tar.gz
export JAVA_HOME=<Install_Path>/TencentKona-8.0.0-232
export PATH=${JAVA_HOME}/bin:$PATH
export CLASSPATH=.:${JAVA_HOME}/lib
```

### 验证腾讯 Kona 版本

java -version 输出应如下:

```
bash#> java -version
openjdk version "1.8.0_232"
OpenJDK Runtime Environment (Tencent Kona 8.0.0) (build 1.8.0_232-18)
OpenJDK 64-Bit Server VM (Tencent Kona 8.0.0) (build 25.232-b18, mixed mode, sharing)
```

项目地址
----

开源地址：https://github.com/Tencent/TencentKona-8