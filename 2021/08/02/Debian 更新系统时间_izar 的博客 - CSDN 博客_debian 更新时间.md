> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/ruiyelp/article/details/79755060?locationNum=2&fps=1)

Debian 更新时间
===========

**A 更新源，并安装 ntpdate**：  
0.`date` 查看当前的系统时间  
1.`sudo apt-get update` 更新源  
2.`sudo apt-get install ntpdate` 安装 ntpdate  
3.`sudo ntpdate ntp1.aliyun.com` 更新系统时间

**B 更新时区**：  
4.`date -R` 可查看时区 不是 08 区，则更新时区  
5.`sudo tzselect`  
6. 选择`5 Asia`  
7. 选择`9 China`  
8. 选择`1 Beijing Time`  
9. 选择`1 Yes`

**C 更新系统变量**：  
10. 执行 `sudo nano /etc/profile`  
11. 在 export PATH 前加一行：`export TZ='Asia/Shanghai'`  
12.export PATH 添加：`export PATH=$JAVA_HOME/bin:$TZ:$PATH`  
13.Crtl+O 写入、Ctrl+X 退出  
14. 执行：`source /etc/profile`更新系统变量

15. 再 date 查看时间，完成