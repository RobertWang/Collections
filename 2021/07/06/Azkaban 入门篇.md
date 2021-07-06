> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/22184691)

Azkaban 是由 Linkedin 公司推出的一个批量工作流任务调度器，用于在一个工作流内以一个特定的顺序运行一组工作和流程。Azkaban 使用 job 配置文件建立任务之间的依赖关系，并提供一个易于使用的 web 用户界面维护和跟踪你的工作流。

![](https://pic3.zhimg.com/913dfc718a465fd3d18acca50626f2be_b.png)

在介绍 Azkaban 之前，我们先来看一下现有的两个工作流任务调度系统。知名度比较高的应该是 Apache Oozie，但是其配置工作流的过程是编写大量的 XML 配置，而且代码复杂度比较高，不易于二次开发。另外一个应用也比较广泛的调度系统是 Airflow，但是其开发语言是 Python。由于我们团队内部使用 Java 作为主流开发语言，所以选型的时候就被淘汰掉了。我们选择 Azkaban 的原因基于以下几点：

*   提供功能清晰，简单易用的 Web UI 界面  
    
*   提供 job 配置文件快速建立任务和任务之间的依赖关系  
    
*   提供模块化和可插拔的插件机制，原生支持 command、Java、Hive、Pig、Hadoop  
    
*   基于 Java 开发，代码结构清晰，易于二次开发  
    

**1、Azkaban 的安装**

Azkaban 有两种部署方式：solo server mode 和 cluster server mode。

*   solo server mode(单机模式)：该模式中 webServer 和 executorServer 运行在同一个进程中，进程名是 AzkabanSingleServer。可以使用自带的 H2 数据库或者配置 mysql 数据。该模式适用于小规模的使用。
*   cluster server mode(集群模式)：该模式使用 MySQL 数据库，webServer 和 executorServer 运行在不同进程中，该模式适用于大规模应用。  
    

Azkaban 的组成如下：

![](https://pic3.zhimg.com/d9843f0a65a47ef123a340c7a3d1f056_b.png)

其实在单机模式中，AzkabanSingleServer 进程只是把 AzkabanWebServer 和 AzkabanExecutorServer 合到一起启动而已。

下面介绍单机版的安装过程。

(1) 下载 Azkaban

下载地址：[http://azkaban.github.io/downloads.html](https://link.zhihu.com/?target=http%3A//azkaban.github.io/downloads.html)

官网提供的压缩包是 Azkaban2.5 版本的，但是最新的版本是 3.0。如果需要使用 3.0 版本，只需要到 github 上下载源码后编译即可。github 地址是：[GitHub - azkaban/azkaban: Azkaban workflow manager.](https://link.zhihu.com/?target=https%3A//github.com/azkaban/azkaban)

(2) 解压安装包

下载后的安装包是：azkaban-solo-server-x.x.x.tar.gz，解压后的主要目录如下所示：

```
bin：包含启动和停止的脚本

conf：配置文件目录

data：默认自带数据库H2的数据目录

executions：每个执行的工作流都会生成一个对应的文件夹，放在该目录下

extlib：第三方扩展包

lib：依赖的jar包

logs：执行日志

plugins：可拔插的插件包

projects：用户上传的压缩包所在目录

sql：运行所需要的mysql表的建表语句

web：前端展示界面对应的HTML/CSS/JS文件


```

(3) 数据库安装配置  

MySQL 安装过程 (略)

进入 MySQL 命令行后，创建数据库：

*   CREATE DATABASE azkaban;  
    

创建用户名和密码：

*   CREATE USER 'username'@'%' IDENTIFIED BY 'password';  
    

给用户授权：

*   GRANT SELECT,INSERT,UPDATE,DELETE ON <database>.* to '<username>'@'%' WITH GRANT OPTION;  
    

导入下载包 azkaban-sql-script-x.x.x.tar.gz 中脚本 “create.all.sql”

*   SOURCE create.all.sql;  
    

检查下载包 web 和 executor 的 lib 文件下是否有 mysql 驱动，若不存在，则拷贝一个。

(4) 修改配置文件 conf/azkaban.properties(以下为可修改部分，其他默认即可)

```
#设置项目主标题

azkaban.name=Local

#设置项目副标题

azkaban.label=My Local Azkaban

#设置为上海时间(东八区)，否则会按美国时间执行

default.timezone.id=Asia/Shanghai

#注释掉默认的H2数据库配置后，配置MySQL数据库

database.check.version=false

database.type=mysql

mysql.port=3306

mysql.host=192.168.0.1

mysql.database=azkaban

mysql.user=username

mysql.password=password

mysql.numconnections=100

#配置禁用Jetty服务器

jetty.use.ssl=false

jetty.ssl.port=8043

jetty.maxThreads=25

jetty.port=8081

#配置告警邮件

mail.sender=xxx@163.com

mail.host=smtp.163.com

mail.user=mailUsername

mail.password=mailPassword

#配置azkaban web url

azkaban.webserver.url=http://azkaban.xxx.com/


```

(5) 配置 plugin 插件  

在官网下载 azkaban-jobtype-x.x.x.tar.gz 文件，解压到 azkaban 安装目录下的 plugin 目录下，并重命名为 jobtypes 目录，然后配置 commonprivate.properties 文件如下：

```
jobtype.global.classpath=本地hadoop的config文件和lib文件

hadoop.classpath=同上

#配置各个组件的home

hadoop.home=/home/hadoop/hadoop-2.6.0-cdh5.5.0

pig.home=/home/hadoop/azkaban/plugins/jobtypes/pig

hive.home=/home/hadoop/hive-1.1.0-cdh5.5.0

spark.home=/home/hadoop/spark-1.6.0-bin-hadoop2.6

azkaban.home=/home/hadoop/azkaban/azkaban-solo-server-3.0.0


```

(6) 启动与关闭  

启动命令：

*   ./bin/azkaban-solo-start.sh  
    

关闭命令：

*   ./bin/azkaban-solo-shutdown.sh  
    

至此，Azkaban 的单机版安装完成，在浏览器中访问：[http://hostname:8081](https://link.zhihu.com/?target=http%3A//hostname%3A8081) 就可以看到 Azkaban 的 UI 界面啦！

2、Azkaban 界面功能说明

Azkaban 主界面中四个标签页，分别对应了它的四大功能模块：

![](https://pic1.zhimg.com/9c55e99e276f409ecb7429a591cae24c_b.png)

*   Projects：每个独立的 Flows 都对应一个 Projects，Flows 将在 Projects 中运行  
    
*   Scheduling：显示定时执行任务  
    
*   Executing：显示正在执行的任务  
    
*   History：显示历史执行任务  
    

由于 Projects 模块涉及的功能比较多，下面重点介绍一下。点击进入一个 Project 后，如下：

![](https://pic4.zhimg.com/58953a81cb930858679d1b5938aa250f_b.png)

界面右上角的三个按钮分别可以删除、上传、下载 Project，我们编写完成并打包的 zip 文件，就是在这里上传的。界面中的三个标签页功能如下：

*   Flows：工作流程，由多个 job 组成，点击还可以进入图表界面  
    

![](https://pic3.zhimg.com/3e4035f6352835eb0803c530b39f84f2_b.png)

选择任意一个 job 节点，右键点击 “Open Job...” 可以看到该节点的配置参数，可编辑

![](https://pic3.zhimg.com/41f9a84c41f89a57c95d5a50fa958646_b.png)

*   Permissions：用于权限管理  
    
*   Project Logs：工程日志  
    

点击 Execute Flow 按钮开始执行工作流，弹出界面如下：

![](https://pic2.zhimg.com/6727ba6bbc8c08ab1438e270a8b8da95_b.png)

*   Flow View：该界面右边的流程图中的每个节点，都可以 Disable 和 Enable  
    

![](https://pic1.zhimg.com/60e9271c3feaf6f1ab29ff6d2ac694a0_b.png)

*   Notification：可以配置成功或失败时是否发出邮件。  
    
*   Failure Options：当任务流执行失败是选择处理策略，默认是仅完成当前运行的 job  
    
*   Concurrent：并行任务执行设置  
    
*   Flow Parameters：设置执行的参数，用于覆盖全局设置的参数  
    

最后点击 Execute 按钮即可执行。执行完成的效果如下：

![](https://pic1.zhimg.com/dd76dd54bcf8c3b0ab87d0eb05b1fe08_b.png)

每个执行成功的节点是绿色表示，如果执行失败，则用红色表示。

3、一个简单的例子

下面通过一个列子讲解 Azkaban 的开发流程。

(1) 新建一个文件夹，在文件夹下创建两个文件 hello1.job 和 hello2.job，内容如下：

hello1.job：

```
type=command

command=echo “this is hello1 job”


```

hello2.job：  

```
type=command

command=echo “this is hello2 job”

dependencies=hello1


```

(2) 把两个文件打包成 hello.zip 文件  

(3) 新建一个 Azkaban 工程：Hello，并上传 hello.zip 压缩文件

![](https://pic3.zhimg.com/4985c9ccd3da6db6e05d0e50527f69ae_b.png)

(4) 上传成功后，可以在界面分别看到两个节点

![](https://pic2.zhimg.com/a2e63191b302e68c2ca15b5525eb2851_b.png)

(5) 执行并查看输出

![](https://pic4.zhimg.com/ee86c250c7395f09d0398c17c2d639af_b.png)

(6) 点击 Details，查看具体日志

![](https://pic3.zhimg.com/2001bef566929050d4909da181129ade_b.png)![](https://pic3.zhimg.com/944417fb7ce9902e703d6edf48159196_b.png)

至此，一个完整的任务从开发到执行完成。