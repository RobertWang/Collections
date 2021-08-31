> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/wolf1105/article/details/116019044)

**一、安装 docker**
---------------

[root@~]# yum install -y yum-utils device-mapper-persistent-data lvm2      #安装依赖包

```
Installed:
  device-mapper-persistent-data.x86_64 0:0.8.5-3.el7_9.2                                     lvm2.x86_64 7:2.02.187-6.el7_9.4                                     yum-utils.noarch 0:1.1.31-54.el7_8                                    
 
Dependency Installed:
  device-mapper-event.x86_64 7:1.02.170-6.el7_9.4     device-mapper-event-libs.x86_64 7:1.02.170-6.el7_9.4     libaio.x86_64 0:0.3.109-13.el7     libxml2-python.x86_64 0:2.9.1-6.el7.5     lvm2-libs.x86_64 7:2.02.187-6.el7_9.4    
  python-chardet.noarch 0:2.2.1-3.el7                 python-kitchen.noarch 0:1.1.1-5.el7                     
 
Dependency Updated:
  device-mapper.x86_64 7:1.02.170-6.el7_9.4                                      device-mapper-libs.x86_64 7:1.02.170-6.el7_9.4                                      libxml2.x86_64 0:2.9.1-6.el7.5  
 
Complete!                         
```

  
[root@~]#   
[root@~]# yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo     # 添加 yum 资源包  
Loaded plugins: fastestmirror  
adding repo from: http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo  
grabbing file http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo  
repo saved to /etc/yum.repos.d/docker-ce.repo  
[root@~]#   
[root@~]# yum makecache  
[root@~]#    
[root@~]# yum list docker-ce.x86_64  --showduplicates | sort -r          #查看可以用版本  
[root@~]#   
[root@~]# yum install docker-ce                                                           # 这里选择安装最新版本

Installed:  
  docker-ce.x86_64 3:20.10.6-3.el7                                                                                                                                                                                                       

Dependency Installed:  
  audit-libs-python.x86_64 0:2.8.5-4.el7              checkpolicy.x86_64 0:2.5-8.el7                container-selinux.noarch 2:2.119.2-1.911c772.el7_8    containerd.io.x86_64 0:1.4.4-3.1.el7    docker-ce-cli.x86_64 1:20.10.6-3.el7     
  docker-ce-rootless-extras.x86_64 0:20.10.6-3.el7    docker-scan-plugin.x86_64 0:0.7.0-3.el7       fuse-overlayfs.x86_64 0:0.7.2-6.el7_8                 fuse3-libs.x86_64 0:3.6.1-4.el7         libcgroup.x86_64 0:0.41-21.el7           
  libsemanage-python.x86_64 0:2.5-14.el7              policycoreutils-python.x86_64 0:2.5-34.el7    python-IPy.noarch 0:0.75-6.el7                        setools-libs.x86_64 0:3.3.8-4.el7       slirp4netns.x86_64 0:0.4.3-4.el7_8     

Dependency Updated:  
  audit.x86_64 0:2.8.5-4.el7                                              audit-libs.x86_64 0:2.8.5-4.el7                                              policycoreutils.x86_64 0:2.5-34.el7                                             

Complete!  
[root@~]#   
[root@~]#   
[root@~]# docker --version                                                         #查看版本  
Docker version 20.10.6, build 370c289  
[root@~]# 

配置 docker 文件在宿主机的目录（适用与服务器数据盘不在 / 分区）

[root@~]#mkdir -p  /data/docker-data

[root@~]# cat <<EOF>>/etc/docker/daemon.json  
> {  
>     "data-root": "/data/docker-data"  
> }  
> EOF  
[root@~]#   
[root@~]# cat /etc/docker/daemon.json   
{  
    "data-root": "/data/docker-data"  
}

[root@~]#   
[root@~]# service docker start                                                     #启动 docker 服务  
Redirecting to /bin/systemctl start docker.service

**二、docker 安装 clickhouse**
--------------------------

[root@~]#  docker pull yandex/clickhouse-server:21.4.3.21                          # 拉取 clickhouse 镜像

[root@~]# docker images                                                                               # 查看已下载的镜像  
REPOSITORY                         TAG         IMAGE ID       CREATED      SIZE  
yandex/clickhouse-server   21.4.3.21   9aeb1b103595   9 days ago   557MB

[root@~]#  docker run -it -d --restart always --name=clickhouse-server  -v /etc/localtime:/etc/localtime -p 8123:8123 -p 9000:9000 -p 9009:9009  yandex/clickhouse-server:21.4.3.21                  #生成容器

f7c488c3e31a66c96dc8fbc7b0a024e7092f59c70fd4c2d780a28dd3dca56bff

[root@~]# docker ps  
CONTAINER ID   IMAGE                                COMMAND            CREATED         STATUS         PORTS                                                                                                                             NAMES  
f7c488c3e31a   yandex/clickhouse-server:21.4.3.21   "/entrypoint.sh"   4 seconds ago   Up 3 seconds   0.0.0.0:8123->8123/tcp, :::8123->8123/tcp, 0.0.0.0:9000->9000/tcp, :::9000->9000/tcp, 0.0.0.0:9009->9009/tcp, :::9009->9009/tcp   clickhouse-server

**三、 clickhouse 数据库修改密码 && 添加用户** 
----------------------------------

**算出密码的明文和密文**  
[root@~]# PASSWORD=$(base64 < /dev/urandom | head -c8); echo "$PASSWORD"; echo -n "$PASSWORD" | sha256sum | tr -d '-'    #安全起见此处密码为虚构，读者请根据实际算出的密码来配置  
明文密码：AN7u8CHa  
加密密码：0d4fb009a2082713343e434af372ddfeb4c7utr02f42916970e283e974449533 

  
**添加用户和密码**  
cd /etc/clickhouse-server  
cat  users.xml

将 65 行的 替换为 <password_sha256_hex>0d4fb009a2082713343e434af372ddfeb4c7d9f02f42916970e283e974449533</password_sha256_hex>

把 default 账号设为只读权限，并设置密码 yandex-->users-->default-->profile 节点设为 readonly 注释掉 yandex-->users-->default-->password 节点 新增  yandex-->users-->default-->password_sha256_hex 节点，填入生成的密码  
新增 root 账号  
<root>  
    <password_sha256_hex>0d4fb009a2082713343e434af372ddfeb4c7d9f02f42916970e283e974449533</password_sha256_hex>  
    <networks incl="networks" replace="replace">  
    <ip>::/0</ip>  
    </networks>  
    <profile>default</profile>  
    <quota>default</quota>  
</root>

  
也可本地编辑好 users.xml 文件 再复制到 docker 容器内  
[root@~]# docker cp users.xml  clickhouse-server:/etc/clickhouse-server/ 

**使用客户端工具测试连接**

用户名：root

密码为：AN7u8CHa

![](https://img-blog.csdnimg.cn/20210422165620419.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dvbGYxMTA1,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20210422165645732.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dvbGYxMTA1,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20210422170350346.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dvbGYxMTA1,size_16,color_FFFFFF,t_70)

已可以看到数据，数据库安装完成！