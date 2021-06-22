> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [jitwxs.cn](https://jitwxs.cn/d6d760c4.html)

> 一、前言 在日常开发中，经常会遇到需要在配置文件中，存储 List 或是 Map 这种类型的数据。

[](#一-前言)一、前言
-------------

在日常开发中，经常会遇到需要在配置文件中，存储 `List` 或是 `Map` 这种类型的数据。Spring 原生是支持这种数据类型的，以配置 `List` 类型为例，对于 `.yaml` 文件配置如下：

```
test:
  list:
    - aaa
    - bbb
    - ccc
```

对于 `.properties` 文件配置如下所示：

```
test.list[0]=aaa
test.list[1]=bbb
test.list[2]=ccc
```

当我们想要在程序中使用时候，想当然的使用 `@Value` 注解去读取这个值，就像下面这种写法一样：

```
@Value("${test.list}")
private List<String> testList;
```

你会发现程序直接报错了，报错信息如下：

```
java.lang.IllegalArgumentException: Could not resolve placeholder 'test.list' in value "${test.list}"
```

这个问题也是可以解决的，以我们要配置的 key 为 `test.list` 为例，新建一个 `test` 的配置类，将 `list` 作为该配置类的一个属性：

```
@Configuration
@ConfigurationProperties("test")
public class TestListConfig {
    private List<String> list;

    public List<String> getList() {
        return list;
    }

    public void setList(List<String> list) {
        this.list = list;
    }
}
```

在程序其他地方使用时候。采用自动注入的方式，去获取值：

```
@Autowired
private TestListConfig testListConfig;
```

可以看见，这种方式十分的不方便，最大的问题是配置和代码高耦合了，增加一个配置，还需要对配置类做增减改动。

[](#二-数组怎么样)二、数组怎么样
-------------------

数组？说实话，业务代码写多了，这个 “古老” 的数据结构远远没有 list 用的多，但是它在解决上面这个问题上，出乎异常的好用。

```
test:
  array1: aaa,bbb,ccc
  array2: 111,222,333
  array3: 11.1,22.2,33.3
```

```
@Value("${test.array1}")
private String[] testArray1;

@Value("${test.array2}")
private int[] testArray2;

@Value("${test.array3}")
private double[] testArray3;
```

这样就能够直接使用了，就是这么的简单方便，如果你想要支持不配置 key 程序也能正常运行的话，给它们加上默认值即可：

```
@Value("${test.array1:}")
private String[] testArray1;

@Value("${test.array2:}")
private int[] testArray2;

@Value("${test.array3:}")
private double[] testArray3;
```

仅仅多了一个 `:` 号，冒号后的值表示当 key 不存在时候使用的默认值，使用默认值时数组的 length = 0。

总结下使用数组实现的优缺点：

**优点：**

*   不需要写配置类
*   使用逗号分割，一行配置，即可完成多个数值的注入，配置文件更加精简

**缺点：**

*   业务代码中数组使用很少，基本需要将其转换为 List，去做 `contains`、`foreach` 等操作。

[](#三-替代方法)三、替代方法
-----------------

那么我们有没有办法，在解析 `list`、`map` 这些类型时，像数组一样方便呢？答案是可以的，这就依赖于 `EL` 表达式。

### [](#31-解析-list)3.1 解析 List

以使用 `.yaml` 文件为例，我们只需要在配置文件中，跟配置数组一样去配置：

```
test:
  list: aaa,bbb,ccc
```

在调用时，借助 `EL` 表达式的 `split()` 函数进行切分即可。

```
@Value("#{'${test.list}'.split(',')}")
private List<String> testList;
```

同样，为它加上默认值，避免不配置这个 key 时候程序报错：

```
@Value("#{'${test.list:}'.split(',')}")
private List<String> testList;
```

但是这样有个问题，当不配置该 key 值，默认值会为空串，它的 length = 1（不同于数组，length = 0），这样解析出来 list 的元素个数就不是空了。

![](https://cdn.jsdelivr.net/gh/jitwxs/cdn/blog/posts/202006/20200626125526383.png)

这个问题比较严重，因为它会导致代码中的判空逻辑执行错误。这个问题也是可以解决的，在 `split()` 之前判断下是否为空即可。

```
@Value("#{'${test.list:}'.empty ? null : '${test.list:}'.split(',')}")
private List<String> testList;
```

如上所示，即为最终的版本，它具有数组方式的全部优点，且更容易在业务代码中去应用。

### [](#32-解析-set)3.2 解析 Set

解析 `Set` 和解析 `List` 本质上是相同的，唯一的区别是 Set 会做去重操作。

```
test:
  set: 111,222,333,111
```

```
@Value("#{'${test.set:}'.empty ? null : '${test.set:}'.split(',')}")
private Set<Integer> testSet;
```

### [](#32-解析-map)3.2 解析 Map

解析 Map 的写法如下所示，value 为该 map 的 JSON 格式，注意这里使用的引号：整个 JSON 串使用引号包裹，value 值使用引号包裹。

```
test:
  map1: '{"name": "zhangsan", "sex": "male"}'
  map2: '{"math": "90", "english": "85"}'
```

在程序中，利用 EL 表达式注入：

```
@Value("#{${test.map1}}")
private Map<String,String> map1;

@Value("#{${test.map2}}")
private Map<String,Integer> map2;
```

注意，使用这种方式，必须得在配置文件中配置该 key 及其 value。我在网上找了许多资料，都没找到利用 EL 表达式支持不配置 key/value 的写法。

如果你真的很需要这个功能，就得自己写解析方法了，这里以使用 `fastjson` 进行解析为例：

(1) 自定义解析方法

```
public class MapDecoder {
    public static Map<String, String> decodeMap(String value) {
        try {
            return JSONObject.parseObject(value, new TypeReference<Map<String, String>>(){});
        } catch (Exception e) {
            return null;
        }
    }
}
```

(2) 在程序中指定解析方法

```
@Value("#{T(com.github.jitwxs.demo.MapDecoder).decodeMap('${test.map1:}')}")
private Map<String, String> map1;
@Value("#{T(com.github.jitwxs.demo.MapDecoder).decodeMap('${test.map2:}')}")
private Map<String, String> map2;
```

[](#四-后续)四、后续
-------------

以上就是本文的全部内容，利用 EL 表达式、甚至是自己的解析方法，可以让我们更加方便的配置和使用 Collection 类型的配置文件。

唯一不优雅的地方就是，这样写出来的 `@Value` 的内容都很长，既不美观，也不容易阅读。后续我将为大家介绍下利用 `AST`(抽象语法树) 和自定义注解来更加方便的实现这一功能，敬请期待！