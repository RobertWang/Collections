> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [cloud.tencent.com](https://cloud.tencent.com/developer/article/1699344?from=article.detail.1699382)

> 参考官方教程：https://clickhouse.tech/docs/en/getting-started/tutorial/

### 1、说明

参考官方教程：[https://clickhouse.tech/docs/en/getting-started/tutorial/](https://clickhouse.tech/docs/en/getting-started/tutorial/)

Yandex.Metrica is a web analytics service, and sample dataset doesn’t cover its full functionality, so there are only two tables to create: YandexMetrica 是一个网络分析服务，样本数据集不包括其全部功能，因此只有两个表可以创建:

*   hits is a table with each action done by all users on all websites covered by the service.
*   visits is a table that contains pre-built sessions instead of individual actions.
*   hits 是一个表格，其中包含所有用户在服务所涵盖的所有网站上完成的每个操作。
*   visits 是一个包含预先构建的会话而不是单个操作的表。

### 2、从准备好的分区获取表

#### 2.1 下载数据

由于公司内网限制，不能直接下载数据到服务器。

这里采用 Windows 系统下载数据，然后上传到 Linux 服务器

https://clickhouse-datasets.s3.yandex.net/hits/partitions/hits_v1.tar https://clickhouse-datasets.s3.yandex.net/visits/partitions/visits_v1.tar

![](https://ask.qcloudimg.com/http-save/yehe-1147621/sxvot7geut.png)

#### 2.2 导入数据

（1）上传至服务器

```
[root@elastic1 ~]# ll /tpdata/data/
total 4110280
-rw-r--r-- 1 root root 1271623680 Sep 14 13:59 hits_v1.tar
-rw-r--r-- 1 root root  563968000 Sep 14 13:44 visits_v1.tar
[root@elastic1 ~]#
```

（2）导入数据 注意 /var/lib/clickhouse 是当前数据目录。如果数据目录修改，则需要解压缩到对应的目录。

```
[root@elastic1 data]# tar xvf hits_v1.tar -C /var/lib/clickhouse
data/datasets/hits_v1/
data/datasets/hits_v1/format_version.txt
data/datasets/hits_v1/detached/
data/datasets/hits_v1/201403_10_18_2/
...
```

```
[root@elastic1 data]# tar xvf visits_v1.tar -C /var/lib/clickhouse
data/datasets/visits_v1/
data/datasets/visits_v1/format_version.txt
data/datasets/visits_v1/20140317_20140323_3_4_1/
...
```

（3）重启服务

```
[root@elastic1 data]# systemctl restart clickhouse-server
```

#### 2.3 简单统计

```
[root@elastic1 data]# clickhouse-client --password --query "SELECT COUNT(*) FROM datasets.hits_v1"
Password for user (default): 
[root@elastic1 data]# clickhouse-client --password --query "SELECT COUNT(*) FROM datasets.visits_v1"
Password for user (default): 
[root@elastic1 data]#
```

### 3、从压缩 TSV 文件获取表

#### 3.1 下载压缩文件

https://clickhouse-datasets.s3.yandex.net/hits/tsv/hits_v1.tsv.xz https://clickhouse-datasets.s3.yandex.net/visits/tsv/visits_v1.tsv.xz

```
[root@elastic1 data]# ll
total 6478640
drwxr-xr-x 3 root root       4096 Sep 14 15:09 data
-rw-r--r-- 1 root root 1271623680 Sep 14 13:59 hits_v1.tar
-rw-r--r-- 1 root root  841046840 Sep 14 16:37 hits_v1.tsv.xz
drwxr-xr-x 3 root root       4096 Sep 14 15:09 metadata
-rw-r--r-- 1 root root 3532492212 Sep 14 15:38 ontime.csv.xz
-rw-r--r-- 1 root root  563968000 Sep 14 13:44 visits_v1.tar
-rw-r--r-- 1 root root  424963796 Sep 14 16:27 visits_v1.tsv.xz
[root@elastic1 data]#
```

#### 3.2 新建数据表

（1）新建数据库

```
[root@elastic1 ~]# clickhouse-client --password --query "CREATE DATABASE IF NOT EXISTS tutorial"
Password for user (default): 
[root@elastic1 ~]#
```

（2）新建 tutorial.hits_v1

```
CREATE TABLE tutorial.hits_v1
(
    `WatchID` UInt64,
    `JavaEnable` UInt8,
    `Title` String,
    `GoodEvent` Int16,
    `EventTime` DateTime,
    `EventDate` Date,
    `CounterID` UInt32,
    `ClientIP` UInt32,
    `ClientIP6` FixedString(16),
    `RegionID` UInt32,
    `UserID` UInt64,
    `CounterClass` Int8,
    `OS` UInt8,
    `UserAgent` UInt8,
    `URL` String,
    `Referer` String,
    `URLDomain` String,
    `RefererDomain` String,
    `Refresh` UInt8,
    `IsRobot` UInt8,
    `RefererCategories` Array(UInt16),
    `URLCategories` Array(UInt16),
    `URLRegions` Array(UInt32),
    `RefererRegions` Array(UInt32),
    `ResolutionWidth` UInt16,
    `ResolutionHeight` UInt16,
    `ResolutionDepth` UInt8,
    `FlashMajor` UInt8,
    `FlashMinor` UInt8,
    `FlashMinor2` String,
    `NetMajor` UInt8,
    `NetMinor` UInt8,
    `UserAgentMajor` UInt16,
    `UserAgentMinor` FixedString(2),
    `CookieEnable` UInt8,
    `JavascriptEnable` UInt8,
    `IsMobile` UInt8,
    `MobilePhone` UInt8,
    `MobilePhoneModel` String,
    `Params` String,
    `IPNetworkID` UInt32,
    `TraficSourceID` Int8,
    `SearchEngineID` UInt16,
    `SearchPhrase` String,
    `AdvEngineID` UInt8,
    `IsArtifical` UInt8,
    `WindowClientWidth` UInt16,
    `WindowClientHeight` UInt16,
    `ClientTimeZone` Int16,
    `ClientEventTime` DateTime,
    `SilverlightVersion1` UInt8,
    `SilverlightVersion2` UInt8,
    `SilverlightVersion3` UInt32,
    `SilverlightVersion4` UInt16,
    `PageCharset` String,
    `CodeVersion` UInt32,
    `IsLink` UInt8,
    `IsDownload` UInt8,
    `IsNotBounce` UInt8,
    `FUniqID` UInt64,
    `HID` UInt32,
    `IsOldCounter` UInt8,
    `IsEvent` UInt8,
    `IsParameter` UInt8,
    `DontCountHits` UInt8,
    `WithHash` UInt8,
    `HitColor` FixedString(1),
    `UTCEventTime` DateTime,
    `Age` UInt8,
    `Sex` UInt8,
    `Income` UInt8,
    `Interests` UInt16,
    `Robotness` UInt8,
    `GeneralInterests` Array(UInt16),
    `RemoteIP` UInt32,
    `RemoteIP6` FixedString(16),
    `WindowName` Int32,
    `OpenerName` Int32,
    `HistoryLength` Int16,
    `BrowserLanguage` FixedString(2),
    `BrowserCountry` FixedString(2),
    `SocialNetwork` String,
    `SocialAction` String,
    `HTTPError` UInt16,
    `SendTiming` Int32,
    `DNSTiming` Int32,
    `ConnectTiming` Int32,
    `ResponseStartTiming` Int32,
    `ResponseEndTiming` Int32,
    `FetchTiming` Int32,
    `RedirectTiming` Int32,
    `DOMInteractiveTiming` Int32,
    `DOMContentLoadedTiming` Int32,
    `DOMCompleteTiming` Int32,
    `LoadEventStartTiming` Int32,
    `LoadEventEndTiming` Int32,
    `NSToDOMContentLoadedTiming` Int32,
    `FirstPaintTiming` Int32,
    `RedirectCount` Int8,
    `SocialSourceNetworkID` UInt8,
    `SocialSourcePage` String,
    `ParamPrice` Int64,
    `ParamOrderID` String,
    `ParamCurrency` FixedString(3),
    `ParamCurrencyID` UInt16,
    `GoalsReached` Array(UInt32),
    `OpenstatServiceName` String,
    `OpenstatCampaignID` String,
    `OpenstatAdID` String,
    `OpenstatSourceID` String,
    `UTMSource` String,
    `UTMMedium` String,
    `UTMCampaign` String,
    `UTMContent` String,
    `UTMTerm` String,
    `FromTag` String,
    `HasGCLID` UInt8,
    `RefererHash` UInt64,
    `URLHash` UInt64,
    `CLID` UInt32,
    `YCLID` UInt64,
    `ShareService` String,
    `ShareURL` String,
    `ShareTitle` String,
    `ParsedParams` Nested(
        Key1 String,
        Key2 String,
        Key3 String,
        Key4 String,
        Key5 String,
        ValueDouble Float64),
    `IslandID` FixedString(16),
    `RequestNum` UInt32,
    `RequestTry` UInt8
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(EventDate)
ORDER BY (CounterID, EventDate, intHash32(UserID))
SAMPLE BY intHash32(UserID)
SETTINGS index_granularity = 8192
```

（3）新建 tutorial.visits_v1

```
CREATE TABLE tutorial.visits_v1
(
    `CounterID` UInt32,
    `StartDate` Date,
    `Sign` Int8,
    `IsNew` UInt8,
    `VisitID` UInt64,
    `UserID` UInt64,
    `StartTime` DateTime,
    `Duration` UInt32,
    `UTCStartTime` DateTime,
    `PageViews` Int32,
    `Hits` Int32,
    `IsBounce` UInt8,
    `Referer` String,
    `StartURL` String,
    `RefererDomain` String,
    `StartURLDomain` String,
    `EndURL` String,
    `LinkURL` String,
    `IsDownload` UInt8,
    `TraficSourceID` Int8,
    `SearchEngineID` UInt16,
    `SearchPhrase` String,
    `AdvEngineID` UInt8,
    `PlaceID` Int32,
    `RefererCategories` Array(UInt16),
    `URLCategories` Array(UInt16),
    `URLRegions` Array(UInt32),
    `RefererRegions` Array(UInt32),
    `IsYandex` UInt8,
    `GoalReachesDepth` Int32,
    `GoalReachesURL` Int32,
    `GoalReachesAny` Int32,
    `SocialSourceNetworkID` UInt8,
    `SocialSourcePage` String,
    `MobilePhoneModel` String,
    `ClientEventTime` DateTime,
    `RegionID` UInt32,
    `ClientIP` UInt32,
    `ClientIP6` FixedString(16),
    `RemoteIP` UInt32,
    `RemoteIP6` FixedString(16),
    `IPNetworkID` UInt32,
    `SilverlightVersion3` UInt32,
    `CodeVersion` UInt32,
    `ResolutionWidth` UInt16,
    `ResolutionHeight` UInt16,
    `UserAgentMajor` UInt16,
    `UserAgentMinor` UInt16,
    `WindowClientWidth` UInt16,
    `WindowClientHeight` UInt16,
    `SilverlightVersion2` UInt8,
    `SilverlightVersion4` UInt16,
    `FlashVersion3` UInt16,
    `FlashVersion4` UInt16,
    `ClientTimeZone` Int16,
    `OS` UInt8,
    `UserAgent` UInt8,
    `ResolutionDepth` UInt8,
    `FlashMajor` UInt8,
    `FlashMinor` UInt8,
    `NetMajor` UInt8,
    `NetMinor` UInt8,
    `MobilePhone` UInt8,
    `SilverlightVersion1` UInt8,
    `Age` UInt8,
    `Sex` UInt8,
    `Income` UInt8,
    `JavaEnable` UInt8,
    `CookieEnable` UInt8,
    `JavascriptEnable` UInt8,
    `IsMobile` UInt8,
    `BrowserLanguage` UInt16,
    `BrowserCountry` UInt16,
    `Interests` UInt16,
    `Robotness` UInt8,
    `GeneralInterests` Array(UInt16),
    `Params` Array(String),
    `Goals` Nested(
        ID UInt32,
        Serial UInt32,
        EventTime DateTime,
        Price Int64,
        OrderID String,
        CurrencyID UInt32),
    `WatchIDs` Array(UInt64),
    `ParamSumPrice` Int64,
    `ParamCurrency` FixedString(3),
    `ParamCurrencyID` UInt16,
    `ClickLogID` UInt64,
    `ClickEventID` Int32,
    `ClickGoodEvent` Int32,
    `ClickEventTime` DateTime,
    `ClickPriorityID` Int32,
    `ClickPhraseID` Int32,
    `ClickPageID` Int32,
    `ClickPlaceID` Int32,
    `ClickTypeID` Int32,
    `ClickResourceID` Int32,
    `ClickCost` UInt32,
    `ClickClientIP` UInt32,
    `ClickDomainID` UInt32,
    `ClickURL` String,
    `ClickAttempt` UInt8,
    `ClickOrderID` UInt32,
    `ClickBannerID` UInt32,
    `ClickMarketCategoryID` UInt32,
    `ClickMarketPP` UInt32,
    `ClickMarketCategoryName` String,
    `ClickMarketPPName` String,
    `ClickAWAPSCampaignName` String,
    `ClickPageName` String,
    `ClickTargetType` UInt16,
    `ClickTargetPhraseID` UInt64,
    `ClickContextType` UInt8,
    `ClickSelectType` Int8,
    `ClickOptions` String,
    `ClickGroupBannerID` Int32,
    `OpenstatServiceName` String,
    `OpenstatCampaignID` String,
    `OpenstatAdID` String,
    `OpenstatSourceID` String,
    `UTMSource` String,
    `UTMMedium` String,
    `UTMCampaign` String,
    `UTMContent` String,
    `UTMTerm` String,
    `FromTag` String,
    `HasGCLID` UInt8,
    `FirstVisit` DateTime,
    `PredLastVisit` Date,
    `LastVisit` Date,
    `TotalVisits` UInt32,
    `TraficSource` Nested(
        ID Int8,
        SearchEngineID UInt16,
        AdvEngineID UInt8,
        PlaceID UInt16,
        SocialSourceNetworkID UInt8,
        Domain String,
        SearchPhrase String,
        SocialSourcePage String),
    `Attendance` FixedString(16),
    `CLID` UInt32,
    `YCLID` UInt64,
    `NormalizedRefererHash` UInt64,
    `SearchPhraseHash` UInt64,
    `RefererDomainHash` UInt64,
    `NormalizedStartURLHash` UInt64,
    `StartURLDomainHash` UInt64,
    `NormalizedEndURLHash` UInt64,
    `TopLevelDomain` UInt64,
    `URLScheme` UInt64,
    `OpenstatServiceNameHash` UInt64,
    `OpenstatCampaignIDHash` UInt64,
    `OpenstatAdIDHash` UInt64,
    `OpenstatSourceIDHash` UInt64,
    `UTMSourceHash` UInt64,
    `UTMMediumHash` UInt64,
    `UTMCampaignHash` UInt64,
    `UTMContentHash` UInt64,
    `UTMTermHash` UInt64,
    `FromHash` UInt64,
    `WebVisorEnabled` UInt8,
    `WebVisorActivity` UInt32,
    `ParsedParams` Nested(
        Key1 String,
        Key2 String,
        Key3 String,
        Key4 String,
        Key5 String,
        ValueDouble Float64),
    `Market` Nested(
        Type UInt8,
        GoalID UInt32,
        OrderID String,
        OrderPrice Int64,
        PP UInt32,
        DirectPlaceID UInt32,
        DirectOrderID UInt32,
        DirectBannerID UInt32,
        GoodID String,
        GoodName String,
        GoodQuantity Int32,
        GoodPrice Int64),
    `IslandID` FixedString(16)
)
ENGINE = CollapsingMergeTree(Sign)
PARTITION BY toYYYYMM(StartDate)
ORDER BY (CounterID, StartDate, intHash32(UserID), VisitID)
SAMPLE BY intHash32(UserID)
SETTINGS index_granularity = 8192
```

![](https://ask.qcloudimg.com/http-save/yehe-1147621/ai2mjekkij.png)

#### 3.3 导入数据

（1）加压数据

```
[root@elastic1 data]# unxz hits_v1.tsv.xz 
[root@elastic1 data]# unxz visits_v1.tsv.xz
[root@elastic1 data]# ll
total 15439336
-rw-r--r-- 1 root root 1271623680 Sep 14 13:59 hits_v1.tar
-rw-r--r-- 1 root root 7784351125 Sep 14 16:37 hits_v1.tsv
-rw-r--r-- 1 root root 3532492212 Sep 14 15:38 ontime.csv.xz
-rw-r--r-- 1 root root  563968000 Sep 14 13:44 visits_v1.tar
-rw-r--r-- 1 root root 2657415178 Sep 14 16:27 visits_v1.tsv
[root@elastic1 data]#
```

（2）导入数据

```
[root@elastic1 data]# cat hits_v1.tsv | clickhouse-client --password --query "INSERT INTO tutorial.hits_v1 FORMAT TSV" --max_insert_block_size=100000
Password for user (default): 
[root@elastic1 data]# cat visits_v1.tsv | clickhouse-client --password --query "INSERT INTO tutorial.visits_v1 FORMAT TSV" --max_insert_block_size=100000
Password for user (default): 
[root@elastic1 data]#
```

#### 3.4 简单查询

```
[root@elastic1 data]# clickhouse-client --password --query "OPTIMIZE TABLE tutorial.visits_v1 FINAL"
Password for user (default): 
[root@elastic1 data]# clickhouse-client --password --query "OPTIMIZE TABLE tutorial.hits_v1 FINAL"
Password for user (default): 
[root@elastic1 data]#
```

```
[root@elastic1 data]# clickhouse-client --password --query "SELECT COUNT(*) FROM tutorial.hits_v1"
Password for user (default): 
[root@elastic1 data]# clickhouse-client --password --query "SELECT COUNT(*) FROM tutorial.visits_v1"
Password for user (default): 
[root@elastic1 data]#
```

### 4、查询样例

```
SELECT
    StartURL AS URL,
    AVG(Duration) AS AvgDuration
FROM tutorial.visits_v1
WHERE StartDate BETWEEN '2014-03-23' AND '2014-03-30'
GROUP BY URL
ORDER BY AvgDuration DESC
LIMIT 10
```

![](https://ask.qcloudimg.com/http-save/yehe-1147621/hgdjhma3mb.png)

```
SELECT
    sum(Sign) AS visits,
    sumIf(Sign, has(Goals.ID, 1105530)) AS goal_visits,
    (100. * goal_visits) / visits AS goal_percent
FROM tutorial.visits_v1
WHERE (CounterID = 912887) AND (toYYYYMM(StartDate) = 201403) AND (domain(StartURL) = 'yandex.ru')
```

![](https://ask.qcloudimg.com/http-save/yehe-1147621/uhna0ts8u2.png)