> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/huochen1994/article/details/78771379?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-1&spm=1001.2101.3001.4242)

使用 hangout 将 Kafka 数据实时清洗写入 ClickHouse
--------------------------------------

### 什么是 Hangout

Hangout 可以说是 JAVA 版的 Logstash，可以进行数据收集、分析并且将分析后的结果写入指定的地方  
[项目地址](https://github.com/childe/hangout)

### 什么是 ClickHouse

ClickHouse 是一个数据分析的数据库，由 Yandex 开源  
[项目地址](https://github.com/yandex/ClickHouse)

### 什么是 hangout-output-clickhouse

hangout-output-clickhouse 是一个将数据源中的数据实时写入 ClickHouse 的插件。  
[项目地址](https://github.com/RickyHuo/hangout-output-clickhouse)

#### 使用方法

*   从项目 _release_ 中下载 jar 包，放置`hangout/modules`目录下
*   插件使用

```
- com.sina.bip.hangout.outputs.Clickhouse:
        host: clickhouse.bip.sina.com.cn:8123
        database: apm
        table: apm_netdiagno
        fields: ['_device_id', '_ping_small', '_domain', '_traceroute', '_ping_big', 'date', 'ts', '_snet']
        bulk_size: 500
```

具体使用方法参考 [Hangout with Clickhouse](http://blog.csdn.net/huochen1994/article/details/78913626)