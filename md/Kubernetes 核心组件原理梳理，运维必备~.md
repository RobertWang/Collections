> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA3OTc0MzY1Mg==&mid=2247515151&idx=4&sn=0a9a9d9cdd573c340b366bcd526c38e9&chksm=9fac23c4a8dbaad247fe3db35b0f0c9c3b9dc25ad0d1ffeb084545fa76d8c0643b05c4078aae&mpshare=1&scene=1&srcid=0819cjoTgK6n5cg2cflWVk1I&sharer_sharetime=1629345297190&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

### 1. 核心组件原理 —— pod 核心原理

#### 1.1 pod 是什么

*   pod 也可以理解是一个容器，装的是 docker 创建的容器，也就是用来封装容器的一个容器；
    
*   pod 是一个虚拟化分组， 有自己的 IP 地址和主机名 hostname，利用 namespace 进行资源隔离，相当于一台独立沙箱环境；
    
*   pod 相当于一台独立主机，内部可以封装一个或多个容器 (通常是一组相关的容器)，内部容器之间访问采用 localhost。
    

#### 1.2 pod 用来干什么

通常情况下，在服务部署的时候，使用 pod 来管理一组相关的服务 (一个 pod 中要么部署一个服务，要么部署一组有关系的服务)。如下图是部署了一组有关系的服务的结构图，其中 C 表示容器 (container)，下面的 pod 里就有很多个容器。

![](https://mmbiz.qpic.cn/mmbiz_png/yNKv1P4Q9eXfK9Siav1Kuibwkj7JqwIRFVQ9qGeclIEiav5cQJrdbLVOmAapcfK97EKp5h3NIicN9ogLenET9HDmdQ/640?wx_fmt=png)

如何理解一组相关的服务？

如下图：有一个请求是访问 Nginx，然后部署了 Nginx 的容器就把请求转发给部署了 web 服务的容器，web 再访问数据库，然后请求会依次返回来数据，最后再返回给用户。

因此在 链式调用的调用链路上的服务 叫做一组相关的服务。

![](https://mmbiz.qpic.cn/mmbiz_png/yNKv1P4Q9eXfK9Siav1Kuibwkj7JqwIRFV728ZHTXSIRZDUpJVOGq90m9AEnWr2CLlcoM6SsvmwQicpPF6H5eJETQ/640?wx_fmt=png)

#### 1.3 实现 web 服务集群

只需要复制多个 pod 的副本即可，这也是 k8s 管理的先进之处。k8s 如果要进行扩容或缩容，只需要控制 pod 的数量即可。比如上面那个部署模式，服务集群就是复制多个这样的 pod。

![](https://mmbiz.qpic.cn/mmbiz_png/yNKv1P4Q9eXfK9Siav1Kuibwkj7JqwIRFV3BHibfCeGLTo2icraZYicicu1kmx276kp4ykpaDdQebrUeou6Q1xZfSicmA/640?wx_fmt=png)

#### 1.4 pod 底层网络和数据存储是如何进行的

前面说过 pod 内部的容器也是一个独立的沙箱环境，因此也有自己的 ip 和 端口。如果内部容器还是通过 ip:port 来通信，相当于还是远程访问，这样的话性能会受到一定的影响。如何提高内部容器之间访问的性能呢？

![](https://mmbiz.qpic.cn/mmbiz_png/yNKv1P4Q9eXfK9Siav1Kuibwkj7JqwIRFVO9BtnBv92r1tdFCDicT7Gss2zF7V1oysm2MulAkFoVCQCqEtvOwuPAw/640?wx_fmt=png)

**pod 底层**

*   pod 内部容器创建之前，必须先创建 pause 容器。pause 有两个作用：共享网络和共享存储。
    
*   每个服务容器共享 pause 存储，不需要自己存储数据，都交给 pause 维护。
    
*   pause 也相当于这三个容器的网卡，因此他们之间的访问可以通过 localhost 方式访问，相当于访问本地服务一样，性能非常高（就像本地几台虚拟机之间可以 ping 通）。
    

### 2. ReplicaSet 副本控制器

#### 2.1 副本控制器基本理解

作用：管理控制 pod 副本（服务集群）的数量，以使其永远与预期设定的数量保持一致。

例如：replicas = 3 （创建 3 个副本，这是提前设置好的）

![](https://mmbiz.qpic.cn/mmbiz_png/yNKv1P4Q9eXfK9Siav1Kuibwkj7JqwIRFVf8D3QTbHQ6BxyAYanlKnsic61luR8P1BB3VpC4Gic2W9eXXeFyU8lMpQ/640?wx_fmt=png)

当副本设置为 3 时，副本控制器将会永远保证副本数量为 3。因此当有 pod 服务宕机时（如上面第 3 个 pod），那副本控制器会立马重新创建一个新的 pod，就能够保证副本数量一直为预先设定好的 3 个。

#### 2.2 ReplicaSet 和 ReplicationController 的区别

ReplicaSet 和 ReplicationController 都是副本控制器，其中：

*   相同点：都有前面 2.1 节所描述的功能
    
*   不同点：标签选择器的功能不同。ReplicaSet 可以使用标签选择器进行 单选 和 复合选择；而 ReplicationController 只支持 单选操作。
    

**什么意思呢？**

假设下面有下面两个不同机器上的 Node 结点，如何知道它们的 pod 其实都是相同的呢？答案是通过标签。

给每个 pod 打上标签 (key=value 格式，如下图中的 app=web, release=stable，这有两个选项，相同的 pod 副本的标签是一样的)，于是副本控制器可以通过标签选择器 seletor 去选择一组相关的服务。

一旦 selector 和 pod 的标签匹配上了，就表明这个 pod 是当前这个副本控制器控制的，表明了副本控制器和 pod 的所属关系。如下图中 seletor 指定了 app = web 和 release=stable 是复合选择，要用 ReplicaSet 才能实现若用 ReplicationController 的话只能选择一个，如只选择匹配 app=web 标签。这样下面的 3 个 pod 就归这个副本控制器管。

![](https://mmbiz.qpic.cn/mmbiz_png/yNKv1P4Q9eXfK9Siav1Kuibwkj7JqwIRFVCmm0XpP0VJwUd2UZLH4JWib0y07dPwheCLYTicXaE2ibJP8rpuSbDoEQQ/640?wx_fmt=png)

可见 ReplicaSet 功能更齐全，所以在新版的 k8s 中，建议使用 ReplicaSet 作为副本控制器，不再使用 ReplicationController。

### 3. Deployment 部署对象

#### 3.1 滚动更新

ReplicaSet 副本控制器可以永久保持 pod 副本的数量。但是项目的需求在不断的迭代、更新，项目在不断发版。那如何做到服务更新？难道把服务停掉再把新版本部署上去吗？当然不是，答案是用滚动更新。就是重新创建一个 pod (v2 版本) 来代替 之前的 pod (v1 版本)。

![](https://mmbiz.qpic.cn/mmbiz_png/yNKv1P4Q9eXfK9Siav1Kuibwkj7JqwIRFVxIkks7WBttrdXOaozLyYia3uRqccXsXKXSKvT4P1icia968b7M8OozuVA/640?wx_fmt=png)

那是如何滚动更新的呢？涉及到下面要讲到的部署模型。

#### 3.2 部署模型

单独的 ReplicaSet 是不支持滚动更新的，Deployment 对象支持滚动更新，通常和 ReplicaSet 一起使用。

需要滚动更新时的步骤：

1.  Deployment 建立新的 Replicaset
    
2.  Replicaset 重新建立新的 pod
    

所以它们之间是有层次关系的，Deployment 管 Replicaset，Replicaset 维护 pod。在更新时删除的是旧的 pod，老版本的 ReplicaSet 是不会删除的，所以在需要时还可以回退以前的状态。

![](https://mmbiz.qpic.cn/mmbiz_png/yNKv1P4Q9eXfK9Siav1Kuibwkj7JqwIRFVWEIUHmKZxH7coezHJpib0ibR6VDhVsrhGGst7rgLunjt7YKyI4AanIibg/640?wx_fmt=png)

### 4. StatefulSet 部署有状态服务

#### 4.1 引入定义

思考：如果 MySQL(有状态服务) 使用容器化部署，会存在什么问题？

1.  容器都是有生命周期的，一旦宕机数据就很可能丢失
    
2.  pod 也有生命周期的，用 pod 部署时把 pod 集群副本重启以后也可能会出现数据丢失
    

因此对 k8s 来说，不能使用 Deployment 部署有状态的服务。通常情况下，Deployment 被用来部署无状态服务。

然后 StatefulSet 就是为了解决有状态服务使用容器化部署的一个问题。

#### 4.2 如何理解状态服务

*   有状态服务
    

*   有实时的数据需要存储
    
*   在有状态服务集群中，如果把某一个服务抽离出来，一段时间后再加入回集群网络，此后集群网络会无法使用
    

*   无状态服务
    

*   没有实时的数据需要存储
    
*   在无状态服务集群中，如果把某一个服务抽离出去，一段时间后再加入回集群网络，对集群服务无任何影响，因为它们不需要做交互，不需要数据同步等等。
    

#### 4.3 部署模型

StatefulSet 的部署模型和 Deployment 的很相似。

比如下图，借助 PVC(与存储有关) 文件系统来存储的实时数据，因此下图就是一个有状态服务的部署。

在 pod 宕机之后重新建立 pod 时，StatefulSet 通过保证 hostname 不发生变化来保证数据不丢失。因此 pod 就可以通过 hostname 来关联 (找到) 之前存储的数据。

![](https://mmbiz.qpic.cn/mmbiz_png/yNKv1P4Q9eXfK9Siav1Kuibwkj7JqwIRFVVTuIVQK3JLI30cO9BkTWtRYP7bRjmK4TwvsGHyuVhZjMKNIspibynibA/640?wx_fmt=png)

原文链接：https://blog.csdn.net/qq_43280818/article/details/106910187