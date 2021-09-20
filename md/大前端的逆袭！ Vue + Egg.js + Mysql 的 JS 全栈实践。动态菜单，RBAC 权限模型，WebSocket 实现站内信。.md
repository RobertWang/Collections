> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7007212688866541576)

如果对您有帮助，您可以点右上角 "Star" 支持一下 谢谢！ ^_^

> 或者您可以 "follow" 一下，我会不断开源更多的有趣的项目。如：Vue3 + NestJS + TypeScript ✨

> 如有问题请直接在 Issues 中提，或者您发现问题并有非常好的解决方案，欢迎 PR 👍

目标功能
----

*   [x]  登录、注册 -- 完成
*   [x]  github 授权登录 -- 完成
*   [x]  找回密码 -- 完成
*   [x]  滑块验证 -- 完成
*   [x]  邮箱验证 -- 完成
*   [x]  动态首页 -- 完成
*   [x]  个人设置 -- 完成
*   [x]  用户管理 -- 完成
*   [x]  角色管理 -- 完成
*   [x]  菜单管理 -- 完成
*   [x]  资源管理 -- 完成
*   [x]  操作日志 -- 完成
*   [x]  动态菜单 -- 完成
*   [x]  部门管理 -- 完成
*   [x]  项目列表 -- 完成
*   [x]  任务看板 -- 完成
*   [x]  任务列表 -- 完成
*   [x]  项目文件 -- 完成
*   [x]  项目概览 -- 完成
*   [x]  项目成员 -- 完成
*   [x]  项目邀请 -- 完成
*   [x]  项目设置 -- 完成
*   [x]  项目回收站 -- 完成
*   [x]  任务筛选 -- 完成
*   [x]  任务详情 -- 完成
*   [x]  任务标签 -- 完成
*   [x]  任务参与者 -- 完成
*   [x]  任务动态 -- 完成
*   [x]  任务工时 -- 完成
*   [x]  任务关联文件 -- 完成
*   [x]  任务更新即时同步 -- 完成
*   [x]  公开项目的业务权限控制（非项目成员不可编辑项目） -- 完成
*   [x]  项目模板 -- 完成
*   [x]  消息提醒 -- 完成
*   [x]  工作台 -- 完成
*   [x]  站内信 -- 完成
*   [x]  页面元素权限控制 -- 完成
*   [ ]  项目版本 -- 待开发
*   [ ]  项目日程 -- 待开发

部分截图
----

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/98bd6c8cc8934a468323a727a0418c14~tplv-k3u1fbpfcp-watermark.awebp)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf106182c18b4f34b93d0b03a51c6f66~tplv-k3u1fbpfcp-watermark.awebp)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/536aa525ed88463aae1ba10de25ec9fc~tplv-k3u1fbpfcp-watermark.awebp)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca8fdca695024a4b8d7a2728b717adad~tplv-k3u1fbpfcp-watermark.awebp)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89a573d49b2646c6aa35cd74a7bbb680~tplv-k3u1fbpfcp-watermark.awebp)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0d5b6678731a4092ae53289ee367c008~tplv-k3u1fbpfcp-watermark.awebp)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a9a31bc814104964b5e63b4a0e6dd039~tplv-k3u1fbpfcp-watermark.awebp)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6c726195939f4823a27d9a56e7150fdc~tplv-k3u1fbpfcp-watermark.awebp)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d49517621874f819df6991107c68252~tplv-k3u1fbpfcp-watermark.awebp)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/30038b8cbfed4cc2aa7e8dc8d03aaf81~tplv-k3u1fbpfcp-watermark.awebp)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e8ddd9c5eb4b4219a6008b9944cb59bc~tplv-k3u1fbpfcp-watermark.awebp)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e3caf807ee6a4c108d0bf533117ea9d6~tplv-k3u1fbpfcp-watermark.awebp)

后端 egg 项目部署
-----------

#### 运行环境：

Node.js >= v10; Mysql >= 5.7; Redis >= 5.0;

```
git clone https://github.com/Imfdj/egg-beehive.git

cd egg-beehive

npm install 或 yarn(推荐)

将database目录下的egg-beehive-dev.sql和egg-beehive-test.sql导入mysql（推荐navicat）。

在config目录下的config.local.js和config.unittest.js中的exports.sequelize、exports.redis、exports.io.redis下填入Mysql和Redis的配置参数

npm run dev

npm run test-local (单元测试)
复制代码
```

#### 如何快速 CRUD：

```
在generator文件夹中的config.js文件中定义各个字段的描述，完成后执行npm run generator-entity。
里面还有很多config-*.js的配置文件可供参考。也可以在template文件夹中自定义各个文件的模板。

// 这是一个字段的描述模板
fieldsItemExample: {
    name: 'xx_id',
    type: 'INTEGER',
    length: 11,
    min: 1,
    max: 1,
    required: true,
    description: '这里是描述', // 供swagger使用
    primaryKey: false, // 是否为主键
    unique: false, // 是否唯一
    allowNull: false, // 是否允许为空
    autoIncrement: false, // 是否自增
    defaultValue: '', // 数据库表中字段的默认值
    comment: '外键', // 数据库表中字段的描述
    references: {
      // 外键设置
      model: 'xxxs', // 外键关联表
      key: 'id', // 外键字段名
    },
    onUpdate: 'NO ACTION', // 外键更新约束 CASCADE RESTRICT SET NULL SET DEFAULT NO ACTION
    onDelete: 'NO ACTION', // 外键删除约束 CASCADE RESTRICT SET NULL SET DEFAULT NO ACTION
}
复制代码
```

前端 vue 项目部署
-----------

```
git clone https://github.com/Imfdj/vue-beehive.git

cd vue-beehive

npm install 或 yarn(推荐)

npm run serve
复制代码
```

功能设计
----

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3a57ff9d3eb8482691bc50aa5e029bf9~tplv-k3u1fbpfcp-watermark.awebp)

后端设计
----

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ae66afbb7bc24cb5af6b07249f99c88a~tplv-k3u1fbpfcp-watermark.awebp)

数据库设计
-----

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6bd0481b00f4475fa77a970db1df5011~tplv-k3u1fbpfcp-watermark.awebp)

### License

[MIT](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FImfdj%2Fvue-beehive%2Fblob%2Fmaster%2FLICENSE "https://github.com/Imfdj/vue-beehive/blob/master/LICENSE")

Copyright (c) 2021 Imfdj