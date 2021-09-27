> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/shoufeng/p/11290669.html) 目录

*   [1 普通聚合分析](#1--普通聚合分析)
    *   [1.1 直接聚合统计](#11--直接聚合统计)
    *   [1.2 先检索, 再聚合](#12--先检索-再聚合)
    *   [1.3 扩展: fielddata 和 keyword 的聚合比较](#13--扩展-fielddata和keyword的聚合比较)
*   [2 嵌套聚合](#2--嵌套聚合)
    *   [2.1 先分组, 再聚合统计](#21--先分组-再聚合统计)
    *   [2.2 先分组, 再统计, 最后排序](#22--先分组-再统计-最后排序)
    *   [2.3 先分组, 组内再分组, 然后统计、排序](#23--先分组-组内再分组-然后统计、排序)
*   [版权声明](#版权声明)

1 普通聚合分析
========

1.1 直接聚合统计
----------

**(1) 计算每个 tag 下的文档数量, 请求语法:**

```
GET book_shop/it_book/_search
{
    "size": 0,    			// 不显示命中(hits)的所有文档信息
    "aggs": {
        "group_by_tags": {	// 聚合结果的名称, 需要自定义(复制时请去掉此注释)
            "terms": {
                "field": "tags"
            }
        }
    }
}


```

**(2) 发生错误:**

说明: 索引 book_shop 的 mapping 映射是 ES 自动创建的, 它把 tag 解析成了 text 类型, 在发起对 tag 的聚合请求后, 将抛出如下错误:

```
{
    "error": {
        "root_cause": [
            {
                "type": "illegal_argument_exception",
                "reason": "Fielddata is disabled on text fields by default. Set fielddata=true on [tags] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead."
            }
        ],
        "type": "search_phase_execution_exception",
        "reason": "all shards failed",
        "phase": "query",
        "grouped": true,
        "failed_shards": [......]
    },
    "status": 400
}


```

**(3) 错误分析:**

> 错误信息: `Set fielddata=true on [xxxx] ......`  
> 错误分析: 默认情况下, Elasticsearch 对 text 类型的字段 (field) 禁用了 fielddata;  
> text 类型的字段在创建索引时会进行分词处理, 而聚合操作必须基于字段的原始值进行分析;  
> 所以如果要对 text 类型的字段进行聚合操作, 就需要存储其原始值 —— 创建 mapping 时指定`fielddata=true`, 以便通过反转倒排索引 (即正排索引) 将索引数据加载至内存中.

**(4) 解决方案一: 对 text 类型的字段开启 fielddata 属性:**

*   将要分组统计的 text field(即 tags) 的 fielddata 设置为 true:
    
    ```
    PUT book_shop/_mapping/it_book
    {
        "properties": {
            "tags": {
                "type": "text",
                "fielddata": true
            }
        }
    }
    
    
    ```
    
*   可参考官方文档进行设置:  
    [https://www.elastic.co/guide/en/elasticsearch/reference/6.6/fielddata.html](https://www.elastic.co/guide/en/elasticsearch/reference/6.6/fielddata.html). 成功后的结果如下:
    
    ```
    {
      "acknowledged": true
    }
    
    
    ```
    
*   再次统计, 得到的结果如下:
    
    ```
    {
        "took": 153,
        "timed_out": false,
        "_shards": {
            "total": 5,
            "successful": 5,
            "skipped": 0,
            "failed": 0
        },
        "hits": {
            "total": 4,
            "max_score": 0.0,
            "hits": []
        },
        "aggregations": {
            "group_by_tags": {
                "doc_count_error_upper_bound": 0,
                "sum_other_doc_count": 6,
                "buckets": [
                    {
                        "key": "java",
                        "doc_count": 3
                    },
                    {
                        "key": "程",
                        "doc_count": 2
                    },
                    ......
                ]
            }
        }
    }
    
    
    ```
    

**(5) 解决方法二: 使用内置 keyword 字段:**

*   开启 fielddata 将占用大量的内存.
    
*   Elasticsearch 5.x 版本开始支持通过 text 的内置字段 keyword 作精确查询、聚合分析:
    
    ```
    GET shop/it_book/_search
    {
        "size": 0,
        "aggs": {
            "group_by_tags": {
                "terms": {
                    "field": "tags.keyword"	// 使用text类型的内置keyword字段
        	    }
        	}
        }
    }
    
    
    ```
    

1.2 先检索, 再聚合
------------

**(1) 统计 name 中含有 “jvm” 的图书中每个 tag 的文档数量, 请求语法:**

```
GET book_shop/it_book/_search
{
    "query": {
        "match": { "name": "jvm" }
    }, 
    "aggs": {
        "group_by_tags": {  // 聚合结果的名称, 需要自定义. 下面使用内置的keyword字段: 
            "terms": { "field": "tags.keyword" }
        }
    }
}


```

**(2) 响应结果:**

```
{
  "took" : 7,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 0.64072424,
    "hits" : [
      {
        "_index" : "book_shop",
        "_type" : "it_book",
        "_id" : "2",
        "_score" : 0.64072424,
        "_source" : {
          "name" : "深入理解Java虚拟机：JVM高级特性与最佳实践",
          "author" : "周志明",
          "category" : "编程语言",
          "desc" : "Java图书领域公认的经典著作",
          "price" : 79.0,
          "date" : "2013-10-01",
          "publisher" : "机械工业出版社",
          "tags" : [
            "Java",
            "虚拟机",
            "最佳实践"
          ]
        }
      }
    ]
  },
  "aggregations" : {
    "group_by_tags" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "Java",
          "doc_count" : 1
        },
        {
          "key" : "最佳实践",
          "doc_count" : 1
        },
        {
          "key" : "虚拟机",
          "doc_count" : 1
        }
      ]
    }
  }
}


```

1.3 扩展: fielddata 和 keyword 的聚合比较
---------------------------------

*   为某个 text 类型的字段开启 fielddata 字段后, 聚合分析操作会对这个字段的所有分词分别进行聚合, 获得的结果大多数情况下并不符合我们的需求.
    
*   **使用 keyword 内置字段, 不会对相关的分词进行聚合, 结果可能更有用.**
    

—— **推荐使用 text 类型字段的内置 keyword 进行聚合操作.**

2 嵌套聚合
======

2.1 先分组, 再聚合统计
--------------

**(1) 先按 tags 分组, 再计算每个 tag 下图书的平均价格, 请求语法:**

```
GET book_shop/it_book/_search
{
    "size": 0, 
    "aggs": {
        "group_by_tags": {
            "terms": { "field": "tags.keyword" },
            "aggs": {
                "avg_price": {
                    "avg": { "field": "price" }
                }
            }
        }
    }
}


```

**(2) 响应结果:**

```
  "hits" : {
    "total" : 3,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_tags" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "Java",
          "doc_count" : 3,
          "avg_price" : {
            "value" : 102.33333333333333
          }
        },
        {
          "key" : "编程语言",
          "doc_count" : 2,
          "avg_price" : {
            "value" : 114.0
          }
        },
        ......
      ]
    }
  }


```

2.2 先分组, 再统计, 最后排序
------------------

**(1) 计算每个 tag 下图书的平均价格, 再按平均价格降序排序, 查询语法:**

```
GET book_shop/it_book/_search
{
    "size": 0,
    "aggs": {
        "all_tags": {
            "terms": {
                "field": "tags.keyword", 
                "order": { "avg_price": "desc" } // 根据下述统计的结果排序
            },
            "aggs": {
                "avg_price": {
                    "avg": { "field": "price" }
                }
            }
        }
    }
}


```

**(2) 响应结果:**

与 #2.1 节内容相似, 区别在于按照价格排序显示了.

2.3 先分组, 组内再分组, 然后统计、排序
-----------------------

**(1) 先按价格区间分组, 组内再按 tags 分组, 计算每个 tags 组的平均价格, 查询语法:**

```
GET book_shop/it_book/_search
{
    "size": 0, 
    "aggs": {
        "group_by_price": {
            "range": {
                "field": "price", 
                "ranges": [
                    { "from": 00,  "to": 100 },
                    { "from": 100, "to": 150 }
                ]
            }, 
            "aggs": {
                "group_by_tags": {
                    "terms": { "field": "tags.keyword" }, 
                    "aggs": {
                        "avg_price": {
                            "avg": { "field": "price" }
                        }
                    }
                }
            }
        }
    }
}


```

**(2) 响应结果:**

```
  "hits" : {
    "total" : 3,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_price" : {
      "buckets" : [
        {
          "key" : "0.0-100.0",    // 区间0.0-100.0
          "from" : 0.0,
          "to" : 100.0,
          "doc_count" : 1,        // 共查找到了3条文档
          "group_by_tags" : {     // 对tags分组聚合
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
              {
                "key" : "Java",
                "doc_count" : 1,
                "avg_price" : {
                  "value" : 79.0
                }
              },
              ......
            ]
          }
        },
        {
          "key" : "100.0-150.0",
          "from" : 100.0,
          "to" : 150.0,
          "doc_count" : 2,
          "group_by_tags" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
              {
                "key" : "Java",
                "doc_count" : 2,
                "avg_price" : {
                  "value" : 114.0
                }
              },
              ......
              }
            ]
          }
        }
      ]
    }
  }


```

> 版权声明
> ====
> 
> 作者: [马瘦风 (https://healchow.com)](https://healchow.com)
> 
> 出处: 博客园 [马瘦风的博客 (https://www.cnblogs.com/shoufeng)](https://www.cnblogs.com/shoufeng)
> 
> 感谢阅读, 如果文章有帮助或启发到你, 点个 [好文要顶👆] 或 [推荐👍] 吧😜
> 
> 本文版权归博主所有, 欢迎转载, 但 **[必须在文章页面明显位置标明原文链接]**, 否则博主保留追究相关人员法律责任的权利.