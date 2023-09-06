> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg5MDk2MDI5OA==&mid=2247484977&idx=1&sn=7039327be0e3889f6222c85e2ac92033&chksm=cfd5e6f4f8a26fe20cb1a4f662f83d288ea08fd03802d298454af4a5df100b572b0b7c6ecf46&scene=21#wechat_redirect)

> 实现用户之间的关注和取消关注、查询是否关注、共同关注及关注后消息采用 feed 方式推送及滚动分页查看效果等相关功能。

**一、简介**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm46BqicD5yJxfUmfBTD3qibf1TgR01ulae6Ej4Mo0qicJMWI3QRSr9BRVFMibWlGvdh2dzMbycey7h3mAA/640?wx_fmt=png)

       实现用户之间的关注和取消关注、查询是否关注、共同关注及关注后消息采用 feed 方式推送及滚动分页查看效果等相关功能。利用 redis 里面的 Set 集合实现关注，取关，共同关注，消息推送等，结合 Java 代码实现具体的功能。

**二、实现关注和取关**  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm46BqicD5yJxfUmfBTD3qibf1TJfMpic2swG2HQicwldmhh8cZvveVfgycB1FqiasgiaPTuCYFdSGG8q1nLg/640?wx_fmt=png)

**2.1、关注和取消关注功能**

**业务流程图：**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45sEp7nvnJqG1icbEXkAVibZGmQBhIgXMVdv5AvmGpUTqvJGzk3nalvCVxmzO01TicA4l6CwL5rAna8g/640?wx_fmt=png)

**核心代码实现：**

```
    public Result follow(Long followUserId, Boolean isFollow,Long userId) {
        // 1.获取登录用户 暂时先缓存前端传用户ID
//        Long userId = UserHolder.getUser().getId();
        String key = "follows:" + userId;
        // 1.判断到底是关注还是取关
        if (isFollow) {
            // 2.关注，新增数据
            Follow follow = new Follow();
            follow.setUserId(userId);  //当前登录人员ID
            follow.setFollowUserId(followUserId);  //关注用户的id
            boolean isSuccess = save(follow);
            if (isSuccess) {
                // 把关注用户的id，放入redis的set集合 sadd userId followerUserId
                stringRedisTemplate.opsForSet().add(key, followUserId.toString());
            }
        } else {
            // 3.取关，删除 delete from tb_follow where user_id = ? and follow_user_id = ?
            boolean isSuccess = remove(new QueryWrapper<Follow>()
                    .eq("user_id", userId).eq("follow_user_id", followUserId));
            if (isSuccess) {
                // 把关注用户的id从Redis集合中移除
                stringRedisTemplate.opsForSet().remove(key, followUserId.toString());
            }
        }
        return Result.ok();
    }

```

**结果展示：**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45sEp7nvnJqG1icbEXkAVibZGnnynA5BFTXRNHOAQatyMJC5p03HusCeajHCtHcIic7zleR5OTowA3Ng/640?wx_fmt=png)

**2.2、共同关注功能**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm46BqicD5yJxfUmfBTD3qibf1TKdcqOFZt3CAsibFTjudDXK9cI6A1mg6dwLrAgicaIlYT5nzGgftfBBgA/640?wx_fmt=png)

**2.3、使用的 redis 命令如下：**  

```
SINTER key1 [key2]
返回给定所有集合的交集

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm46BqicD5yJxfUmfBTD3qibf1Tlyn5GEWerE9lxqebQyR0EX3YQBWI3ZluLperSoI8HXaN4I8yB8R16A/640?wx_fmt=png)

**2.4、代码实现**

```
    public Result followCommons(Long id,Long userId) {
        // 1.获取当前用户
//        Long userId = UserHolder.getUser().getId();
        String key = "follows:" + userId;
        // 2.求交集
        String key2 = "follows:" + id;
        Set<String> intersect = stringRedisTemplate.opsForSet().intersect(key, key2);
        if (intersect == null || intersect.isEmpty()) {
            // 无交集
            return Result.ok(Collections.emptyList());
        }
        // 3.解析id集合
        List<Long> ids = intersect.stream().map(Long::valueOf).collect(Collectors.toList());
        // 4.查询用户
        List<UserDTO> users = userService.listByIds(ids)
                .stream()
                .map(user -> BeanUtil.copyProperties(user, UserDTO.class))
                .collect(Collectors.toList());
        return Result.ok(users);
    }

```

****2.5、结果展示****

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45sEp7nvnJqG1icbEXkAVibZGEyE4oO0h7Tqgic3Lxj4wJaAtSY93VVribqmMAIC98wUpib1ef7UP22cew/640?wx_fmt=png)

**三、关注推送**  

**3.1、Feed 流模式**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm448dcIWY5bgJwnG5yfZdwHEmTic2nCZjhe5Kib55VomeIJh1HpjjMoFAaVaylFhYicXNeRbQMHv97b9w/640?wx_fmt=png)

**3.2、常见模式**

**Feed 流产品有两种常见模式：**

**●Timeline:** 不做内容筛选，简单的按照内容发布时间排序，常用于**好友或关注**。例如：朋友圈。

    > **优点**：信息全面，不会有缺失。并且实现也相对简单。

    > **缺点**：信息噪音较多，用户不一定感兴趣，内容获取效率低。

**●智能排序**：利用智能算法屏蔽掉违规的、用户不感兴趣的内容。推送用户感兴趣信息来吸引用户

    **> 优点**：投喂用户感兴趣信息，用户粘度很高，容易沉迷。

    **> 缺点**：如果算法不精准，可能起到反作用。

     本例中的个人页面，是基于关注的好友来做 Feed 流，因此采用 Timeline 的模式。该模式的实现方案有三种：

①拉模式   ②推模式    ③推拉结合

**3.3、三种实现方案**  

**3.3.1、拉模式**

**说明**：粉丝主动去拉取相关信息。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm448dcIWY5bgJwnG5yfZdwHEx1D3RHNYNmibA0ualTwpl6HGB6505eAaH5SRtm6cR1LuRJwGGwXnGXw/640?wx_fmt=png)

**3.3.2、推模式**  

**说明**：博主主动推送相关信息给粉丝。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm448dcIWY5bgJwnG5yfZdwHEkBgjEsTcxQr6A80zia6ImW4w7n5OlGOVb5wEPhJ5VDEpLIpciaibXSnkQ/640?wx_fmt=png)

**3.3.3、推拉结合模式**

****说明**：****活跃粉丝**：博主主动推送相关信息给粉丝。**普通粉丝**：粉丝主动去拉取相关信息。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm448dcIWY5bgJwnG5yfZdwHEViczj2766hAE7xOQ4V37FialI3zvkOfn2DSZtr8t8kb1rX9UibmbyO7Lw/640?wx_fmt=png)

**3.3.4、总结**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm448dcIWY5bgJwnG5yfZdwHEAiaM6CvSenF5AW66kN0LF5jcNsDH2CHFzN9EviaMBwcvuNl4pmke8MJw/640?wx_fmt=png)

       对于大部分中小型公司需求，采取推模式基本上满足需求，对于超千万的用户量的大型公司，需要采取推拉模式，但是实现上相比就复杂多了。

**四、案例**

**4.1、需求分析**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm448dcIWY5bgJwnG5yfZdwHEb9mXfkdiclwVxu7G3G8KAUgFfSmmo2QGQJeRgsBQ2t4FWxsTzLuT6YQ/640?wx_fmt=png)

**滚动分页：**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm448dcIWY5bgJwnG5yfZdwHEzacNKzK7sWxcFum7icPtysDdkhjGneeWDBvr6dWC4x51N1vl9nic7TSA/640?wx_fmt=png)

**4.2、使用到的 redis 命令**

**1、ZREVRANGEBYSCORE：**Redis 的一个 Sorted Set 命令，用于按照分数从高到低的顺序返回满足指定分数范围条件的元素。它的语法如下：

```
ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]

```

**参数解释如下：**

    key：排序集合的键名。

    max：分数范围的上限，可以使用 "+inf" 表示正无穷大。

    min：分数范围的下限，可以使用 "-inf" 表示负无穷大。

    WITHSCORES：可选参数，表示返回结果时同时返回元素与分数。

    LIMIT：可选参数，用于限制返回结果的偏移量和数量。

**示例用法：**

```
ZREVRANGEBYSCORE myset 100 0
ZREVRANGEBYSCORE myset (100 0 WITHSCORES
ZREVRANGEBYSCORE myset 100 0 LIMIT 0 10

```

第一个示例命令将返回分数从最高到最低的所有元素，分数范围为 (100，0]，不包括 100。

第二个示例命令将返回分数从最高到最低的所有元素及其对应的分数，分数范围为 (100，0]，不包括 100。

第三个示例命令将返回分数从最高到最低的前 10 个元素，分数范围为 (100，0]，不包括 100。

**注意**：ZREVRANGEBYSCORE 是按照分数从高到低的顺序返回结果的，如果需要按照分数从低到高的顺序返回结果，可以使用 ZRANGEBYSCORE 命令。

**2、ZADD：**在 Redis 中，ZADD 命令用于向有序集合 (Sorted Set) 中添加一个或多个成员。有序集合是一种将成员与分数 (score) 关联的数据结构，通过分数可以对成员进行排序。

**ZADD 命令的语法如下：**

```
ZADD key [NX|XX] [CH] [INCR] score member [score member ...]

```

**参数说明：**

    key：有序集合的键名。

    NX：仅在键不存在时添加成员。

    XX：仅在键已经存在时添加成员。

    CH：返回修改的成员数量，包括新增和更新的成员。

    INCR：对已经存在的成员的分数进行自增操作。

    score：成员的分数，用于排序。

    member：要添加的成员。

**示例：**

```
ZADD myset 1 "member1"
ZADD myset 2 "member2" 3 "member3"

```

**注意：ZADD 命令在 Redis 版本 2.4 以及以上版本可用。**

**reids 中实现滚动分页功能：**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm44XhLJqQFn2Cibiaso3rCjmz2DibgegQNqRnCAEl2rC1bt0IRZZQfbxfhyMM5FPyM2OkxU6opUt3MiaLA/640?wx_fmt=png)

**4.3、业务流程图**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm44XhLJqQFn2Cibiaso3rCjmz2XxuQaH6GeaVk2oVEcRhD3Np7OwDsxY8NiaZfdTwaYQxynZIibo6cWiaEA/640?wx_fmt=png)

**4.4、核心代码展示**

**4.4.1、保存探店笔记并发送收件箱**

```
    public Result saveBlog(Blog blog) {
        Long userId = 999L;  //暂时写死
        // 1.获取登录用户
//        UserDTO user = UserHolder.getUser();
//        blog.setUserId(user.getId());
        blog.setUserId(userId);
        // 2.保存探店笔记
        boolean isSuccess = save(blog);
        if(!isSuccess){
            return Result.fail("新增笔记失败!");
        }
        // 3.查询笔记作者的所有粉丝 select * from tb_follow where follow_user_id = ?
        List<Follow> follows = followService.query().eq("follow_user_id", userId).list();
        //非空判断
        if(CollUtil.isEmpty(follows)){
            return Result.ok();
        }
        // 4.推送笔记id给所有粉丝
        for (Follow follow : follows) {
            // 4.1.获取粉丝id
            Long uId = follow.getUserId();
            // 4.2.feed推送消息
            String key = FEED_KEY + uId;
            stringRedisTemplate.opsForZSet().add(key, blog.getId().toString(), System.currentTimeMillis());
        }
        // 5.返回id
        return Result.ok(blog.getId());
    }

```

**4.4.2、查询关注的所有探店笔记 (并含滚动分页)**

```
public Result queryBlogOfFollow(Long max, Integer offset) {
        // 1.获取当前用户
//        Long userId = UserHolder.getUser().getId();
        Long userId = 111L;
        // 2.查询收件箱 ZREVRANGEBYSCORE key Max Min LIMIT offset count
        String key = FEED_KEY + userId;
        Set<ZSetOperations.TypedTuple<String>> typedTuples = stringRedisTemplate.opsForZSet()
                .reverseRangeByScoreWithScores(key, 0, max, offset, 2);
        // 3.非空判断
        if (typedTuples == null || typedTuples.isEmpty()) {
            return Result.ok();
        }
        // 4.解析数据：blogId、minTime（时间戳）、offset
        List<Long> ids = new ArrayList<>(typedTuples.size());
        long minTime = 0; // 2
        int os = 1; // 2
        for (ZSetOperations.TypedTuple<String> tuple : typedTuples) { // 5 4 4 2 2
            // 4.1.获取id
            ids.add(Long.valueOf(tuple.getValue()));
            // 4.2.获取分数(时间戳）
            long time = tuple.getScore().longValue();
            if(time == minTime){
                os++;
            }else{
                minTime = time;
                os = 1;
            }
        }
        // 5.根据id查询blog
        String idStr = StrUtil.join(",", ids);
        List<Blog> blogs = query().in("id", ids).last("ORDER BY FIELD(id," + idStr + ")").list();
        for (Blog blog : blogs) {
            // 5.1.查询blog有关的用户
            queryBlogUser(blog);
            // 5.2.查询blog是否被点赞
            isBlogLiked(blog);
        }
        // 6.封装并返回
        ScrollResult r = new ScrollResult();
        r.setList(blogs);
        r.setOffset(os);
        r.setMinTime(minTime);
        return Result.ok(r);
    }

```

**备注：**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45sEp7nvnJqG1icbEXkAVibZGEObXnZX9CiaELZmT6KZ5ZQUX7SfT3H1I1kzrPNMoKicGJbSSynEDRQiaw/640?wx_fmt=png)**4.4.3、请求参数**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm44XhLJqQFn2Cibiaso3rCjmz2lEXHnKPZC1R8sRbDBHYpZDJrlH400Sia7cfUbIpfVay2b0nQvSmYIIw/640?wx_fmt=png)

**参考案例：**

```
{
  "comments": 0,
  "content": "生活就是一半烟火·一半诗意",
  "createTime": "2023-08-17T05:17:34.287Z",
  "id": 90,
  "images": "111111",
  "liked": 0,
  "name": "11111",
  "shopId": 10,
  "title": "生活就是一半烟火",
  "updateTime": "2023-08-17T05:17:34.287Z",
  "userId": 999
}

```