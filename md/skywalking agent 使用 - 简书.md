> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/963c322fc579)

概述
==

agent 探针可以让我们不修改代码的情况下，对 java 应用上使用到的组件进行动态监控，获取运行数据发送到 OAP 上进行统计和存储。agent 探针在 java 中是使用 java agent 技术实现的，不需要更改任何代码，java agent 会通过虚拟机 (VM) 接口来在运行期更改代码。

各种案例
====

### 1.tomcat

*   修改配置

```
cd /usr/local/skywalking/apache-skywalking-apm-bin/agent/config 
vi agent.config
agent.service_name=${SW_AGENT_NAME:skywalking_tomcat}

#修改tomcat启动文件，在文件顶部添加
vi /usr/local/skywalking/apache-tomcat-8.5.47/bin/catalina.sh
CATALINA_OPTS="$CATALINA_OPTS -javaagent:/usr/local/skywalking/apache- skywalking-apm-bin/agent/skywalking-agent.jar"; export CATALINA_OPTS

#修改tomcat启动端口
vi conf/server.xml
#修改这一行的端口为8081 <Connector port="8080" protocol=...


```

*   启动命令

```
./shutdown.sh 
./startup.sh


```

### 2.spring boot

*   拷贝并修改配置

```
copy agent apent_boot
cd /usr/local/skywalking/apache-skywalking-apm-bin/agent/config 
vi agent.config
agent.service_name=${SW_AGENT_NAME:skywalking_boot}


```

*   启动命令

```
java -javaagent:/usr/local/skywalking/apache-skywalking-apm-bin/agent_boot/skywalking-agent.jar -Dserver.port=8082 -jar skywalking_springboot.jar &


```

### 3.dubbo

*   拷贝并修改配置

```
copy agent apent_provider
cd /usr/local/skywalking/apache-skywalking-apm-bin/agent_provider/config 
vi agent.config
agent.service_name=${SW_AGENT_NAME:skywalking_provider}

copy agent apent_consumer
cd /usr/local/skywalking/apache-skywalking-apm-bin/agent_consumer/config 
vi agent.config
agent.service_name=${SW_AGENT_NAME:skywalking_consumer}


```

*   启动命令

```
java -javaagent:/usr/local/skywalking/apache-skywalking-apm-bin/agent_dubbo_provider/skywalking-agent.jar -jar skywalking_dubbo_provider.jar &

java -javaagent:/usr/local/skywalking/apache-skywalking-apm-bin/agent_dubbo_consumer/skywalking-agent.jar -jar skywalking_dubbo_consumer.jar & 


```

### 4.spring boot && mysql

*   拷贝并修改配置

```
copy agent apent_mysql
cd /usr/local/skywalking/apache-skywalking-apm-bin/agent_mysql/config 
vi agent.config
agent.service_name=${SW_AGENT_NAME:skywalking_mysql}


```

*   安装数据库

```
docker run -di --name=skywalking_mysql -p 33306:3306 -e MYSQL_ROOT_PASSWORD=123456 centos/mysql-57-centos7 

docker exec -it cc94e4bbd279 bash
mysql
update mysql.user set authentication_string=password(‘123456’) where user='root' ;
flush privileges;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
flush privileges;
exit;

mysql -uroot -p123456
CREATE DATABASE IF NOT EXISTS skywalking DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
CREATE TABLE `t_user` ( `id` int(11) NOT NULL AUTO_INCREMENT, `name` varchar(50) DEFAULT NULL, PRIMARY KEY (`id`) ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
insert into `t_user`(`name`) values ('张三'),('李四'),('王五');


```

*   启动命令

```
java -javaagent:/usr/local/skywalking/apache-skywalking-apm-bin/agent_mysql/skywalking-agent.jar -jar skywalking_mysql.jar --spring.datasource.url=jdbc:mysql://localhost:33306/skywalking?useSSL=false&serverTimezone=GMT &


```

### 5. 常用插件

*   配置覆盖  
    a) 系统配置 - javaagent:/path/to/skywalking-agent.jar=skywalking.agent.service_name=skywalking_db222

```
java -javaagent:/usr/local/skywalking/apache-skywalking-apm-bin/agent/skywalking-agent.jar -Dskywalking.agent.service_name=skywalking_db222 -jar skywalking_mysql.jar --spring.datasource.url=jdbc:mysql://localhost:33306/skywalking?useSSL=false&serverTimezone=GMT &


```

b) 探针配置 - Dskywalking.agent.service_name=skywalking_db

```
java -javaagent:/usr/local/skywalking/apache-skywalking-apm-bin/agent/skywalking-agent.jar=agent.service_name=skywalking_db -jar skywalking_mysql.jar --spring.datasource.url=jdbc:mysql://localhost:33306/skywalking?useSSL=false&serverTimezone=GMT &


```