> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NTY1MjY0MQ==&mid=2650865936&idx=2&sn=40440c711a6fabf38c449c76619bed3f&chksm=bd00945e8a771d4850920ba60a9082219f0ccfbc8dcdc9c5e2c93bba38e1a3f2db250bd2b5ee&mpshare=1&scene=1&srcid=1009C48EwqYIs57S5c9DQxEG&sharer_sharetime=1665290337589&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

_来源丨网管叨 bi 叨（ID：kevin_tech）_

已获得原公众号授权转载

想要让 K8s 听从我们的调遣，我们就得通过`kubectl`给它发送指示才行，可是这么多操作我们全看一遍真的是挺耗费脑力的，更别提记下来了。

所以今天给大家总结了几个最常用也最实用的命令操作，以后实在忘了，翻开看看马上就能捡起来。嗯，平时用的 go mod 和 maven 那些命令，我就能记住常用的那两三个，解决依赖问题的时候每次都上搜索引擎，时候长了就搞了个笔记，用到了翻一翻。

感觉没了这些笔记和搜索引擎我已经不会干活了...

好了，闲话不多说，马上开始。

Kubectl 的语法结构
-------------

首先我们来理解一下 K8s 的 CLI 命令工具`kubectl` 它的语法结构是什么样的，不然就真得每个操作都靠抄了。

所有的 K8s 命令操作在 CLI 中都使用以下结构：

这个命令语法中每个部分的顺序不能调换，否则 K8s 就不理解我们要干什么了。

### command

`command`部分描述了要执行的操作类型，主要的操作类型有：

*   `create` 从文件或命令行输入提供的配置，生成资源对象。
    
*   `describe` 检索资源对象的详细信息
    
*   `get` 获取各种资源在集群里的信息
    
*   `delete` 从集群中删除需要擦除的资源对象
    
*   `apply` 搭配声明文件使用，把资源对象的定义提交给集群，由集群进行应用。
    

### TYPE

命令操作中的 `TYPE` 部分用于制定 `kubectl` 发起的操作，所针对的资源类型。常用的资源类型有`pod`，`service`，`deployment`， `statefulset` ，`node`这些。

### NAME

`NAME` 部分区分大小写，是 K8s 里资源对象的唯一标识，用于制定指定`TYPE`部分指明的相关资源的名称。将名称附加到命令操作上会将该命令操作只对该资源对象游泳。

### flags

`flags` 部分表示对特定资源的特殊选项或请求。它们是用作覆盖默认值或环境变量。

比如任何`kubectl`发起的命令操作，都是在`default` 这个命名空间下起作用的，想要作用到其他命名空间，可以通过在`flags` 部分用`-n`选项指定命名空间，例如：

```
kubectl get pod -n web


```

就是查看 web 命名空间下有哪些 pod 资源。

好了下面列举几个非常实用的命令操作，建议收藏。

实用命令推荐
------

### 1. 查看所有命名空间下的资源

命名空间在 K8s 中非常重要。它们是一种在集群中隔离某些资源组，然后相应地管理它们的机制。命名空间提供的可见性隔离在 K8s 中也起着至关重要的作用。

默认我们所有命令生效的命名空间都是 `default` 。

```
kubectl get pods


```

那么有时候在查问题，看集群大体布局的时候，往往需要看某类资源在集群中整体的情况，这就需要能查出所有命名空间下的信息，这个时候我们可以在`flags` 部分使用`--all-namespaces` 选项：

```
kubectl get pods --all-namespaces


```

### 2. 查询命名空间下所有在运行的 pod

```
kubectl get pods --field-selector=status.phase=Running


```

这个就不多解释了，其实擅用`—field-selector` 能根据资源的属性查出各种在某个状态、拥有某个属性值的资源。

那怎么知道某个类型的资源对象有哪些属性值呢，毕竟 K8s 资源的类型十几种，每种的属性就更多了，这个时候就可以看下个命令。

### 3. 查询资源当下在集群中的属性

```
kubectl get pod pod-name -o=yaml


```

上面这个命令就能把指定名称的 pod 对象在集群中当前拥有的属性以 `YAML`格式的形式全打印出来，也支持 JSON 格式。

这里例子里 TYPE 部分用的是 pod，可以替换成任何 K8s 支持的资源类型，查看他们的属性。

### 4. 提交资源给集群应用，并记录版本

提交资源定义，让集群进行应用调度，我们统一用的是

```
kubectl apply -f resources.yaml


```

不过，如果你想用 K8s 中 -- Deployment 资源的回滚能力的话，还得让 K8s 记住每个版本都提交了什么，这个功能可以通过`--record`选项开启。

```
kubectl apply -f resources.yaml --record


```

### 5. 查看资源对象的事件信息

有的时候，Pod 挂了，一直停在挂起状态，这个时候就需要看看它经理过哪些事件了，好做排查。

```
kubectl describe pod pod-name 


```

时候回打印出来这个 Pod 经历过的所有事件信息

```
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Warning  Failed     20s (x4 over 2m4s)   kubelet            Failed to pull image "xxx
 ": rpc error: code = Unknown desc = Error response from daemon: manifest for xxx not found: manifest unknown: manifest unknown
  Warning  Failed     20s (x4 over 2m4s)   kubelet            Error: ErrImagePull
  Normal   BackOff    4s (x5 over 2m4s)    kubelet            Back-off pulling image "xxx"
  Warning  Failed     4s (x5 over 2m4s)    kubelet            Error: ImagePullBackOff


```

同样除了 Pod 外，用 describe 还能看其他资源的事件。

### 6. 查看容器日志

我们所有的应用在 K8s 运行前都是先封装在容器里，再以 Pod 为单位调度到集群上的，那么一旦不符合预期，有错的时候，肯定第一时间想到的是看日志，这时候就需要用到下面这个命令：

```
kubectl  logs <podname> -n <namespace>


```

如果恰巧这个 Pod 被重启了，查不出来任何东西，可以通过增加 — previous 参数选项，查看之前容器的日志。

```
kubectl logs <podname> --previous


```

总结
--

今天给大家总结了几个使用频率高的 K8s 命令操作，其实最主要的还是第一部分讲的命令语法结构，掌握了这个结构，我们只需要把各个资源类型、操作类型、资源名称这些变量填空到结构里就能指示 K8s 完成我们想要的操作啦。

**<END>**