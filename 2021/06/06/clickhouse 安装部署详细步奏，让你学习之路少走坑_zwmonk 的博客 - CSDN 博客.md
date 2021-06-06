> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/zwmonk/article/details/108911408)

1. 安装前准备工作  
1.1 设置 CentOS7 打开文件数限制  
sudo vim /etc/security/limits.conf

*   ```
    soft    nofile            65536
    ```
    
*   ```
    hard    nofile            65536
    ```
    
*   ```
    soft    nproc             131072
    ```
    
*   ```
    hard    nproc             131072
    ```
    

在 sudo vim /etc/security/limits.d/90-nproc.conf 这个文件的末尾加入一下内容：

*   ```
    soft    nofile            65536
    ```
    
*   ```
    hard    nofile            65536
    ```
    
*   ```
    soft    nproc             131072
    ```
    
*   ```
    hard    nproc             131072
    ```
    

设置完成重启虚拟机 sudo reboot  
重启服务器之后生效，用 ulimit -n 或者 ulimit -a 查看设置结果  
ulimit -n

1.2 系统要求  
ClickHouse 可以在任何具有 x86_64，AArch64 或 PowerPC64LE CPU 架构的 Linux，FreeBSD 或 Mac OS X 上运行。  
虽然预构建的二进制文件通常是为 x86 _64 编译并利用 SSE 4.2 指令集，但除非另有说明，否则使用支持它的 CPU 将成为额外的系统要求。这是检查当前 CPU 是否支持 SSE 4.2 的命令：  
$ grep -q sse4_2 /proc/cpuinfo && echo “SSE 4.2 supported” || echo “SSE 4.2 not supported”  
设置防火墙状态 systemctl status firewalld.service  
设置防火墙开机自动重启 systemctl enable firewalld  
2. 下载安装包  
安装官网简略教程下载 tgz 安装包 https://repo.clickhouse.tech/tgz/stable/  
2.1 将安装包上传到虚拟机中  
![](https://img-blog.csdnimg.cn/20201004171121806.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3bW9uaw==,size_16,color_FFFFFF,t_70#pic_center)  
2.1 将安装包 tar 开并依次执行./doinst.sh 文件  
tar -zxvf clickhouse-common-static-20.5.4.40.tgz  
cd clickhouse-common-static-20.5.4.40/install  
sudo doinst.sh  
tar -zxvf clickhouse-common-static-dbg-20.5.4.40.tgz  
cd clickhouse-common-static-dbg-20.5.4.40/install  
sudo doinst.sh  
tar -zxvf clickhouse-server-20.5.4.40.tgz  
cd clickhouse-server-20.5.4.40/install  
sudo doinst.sh  
tar -zxvf clickhouse-client-20.5.4.40.tgz  
cd clickhouse-client-20.5.4.40/install  
sudo doinst.sh  
2.2 安装核心目录介绍  
![](https://img-blog.csdnimg.cn/2020100417170153.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3bW9uaw==,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20201004171734990.png#pic_center)  
2.3 修改 sevetl 的默认存储目录需要  
sudo vim /etc/clickhouse-server/config.xml  
![](https://img-blog.csdnimg.cn/20201004172446740.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3bW9uaw==,size_16,color_FFFFFF,t_70#pic_center)  
3. 启动服务 clickhouse-server --config-file=/etc/clickhouse-server/config.xml  
报错如下解决方案：修改安装目录的权限！，默认使用 clickhouse 用户！命令为  
cd /var/lib/  
chown -R root:root clickhous  
![](https://img-blog.csdnimg.cn/20201006215357506.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3bW9uaw==,size_16,color_FFFFFF,t_70#pic_center)  
4. 修改完成后，启动 clickhouse 服务  
clickhouse-server --config-file=/etc/clickhouse-server/config.xml  
![](https://img-blog.csdnimg.cn/20201006220930960.png#pic_center)  
5. 查看服务是否启动起来  
ps -aux | grep clickhouse  
![](https://img-blog.csdnimg.cn/20201006221133941.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3bW9uaw==,size_16,color_FFFFFF,t_70#pic_center)  
6. 启动 client 客户端  
![](https://img-blog.csdnimg.cn/20201006221221964.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3bW9uaw==,size_16,color_FFFFFF,t_70#pic_center)  
后续集群模式敬请期待