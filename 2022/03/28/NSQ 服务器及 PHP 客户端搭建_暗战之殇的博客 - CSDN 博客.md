> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_33649725/article/details/76977499)

NSQ 服务器及 PHP 客户端搭建
==================

在对比了市面上多款[消息队列](https://so.csdn.net/so/search?q=%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97&spm=1001.2101.3001.7020)之后，基于我们研发团队的现状，我决定选用 nsq 作为我们的消息队列。其最吸引我的特性倒并非是高并发，水平扩展；而是支持 HTTP 请求，使用简单。然而 NSQ 的文档不够详尽，社区不够活跃的问题真让我耗费了很长时间才搞定。

GOLANG 安装：
----------

> NSQ 基于 [GO 语言](https://so.csdn.net/so/search?q=GO%E8%AF%AD%E8%A8%80&spm=1001.2101.3001.7020)，先安装 Go, 问题不大, 注意与 NSQ 要求的版本相适应，我选用的 go1.9

```
下载标准安装包

```

```
    https://golang.org/dl/
```

解压到 / usr/local 目录

```
$ sudo tar -xzvf go1.5.2.linux-amd64.tar.gz /usr/local
```

在 $HOME 目录下创建文件夹 gopath

```
$ vi /etc/profile 
```

在 /etc/profile 添加如下内容

```
export GOPATH=$HOME/gopath
export PATH=$PATH:/usr/local/go/bin:$GOPATH/bin
```

然后我们刷新环境变量

```
$ source /etc/profile
```

最后我们验证一下是否安装成功

```
$ go version
```

NSQ 安装：
-------

直接下载官网安装包

```
$ wget https://s3.amazonaws.com/bitly-downloads/nsq/nsq-1.0.0-compat.linux-amd64.go1.8.tar.gz
$ sudo tar -xzvf nsq-1.0.0-compat.linux-amd64.go1.8.tar.gz
$ sudo mv nsq-1.0.0-compat.linux-amd64.go1.8 /usr/local/nsq
$ cd /usr/local/nsq
```

NSQ 启动：
-------

NSQ 需要主要启动三个模块 nsqd nsqlokkupd nsqadmin

```
$ nohup ./nsqlookupd &
```

```
$ nohup ./nsqd --lookupd-tcp-address=127.0.0.1:4160 --broadcast-address=139.196.205.* &
```

> 这一步就是官方文档坑爹的地方，不加上–broadcast-address=139.196.205.* 客户端是无法连接的 后面的 IP 是你的实际的服务器 IP

```
$ nohup ./nsqadmin --lookupd-http-address=127.0.0.1:4161 &
```

此时打开 139.196.205.*:4161 会看到 NSQadmin 的 web 界面

创建一个 topic

```
$ curl -d 'hello world 1' 'http://127.0.0.1:4151/pub?topic=test'
```

持久化

```
$ nohup ./nsq_to_file --topic=test --output-dir=/tmp --lookupd-http-address=127.0.0.1:4161 &
```

持续推送消息

```
$ curl -d 'hello world 2' 'http://127.0.0.1:4151/pub?topic=test'
$ curl -d 'hello world 3' 'http://127.0.0.1:4151/pub?topic=test'
```