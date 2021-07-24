> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/moris5013/p/12371549.html)

 1》docker 中安装 mysql 容器

开启 binlog 模式

修改 / etc/mysql/mysql.conf.d/mysqld.cnf 

```
docker exec -it mysql /bin/bash
cd /etc/mysql/mysql.conf.d
vi mysqld.cnf
```

添加这两行

[![](https://img2018.cnblogs.com/i-beta/1458513/202002/1458513-20200227123542528-1104973297.png)](https://img2018.cnblogs.com/i-beta/1458513/202002/1458513-20200227123542528-1104973297.png)

 2》创建用于同步的账号并授权

采用 root 账号登录

```
mysql -uroot -p123456
```

```
create user canal@'%' IDENTIFIED by 'canal';
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT,SUPER ON *.* TO 'canal'@'%';
FLUSH PRIVILEGES;
```

[![](https://img2018.cnblogs.com/i-beta/1458513/202002/1458513-20200227125128810-905809370.png)](https://img2018.cnblogs.com/i-beta/1458513/202002/1458513-20200227125128810-905809370.png)

 3》重启 mysql 容器

```
docker restart mysql
```

4》docker 中安装 canal

docker pull docker.io/canal/canal-server

docker run -p 11111:11111 --name canal -d docker.io/canal/canal-server

```
docker exec -it canal /bin/bash
cd canal-server/conf/
vi canal.properties
cd example/
vi instance.properties
```

```
canal.properties的配置只要保证 canal.id 和master数据库中的serverid不重复
```

```
instance.properties 
要配置master的地址 canal.instance.master.address 
用于同步的账号 canal.instance.dbUsername
用于同步的密码 canal.instance.dbPassword
需要监听哪些表的正则过滤 canal.instance.filter.regex
canal的实例名称 canal.mq.topic
由于mysql主数据库是用docker安装的，这里master的地址要填写mysql容器的地址，先进入mysql容器，再查看ip就可以。
```

[![](https://img2018.cnblogs.com/i-beta/1458513/202002/1458513-20200227134537420-797220787.png)](https://img2018.cnblogs.com/i-beta/1458513/202002/1458513-20200227134537420-797220787.png)

 [![](https://img2018.cnblogs.com/i-beta/1458513/202002/1458513-20200227135143406-1923981283.png)](https://img2018.cnblogs.com/i-beta/1458513/202002/1458513-20200227135143406-1923981283.png)

canal.instance.filter.regex = .*\\..*    表示所有的表

canal.mq.topic 配置的实例表示微服务中要监听相同实例

```
docker update --restart=always canal
docker restart canal
```