> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.luymm.com](https://www.luymm.com/archives/404/)

> 新型冠状肺炎还在世界各地肆虐，许多国家都出现疫情病例，每天的疫情数据也在不断的变化，一天一个样，现在很多的平台和公司都提供平台去提供疫情每天变化数据，以下是我搜集的一些疫情开源接口。疫情接口 1 ...

![](https://pic.luymm.com/2020/05/31/742bdcdef086f.png)  
新型冠状肺炎还在世界各地肆虐，许多国家都出现疫情病例，每天的疫情数据也在不断的变化，一天一个样，现在很多的平台和公司都提供平台去提供疫情每天变化数据，以下是我搜集的一些疫情开源接口。

### 疫情接口

##### 1 getpostman

**URL：** [https://documenter.getpostman.com/view/2568274/SzS8rjbe?version=latest#license](https://www.luymm.com/go/aHR0cHM6Ly9kb2N1bWVudGVyLmdldHBvc3RtYW4uY29tL3ZpZXcvMjU2ODI3NC9TelM4cmpiZT92ZXJzaW9uPWxhdGVzdCNsaWNlbnNl)  
**说明**：提供接口功能比较完善，接口文档也比较详细，缺点数据滞后  
**推荐指数**：★★★

#### 2 covid19api

**URL**: [https://covid19api.com/](https://www.luymm.com/go/aHR0cHM6Ly9jb3ZpZDE5YXBpLmNvbS8=)  
说明：接口提供的功能完成，提供详细的国家列表，数据同步更新，缺点特定国家不更新【最近中国疫情数据为空】  
**推荐指数**：★★★★

#### 3 github 疫情数据文件

**URL**: [https://github.com/CSSEGISandData/COVID-19](https://www.luymm.com/go/aHR0cHM6Ly9naXRodWIuY29tL0NTU0VHSVNhbmREYXRhL0NPVklELTE5)  
说明：提供国家到省的详细疫情数据，数据更新也比较及时，缺点需要自己按维度去统计国家数据  
**推荐指数**：★★★★

#### 总结

网上提供开源接口都是人为或者热心的开发者提供，数据终归是缺乏准确性和及时性的，要做到数据的准确性最后人为的去抓取和统计，在设计接口过程中的小建议、直接统计每天累计的疫情数据，如果要每天的增量数据，在接口中计算，采用多个数据源互补来保证数据的准确性。