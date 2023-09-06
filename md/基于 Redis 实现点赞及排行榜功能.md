> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/2_wgTf_ikDqnZrACj_nD8w)

> 基于 Redis 实现点赞及排行榜功能

**一、简介**

**1.1、点赞相关需求：**

      1、同一个用户只能点赞一次，再次点击则取消点赞 (点赞 / 取消点赞);

      2、如果当前用户已经点赞，则点赞按钮高亮显示（前端实现，判断字段 Blog 类的 isLik 属性）。

**1.2、实现步骤：**

      1、给 Blog 类中添加一个 isLike 字段，标识是否被当前用户点赞；

      2、修改点赞功能，利用 Redis 的 set 集合判断是否点赞过，未点赞过则点赞数 + 1，已点赞过则点赞数 - 1;

      3、修改根据 id 查询 Blog 的业务，判断当前登录用户是否点赞过，赋值给 isLike 字;

      4、修改分页查询 Blog 业务，判断当前登录用户是否点赞过，赋值给 isLike 字段。

**1.3、点赞实现思路：**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm46BqicD5yJxfUmfBTD3qibf1TwwXZibH4q8e85ClODaWj7MicPpuwunnibLymM9rFtngFgyuoDCZGJUUvw/640?wx_fmt=png)

**1.4、排行榜实现：**  

**![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm46BqicD5yJxfUmfBTD3qibf1TVvsFyVTed8gicG7rcyA8tIqKAXuzkvhKF5UERkQDkXOokUibls7UlMuw/640?wx_fmt=png)**

  **分析**：由于需要对点赞功能进行排行榜分析，按照点赞时间先后排序，返回最早点赞的 Top5 的用户，所以最好是选择 SortedSet 实现功能。(**特别说明：****代码获取方式在文章结尾**)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm46BqicD5yJxfUmfBTD3qibf1TwKQXoro9Cf8ECicmjFxpmSruSHldL3bicfG6ZUCIyK2KDk0Z9lhy1WRw/640?wx_fmt=png)

**涉及到表信息：**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45sEp7nvnJqG1icbEXkAVibZGibiaiah6Hx8E0iaRagOxMkn0zTaAwrSbOhOh7kBqGEjTaAN3aZ62OP7yQw/640?wx_fmt=png)

**二、常用的命令**

**2.1、zadd**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm46BqicD5yJxfUmfBTD3qibf1TVcibxZ9xesMuiaxO2FzoPIQAdibDWfGnHJlmaogNfeZvhicHjMia5icWa12Q/640?wx_fmt=png)

**案例：**

```
redis:0>zadd myzset 1 a1
"1"
redis:0>zadd myzset 2 a2
"1"

```

**2.2、zscore**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm46BqicD5yJxfUmfBTD3qibf1TFgyP2PDZpO07n0PCJHShhnr8k1EdnysItH7r8Totc7wIoJxY1Zng3Q/640?wx_fmt=png)

**案例：**  

```
redis:0>zscore myzset a2
"2"
redis:0>zscore myzset a3
"5"

```

**2.3、zrem**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm46BqicD5yJxfUmfBTD3qibf1TjC18IuuZ7GlNuUDH5RHVvUXP0PXnZewN4OldwfqHTibhrJ7SausU92g/640?wx_fmt=png)

**案例：**

```
redis:0>zrem myzset a2
"1"
redis:0>zscore myzset a2
null
redis:0>

```

**2.4、zrange**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm46BqicD5yJxfUmfBTD3qibf1TQ8iaSO3DcaxawRJprmGkgx2MRPiaNc99xNCoebne4ibHhW4hNHSuq8WVw/640?wx_fmt=png)

****案例：获取前四个元素****

```
redis:0>zrange blog:liked:6 0 4
1) "999"
2) "111"
3) "222"

```

**三、代码实现**  

**3.1、点赞 / 取消点赞功能**

**核心代码实现：**  

```
    public Result likeBlog(Long id, Long userId) {
        // 1.获取登录用户 展示用接口传值
//        Long userId = UserHolder.getUser().getId();
        // 2.判断当前登录用户是否已经点赞
        String key = BLOG_LIKED_KEY + id;
        //redis:0>zscore myzset a3  -> "5"
        Double score = stringRedisTemplate.opsForZSet().score(key, userId.toString());
        if (score == null) {
            // 3.如果未点赞，可以点赞
            // 3.1.数据库点赞数 + 1
            boolean isSuccess = update().setSql("liked = liked + 1").eq("id", id).update();
            // 3.2.保存用户到Redis的set集合  zadd key value score
            if (isSuccess) {
                //ZADD key score1 member1 [score2 member2]
                stringRedisTemplate.opsForZSet().add(key, userId.toString(), System.currentTimeMillis());
            }
        } else {
            // 4.如果已点赞，取消点赞
            // 4.1.数据库点赞数 -1
            boolean isSuccess = update().setSql("liked = liked - 1").eq("id", id).update();
            // 4.2.把用户从Redis的set集合移除
            if (isSuccess) {
                stringRedisTemplate.opsForZSet().remove(key, userId.toString());
            }
        }
        return Result.ok();
    }

```

**控制层：**  

```
   @ApiOperation(value="点赞功能", notes="likeBlog")
    @PostMapping("/liked")
    public Result likeBlog(@RequestParam("id") Long id,@RequestParam("userId") Long userId) {
        return blogService.likeBlog(id,userId);
    }

```

**接口调用：**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm46BqicD5yJxfUmfBTD3qibf1TRYUTAp7rlLczR7bR7lJFJYrF6UMdITdVbPkr6Gah8xCVKibrDxPckPg/640?wx_fmt=png)

**结果展示：**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm46BqicD5yJxfUmfBTD3qibf1TR0y6R7oW7ibUDuTBHEMvqdNhmQq2ya74TSs9D5icvHKyotkAtXwKZLVw/640?wx_fmt=png)

**3.2、排行榜功能**

**核心代码实现：**  

```
   public Result queryBlogLikes(Long id) {
        String key = BLOG_LIKED_KEY + id;
        // 1.查询top5的点赞用户 zrange key 0 4
        Set<String> top5 = stringRedisTemplate.opsForZSet().range(key, 0, 4);
        //判断集合是否为空（包括null和没有元素的集合）
        //参考hutool工具：https://www.hutool.cn/docs/#/core
        if(CollUtil.isEmpty(top5)){
            return Result.ok(Collections.emptyList());
        }
        // 2.解析出其中的用户id
        List<Long> ids = top5.stream().map(Long::valueOf).collect(Collectors.toList());
        String idStr = StrUtil.join(",", ids);
        // 3.根据用户id查询用户 WHERE id IN ( 5 , 1 ) ORDER BY FIELD(id, 5, 1)
        //数据量少基本不影响性能，若获取top数量比较大的时候，就需要优化性能
        List<UserDTO> userDTOS = userService.query()
                .in("id", ids).last("ORDER BY FIELD(id," + idStr + ")").list()
                .stream()
                .map(user -> BeanUtil.copyProperties(user, UserDTO.class))
                .collect(Collectors.toList());
        // 4.返回
        return Result.ok(userDTOS);
    }

```

****控制层：****

```
@ApiOperation(value="根据ID查询点赞的所有用户", notes="queryBlogLikes")
    @GetMapping("/likes/{id}")
    public Result queryBlogLikes(@PathVariable("id") Long id) {
        return blogService.queryBlogLikes(id);
    }

```

**接口调用及展示：**  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm46BqicD5yJxfUmfBTD3qibf1TRiaokqg7KQJRIzZgZemPY3DO94soQibvUETnB8TXm75nQwGMLbwmEhUQ/640?wx_fmt=png)