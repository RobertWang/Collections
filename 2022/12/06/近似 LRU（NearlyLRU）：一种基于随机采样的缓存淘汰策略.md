> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7145457156433641502)*   从所有元素里面随机采样 5 个元素
*   淘汰 5 个里面`最后一次被访问的时间戳最小的元素`

可以发现第 2 步其实就有 LRU 的思想，只是 LRU 是选取全部元素里面`最后一次被访问的时间戳最小的元素`，而 NearlyLRU 则是采样一小批。

因此 NearlyLRU 其实命中率是不如 LRU 的，但是它的好处也是明显的，不需要额外的数据结构。

如果想提高命中率，可以增大采样数量，这样会更加接近 LRU，当然时间成本也会相应的上升。

实现
==

一个 Golang 的简单实现。

数据结构
----

可以看到数据结构非常简单，基本上只有一个 map 类型的 entries，entries 的值是`lastAccessEntry`，也就是 Key 和 Value 加上一个最后一次访问的时间戳，其实这个时间戳可以使用更加轻量的类型，比如 int32。

```
type Cache[K comparable, V any] struct {
	entries  map[K]*lastAccessEntry[K, V]
	capacity int                 // 容量
	samples  int                 // 淘汰时采样数量
	onEvict  cache.OnEvict[K, V] // 淘汰时的回调函数
}

type lastAccessEntry[K comparable, V any] struct {
	entry      *cache.Entry[K, V]
	lastAccess time.Time // 最后一次使用时间
}
复制代码

```

添加元素
----

添加元素分为三个小部分，第一个是如果元素存在直接设置新值和时间戳；第二个是如果缓存满了则淘汰一个元素；最后是添加新元素。

```
// 添加或更新元素
// 返回被淘汰的元素
func (c *Cache[K, V]) Put(key K, value V) *cache.Entry[K, V] {
	// 如果 key 已经存在，直接设置新值
	if entry, ok := c.entries[key]; ok {
		entry.entry.Value = value
		entry.lastAccess = time.Now()
		return nil
	}

	// 如果已经到达最大尺寸，先剔除一个元素
	var evicted *cache.Entry[K, V]
	if c.Full() {
		evicted = c.Evict()
	}

	// 添加元素
	c.entries[key] = &lastAccessEntry[K, V]{
		entry: &cache.Entry[K, V]{
			Key:   key,
			Value: value,
		},
		lastAccess: time.Now(),
	}
	return evicted
}
复制代码

```

淘汰元素
----

因为 Golang 的 map 遍历的时候本来就是随机的（Golang 故意加了随机种子，避免依赖于 map 的顺序），因此我们采样的时候直接使用遍历采集需要的样本，然后挑选里面时间戳最小的进行淘汰。

```
// 淘汰元素
func (c *Cache[K, V]) Evict() *cache.Entry[K, V] {
	// 采样
	var evictEntry *lastAccessEntry[K, V]
	i := 0
	for _, entry := range c.entries {
		// 挑选里面时间戳最小的
		if evictEntry == nil || entry.lastAccess.Before(evictEntry.lastAccess) {
			evictEntry = entry
		}
		i++
		if i >= c.samples {
			break
		}
	}
	if evictEntry == nil {
		return nil
	}
        // 淘汰
	delete(c.entries, evictEntry.entry.Key)
	// 回调
	if c.onEvict != nil {
		c.onEvict(evictEntry.entry)
	}
	return evictEntry.entry
}
复制代码

```

设置采样数量
------

这个策略有一个参数需要设置，也就是采样数量。

```
// 设置采样个数
func (c *Cache[K, V]) SetSamples(samples int) {
        // 采样数量不能太小，否则和随机没区别
	if samples < MinSamples {
		samples = MinSamples
	}
        // 也不能太大，否则和LRU没区别
	if c.Cap() < samples {
		panic("too large samples")
	}
	c.samples = samples
}
复制代码

```

测试
==

测试是在一个有 20.6w 条日志记录的数据集里面进行，分别测试缓存大小为数据集合大小的 0.1%、0.3%、0.5%、0.7%、1%、2%、3%、5% 和 10% 的情况下的命中率。

LRU
---

<table><thead><tr><th>缓存比例</th><th>命中数量</th><th>命中率</th></tr></thead><tbody><tr><td>0.1%</td><td>26717</td><td>12.97%</td></tr><tr><td>0.3%</td><td>58169</td><td>28.23%</td></tr><tr><td>0.5%</td><td>87446</td><td>42.44%</td></tr><tr><td>0.7%</td><td>114358</td><td>55.50%</td></tr><tr><td>1.0%</td><td>148556</td><td>72.10%</td></tr><tr><td>2.0%</td><td>187286</td><td>90.89%</td></tr><tr><td>3.0%</td><td>190649</td><td>92.53%</td></tr><tr><td>5.0%</td><td>192606</td><td>93.48%</td></tr><tr><td>10.0%</td><td>192842</td><td>93.59%</td></tr></tbody></table>

Random
------

<table><thead><tr><th>缓存比例</th><th>命中数量</th><th>命中率</th></tr></thead><tbody><tr><td>0.1%</td><td>26320</td><td>12.77%</td></tr><tr><td>0.3%</td><td>55328</td><td>26.85%</td></tr><tr><td>0.5%</td><td>79768</td><td>38.71%</td></tr><tr><td>0.7%</td><td>100646</td><td>48.85%</td></tr><tr><td>1.0%</td><td>125155</td><td>60.74%</td></tr><tr><td>2.0%</td><td>167517</td><td>81.30%</td></tr><tr><td>3.0%</td><td>181521</td><td>88.10%</td></tr><tr><td>5.0%</td><td>190827</td><td>92.61%</td></tr><tr><td>10.0%</td><td>192842</td><td>93.59%</td></tr></tbody></table>

NearlyLRU
---------

这里多添加了一个采样数量参数，分别是 5、10 和 50 个。

<table><thead><tr><th>采样数量</th><th>缓存比例</th><th>命中数量</th><th>命中率</th></tr></thead><tbody><tr><td>5</td><td>0.1%</td><td>26692</td><td>12.95%</td></tr><tr><td>5</td><td>0.3%</td><td>56646</td><td>27.49%</td></tr><tr><td>5</td><td>0.5%</td><td>84593</td><td>41.05%</td></tr><tr><td>5</td><td>0.7%</td><td>108402</td><td>52.61%</td></tr><tr><td>5</td><td>1.0%</td><td>140077</td><td>67.98%</td></tr><tr><td>5</td><td>2.0%</td><td>182233</td><td>88.44%</td></tr><tr><td>5</td><td>3.0%</td><td>189174</td><td>91.81%</td></tr><tr><td>5</td><td>5.0%</td><td>192474</td><td>93.41%</td></tr><tr><td>5</td><td>10.0%</td><td>192842</td><td>93.59%</td></tr><tr><td>10</td><td>0.1%</td><td>26442</td><td>12.83%</td></tr><tr><td>10</td><td>0.3%</td><td>57389</td><td>27.85%</td></tr><tr><td>10</td><td>0.5%</td><td>86084</td><td>41.78%</td></tr><tr><td>10</td><td>0.7%</td><td>112183</td><td>54.45%</td></tr><tr><td>10</td><td>1.0%</td><td>144483</td><td>70.12%</td></tr><tr><td>10</td><td>2.0%</td><td>185303</td><td>89.93%</td></tr><tr><td>10</td><td>3.0%</td><td>190134</td><td>92.28%</td></tr><tr><td>10</td><td>5.0%</td><td>192562</td><td>93.45%</td></tr><tr><td>10</td><td>10.0%</td><td>192842</td><td>93.59%</td></tr><tr><td>50</td><td>0.1%</td><td>26701</td><td>12.96%</td></tr><tr><td>50</td><td>0.3%</td><td>57972</td><td>28.14%</td></tr><tr><td>50</td><td>0.5%</td><td>87013</td><td>42.23%</td></tr><tr><td>50</td><td>0.7%</td><td>113776</td><td>55.22%</td></tr><tr><td>50</td><td>1.0%</td><td>147484</td><td>71.58%</td></tr><tr><td>50</td><td>2.0%</td><td>187028</td><td>90.77%</td></tr><tr><td>50</td><td>3.0%</td><td>190618</td><td>92.51%</td></tr><tr><td>50</td><td>5.0%</td><td>192600</td><td>93.47%</td></tr><tr><td>50</td><td>10.0%</td><td>192842</td><td>93.59%</td></tr></tbody></table>

测试结果
----

可以看到 NearlyLRU 的命中率是不如 LRU 的，但是比 Random 好很多，在采样数量 5 就能达到 67% 的命中率。而且随着采样数量增加，在采样数量 50 的时候已经接近 LRU。

总结
==

这是一个非常有意思的结构，它也是带有随机性的，但是它通过引入时间戳 + 采样，在避免了空间的消耗的同时，还能保证不错的命中率，而且实现非常简单。适合那些需要简单的淘汰策略，不能有太多的额外空间消耗的场景。