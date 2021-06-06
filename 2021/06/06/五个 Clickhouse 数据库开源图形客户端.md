> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1622874365&ver=3111&signature=Dk3E2fNB-vSLli9Xtp*9QOAPgVwkMIXnKhUTVF9ZzWtxGKc3R1BZpTS48gBX9akzjZQZPQwjGR6wQ6HEPDaKFUvsrMGmoxWhuHfjddUIqMTL0HWq8gxj3NqfOG9yThoN&new=1)

俄罗斯搜索巨头 Yandex 开发的面向列存的关系型数据库。ClickHouse 是过去两年中 OLAP 领域中非常热门，并于 2016 年开源。典型的用户包括著名的公司，例如字节，新浪和腾讯。

从 DBEngine 给出的趋势来看，自打开源以来 Clickhouse 被关注的趋势上升明显。

![](https://mmbiz.qpic.cn/mmbiz_png/OKUeiaP72uRwRl9cs7cQbfJib7Qs6rgsUWmKjqcbWy5Qme1PZbBaGycU3Tvt6mvU2230QIgdGFTHEcPnnRsoAKUw/640?wx_fmt=png)

使用数据库，一款趁手的客户端查询工具非常重要，这里我就给大家推荐几款好用的支持 Clickhouse 的图形客户端。

### TabixUI

https://github.com/tabixio/tabix

Tabix 是一个纯前端，可以运行在浏览器中的 Clickhouse 的客户端，它为 Clickhouse 量身定制，又比较轻量。

它用到了以下的开源库：

*   *   Ajax.org 的 Ace.JS
    

*   百度 eCharte
    
*   Handsontable
    
*   Lodash
    
*   pivottable
    

Tabix 支持以下的功能：

*   直接从浏览器连接 ClickHouse 一起使用，而无需安装其他软件；
    
*   查询编辑器，支持突出高亮 SQL 语法，对所有对象自动完成，包括字典和内置函数的上下文相关帮助。
    
*   用于映射查询结果的图形，图表和地理参考，包含 map / bar / heatmap / river / sankeys / treemap
    
*   用于查询结果的交互式设计器数据透视表（pivot）；
    
*   用于分析 ClickHouse 的图形工具；
    
*   两种颜色的主题：浅色和深色。
    
*   支持 Clickhouse 的 Metrics
    
*   支持 Clickhouse 的 Schema
    

![](https://mmbiz.qpic.cn/mmbiz_png/OKUeiaP72uRwRl9cs7cQbfJib7Qs6rgsUW3kqzQcUHTFNiaYwAv5WKcvJbr8zF98icHjWqRf2Yt18O4vKUN3HtkDwg/640?wx_fmt=png)

### SQLPad

https://github.com/sqlpad/sqlpad

SQLPad 一个 Web 应用程序，用于编写和运行 SQL 查询并可视化结果。通过 ODBC 支持 Postgres，MySQL，SQL Server，ClickHouse，Crate，Vertica，Presto，SAP HANA，Cassandra，Snowflake，Google BigQuery，SQLite 等。

![](https://mmbiz.qpic.cn/mmbiz_png/OKUeiaP72uRwRl9cs7cQbfJib7Qs6rgsUWkKcO5lx168mHTFBtiaibUyMeRibvfMaxO0xxHYhcoAxdaBSJgFiavEQxMw/640?wx_fmt=png)

SQLPad 支持几种基本的图表类型，包含 Line，Bar，Scatter Plot，Stacked Bar。

SQLPad 采用了 Client/Server 架构，Server 使用了 Nodejs，客户端是 React，图表库使用了 D3.js

如果你的工作需要用到除了 Clickhouse 之外的这几种支持 ODBC 的数据库，SQLPad 可以一用。

### Superset

https://superset.apache.org/

Superset 是 Airbnb 开源的 BI 和数据可视化工具箱。Superset 快速，轻巧，直观，并带有各种选项，使各种技能的用户都可以轻松浏览和可视化其数据，从简单的折线图到高度详细的地理空间图。

目前，Superset 已在许多公司大规模运行。例如，Superset 在 Kubernetes 内的 Airbnb 生产环境中运行，每天为 600 多个活跃用户提供服务，每天查看超过 10 万张图表。

![](https://mmbiz.qpic.cn/mmbiz_jpg/OKUeiaP72uRwRl9cs7cQbfJib7Qs6rgsUWhsBBFHtmwyo9DbHTkicQ5uXoOgcPVibzyspIDq2FicxiaCj654icv423uaw/640?wx_fmt=jpeg)

Clickhouse 可用的五个图形客户端

Superset 提供以下的功能：

*   直观的界面，用于可视化数据集和制作交互式仪表板
    
*   多种精美的可视化展示数据
    
*   无代码可视化构建器，用于提取和呈现数据集
    
*   世界一流的 SQL IDE，用于准备数据以进行可视化，其中包括丰富的元数据浏览器
    
*   轻量级的语义层，使数据分析人员能够快速定义自定义维度和指标
    
*   对大多数说 SQL 的数据库提供开箱即用的支持
    
*   无缝的内存中异步缓存和查询
    
*   一种可扩展的安全模型，允许配置关于谁可以访问哪些产品功能和数据集的非常复杂的规则。
    
*   与主要的身份验证后端（数据库，OpenID，LDAP，OAuth，REMOTE_USER 等）集成
    
*   添加自定义可视化插件的功能
    
*   用于程序化定制的 API
    
*   云原生架构，专为大规模应用而设计
    

### Redash

https://github.com/getredash/redash

Redash 旨在使任何人，无论技术水平如何，都可以利用数据的力量。SQL 用户可以利用 Redash 来探索，查询，可视化和共享来自任何数据源的数据。他们的工作反过来使组织中的任何人都可以使用数据。每天，全球成千上万个组织中的数百万用户使用 Redash 来开发见解并制定数据驱动的决策。

![](https://mmbiz.qpic.cn/mmbiz_png/OKUeiaP72uRwRl9cs7cQbfJib7Qs6rgsUW47tnvNeC0CtuyNicj5HaFqkyAGfJvocdY9PyUrmt1Y9NFkYzqV2q4sQ/640?wx_fmt=png)

Redash 背后的公司创建于 2015 年，并于 2020 年被 Spark 的所有公司 Databrick 收购。

Redash 功能：

*   基于浏览器：浏览器支持所有内容，均带有可共享的 URL。
    
*   易于使用：无需掌握复杂软件即可立即获得数据生产力。
    
*   查询编辑器：使用浏览器快速编辑 SQL 和 NoSQL 查询并自动完成。
    
*   可视化和仪表板：通过拖放创建漂亮的可视化文件，并将它们组合到单个仪表板中。
    
*   共享：通过共享可视化文件及其关联的查询轻松进行协作，从而实现对报告和查询的同行审阅。
    
*   计划刷新：自定义的定期自动更新图表和仪表板。
    
*   告警：定义条件并在数据更改时立即收到警报。
    
*   REST API：UI 中可以完成的所有操作也可以通过 REST API 获得。
    
*   对数据源的广泛支持：可扩展的数据源 API，具有对一长串常见数据库和平台的本机支持。