> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/KHCUa_xjv9K-sQyAOlPjzQ)

* * *

一、数据处理
------

数据表的变量含义如下。

首先读取数据，由于原数据没有 header，直接就是数据，因此需设置为 None，然后手动添加列索引名称。

```
# 读取数据
hd = ['id','time','long','lati','status','speed'] 
df = pd.read_csv('taxi.csv', header=None, names=hd)   

```

_数据来源：《交通时空大数据分析、挖掘与可视化》_

看下数据整体情况。

```
df.info()


```

共 54.5 万条数据，没有缺失的变量，且类型除时间 time 以外都是数值型。

接着看下具体数据，猜测和理解下业务场景，并了解数据的形式。

```
# 查看前20行    
df.head(20)  


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/NOM5HN2icXzyDiacN7g9cC8jY0OF3MwKOvFXF7cUhOAGO5zdPw1LRMAlJFMhOgT1cNYNTmBopkYODtORsX6VpC2g/640?wx_fmt=png)

通过以上数据，我们可以发现：

*   **获取方式**：这个数据是通过出租车上的机器（比如传感器）采集的 GPS 信息，并且每隔一段时间进行采集和上报，**采集频率不固定**。
    
*   **时间数据**：每个采集时间都提供了经纬度、载客状态、和车速信息，是一组时间序列数据，但仔细发现**原数据时间没有排序。**
    
*   **车辆行驶**：经纬度、车速随着车的行驶状态动态变化，比如车停了等红灯车速变为 0，经纬度也保持不变，车开起来了车速和经纬度会实时变化。
    
*   **载客状态**：**根据是否有乘客上车而发生变化，与车辆行驶状态没有任何关系。** 出租车的初始状态是 0 的话，如果有乘客上车，那么载客状态变为 1，并且在乘客未下车之前机器采集上报的状态会一直是 1，直到乘客下车为止才会再变为 0，然后循环反复。
    

以上是我们对数据的简单理解。

二、数据处理
------

##### 1）排序

**需求 1：对 id 和 time 进行升序排序，然后重置索引**

```
# 排序
df = df.sort_values(by = ['id','time']).reset_index(drop=True)
df.head(10)


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/NOM5HN2icXzyDiacN7g9cC8jY0OF3MwKOvgrLbjAkMlVJK0lXYlJhhC4BQra2md2xJibZOpoKMPTmVBf8krvKNrKA/640?wx_fmt=png)

可以看到 time 已经按照升序排序了，索引重置为 0,1,..,n。

##### 2）类型转换

前面我们发现 time 变量是 object 类型，不利于我们做日期的操作，因此我们要转换为时间戳类型。

```
# 转换日期类型
df['time'] = pd.to_datetime(df['time'])


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/NOM5HN2icXzyDiacN7g9cC8jY0OF3MwKOvaBadcNjfiaCjvHYSCSCdDGlwHRcmXn9WOPsA7UiaZMWQ1Ts9HuRRiaBAQ/640?wx_fmt=png)

##### 3）重复值

理论上说，id 和 time 都一样就是重复的数据，因为时间是按一定频率采样的，一个车辆在一个时间点只对应一条数据。因此设置`subset`子集对 id 和 time 查重，同时设置`keep=False`保留全部重复数据。查重的具体用法可参考。

```
df_dup = df[df.duplicated(subset=['id','time'], keep=False)==True].reset_index()
df_dup.shape
------
(910, 6)


```

重复数据全部保留共有 910 条（这里使用`reset_index`将原数据 df 的索引变为变量，后面去重时有用）。

仔细观察发现，重复数据在 id 和 time 相同的情况下，其他变量还存在多种不同形式（如下图红框），形式总结如下。

*   status 相同都是 0 或都是 1，但经纬度、车速可能不同
    
*   status 不同，是 1 和 0，但经纬度、车速相同
    

**那具体该保留哪个，去除哪个呢？**

这需要我们找到一个保留或去除的判断依据。

这里我们尝试通过 status 的前后变化对重复数据进行判断和筛选。一是因为同一时间不可能有两个载客状态，二是 status 变化频率低利于观察。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/NOM5HN2icXzyDiacN7g9cC8jY0OF3MwKOv899XAhCmR16sCHP8mT0rlTtyoXWxdTvIsU2yUoUI5f8EZDeEaicRvkQ/640?wx_fmt=png)

**发现了几种不同的形式，我们如何处理呢？**

根据 status 前后变化的规律，处理方式如下：

*   status 相同时，但经纬度和车速不同时，删除其一即可，因为采样频率过低无法具体判断哪个是准确的。
    
*   status 不同时，但经纬度和车速相同时，删除时间序列下 status 异常数据，因为乘客坐车需要时间，载客状态不可能在极短的时间内突然变化。比如，时序下的 status 为 0001000，中间突然出现一个 1，那么删除 1 所在行。同理 1110111 突然出现一个 0，那么删除 0 所在行（这部分也算是异常值，只不过与重复值交叉同时出现了）。
    

**需求 4：对重复数据进行分组的重复数量统计，检查是否有 3 个以上（包含）重复的**

以上重复数据的数量都是 2 个，那有没有大于 2 个重复的呢？

数据量太多，肉眼无法观察，我们通过以下语句判断。

```
(df_dup.groupby(['id','time'])['status'].count()==2).all()
------
False


```

对 id 和 time 分组统计重复数量，是否全部等于 2，结果并不是，说明存在大于 2 个重复的数据。

**需求 5：筛选出重复数量大于 2 的数据**

```
(
    df_dup.groupby(['id','time'])['status'].count()
          .reset_index()
          .pipe(lambda x:x.loc[x.status>2])
)


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/NOM5HN2icXzyDiacN7g9cC8jY0OF3MwKOvT6uoBtuicELG1WGFiaib2y94rmqW1Dn6DjFNqdicicJJHS7yFdRib62XmNdA/640?wx_fmt=png)

id 和 time 分组下有 6 个重复数量为 3 的，且最大重复数量就是 3 了。

对于该情况下 status 前后的变化情况就需要逐个筛选去看了，这里数量不多大家可以自行观察数据。

经过观察后，我们可以这样做去重的处理：

*   如果 status 全部相同，那么任意选一个，比如选第一个
    
*   如果 status 不同，那么基于少数服从多数原则，从多个值里选择一个。比如时序的 status 值分别为 101/110/011，从两个 1 中选其一；再比如 status 为 001/100/010，从两个 0 中选其一。
    

至此，查重部分结束。我们发现了一些规律并且制定了去重的逻辑，**那么如何实现去重呢？**

**需求 6：对 id 和 time 分组统计 status 个数、求和，与重复数据 df_dup 匹配合并**

很显然，在这种复杂的情况下直接用`drop_duplicates`是不管用的，所以我们必须想其他的方法。

下面我们通过加工一组特征来辅助我们进行去重的筛选，对 id 和 time 分组统计 status 个数、求和。

```
dup_grp = (
    df_dup.groupby(['id','time'])
          .agg(stat_cnt=('status','count'),stat_sum=('status','sum'))
          .reset_index())


```

加工结果如下，每个唯一的 id 和 time 组合都对应着 stat_cnt 和 stat_sum 两个特征，**根据两个特征值的不同组合就可以判断重复的不同情况了（如图）。**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/NOM5HN2icXzyDiacN7g9cC8jY0OF3MwKOvhrTXnTZsNT1bpXnLuXsDmPVbnt0aWYzBx3iaT7DNupiaI46tDqBM9ndA/640?wx_fmt=png)

然后我们再通过`merge`用法将特征值匹配到重复数据 df_dup 上。

```
dup_mrg = pd.merge(df_dup, dup_grp, on=['id','time'], how='left')
dup_mrg.head(6)


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/NOM5HN2icXzyDiacN7g9cC8jY0OF3MwKOvYmv5UibM03rPKxIiaibqyINyQwQpmg9iaZ8jETQSO0nOJUNByib4QOHKTYw/640?wx_fmt=png)

**需求 7：根据以上需求 3 和 5 中查重的判断逻辑对重复数据筛选，返回需要保留的行索引**

这里是去重的最后一步了，前面的处理都是为了这一步的逻辑判断。

可以想到用`groupby+apply`的方法组合对重复数据分组聚合来进行筛选，结果返回需要保留数据的原数据索引（在需求 3 中已经重置索引）。

```
def dup_check(x):
    if (x.stat_cnt.max() == 2) & (x.stat_sum.max() == 0): #重复数量为2,status均为0,返回第一个索引
        return x['index'].values[0]
    elif (x.stat_cnt.max() == 2) & (x.stat_sum.max() == 1):  #重复数量为2,status为10,返回0值的索引
        return x.loc[x.status==0,'index'].values[0]
    elif (x.stat_cnt.max() == 2) & (x.stat_sum.max() == 2): #重复数量为2,status均为1,返回第一个索引
        return x['index'].values[0]
    elif (x.stat_cnt.max() == 3) & (x.stat_sum.max() == 0): #重复数量为3,status均为0,返回第一个索引
        return x['index'].values[0]
    elif (x.stat_cnt.max() == 3) & (x.stat_sum.max() == 1): #重复数量为3,status为100/010/001,返回第一个0值的索引
        return x.loc[x.status==0,'index'].values[0]
    elif (x.stat_cnt.max() == 3) & (x.stat_sum.max() == 2): #重复数量为3,status为110/101/011,返回第一个1值的索引
        return x.loc[x.status==1,'index'].values[0]
    elif (x.stat_cnt.max() == 3) & (x.stat_sum.max() == 3): #重复数量为3,status均为1,返回第一个索引
        return x['index'].values[0]

# 重复数据中需保留的行索引
kp_index = dup_mrg.groupby(['id','time']).apply(dup_check)
# 重复数据中需去掉的行索引
drp_index = dup_mrg.loc[~dup_mrg['index'].isin(kp_index.values),'index']
drp_index


```

得到保留数据的索引，那么剩下的全都是需要去重的索引了。

最后我们再通过`loc`筛选从原始数据 df 中筛选掉这些需要去除的行索引，最终达到去重的目的。

```
print(df.shape)
df = df.loc[~df.index.isin(drp_index.values)]
print(df.shape)
------
(544999, 6)
(544541, 6)


```

可以看到共去重了 458 条数据。

##### 4）异常值

其实前面重复值处理时已经遇到了异常值，但那是在重复情况下发生的异常，一定也还有非重复情况下的异常。

> 说明：由于是机器采集的 GPS 数据，在采集过程中可能会因传感器问题出现一定概率的异常值，这是经常发生的，所以我们必须对数据进行异常的排查。

前面提到过，我们发现存在以下异常的情况：**同一个车辆在极短的时间内完成载客状态的切换。**

比如下面红框里，最大时间差为 40 秒，载客状态由**空客 -> 载客 -> 空客。**

理论上说基本不可能，因为乘客乘坐出租车只有 40 秒的情况极其少见，因此我们判定中间 status=1 的出现是一种异常情况，需要剔除或将其状态还原为 0。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/NOM5HN2icXzyDiacN7g9cC8jY0OF3MwKOv5wjALXzpbRNXyz57mS9EW0RE6pTlBQYkHJ7bJdUxrYOIKfMMr8JbrQ/640?wx_fmt=png)

上面是 0-1-0 的异常，同理 1-0-1 也是异常，都是短时间内的状态切换。

既然我们发现了这种异常，**如何使用 pandas 将此类异常全部筛选出来呢？**

我们给出的判断逻辑是：

*   载客状态不连续，当前状态与前后状态不一样，比如 0-1-0 或 1-0-1
    
*   且这段不连续状态属于同一个车辆 id
    
*   且这段不连续状态的最大时间差很小，我们设定 60 秒为阈值
    

**需求 8：将 id、time、status 变量分别上移和下移 1 个单位，生成 6 个新变量**

现在问题的关键如何用当前状态与前后状态进行对比，pandas 中可以使用`shift`函数对列进行上下的移动，这样就可以实现前后对比了。

```
# 上下平移一个单位
df['status_up'] = df['status'].shift(1) # 向下移动 1
df['status_down'] = df['status'].shift(-1) # 向上移动 1

df['id_up'] = df['id'].shift(1) # 向下移动 1 
df['id_down'] = df['id'].shift(-1) # 向上移动 1

df['time_up'] = df['time'].shift(1) # 向下移动 1 
df['time_down'] = df['time'].shift(-1) # 向上移动 1


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/NOM5HN2icXzyDiacN7g9cC8jY0OF3MwKOvA9aT4cGZJOtaiaQ5X6l7nWXwUEBmcic1CofwsVPE8U8XmplxH8iaiczoiaQ/640?wx_fmt=png)

以这样就可以对每一行进行前后值是否相等的判断了。

比如上面图里红框，对`status=1`与`status_up=0`和`status_down=0`对比以后，筛选出了 0-1-0 这种异常模式。

**需求 9：以上存在异常状态的数据全部筛选出来**

筛选逻辑如前面所说，以下是对应的 5 个筛选条件。

```
#剔除异常数据
cond_1 = (df['status'] != df['status_down'])
cond_2 = (df['status'] != df['status_up'])
cond_3 = (df['id'] == df['id_up'])
cond_4 = (df['id'] == df['id_down'])
cond_5 = ((df['time_down']-df['time_up']).dt.seconds < 60)

df_abn = df[cond_1 & cond_2 & cond_3 & cond_4 & cond_5].reset_index()
df_abn.shape
------
(409, 13)


```

然后我们就可以得到满足这些条件的异常数据，共有 409 条（前面去重时还删了一部分）。

```
# 异常数据id分布
df_abn.id.value_counts()


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/NOM5HN2icXzyDiacN7g9cC8jY0OF3MwKOvUCOvO6YVTc52HgSdQHZ1o17mHmgPd9XxxFMRMjlGvQCLP8wBWIbkXA/640?wx_fmt=png)

这 409 个非重复数据异常值里面，有的车辆（比如 id 为 28159）高达 70 次的异常。

通过这样的异常检查，我们就可以对车辆数据进行监控，比如每隔一段时间内车辆所发生的异常次数，进而做相应的处置。

**需求 10：对非重复异常值进行剔除**

与重复值去除一样，这里我们通过记录原数据索引的方式，将异常值索引所在行数据从原数据中剔除。

```
print(df.shape)
df = df.loc[~df.index.isin(df_abn['index'].values)]
print(df.shape)
------
(544541, 12)
(544132, 12)


```

三、数据分析及可视化
----------

##### 1）订单出行特征

原数据是一个 GPS 信息表，每一行代表了 xx 车辆在 xx 订单下 xx 时间的 GPS 数据，是一个明细数据。这非常不利于业务人员使用，业务更多关心的是车辆在什么时间什么地点最终到了哪里去，而不是每时每刻的信息。

**需求 11：我们需要把 GPS 信息表转换为出行信息表**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/NOM5HN2icXzyDiacN7g9cC8jY0OF3MwKOv2gFaSUibnJZ0u8Vfsd8wPjEVUMTxicicrMh5SGxiaSGeGFRLllKvymnyicg/640?wx_fmt=png)

转换后的形式如上图所示，地点可用经纬度代替。

**那么这个转换过程如何实现呢？**

可以通过下面两个步骤实现。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/NOM5HN2icXzyDiacN7g9cC8jY0OF3MwKOvsZbYj4gqvpBb4hZb4LyVQTAzU6X83OiajXKPmECaVOGe02u2l16eniaw/640?wx_fmt=png)

*   捕捉每个订单上下车的时间和地点，并筛选出来
    

判断条件是：如果此时点的 status 载客状态与上一状态差为 1，即由 0 变为 1，说明是上车。反之，如果由 1 变为 0 则差值为 - 1，即为下车。

那么用此时点与上一时点状态作差还是可以通过`shift`偏移来实现，前面检查异常值时我们已经创建了辅助特征 status_up 和 id_up，所以这里直接拿来用即可。

将状态差值为 1（上车）和 -1（下车）筛选出来，并且两个状态下需为同一辆车。

```
# 与上一个状态作差
df['status_chg'] = df['status']-df['status_up']
# 车辆id与上一个作差
df['id_chg'] = df['id']-df['id_up']
# 筛选上车和下车的标识1和-1，并为同一车辆
df_temp = df.loc[((df['status_chg']==1) | (df['status_chg']==-1)) & (df['id_chg']==0)]


```

*   将上、下车时间的地点时间错位拼接
    

现在上下车分别为两行数据，我们最终想移动变成一行。还是利用 shift 将我们想要的变量向上偏移一个单位即可。偏移后每一行都是上车、下车或下车、上车的信息，我们最后再通过 loc 筛选从上车到下车的所有行，同样指定是同一车辆。

最后修改列名完成了出行信息表。

```
# 将时间、经纬度向上偏移1个单位
df_temp['Etime'] = df_temp['time'].shift(-1)
df_temp['Elong'] = df_temp['long'].shift(-1)
df_temp['Elati'] = df_temp['lati'].shift(-1)
# 
df_order = df_temp.loc[(df_temp['status_chg']==1) & (df_temp['id']==df_temp['id'].shift(-1)),
                       ['id','time','long','lati','Etime','Elong','Elati']]
# 修改列名
df_order.columns=['车辆id','开始时间','开始经度','开始纬度','结束时间','结束经度','结束纬度']
df_order.head(10)


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/NOM5HN2icXzyDiacN7g9cC8jY0OF3MwKOvdbWAneTPPtNaIyBiaDY4ic1v0HGFoALHnb3hxG0jEhNtAS6w9JeBeRiaA/640?wx_fmt=png)

下面根据已完成的**订单出行信息表**进行一些统计分析。

##### 2）订单时段数量统计

**需求 12：统计各小时的订单数分布**

前面我们已经将 time 时间转换为时间类型了，那么将时间戳转换为小时就非常简单了，时间属性方法可以参考传送门。转换后为一天 0 到 24 小时之内的小时数值，比如 2023-06-28 04:30:13 转换为小时 4。

然后对小时 groupby 分组求订单数量即可，最后使用 pandas 的内置方法进行可视化，可视化方法参考传送门。

```
# 转换为小时
df_order['小时'] = df_order['开始时间'].dt.hour
# 对小时分组求订单数量
df_hourcnt = df_order.groupby('小时')['车辆id'].count()      
df_hourcnt = df_hourcnt.rename('数量').reset_index()   

# 绘制折线图    
fig = plt.figure(1,(8,4),dpi = 200)    
ax = plt.subplot(111)    
plt.plot(df_hourcnt['小时'],df_hourcnt['数量'],'k-')    
plt.plot(df_hourcnt['小时'],df_hourcnt['数量'],'k.')    
plt.bar(df_hourcnt['小时'],df_hourcnt['数量'])    
plt.ylabel('数量')  
plt.xlabel('小时')  
plt.xticks(range(24),range(24))  
plt.title('出行小时数量统计')    
plt.ylim(0,350)    
plt.show()


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/NOM5HN2icXzyDiacN7g9cC8jY0OF3MwKOvSCacib52SgC8hibApzWcsEyrdM6wk0NwSlibN2VNVjuWfzEpsKURE2lNg/640?wx_fmt=png)

观察结果：

*   出租车在各时段都有订单出行，其中早上 8 点到晚上 5 点起伏较为平稳，从晚上 8 点到 10 点是高峰期，后半夜 3 点到 5 点（夜车）是低峰期。观察结果比较符合我们的现实情况。
    

##### 3）订单时长分布

**需求 13：统计各时段订单的时长分布**

下面通过箱型图对各时段订单时长做可视化。

```
# 开始结束时间作差并转化为秒单位
df_order['订单时长'] = (df_order['结束时间']-df_order['开始时间']).dt.seconds
# 各时段订单时长箱型图 
fig = plt.figure(1,(6,4),dpi = 150)      
ax = plt.subplot(111)  
plt.sca(ax)  
sns.boxplot(x="小时", y=df_order["订单时长"]/60, data=df_order,ax=ax)  
plt.ylabel('订单时长(分钟)')  
plt.xlabel('订单时段')  
plt.ylim(0,60)  
plt.show()  


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/NOM5HN2icXzyDiacN7g9cC8jY0OF3MwKOv1Mwhklp7nDpgefpJndmxmpH6TMOn1G0rPVvMC3gUGqPJ3ylQOurPMQ/640?wx_fmt=png)

观察结果：

*   订单时长在各时段均有一定的差异，但在早晚高峰时段（8-9 点、17-18 点）订单时长明显多，是因为高峰期堵车所以导致时间比较长。