> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_42310154/article/details/119386707)

### 一、简介

[布隆过滤器](https://so.csdn.net/so/search?q=%E5%B8%83%E9%9A%86%E8%BF%87%E6%BB%A4%E5%99%A8&spm=1001.2101.3001.7020)是由布隆（Burton Howard Bloom）在 1970 年提出的，它实际上是由**一个很长的二进制向量**和**一系列随机映射函数**组成，布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率（假正例 False positives，即 Bloom Filter 报告某一元素存在于某集合中，但是实际上该元素可能并不在集合中）和删除困难，但是没有识别错误的情形（即假反例 False negatives，如果 Bloom Filter 报告某元素不在集合中，那么该元素一定没有在该集合中）

**关键两点：**（第三部分图解例子解释原因）

*   Bloom Filter 报告某一元素存在于集合中，但是实际上该元素可能并不在集合中
*   Bloom Filter 报告某一元素不存在于集合中，那么该元素一定不在集合中

### 二、算法原理

布隆过滤器由两部分组成：

*   二进制数组（元素只为 0 或 1，java 中可用 byte 数组来存）
*   多个哈希函数（用来计算每个 key 在数组中的位置）

算法原理：  
1、初始化  
（1）二进制数组初始化长度 n，所有值默认为 0  
（2）多个哈希函数初始化（常用哈希计算方法均可），其可把传入 key 映射为一个整数，然后对数组长度求余（或和数组长度减 1 相与）得到一个小于 n 的索引

2、添加元素  
（1）用多个哈希函数分别计算传入 key 在数组中的索引值 i  
（2）将数组中对应 i 位置设为 1

3、判断元素是否存在  
（1）用多个哈希函数分别计算传入 key 在数组中的索引值 i1、i2…  
（2）依次查询数组中对应 i1、i2… 位置是否为 1，若有某一个不为 1，则直接返回不存在；否则认为存在

### 三、图解例子

![](https://img-blog.csdnimg.cn/3b2f3240f514437e9cee6f07cfefa395.png?)  
如上图，设置哈希函数个数为 3，数组长度为 18。按照算法步骤添加 x、y、z，将相应位置 1。

判断 w 是否存在，通过哈希值计算，发现第 16 个位置上的值为 0，表明该元素一定不存在。因为如果 w 存在，由于哈希函数计算方式是固定的，则对应位一定为 1，如果不为 1，就说明 w 之前肯定没有出现过。

但另一方面，如果此时传入一个元素三个位置都为 1，认为它存在，但实际上有可能它并不存在，因为存在**哈希冲突**，可能之前有某一个元素经过哈希计算也是这三个位置的索引，因此会误判。

### 四、优缺点

优点：

*   时间和空间上相比于其他数据结构有很大优势，和普通哈希表相比不需要存储元素本身

缺点：

*   存在误算，即认为本不存在的元素存在
*   不能删除元素

### 五、常见问题

1、为何要使用多个哈希函数？

Hash 本身就会面临冲突，如果只使用一个哈希函数，那么冲突的概率会比较高。例如长度 100 的数组，如果只使用一个哈希函数，添加一个元素后，添加第二个元素时冲突的概率为 1%，添加第三个元素时冲突的概率为 2%… 但如果使用两个哈希函数，添加一个元素后，添加第二个元素时冲突的概率降为万分之 4（四种可能的冲突情况，情况总数 100x100）

### 六、[go 语言](https://so.csdn.net/so/search?q=go%E8%AF%AD%E8%A8%80&spm=1001.2101.3001.7020)实现

```
package main

import (
	"fmt"

	"github.com/bits-and-blooms/bitset"
)

//设置哈希数组默认大小为16
const DefaultSize = 16

//设置种子，保证不同哈希函数有不同的计算方式
var seeds = []uint{7, 11, 13, 31, 37, 61}

//布隆过滤器结构，包括二进制数组和多个哈希函数
type BloomFilter struct {
	//使用第三方库
	set *bitset.BitSet
	//指定长度为6
	hashFuncs [6]func(seed uint, value string) uint
}

//构造一个布隆过滤器，包括数组和哈希函数的初始化
func NewBloomFilter() *BloomFilter {
	bf := new(BloomFilter)
	bf.set = bitset.New(DefaultSize)

	for i := 0; i < len(bf.hashFuncs); i++ {
		bf.hashFuncs[i] = createHash()
	}
	return bf
}

//构造6个哈希函数，每个哈希函数有参数seed保证计算方式的不同
func createHash() func(seed uint, value string) uint {
	return func(seed uint, value string) uint {
		var result uint = 0
		for i := 0; i < len(value); i++ {
			result = result*seed + uint(value[i])
		}
		//length = 2^n 时，X % length = X & (length - 1)
		return result & (DefaultSize - 1)
	}
}

//添加元素
func (b *BloomFilter) add(value string) {
	for i, f := range b.hashFuncs {
		//将哈希函数计算结果对应的数组位置1
		b.set.Set(f(seeds[i], value))
	}
}

//判断元素是否存在
func (b *BloomFilter) contains(value string) bool {
	//调用每个哈希函数，并且判断数组对应位是否为1
	//如果不为1，直接返回false，表明一定不存在
	for i, f := range b.hashFuncs {
		//result = result && b.set.Test(f(seeds[i], value))
		if !b.set.Test(f(seeds[i], value)) {
			return false
		}
	}
	return true
}

func main() {
	filter := NewBloomFilter()
	filter.add("asd")
	fmt.Println(filter.contains("asd"))
	fmt.Println(filter.contains("2222"))
	fmt.Println(filter.contains("155343"))
}

```

输出结果如下：

```
true
false
false

```