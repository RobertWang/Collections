> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/wings-xh/p/12000864.html)

1 前后端分离
=======

1.1 后端  

---------

　　ssm+maven 多模块

　　swagger 文档描述（代码拷贝过来，就可以生成了，[https://www.cnblogs.com/wings-xh/p/11991511.html](https://www.cnblogs.com/wings-xh/p/11991511.html)  里面的 swagger）  

　　postman 测试 

　　请求 put / get / post / patch / delete

1.2 前端  

---------

Nodejs：前端服务

npm：管理 js 库 依赖关系

webpack：对静态资源打包

vuejs:MVVM(model view view Model 双向绑定) 模式的开发 js 库

ElementUI：前端 ui 框架

　　vuecli:vue 开发脚手架，能够快速搭建 vue 项目，里面包含了 webpack 的打包配置，服务器热启动

2 后端的 crud
==========

(1) mapper.xml 中写 select、insert、update、delete

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
<mapper namespace="cn.itsource.crm.mapper.DepartmentMapper">
    <select resultType="Department">
        select * from t_department
    </select>
    <select parameterType="long" resultType="Department">
        select * from t_department where id=#{id}
    </select>
    <insert parameterType="Department">
        insert into t_department(name) values(#{name})
    </insert>
    <update parameterType="Department">
        update t_department set name=#{name} where id=#{id}
    </update>
    <delete parameterType="long">
        delete from t_department where id=#{id}
    </delete>
</mapper>
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

(2) 在对应的 controller 写上 crud 方法

3 前端界面处理
========

(1) 配置一个 xxx.vue 页面

(2) 配置路由

(3) main.js 引入 axios

(4) 注释掉模拟数据

(5) 修改 xxx.vue 这里面的内容

4 前端访问后端
========

4.1 哪些情况下会存在跨域
--------------

(1) 同一个域名，不同的端口 =》 属于跨域  localhost:8080 --> localhost:80

(2) 不同域名 =》 属于跨域  192.168.0.1 --> 192.168.0.2

(3) 二级域名不同 =》 属于跨域  miaosha.jd.com --> qianggou.jd.com

4.2 跨域问题
--------

浏览器（同源策略）针对 ajax 请求，如果访问的是不同的域名，或者不同的端口，就会存在跨域问题。

同源策略：只允许相同协议，相同域名，相同的端口

4.3 跨域问题怎么解决
------------

(1) jsonp -- 很早的

动态构造的标签，标签 <script> 去访问资源

 缺陷：get 请求 / 需要服务支持

(2) 通过 nginx（部署）--> 解决跨域问题 --> 反向代理机制

类似中间商，把访问后台的请求转换为访问自己，让 nginx 去访问后台，把结果拿回再转给前台。

缺点：部署服务，做配置

**(3) CORS 机制：跨域资源共享 "(Cross-origin resource sharing)"**

解决跨域问题：浏览器有同源策略（相同协议，相同域名，相同端口），如果不是同源，存在跨域问题，

浏览器会将 ajax 请求分为两类，其处理方案略有差异：简单请求、特殊请求

　　简单请求，发送一次，后台服务需要运行访问，

　　特殊请求，发送两次，第一次是遇见请求，服务器也要运行预检，前台发现预检通过，再发送真实请求，真实请求有需要服务器允许

可以使用注解一键配置：@CrossOrigin

**注意：spring 版本需要是 4.2.5 以后才支持注解**

5 完成前端的 crud
============

(1) 列表展示（一定要挂载）

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
/获取部门列表
getDepartments() {
    this.listLoading = true;
    //NProgress.start();
    this.$http.patch("/department/list").then((res) => {
        // this.total = res.data.total;
        this.departments = res.data;
        this.listLoading = false;
    });
}
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

挂载代码：

```
mounted() {
    this.getDepartments();
}
```

(2) 新增

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
//显示新增界面
handleAdd() {
    this.addFormVisible = true;
    this.addForm = {
        name: ''
    };
},
//新增
addSubmit() {
    this.$refs.addForm.validate((valid) => {
        if (valid) {
            this.$confirm('确认提交吗？', '提示', {}).then(() => {
                //加载
                this.addLoading = true;
                //拷贝 this.addForm这个对象 para = {name:'xxx'}
                let para = Object.assign({}, this.addForm);
                this.$http.put("/department",para).then((res) => {
                    this.addLoading = false;
                    this.$message({
                        message: '提交成功',
                        type: 'success'
                    });
                    //验证的重置
                    this.$refs['addForm'].resetFields();
                    //关闭新增对话框
                    this.addFormVisible = false;
                    this.getDepartments();
                });
            });
        }
    });
}
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

(3) 编辑修改

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
//显示编辑界面
handleEdit(index, row) {
    this.editFormVisible = true;
    this.editForm = Object.assign({}, row);
},
//编辑
editSubmit() {
    this.$refs.editForm.validate((valid) => {
        if (valid) {
            this.$confirm('确认提交吗？', '提示', {}).then(() => {
                this.editLoading = true;
                //NProgress.start();
                let para = Object.assign({}, this.editForm);
                this.$http.post("/department",para).then((res) => {
                    this.editLoading = false;
                    //NProgress.done();
                    this.$message({
                        message: '提交成功',
                        type: 'success'
                    });
                    this.$refs['editForm'].resetFields();
                    this.editFormVisible = false;
                    this.getDepartments();
                });
            });
        }
    });
}
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

(4) 删除

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
//删除
handleDel(index, row) {
    this.$confirm('确认删除该记录吗?', '提示', {
        type: 'warning'
    }).then(() => {
        this.listLoading = true;
        this.$http.delete("/department/" + row.id).then((res) => {
            this.listLoading = false;
            this.$message({
                message: '删除成功',
                type: 'success'
            });
            this.getDepartments();
        });
    }).catch(() => {

    });
}
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")