> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/liumiaocn/article/details/89703177?utm_term=alpineglibc%E5%AE%89%E8%A3%85&utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~all~sobaiduweb~default-0-89703177&spm=3001.4430)

Alpine 镜像中的 musl 的 libc，所以一般来说使用 gnu libc 的软件最好使用其他发行版的 linux，但是 Alpine 镜像小而简单的特性是很多用户无法放弃的，这篇文章介绍一下使用 Alpine 中支持 Gnu Libc 的方法。

Musl libc vs Gnu libc
=====================

为什么需要提及到这个问题，可以参看下面的文章，以具体的例子来说明了 Alpine 下 libc 的问题以及简单的介绍

*   [https://liumiaocn.blog.csdn.net/article/details/89702529](https://liumiaocn.blog.csdn.net/article/details/89702529)
*   [https://liumiaocn.blog.csdn.net/article/details/89682894](https://liumiaocn.blog.csdn.net/article/details/89682894)

Alpine 下安装 Gnu libc
===================

在下面的文章中已经有所介绍，这里不再过多赘述，基本上分为 3 步

*   步骤 1: 下载 key

> wget -q -O /etc/apk/keys/sgerrand.rsa.pub [https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub](https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub)

*   步骤 2: 下载 apk 安装文件

> wget [https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.29-r0/glibc-2.29-r0.apk](https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.29-r0/glibc-2.29-r0.apk)

*   步骤 2: 安装

> apk add glibc-2.29-r0.apk

详细参看如下文章：

*   [https://liumiaocn.blog.csdn.net/article/details/89702529](https://liumiaocn.blog.csdn.net/article/details/89702529)

设定 Locale
=========

为什么需要设定 Locale，下面的文章中提到的一个不是很容易确认到这个原因的问题现象，有兴趣的可以读一下，很容易造成中文字符的处理不正确。

*   Maven 镜像打包 war 文件中中文文件名无法生成的问题

事前准备
----

事前准备一个 Alpine 镜像，然后将 Oracle 的 JDK 放入其中，然后执行上述 gnu libc 的安装步骤。详细可参看：[https://liumiaocn.blog.csdn.net/article/details/89702529](https://liumiaocn.blog.csdn.net/article/details/89702529)

```
/tmp # java -version
java version "1.7.0_79"
Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)
/tmp #

```

安装 maven
--------

此处安装老版本的 Maven 3.2.5，其他版本也可同样进行操作，一般并无区别。

*   下载 Maven

```
/tmp # MAVEN_VERSION=3.2.5
/tmp # wget https://apache.osuosl.org/maven/maven-3/${MAVEN_VERSION}/binaries/ap
ache-maven-${MAVEN_VERSION}-bin.tar.gz
Connecting to apache.osuosl.org (140.211.166.134:443)
apache-maven-3.2.5-b 100% |********************************| 7770k  0:00:00 ETA
/tmp # ls apache-maven-3.2.5*
apache-maven-3.2.5-bin.tar.gz
/tmp #

```

*   解压

```
/tmp # mkdir -p /usr/local/share/maven
/tmp # tar xzf apache-maven-${MAVEN_VERSION}-bin.tar.gz -C /usr/local/share/maven
/tmp # ls /usr/local/share/maven
apache-maven-3.2.5
/tmp # 

```

*   设定环境变量并确认

```
/tmp # export PATH=$PATH:/usr/local/share/maven/apache-maven-3.2.5/bin
/tmp # 
/tmp # mvn --version
Apache Maven 3.2.5 (12a6b3acb947671f09b81f49094c53f426d8cea1; 2014-12-14T17:29:23+00:00)
Maven home: /usr/local/share/maven/apache-maven-3.2.5
Java version: 1.7.0_79, vendor: Oracle Corporation
Java home: /usr/local/share/java/jdk1.7.0_79/jre
Default locale: en_US, platform encoding: ANSI_X3.4-1968
OS name: "linux", version: "4.9.87-linuxkit-aufs", arch: "amd64", family: "unix"
/tmp #

```

准备验证 Maven 项目
-------------

准备如下 maven 的 pom.xml 文件以及文件构成即可进行测试，除了 pom.xml 文件之外，其余均可使用 touch 只生成无内容的空文件即可。可以考虑准备好之后拷贝进去或者挂载进去等方式都可以。

```
liumiaocn:multibytes liumiao$ cat pom.xml 
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.liumiaocn</groupId>
	<artifactId>springbootdemo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>war</packaging>

	<name>springbootdemo</name>
	<description>spring boot demo project</description>
</project>
liumiaocn:multibytes liumiao$

```

其他空文件和目录结构, 注意满足约定方式的文件名称 webapp/WEB-INF/web.xml，然后关键是中文文件名的存在

```
liumiaocn:multibytes liumiao$ find . -type d
.
./src
./src/main
./src/main/webapp
./src/main/webapp/WEB-INF
./src/main/webapp/WEB-INF/test
./src/main/java
./src/main/java/com
./src/main/java/com/liumiaocn
./src/main/java/com/liumiaocn/springbootdemo
liumiaocn:multibytes liumiao$ find . -type f
./pom.xml
./src/main/webapp/index.html
./src/main/webapp/WEB-INF/test/测试用例集模版.robot
./src/main/webapp/WEB-INF/test/测试用例模版.robot
./src/main/webapp/WEB-INF/test/测试结果报告模版.html
./src/main/webapp/WEB-INF/web.xml
liumiaocn:multibytes liumiao$ 

```

事前确认
----

确认之后发现 LANG 未设定，于是设定 Locale 的 LANG

```
/tmp # echo $LANG
/tmp # 
/tmp # export LANG=en_US.UTF-8
/tmp #

```

在此基础之上，进行 mvn pacakge 的生成，然后进行确认 war 文件的内容，可以看到中文文件名称的文件未被包含进来

```
liumiaocn:multibytes liumiao$ tar tf target/springbootdemo-0.0.1-SNAPSHOT.war 
META-INF/
META-INF/MANIFEST.MF
WEB-INF/
WEB-INF/classes/
index.html
WEB-INF/web.xml
META-INF/maven/
META-INF/maven/com.liumiaocn/
META-INF/maven/com.liumiaocn/springbootdemo/
META-INF/maven/com.liumiaocn/springbootdemo/pom.xml
META-INF/maven/com.liumiaocn/springbootdemo/pom.properties
liumiaocn:multibytes liumiao$

```

设定方式
----

设定方式其实很简单，安装两个 apk，设定 locale 为 UTF8，具体操作如下所示

*   步骤 1: 安装 glibc-bin

> 下载命令：wget [https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.29-r0/glibc-bin-2.29-r0.apk](https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.29-r0/glibc-bin-2.29-r0.apk)  
> 安装命令：apk add glibc-bin-2.29-r0.apk

执行日志示例如下所示：

```
/multibytes # wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.29-r0/glibc-bin-2.29-r0.apk
Connecting to github.com (52.74.223.119:443)
Connecting to github-production-release-asset-2e65be.s3.amazonaws.com (52.216.100.227:443)
glibc-bin-2.29-r0.ap 100% |****************************************************************************************|  963k  0:00:00 ETA
/multibytes # 
/multibytes # apk add glibc-bin-2.29-r0.apk
(1/2) Installing libgcc (8.3.0-r0)
(2/2) Installing glibc-bin (2.29-r0)
Executing glibc-bin-2.29-r0.trigger
OK: 12 MiB in 17 packages
/multibytes # 

```

*   步骤 1: 安装 glibc-i18n

> 下载命令： wget [https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.29-r0/glibc-i18n-2.29-r0.apk](https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.29-r0/glibc-i18n-2.29-r0.apk)  
> 安装命令：apk add glibc-i18n-2.29-r0.apk

执行日志示例如下所示：

```
/multibytes # wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.29-r0/glibc-i18n-2.29-r0.apk
Connecting to github.com (13.250.177.223:443)
Connecting to github-production-release-asset-2e65be.s3.amazonaws.com (52.216.10.107:443)
glibc-i18n-2.29-r0.a 100% |****************************************************************************************| 7471k  0:00:00 ETA
/multibytes # 
/multibytes # apk add glibc-i18n-2.29-r0.apk
(1/1) Installing glibc-i18n (2.29-r0)
OK: 37 MiB in 18 packages
/multibytes # 

```

*   步骤 3: 设定 locale

> 安装命令：/usr/glibc-compat/bin/localedef -i en_US -f UTF-8 en_US.UTF-8

执行日志示例如下所示：

```
/multibytes # /usr/glibc-compat/bin/localedef -i en_US -f UTF-8 en_US.UTF-8
/multibytes # 

```

确认结果
----

在安装之前已经设定过了 LANG，发现其实并未起作用，发挥作用的 locale 相关的基础不存在自然无法正常动作。上述安装之后，无需任何其他操作，在同一终端执行 mvn clean 和 mvn package，然后通过 tar 命令确认，得到如下结果

```
liumiaocn:multibytes liumiao$ tar tf target/springbootdemo-0.0.1-SNAPSHOT.war 
META-INF/
META-INF/MANIFEST.MF
WEB-INF/
WEB-INF/test/
WEB-INF/classes/
index.html
WEB-INF/test/测试用例集模版.robot
WEB-INF/test/测试用例模版.robot
WEB-INF/test/测试结果报告模版.html
WEB-INF/web.xml
META-INF/maven/
META-INF/maven/com.liumiaocn/
META-INF/maven/com.liumiaocn/springbootdemo/
META-INF/maven/com.liumiaocn/springbootdemo/pom.xml
META-INF/maven/com.liumiaocn/springbootdemo/pom.properties
liumiaocn:multibytes liumiao$ 

```

可以看到已经能够正常动作了，此根本原因是由于 glibc 和相关的内容缺失导致。

> 建议事项：  
> 如果一定要使用 Alpine 版本的 Oracle 的 JDK，建议在 JDK 层的镜像中就加入上述所有用到的 apk 并进行 locale 设定。

注意事项
====

目前 google 出来的大部分文档中包含的如下链接，key 已经失效

*   [https://raw.githubusercontent.com/sgerrand/alpine-pkg-glibc/master/sgerrand.rsa.pub](https://raw.githubusercontent.com/sgerrand/alpine-pkg-glibc/master/sgerrand.rsa.pub)  
    请像本文一样使用官方的最新源，地址如下，这点在官方说明中也进行了强调。
*   [https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub](https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub)

参考文章
====

[https://github.com/sgerrand/alpine-pkg-glibc](https://github.com/sgerrand/alpine-pkg-glibc)