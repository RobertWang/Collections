> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/5_349rhITLCIyT0eGl-qgg)

> 告别传统自动化测试，迈入录制回放的新世界大门！

一、背景

传统的接口自动化测试框架，测试数据，测试用例脚本都需要人工手动编写，比较耗时，且对于编程知识比较薄弱的同学来说，要编写脚本有比较高的学习成本。因此考虑是否有更简单的自动化实现方式，让自动化测试学习成本和脚本编写时间都能更低，能够更加高效的实施自动化。

基于此目的，选型了 python + httprunner + pytest + allure + mitmproxy 的框架。通过录制的方式可以一边操作，一边自动生成对应的测试数据、测试用例，实现录制回放的自动化测试效果。

二、HttpRunner 介绍

HttpRunner 是一款面向 HTTP(S) 协议的通用测试框架，只需编写维护一份 YAML/JSON 脚本，即可实现自动化测试。主要特征如下：

*   以 YAML 或 JSON 格式定义测试用例。
    
*   通过 har 文件即可自动生成接口文档、测试数据文件、测试用例。
    
*   支持 variables/ extract/ validate/hooks 机制，以创建复杂的测试方案。
    
*   使用 debugtalk.py 插件，自定义函数功能，可以在测试用例的任何部分使用。
    
*   使用 jmespath，提取和验证 json 响应非常简单。
    
*   包装了 pytest，pytest 的各类插件随时可用。
    
*   使用 allure，测试报告可以非常全面丰富。
    

httprunner 具体使用方法，可以参考：  
https://ontheway-cool.gitee.io/httprunner3doc_cn/

三、框架实现思路

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Vx46Ba7qgDshVDSDIDVJBPv9T4Ixy5KBJDjW42fAu6G7tHqzgOiaKNWibKuGn1HK56F7iciasVLW1nWoVl8lnbNb8w/640?wx_fmt=png)  

1、通过抓包工具 Charles/Fiddler 抓取请求生成. har 文件，使用 httprunner 的 har2case 功能，进一步封装为：make_case.py，提取接口信息转换为相应的：

*   接口文档：xxx_api.yml
    
*   测试数据：xxx_data.csv
    
*   测试用例：xxx_test.yml
    
*   测试用例信息管理文件：api_map.yml
    

如果觉得一个一个转换比较麻烦，也可以先保存一批. har 文件，然后使用批量转换功能：bulk_make_cases.py，一次性转换多个. har 文件。

2、通过步骤 1，某个接口自动化运行的基础信息都已经有了。这时如果同一个接口，要测试不同入参的多个场景，依然按步骤 1 一个一个请求手工去转换，则显得比较笨重。

因此使用 mitmproxy 的 mitmproxy.http.HTTPFlow 功能，进一步封装为: record_data.py，可以实现一边操作，一边自动抓取该接口的入参生成到测试数据文件中：xxx_data.csv。

3、至此，接口自动化录制功能已实现。接下来，通过 httprunner 以数据驱动的方式运行接口的测试用例，并生成 allure 的测试报告，完成整个自动化测试闭环。

四、框架项目结构

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Vx46Ba7qgDshVDSDIDVJBPv9T4Ixy5KBW74tMVRx52vhHBW126OBkExaNvPaXCWcwiaC0Ac10licLoPH9Ft6clAA/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Vx46Ba7qgDshVDSDIDVJBPv9T4Ixy5KBdVmgmlEErl8zNpuo0hk6hqVdUQiawQtdfoEVicL6MKQJT7gxRoJcz3aA/640?wx_fmt=png)

五、具体示例操作

5.1 获取. har 文件

  

  

  

HAR（HTTP 档案规范），是一个用来储存 HTTP 请求 / 响应信息的通用文件格式，基于 JSON。这种格式的数据可以使 HTTP 监测工具以一种通用的格式导出所收集的数据，这些数据可以被其他支持 HAR 的 HTTP 分析工具使用。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Vx46Ba7qgDshVDSDIDVJBPv9T4Ixy5KBY4BrpP3M5uo4U4zPvQsA7xBVdBxibu9fOlqnIOjEm90FmbbxzvicTuqw/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Vx46Ba7qgDshVDSDIDVJBPv9T4Ixy5KBcjO5lNrKjR6dAet50KA1QKV482qwT6icAHlmoX3mkfUYn4tFRPSNYzg/640?wx_fmt=png)  
.har 文件记录着一个接口请求响应的一些基本信息，包括 url、请求方法、headers、param、body、response、请求响应时间等等相关信息。从 Charles 导出. har 文件之后，存放到项目中的 har 目录下。

5.2 转换成接口模版

单个. har 文件的转换，Terminal 窗口下执行以下命令：