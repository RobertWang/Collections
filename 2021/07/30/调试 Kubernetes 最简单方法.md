> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI5ODQ2MzI3NQ==&mid=2247501879&idx=2&sn=4dde058f6b2ae0443b06923696ecac93&chksm=eca7f173dbd0786597f76c4eed8b4179b448a81ec37dbfe263ee165b456c96d87b48f1a87663&mpshare=1&scene=1&srcid=0727Vt5c1KV9AFFUnuZ3BjQg&sharer_sharetime=1627375469608&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> 少为人知的调试方法！

调试容器化工作负载和 Pod 是每位使用 Kubernetes 的开发人员和 DevOps 工程师的日常任务。通常情况下，我们简单地使用 `kubectl logs` 或者 `kubectl describe pod` 便足以找到问题所在，但有时候，一些问题会特别难查。这种情况下，大家可能会尝试使用 `kubectl exec`，但有时候这样也还不行，因为 Distroless 等容器甚至不允许通过 SSH 进入 shell。那么，如果以上所有方法都失败了，我们要怎么办？

**更好的方法**

其实我们只需要使用更合适的工具。**如果在 Kubernetes 上调试工作负载，那么合适的工具就是 `kubectl debug`。**这是不久前添加的一个新命令（v1.18），允许调试正在运行的 pod。它会将名为 EphemeralContainer（临时容器）的特殊容器注入到问题 Pod 中，让我们查看并排除故障。`kubectl debug` 看起来非常不错，但要使用它需要临时容器，临时容器到底是什么？

**临时容器其实是 Pod 中的子资源，类似普通 `container`。但与普通容器不同的是，临时容器不用于构建应用程序，而是用于检查。**我们不会在创建 Pod 时定义它们，而使用特殊的 API 将其注入到运的行 Pod 中，来运行命令并检查 Pod 环境。除了这些不同，临时容器还缺少一些基本容器的字段，例如 `ports`、`resources`。

那么我们为什么不直接使用基本容器？这是因为我们不能向 Pod 添加基本容器，它们应该是一次性的（需要随时删除或重新创建），这会导致难以重现问题 Pod 的错误，排除故障也会很麻烦。这就是将临时容器添加到 API 的原因——它们允许我们将临时容器添加到现有 Pod，从而检查正在运行的 Pod。

虽然临时容器是作为 Kubernetes 核心的 Pod 规范的一部分，但很多人可能还没有听说过。这是因为临时容器处于早期 Alpha 阶段，这意味着默认情况下不启用。Alpha 阶段的资源和功能可能会出现重大变化，或者在 Kubernetes 的某个未来版本中被完全删除。**因此，要使用它们必须在 kubelet 中使用 Feature Gate（功能门）显式启用。**

**Configuring Feature Gates**

**现在如果确定要试用 `kubectl debug`，那么如何启用临时容器的功能门？这取决于集群设置。**例如，现在使用 kubeadm 启动创建集群，那么可以使用以下集群配置来启用临时容器：

```
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.20.2
apiServer:
  extraArgs:
    feature-gates: EphemeralContainers=true
```

在以下示例中，为了简单和测试目的，我们使用 KinD（Docker 中的 Kubernetes）集群，这允许我们指定要启用的功能门。创建我们的测试集群：

```
# File: config.yaml
# Run:  kind create cluster --config ./config.yaml --name kind --image=kindest/node:v1.20.2
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
featureGates:
  EphemeralContainers: true
nodes:
- role: control-plane
```

随着集群的运行，我们需要验证其有效性。最简单方法是检查 Pod API，它现在应该包含临时容器部分以及通常容器：

```
~ $ kubectl explain pod.spec.ephemeralContainers
KIND:     Pod
VERSION:  v1

RESOURCE: ephemeralContainers <[]Object>

DESCRIPTION:
     List of ephemeral containers run in this pod....
...
```

现在都有了，可以开始使用 `kubectl debug`。从简单的例子开始：

```
~ $ kubectl run some-app --image=k8s.gcr.io/pause:3.1 --restart=Never
~ $ kubectl debug -it some-app --image=busybox --target=some-app
Defaulting debug container name to debugger-tfqvh.
If you don't see a command prompt, try pressing enter.
/ #
# From other terminal...
~ $ kubectl describe pod some-app
...
Containers:
  some-app:
    Container ID:   containerd://60cc537eee843cb38a1ba295baaa172db8344eea59de4d75311400436d4a5083
    Image:          k8s.gcr.io/pause:3.1
    Image ID:       k8s.gcr.io/pause@sha256:f78411e19d84a252e53bff71a4407a5686c46983a2c2eeed83929b888179acea
...
Ephemeral Containers:
  debugger-tfqvh:
    Container ID:   containerd://12efbbf2e46bb523ae0546b2369801b51a61e1367dda839ce0e02f0e5c1a49d6
    Image:          busybox
    Image ID:       docker.io/library/busybox@sha256:ce2360d5189a033012fbad1635e037be86f23b65cfd676b436d0931af390a2ac
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 15 Mar 2021 20:33:51 +0100
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:         <none>
```

我们首先启动一个名为 `some-app` 的 Pod 来进行 “调试”。然后针对这个 Pod 运行 `kubectl debug`，指定 `busybox` 为临时容器的镜像，并作为原始容器的目标。此外，还需要包括 `-it` 参数，以便我们立即附加到容器获得 shell 会话。

在上面的代码中可以看到，如果我们在 Pod 上运行 kubectl debug 后对其进行描述，那么它的描述将包括具有之前指定为命令选项值的临时容器部分。

**Process Namespace Sharing**

`kubectl debug` 是非常强大的工具，但有时向 Pod 添加一个容器还不足以获取 Pod 的另一个容器中运行的应用程序相关信息。当故障容器不包括必要的调试工具甚至 shell 时，可能就是这种情况。**在这种情况下，我们可以使用 Process Sharing（进程共享）来使用注入的临时容器检查 Pod 的原有容器。**

进程共享的一个问题是它不能应用于现有的 Pod，因此我们必须创建一个新 Pod，将其 `spec.shareProcessNamespace` 设置为 `true`，并将一个临时容器注入其中。这样有点麻烦，尤其是需要调试多个 Pod 或容器，亦或者需要重复执行该操作时。幸运的是，`kubectl debug` 可以使用 `--share-processes` 做到：

```
~ $ kubectl run some-app --image=nginx --restart=Never
~ $ kubectl debug -it some-app --image=busybox --share-processes --copy-to=some-app-debug
Defaulting debug container name to debugger-tkwst.
If you don't see a command prompt, try pressing enter.
/ # ps ax
PID   USER     TIME  COMMAND
    1 root      0:00 /pause
    8 root      0:00 nginx: master process nginx -g daemon off;
   38 101       0:00 nginx: worker process
   39 root      0:00 sh
   46 root      0:00 ps ax
~ $ cat /proc/8/root/etc/nginx/conf.d/default.conf 
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;
...
```

上面的代码表明，通过进程共享，我们可以看到 Pod 中另一个容器内的所有内容，包括其进程和文件，这对于调试来说非常方便。另外，除了 `--share-processes` 还包括了 `--copy-to=new-pod-name`，这是因为我们需要创建一个新的 Pod，其名称由该 flag 指定。如果我们从另一个终端列出正在运行的 Pod，我们将看到以下内容：

```
# From other terminal:
~ $ kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
some-app         1/1     Running   0          23h
some-app-debug   2/2     Running   0          20s
```

这就是我们在原始应用程序 Pod 上的新调试 Pod。与原始容器相比，它有 2 个容器，因为它还包括临时容器。此外，如果想在任何时候验证 Pod 中是否允许进程共享，那么可以运行：

```
~ $ kubectl get pod some-app-debug -o json  | jq .spec.shareProcessNamespace
true
```

**好好使用**

**既然我们已经启用了功能并且知道命令是如何工作的，那就试着使用它并调试一些应用程序。**想象这样一个场景——我们有一个问题应用程序，我们需要在它的容器中对网络相关的问题进行故障排除。该应用程序没有我们可以使用的必要的网络 CLI 工具。为了解决这个问题，我们通过以下方式使用 `kubectl debug`：  

```
~ $ kubectl run distroless-python --image=martinheinz/distroless-python --restart=Never
~ $ kubectl exec -it distroless-python -- /bin/sh
# id
/bin/sh: 1: id: not found
# ls
/bin/sh: 2: ls: not found
# env
/bin/sh: 3: env: not found
#
...

kubectl debug -it distroless-python --image=praqma/network-multitool --target=distroless-python -- sh
Defaulting debug container name to debugger-rvtd4.
If you don't see a command prompt, try pressing enter.
/ # ping localhost
PING localhost(localhost (::1)) 56 data bytes
64 bytes from localhost (::1): icmp_seq=1 ttl=64 time=0.025 ms
64 bytes from localhost (::1): icmp_seq=2 ttl=64 time=0.044 ms
64 bytes from localhost (::1): icmp_seq=3 ttl=64 time=0.027 ms
```

在启动一个 Pod 之后，我们首先尝试将 shell 会话放入它的容器中，这看起来有效，但是实际上我们尝试运行一些基本命令时，将看到那里什么都没有。所以，我们要使用 `praqma/network-multitool` 将临时容器注入到 Pod 中，该镜像包含了 `curl`、`ping`、`telnet` 等工具，现在我们可以进行所有必要的故障排除。

在上面的例子中，我们进入 Pod 的另一个容器中就足够了。但有时可能需要直接查看有问题的容器。这种情况下，我们可以像这样使用进程共享：

```
~ $ kubectl run distroless-python --image=martinheinz/distroless-python --restart=Never
~ $ kubectl debug -it distroless-python --image=busybox --share-processes --copy-to=distroless-python-debug
Defaulting debug container name to debugger-l692h.
If you don't see a command prompt, try pressing enter.
/ # ps ax
PID   USER     TIME  COMMAND
    1 root      0:00 /pause
    8 root      0:00 /usr/bin/python3.5 sleep.py  # Original container is just sleeping forever
   14 root      0:00 sh
   20 root      0:00 ps ax
/ # cat /proc/8/root/app/sleep.py 
import time
print("sleeping for 1 hour")
time.sleep(3600)
```

在这里，我们再次运行使用 Distroless 镜像的容器。我们无法在它的 shell 中做任何事情。我们运行 `kubectl debug` 以及 `--share-processes --copy-to=...`，它创建了一个新的 Pod，带有额外的临时容器，可以访问所有进程。当我们列出正在运行的进程时，能看到应用程序容器的进程有 PID 8，可以用它来探索文件和环境。为此，我们需要通过 `/proc/<PID>/...` 目录，这个例子中是 `/proc/8/root/app/...`。

另一种常见情况是应用程序在容器启动时不断崩溃，这让调试非常困难，因为没有足够的时间将 shell 会话导入容器并运行故障排除命令。**在这种情况下，解决方案是创建具有不同入口点、命令的容器，这可以阻止应用程序立即崩溃并允许我们调试：**

```
~ $ kubectl get pods
NAME                READY   STATUS             RESTARTS   AGE
crashing-app        0/1     CrashLoopBackOff   1          8s

~ $ kubectl debug crashing-app -it --copy-to=crashing-app-debug --container=crashing-app -- sh
If you don't see a command prompt, try pressing enter.
# id
uid=0(root) gid=0(root) groups=0(root)
#
...
# From another terminal
~ $ kubectl get pods
NAME                READY   STATUS             RESTARTS   AGE
crashing-app        0/1     CrashLoopBackOff   3          2m7s
crashing-app-debug  1/1     Running            0          16s
```

**调试集群节点**

本文主要关注 Pod 及其容器的调试，但任何集群管理员都知道常常需要调试的是节点而不是 Pod。幸运的是，`kubectl debug` 允许通过创建 Pod 来调试节点，该 Pod 将在指定节点上运行，节点的根文件系统安装在 `/root` 目录中。我们甚至可以用 `chroot` 访问主机二进制文件，这本质上充当了节点的 SSH 连接：

```
~ $ kubectl get nodes
NAME                 STATUS   ROLES                  AGE   VERSION
kind-control-plane   Ready    control-plane,master   25h   v1.20.2

~ $ kubectl debug node/kind-control-plane -it --image=ubuntu
Creating debugging pod node-debugger-kind-control-plane-hvljt with container debugger on node kind-control-plane.
If you don't see a command prompt, try pressing enter.
root@kind-control-plane:/# chroot /host
# head kind/kubeadm.conf
apiServer:
  certSANs:
  - localhost
  - 127.0.0.1
  extraArgs:
    feature-gates: EphemeralContainers=true
    runtime-config: ""
apiVersion: kubeadm.k8s.io/v1beta2
clusterName: kind
controlPlaneEndpoint: kind-control-plane:6443
```

在上面的代码中，我们首先确定了想要调试的节点，然后使用 `node/...` 作为参数显式运行 `kubectl debug` 以访问我们集群的节点。在那之后，当连接到 Pod 后，我们使用 `chroot /host` 突破 `chroot`，并完全进入主机。最后，为了验证是否真的可以看到主机上的所有内容，我们了查看一部分的 `kubeadm.conf`，最终看到我们在文章开头配置的内容 `feature-gates: EphemeralContainers=true`。

**小结**

**能够快速有效地调试应用程序和服务可以节省大量时间，但更重要的是，它能极大地帮助解决那些如果不立即解决可能最终会花费大量资金的问题。**这就是为什么 kubectl debug 之类的工具能随意使用非常重要，即使它们尚未正式发布或默认启用。如果启用临时容器不是一种选择，那么尝试替代调试方法可能是一个好主意，例如使用包含故障排除工具的应用程序镜像的调试版本；或临时更改 Pod 的容器命令以阻止其崩溃。  

原文链接：https://towardsdatascience.com/the-easiest-way-to-debug-kubernetes-workloads-ff2ff5e3cc75