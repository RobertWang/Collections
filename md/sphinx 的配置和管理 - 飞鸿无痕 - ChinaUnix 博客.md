> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.chinaunix.net](http://blog.chinaunix.net/uid-20639775-id-3277497.html)

> sphinx 的配置和管理 网上配置文档众多，但是对着他们的文档来做老是出问题，于是花了点时间研究了一下，写成总结，方便以后查阅。

**sphinx** **的配置和管理**

网上配置文档众多，但是对着他们的文档来做老是出问题，于是花了点时间研究了一下，写成总结，方便以后查阅。也希望学习 sphinx 的朋友能少走弯路。Coreseek 的安装请参考：[http://blog.chinaunix.net/uid-20639775-id-3261834.html](http://blog.chinaunix.net/uid-20639775-id-3261834.html)。

一、sphinx 的配置

1.   sphinx 配置文件结构介绍

Sphinx 的配置文件结构如下：

Source 源名称 1{     

#添加数据源，这里会设置一些连接数据库的参数比如数据库的 IP、用户名、密码等

#设置 sql_query、设置 sql_query_pre、设置 sql_query_range 等后面会结合例子做详细介绍

 ……

}

Index 索引名称 1{

     Source= 源名称 1

#设置全文索引

     ……

}

Indexer{

#设置 Indexer 程序配置选项，如内存限制等

……

}

Searchd{  

#设置 Searchd 守护进程本身的一些参数

……

}

Source 和 Index 都可以配置多个。

2.   spinx 配置案例详细解释

接下来就来针对一个配置案例来做详细的配置介绍：

#定义一个数据源

source search_main

{

           #定义数据库类型

    type                 = mysql

           #定义数据库的 IP 或者计算机名

    sql_host             = localhost

           #定义连接数据库的帐号

    sql_user             = root

           #定义链接数据库的密码

    sql_pass             = test123

           #定义数据库名称

    sql_db               = test

           #定义连接数据库后取数据之前执行的 SQL 语句

    sql_query_pre        = SET NAMES utf8

    sql_query_pre        = SET SESSION query_cache_type=OFF

           #创建一个 sph_counter 用于增量索引

    sql_query_pre        = CREATE TABLE IF NOT EXISTS sph_counter \

                                      ( counter_id INTEGER PRIMARY KEY NOT NULL,max_doc_id INTEGER NOT NULL)

           #取数据之前将表的最大 id 记录到 sph_counter 表中

    sql_query_pre        = REPLACE INTO sph_counter SELECT 1, MAX(searchid) FROM v9_search

           #定义取数据的 SQL，第一列 ID 列必须为唯一的正整数值

    sql_query            = SELECT searchid,typeid,id,adddate,data FROM v9_search where \

                                      searchid<(SELECT max_doc_id FROM sph_counter WHERE counter_id=1) \

                                        and searchid>=$start AND searchid<=$end

           # sql_attr_uint 和 sql_attr_timestamp 用于定义用于 api 过滤或者排序，写多行制定多列

    sql_attr_uint        = typeid

    sql_attr_uint        = id

    sql_attr_timestamp   = adddate

           #分区查询设置

    sql_query_range      = SELECT MIN(searchid),MAX(searchid) FROM v9_search

           #分区查询的步长

    sql_range_step       = 1000

           #设置分区查询的时间间隔

    sql_ranged_throttle  = 0

           #用于 CLI 的调试

    sql_query_info       = SELECT * FROM v9_search WHERE searchid=$id

}

#定义一个增量的源

source search_main_delta : search_main

{

    sql_query_pre       = set names utf8

           #增量源只查询上次主索引生成后新增加的数据

#如果新增加的 searchid 比主索引建立时的 searchid 还小那么会漏掉

    sql_query           = SELECT searchid,typeid,id,adddate,data FROM v9_search where  \

                                  searchid>(SELECT max_doc_id FROM sph_counter WHERE counter_id=1) \

                                   and searchid>=$start AND searchid<=$end

    sql_query_range     = SELECT MIN(searchid),MAX(searchid) FROM v9_search where \

                                       searchid>(SELECT max_doc_id FROM sph_counter WHERE counter_id=1)

}

#定义一个 index_search_main 索引

index index_search_main

{

           #设置索引的源

    source            = search_main

           #设置生成的索引存放路径

    path         = /usr/local/coreseek/var/data/index_search_main

           #定义文档信息的存储模式，extern 表示文档信息和文档 id 分开存储

    docinfo           = extern

           #设置已缓存数据的内存锁定，为 0 表示不锁定

    mlock             = 0

           #设置词形处理器列表，设置为 none 表示不使用任何词形处理器

    morphology        = none

           #定义最小索引词的长度

    min_word_len      = 1

           #设置字符集编码类型，我这里采用的 utf8 编码和数据库的一致

    charset_type      = zh_cn.utf-8

           #指定分词读取词典文件的位置

    charset_dictpath  = /usr/local/mmseg3/etc

           #不被搜索的词文件里表。

    stopwords       = /usr/local/coreseek/var/data/stopwords.txt

           #定义是否从输入全文数据中取出 HTML 标记

    html_strip       = 0

}

#定义增量索引

index index_search_main_delta : index_search_main

{

    source   = search_main_delta

    path    = /usr/local/coreseek/var/data/index_search_main_delta

}

#定义 indexer 配置选项

indexer

{

           #定义生成索引过程使用索引的限制

    mem_limit        = 512M

}

#定义 searchd 守护进程的相关选项

searchd

{

           #定义监听的 IP 和端口

    #listen            = 127.0.0.1

    #listen            = 172.16.88.100:3312

    listen            = 3312

    listen            = /var/run/searchd.sock

           #定义 log 的位置

    log                = /usr/local/coreseek/var/log/searchd.log

           #定义查询 log 的位置

    query_log          = /usr/local/coreseek/var/log/query.log

           #定义网络客户端请求的读超时时间

    read_timeout       = 5

           #定义子进程的最大数量

    max_children       = 300

           #设置 searchd 进程 pid 文件名

    pid_file           = /usr/local/coreseek/var/log/searchd.pid

           #定义守护进程在内存中为每个索引所保持并返回给客户端的匹配数目的最大值

    max_matches        = 100000

           #启用无缝 seamless 轮转，防止 searchd 轮转在需要预取大量数据的索引时停止响应

    #也就是说在任何时刻查询都可用，或者使用旧索引，或者使用新索引

    seamless_rotate    = 1

           #配置在启动时强制重新打开所有索引文件

    preopen_indexes    = 1

           #设置索引轮转成功以后删除以.old 为扩展名的索引拷贝

    unlink_old         = 1

           # MVA 更新池大小，这个参数不太明白

    mva_updates_pool   = 1M

           #最大允许的包大小

    max_packet_size    = 32M

           #最大允许的过滤器数

    max_filters        = 256

           #每个过滤器最大允许的值的个数

    max_filter_values  = 4096

}

二、sphinx 的管理

1.    生成 Sphinx 中文分词词库 (新版本的中文分词库已经生成在了 /usr/local/mmseg3/etc 目录下)

cd /usr/local/mmseg3/etc

/usr/local/mmseg3/bin/mmseg -u thesaurus.txt

mv thesaurus.txt.uni uni.lib

2.   生成 Sphinx 中文同义词库

#同义词库是说比如你搜索深圳的时候，含有深圳湾等字的也会被搜索出来

/data/software/sphinx/coreseek-3.2.14/mmseg-3.2.14/script/build_thesaurus.py unigram.txt > thesaurus.txt

/usr/local/mmseg3/bin/mmseg -t thesaurus.txt

将 thesaurus.lib 放到 uni.lib 同一目录

3.    生成全部索引

/usr/local/coreseek/bin/indexer --config /usr/local/coreseek/etc/sphinx.conf –all

若此时 searchd 守护进程已经启动，那么需要加上—rotate 参数：

/usr/local/coreseek/bin/indexer --config /usr/local/coreseek/etc/sphinx.conf --all --rotate

4.    启动 searchd 守护进程

/usr/local/coreseek/bin/searchd --config /usr/local/coreseek/etc/sphinx.conf

5.   生成主索引

写成 shell 脚本，添加到 crontab 任务，设置成每天凌晨 1 点的时候重建主索引

/usr/local/coreseek/bin/indexer --config /usr/local/coreseek/etc/sphinx.conf --rotate index_search_main

6.     生成增量索引

写成 shell 脚本，添加到 crontab 任务，设置成每 10 分钟运行一次

/usr/local/coreseek/bin/indexer --config /usr/local/coreseek/etc/sphinx.conf --rotate index_search_main_delta

7.    增量索引和主索引的合并

写成 shell 脚本，添加到计划任务，每 15 分钟跑一次

/usr/local/coreseek/bin/indexer --config /usr/local/coreseek/etc/sphinx.conf --merge index_search_main index_search_main_delta --rotate

8.    使用 search 命令在命令行对索引进行检索

/usr/local/coreseek/bin/search --config /usr/local/coreseek/etc/sphinx.conf  游戏

三、参考文章链接：

http://baobeituping.iteye.com/blog/870354

http://blog.s135.com/post/360/

http://youngerblue.iteye.com/blog/1513140

阅读 (15084) | 评论 (2) | 转发 (3) |