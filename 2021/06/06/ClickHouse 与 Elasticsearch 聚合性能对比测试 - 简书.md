> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/0defb5471bc6)

概述
==

Elasticsearch 以其优秀的分布式架构与全文搜索引擎等特点在机器数据的存储、分析领域广为使用，但随着数据量的增长，其聚合分析性能已无法满足业务需求。而 ClickHouse 作为一个高性能的 OLAP 列式数据库管理系统有望解决这一痛点。

本文是对 ClickHouse 与 Elasticsearch 聚合性能的简单对比测试。主要关注查询语句的响应时间，暂不考虑资源占用情况。

测试环境
====

<table><thead><tr><th>组件</th><th>版本</th><th>CPU</th><th>内存</th></tr></thead><tbody><tr><td>ClickHouse</td><td>7.9.0</td><td>4C</td><td>8G</td></tr><tr><td>Elasticsearch</td><td>20.11.4.13</td><td>4C</td><td>8G</td></tr></tbody></table>

使用 ClickHouse 官方提供的测试数据集，共 67G，约 6 亿行。

![](http://upload-images.jianshu.io/upload_images/7617337-c52c42ab5441d306.png) 测试数据集. png

其中，ClickHouse 使用 LO_ORDERDATE 字段作为分区键，使用 LO_ORDERDATE, LO_ORDERKEY 作为排序键。

测试内容
====

某字段出现次数 TOP 10
--------------

```
# ClickHouse
SELECT LO_SHIPMODE,COUNT() FROM lineorder GROUP BY LO_SHIPMODE ORDER BY COUNT() DESC LIMIT 10

# Elasticsearch
GET lineorder/_search
{
  "aggs": {
    "1": {
      "terms": {
        "field": "LO_SHIPMODE.keyword",
        "order": {
          "_count": "desc"
        },
        "size": 10
      }
    }
  },
  "size": 0
}
```

某字段按年进行计数
---------

```
# ClickHouse
SELECT toYear(LO_ORDERDATE),COUNT() FROM lineorder GROUP BY toYear(LO_ORDERDATE) FORMAT PrettyCompactMonoBlock

# Elasticsearch
GET lineorder/_search
{
  "aggs": {
    "2": {
      "date_histogram": {
        "field": "LO_ORDERDATE",
        "calendar_interval":"1y",
        "format":"yyyy-MM-dd"
      }
    }
  },
  "size": 0
}
```

多个字段按年进行统计
----------

```
# ClickHouse
SELECT LO_ORDERDATE,LO_ORDERKEY,LO_SHIPMODE,LO_ORDERPRIORITY,LO_COMMITDATE FROM lineorder WHERE LO_ORDERDATE >= '1992-01-01' AND LO_ORDERDATE < '1993-01-01' ORDER BY LO_ORDERDATE  LIMIT 500

# Elasticsearch
GET lineorder/_search
{
  "size": 500,
  "sort": [
    {
      "timestamp": {
        "order": "desc",
        "unmapped_type": "boolean"
      }
    }
  ],
  "query": {
    "bool": {
      "must": [],
      "filter": [
        {
          "match_all": {}
        },
        {
          "match_all": {}
        },
        {
          "range": {
            "LO_ORDERDATE": {
              "gte": "1992-01-01",
              "lte": "1993-01-01",
              "format": "strict_date_optional_time"
            }
          }
        }
      ],
      "should": [],
      "must_not": []
    }
  }
}
```

基于时间的多字段聚合
----------

```
# ClickHouse
SELECT toYear(LO_ORDERDATE),LO_SHIPMODE,COUNT() FROM lineorder GROUP BY toYear(LO_ORDERDATE),LO_SHIPMODE ORDER BY toYear(LO_ORDERDATE) FORMAT PrettyCompactMonoBlock

# Elasticsearch
GET lineorder/_search
{
  "aggs": {
    "3": {
      "terms": {
        "field": "LO_SHIPMODE.keyword",
        "order": {
          "_count": "desc"
        },
        "size": 10
      },
      "aggs": {
        "2": {
          "date_histogram": {
            "field": "LO_ORDERDATE",
            "calendar_interval": "1y",
            "time_zone": "Asia/Shanghai",
            "min_doc_count": 1
          }
        }
      }
    }
  },
  "size": 0
}
```

基于时间的多字段聚合
----------

```
# ClickHouse
SELECT toYear(LO_ORDERDATE),LO_SHIPMODE,COUNT() FROM lineorder GROUP BY toYear(LO_ORDERDATE),LO_SHIPMODE ORDER BY toYear(LO_ORDERDATE) FORMAT PrettyCompactMonoBlock

# Elasticsearch
GET lineorder/_search
{
  "aggs": {
    "3": {
      "terms": {
        "field": "LO_SHIPMODE.keyword",
        "order": {
          "_count": "desc"
        },
        "size": 10
      },
      "aggs": {
        "2": {
          "date_histogram": {
            "field": "LO_ORDERDATE",
            "calendar_interval": "1y",
            "time_zone": "Asia/Shanghai",
            "min_doc_count": 1
          }
        }
      }
    }
  },
  "size": 0
}
```

聚合嵌套（非时间字段）
-----------

```
# ClickHouse
SELECT LO_SHIPMODE,COUNT(LO_SHIPMODE),LO_ORDERPRIORITY,COUNT(LO_ORDERPRIORITY) FROM lineorder GROUP BY LO_SHIPMODE,LO_ORDERPRIORITY ORDER BY COUNT(LO_SHIPMODE),COUNT(LO_ORDERPRIORITY) LIMIT 5 BY LO_SHIPMODE,LO_ORDERPRIORITY

# Elasticsearch
GET lineorder/_search
{
  "aggs": {
    "2": {
      "terms": {
        "field": "LO_SHIPMODE.keyword",
        "order": {
          "_count": "desc"
        },
        "size": 5
      },
      "aggs": {
        "3": {
          "terms": {
            "field": "LO_ORDERPRIORITY.keyword",
            "order": {
              "_count": "desc"
            },
            "size": 5
          }
        }
      }
    }
  },
  "size": 0
}
```

测试结论
====

<table><thead><tr><th>聚合场景</th><th>ClickHouse(ms)</th><th>Elasticsearch(ms)</th><th>性能对比</th></tr></thead><tbody><tr><td>基于时间的多字段聚合</td><td>5506</td><td>15599</td><td>近 3 倍</td></tr><tr><td>多个字段按年进行计数（数据表）</td><td>381</td><td>6267</td><td>16 倍多</td></tr><tr><td>某字段出现次数 TOP 10（饼图）</td><td>4048</td><td>7317</td><td>近 2 倍</td></tr><tr><td>某字段按年进行计数（时间趋势图）</td><td>901</td><td>23257</td><td>25 倍多</td></tr><tr><td>聚合嵌套（非时间字段）</td><td>6937</td><td>15767</td><td>2 倍多</td></tr></tbody></table>

相同数据量下，ClickHouse 的聚合性能都要优于 Elasticsearch，且如果基于排序键进行聚合，性能更好，是 ES 的数倍。  
此外，ClickHouse 的 SummaryMergeTree、AggregatingMergeTree 表引擎支持后台自动聚合数据，所以在某些场景下其聚合分析性能会更优。