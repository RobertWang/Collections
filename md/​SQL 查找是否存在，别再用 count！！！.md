> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/hI_44JwYg21ue9bWtuI0NA)

问题背景
----

在业务开发中，经常遇到如下需求：从数据库中查找符合一组条件的数据是否存在。结果也无非两种状态：“有” 或者 “没有”。

模拟数据表
-----

```
CREATE TABLE `user`  (
  `id` bigint NOT NULL COMMENT 'id',
  `userName` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL COMMENT '用户昵称',
  `userAccount` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '账号',
  `userAvatar` varchar(1024) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL COMMENT '用户头像',
  `gender` tinyint NULL DEFAULT NULL COMMENT '性别',
  `userRole` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL DEFAULT 'user' COMMENT '用户角色：user / admin',
  `userPassword` varchar(512) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL COMMENT '密码',
  `mobile` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL COMMENT '手机号',
  `createTime` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updateTime` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `isDelete` tinyint NOT NULL DEFAULT 0 COMMENT '是否删除',
  PRIMARY KEY (`id`) USING BTREE,
  UNIQUE INDEX `uni_userAccount`(`userAccount`) USING BTREE,
  UNIQUE INDEX `uni_mobile`(`mobile`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_0900_ai_ci COMMENT = '用户' ROW_FORMAT = Dynamic;

```

大多数人的写法
-------

### SQL 写法

```
SELECT COUNT(*) FROM user WHERE gender = '女';

```

### Java 写法

```
QueryWrapper<User> queryWrapper = new QueryWrapper();
queryWrapper.eq("gender", '女');
int countNum = userService.count(queryWrapper);
if (countNum > 0) {
    //...
} else {
    //...
}

```

优化方案
----

在业务代码中，根据一个或者一组条件，查询记录是否存在，并不关心记录有多少条。

### SQL 写法

```
SELECT 1 FROM user WHERE gender = '女' LIMIT 1;

```

### Java 写法

```
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.select("1")
            .eq("gender", "女")
            .last("LIMIT 1");
User user = userMapper.selectOne(queryWrapper);
if (user != null) {
    //...
} else {
    //...
}

```

拓展知识
----

### `SELECT 1 FROM user WHERE gender = '女' LIMIT 1;` 解释

这是一条 SQL 查询语句，它的作用是从名为 "用户"（user）的表中选择一个性别为女（gender = '女'）的记录，并且只返回第一条匹配项（LIMIT 1）。

具体而言，SELECT 1 表示只需要返回一个值，因此并不会返回整个记录。如果有多个符合条件的记录，也只会返回第一条记录。

在该表中，可能会包含其他字段，例如用户名、密码等等，但在此查询中，我们只关心返回一个值，以便验证表中是否存在至少一个女性用户。

`SELECT 1 和 LIMIT 1`是否重复？
-------------------------

SELECT 1 和 LIMIT 1 不重复，它们之间有不同的作用。

SELECT 1 是一条 SQL 查询语句，表示要选取表中的数据列或者返回一个常量。下面这个示例中，SELECT 1 返回了一个常量值 "1"：

```
SELECT 1 FROM table;

```

常见的用途是检查表是否存在或者用于查询返回布尔值（true/false）。

LIMIT 1 也是一条 SQL 语句，用于限制查询结果的返回行数。例如：

```
SELECT * FROM table LIMIT 1;

```

这将返回 “table” 表中的第一行（行号为 1）。

因此，两条语句具有不同的作用：SELECT 1 可以返回一个常量值，而 LIMIT 1 则限制返回的行数，并不会返回任何特定的值。在某些情况下，可以将两者组合使用，例如：

```
SELECT 1 FROM table WHERE id = 1234 LIMIT 1;

```

该查询将返回 id 等于 1234 的记录中的第一行，如果不存在则返回空结果集。