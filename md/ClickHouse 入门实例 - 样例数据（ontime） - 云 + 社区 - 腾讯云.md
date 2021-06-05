> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [cloud.tencent.com](https://cloud.tencent.com/developer/article/1699382)

> 参考官方文档 https://clickhouse.tech/docs/en/getting-started/example-datasets/ontime/

参考官方文档 [https://clickhouse.tech/docs/en/getting-started/example-datasets/ontime/](https://clickhouse.tech/docs/en/getting-started/example-datasets/ontime/)

### 1、下载数据

https://yadi.sk/d/pOZxpa42sDdgm

![](https://ask.qcloudimg.com/http-save/yehe-1147621/hulel7lztn.png?imageView2/2/w/1620)

### 2、加压缩

```
[root@elastic1 data]# ll
total 15439336
-rw-r--r-- 1 root root 3532492212 Sep 14 15:38 ontime.csv.xz
...
[root@elastic1 data]# xz -dk ontime.csv.xz 
[root@elastic1 data]#
```

```
[root@elastic1 data]# ls -lh /tpdata/data/
total 75G
-rw-r--r-- 1 root root  62G Sep 14 15:38 ontime.csv
-rw-r--r-- 1 root root 3.3G Sep 14 15:38 ontime.csv.xz
....

[root@elastic1 data]#
```

### 3、新建表

```
CREATE TABLE `ontime` (
  `Year` UInt16,
  `Quarter` UInt8,
  `Month` UInt8,
  `DayofMonth` UInt8,
  `DayOfWeek` UInt8,
  `FlightDate` Date,
  `UniqueCarrier` FixedString(7),
  `AirlineID` Int32,
  `Carrier` FixedString(2),
  `TailNum` String,
  `FlightNum` String,
  `OriginAirportID` Int32,
  `OriginAirportSeqID` Int32,
  `OriginCityMarketID` Int32,
  `Origin` FixedString(5),
  `OriginCityName` String,
  `OriginState` FixedString(2),
  `OriginStateFips` String,
  `OriginStateName` String,
  `OriginWac` Int32,
  `DestAirportID` Int32,
  `DestAirportSeqID` Int32,
  `DestCityMarketID` Int32,
  `Dest` FixedString(5),
  `DestCityName` String,
  `DestState` FixedString(2),
  `DestStateFips` String,
  `DestStateName` String,
  `DestWac` Int32,
  `CRSDepTime` Int32,
  `DepTime` Int32,
  `DepDelay` Int32,
  `DepDelayMinutes` Int32,
  `DepDel15` Int32,
  `DepartureDelayGroups` String,
  `DepTimeBlk` String,
  `TaxiOut` Int32,
  `WheelsOff` Int32,
  `WheelsOn` Int32,
  `TaxiIn` Int32,
  `CRSArrTime` Int32,
  `ArrTime` Int32,
  `ArrDelay` Int32,
  `ArrDelayMinutes` Int32,
  `ArrDel15` Int32,
  `ArrivalDelayGroups` Int32,
  `ArrTimeBlk` String,
  `Cancelled` UInt8,
  `CancellationCode` FixedString(1),
  `Diverted` UInt8,
  `CRSElapsedTime` Int32,
  `ActualElapsedTime` Int32,
  `AirTime` Int32,
  `Flights` Int32,
  `Distance` Int32,
  `DistanceGroup` UInt8,
  `CarrierDelay` Int32,
  `WeatherDelay` Int32,
  `NASDelay` Int32,
  `SecurityDelay` Int32,
  `LateAircraftDelay` Int32,
  `FirstDepTime` String,
  `TotalAddGTime` String,
  `LongestAddGTime` String,
  `DivAirportLandings` String,
  `DivReachedDest` String,
  `DivActualElapsedTime` String,
  `DivArrDelay` String,
  `DivDistance` String,
  `Div1Airport` String,
  `Div1AirportID` Int32,
  `Div1AirportSeqID` Int32,
  `Div1WheelsOn` String,
  `Div1TotalGTime` String,
  `Div1LongestGTime` String,
  `Div1WheelsOff` String,
  `Div1TailNum` String,
  `Div2Airport` String,
  `Div2AirportID` Int32,
  `Div2AirportSeqID` Int32,
  `Div2WheelsOn` String,
  `Div2TotalGTime` String,
  `Div2LongestGTime` String,
  `Div2WheelsOff` String,
  `Div2TailNum` String,
  `Div3Airport` String,
  `Div3AirportID` Int32,
  `Div3AirportSeqID` Int32,
  `Div3WheelsOn` String,
  `Div3TotalGTime` String,
  `Div3LongestGTime` String,
  `Div3WheelsOff` String,
  `Div3TailNum` String,
  `Div4Airport` String,
  `Div4AirportID` Int32,
  `Div4AirportSeqID` Int32,
  `Div4WheelsOn` String,
  `Div4TotalGTime` String,
  `Div4LongestGTime` String,
  `Div4WheelsOff` String,
  `Div4TailNum` String,
  `Div5Airport` String,
  `Div5AirportID` Int32,
  `Div5AirportSeqID` Int32,
  `Div5WheelsOn` String,
  `Div5TotalGTime` String,
  `Div5LongestGTime` String,
  `Div5WheelsOff` String,
  `Div5TailNum` String
) ENGINE = MergeTree
PARTITION BY Year
ORDER BY (Carrier, FlightDate)
SETTINGS index_granularity = 8192;
```

![](https://ask.qcloudimg.com/http-save/yehe-1147621/gykpocjffn.png)

### 4、导入数据

```
[root@client tpdata]# cat ontime.csv | clickhouse-client --password --query "INSERT INTO ontime FORMAT CSV" --max_insert_block_size=100000
Password for user (default): 
[root@client tpdata]#
```

报错超时

```
[root@client tpdata]# clickhouse-client --password --query "OPTIMIZE TABLE ontime FINAL"
Password for user (default): 
Timeout exceeded while receiving data from server. Waited for 300 seconds, timeout is 300 seconds.
[root@client tpdata]#
```

### 5、查询实例

（1）查询总记录数

```
[root@client tpdata]# clickhouse-client --password --query "select count(*) from ontime"
Password for user (default): 
[root@client tpdata]#
```

（2）没有平均班次

```
SELECT avg(c1)
FROM
(
    SELECT Year, Month, count(*) AS c1
    FROM ontime
    GROUP BY Year, Month
);
```

![](https://ask.qcloudimg.com/http-save/yehe-1147621/xqtlfjwmug.png)

（3）查询从 2000 年到 2008 年每天的航班数

```
SELECT DayOfWeek, count(*) AS c
FROM ontime
WHERE Year>=2000 AND Year<=2008
GROUP BY DayOfWeek
ORDER BY c DESC;
```

![](https://ask.qcloudimg.com/http-save/yehe-1147621/7cfx2rcvsu.png)

（4）查询从 2000 年到 2008 年每周延误超过 10 分钟的航班数。

```
SELECT DayOfWeek, count(*) AS c
FROM ontime
WHERE DepDelay>10 AND Year>=2000 AND Year<=2008
GROUP BY DayOfWeek
ORDER BY c DESC;
```

![](https://ask.qcloudimg.com/http-save/yehe-1147621/uer0yq4uxb.png)

（5）查询 2000 年到 2008 年每个机场延误超过 10 分钟以上的次数

```
SELECT Origin, count(*) AS c
FROM ontime
WHERE DepDelay>10 AND Year>=2000 AND Year<=2008
GROUP BY Origin
ORDER BY c DESC
LIMIT 10;
```

![](https://ask.qcloudimg.com/http-save/yehe-1147621/88rd3ptj73.png)

（6）查询 2007 年各航空公司延误超过 10 分钟以上的次数

```
SELECT Carrier, count(*)
FROM ontime
WHERE DepDelay>10 AND Year=2007
GROUP BY Carrier
ORDER BY count(*) DESC;
```

![](https://ask.qcloudimg.com/http-save/yehe-1147621/zy341xjbum.png)

（7）查询 2007 年各航空公司延误超过 10 分钟以上的百分比

```
SELECT Carrier, avg(DepDelay>10)*100 AS c3
FROM ontime
WHERE Year=2007
GROUP BY Carrier
ORDER BY c3 DESC
```

![](https://ask.qcloudimg.com/http-save/yehe-1147621/hj77cjas4x.png)

（8）同上一个查询一致, 只是查询范围扩大到 2000 年到 2008 年

```
SELECT Carrier, avg(DepDelay>10)*100 AS c3
FROM ontime
WHERE Year>=2000 AND Year<=2008
GROUP BY Carrier
ORDER BY c3 DESC;
```

![](https://ask.qcloudimg.com/http-save/yehe-1147621/xywmgz9lcf.png)

（9）每年航班延误超过 10 分钟的百分比

```
SELECT Year, avg(DepDelay>10)*100
FROM ontime
GROUP BY Year
ORDER BY Year;
```

![](https://ask.qcloudimg.com/http-save/yehe-1147621/80qlsft52k.png)

（10）每年更受人们喜爱的目的地

```
SELECT DestCityName, uniqExact(OriginCityName) AS u
FROM ontime
WHERE Year >= 2000 and Year <= 2010
GROUP BY DestCityName
ORDER BY u DESC LIMIT 10;
```

![](https://ask.qcloudimg.com/http-save/yehe-1147621/2xpxbafgmz.png)

（11）按年份统计

```
SELECT Year, count(*) AS c1
FROM ontime
GROUP BY Year;
```

![](https://ask.qcloudimg.com/http-save/yehe-1147621/bt2til0sp7.png)

（12）查询 2010 年最受欢迎的目的地

```
SELECT 
    OriginCityName, 
    DestCityName, 
    count(*) AS flights, 
    bar(flights, 0, 20000, 40)
FROM ontime
WHERE Year = 2010
GROUP BY 
    OriginCityName, 
    DestCityName
ORDER BY flights DESC
LIMIT 20
```

![](https://ask.qcloudimg.com/http-save/yehe-1147621/zz3v352x28.png)

（13）最长飞行时间

```
SELECT 
    OriginCityName, 
    DestCityName, 
    count(*) AS flights, 
    avg(AirTime) AS duration
FROM ontime
GROUP BY 
    OriginCityName, 
    DestCityName
ORDER BY duration DESC
```

![](https://ask.qcloudimg.com/http-save/yehe-1147621/gya9raik9u.png)