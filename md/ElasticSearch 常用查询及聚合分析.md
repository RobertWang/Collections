> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/183816335)

1. ES 中基本概念
-----------

### 1.1 接近实时 (Near Real Time 简称 NRT)

> **Elasticsearch 是一个接近实时的搜索平台**。这意味着，**从索引一个文档直到这个文档能够被搜索到有一个轻微的延迟 (通常是 1 秒内)**  

### 1.2 索引 (index)

> **`一个索引就是一个拥有几分相似特征的文档的集合`**。比如说，你可以有一个客户数据的索引，另一个产品目录的索引，还有一个订单数据的索引。**`一个索引由一个名字来标识(必须全部是小写字母的)`**，**`并且当我们要对这个索引中的文档进行索引、搜索、更新和删除的时候，都要使用到这个名字`**。索引类似于关系型数据库中 Database 的概念。在一个集群中，如果你想，可以定义任意多的索引。  

### 1.3 类型 (type)

> **`一个类型是你的索引的一个逻辑上的分类/分区，其语义完全由你来定`**。在一个索引中，你可以定义一种或多种类型。通常，会为具有一组共同字段的文档定义一个类型。比如说，我们假设你运营一个博客平台并且将你所有的数 据存储到一个索引中。在这个索引中，你可以为用户数据定义一个类型，为博客数据定义另一个类型，当然，也可 以为评论数据定义另一个类型。类型类似于关系型数据库中 Table 的概念。  
> **NOTE: 在 5.x 版本以前可以在一个索引中定义多个类型, 6.x 之后版本也可以使用, 但是不推荐, 在 8.x 版本中彻底移除一个索引中创建多个类型**  

### 1.4 映射 (Mapping)

> **Mapping** 是 ES 中的一个很重要的内容，**`它类似于传统关系型数据中table的schema，用于定义一个索引(index)中的类型(type)的数据的结构`**。 在 ES 中，我们可以手动创建 type(相当于 table) 和 mapping(相关与 schema), 也可以采用默认创建方式。**`在默认配置下，ES可以根据插入的数据自动地创建type及其mapping。 mapping中主要包括字段名、字段数据类型和字段索引类型`**  

### 1.5 文档 (document)

> **`一个文档是一个可被索引的基础信息单元，类似于表中的一条记录`**。比如，你可以拥有某一个员工的文档, 也可以拥有某个商品的一个文档。文档以采用了轻量级的数据交换格式 JSON(Javascript Object Notation) 来表示。  

2. Kibana 的基本操作
---------------

### 2.1 索引 (Index) 的基本操作

```
PUT /dangdang/              创建索引
DELETE /dangdang            删除索引
DELETE /*                   删除所有索引
GET /_cat/indices?v         查看索引信息   类似于关系型数据库中 show databases

```

### 2.2 类型 (type) 的基本操作

### 创建类型

```
1.创建/dangdang索引并创建(product)类型
PUT /dangdang             
{
  "mappings": {
    "product": {
      "properties": {
            "title":    { "type": "text"  },
            "name":     { "type": "text"  },
            "age":      { "type": "integer" },
            "created":  {
                 "type":   "date",
                 "format": "strict_date_optional_time||epoch_millis"
                }
            }
        }
    }
}
注意: 这种方式创建类型要求索引不能存在

```

> Mapping Type: **: text , keyword , date ,integer, long , double , boolean or ip**  

### 查看类型

```
GET /dangdang/_mapping/product # 语法:GET /索引名/_mapping/类型名

```

### 2.3 文档 (document) 的基本操作

### 添加文档

```
PUT /ems/emp/1   #/索引/类型/id
{
  "name":"赵小六",
  "age":23,
  "bir":"2012-12-12",
  "content":"这是一个好一点的员工"
}

```

### 查询文档

```
GET /ems/emp/1  
返回结果:
{
  "_index": "ems",
  "_type": "emp",
  "_id": "1",
  "_version": 1,
  "found": true,
  "_source": {
    "name": "赵小六",
    "age": 23,
    "bir": "2012-12-12",
    "content": "这是一个好一点的员工"
  }
}

```

### 删除文档

```
DELETE /ems/emp/1
{
  "_index": "ems",
  "_type": "emp",
  "_id": "1",
  "_version": 2,
  "result": "deleted", #删除成功
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 1,
  "_primary_term": 1
}

```

### 更新文档

```
1.第一种方式  更新原有的数据
   POST /ems/emp/1/_update
    {
      "doc":{
        "name":"xiaohei"
      }
    }
2.第二种方式  添加新的数据
    POST /ems/emp/1/_update
    {
      "doc":{
        "name":"xiaohei",
        "age":11,
        "dpet":"你好部门"
      }
    }
3.第三种方式 在原来数据基础上更新
    POST /ems/emp/1/_update
    {
      "script": "ctx._source.age += 5"
    }

```

### 批量操作

```
1. 批量索引两个文档
    PUT /ems/emp/_bulk
    {"index":{"_id":"1"}} 
        {"name": "John Doe","age":23,"bir":"2012-12-12"}
    {"index":{"_id":"2"}}  
        {"name": "Jane Doe","age":23,"bir":"2012-12-12"}

2. 更新文档同时删除文档
    POST /ems/emp/_bulk
        {"update":{"_id":"1"}}
            {"doc":{"name":"lisi"}}
        {"delete":{"_id":2}}
        {"index":{}}
            {"name":"xxx","age":23}

注意:批量时不会因为一个失败而全部失败,而是继续执行后续操作,批量在返回时按照执行的状态开始返回

```

3. ES 中高级检索
-----------

### 3.1 检索方式

> ES 官方提供了两中检索方式: **一种是通过 URL 参数进行搜索, 另一种是通过 DSL(Domain Specified Language) 进行搜索**。**官方更推荐使用第二种方式， 第二种方式是基于传递 JSON 作为请求体 (request body) 格式与 ES 进行交互，这种方式更强大，更简洁**。  

### 测试数据

```
1.删除索引
DELETE /ems

2.创建索引并指定类型
PUT /ems
{
  "mappings":{
    "emp":{
      "properties":{
        "name":{
          "type":"keyword"
        },
        "age":{
          "type":"integer"
        },
        "bir":{
          "type":"date"
        },
        "content":{
          "type":"text"
        },
        "address":{
          "type":"keyword"
        }
      }
    }
  }
}

3.插入测试数据
PUT /ems/emp/_bulk
  {"index":{}}
  {"name":"小黑","age":23,"bir":"2012-12-12","content":"为开发团队选择一款优秀的MVC框架是件难事儿，在众多可行的方案中决择需要很高的经验和水平","address":"北京"}
  {"index":{}}
  {"name":"王小黑","age":24,"bir":"2012-12-12","content":"Spring 框架是一个分层架构，由 7 个定义良好的模块组成。Spring 模块构建在核心容器之上，核心容器定义了创建、配置和管理 bean 的方式","address":"上海"}
  {"index":{}}
  {"name":"张小五","age":8,"bir":"2012-12-12","content":"Spring Cloud 作为Java 语言的微服务框架，它依赖于Spring Boot，有快速开发、持续交付和容易部署等特点。Spring Cloud 的组件非常多，涉及微服务的方方面面，井在开源社区Spring 和Netflix 、Pivotal 两大公司的推动下越来越完善","address":"无锡"}
  {"index":{}}
  {"name":"win7","age":9,"bir":"2012-12-12","content":"Spring的目标是致力于全方位的简化Java开发。 这势必引出更多的解释， Spring是如何简化Java开发的？","address":"南京"}
  {"index":{}}
  {"name":"梅超风","age":43,"bir":"2012-12-12","content":"Redis是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API","address":"杭州"}
  {"index":{}}
  {"name":"张无忌","age":59,"bir":"2012-12-12","content":"ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口","address":"北京"}

```

### 3.2 URL 检索

> **GET /ems/emp/_search?q=*&sort=age:asc**  
> ​ _search 搜索的 API ​ q=* 匹配所有文档 ​ sort 以结果中的指定字段排序  
> ​ & 多个查询条件用 & 符号连接  

### 3.3 DSL 检索

> **NOTE: 以下重点讲解 DSL 语法**  

```
GET /ems/emp/_search
{
    "query": {"match_all": {}},
    "sort": [
        {
            "age": {
                "order": "desc"
            }
        }
    ]
}

```

### 3.4 DSL 高级检索 (Query)

### 0. 查询所有 (match_all)

> **match_all 关键字:** 返回索引中的全部文档  

```
GET /ems/emp/_search
{
    "query": { "match_all": {} }
}

```

### 1. 查询结果中返回指定条数 (size)

> **size 关键字**: 指定查询结果中返回指定条数。 **默认返回值 10 条**  

```
GET /ems/emp/_search
{
    "query": { "match_all": {} },
    "size": 1
}

```

### 2. 分页查询 (from)

> **from 关键字**: 用来指定起始返回位置，和 **size 关键字连用可实现分页效果**，size 表示从起始位置开始的文档数量；类似于 mysql 中的 select * from tablename limit 1, 2;  
> ES 默认的分页深度是 10000，也就是 from+size 超过了 10000 就会报错，ES 内部是通过 index.max_result_window 这个参数控制分页深度的，可进行修改。分页越深，ES 的处理开销越大，占用内存越大。  

```
GET /ems/emp/_search
{
      "query": {"match_all": {}},
      "sort": [
        {
          "age": {
            "order": "desc"
          }
        }
      ],
      "size": 2, 
      "from": 1
}

```

### 3. 查询结果中返回指定字段 (_source)

> **_source 关键字**: 是一个数组, 在数组中用来指定展示那些字段  

```
GET /ems/emp/_search
{
      "query": { "match_all": {} },
      "_source": ["name", "age"]
}

```

### 4. 关键词查询 (term)

> **term 关键字**: 用来使用关键词查询  

```
GET /ems/emp/_search
{
  "query": {
    "term": {
      "address": {
        "value": "北京"
      }
    }
  }
}

```

> **NOTE1: 通过使用 term 查询得知 ES 中默认使用分词器为标准分词器 (StandardAnalyzer), 标准分词器对于英文单词分词, 对于中文单字分词**。  
> **NOTE2: 通过使用 term 查询得知, 在 ES 的 Mapping Type 中 keyword , date ,integer, long , double , boolean or ip 这些类型不分词**，**只有 text 类型分词**。  

### 5. 范围查询 (range)

> **range 关键字**: 用来指定查询指定范围内的文档  

```
GET /ems/emp/_search
{
  "query": {
    "range": {
      "age": {
        "gte": 8,
        "lte": 30
      }
    }
  }
}

```

### 6. 前缀查询 (prefix)

> **prefix 关键字**: 用来检索含有指定前缀的关键词的相关文档  

```
GET /ems/emp/_search
{
  "query": {
    "prefix": {
      "content": {
        "value": "redis"
      }
    }
  }
}

```

### 7. 通配符查询 (wildcard)

> **wildcard 关键字**: 通配符查询 **? 用来匹配一个任意字符 * 用来匹配多个任意字符**  

```
GET /ems/emp/_search
{
  "query": {
    "wildcard": {
      "content": {
        "value": "re*"
      }
    }
  }
}

```

### 8. 多 id 查询 (ids)

> **ids 关键字** : 值为数组类型, 用来根据一组 id 获取多个对应的文档  

```
GET  /ems/emp/_search
{
  "query": {
    "ids": {
      "values": ["lg5HwWkBxH7z6xax7W3_","lQ5HwWkBxH7z6xax7W3_"]
    }
  }
}

```

### 9. 模糊查询 (fuzzy)

> **fuzzy 关键字**: 用来模糊查询含有指定关键字的文档 注意: 允许出现的错误必须在 0-2 之间  

```
GET /ems/emp/_search
{
  "query": {
    "fuzzy": {
      "content":"spoong"
    }
  }
}

# 注意: 最大编辑距离为 0 1 2
如果关键词为2个长度      0..2 must match exactly  必须完全匹配
如果关键词长度3..5之间  one edit allowed    允许一个失败
如果关键词长度>5   two edits allowed       最多允许两个错误

```

### 10. 布尔查询 (bool)

> **bool 关键字**: 用来组合多个条件实现复杂查询 boolb 表达式查询  
> ​ **must: 相当于 && 同时成立**  
> ​ **should: 相当于 || 成立一个就行**  
> ​ **must_not: 相当于! 不能满足任何一个**  

```
GET /ems/emp/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "age": {
              "gte": 0,
              "lte": 30
            }
          }
        }
      ],
      "must_not": [
        {"wildcard": {
          "content": {
            "value": "redi?"
          }
        }}
      ]
    }
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    }
  ]
}

```

### 11. 高亮查询 (highlight)

> **highlight 关键字**: 可以让符合条件的文档中的关键词高亮  

```
GET /ems/emp/_search
{
  "query": {
    "term": {
      "content": {
        "value": "redis"
      }
    }
  },
  "highlight": {
    "fields": {
      "*": {}
    }
  }
}

```

> **自定义高亮 html 标签**: 设置高亮 html 标签，默认是_标签，可以在 highlight 中使用`pre_tags`和`post_tags`属性自定义高亮显示的 html 标签，去替代默认的 em 标签。_  

```
GET /ems/emp/_search
{
  "query":{
    "term":{
      "content":"spring"
    }
  },
  "highlight": {
    "pre_tags": ["<span style='color:red'>"],
    "post_tags": ["</span>"],
    "fields": {
      "*":{}
    }
  }
}

```

> _多字段高亮 使用`require_field_match`设置为 false，开启多个字段高亮，默认为 true。_  

```
GET /ems/emp/_search
{
  "query":{
    "term":{
      "content":"spring"
    }
  },
  "highlight": {
    "pre_tags": ["<span style='color:red'>"],
    "post_tags": ["</span>"],
    "require_field_match":false,
    "fields": {
      "*":{}
    }
  }
}

```

### _12. 多字段查询 (multi_match)_

> _`注意:使用这种方式进行查询时,为了更好获取搜索结果,在查询过程中先将查询条件根据当前的分词器分词之后进行查询`_  

```
GET /ems/emp/_search
{
  "query": {
    "multi_match": {
      "query": "中国",
      "fields": ["name","content"] #这里写要检索的指定字段
    }
  }
}

```

### _13. 多字段分词查询 (query_String)_

> _`注意:使用这种方式进行查询时,为了更好获取搜索结果,在查询过程中先将查询条件根据当前的分词器分词之后进行查询`_  

```
GET /dangdang/book/_search
{
  "query": {
    "query_string": {
      "query": "中国声音",
      "analyzer": "ik_max_word", 
      "fields": ["name","content"]
    }
  }
}

```

_4. (过滤查询) Filter Query_
------------------------

### _4.1 过滤查询_

> _其实准确来说，ES 中的查询操作分为 2 种: `查询(query)`和`过滤(filter)`。`查询即是之前提到的query查询，它 (查询)默认会计算每个返回文档的得分，然后根据得分排序`。`而过滤(filter)只会筛选出符合的文档，并不计算 得分，且它可以缓存文档 。所以，单从性能考虑，过滤比查询更快`。_  
> _换句话说，过滤适合在大范围筛选数据，而查询则适合精确匹配数据。一般应用时， 应先使用过滤操作过滤数据， 然后使用查询匹配数据。_  

### _4.2 过滤语法_

```
GET /ems/emp/_search
{
  "query": {
    "bool": {
      "must": [
        {"match_all": {}}
      ],
      "filter": {
        "range": {
          "age": {
            "gte": 10
          }
        }
      }
    }
  }
}

```

> _**NOTE: 在执行 filter 和 query 时, 先执行 filter 在执行 query**{}_  
> _**NOTE:Elasticsearch 会自动缓存经常使用的过滤器，以加快性能。**_  

### _4.3 常见的过滤器类型_

### _term 、 terms_

_含义与查询时一致，term 用于精确匹配，terms 用于多词条匹配，过滤上使用没有很大区别_

```
GET /ems/emp/_search   # 使用term过滤
{
  "query": {
    "bool": {
      "must": [
        {"term": {
          "name": {
            "value": "小黑"
          }
        }}
      ],
      "filter": {
        "term": {
          "content":"spring"
        }
      }
    }
  }
}
GET /ems/emp/_search  #使用terms过滤
{
  "query": {
    "bool": {
      "must": [
        {"term": {
          "name": {
            "value": "梅超风"
          }
        }}
      ],
      "filter": {
        "terms": {
          "content":[
              "redis",
              "开源"
            ]
        }
      }
    }
  }
}

```

### _ranage filter_

```
GET /ems/emp/_search
{
  "query": {
    "bool": {
      "must": [
        {"term": {
          "name": {
            "value": "中国"
          }
        }}
      ],
      "filter": {
        "range": {
          "age": {
            "gte": 7,
            "lte": 20
          }
        }
      }
    }
  }
}

```

### _exists filter_

> _**过滤存在指定字段, 获取字段不为空的索引记录使用**_  

```
GET /ems/emp/_search
{
  "query": {
    "bool": {
      "must": [
        {"term": {
          "name": {
            "value": "中国"
          }
        }}
      ],
      "filter": {
        "exists": {
          "field":"aaa"
        }
      }
    }
  }
}

```

### _ids filter_

> _**过滤含有指定字段的索引记录**_  

```
GET /ems/emp/_search
{
  "query": {
    "bool": {
      "must": [
        {"term": {
          "name": {
            "value": "中国"
          }
        }}
      ],
      "filter": {
        "ids": {
          "values": ["1","2","3"]
        }
      }
    }
  }
}

```

_5. （Metric 聚合）聚合查询_
--------------------

### _5.1 聚合的概念_

_官方对聚合有四个关键字：`Metric(指标)、Bucketing(桶)、Matrix(矩阵)、Pipeline(管道)`，在查询请求体中以 aggregations 语法来定义聚合分析，也可简写成 aggs_

```
Metric(指标)：指标分析类型，如计算最大值、最小值、平均值等（对桶内的文档进行聚合分析的操作）
Bucket(桶)：分桶类型，类似sql中的group by语法（满足特定条件的文档的集合）
Pipeline(管道)：管道分析类型，基于上一级的聚合分析结果进行再分析
Matrix(矩阵)：矩阵分析类型（聚合是一种面向数值型的聚合，用于计算一组文档字段中的统计信息）

```

### _5.2 指标（Metric）详解_

_Metric 聚合分析分为单值分析和多值分析两类_

```
#1、单值分析，只输出一个分析结果
min, max, avg, sum, cardinality

#2、多值分析，输出多个分析结果
stats, extended_stats, percentile_rank, top hits

```

### _0. 插入测试数据_

```
#创建索引
PUT /employees/
{
  "mappings": {
    "employees": {
      "properties": {
        "age": {
          "type": "integer"
        },
        "gender": {
          "type": "keyword"
        },
        "job": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 50
            }
          }
        },
        "name": {
          "type": "keyword"
        },
        "salary": {
          "type": "integer"
        }
      }
    }
  }
}


#添加数据
PUT /employees/employees/_bulk
{ "index" : {  "_id" : "1" } }
{ "name" : "Emma","age":32,"job":"Product Manager","gender":"female","salary":35000 }

```

### _1. Avg(平均值)_

> _**avg 关键字:** 计算从聚合文档中提取的数值的平均值_  

```
GET /employees/employees/_search
{
  "size": 0,
  "aggs": {
    "avg_salary": {
      "avg": {
        "field": "salary"
      }
    }
  }
}

```

### _2. 查询最低、最高工资_

> _**min, max 关键字**_  

```
GET /employees/employees/_search
{
  "size": 0,
  "aggs": {
    "min_salary": {
      "min": {
        "field": "salary"
      }
    },
    "max_salary":{
      "max": {
        "field": "salary"
      }
    }
  }
}

```

### _3. 一个聚合输出多值_

> _**stats 关键字**: 统计，请求后会直接显示各种聚合结果_  

```
GET /employees/employees/_search
{
  "size": 0,
  "aggs": {
    "stats_salary": {
      "stats": {
        "field": "salary"
      }
    }
  }
}

```

### _4. 唯一值（Cardinality）_

> _**cardinality 关键字**: 求唯一值，即不重复的字段有多少（相当于 sql 中的 distinct）_  

```
GET /employees/employees/_search
{
  "size": 0,
  "aggs": {
    "cardinate": {
      "cardinality": {
        "field": "job.keyword"    #共有多少种工作类型，把job的类型为keyword类型，这样就不会分词，把它当作一个整体
      }
    }
  }
}

```

### _5. 百分比（Percentiles）_

> _**Percentiles 关键字**: 对指定字段的值按从小到大累计每个值对应的文档数的占比，返回指定占比比例对应的值，默认按照 [1,5,25,50,75,95,99] 来统计_  

```
GET /employees/employees/_search
{
  "size": 0,
  "aggs": {
    "load_time_outlier": {
      "percentiles": {
        "field": "salary",
        "percents": [
          50,                   #查看中位数薪资
          99
        ],
        "keyed":false
      }
    }
  }
}

```

_**默认情况下，keyed 标志设置为 true，它将唯一的字符串键与每个存储桶相关联，并将范围作为哈希而不是数组返回**_

### _6. 多层嵌套_

> _**Top Hits 关键字**: 一般用于分桶后获取该桶内匹配前 n 的文档列表_  

```
GET /employees/employees/_search
{
  "size": 0,
  "aggs": {
    "job_gender_stats": {
      "terms": {
        "field": "job.keyword"    根据工作类型聚合
      },
      "aggs": {
        "gender_stats": {
          "terms": {
            "field": "gender"     根据性别聚合
          },
          "aggs": {
            "salary_stats": {
              "top_hits": {
                "sort": [
                  {
                    "salary": {
                      "order": "desc"     #根据薪资排序
                    }
                  }
                ],
                "size": 2     #取最高的前2
              }
            }
          }
        }
      }
    }
  }
}

```

_6. （Bucket 聚合）聚合查询_
--------------------

### _6.1 Bucket Aggregations_

_**Bucket** 可以理解为一个桶，它会遍历文档中的内容，凡是符合某一要求的就放在一个桶中，分桶相当于 sql 中的 group by, 关键字有 Terms Aggregation，Filter Aggregation，Histogram Aggregation， Date Aggregation_

```
#创建索引类型
PUT /cars
{
  "mappings": {
    "cars": {
      "properties": {
        "price": {
          "type": "long"
        },
        "color": {
          "type": "keyword"
        },
        "brand": {
          "type": "keyword"
        },
        "sellTime": {
          "type": "date"
        }
      }
    }
  }
}


#添加数据
POST /cars/cars/_bulk
{ "index": {}}
{ "price" : 80000, "color" : "red", "brand" : "BMW", "sellTime" : "2014-01-28" }
{ "index": {}}
{ "price" : 85000, "color" : "green", "brand" : "BMW", "sellTime" : "2014-02-05" }
{ "index": {}}
{ "price" : 120000, "color" : "green", "brand" : "Mercedes", "sellTime" : "2014-03-18" }
{ "index": {}}
{ "price" : 105000, "color" : "blue", "brand" : "Mercedes", "sellTime" : "2014-04-02" }
{ "index": {}}
{ "price" : 72000, "color" : "green", "brand" : "Audi", "sellTime" : "2014-05-19" }
{ "index": {}}
{ "price" : 60000, "color" : "red", "brand" : "Audi", "sellTime" : "2014-06-05" }
{ "index": {}}
{ "price" : 40000, "color" : "red", "brand" : "Audi", "sellTime" : "2014-07-01" }
{ "index": {}}
{ "price" : 35000, "color" : "blue", "brand" : "Honda", "sellTime" : "2014-08-12" }

```

### _6.2. Terms Aggregation_

> _**Terms Aggregation 关键字:** 根据某一项的每个唯一的值来聚合_  

```
GET /cars/cars/_search
{
  "aggs": {
    "car_brand": {
      "terms": {
        "field": "brand"
      }
    }
  }
}

#分桶后只显示文档数量的前3的桶
GET /cars/cars/_search
{
  "aggs": {
    "car_brand": {
      "terms": {
        "field": "brand",
        "size": 3
      }
    }
  }
}

#分桶后排序
GET /cars/cars/_search
{
  "aggs": {
    "car_brand": {
      "terms": {
        "field": "brand",
        "order": {
          "_count": "asc"
        }
      }
    }
  }
}

#显示文档数量大于3的桶
GET /cars/cars/_search
{
  "aggs": {
    "brands_max_num": {
      "terms": {
        "field": "brand",
        "min_doc_count": 3
      }
    }
  }
}

#使用精确指定的词条进行分桶
GET /cars/cars/_search
{
  "aggs": {
    "brand_cars": {
      "terms": {
        "field": "brand",
        "include": ["BMW", "Audi"]
      }
    }
  }
}

```

### _6.3. Filter Aggregation_

> _**Filter Aggregation 关键字:** 指具体的域和具体的值，可以在 Terms Aggregation 的基础上进行了过滤，只对特定的值进行了聚合_  

```
#过滤获取品牌为BMW的桶，并求该桶平均值
GET /cars/cars/_search
{
  "aggs": {
    "car_brands": {
      "filter": {
        "term": {
          "brand": "BMW"
        }
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "price"
          }
        }
      }
    }
  }
}

```

> _**Filters Aggregation 关键字:** Filter Aggregation 只能指定一个过滤条件，响应也只是单个桶。如果要对特定多个值进行聚合，使用 Filters Aggragation_  

```
#过滤获取品牌为BMW的或color为绿色的桶
GET /cars/cars/_search
{
  "aggs": {
    "cars": {
      "filters": {
        "filters": {
          "colorBucket":{
            "match":{
              "color":"red"
            }
          },
          "brandBucket":{
            "match":{
              "brand":"Audi"
            }
          }
        }
      }
    }
  }
}

```

### _6.4. Histogram Aggregation_

> _**Histogram Aggregation 关键字:** Histogram 与 Terms 聚合类似，都是数据分组，区别是 Terms 是按照 Field 的值分组，而 Histogram 可以按照指定的间隔对 Field 进行分组_  

```
#根据价格区间为10000分桶
GET /cars/cars/_search
{
  "aggs": {
    "prices": {
      "histogram": {
        "field": "price",
        "interval": 10000
      }
    }
  }
}

#根据价格区间为10000分桶，同时如果桶中没有文档就不显示桶
GET /cars/cars/_search
{
  "aggs": {
    "prices": {
      "histogram": {
        "field": "price",
        "interval": 10000,
        "min_doc_count": 1
      }
    }
  }
}

```

### _6.5. Range Aggregation_

> _**Range Aggregation 关键字:** 根据用户传递的范围参数作为桶，进行相应的聚合。在同一请求中，请求传递多组范围，每组范围作为一个桶_  

```
#根据价格区间分桶
GET /cars/cars/_search
{
  "aggs": {
    "prices_range": {
      "range": {
        "field": "price",
        "ranges": [
          {
            "to":50000
          },
          {
            "from": 50000,
            "to": 80000
          },
          {
            "from": 80000
          }
        ]
      }
    }
  }
}

#也可以指定key的名称
GET /cars/cars/_search
{
  "aggs": {
    "prices_range": {
      "range": {
        "field": "price",
        "ranges": [
          {
            "key": "<50000", 
            "to":50000
          },
          {
            "key": "50000~80000", 
            "from": 50000,
            "to": 80000
          },
          {
            "key": ">80000", 
            "from": 80000
          }
        ]
      }
    }
  }
}

```

### _6.5. Date Aggregation_

> _**Date Aggregation 关键字:** 分为 Date Histogram Aggregation 和 Date Range Aggregation_  

### _1. Date Histogram_

> _**Date Histogram 关键字:** 针对时间格式数据的直方图聚合，基本特性与 Histogram Aggregation 一致_  

```
#按月分桶显示每个月的销量
GET /cars/cars/_search
{
  "aggs": {
    "sales_over_time": {
      "date_histogram": {
        "field": "sellTime",
        "interval": "month",
        "format": "yyyy-MM-dd"
      }
    }
  }
}

```

### _2. Date Range_

> _**Date Range 关键字:** 针对时间格式数据的直范围聚合，基本特性与 Range Aggregation 一致_  

```
GET /cars/cars/_search
{
  "aggs": {
    "range": {
      "date_range": {
        "field": "sellTime",
        "format": "yyyy", 
        "ranges": [
          {
            "from": "2014",
            "to": "2019"
          }
        ]
      }
    }
  }
}

```