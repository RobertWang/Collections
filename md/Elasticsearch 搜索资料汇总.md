> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/heyuquan/p/14037887.html)

       Elasticsearch（ES）是近实时的分布式搜索分析引擎。这篇文章整理和筛选了 ES 相关资料，包括索引、分词、多条件查询、聚合、自动补全、建议词、同义词、安全性等等，便于大家学习和使用 ES 搜索引擎。

Elasticsearch 简介

Elasticsearch（ES）是一个基于 Lucene 构建的开源分布式搜索分析引擎，可以近实时的索引、检索数据。具备高可靠、易使用、社区活跃等特点，在全文检索、日志分析、监控分析等场景具有广泛应用。

lucene

[Lucene 介绍与入门使用](https://www.cnblogs.com/xiaobai1226/p/7652093.html)

[Lucene.Net 文档](https://lucenenet.apache.org/docs/)

[Lucene 可视化工具 Luke](https://blog.csdn.net/alex_xfboy/article/details/85329837)

Elasticsearch 中文社区：[https://elasticsearch.cn/article/](https://elasticsearch.cn/article/)

Elasticsearch 官方文档：[https://www.elastic.co/guide/index.html](https://www.elastic.co/guide/index.html)

[Elasticsearch 各客户端 API（eg：.NET、JAVA、Python、Go）](https://www.elastic.co/guide/en/elasticsearch/client/index.html)

[Elasticsearch .net client NEST 5.x 使用总结（初始化、查询、权重、排序、聚合等）](https://www.cnblogs.com/huhangfei/p/7524886.html)

Elasticsearch 客户端 SDK 使用建议：创建索引的 Setting 和 mapping 使用 elasticsearch 提供的 DSL 语法更加简单。因为客户端 API 代码里面只提供基础的 SDK，如（ik 拼音等）插件就没有对应接口提供

[Elasticsearch 术语（索引、类型、文档、集群、节点、分片）](https://www.cnblogs.com/wupeixuan/p/12375031.html)

**ES** **数据架构的主要概念（与关系数据库 Mysql 对比）**

[![](https://img2020.cnblogs.com/blog/106337/202011/106337-20201125191307471-823696285.png)](https://www.cnblogs.com/wupeixuan/p/12375031.html)

在 ES 早期版本，一个索引下是可以有多个 Type 。[从 6.0 开始，一个索引只有一个 Type，即](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/breaking-changes-6.0.html)[_doc](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/breaking-changes-6.0.html)（查询语句中也不要包含 type）。一个 Type 下的文档，都有相同的字段（Field）

查询语句：

GET [index]/[type]/_search  
变更为  
GET [index]/_search

安装

[docker 版本的 ELK 快速部署](https://www.cnblogs.com/ShaoJianan/p/11455250.html)

ELK

ELK 是 elastic 公司旗下三款产品 ElasticSearch 、Logstash 、Kibana 的首字母组合。

#、ElasticSearch 是一个基于 Lucene 构建的开源，分布式，RESTful 搜索引擎。

#、Logstash 传输和处理你的日志、事务或其他数据。

#、Kibana 将 Elasticsearch 的数据分析并渲染为可视化的报表。

[Kibana User Guide](https://www.elastic.co/guide/en/kibana/current/index.html)

es-head 可视化插件

[docker 安装 elasticsearch 和 head 插件](https://www.cnblogs.com/wxy0126/p/11381598.html)

[Elasticsearch Head 数据浏览 406](https://shanhy.blog.csdn.net/article/details/103737698)

分词器

分词器是专门处理分词的组件，分词器由如下三部分组成：

1、Character Filters：针对原始文本处理，比如：去除 html 标签

2、Tokenizer：按照规则切分为单词，比如：按照空格切分

3、Token Filters：将切分的单词进行加工，比如：大写转小写，删除 stopwords，拼音，同义词等

analyzer = CharFilters（0 个或多个）+ Tokenizer(一个) + TokenFilters(0 个或多个)

[![](https://img2020.cnblogs.com/blog/106337/202011/106337-20201125191308074-107739394.jpg)](https://www.cnblogs.com/wxy0126/p/11381598.html)

从图中能够看出，从上到下依次通过 Character Filters，Tokenizer 以及 Token Filters，这个顺序比较好理解，一个文本进来确定要先对文本数据进行处理，再去分词，最后对分词的结果进行过滤。

[ElasticSearch 分词器是什么](https://www.cnblogs.com/wupeixuan/p/12444528.html)

[一些分词器介绍（比如适用于英语的 Snowball ）](https://www.ibm.com/support/knowledgecenter/zh/SSGU8G_12.1.0/com.ibm.dbext.doc/ids_dbxt_522.htm)

[elasticSearch Analysis Token Filters 作用及相关样例](https://blog.csdn.net/wangzhuo0978/article/details/79914849)

[Writing analyzers](https://www.elastic.co/guide/en/elasticsearch/client/net-api/current/writing-analyzers.html)

[ElasticSearch 查看字段分词结果](https://blog.csdn.net/u014078154/article/details/79760766?utm_source=blogxgwz8) （便于查为什么匹配不出的问题）

[Elasticsearch7 分词器 (内置分词器和自定义分词器)](https://blog.csdn.net/white_while/article/details/98504574)

[Elasticsearch-Analysis-IK 中文分词器配置使用](https://blog.csdn.net/chy2z/article/details/82962903)

[elasticsearch 之分词器配置 (IK+pinyin)](https://www.cnblogs.com/greatom/p/10560411.html)

[Elasticsearch 使用 ik 中文分词器增加分词热词（自定义词）](https://www.cnblogs.com/a393060727/p/12099651.html)

Elasticsearch mapping

[搞懂 Elasticsearch 之 Mapping （Reindex）](https://www.cnblogs.com/wupeixuan/p/12514843.html)

Mapping 中的 store 属性（按需查询字段）

[Elasticsearch 中的 store field 跟 non-store field 的区别](https://blog.csdn.net/west_609/article/details/74906485)

[Elasticsearch 理解 mapping 中的 store 属性](https://www.cnblogs.com/sanduzxcvbnm/p/12157453.html)

[Elasticsearch 动态模板 (dynamic_templates)](https://www.cnblogs.com/zhb-php/p/7510233.html)

normalizer 的使用

[ElasticSearch Normalizer 的使用方法](https://blog.csdn.net/yinni11/article/details/104431632)

[elasticsearch 大小写无法使用 term 查询的问题](https://www.jianshu.com/p/a86074177585)

[Adding normalizer for all keyword fields NEST](https://stackoverflow.com/questions/61062996/adding-normalizer-for-all-keyword-fields-nest)

Elasticsearch DLS 语法

[Elasticsearch 查询语法（模糊、精确、sort、相关性、and|or、slop 间隔等）](https://www.cnblogs.com/wupeixuan/p/12483846.html)

[Elasticsearch 查询语法（多条件 bool 复杂查询（must、should、filter）、日期范围查询）](https://www.elastic.co/guide/en/elasticsearch/client/net-api/current/writing-queries.html)

[Elasticsearch 查询语法（bool 复杂查询、operator（||、&&、!、+）)](https://www.elastic.co/guide/en/elasticsearch/client/net-api/current/bool-queries.html)

[Elasticsearch 组合多查询 (bool, must, should, must_not, filter)](https://www.it610.com/article/1297259129663987712.htm)

[Elasticsearch 多字符串多字段查询，权重](https://www.cnblogs.com/powercto/p/14532963.html)

[Elasticsearch 中 match、match_phrase、query_string 和 term 的区别](https://blog.csdn.net/ahwsk/article/details/101270300)

相关性 score

[ElasticSearch 的分数 (_score) 是怎么计算得出 (2.X & 5.X)](https://ruby-china.org/topics/31934)

[Elasticsearch filter 和 query 的不同](https://blog.csdn.net/laoyang360/article/details/80468757)

[ElasticSearch 多级排序（eg：产品要根据：销量、热度、相关性排序）](https://www.cnblogs.com/shaosks/p/7542076.html)

[Elasticsearch 搜索条件权重控制（boost）](https://blog.csdn.net/cs1509235061/article/details/89450553)-- 默认情况下，搜索条件的权重都是 1

聚合查询

[Elasticsearch 聚合语法（Aggregations）](https://www.elastic.co/guide/en/elasticsearch/client/net-api/current/writing-aggregations.html)

[Elasticsearch 聚合查询](https://blog.csdn.net/alex_xfboy/article/details/86100037)

[通过 Elasticsearch 实现聚合检索 (分组统计)](https://www.cnblogs.com/shoufeng/p/11290669.html)

[Elasticsearch 范围查询（数值、日期）](https://blog.csdn.net/qq_32165041/article/details/83714203)

分页查询

[Elasticsearch 分页查询](https://blog.csdn.net/u011228889/article/details/79760167)

[Elasticsearch 查询语法（使用 scroll 响应式返回大集合文档）](https://www.elastic.co/guide/en/elasticsearch/client/net-api/current/scrolling-documents.html)

[Elasticsearch 嵌套查询，父子关系查询](https://www.elastic.co/guide/en/elasticsearch/client/net-api/current/inner-hits-usage.html)

[Elasticsearch 高亮显示匹配关键词（Highlight）](https://www.elastic.co/guide/en/elasticsearch/client/net-api/current/highlighting-usage.html)

同义词

[elasticsearch 使用同义词（synonym.txt）](https://www.cnblogs.com/spectrelb/p/8038980.html)

搜索建议词（Suggest 功能）

[Elasticsearch 实现搜索推荐词（C#）](https://www.cnblogs.com/starrynightsky/p/13391809.html)

[基于 Elasticsearch 实现搜索推荐](https://www.jianshu.com/p/4ab3c69e7b19)

[ElasticSearch 使用 completion 实现补全功能](https://blog.csdn.net/qushaming/article/details/90479091)

[Elasticsearch Suggester 详解（自动补全）](https://blog.csdn.net/wwd0501/article/details/80595201)

[Elasticsearch 搜索 Suggest 功能优化](https://www.jianshu.com/p/9e2c6a8e1b54)

[elasticsearch 7.0 新特性之 search as you type](https://www.jianshu.com/p/32b8f8dfa99b)

[模拟实战京东搜索效果（一）](https://www.jianshu.com/p/45752375f903)

[模拟实战京东搜索效果（二）](https://www.jianshu.com/p/ccc855c89071)

安全性

[Meow 攻击删除开放的的 Elasticsearch（及 MongoDB） 索引，建一堆以 Meow 结尾的奇奇怪怪的索引（如：m3egspncll-meow）](https://www.cnblogs.com/NaughtyCat/archive/2020/08/05/meow-attack-delete-unsafe-es-index.html)---- 关闭外网访问端口，或至少修改 ES 默认端口

[用 nginx 给 kibana、elasticsearch 做权限认证](https://blog.csdn.net/u011419453/article/details/39395627)

[集中式日志分析平台 - ELK Stack - 安全解决方案 X-Pack](https://www.jianshu.com/p/a49d93212eca)

[elasticsearch7.8 权限控制和规划（版本 7 开始，x-pack 部分安全功能可以免费使用）](https://www.cnblogs.com/zsql/p/14373370.html)

常用的 es 语句

版本：Elasticsearch 7.9.0

在线 kibana：[http://134.175.121.78:5601/app/dev_tools#/console](http://134.175.121.78:5601/app/dev_tools#/console)

（是我自己的服务器搭建的，请大家友好的体验）

删除索引

DELETE mall.completion

创建索引，并指定 settings

PUT mall.completion

{

 "settings":{

   "analysis":{

     "analyzer":{

        "ik_smart_pinyin":{

          "type":"custom",

          "tokenizer":"ik_smart",

          "filter":["g_pinyin","word_delimiter"]

        },

        "ik_max_word_pinyin":{

          "type":"custom",

          "tokenizer":"ik_max_word",

          "filter":["g_pinyin","word_delimiter"]

        }

      },

     "filter":{

        "g_pinyin":{

          "type":"pinyin",

          "keep_separate_first_letter":false,

          "keep_full_pinyin":true,

          "keep_original":true,

          "limit_first_letter_length":16,

          "lowercase":true,

          "remove_duplicated_term":true

        }

      }

    }

  },

 "mappings": {

   "properties": {

     "kw_completion": {

        "type": "completion"

      },

     "kw_text":{

        "type": "text",

        "analyzer": "ik_smart_pinyin"

      }

    }

  }

}

查看索引设置

GET mall.completion/_settings

查看 mapping 结构

GET mall.completion/_mapping

批量插入数据

POST _bulk/?refresh=true

{"index": { "_index": "mall.completion"}}

{ "kw_completion": " 项目 ","kw_text":" 项目 "}

{"index": { "_index": "mall.completion"}}

{ "kw_completion": " 项目进度 ","kw_text":" 项目进度 "}

{"index": { "_index": "mall.completion"}}

{ "kw_completion": " 项目管理 ","kw_text":" 项目管理 "}

{"index": { "_index": "mall.completion"}}

{ "kw_completion": " 项目进度及调整 汇总.doc_文档 ","kw_text":" 项目进度及调整 汇总.doc_文档 "}

{"index": { "_index": "mall.completion"}}

{ "kw_completion": " 项目 ","kw_text":" 项目 "}

查看指定分词器对文本进行分词的结果

GET mall.completion/_analyze

{

 "analyzer": "ik_smart_pinyin",

 "text": " 很棒的冬天暖心羽绒服 "

}

根据字段的 mapping，进行分词测试

GET mall.completion/_analyze

{

 "field": "kw_text",

 "text": " 很棒的冬天暖心羽绒服 "

}

查询文档

GET mall.completion/_search

{

 "query": {

   "match": {

     "kw_text": " 项目 "

    }

  }

}

查询所有文档

GET mall.completion/_search

{

 "query": {

   "match_all": {}

  }

}

查看文档中的分词结果

GET mall.completion/_doc/CYlJTnUBrvWtEbASfvRa/_termvectors?fields=kw_text

使用 completion 获取搜索补全建议 (前缀搜索)

GET mall.completion/_search

{

   "suggest": {

        "my-completion": {

            "prefix": " 项目 ",

            "completion": {

                "field": "kw_completion",

                "size": 20,

                "skip_duplicates": true

            }

        }

    }

}

获取搜索建议词 （xang 为拼写错误，会建议为：xiang）

GET mall.completion/_search

{

 "suggest": {

    "my-suggestion": {

     "text": "xang",

     "term": {

        "suggest_mode": "missing",

        "field": "kw_text"

      }

    }

  }

}

多字段匹配案例

GET mall.completion/_search

{

 "query":{

   "multi_match": {

     "query": " 米 ",

     "fields": ["name","description","brandName","labelName","menuCategoryNamePath"]

    }

  }

}

查询包含字段 "keyword" 的文档

GET mall.completion/_search

{

 "query":{

   "exists": {

     "field": "keyword"

    }

  }

}

多条件查询语法案例

must        文档 必须 匹配这些条件才能被包含进来。

must_not    文档 必须不 匹配这些条件才能被包含进来。

should      如果满足这些语句中的任意语句，将增加_score ，否则，无任何影响。它们主要用于修正每个文档的相关性得分。

filter      必须 匹配，但它以不评分、过滤模式来进行。这些语句对评分没有贡献，只是根据过滤标准来排除或包含文档。

{

   "bool": {

        "must":     {"match": { "title": "how to make millions"}},

        "must_not": { "match": { "tag":   "spam" }},

        "should": [

            {"match": { "tag": "starred"}}

        ],

        "filter": {

          "bool": {

              "must": [

                  {"range": { "date": { "gte": "2014-01-01"}}},

                  {"range": { "price": { "lte": 29.99}}}

              ],

              "must_not": [

                  {"term": { "category": "ebooks"}}

              ]

          }

        }

    }

}

其他推荐阅读

        [百度搜索的高级搜索技巧](https://jingyan.baidu.com/article/948f5924cff4bcd80ff5f9be.html)

        [ElasticSearch 电商搜索实现（按 "地里坐标" 排序）](https://blog.csdn.net/mushang8923/article/details/103614738)

        [Implementing A Modern E-Commerce Search](https://spinscale.de/posts/2020-06-22-implementing-a-modern-ecommerce-search.html)

        [.Net Core with 微服务 - Elastic APM](https://www.cnblogs.com/kklldog/p/netcore-with-microservices-06.html)

==============================================================================

over，谢谢查阅，觉得文章对你有收获，请多帮推荐。欢迎向我提供更好的资料信息。