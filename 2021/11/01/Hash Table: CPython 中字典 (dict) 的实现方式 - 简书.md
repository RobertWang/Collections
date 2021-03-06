> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/d234acb0e150)

众所周知在 Python 中，字典（dict）的插入、删除、查询操作的平均复杂度都是 O(1)。对比来看其他常见的数据结构，比如动态数据、链表、二叉树等等都无法同时满足三种操作的平均复杂度为 O(1)。面对这种开了挂的数据结构，之前只知道 dict 是用 Hash Table 实现的，这次通过阅读 Python 的 wiki 把零散的实现细节拼凑了起来：

**Hash 函数:**

hash 函数是实现 Hash Table 的基础，hash 函数可以接收一个对象并返回一个数值，这个数值就是 hash 值，前提是传入的对象是不可变对象，在 Python 中数值型，字符串，元组等属于不可变对象，而数组、dict 等对象属于可变对象。如果传入一个可变对象，hash 函数会抛出异常，比如 a = {}  a[[1,2]] = 1 就会抛出异常，异常信息大概是说这里的 [1,2] 属于数组，数组是一个 unhashable 的类型，这样的解释等于没有解释。那么为什么数组是 unhashable 的呢，因为数组作为可变对象，如果我们采用数组的地址作为计算 hash 值的依据，对于两个数组比如 a = [1, 2] b = [1, 2]，a == b is True，但是 a 与 b 的地址是不一样的，所以不能采用数组的地址作为 hash 依据；如果采用数组的值作为 hash 的依据，因为数组是可变对象，当数组发生变化时 hash 值是不能跟着变化的，这个数组对象的 hash 值与数组的值不再是 hash 关系，所以也不能采用数组的值作为 hash 依据，所以数组这类可变对象是 unhashable 的。但是对于一个合格的 hash 函数来说，还需要满足在绝大多数情况下当 a != b 时 hash(a) != hash(b)，这并不容易，而且是比较有难度的数学问题，我就无法解释了，但无论如何 CPython 已经实现了一个合格的 hash 函数，并且将在后面要介绍的 **PyDictEntry** 中用到。

**PyDictEntry:**

我们知道 dict 是 key-value 的结构，而其中的 key 又可以通过 hash(key) 获得一个 hash value。通过查看 CPython 的源码可以发现，Python 实现了这样一个数据结构: <hash value, key, value>，就是将 key 的 hash 值，key 以及 value 封装到一起形成一个 **PyDictEntry**（Python 源码中的命名，就不乱翻译了）。这个数据结构就是 dict 中每个 key-value 本身的真实形式。在后面要介绍的 dict 插入、查询、删除都要使用到 **PyDictEntry**。

**Hash Table:**

说到这里 Hash Table 就非常容易解释了，Hash Table 就是一个数组，这个数组采用了连续的地址，数组中每个元素的长度都是一样的，并且每个元素就是上面介绍的 **PyDictEntry** 的实例。我们都知道在 C 语言中根据下标从数组中获取一个元素的复杂度是 O(1)，因为数组的起始地址 p + 下标 k * 每个元素的长度 L，就可以获得下标 k 这个元素的起始地址了。CPython 的 Hash Table 是一个初始长度为 8 的可变长度数组。变长数组的查询肯定还是 O(1)，至于插入跟删除是不是 O(1) 后面会介绍，现在我们离 dict 的 O(1) 只差一步了，就是如何把一个 key 的 hash value 对应到变长数组的下标。下面要介绍的 Hash Table 的插入，查询，删除这三种操作的实现方式，实际上就是完成了这一步。

**Hash Table 插入：**

根据上面的介绍，Hash Table 的插入就是指在一个 Hash Table 数组中插入 **PyDictEntry**。我们假设目前 Hash Table 的长度是 n，把一个 hash value 对 n 取模就可以得到一个 0 到 n-1 之间的数字，之所以是取模是因为 hash value 可能是负数。很显然取模后的值正好是 Hash Table 的一个下标，假如这个下标对应的元素是空的，我们就直接插入 **PyDictEntry，**但是这个下标有可能已经被其他 **PyDictEntry** 占用了，如果占用的 PyDictEntry 与被占用的 PyDictEntry 两者的 hash value 以及 key 都是一样的，就说明这个插入操作其实是修改操作，所以只要更新 value 就可以了，如果不是这种情况那就是发生了冲突，面对冲突 CPython 做了两件事情来避免或者解决：机制 1：设置一个装填系数，比如 0.5，就是说当 Hash Table 中的元素个数达到数组长度的 1/2 时，数组长度就扩大一倍，在 CPython 中这个装填系数是 2/3，数组的长度扩大了，冲突的可能性就大大降低了，这个机制可以避免大多数的冲突，但是毕竟不是绝对避免。所以当冲突无法避免时就需要机制 2 来解决了；机制 2：. 假设发生冲突的是下标 j，Hash Table 的长度是 2**k，就根据这个下标 j 通过一个 probe 算法生成一个下标 probe(j)， 假设 probe(j) 等于 f，就把这个冲突的 **PyDictEntry** 尝试放到下标 f 里，当然下标 f 可能依然是冲突的，那就放到下标 probe(f) 里，依次类推直到找到一个空的下标。一个好的 probe 算法就可以保证寻找的次数是很少的并且最终必然能找到，至于 CPython 用的 probe 算法为什么可以保证这一点嘛，还是那句话，这个是离散数学问题，恕我没有能力解释了。。。probe 算法虽然没有能力介绍但是有一点是可以看出来的，根据这个算法，对于一个冲突的下标 j，使用 probe 算法最终会找到一个空的下标，这个推论将在下面要介绍的 Hash Table 查询中用到。

**Hash Table 插入的复杂度：**  

还有一个问题：机制 1 中的扩容操作，每次扩大一倍以后，需要把原有 Hash Table 里的 **PyDictEntry** 重新插入到新的数组里，这带来了额外的开销，那么 Hash Table 的插入复杂度还是 O(1) 吗？根据求极限方式可以证明当 n 足够大时，扩容带来的复杂度增加最终会被平均掉。这里简单的证明下（不严谨，但可以说明问题）：先假设装填系数是 1， 就是说满了才扩容一倍，因为初始的长度是 8，所以扩容都是发生在 2 的平方数（假设这个 2 的平方数是 i，i 依次是 8，16，32。。。），在 i 这些节点上重新插入带来的额外开销也是 i 次，所以对于长度是 n 的 Hash Table，额外的开销就是 8+16+32+...+i+...n/2，这个等比数列求和等于 n-8，n-8 次的额外开销被插入 n 次平均一下就是 O(1)，所以总体的开销是 O(1)+O(1) 还是 O(1)。这不禁让我想起了那句名言：“所谓的效率，无非就是空间换时间”

**Hash Table 查询：**  

其实 Hash Table 的查询不需要介绍了，因为查询与插入是一样的，先根据 key 的 hash value 找下标，如果找到的 hash value 与 key 都匹配则返回对应的 value，查询就完成了复杂度是 O(1)，但是也有冲突的时候，就需要用到 probe 算法了，根据上面提到的推论，最终还是能找到，这个时候复杂度就比 O(1) 大了，但是在大部分的情况下扩容避免了冲突的发生，即便在冲突发生时，通过 probe 算法还是可以很快的找到 hash value 与 key 都匹配的 **PyDictEntry**，所以平均复杂度依然是 O(1)。

**Hash Table 删除：**

下意识的，我们认为删除操作与查询操作是一样的，只不过是查询到后删掉而已，跟查询没有本质区别。这样理解是对的，但是这样做却是错的，因为 Hash Table 的删除操作并不是真的删掉查询到的下标对应的 **PyDictEntry，**或者说并不能把这个下标的元素设置成空指针，为什么呢？其实在上面介绍查询的时候还有一点没有说明，插入始终能成功，但是查询就不是了，假如要查询的 key 不存在，那应当在 O(1) 内抛出异常，那如何做到 O(1) 而不是 O(logn) 呢，其实很简单，从插入的算法来看，插入的顺序是一定的，所以在查询的顺序上发现一个下标对应的元素是空的就可以停止查询了，因为在假设没有删除操作的情况下，**如果要查询的 key 不存在，那么在查询到这个 key 之前的下标都必然是不为空的**，那么在有删除操作的情况下，上面加粗的判断依然要成立，所以删除不能让被删除的下标指向空，但是不指向空不是等于没删除吗，怎么办呢？C 语言自有妙计，不能指向 Null 就指向 dummy 指针，*dummy 不与任何值相等，包括 Null。这样就既实现了删除（不与任何值相等）又避免了降低查询效率（也不与 Null 相等）。

**总结：**

写到这里感觉自己等于什么都没讲，毕竟离散数学才是本质，但是数学证明嘛，只能呵呵了。好在计算机与纯数学还是有区别的，计算机可以把一些数学上错误的 “定理” 拿来先用着，比如上面的 Hash 函数，比如 md5。