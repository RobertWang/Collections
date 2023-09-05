> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/vesr8_9g3f1B4OA9a9myQA)

**一、简介**

       单个 Redis 的读写能力是有限的，并且存在不稳定性。当唯一的 Redis 服务宕机了，就没有可用的 Redis 服务了，另外当硬件出现问题，单机的数据便无法恢复。Redis 集群的出现解决了单节点故障的问题，同时强化了 Redis 的读写能力 (负载均衡)。

      主从复制：指一台 Redis 服务器的数据复制到其他的 Redis 服务器，前者为主节点 (master), 后者为从节点 (slave)。数据的复制是单向的，只能由主节点到从节点。默认情况下，每台 Redis 服务器都是主节点，一个主节点可以有 0 或者多个从节点，但从节点只能有一个主节点。

      主节点提供读写服务，从节点提供读服务，多个从节点分担读负载，提高了 redis 的并发量。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm450er8SlickycQHeqEEHEPpBv20CDibAydC4xdGCoxYy84rClQ6LAib62r0ASLeJomia6k6U3b8ETWL1A/640?wx_fmt=png)

**二、搭集群建步骤**  

**特别说明**：window 下 6.2 的安装包在文章最后有获取方式，大家如果需要，可以自行获取，linux 的直接去官网下载即可。(官网访问地址：https://redis.io/download/#redis-downloads)

**2.1、环境准备**

<table width="-123" interlaced="enabled"><tbody><tr data-style=""><td width="123" valign="top">IP<br></td><td width="123" valign="top">端口<br></td><td width="123" valign="top">角色<br></td></tr><tr data-style=""><td width="123" valign="top">127.0.0.1<br></td><td width="123" valign="top" class="">7001<br></td><td width="123" valign="top">Master<br></td></tr><tr data-style=""><td width="123" valign="top">127.0.0.1<br></td><td width="123" valign="top">7002<br></td><td width="123" valign="top">slave</td></tr><tr data-style=""><td width="123" valign="top">127.0.0.1</td><td width="123" valign="top">7007<br></td><td width="123" valign="top">slave</td></tr></tbody></table>

**2.2、创建目录**

```
# 进入/tmp目录
cd /tmp
# 创建目录
mkdir 7001 7002 7003

```

如图：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm450er8SlickycQHeqEEHEPpBibbqXLnLG6B9Ehicp2ch8jBzvdF2jUZmibyaqYXfzAjEvRSnnpGuHurAQ/640?wx_fmt=png)

**2.3、配置文件修改**

**拷贝文件：  
**

```
# 方式一：逐个拷贝
cp redis-6.2.4/redis.conf 7001
cp redis-6.2.4/redis.conf 7002
cp redis-6.2.4/redis.conf 7003
# 方式二：管道组合命令，一键拷贝
echo 7001 7002 7003 | xargs -t -n 1 cp redis-6.2.4/redis.conf

```

**修改参数：**

```
#修改每个文件夹内的配置文件，
#将端口分别修改为7001、7002、7003，将rdb文件保存位置都修改为
# 自己所在目录（在/tmp目录执行下列命令
sed -i -e 's/6379/7001/g' -e 's/dir .\//dir \/tmp\/7001\//g' 7001/redis.conf
sed -i -e 's/6379/7002/g' -e 's/dir .\//dir \/tmp\/7002\//g' 7002/redis.conf
sed -i -e 's/6379/7003/g' -e 's/dir .\//dir \/tmp\/7003\//g' 7003/redis.conf
# 修改每个实例的声明IP
# redis实例的声明  IPreplica-announce-ip 127.0.0.1
# 1、逐个修改
sed -i '1a replica-announce-ip 127.0.0.1' 7001/redis.conf
sed -i ' 1a replica-announce-ip 127.0.0.1' 7002/redis.conf
sed -i '1a replica-announce-ip 127.0.0.1' 7003/redis.conf# 
# 2、或者一键修改
printf '%s\n' 7001 7002 7003 | xargs -I{} -t sed -i '1a replica-announce-ip 127.0.0.1' {}/redis.conf
# 开启主从关系
# 可以使用replicaof（5.0版本之后的）或者slaveof(5.0版本之前的)命令
# slaveof/replicaof <masterip> <masterport>
# 若选择7001为主机,其他为从机，则分别在7002和7003上配置
replicaof 127.0.0.1 7001

```

**启动：**

```
# 第1个
redis-server 7001/redis.conf
# 第2个
redis-server 7002/redis.conf
# 第3个
redis-server 7003/redis.conf

```

**查看集群状态：**info replication

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm450er8SlickycQHeqEEHEPpBVpQmaOZTtLAbGETrsgNSd8TJw32QbIutjPIamtaVukZsET36U5icQJg/640?wx_fmt=png)

**三、数据同步原理**

**3.1、全量同步**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm450er8SlickycQHeqEEHEPpBDO7sb0sicEia7ibaicXuuFKDd1dHDicKl1nFVfgNYibbEALY1SrGCgaIpticA/640?wx_fmt=png)

       当从节点 (slave) 启动成功并连接到主节点 (master) 后，会向主节点发送 sync 命令，请求数据同步。主节点接收到命令后会启动一个子进程，将数据写入数据文件中(RDB 操作)，待数据写入完毕后，会将数据文件传送给从节点，从节点接收到文件后保存到磁盘上，并将数据加载到内存中，完成一次全量复制。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm450er8SlickycQHeqEEHEPpBrSCMZhiaffTcSkAzbhG8gGeNaqMsBDNVaNlb0xRTDOb6nweWQibf49uQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm450er8SlickycQHeqEEHEPpB2wFDodcvm9RRAJDnHGAGNRK31ZvKxsgx2Y2h40ibehOUb3eLGCF5tqg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm450er8SlickycQHeqEEHEPpBDTjPNibdwM5uTqbJQpHRMoiaL8QnPTmf40jgUfHRA9n7oGA59urjtb1g/640?wx_fmt=png)

**3.2、增量同步**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm450er8SlickycQHeqEEHEPpBSiaVTUBUZrKE5Y3NXiawCU3lB1FclBWcZBYWBcI3fxSaaVOG21alRVcg/640?wx_fmt=png)

       在完成一次全量复制后，如果主节点收到写操作的命令时，会以异步方式将命令复制给从节点 (在全量复制期间，如果主节点接收到了新的写操作命令，会先存入缓存，待从节点完成数据同步后，将新的写操作命令复制给从节点)。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm450er8SlickycQHeqEEHEPpBDQdpd1T2fhv0GgbgURnmNzQTfSwNGNhfllAviab4fOuOfmxszyjwia6A/640?wx_fmt=png)

**四、总结**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm450er8SlickycQHeqEEHEPpB3mcOcVI5nzsniaEBRFPibkSsUpSJmywOtIT0qlDEreOBLyuH5OTgMcUQ/640?wx_fmt=png)

**全量同步和增量同步的区别：**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm450er8SlickycQHeqEEHEPpBOUBmuNlyLJSj82dvDneibTho839R2kMBrjiauzRNfKweqNiaMytjgYHTg/640?wx_fmt=png)

**优缺点：**

**优点：**

    1、实现了读写分离，缓解了主节点读操作的压力，提高了可用性，解决了单机故障问题;

    2、主从复制期间主节点与从节点都是非阻塞的方式，仍然可用。

**缺点：**

    1、如果主节点发生宕机，需要手动切换主节点；

    2、如果 RDB 文件过大，同步过程比较耗时。

**五、Redis 的 window 版本获****取方式**

**参考网站：  
**

https://www.cnblogs.com/shenStudy/p/16859463.html