> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/44c88e85386e)

优点：  
1 . 一次返回所有数据。不需要多个接口。

2.  只返回想要的属性，不会获取多余的属性。

  
测试工具 GraphiQL 。 测试地址：[http://snowtooth.moonhighway.com](https://links.jianshu.com/go?to=http%3A%2F%2Fsnowtooth.moonhighway.com)

1. 最简单的查询。
----------

查询出所有电梯 (`lifts`) 的 name 和 staus 两个属性,  
allLifts ...... 等 被大括号括起来，叫做选择集（selection set)，选择集之间也可以相互嵌套。（这些可以查看测试地址的 docs）

```
query {
  allLifts {
    name
    status
  }
}


```

2. 多个查询，条件查询
------------

查询所有开启状态的电梯总数`liftCount`；以及所有的电梯和缆车 (`trails`)

```
query liftsAndTrails {
  liftCount (status: OPEN)
  allLifts {
    name
    status
  }
  allTrails {
    name
    difficulty
  }
}


```

查询结果大概如图所示。

```
{
  "data": {
    "liftCount": 10,
    "allLifts": [
      {
        "name": "Astra Express",
        "status": "HOLD"
      },
    // 省略  ......
    ],
    "allTrails": [
      {
        "name": "Blue Bird",
        "difficulty": "intermediate"
      },
    // 省略   ......
    ]
  }
}


```

3. 指定别名
-------

默认返回 json 数据的字段名和选择集中的一直。也可以指定别名。将返回数据中的`liftCount` 重命名为 `open`,`allLifts`重命名为 `chairLifts`。

```
query liftsAndTrails {
  open:liftCount (status: OPEN)
  chairLifts: allLifts {
    name
    status
  }
  allTrails {
    name
    difficulty
  }
}


```

4. 条件查询和排序
----------

筛选出 状态为 HOLD 的数据

```
query closedLifts {
  allLifts (status: HOLD){
    name
    status
  }
}


```

查询 名字为`jazz-cat` 的电梯的属性

```
query oneLift {
  Lift (id: "jazz-cat"){
    name
    status
    night
    elevationGain
  }
}


```

5. 边和连接
-------

相当于 sql 数据库查询时候关联表。trailAccess 是 Lift 的一个属性，但是他的类型是 Trail

```
query tailsAccessByJazzCat {
  Lift (id: "jazz-cat"){
    name
    status
    night
    elevationGain
    capacity
    trailAccess {
      name
      difficulty
    }
  }
}


```

6. 片段 fragment
--------------

白话就是抽取出公用的属性，方便统一修改。  
eg: 我在两个地方都需要使用 Trail 里面的 name 和 difficulty 属性。那么我们就把他写成一个片段。

```
// 这就是一个片段
fragment TrailInfo on Trail {
  name
  difficulty
}

query {
  Lift (id: "jazz-cat"){
    name
    status
    night
    elevationGain
    capacity
    trailAccess {
      ...TrailInfo
    }
  }
  Trail (id: "river-run"){
    ...TrailInfo
  }
}


```