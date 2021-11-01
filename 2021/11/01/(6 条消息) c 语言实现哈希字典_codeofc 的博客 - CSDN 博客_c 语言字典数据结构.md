> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/u013113678/article/details/113765937)

做项目写业务逻辑的时候经常用到的数据结构有 2 个，一个是动态数组还一个是字典。大部分编程语言，都有提供上述两种数据结构的集合类标准库。但是 c 语言是没有提供这类标准库的，所以本文的目的是提供一种 c 语言实现哈希字典的方法。

程序设计
----

**一、接口的设计**  
**1、数据结构**  
虽然基于接口最小化原则，最好是不要提供任何结构体，而提供句柄之类的方式。但是字典这种数据结构有时还是需要在栈上使用的，为了让使用者能够更灵活的选择不同的内存分配策略，所以个人认为字典应该对外暴露其结构体。具体定义如下:

```
#define TKey void*
#define TValue void*
typedef struct {
	void* key;
	void*value;
	int _isptr;
	int _keySize;
	int _hashCode;
	int _next;
}acf_map_entry;
typedef struct  {
	int count;
	int capacity;
	int _freeIndex;
	int* _buckets;
	map_entry* _entries;
	int _removeList;
}acf_map;
typedef struct  {
	map_entry* current;
	map * _map;
	int _index;
}acf_ map_iter;
```

**（1）acf_map_entry**  
是字典中的数据元素的载体，一个 acf_map_entry 表示一个键值对。key 存储键，value 存储值。字典的键有 4 种形式，元类型、指针类型、结构体、字符串。这里把指针类型和小于等于指针长度的元类型算作值类型、大于指针长度的元类型和结构体和字符串算作指针类型，通过_isptr 区分，0 为值类型，1 为指针类型。_keySize 是当_isptr 为 1 时，key 指向的数据的长度。_hashCode 为当前键计算出的哈希值，_next 为开放地址线性搜索的下一个元素。

**（2）acf_map**  
表示一个字典对象。count 表示键值对长度，capacity 表示数组长度，_freeIndex 为可用的下标， _buckets 为索引表数组， _entries 为键值对数组， _removeList 为删除列表起始下标

**（3）acf_map_iter**  
则是迭代器，用于遍历字典元素。current 为当前元素，_map 为当前哈希字典对象，_index 为当前迭代下标。

**2、提供的方法**  
一个能满足大部分使用场景的字典一般需要提供如下方法：

```
/// <summary>
/// 初始化字典，使用字典之前需要初始化
/// </summary>
/// <param >[in]字典对象的指针</param>
void acf_map_init(acf_map*map);
/// <summary>
/// 反初始化字典，在字典销毁之前需要进行反初始化
/// </summary>
/// <param >[in]字典对象的指针</param>
void acf_map_deinit(acf_map* map);
/// <summary>
/// 插入元素
/// </summary>
/// <param >[in]字典对象的指针</param>
/// <param >[in]键，值类型的键，直接传值</param>
/// <param >[in]值</param>
void acf_map_add(acf_map* map, TKey  key , TValue value);
/// <summary>
/// 插入元素
/// </summary>
/// <param >[in]字典对象的指针</param>
/// <param >[in]键，指针类型的键，传键的地址</param>
/// <param >[in]键的长度，key 指向数据的长度</param>
/// <param >[in]值</param>
void acf_map_add_ptr(acf_map* map, TKey key, int len, TValue value);
/// <summary>
/// 删除元素
/// </summary>
/// <param >[in]字典对象的指针</param>
/// <param >[in]键，值类型的键，直接传值</param>
/// <returns>是否删除成功</returns>
int acf_map_remove(acf_map* map, TKey key);
/// <summary>
/// 插入元素
/// </summary>
/// <param >[in]字典对象的指针</param>
/// <param >[in]键，指针类型的键，传键的地址</param>
/// <param >[in]键的长度，key 指向数据的长度</param>
/// <param >[in]值</param>
/// <returns>是否删除成功</returns>
int acf_map_remove_ptr(acf_map* map, TKey key, int len);
/// <summary>
/// 查询
/// </summary>
/// <param >[in]字典对象的指针</param>
/// <param >[in]键，值类型的键，直接传值</param>
/// <param >[out]值，传地址作为返回值</param>
/// <returns>是否查找成功</returns>
int acf_map_get(acf_map* map, TKey key, TValue* value);
/// <summary>
/// 查询
/// </summary>
/// <param >[in]字典对象的指针</param>
/// <param >[in]键，指针类型的键，传键的地址</param>
/// <param >[in]键的长度，key 指向数据的长度</param>
/// <param >[out]值，传地址作为返回值</param>
/// <returns>是否查找成功</returns>
int acf_map_get_ptr(acf_map* map, TKey key,  int len, TValue* value);
/// <summary>
/// 清除
/// </summary>
/// <param >[in]字典对象的指针</param>
void acf_map_clear(acf_map* map);
/// <summary>
/// 初始化迭代器，迭代器可重复初始化
/// </summary>
/// <param >[in]字典对象的指针</param>
/// <param >[in]迭代器对象的指针</param>
void acf_map_iter_reset(acf_map* map,acf_map_iter*iter);
/// <summary>
/// 迭代器迭代
/// </summary>
/// <param >[in]字典对象的指针</param>
/// <param >[in]迭代器对象的指针</param>
/// <returns>是否取得下一个元素/returns>
int acf_map_iter_next(acf_map_iter*itierator);
/// <summary>
/// 迭代宏
/// </summary>
/// <param >[in]迭代器对象的指针</param>
/// <param >[in]字典对象的指针</param>
#define acf_map_for_each(iter, map)  acf_map_iter_reset(map, &iter);while (acf_map_iter_next(&iter))
```

**（1）初始化和反初始化方法**  
类似于 c++ 的构造析构函数，字典对象在创建后是需要初始化，销毁之前需要反初始化的。参数是字典对象的指针，这就可以由使用者决定，字典对象是放在栈内存还是堆内存。

**（2）插入**  
字典必然是有插入方法的，向字典添加一个键值对，参数为 key 和 value，如果有相同的 key 则覆盖原来的 value。提供了两个方法，一个是 key 是值类型，另外一个 key 是指针类型。

**（3）删除**  
删除是与插入相反的操作，从字典中移除某个键值对，如果成功查找到并删除则返回 1 否则返回 0。同样提供了两个方法一个是 key 是值类型，另外一个 key 是指针类型。  
**（4）查询**  
查询是通过 key 值搜索到字典中对应的 value，如果成功查找到元素则返回 1 否则返回 0，value 通过出参的方式提供。同上提供了 2 个方法。  
**（5）清除**  
清除则是将字典中的所有元素清除。  
**（6）遍历**  
遍历字典中的所有元素。提供的是迭代器的方法，同时用宏进行包装，简化调用。

**二、关键实现**  
**1、存储方式**  
如一、1 的 acf_map 所示，数据存储方式是通过一个数组实现的，即字典内的所有键值对都存储在一个数组_entries 中。然后再通过一个索引表_buckets 去指向它们，索引表也是一个数组长度与键值对数组相同。

```
int* _buckets;
map_entry* _entries;
```

**2、计算哈希值**  
进行哈希查找的时候需要进行哈希计算，由键计算出哈希值，哈希值即是索引表_bucket 的下标，通过_buckets 对应下标的值就可找到具体的键值对数组_entries 对应的元素。由两个步骤如下：  
**（1）由键收缩到指针长度的哈希值**  
这一步是针对键的长度大于指针长度的情况，需要将键通过一个哈希函数收缩到指针长度大小。这里有一定的选择策略，要考虑让分散以及计算速度达到一个较好的平衡，计算结果过于集中计算速度快意义不大，计算结果分散但计算速度慢也不行。本文使用的是 murmur_hash2 算法，略微偏向于分散。murmur_hash2 算法见于五、2。

**（2）指针长度的哈希值收缩到索引表长度的哈希值**  
将一个大数字收缩到一个小的数字可使用如下方法：  
①取余数法  
使用哈希值对数组长度取余数，结果必然小于数组长度。  
②位与法  
使用哈希值对数组长度 - 1 位与，结果必然小于数组长度。但是需要保证数组长度是 2 的 n 次方，来确保结果的分散。

具体的基于取余数法的查找实现如下：

```
static int getHashCode(void* key, int length,int isptr)
{
	int hashcode;
	if (isptr)
	{
		hashcode = murmur_hash2(key,length);
	}
	else {
		hashcode = key;
	}
	return hashcode;
}
static int acf_map_find_entry(acf_map* _this,TKey key, int len,  int isptr,int * pHashCode,int * pBucket,int*pLast)
{
	 int hashCode = getHashCode(key, len, isptr) & 0x7FFFFFFF;
	 int bucket = hashCode % _this->capacity;
	 *pHashCode = hashCode;
	 *pBucket = bucket;
	 *pLast = -1;
	 if (isptr)
	 {
		 for (int i = _this->_buckets[bucket]; i >= 0; i = _this->_entries[i]._next)
		 {
			 if (_this->_entries[i]._hashCode == hashCode && _this->_entries[i]._keySize == len&& memcmp(key, _this->_entries[i].key, len)==0)
				 //找到元素了
			 {		
				 return i;
			 }
			 *pLast = i;
		 }
	 }
	 else
	 {
		 for (int i = _this->_buckets[bucket]; i >= 0; i = _this->_entries[i]._next)
		 {
			 if (_this->_entries[i]._hashCode == hashCode&&key == _this->_entries[i].key)
				 //找到元素了
			 {				 
				 return i;
			 }
			 *pLast = i;
		 }
	 }	 
	 return -1;
}
```

**3、解决冲突**  
上面的步骤经过计算得到的哈希值，是可能出现冲突的，如果出现冲突，那就需要一个解决冲突的策略。  
这里使用开放地址法之线性探测法：  
一、1 中的 map_entry 有个_next 字段，此字段记录的是下个探测的元素数组下标，为 - 1 时则没有下一个元素。  
具体实现如下:

```
static void acf_map_add_internal(acf_map * _this, TKey key, int len, TValue value, int isptr)
{
	int hashCode;
	int bucket;
	int last;
	int index;
	if (_this->_buckets == 0)
	{
		int size = getPrime(0);
		_this->_buckets = malloc(sizeof(int)*size);
		_this->_entries = malloc(sizeof(acf_map_entry)*size);
		for (int i = 0; i < size; i++) {
			_this->_buckets[i] = -1;
			_this->_entries[i]._next = -1;
			_this->_entries[i]._hashCode = -1;
		}
		_this->capacity = size;
		_this->_freeIndex = 0;
		 hashCode = getHashCode(key, len, isptr) & 0x7FFFFFFF;
		 bucket = hashCode % _this->capacity;
	}
	else
	{
		int findIndex = acf_map_find_entry(_this, key, len, isptr, &hashCode, &bucket, &last);
		if (findIndex > -1)
		{
			_this->_entries[findIndex].value = value;
			return;
		}
	}


	if (_this->_freeIndex < _this->capacity)//有可用下标
	{
		index = _this->_freeIndex++;
	}
	else if (_this->_removeList >= 0)//有删除可用下标
	{
		index = _this->_removeList;
		_this->_removeList = _this->_entries[_this->_removeList]._next;
	}
	else//无可用下标，拓展容量 
	{
		acf_map_resize(_this, _this->capacity << 1);
		bucket = hashCode % _this->capacity;
		index = _this->_freeIndex++;
	}
	_this->_entries[index]._next = _this->_buckets[bucket];
	_this->_entries[index]._hashCode = hashCode;
	_this->_entries[index]._isptr = isptr;
	if (isptr)
	{
		_this->_entries[index].key = malloc(len+1);
		memcpy(_this->_entries[index].key,key,len);
		((char*)(_this->_entries[index].key))[len] = 0;
	}
	else
	{
		_this->_entries[index].key = key;
	}
	_this->_entries[index]._keySize = len;
	_this->_entries[index].value = value;
	_this->_buckets[bucket] = index;
	_this->count++;
}
```

**4、动态增容**  
大部分字典使用场景都是需要动态增容的，而不是预先分配固定的长度不能再改变，所以字典长度大于存储空间的时候就需要增容了。  
**（1）申请一段更大的内存，同时将原数据拷贝过去**  
由于字典的数据存储方式是一个键值对表以及一个索引表，两个表都是数组即内存连续，这样我们就很容易的直接申请一段新的连续内存，然后将旧的内存的数据拷贝过去即可。  
**（2）重新计算哈希值**  
参考二、2，由于数组的长度改变，所以原来计算的哈希值就可能不正确了，所以需要遍历存在的键值对及索引重新计算哈希值及索引指向。

具体实现如下：

```
static	void acf_map_resize(acf_map* dic, int cap)
{
	if (cap < dic->count)
	{
		return;
	}
	//重新申请空间
	int newSize = getPrime(cap);
	int* newBuckets = malloc(sizeof(int) * newSize);
	acf_map_entry* newEntries = malloc(sizeof(acf_map_entry) * newSize);
	for (int i = 0; i < newSize; i++)
	{
		newBuckets[i] = -1;
		newEntries[i]._next = -1;
		newEntries[i]._hashCode = -1;
	}
	if (dic->count > 0)
	{
		memcpy(newEntries, dic->_entries, dic->count * sizeof(acf_map_entry));
		//重新计算哈希碰撞码
		for (int i = 0; i < dic->capacity; i++) {
			if (newEntries[i]._hashCode >= 0)
			{
				int bucket = newEntries[i]._hashCode % newSize;
				newEntries[i]._next = newBuckets[bucket];
				newBuckets[bucket] = i;
			}
		}
	}
	if (dic->_buckets)
	{
		free(dic->_buckets);
	}
	if (dic->_entries)
	{
		free(dic->_entries);
	}
	dic->_buckets = newBuckets;
	dic->_entries = newEntries;
	dic->capacity = newSize;
}
```

关于数组长度由于上述实现使用的是取余法，参考. net 哈希字典使用的是质数做为长度：

```
const  static int PRIMES[72] = {
	3, 7, 11, 17, 23, 29, 37, 47, 59, 71, 89, 107, 131, 163, 197, 239, 293, 353, 431, 521, 631, 761, 919,
	1103, 1327, 1597, 1931, 2333, 2801, 3371, 4049, 4861, 5839, 7013, 8419, 10103, 12143, 14591,
	17519, 21023, 25229, 30293, 36353, 43627, 52361, 62851, 75431, 90523, 108631, 130363, 156437,
	187751, 225307, 270371, 324449, 389357, 467237, 560689, 672827, 807403, 968897, 1162687, 1395263,
	1674319, 2009191, 2411033, 2893249, 3471899, 4166287, 4999559, 5999471, 7199369 };
//获取最接近的且大于cap的质数
static int getPrime(int cap) {
	for (int i = 0; i < 72; i++)
	{
		if (PRIMES[i] > cap)
		{
			return PRIMES[i];
		}
	}
	return cap;
}
```

**三、源码**  
[https://download.csdn.net/download/u013113678/32903369](https://download.csdn.net/download/u013113678/32903369)

**四、使用例子**  
**1、键为值类型**

```
int main(){
acf_map map1;
//初始化
acf_map_init(&map1);
//插入元素
acf_map_add(&map1,5,110);
acf_map_add(&map1,6,120);
acf_map_add(&map1,7,130);
intprt_t value;
//查找元素
if(acf_map_get(&map1,6,&value)){
printf(“get value %lld”,value);
}
//删除元素
acf_map_remove(&map1,6);
//遍历元素
acf_map_iter i;
acf_map_for_each(i, &map1) {
			printf("%d:%d\n", i.current->key, i.current->value);			
		}
//清除元素
acf_map_clear(&map1);
//反初始化
acf_map_deinit(&map1);
return 0;
}
```

**2、键为结构体**

```
typedef struct{
int a;
int b;
int c;
}A

int main(){
A a,b,c;
a.a=1;
b.a=2;
c.a=3;
acf_map map1;
//初始化
acf_map_init(&map1);
//插入元素
acf_map_add_ptr(&map1,&a,sizeof(A),110);
acf_map_add_ptr(&map1,&a,sizeof(B),120);
acf_map_add_ptr(&map1,&a,sizeof(C),130);
Intprt_t value;
//查找元素
if(acf_map_get_ptr(&map1,&b,sizeof(A),&value)){
printf(“get value %lld”,value);
}
//删除元素
acf_map_remove_ptr(&map1,&b,sizeof(A));
//遍历元素
acf_map_iter i;
acf_map_for_each(i, &map1) {
			printf("%s:%d\n",i.current->key , i.current->value);			
		}
//清除元素
acf_map_clear(&map1);
//反初始化
acf_map_deinit(&map1);
return 0;
}
```

**3、键为字符串**

```
int main(){
char * a=”this is a hash map”;
acf_map map1;
//初始化
acf_map_init(&map1);
//插入元素
acf_map_add_ptr(&map1,a,strlen(a),110);
acf_map_add_ptr(&map1,”hello”,strlen(”hello”),120);
acf_map_add_ptr(&map1,”tommy”,strlen(”tommy”),130);
Intprt_t value;
//查找元素
if(acf_map_get_ptr(&map1,a,strlen(a),&value)){
printf(“get value %lld”,value);
}
//删除元素
acf_map_remove_ptr(&map1,a,strlen(a));
//遍历元素
acf_map_iter i;
acf_map_for_each(i, &map1) {
			printf("%s:%d\n",temp->a , i.current->value);			
		}
//清除元素
acf_map_clear(&map1);
//反初始化
acf_map_deinit(&map1);
return 0;
}
```

**五、参考**  
1、.net Dictionary<TKey,TValue>  
https://referencesource.microsoft.com/#mscorlib/system/collections/generic/dictionary.cs,d3599058f8d79be0  
2.murmur_hash2  
https://blog.csdn.net/xyblog/article/details/50593648