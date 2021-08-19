> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA3OTc0MzY1Mg==&mid=2247515151&idx=5&sn=daa9a753b8333319641c3b0b270eec0b&chksm=9fac23c4a8dbaad2ee3dc575d52786effa8342626e65b1fc3cdf56a742f6a29ff8b270c56d36&mpshare=1&scene=1&srcid=0819uBlvGk5gT3aGCNO1BUiY&sharer_sharetime=1629345303195&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

  

### _来源：jiandansuifeng.blog.csdn.net/_

### _article/details/107361190_

整体流程大概如下：

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufKyhtDYaMAVSFNfSQEHJNmURYXUYaQTzrXWIGTD5ht1Qs9T4TpMy6KGHhAEeNbRktaqIjAIk7OjA/640?wx_fmt=png)

### 服务器准备

在这先列出各服务器节点，方便同学们在下文中对照节点查看相应内容

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufKyhtDYaMAVSFNfSQEHJNme2icJMOowD5wCdhbKy0iblKdvlcYlHKib9CyK5VLicDj7A4W6q0qZTHn8A/640?wx_fmt=png)

### SpringBoot 项目准备

引入 log4j2 替换 SpringBoot 默认 log，demo 项目结构如下：

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufKyhtDYaMAVSFNfSQEHJNmHYnG8438iaiaibabC2DQubmTQqQenRZL8EaKa2UZ36OeiaAibGKWa4sDjng/640?wx_fmt=png)

**pom**

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <!--  排除spring-boot-starter-logging -->
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency> 
 <!-- log4j2 -->
 <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-log4j2</artifactId>
 </dependency> 
   <dependency>
     <groupId>com.lmax</groupId>
     <artifactId>disruptor</artifactId>
     <version>3.3.4</version>
   </dependency> 
</dependencies> 
```

**log4j2.xml**

```
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="INFO" schema="Log4J-V2.0.xsd" monitorInterval="600" >
    <Properties>
        <Property >logs</Property>
        <property >collector</property>
        <property >[%d{yyyy-MM-dd'T'HH:mm:ss.SSSZZ}] [%level{length=5}] [%thread-%tid] [%logger] [%X{hostName}] [%X{ip}] [%X{applicationName}] [%F,%L,%C,%M] [%m] ## '%ex'%n</property>
    </Properties>
    <Appenders>
        <Console >
            <PatternLayout pattern="${patternLayout}"/>
        </Console>  
        <RollingRandomAccessFile ${LOG_HOME}/app-${FILE_NAME}.log" filePattern="${LOG_HOME}/app-${FILE_NAME}-%d{yyyy-MM-dd}-%i.log" >
          <PatternLayout pattern="${patternLayout}" />
          <Policies>
              <TimeBasedTriggeringPolicy interval="1"/>
              <SizeBasedTriggeringPolicy size="500MB"/>
          </Policies>
          <DefaultRolloverStrategy max="20"/>         
        </RollingRandomAccessFile>
        <RollingRandomAccessFile ${LOG_HOME}/error-${FILE_NAME}.log" filePattern="${LOG_HOME}/error-${FILE_NAME}-%d{yyyy-MM-dd}-%i.log" >
          <PatternLayout pattern="${patternLayout}" />
          <Filters>
              <ThresholdFilter level="warn" onMatch="ACCEPT" onMismatch="DENY"/>
          </Filters>              
          <Policies>
              <TimeBasedTriggeringPolicy interval="1"/>
              <SizeBasedTriggeringPolicy size="500MB"/>
          </Policies>
          <DefaultRolloverStrategy max="20"/>         
        </RollingRandomAccessFile>            
    </Appenders>
    <Loggers>
        <!-- 业务相关 异步logger -->
        <AsyncLogger >
          <AppenderRef ref="appAppender"/>
        </AsyncLogger>
        <AsyncLogger >
          <AppenderRef ref="errorAppender"/>
        </AsyncLogger>       
        <Root level="info">
            <Appender-Ref ref="CONSOLE"/>
            <Appender-Ref ref="appAppender"/>
            <AppenderRef ref="errorAppender"/>
        </Root>         
    </Loggers>
</Configuration>
```

**IndexController**

测试 Controller，用以[打印日志](http://mp.weixin.qq.com/s?__biz=MzI4Njc5NjM1NQ==&mid=2247505379&idx=1&sn=e09600f3b78d9e4d5e735e9456b5d181&chksm=ebd5eacfdca263d97df7ce591618b37865d0db0b9cf5261e41d63c2ad62ce785a79bd1c82218&scene=21#wechat_redirect)进行调试

```
@Slf4j
@RestController
public class IndexController {

 @RequestMapping(value = "/index")
 public String index() {
  InputMDC.putMDC();
  
  log.info("我是一条info日志");
  
  log.warn("我是一条warn日志");

  log.error("我是一条error日志");
  
  return "idx";
 }


 @RequestMapping(value = "/err")
 public String err() {
  InputMDC.putMDC();
  try {
   int a = 1/0;
  } catch (Exception e) {
   log.error("算术异常", e);
  }
  return "err";
 }
 
}
```

**InputMDC**

用以获取 log 中的`[%X{hostName}]`、`[%X{ip}]`、`[%X{applicationName}]`三个字段值

```
@Component
public class InputMDC implements EnvironmentAware {

 private static Environment environment;
 
 @Override
 public void setEnvironment(Environment environment) {
  InputMDC.environment = environment;
 }
 
 public static void putMDC() {
  MDC.put("hostName", NetUtil.getLocalHostName());
  MDC.put("ip", NetUtil.getLocalIp());
  MDC.put("applicationName", environment.getProperty("spring.application.name"));
 }

}
```

**NetUtil**

```
public class NetUtil {   
 
 public static String normalizeAddress(String address){
  String[] blocks = address.split("[:]");
  if(blocks.length > 2){
   throw new IllegalArgumentException(address + " is invalid");
  }
  String host = blocks[0];
  int port = 80;
  if(blocks.length > 1){
   port = Integer.valueOf(blocks[1]);
  } else {
   address += ":"+port; //use default 80
  } 
  String serverAddr = String.format("%s:%d", host, port);
  return serverAddr;
 }
 
 public static String getLocalAddress(String address){
  String[] blocks = address.split("[:]");
  if(blocks.length != 2){
   throw new IllegalArgumentException(address + " is invalid address");
  } 
  String host = blocks[0];
  int port = Integer.valueOf(blocks[1]);
  
  if("0.0.0.0".equals(host)){
   return String.format("%s:%d",NetUtil.getLocalIp(), port);
  }
  return address;
 }
 
 private static int matchedIndex(String ip, String[] prefix){
  for(int i=0; i<prefix.length; i++){
   String p = prefix[i];
   if("*".equals(p)){ //*, assumed to be IP
    if(ip.startsWith("127.") ||
       ip.startsWith("10.") || 
       ip.startsWith("172.") ||
       ip.startsWith("192.")){
     continue;
    }
    return i;
   } else {
    if(ip.startsWith(p)){
     return i;
    }
   } 
  }
  
  return -1;
 }
 
 public static String getLocalIp(String ipPreference) {
  if(ipPreference == null){
   ipPreference = "*>10>172>192>127";
  }
  String[] prefix = ipPreference.split("[> ]+");
  try {
   Pattern pattern = Pattern.compile("[0-9]+\\.[0-9]+\\.[0-9]+\\.[0-9]+");
   Enumeration<NetworkInterface> interfaces = NetworkInterface.getNetworkInterfaces();
   String matchedIp = null;
   int matchedIdx = -1;
   while (interfaces.hasMoreElements()) {
    NetworkInterface ni = interfaces.nextElement();
    Enumeration<InetAddress> en = ni.getInetAddresses(); 
    while (en.hasMoreElements()) {
     InetAddress addr = en.nextElement();
     String ip = addr.getHostAddress();  
     Matcher matcher = pattern.matcher(ip);
     if (matcher.matches()) {  
      int idx = matchedIndex(ip, prefix);
      if(idx == -1) continue;
      if(matchedIdx == -1){
       matchedIdx = idx;
       matchedIp = ip;
      } else {
       if(matchedIdx>idx){
        matchedIdx = idx;
        matchedIp = ip;
       }
      }
     } 
    } 
   } 
   if(matchedIp != null) return matchedIp;
   return "127.0.0.1";
  } catch (Exception e) { 
   return "127.0.0.1";
  }
 }
 
 public static String getLocalIp() {
  return getLocalIp("*>10>172>192>127");
 }
 
 public static String remoteAddress(SocketChannel channel){
  SocketAddress addr = channel.socket().getRemoteSocketAddress();
  String res = String.format("%s", addr);
  return res;
 }
 
 public static String localAddress(SocketChannel channel){
  SocketAddress addr = channel.socket().getLocalSocketAddress();
  String res = String.format("%s", addr);
  return addr==null? res: res.substring(1);
 }
 
 public static String getPid(){
  RuntimeMXBean runtime = ManagementFactory.getRuntimeMXBean();
        String name = runtime.getName();
        int index = name.indexOf("@");
        if (index != -1) {
            return name.substring(0, index);
        }
  return null;
 }
 
 public static String getLocalHostName() {
        try {
            return (InetAddress.getLocalHost()).getHostName();
        } catch (UnknownHostException uhe) {
            String host = uhe.getMessage();
            if (host != null) {
                int colon = host.indexOf(':');
                if (colon > 0) {
                    return host.substring(0, colon);
                }
            }
            return "UnknownHost";
        }
    }
}
```

启动项目，访问`/index`和`/ero`接口，可以看到项目中生成了`app-collector.log`和`error-collector.log`两个日志文件

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufKyhtDYaMAVSFNfSQEHJNmlVicwtXicPZTjKOzDib3vxO0ia3IYEsrpjvNibWDkMic3bv3lxjUq6SQBFZg/640?wx_fmt=png)

我们将 Springboot 服务部署在 192.168.11.31 这台机器上。

### Kafka 安装和启用

kafka 下载地址：

> http://kafka.apache.org/downloads.html

kafka 安装步骤：首先 kafka 安装需要依赖与 [zookeeper](http://mp.weixin.qq.com/s?__biz=MzI4Njc5NjM1NQ==&mid=2247510278&idx=2&sn=9484de415f3fe2ad0515eec321f9716a&chksm=ebd5962adca21f3ccebd8b6f8cc590664878d5ab152bc1d22b4525b06d9a4b666af5716315eb&scene=21#wechat_redirect)，所以小伙伴们先准备好 zookeeper 环境（三个节点即可），然后我们来一起构建 kafka broker。

```
## 解压命令：
tar -zxvf kafka_2.12-2.1.0.tgz -C /usr/local/
## 改名命令：
mv kafka_2.12-2.1.0/ kafka_2.12
## 进入解压后的目录，修改server.properties文件：
vim /usr/local/kafka_2.12/config/server.properties
## 修改配置：
broker.id=0
port=9092
host.name=192.168.11.51
advertised.host.name=192.168.11.51
log.dirs=/usr/local/kafka_2.12/kafka-logs
num.partitions=2
zookeeper.connect=192.168.11.111:2181,192.168.11.112:2181,192.168.11.113:2181

## 建立日志文件夹：
mkdir /usr/local/kafka_2.12/kafka-logs

##启动kafka：
/usr/local/kafka_2.12/bin/kafka-server-start.sh /usr/local/kafka_2.12/config/server.properties &
```

创建两个 topic

```
## 创建topic
kafka-topics.sh --zookeeper 192.168.11.111:2181 --create --topic app-log-collector --partitions 1  --replication-factor 1
kafka-topics.sh --zookeeper 192.168.11.111:2181 --create --topic error-log-collector --partitions 1  --replication-factor 1 
```

我们可以查看一下 topic 情况

```
kafka-topics.sh --zookeeper 192.168.11.111:2181 --topic app-log-test --describe
```

可以看到已经成功启用了`app-log-collector`和`error-log-collector`两个 topic

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufKyhtDYaMAVSFNfSQEHJNmJZ5FlHVJPGMu7gI7s8K4XvYo1iaI0oZfBmBsBlq714rFYOsxkf3dpicQ/640?wx_fmt=png)

### filebeat 安装和启用

**filebeat 下载**

```
cd /usr/local/software
tar -zxvf filebeat-6.6.0-linux-x86_64.tar.gz -C /usr/local/
cd /usr/local
mv filebeat-6.6.0-linux-x86_64/ filebeat-6.6.0
```

配置 filebeat，可以参考下方 yml 配置文件

```
vim /usr/local/filebeat-5.6.2/filebeat.yml
```

```
###################### Filebeat Configuration Example #########################
filebeat.prospectors:

- input_type: log

  paths:
    ## app-服务名称.log, 为什么写死，防止发生轮转抓取历史数据
    - /usr/local/logs/app-collector.log
  #定义写入 ES 时的 _type 值
  document_type: "app-log"
  multiline:
    #pattern: '^\s*(\d{4}|\d{2})\-(\d{2}|[a-zA-Z]{3})\-(\d{2}|\d{4})'   # 指定匹配的表达式（匹配以 2017-11-15 08:04:23:889 时间格式开头的字符串）
    pattern: '^\['                              # 指定匹配的表达式（匹配以 "{ 开头的字符串）
    negate: true                                # 是否匹配到
    match: after                                # 合并到上一行的末尾
    max_lines: 2000                             # 最大的行数
    timeout: 2s                                 # 如果在规定时间没有新的日志事件就不等待后面的日志
  fields:
    logbiz: collector
    logtopic: app-log-collector   ## 按服务划分用作kafka topic
    evn: dev

- input_type: log

  paths:
    - /usr/local/logs/error-collector.log
  document_type: "error-log"
  multiline:
    #pattern: '^\s*(\d{4}|\d{2})\-(\d{2}|[a-zA-Z]{3})\-(\d{2}|\d{4})'   # 指定匹配的表达式（匹配以 2017-11-15 08:04:23:889 时间格式开头的字符串）
    pattern: '^\['                              # 指定匹配的表达式（匹配以 "{ 开头的字符串）
    negate: true                                # 是否匹配到
    match: after                                # 合并到上一行的末尾
    max_lines: 2000                             # 最大的行数
    timeout: 2s                                 # 如果在规定时间没有新的日志事件就不等待后面的日志
  fields:
    logbiz: collector
    logtopic: error-log-collector   ## 按服务划分用作kafka topic
    evn: dev
    
output.kafka:
  enabled: true
  hosts: ["192.168.11.51:9092"]
  topic: '%{[fields.logtopic]}'
  partition.hash:
    reachable_only: true
  compression: gzip
  max_message_bytes: 1000000
  required_acks: 1
logging.to_files: true
```

**filebeat 启动：**

检查配置是否正确

```
cd /usr/local/filebeat-6.6.0
./filebeat -c filebeat.yml -configtest
## Config OK
```

启动 filebeat

```
/usr/local/filebeat-6.6.0/filebeat &
```

检查是否启动成功

```
ps -ef | grep filebeat
```

可以看到 filebeat 已经启动成功

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufKyhtDYaMAVSFNfSQEHJNm4QZM7CI3puroibSOC4OAGtyxuQzdxSUSsRVPmvAXorrzI6nceaMPzXA/640?wx_fmt=png)

然后我们访问 192.168.11.31:8001/index 和 192.168.11.31:8001/err，再查看 kafka 的 logs 文件，可以看到已经生成了 app-log-collector-0 和 error-log-collector-0 文件，说明 filebeat 已经帮我们把数据收集好放到了 kafka 上。

### logstash 安装

我们在 logstash 的安装目录下新建一个文件夹

```
mkdir scrpit
```

然后 cd 进该文件，创建一个`logstash-script.conf`文件

```
cd scrpit
vim logstash-script.conf
```

```
## multiline 插件也可以用于其他类似的堆栈式信息，比如 linux 的内核日志。
input {
  kafka {
    ## app-log-服务名称
    topics_pattern => "app-log-.*"
    bootstrap_servers => "192.168.11.51:9092"
 codec => json
 consumer_threads => 1 ## 增加consumer的并行消费线程数
 decorate_events => true
    #auto_offset_rest => "latest"
 group_id => "app-log-group"
   }
   
   kafka {
    ## error-log-服务名称
    topics_pattern => "error-log-.*"
    bootstrap_servers => "192.168.11.51:9092"
 codec => json
 consumer_threads => 1
 decorate_events => true
    #auto_offset_rest => "latest"
 group_id => "error-log-group"
   }
   
}

filter {
  
  ## 时区转换
  ruby {
 code => "event.set('index_time',event.timestamp.time.localtime.strftime('%Y.%m.%d'))"
  }

  if "app-log" in [fields][logtopic]{
    grok {
        ## 表达式,这里对应的是Springboot输出的日志格式
        match => ["message", "\[%{NOTSPACE:currentDateTime}\] \[%{NOTSPACE:level}\] \[%{NOTSPACE:thread-id}\] \[%{NOTSPACE:class}\] \[%{DATA:hostName}\] \[%{DATA:ip}\] \[%{DATA:applicationName}\] \[%{DATA:location}\] \[%{DATA:messageInfo}\] ## (\'\'|%{QUOTEDSTRING:throwable})"]
    }
  }

  if "error-log" in [fields][logtopic]{
    grok {
        ## 表达式
        match => ["message", "\[%{NOTSPACE:currentDateTime}\] \[%{NOTSPACE:level}\] \[%{NOTSPACE:thread-id}\] \[%{NOTSPACE:class}\] \[%{DATA:hostName}\] \[%{DATA:ip}\] \[%{DATA:applicationName}\] \[%{DATA:location}\] \[%{DATA:messageInfo}\] ## (\'\'|%{QUOTEDSTRING:throwable})"]
    }
  }
  
}

## 测试输出到控制台：
output {
  stdout { codec => rubydebug }
}


## elasticsearch：
output {

  if "app-log" in [fields][logtopic]{
 ## es插件
 elasticsearch {
       # es服务地址
        hosts => ["192.168.11.35:9200"]
        # 用户名密码      
        user => "elastic"
        password => "123456"
        ## 索引名，+ 号开头的，就会自动认为后面是时间格式：
        ## javalog-app-service-2019.01.23 
        index => "app-log-%{[fields][logbiz]}-%{index_time}"
        # 是否嗅探集群ip：一般设置true；http://192.168.11.35:9200/_nodes/http?pretty
        # 通过嗅探机制进行es集群负载均衡发日志消息
        sniffing => true
        # logstash默认自带一个mapping模板，进行模板覆盖
        template_overwrite => true
    } 
  }
  
  if "error-log" in [fields][logtopic]{
 elasticsearch {
        hosts => ["192.168.11.35:9200"]    
        user => "elastic"
        password => "123456"
        index => "error-log-%{[fields][logbiz]}-%{index_time}"
        sniffing => true
        template_overwrite => true
    } 
  }
  

}
```

启动 logstash

```
/usr/local/logstash-6.6.0/bin/logstash -f /usr/local/logstash-6.6.0/script/logstash-script.conf &
```

等待启动成功，我们再次访问`192.168.11.31:8001/err`

可以看到控制台开始打印日志

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufKyhtDYaMAVSFNfSQEHJNmc6PicMGxFWK9debJGLcNpA1HjfpWvlhJlc0DPKvRPC9L95weDeHibzSg/640?wx_fmt=png)

### ElasticSearch 与 Kibana

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufKyhtDYaMAVSFNfSQEHJNmsWqYkImT4xzzDo9JH14KK7lNc09qcxXd9XeaauSibh2ZoRrJdwZLAPQ/640?wx_fmt=png)

ES 和 Kibana 的搭建之前没写过博客，网上资料也比较多，大家可以自行搜索。

搭建完成后，访问 Kibana 的管理页面`192.168.11.35:5601`，选择 Management -> Kinaba - Index Patterns

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufKyhtDYaMAVSFNfSQEHJNmsQ29FrAaEdzQeicbRcL3h09kxxicIibmpN2KRxbNIvVmnGw1yOektCHDw/640?wx_fmt=png)

然后 Create index pattern

*   index pattern 输入 `app-log-*`
    
*   Time Filter field name 选择 currentDateTime
    

这样我们就成功创建了索引。

我们再次访问`192.168.11.31:8001/err`，这个时候就可以看到我们已经命中了一条 log 信息

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufKyhtDYaMAVSFNfSQEHJNmQPMoUIocUUbd945xAodH3tAaaI3zyWsuZPhEcpBBpQRfOr0ibt6HVMw/640?wx_fmt=png)

里面展示了日志的全量信息

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufKyhtDYaMAVSFNfSQEHJNmuSr9MQhyEkJXmlmGmH3TQiaKs6IoOZTeO67JicWg6OZx0QzoJrgvJ4xw/640?wx_fmt=png)

到这里，我们完整的日志收集及可视化就搭建完成了，grafana loki 也不错，用起来贼爽！