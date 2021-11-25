
> Elasticsearch 线上实战 Mapping 相关问题解读

1、实战线上问题
--------

最近几个线上问题，都和 Mapping 字段更新有关系，问题列表如下：

**问题 1**：

Mapping 新创建后，还可以更新吗?

**问题 2**:

*   群友 A：有人知道怎么在 kibana 里面给索引新增，删除字段吗？
*   群友 B: 不就是改 mapping 吗
*   群友 A：怎么改？
*   群友 B：写 dsl 啊…
*   群友 A：只能加不能删吧？
    

**问题 3：**

各位同学们 现在有个业务需求帮忙看一下？

需求: 将 A 索引中一个为 String 的字段修改为 boolean。
例: sdry:"1" -> sdry:true。

**问题 4：**

join 类型怎么修改 join，append 一个新的 child？
业务需要 append join children，官方也说可以 append，但是又没给方案，我尝试都失败了。

**四个问题**都可以归结为 Mapping 更新问题，我们一起梳理实践一把。

2、问题拆解解读
--------

### 问题 1：Mapping 新创建后，还可以更新吗？

官方文档有强调：

> In general, the mapping for existing fields cannot be updated. There are some exceptions to this rule.

也就是说，已经定义的字段大多数情况不能被更新，除非 reindex 更新 mapping。

但，以下三种情况例外。

*   第一：new properties can be added to Object fields.
    

Object 对象可以添加新的属性。

*   第二: new multi-fields can be added to existing fields.
    

已经存在的 fields 里面可以添加 fields，以构成一个字段多种类型。

*   第三：the ignore_above parameter can be updated.
    

ignore_above 是可以更新的。

问题 1 特例情况实战一把。

```
DELETE my_index
PUT my_index 
{
  "mappings": {
    "properties": {
      "name": {
        "properties": {
          "first": {
            "type": "text"
          }
        }
      },
      "user_id": {
        "type": "keyword"
      }
    }
  }
}
```

更新 Mapping 操作如下示例：

```
PUT my_index/_mapping
{
  "properties": {
    "name": {
      "properties": {
        "first":{
          "type":"text",
          "fields":{
            "field":{
              "type":"keyword"
            }
          }
        },
        "last": { 
          "type": "text"
        }
      }
    },
    "user_id": {
      "type": "keyword",
      "ignore_above": 100
    }
  }
}
```

以上：

对应第一种情况，Object 对象可以添加新的属性。我们添加了 last 字段。

对应第二种情况，first 添加了 keyword 类型，以组合构造 fields。

对应第三种情况，user_id 添加了 ignore_above。

这三种 Mapping 更新特列情况，大家需要掌握。实战环节不需要 reindex 就可以更新 Mapping，还是非常便捷的。

### 问题 2：如何给索引新增、删除字段？

> 有人知道怎么在 kibana 里面给索引新增，删除字段吗？

强调一下：

*   Mapping 中已有的字段是不可以删除的，除非 reindex。
    
*   Mapping 字段设置默认是 "dynamic:true"，表明支持动态添加字段。
    

更新 Mapping 添加字段举例如下：

```
DELETE  my-index-003
#创建索引同时指定 Mapping
PUT my-index-003
{
  "mappings": {
    "properties": {
      "message": {
        "type": "keyword",
        "ignore_above": 20
      }
    }
  }
}
#更新 Mapping
POST my-index-003/_mapping
{
  "properties": {
    "title": {
      "type": "text",
      "analyzer": "ik_max_word"
    }
  }
}
```

dynamic 设置值及含义如下表所示：

<table data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)"><thead data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)"><tr data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); text-align: center; min-width: 85px;"><span data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(255,255,255)|rgb(240, 240, 240)">属性值</span></th><th data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); text-align: center; min-width: 85px;"><span data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(255,255,255)|rgb(240, 240, 240)">含义</span></th></tr></thead><tbody data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)"><tr data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;"><span data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(255,255,255)">true</span></td><td data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;"><span data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(255,255,255)">默认，支持动态更新</span></td></tr><tr data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;"><span data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(248, 248, 248)">false</span></td><td data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;"><span data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(248, 248, 248)">忽略新增字段</span></td></tr><tr data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;"><span data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(255,255,255)">strict</span></td><td data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px; word-break: break-all;"><span data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(255,255,255)">严格定义字段，类似写死固定字段，再新增未设定字段会报错</span></td></tr><tr data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;"><span data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(248, 248, 248)">runtime</span></td><td data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;"><span data-darkmode-color-16378555711063="rgb(163, 163, 163)" data-darkmode-original-color-16378555711063="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16378555711063="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16378555711063="#fff|rgb(248, 248, 248)">和默认 true 有细微差别，参见官方文档</span></td></tr></tbody></table>

### 问题 3：string 类型改成 boolean 类型，如何实现？

> 各位同学们 现在有个业务需求帮忙看一下。

> 需求: 将 A 索引中一个为 String 的字段修改为 boolean。

> 例: sdry:"1" -> sdry:true

可以将问题进一步提炼转换为：修改 Mapping 字段类型。

Mapping 字段是不可以直接更新的，但我们可以 “曲线救国”。

```
# 创建索引
PUT test-002
{
 "mappings": {
  "properties": {
   "sflag":{
    "type":"keyword"
   }
  }
 }
}

# 模拟写入数据
PUT test-002/_bulk
{"index":{"_id":1}}
{"sflag":"1"}
{"index":{"_id":2}}
{"sflag":"0"}

# 更新Mapping
POST test-002/_mapping
{
 "properties":{
  "bflag":{
   "type":"boolean"
  }
 }
}

# 对新增字段做数据处理
PUT _ingest/pipeline/mychangepipeline
{
 "processors":[
  {
    "script": {
     "description": "Extract 'tags' from 'env' field",
     "lang": "painless",
     "source": """
     if(ctx['sflag'] == "1")
     {
      ctx['bflag']=true;
     }else if(ctx['sflag']=="0")
     {
      ctx['bflag']=false;
     }
     """
    }
   }
  ]
}

# 全量更新操作
POST test-002/_update_by_query?pipeline=mychangepipeline
{
 "query": {
  "match_all": {}
 }
}

# 检索结果
POST test-002/_search
```

解读一下：

第一步：新增了字段 bflag，且设置为 boolean 类型。

第二步：自建 ingest 预处理管道，结合原有 sflag 字段更新新增的 bflag 字段。

第三步：全量批量更新已有索引，实现字段的更新。

自此，“曲线救国” 达到目的，如下图所示，bflag 设置成了 boolean 值。

![](https://mmbiz.qpic.cn/mmbiz_png/mjl8GCpsL9YY4XiaW4Th6FGjoZSAriapdgxYupjWbAbVyplxE5x4NWdsc1fCwO6L6bdoFibJXmUZs1qUXMmPp09XA/640?wx_fmt=png)

### 问题 4：join 类型添加新 child 如何实现？

> join 类型怎么修改 join，append 一个新的 child？

> 业务需要 append join children，官方也说可以 append，但是又没给方案，我尝试都失败了。

实践一把，给出答案。

```
DELETE test-join-index
# 创建父子文档关联索引
PUT test-join-index
{
  "mappings": {
    "properties": {
      "my_id": {
        "type": "keyword"
      },
      "my_join_field": {
        "type": "join",
        "relations": {
          "question": "answer_a"
        }
      }
    }
  }
}

# 更新 Mapping
POST test-join-index/_mapping
{
  "properties": {
    "my_join_field": {
      "type": "join",
      "relations": {
        "question": [
          "answer_a",
          "answer_b",
          "answer_c",
          "answer_d"
        ]
      }
    }
  }
}
```

上面的更新 Mapping 部分，由 1 对 1 的父子关联关系，转化为：1 对 4 的父子关联关系，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/mjl8GCpsL9YY4XiaW4Th6FGjoZSAriapdgnXYBay5CmM1Tuco0Hng2pZKFxYww8zMTq0FKQafb5RQYu5oQV02LGw/640?wx_fmt=png)

3、小结
----

Mapping 字段的 dynamic 特性有利有弊，要结合业务场景选型，对不希望动态扩展字段以至字段 “膨胀” 的场景下，建议设置为 strict。

Mapping 创建后，已有字段不可以修改，但可以 “曲线救国” 实现字段更新，间接实现字段的“修改”。

Mapping 中已有字段更新的三个特列要掌握。

Runtime field 运行时类型也能很好的解决本文提出的动态扩展字段的问题，鉴于篇幅原因，本文没有展开。更多 runtime field 实战解读，推荐阅读：  

[Elasticsearch 运行时类型 Runtime fields 深入详解](http://mp.weixin.qq.com/s?__biz=MzI2NDY1MTA3OQ==&mid=2247487524&idx=1&sn=af826b6b6c6bf4458b6f8ede1927e32d&chksm=eaa8380cdddfb11a06740b4ee604fcf942990e388e54570038f3185b8af66766bd6f7b320099&scene=21#wechat_redirect)  

---

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/t0ce5WESP4twECP_LPVI9Q)
