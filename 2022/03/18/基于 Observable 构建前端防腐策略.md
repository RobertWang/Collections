> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIzOTU0NTQ0MA==&mid=2247507816&idx=1&sn=dcdde828985ecbee80c15ba64a1bc60e&chksm=e92ae267de5d6b713ecdde9cd4900260bcf5a7f2d40ef15e3e8898696cc3902fd5ad7393db64&mpshare=1&scene=1&srcid=0316pskxQwNwInHhoL6VOFWW&sharer_sharetime=1647401070548&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

‍To B 业务的生命周期与迭代通常会持续多年，随着产品的迭代与演进，以接口调用为核心的前后端关系会变得非常复杂。在多年迭代后，接口的任何一处修改都可能给产品带来难以预计的问题。在这种情况下，构建更稳健的前端应用，保证前端在长期迭代下的稳健与可拓展性就变得非常重要。本文将重点介绍如何利用接口防腐策略避免或减少接口变更对前端的影响。

  

**一  困境与难题**
------------

为了更清晰解释前端面临的难题，我们以 To B 业务中常见的仪表盘页面为例，该页面包含了可用内存、已使用内存和已使用的内存占比三部分信息展示。

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIhzRZBiaWuic8wicxaFX9M6WhCRjIgwxBOS7G3GiaLD1urEqjm0TPRrY6G8HN2quYNn4U5StKF9AKkoQ/640?wx_fmt=png)

  

此时前端组件与接口之间的依赖关系如下图所示。

  

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIhzRZBiaWuic8wicxaFX9M6Wh6FbDUtVuavWs5GAmHGXWDaK2af167FLKpJIIr7InYr1HgYNHbecjbQ/640?wx_fmt=png)

  

当接口返回结构调整时 MemoryFree 组件对接口的调用方式需要调整。同样的，MemoryUsage 与 MemoryUsagePercent 也要进行修改才能工作。

  

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIhzRZBiaWuic8wicxaFX9M6Why7s3USMQrpWCAGXeTiaicYwzIXGG6EE8icclDEG3v5m6vMJG4iaG0OaFMw/640?wx_fmt=png)

  

真实的 To B 业务面临的接口可能会有数百个，组件与接口的集成逻辑也远比以上的例子要复杂。

  

经过数年甚至更长时间的迭代后，接口会逐步产生多个版本，出于对界面稳定性及用户使用习惯的考量，前端往往会同时依赖接口的多个版本来构建界面。当部分接口需要调整下线或发生变更时，前端需要重新理解业务逻辑，并做出大量代码逻辑调整才能保证界面稳定运行。

  

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIhzRZBiaWuic8wicxaFX9M6Wh97aY0DPzbiaGaq28OHsVnoFplt1y5YlT2kibzyBc5Rp2EASGUNnYlZbA/640?wx_fmt=png)

  

常见的对前端造成影响的接口变更包括但不限于：

  

*   返回字段调整
    
*   调用方式改变
    

*   多版本共存使用
    

  

当前端面对的是平台型业务时，此类问题会变得更为棘手。平台型产品会对一种或多种底层引擎进行封装，例如机器学习平台可能会基于 TensorFlow、Pytorch 等机器学习引擎搭建，实时计算平台可能基于 Flink、Spark 等计算引擎搭建。

  

虽然平台会对引擎的大部分接口进行上层封装，但不可避免的仍然会有部分底层接口会直接被透传到前端，在这个时候，前端不仅要应对平台的接口变更，还会面临着开源引擎接口的变更带来的挑战。

  

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIhzRZBiaWuic8wicxaFX9M6WhraLUAWJSH29zZv6HFt9MtFmJR1YDuxac314Xtj4bOvOU1WQgbq4c0Q/640?wx_fmt=png)

  

前端在面临的困境是由独特的前后端关系决定的。与其他领域不同，在 To B 业务中，前端通常以下游客户的身份接受后端供应商的供给，有些情况下会成为后端的跟随者。

  

在客户 / 供应商关系中，前端处于下游，而后端团队处于上游，接口内容与上线时间通常由后端团队来决定。

  

在跟随者关系中，上游的后端团队不会去根据前端团队的需求进行任何调整，前端只能去顺应上游后端的模型。这种情况通常发生在前端无法对上游后端团队施加影响的时刻，例如前端需要基于开源项目的接口设计界面，或者是后端团队的模型已经非常成熟且难以修改时。

  

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIhzRZBiaWuic8wicxaFX9M6WhllpSCnEMcPG4orvibcFT0cicmok4txFqOibYQb2ErcshOnH0Eolrmplpw/640?wx_fmt=png)

  

《架构整洁之道》的作者描述过这样一个嵌入式架构设计的难题，与上文我们描述的困境十分类似。

  

软件应当是一种使用周期很长的东西，而固件会随着硬件的演进而淘汰过时，但事实上的情况是，虽然软件本身不会随着时间推移而磨损，但硬件及其固件却会随时间推移而过时，随即也需要对软件做相应的改动。

  

无论是客户 / 供应商关系，还是跟随者关系，正如软件无法决定硬件的发展与迭代一样，前端也很难或者无法决定引擎与接口的设计，虽然前端本身不会随着时间的推移而变得不可用，但技术引擎及相关接口却会随着时间推移而过时，前端代码会跟随技术引擎的迭代更换逐步腐烂，最终难逃被迫重写的命运。

  

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIhzRZBiaWuic8wicxaFX9M6Wh0TrhORnztu9iaRC0To0Et58ghEKkwZ0mPkADcibvpf2pHpBUHIGY1NnA/640?wx_fmt=png)

**二  防腐层设计**
------------

早在 Windows 诞生之前，工程师为了解决上文中硬件、固件与软件的可维护性问题，引入了 HAL（Hardware Abstraction Layer）的概念， HAL 为软件提供服务并且屏蔽了硬件的实现细节，使得软件不必由于硬件或者固件的变更而频繁修改。

  

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIhzRZBiaWuic8wicxaFX9M6WhcusDOKsnNc5c5kMicV21gfrpah0pW1ibAFia1QdaiauPb5TWjYpW3RQe8w/640?wx_fmt=png)

  

HAL 的设计思想在领域驱动设计（DDD） 中又被称为防腐层（Anticorruption Layer）。在 DDD 定义的多种上下文映射关系中，防腐层是最具有防御性的一种。它经常被使用在下游团队需要阻止外部技术偏好或者领域模型入侵的情况，可以帮助很好地隔离上游模型与下游模型。

  

我们可以在前端中引入防腐层的概念，降低或避免当前后端的上下文映射接口变更对前端代码造成的影响。

  

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIhzRZBiaWuic8wicxaFX9M6WhKXqkz1KXsfZ4B4IhiaDTlrKB5MRcQpOHsDo4I0v4zbTV9q72XnA0PoQ/640?wx_fmt=png)

  

在行业内有很多种方式可以实现防腐层，无论是近几年大火的 GraphQL 还是 BFF 都可以作为备选方案，但是技术选型同样受限于业务场景。与 To C 业务完全不同，在 To B 业务中，前后端的关系通常为客户 / 供应商或者跟随者 / 被跟随者的关系。在这种关系下，寄希望于后端配合前端对接口进行 GraphQL 改造已经变得不太现实，而 BFF 的构建一般需要额外的部署资源及运维成本。

  

在上述情况下，在浏览器端构建防腐层是更为可行的方案，但是在浏览器中构建防腐层同样面临挑战。

  

无论是 React、Angular 还是 Vue 均有无数的数据层解决方案，从 Mobx、Redux、Vuex 等等，这些数据层方案对视图层实际上都会有入侵，有没有一种防腐层解决方案可以与视图层彻底解耦呢？以 RxJS 为代表的 Observable 方案在这时可能是最好的选择。

  

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIhzRZBiaWuic8wicxaFX9M6Whnokt1VsAfUDvbpqZkDxbnMBg9ePFkSM1qYb4mKM9FUPBVyX5K2ticPA/640?wx_fmt=png)

  

RxJS 是 ReactiveX 项目的 JavaScript 实现，而 ReactiveX 最早是 LINQ 的一个扩展，由微软的架构师 Erik Meijer 领导的团队开发。该项目目标是提供一致的编程接口，帮助开发者更方便的处理异步数据流。目前 RxJS 在开发中经常被作为响应式编程开发工具使用，但是在构建防腐层的场景中，RxJS 代表的 Observable 方案同样可以发挥巨大作用。

  

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIhzRZBiaWuic8wicxaFX9M6Wh8hC7zNH3EM2CLkSaTax5zvxPZo5DOrbW6tNssHZ3Fz4Xl1vPibFwebw/640?wx_fmt=png)

  

我们选择 RxJS 主要基于以下几点考虑：

  

*   统一不同数据源的能力：RxJS 可以将 websocket、http 请求、甚至用户操作、页面点击等转换为统一的 Observable 对象。
    

  

*   统一不同类型数据的能力：RxJS 将异步数据和同步数据统一为 Observable 对象。
    

  

*   丰富的数据加工能力：RxJS 提供了丰富的 Operator 操作符，可以对 Observable 在订阅前进行预先加工。
    

  

*   不入侵前端架构：RxJS 的 Observable 可以与 Promise 互相转换，这意味着 RxJS 的所有概念可以被完整封装在数据层，对视图层可以只暴露 Promise。
    

  

当在引入 RxJS 将所有类型的接口转换为 Observable 对象后，前端的视图组件将仅依赖 Observable，并与接口实现的细节解耦，同时，Observable 可以与 Promise 相互转换，在视图层获得的是单纯的 Promise，可以与任意数据层方案和框架搭配使用。

  

除了转换为 Promise 之外，开发者也可以与 RxJS 在渲染层的解决方案，例如 rxjs-hooks 混用，获得更好的开发体验。

  

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIhzRZBiaWuic8wicxaFX9M6WheWY8luRuaNkJD3j8zChN13rnSyNrFmNoRfgiaJqsKwDRyVSEBpaEmeQ/640?wx_fmt=png)

  

**三  防腐层实现**
------------

参照上文的防腐层设计，我们在开头的仪表盘项目中实现以 RxJS Observable 为核心的防腐层代码。

  

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIhzRZBiaWuic8wicxaFX9M6WhKicCO6gMwFywbIcgicDOxyR0FavVOhx2uicC4COiacNBl8UHBqicSk67kNA/640?wx_fmt=png)

  

其中防腐层的核心代码如下

  

MemoryUsagePercent 的实现代码如下，此时该组件将不再依赖具体的接口，而直接依赖防腐层的实现。

  

### 1  返回字段调整

返回字段变更时，防腐层可以有效拦截接口对组件的影响，当 /api/v2/quota/free 与 /api/v2/quota/usage 的返回数据变更为以下结构时

  

我们只需要调整防腐层的两行代码，注意此时我们的上层封装的 getMemoryUsagePercent 基于 Observable 构建所以不需要进行任何改动。

  

在 Observable 化的防腐层中，会存在高阶 Observable 与 低阶 Observable 两种设计，在上文的例子中，Free Observable 和 Usage Observable 为低阶封装，而 Percent Observable 利用 Free 和 Usage 的 Observable 进行了高阶封装，当低阶封装改动时，由于 Observable 本身的特性，高阶封装经常是不需要进行任何改动的，这也是防腐层给我们带来的额外好处。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIhzRZBiaWuic8wicxaFX9M6WhbGrsKdDpOzqhpN8qd3nwOz15TpTojKibljvKttH1nOWKKiaYkaP34cQQ/640?wx_fmt=png)

### 2  调用方式改变

当调用方式发生改变时，防腐层同样可以发挥作用。/api/v3/memory 直接返回了 free 与 usage 的数据，接口格式如下。

```
{
  requestId: string;
  data: {
    free: number;
    usage: number;
  }
}
```

防腐层代码只需要进行如下更新，就可以保障组件层代码无需修改。

```
export function getMemoryObservable(): Observable<{ free: number; usage: number }> {
  return fromFetch("/api/v3/memory").pipe(
    mergeMap((res) => res.json()),
    map((data) => data.data)
  );
}
export function getMemoryFreeObservable(): Observable<number> {
  return getMemoryObservable().pipe(map((data) => data.free));
}
export function getMemoryUsageObservable(): Observable<number> {
  return getMemoryObservable().pipe(map((data) => data.usage));
}
export function getMemoryUsagePercent(): Promise<number> {
  return lastValue(getMemoryObservable().pipe(
    map(({ usage, free }) => +((usage / (usage + free)) * 100).toFixed(2))
  ));
}
```

### 3  多版本共存使用

当前端代码需要在多套环境下部署时，部分环境下 v3 的接口可用，而部分环境下只有 v2 的接口部署，此时我们依然可以在防腐层屏蔽环境的差异。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIhzRZBiaWuic8wicxaFX9M6Whyng2ibbdky2IPJ9I1LKWRaHzU5BdZPCr6rRNqDFohFqlluhaVCqKWZw/640?wx_fmt=png)

```
export function getMemoryLegacyObservable(): Observable<{ free: number; usage: number }> {
  const legacyUsage = fromFetch("/api/v2/memory/usage").pipe(
    mergeMap((res) => res.json())
  );
  const legacyFree = fromFetch("/api/v2/memory/free").pipe(
    mergeMap((res) => res.json())
  );
  return forkJoin([legacyUsage, legacyFree], (usage, free) => ({
    free: free.data.free,
    usage: usage.data.usage,
  }));
}
export function getMemoryObservable(): Observable<{ free: number; usage: number }> {
  const current = fromFetch("/api/v3/memory").pipe(
    mergeMap((res) => res.json()),
    map((data) => data.data)
  );
  return race(getMemoryLegacyObservable(), current);
}
export function getMemoryFreeObservable(): Observable<number> {
  return getMemoryObservable().pipe(map((data) => data.free));
}
export function getMemoryUsageObservable(): Observable<number> {
  return getMemoryObservable().pipe(map((data) => data.usage));
}
export function getMemoryUsagePercent(): Promise<number> {
  return lastValue(getMemory().pipe(
    map(({ usage, free }) => +((usage / (usage + free)) * 100).toFixed(2))
  ));
}
```

通过 race 操作符，当 v2 与 v3 任何一个版本的接口可用时，防腐层都可以正常工作，在组件层无需再关注接口受环境的影响。

**四  额外应用**
-----------

防腐层不仅仅是多了一层对接口的封装与隔离，它还能起到以下作用。

### 1  概念映射

接口语义与前端需要数据的语义有时并不能完全对应，当在组件层直接调用接口时，所有开发者都需要对接口与界面的语义映射足够了解。有了防腐层后，防腐层提供的调用方法包含了数据的真实语义，减少了开发者的二次理解成本。

### 2  格式适配

在很多情况下，接口返回的数据结构与格式与前端需要的数据格式并不符合，通过在防腐层增加数据转换逻辑，可以降低接口数据对业务代码的入侵。在以上的案例里，我们封装了 getMemoryUsagePercent 的数据返回，使得组件层可以直接使用百分比数据，而不需要再次进行转换。

### 3  接口缓存

对于多种业务依赖同一接口的情况，我们可以通过防腐层增加缓存逻辑，从而有效降低接口的调用压力。

与格式适配类似，将缓存逻辑封装在防腐层可以避免组件层对数据的二次缓存，并可以对缓存数据集中管理，降低代码的复杂度，一个简单的缓存示例如下。

```
class CacheService {
  private cache: { [key: string]: any } = {};
  getData() {
    if (this.cache) {
      return of(this.cache);
    } else {
      return fromFetch("/api/v3/memory").pipe(
        mergeMap((res) => res.json()),
        map((data) => data.data),
        tap((data) => {
          this.cache = data;
        })
      );
    }
  }
}
```

### 4  稳定性兜底

当接口稳定性较差时，通常的做法是在组件层对 response error 的情况进行处理，这种兜底逻辑通常比较复杂，组件层的维护成本会很高。我们可以通过防腐层对稳定性进行兜底，当接口出错时可以返回兜底业务数据，由于兜底数据统一维护在防腐层，后续的测试与修改也会更加方便。在上文中的多版本共存的防腐层中，增加以下代码，此时即使 v2 和 v3 接口都无法返回数据，前端仍然可以保持可用。

```
return race(getMemoryLegacy(), current).pipe(
+   catchError(() => of({ usage: '-', free: '-' }))
  );
```

### 5  联调与测试

接口和前端可能会存在并行开发的状态，此时，前端的开发并没有真实的后端接口可用。与传统的搭建 mock api 的方式相比，在防腐层直接对数据进行 mock 是更方便的方案。

```
export function getMemoryFree(): Observable<number> {
  return of(0.8);
}
export function getMemoryUsage(): Observable<number> {
  return of(1.2);
}
export function getMemoryUsagePercent(): Observable<number> {
  return forkJoin([getMemoryUsage(), getMemoryFree()]).pipe(
    map(([usage, free]) => +((usage / (usage + free)) * 100).toFixed(2))
  );
}
```

在防腐层对数据进行 mock 也可以用于对页面的测试，例如 mock 大量数据对页面性能影响。

```
export function getLargeList(): Observable<string[]> {
  const options = [];
  for (let i = 0; i < 100000; i++) {
    const value = `${i.toString(36)}${i}`;
    options.push(value);
  }
  return of(options);
}
```

**五  总结**
---------

在本文中我们介绍了以下内容：

1.  前端面对接口频繁变动时的困境及原因如何
    
2.  防腐层的设计思想与技术选型
    
3.  使用 Observable 实现防腐层的代码示例
    
4.  防腐层的额外作用
    

请读者注意，只在特定的场景下引入前端防腐层才是合理的，即前端处于跟随者或供应商 / 客户关系中，且面临大量接口无法保障稳定和兼容。如果在防腐层可以在后端 Gateway 构建，或者接口数量较少时，引入防腐层带来的额外成本会大于其带来的好处。

RxJS 在防腐层构建场景下提供的更多的是 Observable 化的能力，如果读者不需要复杂的 operators 转换工具，也可以自行构建 Observable 构建方案，事实上只需要 100 行的代码就可以实现 https://stackblitz.com/edit/mini-rxjs。

改造后的前端架构将不再直接依赖接口实现，不会入侵现有前端数据层设计，还可以承担概念映射、格式适配、接口缓存、稳定性兜底以及协助联调测试等工作。文中所有的示例代码都可以在仓库 https://github.com/vthinkxie/rxjs-acl 获得。