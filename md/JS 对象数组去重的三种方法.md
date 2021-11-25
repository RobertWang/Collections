> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6984625612937773070) // 原数据是这样的 // 去重后数据是这样的 [{ [{ "goodsId": "1", "goodsId": "1", "quota": 12, "quota": 12, "skuId": "1" "skuId": "1" }, }, { { "goodsId": "2", "goodsId": "2", "quota": 12, "quota": 12, "skuId": "2" "skuId": "2" }, }] { "goodsId": "1", "quota": 12, "skuId": "1" }] 复制代码

### 二. 使用方法

1.  使用 filter 和 Map 🌟🌟🌟🌟🌟
2.  使用 reduce 🌟🌟🌟🌟
3.  for 循环 🌟🌟🌟

结论：filter 和 Reduce 时间上差别不是太大，filter 稍微更快一些，但是 filter 语法更简洁

##### 1. 使用 filter 和 Map

代码简洁，好用，4 行代码搞定，平均耗费时间最短，五星推荐

```
function uniqueFunc(arr, uniId){
  const res = new Map();
  return arr.filter((item) => !res.has(item[uniId]) && res.set(item[uniId], 1));
}
复制代码
```

##### 2. 使用 reduce

代码稍多，平均耗费时间和第一不分伯仲，四星推荐

```
function uniqueFunc2(arr, uniId){
  let hash = {}
  return arr.reduce((accum,item) => {
    hash[item[uniId]] ? '' : hash[item[uniId]] = true && accum.push(item)
    return accum
  },[])
}
复制代码
```

##### 3. 使用 for 循环

耗费时间较一二稍多，但是耗费时间平均，三星推荐

```
function uniqueFunc3(arr, uniId){
  let obj = {}
  let tempArr = []
  for(var i = 0; i<arr.length; i++){
    if(!obj[arr[i][uniId]]){
      tempArr.push(arr[i])
      obj[arr[i][uniId]] = true
    }
  }
  return tempArr
}
复制代码
```

### 三. 2400 条数据，三种方法处理时间对比

<table><thead><tr><th>测试次数</th><th>filter Map</th><th>reduce</th><th>for 循环</th></tr></thead><tbody><tr><td>1</td><td>0.139892578125 ms</td><td>0.19189453125 ms</td><td>0.2060546875 ms</td></tr><tr><td>2</td><td>0.12109375 ms</td><td>0.1279296875 ms</td><td>0.195068359375 ms</td></tr><tr><td>3</td><td>0.112060546875ms</td><td>0.11767578125 ms</td><td>0.174072265625 ms</td></tr><tr><td>4</td><td>0.10400390625 ms</td><td>0.1728515625 ms</td><td>0.18701171875 ms</td></tr><tr><td>5</td><td>0.10986328125 ms</td><td>0.12890625 ms</td><td>0.175048828125 ms</td></tr><tr><td>6</td><td>0.113037109375 ms</td><td>0.10791015625 ms</td><td>0.172119140625 ms</td></tr><tr><td>7</td><td>0.134033203125 ms</td><td>0.129150390625 ms</td><td>0.172119140625 ms</td></tr></tbody></table>

##### 测试时间截图展示

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d93acff6edaf4700bfea74d8acd73557~tplv-k3u1fbpfcp-watermark.awebp)

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7ecde5c0453b4b09962b2ada848e3f1a~tplv-k3u1fbpfcp-watermark.awebp)

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0422db0ebe7d43d9b69c785e373066d9~tplv-k3u1fbpfcp-watermark.awebp)

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f49931b82544f94bd207e9906cc3bf6~tplv-k3u1fbpfcp-watermark.awebp)

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3f122dc5c18f486ebd52f4a32c7ad6a5~tplv-k3u1fbpfcp-watermark.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ced7f34c29fe42d1b47c11404bde489a~tplv-k3u1fbpfcp-watermark.awebp)