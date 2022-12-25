> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/thread-1730139-1-1.html)

> [md]# 引言这里按照官方介绍进行 Linux 中的 Nacos 快速搭建。

![](https://avatar.52pojie.cn/data/avatar/001/49/77/18_avatar_middle.jpg)zxdsb666.

引言
==

这里按照官方介绍进行 Linux 中的 Nacos 快速搭建。整个安装主要依赖下面几个环境：

1.  64 bit OS，支持 Linux/Unix/Mac/Windows，推荐选用 Linux/Unix/Mac。
2.  64 bit JDK 1.8+；[下载](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) & [配置](https://docs.oracle.com/cd/E19182-01/820-7851/inst_cli_jdk_javahome_t/)。
3.  Maven 3.2.x+；[下载](https://maven.apache.org/download.cgi) & [配置](https://maven.apache.org/settings.html)。

安装 JDK
======

> Select the appropriate JDK version and click Download.

根据自己的操作系统进行相关的环境下载，由于个人使用的是 Window 宿主机 + 虚拟机安装 Linux CetnerOs 的经典搭配，所以下载的版本为最下面的版本：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20221223062410.png)

勾选然后下载：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20221223063119.png)

之后 Oracle 官方会提示登录之后才能下载：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20221223063017.png)

这个登录比较让人讨厌，有时候会因为外网下载非常缓慢要等很久。个人从网上找到一个 “帖子”，里面有国外网友分享了不登录 Oracle 下载 JDK8 相关包的镜像。

> Java SE Development Kit 8 Downloads：[https://www.oracle.com/java/technologies/downloads/#java8-windows](https://www.oracle.com/java/technologies/downloads/#java8-windows)

原帖：[download java from oracle without login (github.com)](https://gist.github.com/wavezhang/ba8425f24a968ec9b2a8619d7c2d86a6)

*   [https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/windows-i586/jdk-8u331-docs-all.zip](https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/windows-i586/jdk-8u331-docs-all.zip)
*   [https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/linux-i586/jdk-8u331-linux-aarch64.rpm](https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/linux-i586/jdk-8u331-linux-aarch64.rpm)
*   [https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/linux-i586/jdk-8u331-linux-aarch64.tar.gz](https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/linux-i586/jdk-8u331-linux-aarch64.tar.gz)
*   [https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/linux-i586/jdk-8u331-linux-arm32-vfp-hflt.tar.gz](https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/linux-i586/jdk-8u331-linux-arm32-vfp-hflt.tar.gz)
*   [https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/linux-i586/jdk-8u331-linux-i586.rpm](https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/linux-i586/jdk-8u331-linux-i586.rpm)
*   [https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/linux-i586/jdk-8u331-linux-i586.tar.gz](https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/linux-i586/jdk-8u331-linux-i586.tar.gz)
*   [https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/linux-i586/jdk-8u331-linux-x64.rpm](https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/linux-i586/jdk-8u331-linux-x64.rpm)
*   [https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/linux-i586/jdk-8u331-linux-x64.tar.gz](https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/linux-i586/jdk-8u331-linux-x64.tar.gz)
*   [https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/unix-i586/jdk-8u331-macosx-x64.dmg](https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/unix-i586/jdk-8u331-macosx-x64.dmg)
*   [https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/solaris-i586/jdk-8u331-solaris-sparcv9.tar.Z](https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/solaris-i586/jdk-8u331-solaris-sparcv9.tar.Z)
*   [https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/solaris-i586/jdk-8u331-solaris-sparcv9.tar.gz](https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/solaris-i586/jdk-8u331-solaris-sparcv9.tar.gz)
*   [https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/solaris-i586/jdk-8u331-solaris-x64.tar.Z](https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/solaris-i586/jdk-8u331-solaris-x64.tar.Z)
*   [https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/solaris-i586/jdk-8u331-solaris-x64.tar.gz](https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/solaris-i586/jdk-8u331-solaris-x64.tar.gz)
*   [https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/windows-i586/jdk-8u331-windows-i586.exe](https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/windows-i586/jdk-8u331-windows-i586.exe)
*   [https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/windows-i586/jdk-8u331-windows-x64.exe](https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/windows-i586/jdk-8u331-windows-x64.exe)

相关依赖包下载之后，可以选择 Xftp 等工具将包上传到 Linux 系统，有了网友分享的源之后，我们也可以在 Linux 中拉取相关镜像包即可：

```
[zxdhome.php?mod=space&uid=485241 ~]$ wget https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/linux-i586/jdk-8u331-linux-x64.tar.gz 
--2022-12-23 14:40:07--  https://javadl.oracle.com/webapps/download/GetFile/1.8.0_331-b09/165374ff4ea84ef0bbd821706e29b123/linux-i586/jdk-8u331-linux-x64.tar.gz
Resolving javadl.oracle.com (javadl.oracle.com)... 223.119.233.65, 2600:140e:6:9b9::3311, 2600:140e:6:987::3311
Connecting to javadl.oracle.com (javadl.oracle.com)|223.119.233.65|:443... connected.
HTTP request sent, awaiting response... 302 Moved Temporarily
Location: https://sdlc-esd.oracle.com/ESD6/JSCDL/jdk/8u331-b09/165374ff4ea84ef0bbd821706e29b123/jdk-8u331-linux-x64.tar.gz?GroupName=JSC&FilePath=/ESD6/JSCDL/jdk/8u331-b09/165374ff4ea84ef0bbd821706e29b123/jdk-8u331-linux-x64.tar.gz&BHost=javadl.sun.com&File=jdk-8u331-linux-x64.tar.gz&AuthParam=1671750069_f11ee1690781cd0db9f8308ea2097081&ext=.gz [following]
--2022-12-23 14:40:07--  https://sdlc-esd.oracle.com/ESD6/JSCDL/jdk/8u331-b09/165374ff4ea84ef0bbd821706e29b123/jdk-8u331-linux-x64.tar.gz?GroupName=JSC&FilePath=/ESD6/JSCDL/jdk/8u331-b09/165374ff4ea84ef0bbd821706e29b123/jdk-8u331-linux-x64.tar.gz&BHost=javadl.sun.com&File=jdk-8u331-linux-x64.tar.gz&AuthParam=1671750069_f11ee1690781cd0db9f8308ea2097081&ext=.gz
Resolving sdlc-esd.oracle.com (sdlc-esd.oracle.com)... 223.119.245.111, 2402:4f00:4001:1b6::b3b, 2402:4f00:4001:1a5::b3b
Connecting to sdlc-esd.oracle.com (sdlc-esd.oracle.com)|223.119.245.111|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 148003999 (141M) [application/x-gzip]
Saving to: ‘jdk-8u331-linux-x64.tar.gz’

100%[==============================================================================================================================================================================================================================================>] 148,003,999 1.15MB/s   in 1m 57s 

2022-12-23 14:42:05 (1.21 MB/s) - ‘jdk-8u331-linux-x64.tar.gz’ saved [148003999/148003999]


```

下载完成之后，我们进行解压缩，然后把 JDK 文件放到相关位置。

```
[zxd@localhost ~]$ ls
a.out  hello.c  jdk-8u331-linux-x64.tar.gz  mysql-community-release-el7-5.noarch.rpm
[zxd@localhost ~]$ tar -zxf jdk-8u331-linux-x64.tar.gz 


```

下面是设置 **JAVA_HOME** 的介绍，JDK8 只需要设置 JAVA_HOME 就可以正常运行 JAVA 命令：

```
Set JAVA_HOME.
-   Korn and bash shells:
    export JAVA_HOME=/opt/jdk8
    export PATH=\$JAVA_HOME/bin:$PATH

```

我们根据教程进行设置：

```
[zxd@localhost jdk8]$ export JAVA_HOME=/opt/jdk8
[zxd@localhost jdk8]$ echo $JAVA_HOME
/opt/jdk8
[zxd@localhost jdk8]$ export PATH=$JAVA_HOME/bin:$PATH
[zxd@localhost jdk8]$ echo $PATH
/opt/jdk8/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/zxd/.local/bin:/home/zxd/bin

```

设置环境变量之后，执行`java`、`javac`、`java -version`如果存在对应的打印内容说明配置正确。但是注意这种方式是**临时生效**的，服务器遇到重启或者切换用户，那么 JAVA 环境变量的设置就会失效。

所以我们可以使用下面的方式让配置**永久生效**：

```
[zxd@localhost ~]$ vim /etc/bashrc
// 进入之后会看到很多其他命令，直接跳转到末尾，按键 i 修改添加下面内容
// /opt/jdk8 修改为自己存放JDK源码的位置
export JAVA_HOME=/opt/jdk8
export PATH=$JAVA_HOME/bin:$PATH
[zxd@localhost ~]$ source /etc/bashrc

```

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20221223081446.png)

注意在修改配置之后需要`source`或者重启让配置修改生效，此外需要注意`$PATH`的配置需要追加，如果直接设置 JAVA_HOME 的环境变量会出现直接覆盖的问题。

安装 Maven
========

目前个人下载的最新版 maven 为 3.8.6，官方要求的最低版本为 3.2.X 是可以满足的，我们进入 maven 的下载页面对应链接右键复制，然后在 Linux 当中直接下载即可：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20221223084823.png)

```
[zxd@localhost ~]$ wget https://dlcdn.apache.org/maven/maven-3/3.8.6/binaries/apache-maven-3.8.6-bin.zip
--2022-12-23 16:45:19--  https://dlcdn.apache.org/maven/maven-3/3.8.6/binaries/apache-maven-3.8.6-bin.zip
Resolving dlcdn.apache.org (dlcdn.apache.org)... 151.101.2.132, 2a04:4e42::644
Connecting to dlcdn.apache.org (dlcdn.apache.org)|151.101.2.132|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 8760013 (8.4M) [application/zip]
Saving to: ‘apache-maven-3.8.6-bin.zip’

100%[==============================================================================================================================================================================================================================================>] 8,760,013   39.6MB/s   in 0.2s   

2022-12-23 16:45:19 (39.6 MB/s) - ‘apache-maven-3.8.6-bin.zip’ saved [8760013/8760013]

[zxd@localhost ~]$ ls
a.out  apache-maven-3.8.6-bin.zip  hello.c  jdk-8u331-linux-x64.tar.gz  mysql-community-release-el7-5.noarch.rpm


```

由于是 zip 包，这里直接安装`unzip`，安装命令如下：

```
[zxd@localhost ~]$ sudo yum install -y unzip

```

安装完成之后，直接解压 maven 压缩包到当前目录，或者放到自己喜欢的目录。接下来的设置为 maven 一样，需要设置环境变量。

```
[zxd@localhost ~]$ vim /etc/bashrc
export JAVA_HOME=/opt/jdk8
export MAVEN_HOME=/opt/maven-3.8.6
export PATH=$MAVEN_HOME/bin:$JAVA_HOME/bin:$PATH

```

然后执行`source /etc/bashrc`，最后通过`mvn -version`检查版本。

```
[zxd@localhost maven-3.8.6]$ mvn -version
Apache Maven 3.8.6 (84538c9988a25aec085021c365c560670ad80f63)
Maven home: /opt/maven-3.8.6
Java version: 1.8.0_331, vendor: Oracle Corporation, runtime: /opt/jdk8/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-1160.76.1.el7.x86_64", arch: "amd64", family: "unix"

```

安装 Git
======

Linux 当中的 git 安装十分简单，只需要按照官方文档说明进行安装即可：

### Debian/Ubuntu

> For the latest stable version for your release of Debian/Ubuntu

```
apt-get install git

```

> For Ubuntu, this PPA provides the latest stable upstream Git version

```
add-apt-repository ppa:git-core/ppa` `# apt update; apt install git

```

### Fedora

 (up to Fedora 21)  

```
yum install git

```

(Fedora 22 and later)

```
dnf install git

```

这里使用`yum`方式安装，安装完成之后使用下面的命令检查 Git 版本：

```
[root@localhost logs]# git --version
git version 1.8.3.1

```

下载源码或者安装包
=========

通过源码和发行包两种方式来获取 Nacos。

从 Github 上下载源码方式
----------------

```
git clone https://github.com/alibaba/nacos.git
cd nacos/
mvn -Prelease-nacos -Dmaven.test.skip=true clean install -U  
ls -al distribution/target/

// change the $version to your actual path
cd distribution/target/nacos-server-$version/nacos/bin

```

下载编译后压缩包方式
----------

```
unzip nacos-server-$version.zip 或者 tar -xvf nacos-server-$version.tar.gz cd nacos/bin

```

启动 Nacos 服务
===========

### Linux/Unix/Mac

启动命令 (standalone 代表着单机模式运行，非集群模式):

```
sh startup.sh -m standalone

```

如果您使用的是 ubuntu 系统，或者运行脚本报错提示 [[符号找不到，可尝试如下运行：

```
bash startup.sh -m standalone

```

如果出现下面的内容，则证明启动成功：

```
[root@localhost logs]# cat start.out 
/opt/jdk8/bin/java -Djava.ext.dirs=/opt/jdk8/jre/lib/ext:/opt/jdk8/lib/ext  -Xms512m -Xmx512m -Xmn256m -Dnacos.standalone=true -Dnacos.member.list= -Xloggc:/opt/nacos/logs/nacos_gc.log -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=100M -Dloader.path=/opt/nacos/plugins,/opt/nacos/plugins/health,/opt/nacos/plugins/cmdb,/opt/nacos/plugins/selector -Dnacos.home=/opt/nacos -jar /opt/nacos/target/nacos-server.jar  --spring.config.additional-location=file:/opt/nacos/conf/ --logging.config=/opt/nacos/conf/nacos-logback.xml --server.max-http-header-size=524288

         ,--.
       ,--.'|
   ,--,:  : |                                           Nacos 2.2.0
,`--.'`|  ' :                       ,---.               Running in stand alone mode, All function modules
|   :  :  | |                      '   ,'\   .--.--.    Port: 8848
:   |   \ | :  ,--.--.     ,---.  /   /   | /  /    '   Pid: 3479
|   : '  '; | /       \   /     \.   ; ,. :|  :  /`./   Console: http://192.168.58.128:8848/nacos/index.html
'   ' ;.    ;.--.  .-. | /    / ''   | |: :|  :  ;_
|   | | \   | \__\/: . ..    ' / '   | .; : \  \    `.      https://nacos.io
'   : |  ; .' ," .--.; |'   ; :__|   :    |  `----.   \
|   | '`--'  /  /  ,.  |'   | '.'|\   \  /  /  /`--'  /
'   : |     ;  :   .'   \   :    : `----'  '--'.     /
;   |.'     |  ,     .-./\   \  /            `--'---'
'---'        `--`---'     `----'

2022-12-23 18:43:26,832 INFO Tomcat initialized with port(s): 8848 (http)

2022-12-23 18:43:27,037 INFO Root WebApplicationContext: initialization completed in 4515 ms

2022-12-23 18:43:35,759 INFO Adding welcome page: class path resource [static/index.html]

2022-12-23 18:43:36,555 WARN You are asking Spring Security to ignore Ant [pattern='/**']. This is not recommended -- please use permitAll via HttpSecurity#authorizeHttpRequests instead.

2022-12-23 18:43:36,557 INFO Will not secure Ant [pattern='/**']

2022-12-23 18:43:36,674 INFO Will secure any request with [org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@2a76b80a, org.springframework.security.web.context.SecurityContextPersistenceFilter@7a138fc5, org.springframework.security.web.header.HeaderWriterFilter@4816c290, org.springframework.security.web.csrf.CsrfFilter@552518c3, org.springframework.security.web.authentication.logout.LogoutFilter@44a2b17b, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@307765b4, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@2c95ac9e, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@7eb01b12, org.springframework.security.web.session.SessionManagementFilter@16423501, org.springframework.security.web.access.ExceptionTranslationFilter@455351c4]

2022-12-23 18:43:36,724 INFO Exposing 1 endpoint(s) beneath base path '/actuator'

2022-12-23 18:43:36,928 INFO Tomcat started on port(s): 8848 (http) with context path '/nacos'

2022-12-23 18:43:37,047 INFO Nacos started successfully in stand alone mode. use embedded storage

2022-12-23 18:44:49,368 INFO Initializing Servlet 'dispatcherServlet'

2022-12-23 18:44:49,369 INFO Completed initialization in 1 ms


```

介绍启动参数
======

我们根据 Nacos 的启动日志，简单分析 JVM 启动参数的设置：

```
/opt/jdk8/bin/java -Djava.ext.dirs=/opt/jdk8/jre/lib/ext:/opt/jdk8/lib/ext  -Xms512m -Xmx512m -Xmn256m -Dnacos.standalone=true -Dnacos.member.list= -Xloggc:/opt/nacos/logs/nacos_gc.log -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=100M -Dloader.path=/opt/nacos/plugins,/opt/nacos/plugins/health,/opt/nacos/plugins/cmdb,/opt/nacos/plugins/selector -Dnacos.home=/opt/nacos -jar /opt/nacos/target/nacos-server.jar  --spring.config.additional-location=file:/opt/nacos/conf/ --logging.config=/opt/nacos/conf/nacos-logback.xml --server.max-http-header-size=524288

```

下面的参数是读取 JAVA 的环境变量。

```
-Djava.ext.dirs=/opt/jdk8/jre/lib/ext:/opt/jdk8/lib/ext

```

```
-Xms512m -Xmx512m -Xmn256m

```

1.  -Xms 为 jvm 启动时分配的内存，比如 - Xms512m，表示分配 512M
2.  -Xmx 为 jvm 运行过程中分配的最大内存，比如 - Xms512m，表示 jvm 进程最多只能够占用 512M 内存
3.  -Xmn 表示年轻代的大小设置。

```
 -Dnacos.standalone=true

```

标识强制使用单机方式运行，如果是单机运行一定要加入此参数（默认启动为集群方式）。

```
-Dnacos.member.list= 

```

代表设置 nacos 集群节点列表，该参数可以看作是`cluster.conf`文件的一个替代。

1.  单机模式下：**StandaloneMemberLookup**
2.  集群模式
    1.  `cluster.conf`文件存在：**FileConfigMemberLookup**
    2.  cluster.conf 文件不存在或者 -Dnacos.member.list 没有设置：**AddressServerMemberLookup**

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20221224201722.png)

```
-Xloggc:/opt/nacos/logs/nacos_gc.log

```

> -Xloggc：指定 GC log 的位置，以文件输出，帮助开发人员分析问题

gc 文件可以在`/nacos/logs`中找到，比如使用`cat nacos_gc.log.0.current`可以查找 GC 日志：

```
[root@localhost logs]# cat nacos_gc.log.0.current 
Java HotSpot(TM) 64-Bit Server VM (25.331-b09) for linux-amd64 JRE (1.8.0_331-b09), built on Mar 10 2022 11:12:57 by "java_re" with gcc 7.3.0
Memory: 4k page, physical 995668k(78184k free), swap 2097148k(2009596k free)
CommandLine flags: -XX:GCLogFileSize=104857600 -XX:InitialHeapSize=536870912 -XX:MaxHeapSize=536870912 -XX:MaxNewSize=268435456 -XX:NewSize=268435456 -XX:NumberOfGCLogFiles=10 -XX:+PrintGC -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseGCLogFileRotation 
2022-12-23T18:43:21.752+0800: 1.859: [GC (Allocation Failure) 2022-12-23T18:43:21.752+0800: 1.859: [DefNew: 209792K->5071K(235968K), 0.0136621 secs] 209792K->5071K(498112K), 0.0137497 secs] [Times: user=0.02 sys=0.00, real=0.02 secs] 
2022-12-23T18:43:22.539+0800: 2.646: [Full GC (Metadata GC Threshold) 2022-12-23T18:43:22.539+0800: 2.646: [Tenured: 0K->6454K(262144K), 0.0522422 secs] 187940K->6454K(498112K), [Metaspace: 20388K->20388K(1069056K)], 0.0523591 secs] [Times: user=0.05 sys=0.00, real=0.05 secs] 
// 省略大量打印

```

我们提取其中一行进行介绍：

```
2022-12-23T18:43:21.752+0800: 1.859: [GC (Allocation Failure) 2022-12-23T18:43:21.752+0800: 1.859: [DefNew: 209792K->5071K(235968K), 0.0136621 secs] 209792K->5071K(498112K), 0.0137497 secs] [Times: user=0.02 sys=0.00, real=0.02 secs]

```

`DefNew: 209792K->5071K(235968K), 0.0136621 secs]` 标识新生代可用空间是`235968K`，垃圾此时占用了`209792K`，回收之后剩下`5071K`的存活对象。

```
-verbose:gc -XX:+PrintGCDetails

```

`-verbose:gc`和`-XX:+PrintGCDetails`在官方文档中有说明两者**功能一样**，都用于**垃圾收集时的信息打印**。但是也有不同点：

**-verbose:gc** 是 稳定版本  
参见：[http://docs.oracle.com/javase/7/docs/technotes/tools/windows/java.html](http://docs.oracle.com/javase/7/docs/technotes/tools/windows/java.html)

**-XX:+PrintGC** 是 非稳定版本  
可能在未通知的情况下删除，在下面官方文档中是`-XX:-PrintGC` 。  
因为被标记为 manageable，所以可以通过如下三种方式修改：  
1、com.sun.management.HotSpotDiagnosticMXBean API  
2、JConsole  
3、jinfo -flag

介绍完 GC 日志打印之后，是接连几个重要的参数：

```
-XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=100M

```

我们先介绍前两个参数，这两个参数都是给打印日志添加日期使用的：

```
-XX:+PrintGCDateStamps

```

GC 日志中添加日期标志（日志每行开头显示绝日期及时间，单位为秒）

```
-XX:+PrintGCDateStamps
日志输出示例：
2014-01-03T12:08:38.102-0100: [GC 66048K->53077K(251392K), 0,0959470 secs]
2014-01-03T12:08:38.239-0100: [GC 119125K->114661K(317440K), 0,1421720 secs]

```

日志中添加时间标志（日志每行开头显示自从 JVM 启动以来的时间）

```
-XX:+PrintGCTimeStamps
日志输出示例：
0,185: [GC 66048K->53077K(251392K), 0,0977580 secs]
0,323: [GC 119125K->114661K(317440K), 0,1448850 secs]

```

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20221224205510.png)

接下来的三个参数经常被放到一起使用：

```
-XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=100M

```

我们以 Nacos 的配置为例，每个文件为 100M，其实文件为，一共划分为 10 个文件，也就是说最大存储的 GC 日志为 1G 的大小。 此参数的作用是可以将日志文件进行切割，防止单个日志文件过大。这个参数实际上实际上被网友不推荐使用，具体可以看下面的文章：

[http://link.zhihu.com/?target=https%3A//dzone.com/articles/try-to-avoid-xxusegclogfilerotation](http://link.zhihu.com/?target=https%3A//dzone.com/articles/try-to-avoid-xxusegclogfilerotation)

```
- Dloader.path=/opt/nacos/plugins,/opt/nacos/plugins/health,/opt/nacos/plugins/cmdb,/opt/nacos/plugins/selector

```

Java 在启动时`-Dloader.path`参数指定加载 jar 包的路径，`plugins`为扩展插件。

```
/opt/nacos/target/nacos-server.jar

```

这个不需要过多解释，jar 包的运行位置。接下来有一个 SpringBoot 所需的参数：

```
--spring.config.additional-location=file:/opt/nacos/conf/

```

这个配置在 SpringBoot 文档的解释如下：

> 原文：[https://docs.spring.io/spring-boot/docs/2.1.9.RELEASE/reference/html/boot-features-external-config.html](https://docs.spring.io/spring-boot/docs/2.1.9.RELEASE/reference/html/boot-features-external-config.html)

我们提取原文：

> Alternatively, when custom config locations are configured by using `spring.config.additional-location`, they are used in addition to the default locations. Additional locations are searched before the default locations. For example, if additional locations of `classpath:/custom-config/,file:./custom-config/` are configured, the search order becomes the following:

（另外，）当使用 spring.config.extra-location 配置自定义配置位置时，它们会在默认位置之外使用。附加位置会在默认位置之前被搜索到。例如，如果配置了 classpath:`/custom-config/`,file`:./custom-config/`的附加位置，搜索顺序变成如下：

```
1.  file:./custom-config/
2.  classpath:custom-config/
3.  file:./config/
4.  file:./
5.  classpath:/config/
6.  classpath:/

```

```
--logging.config=/opt/nacos/conf/nacos-logback.xml

```

logback 的配置文件位置，这里截取该文件的一段配置：

```
   <appender 
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_HOME}/naming-server.log</file>
        <append>true</append>
        <rollingPolicy>
            <fileNamePattern>${LOG_HOME}/naming-server.log.%d{yyyy-MM-dd}.%i</fileNamePattern>
            <maxFileSize>1GB</maxFileSize>
            <maxHistory>7</maxHistory>
            <totalSizeCap>7GB</totalSizeCap>
            <cleanHistoryOnStart>true</cleanHistoryOnStart>
        </rollingPolicy>
        <encoder>
            <Pattern>%date %level %msg%n%n</Pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>


```

和单词的意思一致，指定 Http 请求头的大小。这里换算下来是 512KB：

```
 --server.max-http-header-size=524288

```

这里说一个题外话，这个值在 Spring Boot 2.1 开始，可使用 DataSize 可解析值，上面的值可以替换为下面的内容：

```
server.max-http-header-size=512KB

```

参数设置有待商榷，因为 Tomcat 默认的大小为 8KB，同时`Server.max-http-header-size` 是无论大小都会为接口**开辟指定的 内存**用来接收请求。

以上就是 Nacos 的 JVM 启动参数介绍，可以发现有部分参数其实是不合理的。

访问 Nacos
========

最后访问：[http://192.168.58.128:8848/nacos/#/login](http://192.168.58.128:8848/nacos/#/login) 出现 Nacos 界面，搭建成功。

> 默认的用户名和密码都是 **nacos**。

总结
==

以上就是快速部署一个 Nacos 的过程，整个过程非常简单，比较值得关注的是最后的 JVM 参数介绍部分。