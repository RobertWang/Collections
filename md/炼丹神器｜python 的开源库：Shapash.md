> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIwOTc2MTUyMg==&mid=2247523033&idx=3&sn=d7352808d924de17f4867d555ac50702&chksm=976c3f44a01bb6525c2c68b6c367d282cbf56f64fd5e6325f4d3d2e04d01f1b0ac98d095e0c9&mpshare=1&scene=1&srcid=0727UH0gJb4LXz0wuNtgpzg2&sharer_sharetime=1627374580093&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

最近时晴又发现了个炼丹神器 Shapash, 就迫不及待的要推荐给大家. 这是个 python 的开源库, 可以让炼丹师们在炼丹过程中理解自己为什么能练出 "好" 丹. 相信诸位炼丹师和我一样, 不仅追求一个好的模型, 同时也追究模型的可解释性, 废话不多说, 我们看看 "太阳女神" 如何解释我们的模型吧.

shapash 适用于很多模型: Catboost,Xgboost,LightGBM,Sklearn Ensemble 等. 可以简单的用 pip 进行安装:

```
$pip install shapash
```

我们用一个实际的例子来说明 shapash 的用法. 我们先训练一个回归模型, 用于预测房价.

数据下载链接:https://www.kaggle.com/c/house-prices-advanced-regression-techniques

先用 shapash 读入数据:

```
import pandas as pd
from shapash.data.data_loader import data_loading
# house_dict里面是特征名到特征含义的映射

house_df, house_dict = data_loading('house_prices')
y_df=house_df['SalePrice'].to_frame()
X_df=house_df[house_df.columns.difference(['SalePrice'])]
```

看下数据如下:  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/gYUsOT36vfrYJvGC695H19qNAKDTVhGfSr1eycqAyJwvrxhH8gT9yJHeeibbssDdIcF4lxXpdPA0zhZ1dotZuzQ/640?wx_fmt=jpeg)

对类别特征进行编码:

```
from category_encoders import OrdinalEncoder

categorical_features = [col for col in X_df.columns if X_df[col].dtype == 'object']
encoder = OrdinalEncoder(cols=categorical_features).fit(X_df)
X_df=encoder.transform(X_df)
```

我们可以看到, 所有特征都变成数值了:

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/gYUsOT36vfrYJvGC695H19qNAKDTVhGfiaZxYriaZLZ9VrgIQIK4kP2TA26alZM9y51iaClcc0QkgzEBIOE0sIE5g/640?wx_fmt=jpeg)

找个任意的回归模型训练, 这里我用随机森林:

```
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestRegressor
Xtrain, Xtest, ytrain, ytest = train_test_split(X_df, y_df, train_size=0.75)
reg = RandomForestRegressor(n_estimators=200, min_samples_leaf=2).fit(Xtrain,ytrain)
#预估测试集
y_pred = pd.DataFrame(reg.predict(Xtest), columns=['pred'], index=Xtest.index)
```

这里我们不探讨该模型效果, 直接看看如何用 "太阳女神" 解释该模型:

```
from shapash.explainer.smart_explainer import SmartExplainer
xpl = SmartExplainer(features_dict=house_dict) # Optional parameter 
xpl.compile(
    x=Xtest,
    model=reg,
    preprocessing=encoder,# Optional: use inverse_transform method
    y_pred=y_pred # Optional
)
```

然后使用一行代码, 就可以解释模型了:

```
app = xpl.run_app()
```

我们可以看到特征重要性:

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/gYUsOT36vfrYJvGC695H19qNAKDTVhGfia4YIpkP0u7HpdYbiaOSJF76lWPc4BGX6ATINWLdxvXsdE1j3N3hvDPQ/640?wx_fmt=jpeg)

已经特征多大程度影响预估:

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/gYUsOT36vfrYJvGC695H19qNAKDTVhGfVc9T0E3By7RrE6FAHhIE74y2J7vrkQgia9VSDyusiafk8fHiasBuXGdCQ/640?wx_fmt=jpeg)

当我们选择特征重要性最低的特征时, 可以发现该特征影响的样本较少, 影响值的范围也小了很多 (-2000~2000).

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/gYUsOT36vfrYJvGC695H19qNAKDTVhGfHKibvmATdyukmvgy7Aw8iazKe6ahHdD2M1NiaZUVkYKkD9MlU8quLeTsw/640?wx_fmt=jpeg)

此外还有一些可视化的特性等待大家探索:

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/gYUsOT36vfrYJvGC695H19qNAKDTVhGfN8LZAcWg60Lap3ibSeC2hEeBhOOclOLpnn1sSsaGzic0Afs43dGplQYQ/640?wx_fmt=jpeg)

* * *

**推荐阅读**

[干货 | 公众号历史文章精选](https://mp.weixin.qq.com/s?__biz=MzIwOTc2MTUyMg==&mid=2247489073&idx=2&sn=01856c4f799c8bb2c72dc3e04ed6fa03&chksm=976fb3aca0183aba785ab0ef547c646a33a55c3e4c1c3689a246f902b47264b4dbff3a68bcc8&token=98277343&lang=zh_CN&scene=21#wechat_redirect)

[我的深度学习入门路线](https://mp.weixin.qq.com/s?__biz=MzIwOTc2MTUyMg==&mid=2247485239&idx=1&sn=c3e1eef018f1a90f221cca1b74fc361d&chksm=976fa2aaa0182bbc383a48a33623631e289483c4a0df56cddac52d22127babb66bdec65a6522&token=919391882&lang=zh_CN&scene=21#wechat_redirect)

[我的机器学习入门路线图](https://mp.weixin.qq.com/s?__biz=MzIwOTc2MTUyMg==&mid=2247484874&idx=1&sn=e4a54e67d7da788c5b75acfa96763e1c&chksm=976fa057a0182941e8dda24a469338100d81de848a6aad467f77c6950202ab02562739aa1c8c&scene=21&token=98277343&lang=zh_CN#wechat_redirect)