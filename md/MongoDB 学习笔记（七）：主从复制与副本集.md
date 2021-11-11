> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/chasewade/archive/2013/10/18/3375954.html)

**一、主从复制**

1、主从复制是一个简单的数据库同步备份的集群技术，如下图：要明确的知道主服务器与从服务器，且从服务器要明确的知道主服务器的存在。

![](http://img-blog.csdn.net/20131015205827593?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2R1MDk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2、在 MongoDB 中在启动数据库服务时，可以用 master 参数来指定主服务器，如下图：bind_ip 是主数据库所在服务器 IP

![](http://img-blog.csdn.net/20131015210702734?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2R1MDk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

而用 slave 参数可以指定从服务器，如下图：source 参数用于指定主服务器

![](http://img-blog.csdn.net/20131015211217546?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2R1MDk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

以上两个数据库的关系如下图：这样过后，在主数据库中的操作就会立马在从数据库中进行复制。

![](http://img-blog.csdn.net/20131015211824671?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2R1MDk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

3、常用配置参数

*   --only：用于配置从服务器，指定复制某个数据库。默认是复制全部数据库。
*   --slavedelay：用于配置从服务器，指定从主服务器同步数据的延迟（单位是秒）。
*   --fastsync：用于配置从服务器，把内存中的数据马上写回主服务器数据库，然后从服务器进行复制。
*   --autoresync：用于配置从服务器，如果从服务器是在主服务器运行一端时间之后才挂上来的，那么指定了此项参数后，从服务器会同步挂接上来之前那段时间主服务器中的数据；如果没有此参数，那么只会从当前挂接时间进行主从复制。
*   --oplogSize：用于配置主服务器，配置主服务器的日志大小。日志会把主服务器的操作都记录下来，从服务器从主服务器的日志中读取数据，然后在自己的数据库中操作一次来完成主从复制。主服务器数据库操作记录存储在 local 数据库的 oplog.$main 集合中，它的每一个文档都会保存一个服务器的操作。

4、利用 shell 动态添加和删除从服务器

如下图所示，这是在上面那台从服务器中 shell 界面的操作结果，可以得知从服务器关于主服务器的信息全部保存在从服务器的 local 数据库的 sources 集合中。所以只要对该集合进行操作就可以动态的配置从服务器。

![](http://img-blog.csdn.net/20131015214547296?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2R1MDk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

删除这台从服务器配置的主服务器，如下图：

![](http://img-blog.csdn.net/20131015214828765?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2R1MDk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

为从服务器挂接主服务器，如下图：

![](http://img-blog.csdn.net/20131015215011921?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2R1MDk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**二、副本集**

1、如下图所示有一个数据库集群，集群中有三台数据库服务器，一台活跃服务器和两台备份服务器。当活跃服务器 A 发生故障时，会根据权重算法从备份服 务器 B 和 C 中选出 B 作为新的活跃服务器，而当 A 恢复时当成备份服务器，继续加入到整个数据库集群中工作，这就是 MongoDB 的副本集。

![](http://img-blog.csdn.net/20131015220421734?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2R1MDk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2、配置一个副本集

首先采用如下图的三个配置配置启动三台数据库服务器，三台服务器相互指向下一个服务器形成一个环。

![](http://img-blog.csdn.net/20131015222539906?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2R1MDk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

然后就需要初始化副本集。进入到这三台数据库服务器中任何一个的 admin 数据库，执行如下图的操作来初始化副本集：这里没有设置这三台服务器的权重，MonggoDB 推选活跃服务器的策略是随机的。

![](http://img-blog.csdn.net/20131015224216531?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2R1MDk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分别进入到这三台服务器的 shell 界面，如下：可以发现端口号为 3333 的服务器被推选成了活跃服务器，而其它两台就是备份服务器

![](http://img-blog.csdn.net/20131015224959734?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2R1MDk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

在活跃服务器 shell 中可以使用 “rs.status()” 来查看副本集的状态。

此时如果把端口 3333 的活跃服务器关掉后，如下图所示：端口为 2222 的服务器就成为了活跃服务器。如果又把端口为 3333 的服务器启动，可以发现端口 3333 的服务器就成为了备份服务器。

![](http://img-blog.csdn.net/20131015230200859?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2R1MDk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

3、节点

*   standard：常规节点，参与投票，有可能成为活跃节点。如上的三台服务器都没有设置此参数，默认就是常规节点。
*   passive：副本节点，参与投票，但不能成为活跃节点。
*   arbiter：仲裁节点，只是参与投票，不复制节点，也不能成为活跃节点。

4、高级参数

*   priority：设置权重。可设置 0 到 1000 之间，0 代表的是副本节点，1 到 1000 是常规节点。
*   arbiterOnly：true 表示设置仲裁节点。
*   用法：在前面的初始化副本集中，只设置了_id 和 host 两个 key，此时就可以加上 priority 与 arbiterOnly 等 key 来初始化副本集。

5、优先级

如下图所示，本来 A 是活跃节点，B 和 C 为副本集，B 节点是在 1 秒之前进行复制更新的，C 节点是在 5 秒之前进行复制更新的，A 宕机之后会自动选择 B 为活跃节点（复制更新时间离宕机时间最近）。这种优先级优先于权重，会先考虑时间的优先级。

![](http://img-blog.csdn.net/20131015231935750?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2R1MDk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

6、读写分离操作

注意：默认情况下非活跃服务器（副本节点）是不能进行数据库读操作的。如下图：

![](http://img-blog.csdn.net/20131015225452406?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2R1MDk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

如果要设置副本节点也可以进行读操作，那么可以设置 slaveOkay 参数为 true。但是此属性在 shell 中无法完成，这个特性是被写到 MongoDB 的高级驱动程序中的。