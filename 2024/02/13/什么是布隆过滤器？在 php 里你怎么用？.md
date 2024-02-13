> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7249933985562984504) // 创建一个布隆过滤器 $filter = new BloomFilter(); // 向过滤器添加元素 $filter->add("element1"); $filter->add("element2"); $filter->add("element3"); // 检查元素是否存在于过滤器中 if ($filter->has("element1")) { echo "Element 1 may exist."; } else { echo "Element 1 does not exist."; }

2. 自行实现布隆过滤器：
-------------

如果你不想使用第三方扩展库，也可以自行实现布隆过滤器。下面是一个简单的自实现布隆过滤器的示例代码：

```
class BloomFilter {
    private $bitArray;
    private $hashFunctions;

    public function __construct($size, $numHashFunctions) {
        $this->bitArray = array_fill(0, $size, false);
        $this->hashFunctions = $numHashFunctions;
    }

    private function hash($value) {
        $hashes = [];
        $hash1 = crc32($value);
        $hash2 = fnv1a32($value);

        for ($i = 0; $i < $this->hashFunctions; $i++) {
            $hashes[] = ($hash1 + $i * $hash2) % count($this->bitArray);
        }

        return $hashes;
    }

    public function add($value) {
        $hashes = $this->hash($value);

        foreach ($hashes as $hash) {
            $this->bitArray[$hash] = true;
        }
    }

    public function has($value) {
        $hashes = $this->hash($value);

        foreach ($hashes as $hash) {
            if (!$this->bitArray[$hash]) {
                return false;
            }
        }

        return true;
    }
}

// 创建一个布隆过滤器
$filter = new BloomFilter(100, 3);

// 向过滤器添加元素
$filter->add("element1");
$filter->add("element2");
$filter->add("element3");

// 检查元素是否存在于过滤器中
if ($filter->has("element1")) {
    echo "Element 1 may exist.";
} else {
    echo "Element 1 does not exist.";
}



```

无论是使用扩展库还是自行实现，布隆过滤器在处理大规模数据集合时可以提供高效的元素存在性检查功能，适用于需要快速判断元素是否属于某个集合的场景。