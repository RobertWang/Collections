> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI4ODE5OTIyMQ==&mid=2650357971&idx=1&sn=f57b35bccdb0fe47d1d7fd898c9cc668&chksm=f3cfe789c4b86e9fb7bd149063291ab85b53a89c48548232be61eaf88a62f75a6ef576f49ed1&scene=21#wechat_redirect)

> 你可以这么理解 SaaS 系统就像一栋大楼，而租户就是大楼里面租办公楼层的公司，平时每家公司做着自己的业务，互不干扰，但是一旦大楼的电梯坏了，那么影响到的就是所有的公司。

背景
==

开发过`SaaS`系统平台的小伙伴一定对`多租户`这个概念不陌生，简单来说一个租户就是一个公司客户，多个租户共用同一个 SaaS 系统，一旦 SaaS 系统不可用，那么所有的租户都不可用。你可以这么理解 SaaS 系统就像一栋大楼，而租户就是大楼里面租办公楼层的公司，平时每家公司做着自己的业务，互不干扰，但是一旦大楼的电梯坏了，那么影响到的就是所有的公司。

多租户问题，其是一种架构设计方式，就是在一台或者一组服务器上运行的 SaaS 系统，可以为多个租户（客户）提供服务，目的是为了让多个租户在互联网环境下使用同一套程序，且保证租户间的数据隔离。从这种架构设计的模式上，不难看出来，多租户架构的重点就是同一套程序下多个租户数据的隔离。由于租户数据是集中存储的，所以要实现数据的安全性，就是看能否实现对租户数据的隔离，防止租户数据不经意或被他人恶意地获取和篡改。在讲多租户数据隔离实现之前，先来看看什么是`SaaS系统`。

什么是 SaaS 系统？
============

SaaS 平台是运营 saas 软件的平台。SaaS 提供商为企业搭建信息化所需要的所有网络基础设施及软件、硬件运作平台，并负责所有前期的实施、后期的维护等一系列服务，租户 (企业) 无需购买软硬件、建设机房、招聘 IT 人员，即可通过互联网使用信息系统。SaaS 是一种软件布局模型，其应用专为网络交付而设计，便于用户通过互联网托管、部署及接入。

简单来说就是租户给 SaaS 平台付租金就能使用平台提供的功能服务，当下比较典型就是各种云平台、云服务厂商。

多租户数据隔离架构设计
===========

目前 saas 多租户系统的数据隔离有三种架构设计，即为每个租户提供独立的数据库、独立的表空间、按字段区分租户，每种方案都有其各自的适用情况。

**一个租户独立一个数据库**

一个租户独立使用一个数据库，那就意味着我们的 SaaS 系统需要连接多个数据库，这种实现方案其实就和分库分表架构设计是一样的，好处就是数据隔离级别高、安全性好，毕竟一个租户单用一个数据库，但是物理硬件成本，维护成本也变高了。

**独立的表空间**

这种方案的实现方式，就是所有租户共用一个数据库系统，但是每个租户在数据库系统中拥有一个独立的表空间。

**按租户 id 字段隔离租户**

这种方案是多租户方案中最简单的数据隔离方法，即在每张表中都添加一个用于区分租户的字段（如 tenant_id 或 org_id 啥的）来标识每条数据属于哪个租户，当进行查询的时候每条语句都要添加该字段作为过滤条件，其特点是所有租户的数据全都存放在同一个表中，数据的隔离性是最低的，完全是通过字段来区分的，很容易把数据搞串或者误操作。

三种数据隔离架构设计的对比如下：

<table><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em;">隔离方案</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em;">成本</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em;">支持租户数量</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em;">优点</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em;">缺点</td></tr></thead><tbody><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">独立数据库系统</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">高</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">少</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">数据隔离级别高，安全性，可以针对单个租户开发个性化需求</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">数据库独立安装，物理成本和维护成本都比较高</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">独立的表空间</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">中</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">较多</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">提供了一定程度的逻辑数据隔离，一个数据库系统可支持多个租户</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">数据库管理比较困难，表繁多，同时数据修复稍复杂</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">按租户 id 字段区分</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">低</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">多</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">维护和购置成本最低，每个数据库能够支持的租户数量最多</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">隔离级别最低，安全性也最低</td></tr></tbody></table>

大部分公司都是采用第三种：**按租户 id 字段隔离租户**架构设计实现多租户数据隔离的。接下来我们就来看看代码层面怎么实现多租户数据隔离的。