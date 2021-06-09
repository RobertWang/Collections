> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI3MjE4MjIyOA==&mid=2651545558&idx=1&sn=5f3d1411fe9810ec3518561bf34dc431&chksm=f0c9816dc7be087bfe9e5c26914c24db24e3153f84520d4a7be0f7e45431c55d9f9078af4ac7&mpshare=1&scene=1&srcid=0608zYwySCKV4B0htMhXlW0Y&sharer_sharetime=1623131579445&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

#### 项目介绍

TIMO 后台管理系统，基于 SpringBoot2.0 + Spring Data Jpa + Thymeleaf + Shiro 开发的后台管理系统，采用分模块的方式便于开发和维护，支持前后台模块分别部署，目前支持的功能有：权限管理、部门管理、字典管理、日志记录、文件上传、代码生成等，为快速开发后台系统而生的脚手架！

#### 技术选型

*   后端技术：SpringBoot + Spring Data Jpa + Thymeleaf + Shiro + Jwt + EhCache
    
*   前端技术：Layui + Jquery + zTree + Font-awesome
    

#### 全新的项目结构

#### 功能列表

*   用户管理：用于管理后台系统的用户，可进行增删改查等操作。
    
*   角色管理：分配权限的最小单元，通过角色给用户分配权限。
    
*   菜单管理：用于配置系统菜单，同时也作为权限资源。
    
*   部门管理：通过不同的部门来管理和区分用户。
    
*   字典管理：对一些需要转换的数据进行统一管理，如：男、女等。
    
*   行为日志：用于记录用户对系统的操作，同时监视系统运行时发生的错误。
    
*   文件上传：内置了文件上传接口，方便开发者使用文件上传功能。
    
*   代码生成：可以帮助开发者快速开发项目，减少不必要的重复操作，花更多精力注重业务实现。
    
*   表单构建：通过拖拽的方式快速构建一个表单模块。
    
*   数据接口：根据业务代码自动生成相关的 api 接口文档
    

#### 安装教程

*   ##### 环境及插件要求
    

*   Jdk8+
    
*   Mysql5.5+
    
*   Maven
    
*   Lombok（重要）
    

*   ##### 导入项目
    

*   IntelliJ IDEA：Import Project -> Import Project from external model -> Maven
    
*   Eclipse：Import -> Exising Mavne Project
    

*   ##### 运行项目
    

*   通过 Java 应用方式运行 admin 模块下的 com.linln.BootApplication.java 文件
    
*   数据库配置：数据库名称 timo 用户 root 密码 root
    
*   访问地址：http://localhost:8080/
    
*   默认帐号密码：admin/123456
    

#### 使用说明

1.  使用文档：sdoc / 使用文档. docx
    
2.  开发手册：TIMO 开发文档. 在线
    
3.  SQL 文件：sdoc/timo.sql（经常忘记同步！）
    

#### 更新记录

*   2019-11-06 更新 重命名菜单类型为：目录、菜单、按钮
    
*   2019-10-30 更新 重写 Shiro“记住我” 系列化数据，减少 cookie 体积
    
*   2019-10-25 更新 加入获取用户角色列表方法，修复获取部门数据时延迟加载超时问题
    
*   2019-10-17 更新 1. 优化登录 js 加载不及时问题 2. 解决 layui 弹出窗口最大化问题
    
*   2019-08-28 更新 加入配置项，可直接通过 yml 文件配置 Shiro 和 XSS 防护忽略规则！
    
*   2019-08-11 更新 根据【阿里巴巴 Java 开发手册】对代码进行优化处理
    
*   2019-06-14 更新 修复接口无法继承多个父接口的问题
    
*   2019-04-28 更新 加入 JWT TOKEN 鉴权机制，实现多端的权限验证！
    
*   2019-04-07 更新 发布 v2.0 版本，带来全新的项目结构！
    
*   2019-02-11 更新 修复字典值过长格式显示问题，加入离线技术文档 1.0
    
*   2019-01-17 更新 重构字典模块，将 mo:dictKey 改成 mo:dict
    
*   2018-12-18 更新 加入导入导出功能
    
*   2018-12-13 更新 修复代码生成路径出现空格的问题
    
*   2018-12-10 更新 1. 加入 xss 防护 2. 加入 swagger 数据接口文档
    
*   2018-12-09 更新 1. 加入 QuerySpec 动态查询实例 2. 加入排序选择功能 3. 完善用户部门查询
    
*   2018-12-07 更新 修复文件上传多次的问题，修改上传实体类名称
    
*   2018-12-05 更新 1. 支持三级菜单 2. 更新管理员权限机制 3. 修复若干问题
    
*   2018-12-03 更新 1. 加入部门管理功能 2. 更新开源协议 3. 修复若干问题
    
*   2018-12-01 更新 发布 v1.0 版本！
    

#### 演示地址

演示地址：http://www.linln.cn

#### 预览图

![](https://mmbiz.qpic.cn/mmbiz_jpg/OND71iceJn3CKrvicun3U95e3JwByt6iaQViaiaibxoSl4EKiaic87pgWvbEWa6rWFRnVtRUicwu0Ribo6XRfeZQ6cpVVFDQ/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/OND71iceJn3CKrvicun3U95e3JwByt6iaQVrWCpVdlklib3ia6n3iaM28hsGYu8Xnoc1icyd6D3axibnJY7iaOlOQKes2Gg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/OND71iceJn3CKrvicun3U95e3JwByt6iaQVKWkPDK2m45Tco6SictryaWRvOjhrzWefJ1T8uo7nBqe9YicjNlZ5C0Bg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/OND71iceJn3CKrvicun3U95e3JwByt6iaQVR0Idnic4lewTPofebSwX7eWVk2CRU7LosPmeGC75q1n2sgT2yfNX8Vg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/OND71iceJn3CKrvicun3U95e3JwByt6iaQVbr4HYx7ickKNBShMYSUYcGB1f9WtURtjia7ASlmTyf9TKficbpcf12PBQ/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/OND71iceJn3CKrvicun3U95e3JwByt6iaQV7zibWBR0g0IicibjNbYBT0Yia1ZtDExZN8rtbouyiaUvQ6vtOicaKC6iaPDQQ/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/OND71iceJn3CKrvicun3U95e3JwByt6iaQVQerZaGOXXbAQaRNVgMPCnvN1hYb0mV738k1RTWRT65NopAJl57ly4w/640?wx_fmt=jpeg)

开源地址：https://gitee.com/aun/Timo

公众号

其它推荐：  

[项目申报系统 html 完整模版](http://mp.weixin.qq.com/s?__biz=MzI3MjE4MjIyOA==&mid=2651545539&idx=1&sn=0a57264ad9a2372fe0f1df32fbcbf50c&chksm=f0c98178c7be086ef4d37b3b90fe18fc5239c14c5ed28fe4f4315c6ff48fe11db7d6b5591eda&scene=21#wechat_redirect)  

[双皮肤全套后台管理系统 html 模版页面](http://mp.weixin.qq.com/s?__biz=MzI3MjE4MjIyOA==&mid=2651545515&idx=1&sn=014498a1a2ec59eef9962d055b9077fe&chksm=f0c98110c7be0806fe781fb29055595670f3fc57378167f868a286599017f48edab89b552082&scene=21#wechat_redirect)