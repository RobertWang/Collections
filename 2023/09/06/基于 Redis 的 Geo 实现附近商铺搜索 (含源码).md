> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg5MDk2MDI5OA==&mid=2247484992&idx=1&sn=6970887f799637fc06af4ff2a0d36214&chksm=cfd5e685f8a26f9330e49a6e585f95b22b325c042da6c05a2d184f51272acf1920059d8a2b7b&scene=21#wechat_redirect)

> 基于 Redis 的 Geo 实现附近商铺搜索及滚动分页

**一、GEO 常用命令及使用示范**

**![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm44XhLJqQFn2Cibiaso3rCjmz2zfJvWWjDic9dBjhZmUPYuibZ6iafVKaFooibltcexbibia9LDQJ8Ew6S61cg/640?wx_fmt=png)**

**1.1、GEO 的数据结构  
**

GEO 就是 Geolocation 的简写形式，代表地理坐标。Redis 在 3.2 版本中加入了对 GEO 的支持，允许存储地理坐标信息，帮助我们根据经纬度来检索数据。

**常见的命令有：**

**1、GEOADD**：添加一个地理空间信息，包含：经度（longitude）、纬度（latitude）、值（member）。

**案例**：添加下面几条数据：

    北京南站（116.378248    39.865275)

    北京站 (116.42803     39.903738)

    北京西站（116.322287     39.893729)

```
0>geoadd  g1 116.378248  39.865275 bjnz 116.42803 39.903738 bjz 116.322287 39.893729 bjxz
"3"

```

**2、GEODIST**：计算指定的两个点之间的距离并返回。

**案例：**计算北京西站到北京站的距离

```
redis6.3:0>geodist g1 bjxz bjz km 
"9.0916"
redis6.3:0>

```

**3、GEOHASH**：将指定 member 的坐标转为 hash 字符串形式并返回.

```
redis6.3:0>geohash g1 bjz
1) "wx4g12k21s0"

```

**4、GEOPOS：**返回指定 member 的坐标  

```
redis6.3:0>GEOPOS g1 bjz
1) 1) "116.42802804708480835"
   2) "39.90373880538094653"

```

**5、GEORADIUS**：指定圆心、半径，找到该圆内包含的所有 member，并按照与圆心之间的距离排序后返回。(**6.2 以后已废弃)**  

**6、GEOSEARCH**：在指定范围内搜索 member，并按照与指定点之间的距离排序后返回。范围可以是圆形或矩形。(**6.2 新功能)。在 Redis 中，GEOSEARCH 命令用于在地理位置的有序集合中，根据给定的位置信息搜索符合条件的成员。**  

**GEOSEARCH 命令的语法如下：**

```
GEOSEARCH key
    [FROMMEMBER member]
    [FROMLONLAT longitude latitude]
    [BYRADIUS radius unit]
    [WITHCOORDINATES]
    [WITHDISTANCES]
    [WITHHASH]
    [COUNT count]
    [ASC|DESC]
    [STORE key]
    [STOREDIST key]

```

**参数说明：**

   **key**：地理位置的有序集合的 key 名称。

    **FROMMEMBER member**：从**指定成员**开始搜索，返回符合条件的成员。

    **FROMLONLAT longitude latitude**：从给定的**经度和纬度**开始搜索，返回符合条件的成员。

    **BYRADIUS radius unit**：根据指定的半径和单位搜索符合条件的成员。

    **WITHCOORD**：返回成员的**经纬度坐标**。

    **WITHDIST**：返回成员与**中心位置的距离**。

    **WITHHASH**：返回成员的**地理哈希值**。

   **COUNT count**：限制返回的成员数量。

 **ASC|DESC**：指定返回结果的排序方式，默认为**升序**。

    **STORE key**：将搜索结果存储到指定的键名中。

    **STOREDIST key**：将搜索结果存储到指定的键名中，并返回成员与中心位置的距离。

**案例：**搜索天安门 (116.397904 39.909005) 附近 10km 内的所有火车站，并按照距离升序排序。

```
redis6.3:0>geosearch g1 fromlonlat 116.397904 39.909005 byradius 10 km withdist
1) 1) "bjz"
   2) "2.6361"
2) 1) "bjnz"
   2) "5.1452"
3) 1) "bjxz"
   2) "6.6723"

```

**7、GEOSEARCHSTORE**：与 GEOSEARCH 功能一致，不过可以把结果存储到一个指定的 key。(**6.2. 新功能)**

**二、应用案例**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm44XhLJqQFn2Cibiaso3rCjmz2fUU5V15Fic37MZBSBNEicXvOQBhI4urticnMibxnfawlEaCF3X6diaZNI0g/640?wx_fmt=png)

**1、将店铺信息添加到缓存中**

**![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm44XhLJqQFn2Cibiaso3rCjmz2oyo1CkJqdFHSYn3DhXNMMINH0A5CeshQscpXrGrib8yicRyGe8GbH1Lg/640?wx_fmt=png)**

**核心代码：  
**

```
   public void shopDataToRedis() {
        // 1.查询店铺信息
        List<Shop> list = query().list();
        // 2.把店铺分组，按照typeId分组，typeId一致的放到一个集合
        Map<Long, List<Shop>> map = list.stream().collect(Collectors.groupingBy(Shop::getTypeId));
        // 3.分批完成写入Redis
        for (Map.Entry<Long, List<Shop>> entry : map.entrySet()) {
            // 3.1.获取类型id
            Long typeId = entry.getKey();
            String key = SHOP_GEO_KEY + typeId;
            // 3.2.获取同类型的店铺的集合
            List<Shop> value = entry.getValue();
            List<RedisGeoCommands.GeoLocation<String>> locations = new ArrayList<>(value.size());
            // 3.3.写入redis GEOADD key 经度 纬度 member
            for (Shop shop : value) {
                // stringRedisTemplate.opsForGeo().add(key, new Point(shop.getX(), shop.getY()), shop.getId().toString());
                locations.add(new RedisGeoCommands.GeoLocation<>(
                        shop.getId().toString(),
                        new Point(shop.getX(), shop.getY())
                ));
            }
            stringRedisTemplate.opsForGeo().add(key, locations);
        }
    }

```

**接口调用及结果：**  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm44XhLJqQFn2Cibiaso3rCjmz2AMiaUXLwxTxAbWicVjXDia59X0CkI2RKgsuPDhARUKDm1KadLkfsFKCBQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm44XhLJqQFn2Cibiaso3rCjmz2vfZ71UfyTwNzDBkw8poaicE3gMibaXFjlylrnPCkXibUT9lNibqvjNtXPA/640?wx_fmt=png)

**2、根据商铺类型基于 Geo 实现附近商铺分页滚动查询**

**业务流程：**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm44XhLJqQFn2Cibiaso3rCjmz21dlgCmjrDh1wX86yqIGssxt7OIpxyIqTRYRhebEz8guNibLWzvBrspg/640?wx_fmt=png)

**核心代码：**  

```
 public Result queryShopByType(Integer typeId, Integer current, Double x, Double y) {
        // 1.判断是否需要根据坐标查询
        if (x == null || y == null) {
            // 不需要坐标查询，按数据库查询
            Page<Shop> page = query()
                    .eq("type_id", typeId)
                    .page(new Page<>(current, SystemConstants.DEFAULT_PAGE_SIZE));
            // 返回数据
            return Result.ok(page.getRecords());
        }
        // 2.计算分页参数
        int from = (current - 1) * SystemConstants.DEFAULT_PAGE_SIZE;
        int end = current * SystemConstants.DEFAULT_PAGE_SIZE;
        // 3.查询redis、按照距离排序、分页。结果：shopId、distance
        String key = SHOP_GEO_KEY + typeId;
        GeoResults<RedisGeoCommands.GeoLocation<String>> results = stringRedisTemplate.opsForGeo()      // GEOSEARCH key BYLONLAT x y BYRADIUS 10 WITHDISTANCE
                .search(
                        key,
                        GeoReference.fromCoordinate(x, y),
                        new Distance(5000),
                        RedisGeoCommands.GeoSearchCommandArgs.newGeoSearchArgs().includeDistance().limit(end)
                );
        // 4.解析出id
        if (results == null) {
            return Result.ok(Collections.emptyList());
        }
        List<GeoResult<RedisGeoCommands.GeoLocation<String>>> list = results.getContent();
        if (list.size() <= from) {
            // 没有下一页了，结束
            return Result.ok(Collections.emptyList());
        }
        // 4.1.截取 from ~ end的部分
        List<Long> ids = new ArrayList<>(list.size());
        Map<String, Distance> distanceMap = new HashMap<>(list.size());
        list.stream().skip(from).forEach(result -> {
            // 4.2.获取店铺id
            String shopIdStr = result.getContent().getName();
            ids.add(Long.valueOf(shopIdStr));
            // 4.3.获取距离
            Distance distance = result.getDistance();
            distanceMap.put(shopIdStr, distance);
        });
        // 5.根据id查询Shop
        String idStr = StrUtil.join(",", ids);
        List<Shop> shops = query().in("id", ids).last("ORDER BY FIELD(id," + idStr + ")").list();
        for (Shop shop : shops) {
            shop.setDistance(distanceMap.get(shop.getId().toString()).getValue());
        }
        // 6.返回
        return Result.ok(shops);
    }

```

**接口调用及展示：**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm44XhLJqQFn2Cibiaso3rCjmz24YahzwaZW3q2SDHkQE7vWRuibFt2X1SFJaqEOPhic0PHK1CWqicj6f97g/640?wx_fmt=png)

**注意事项：**

       SpringDataRedis 的 2.3.9 版本并不支持 Redis 6.2 提供的 GEOSEARCH 命令因此我们需要提示其版本，修改自己的 pom.xml，内容如下：

**![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm44XhLJqQFn2Cibiaso3rCjmz2mI19QXXcynBOuD4p1KByXKZ6EVMjM2xjeeibMQsm1Q1jOfCNmxUa9dg/640?wx_fmt=png)**

**最终结果展示：**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm44XhLJqQFn2Cibiaso3rCjmz2kd6bpj8ZPkTKVVdQkXET2TQSWuWDUhIdYC0e8rIia045ZBU1O9uYKEQ/640?wx_fmt=png)

**三、源码获取方式**