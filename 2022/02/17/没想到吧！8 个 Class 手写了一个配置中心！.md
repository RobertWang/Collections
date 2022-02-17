> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NTY1MjY0MQ==&mid=2650836052&idx=4&sn=078c17b98056dba6b96289c0b3caebfd&chksm=bd01191a8a76900c91358cce1fc7a282466beca6b1c3089a9929eba03ca103455e3d471c03a6&mpshare=1&scene=1&srcid=0127p4A5cdG25e6058olqWEU&sharer_sharetime=1643284739689&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

_________出处：码农参上（ID：CODER_SANJYOU）_________

虽然说功能上有不少差异，但是它们解决的最核心问题，无疑是**配置文件修改后的实时生效**，有时候在搬砖之余 Hydra 就在好奇实时生效是如何实现的、如果让我来设计又会怎么去实现，于是这几天抽出了点空闲时间，摸鱼摸出了个简易版的单机配置中心，先来看看效果：

![](https://mmbiz.qpic.cn/mmbiz_gif/zpom4BeZSicZnD3Ws4Pt0uia9QEqiblCwZycicicdfvIoicznviapo2d9bEhZV6XJiaVpQIjmGicYwS27icnicD9Qqw8KS8jA/640?wx_fmt=gif)

之所以说是简易版本，首先是因为实现的核心功能就只有配置修改后实时生效，并且代码的实现也非常简单，一共只用了 8 个类就实现了这个核心功能，看一下代码的结构，核心类就是`core`包中的这 8 个类：

![](https://mmbiz.qpic.cn/mmbiz_png/zpom4BeZSicZnD3Ws4Pt0uia9QEqiblCwZy0mMB8MSick1N7KMgEibJPF7q2YkmXDbibTHHsTu2l65RdWYdDDLNqjcmw/640?wx_fmt=png)

看到这是不是有点好奇，虽说是低配版，就凭这么几个类也能实现一个配置中心？那么先看一下总体的设计流程，下面我们再细说代码。

![](https://mmbiz.qpic.cn/mmbiz_jpg/zpom4BeZSicZnD3Ws4Pt0uia9QEqiblCwZyzYobicRJ03QicwicwiccBkbVWBlljT9t7lg6KrDAAD0Noj58jBfNaQjptw/640?wx_fmt=jpeg)

代码简要说明
------

下面对 8 个核心类进行一下简要说明并贴出核心代码，有的类中代码比较长，可能对手机浏览的小伙伴不是非常友好，建议收藏后以后电脑浏览器打开（骗波收藏，计划通！）。另外 Hydra 已经把项目的全部代码上传到了`git`，有需要的小伙伴可以移步文末获取地址。

### 1、ScanRunner

`ScanRunner`实现了`CommandLineRunner`接口，可以保证它在 springboot 启动最后执行，这样就能确保其他的 Bean 已经实例化结束并被放入了容器中。至于为什么起名叫`ScanRunner`，是因为这里要实现的主要就是扫描类相关功能。先看一下代码：

真正实现文件扫描功能是调用的`FileScanner`，它的实现我们后面具体再说，在功能上它能够根据文件后缀名扫描某一目录下的全部文件，这里首先扫描出了`target`目录下全部以`.class`结尾的文件：

![](https://mmbiz.qpic.cn/mmbiz_png/zpom4BeZSicZnD3Ws4Pt0uia9QEqiblCwZyerpxGtN39dYjqc81Iahh1wpTheaOkEfwLibf3UgL2bkf0uR2nxcO3Rw/640?wx_fmt=png)

扫描到全部`class`文件后，就可以利用类的全限定名获取到类的`Class`对象，下一步是调用`doFilter`方法对类进行过滤。这里我们暂时仅考虑通过`@Value`注解的方式注入配置文件中属性值的方式，那么下一个问题来了，什么类中的`@Value`注解会生效呢？答案是通过`@Component`、`@Controller`、`@Service`这些注解交给 spring 容器管理的类。

综上，我们通过这些注解再次进行过滤出符合条件的类，找到后交给`VariablePool`对变量进行处理。

### 2、FileScanner

`FileScanner`是扫描文件的工具类，它可以根据文件后缀名筛选出需要的某个类型的文件，除了在`ScanRunner`中用它扫描了 class 文件外，在后面的逻辑中还会用它扫描 yml 文件。下面，看一下`FileScanner`中实现的文件扫描的具体代码：

```
public class FileScanner {
    public static final String TYPE_CLASS=".class";
    public static final String TYPE_YML=".yml";

    public static List<String> findFileByType(String rootPath, List<String> fileList,String fileType){
        if (fileList==null){
            fileList=new ArrayList<>();
        }

        File rootFile=new File(rootPath);
        if (!rootFile.isDirectory()){
            addFile(rootFile.getPath(),fileList,fileType);
        }else{
            String[] subFileList = rootFile.list();
            for (String file : subFileList) {
                String subFilePath=rootPath + "\\" + file;
                File subFile = new File(subFilePath);
                if (!subFile.isDirectory()){
                    addFile(subFile.getPath(),fileList,fileType);
                }else{
                    findFileByType(subFilePath,fileList,fileType);
                }
            }
        }
        return fileList;
    }

    private static void addFile(String fileName,List<String> fileList,String fileType){
        if (fileName.endsWith(fileType)){
            fileList.add(fileName);
        }
    }

    public static String getRealRootPath(String rootPath){
        if (System.getProperty("os.name").startsWith("Windows")
                && rootPath.startsWith("/")){
            rootPath = rootPath.substring(1);
            rootPath = rootPath.replaceAll("/", Matcher.quoteReplacement(File.separator));
        }
        return rootPath;
    }
}
```

查找文件的逻辑很简单，就是在给定的根目录`rootPath`下，循环遍历每一个目录，对找到的文件再进行后缀名的比对，如果符合条件就加到返回的文件名列表中。

至于下面的这个`getRealRootPath`方法，是因为在 windows 环境下，获取到项目的运行目录是这样的：

```
/F:/Workspace/hermit-purple-config/target/classes/
```

而 class 文件名是这样的：

```
F:\Workspace\hermit-purple-config\target\classes\com\cn\hermimt\purple\test\service\UserService.class
```

如果想要获取一个类的全限定名，那么首先要去掉运行目录，再把文件名中的反斜杠`\`替换成点`.`，这里就是为了删掉文件名中的运行路径提前做好准备。

### 3、VariablePool

回到上面的主流程中，每个在`ScanRunner`中扫描出的带有`@Component`、`@Controller`、`@Service`注解的`Class`，都会交给`VariablePool`进行处理。顾名思义，`VariablePool`就是变量池的意思，下面会用这个容器封装所有带`@Value`注解的属性。

```
public class VariablePool {
    public static Map<String, Map<Class,String>> pool=new HashMap<>();
    
    private static final String regex="^(\\$\\{)(.)+(\\})$";
    private static Pattern pattern;
    static{
        pattern=Pattern.compile(regex);
    }

    public static void add(Class clazz){
        Field[] fields = clazz.getDeclaredFields();

        for (Field field : fields) {
            if (field.isAnnotationPresent(Value.class)){
                Value annotation = field.getAnnotation(Value.class);
                String annoValue = annotation.value();
                if (!pattern.matcher(annoValue).matches())
                    continue;

                annoValue=annoValue.replace("${","");
                annoValue=annoValue.substring(0,annoValue.length()-1);

                Map<Class,String> clazzMap = Optional.ofNullable(pool.get(annoValue))
                        .orElse(new HashMap<>());
                clazzMap.put(clazz,field.getName());
                pool.put(annoValue,clazzMap);
            }
        }
    }

    public static Map<String, Map<Class,String>> getPool() {
        return pool;
    }
}
```

简单说一下这块代码的设计思路：

*   通过反射拿到`Class`对象中所有的属性，并判断属性是否加了`@Value`注解
    
*   `@Value`如果要注入配置文件中的值，一定要符合`${xxx}`的格式（这里先暂时不考虑`${xxx:defaultValue}`这种设置了默认值的格式），所以需要使用正则表达式验证是否符合，并校验通过后去掉开头的`${`和结尾的`}`，获取真正对应的配置文件中的字段
    
*   `VariablePool`中声明了一个静态 HashMap，用于存放所有**配置文件中属性 - 类 - 类中属性**的映射关系，接下来就要把这个关系存放到这个`pool`中
    

简单来说，变量池就是下面这样的结构：

![](https://mmbiz.qpic.cn/mmbiz_jpg/zpom4BeZSicZnD3Ws4Pt0uia9QEqiblCwZykdSrRgLxb7NmVrt76dqQhlmv9kHI7jQlbNAhD0BhQicMenWgLqHEaFA/640?wx_fmt=jpeg)

这里如果不好理解的话可以看看例子，我们引入两个测试`Service`：

```
@Service
public class UserService {
    @Value("${person.name}")
    String name;
    @Value("${person.age}")
    Integer age;
}

@Service
public class UserDeptService {
    @Value("${person.name}")
    String pname;
}
```

在所有`Class`执行完`add`方法后，变量池`pool`中的数据是这样的：

![](https://mmbiz.qpic.cn/mmbiz_png/zpom4BeZSicZnD3Ws4Pt0uia9QEqiblCwZyvlfm01JTUdxicvQ8UP5lTnkuiaV4mKtrPria2DZlwKey83zHfqcFx8Bjg/640?wx_fmt=png)

可以看到在`pool`中，`person.name`对应的内层 Map 中包含了两条数据，分别是`UserService`中的`name`字段，以及`UserDeptService`中的`pname`字段。

### 4、EnvInitializer

在`VariablePool`封装完所有变量数据后，`ScanRunner`会调用`EnvInitializer`的`init`方法，开始对 yml 文件进行解析，完成配置中心环境的初始化。其实说白了，这个环境就是一个静态的 HashMap，`key`是属性名，`value`就是属性的值。

```
public class EnvInitializer {
    private static Map<String,Object> envMap=new HashMap<>();

    public static void init(){
        String rootPath = EnvInitializer.class.getResource("/").getPath();
        List<String> fileList = FileScanner.findFileByType(rootPath,null,FileScanner.TYPE_YML);
        for (String ymlFilePath : fileList) {
            rootPath = FileScanner.getRealRootPath(rootPath);
            ymlFilePath = ymlFilePath.replace(rootPath, "");
            YamlMapFactoryBean yamlMapFb = new YamlMapFactoryBean();
            yamlMapFb.setResources(new ClassPathResource(ymlFilePath));
            Map<String, Object> map = yamlMapFb.getObject();
            YamlConverter.doConvert(map,null,envMap);
        }
    }

    public static void setEnvMap(Map<String, Object> envMap) {
        EnvInitializer.envMap = envMap;
    }
    public static Map<String, Object> getEnvMap() {
        return envMap;
    }
}
```

首先还是使用`FileScanner`扫描根目录下所有的`.yml`结尾的文件，并使用 spring 自带的`YamlMapFactoryBean`进行 yml 文件的解析。但是这里有一个问题，所有 yml 文件解析后都会生成一个独立的 Map，需要进行 Map 的合并，生成一份配置信息表。至于这一块具体的操作，都交给了下面的`YamlConverter`进行处理。

我们先进行一下演示，准备两个 yml 文件，配置文件一：`application.yml`

```
spring:
  application:
    name: hermit-purple
server:
  port: 6879
person:
  name: Hydra
  age: 18
```

配置文件二：`config/test.yml`

```
my:
  name: John
  friend:
    name: Jay
    sex: male
run: yeah
```

先来看一看环境完成初始化后，生成的数据格式是这样的：

![](https://mmbiz.qpic.cn/mmbiz_png/zpom4BeZSicZnD3Ws4Pt0uia9QEqiblCwZyoSukwAA3chqV8BkXicY8JUGLLs4e8tgZBcgWQbLycicHPVZ0QhMYeaibA/640?wx_fmt=png)

### 5、YamlConverter

`YamlConverter`主要实现的方法有三个：

*   `doConvert()`：将`EnvInitializer`中提供的多个 Map 合并成一个单层 Map
    
*   `monoToMultiLayer()`：将单层 Map 转换为多层 Map（为了生成 yml 格式字符串）
    
*   `convert()`：yml 格式的字符串解析为 Map（为了判断属性是否发生变化）
    

由于后面两个功能暂时还没有涉及，我们先看第一段代码：

```
public class YamlConverter {
    public static void doConvert(Map<String,Object> map,String parentKey,Map<String,Object> propertiesMap){
        String prefix=(Objects.isNull(parentKey))?"":parentKey+".";
        map.forEach((key,value)->{
            if (value instanceof Map){
                doConvert((Map)value,prefix+key,propertiesMap);
            }else{
                propertiesMap.put(prefix+key,value);
            }
        });
    }
 //...
}    
```

逻辑也很简单，通过循环遍历的方式，将多个 Map 最终都合并到了目的`envMap`中，并且如果遇到多层 Map 嵌套的情况，那么将多层 Map 的 key 通过点`.`进行了连接，最终得到了上面那张图中样式的单层 Map。

其余两个方法，我们在下面使用到的场景再说。

### 6、ConfigController

`ConfigController`作为控制器，用于和前端进行交互，只有两个接口`save`和`get`，下面分别介绍。

#### get

前端页面在开启时会调用`ConfigController`中的`get`接口，填充到`textArea`中。先看一下`get`方法的实现：

```
@GetMapping("get")
public String get(){
    ObjectMapper objectMapper = new ObjectMapper(new YAMLFactory());
    String yamlContent = null;
    try {
        Map<String, Object> envMap = EnvInitializer.getEnvMap();
        Map<String, Object> map = YamlConverter.monoToMultiLayer(envMap, null);
        yamlContent = objectMapper.writeValueAsString(map);
    } catch (Exception e) {
        e.printStackTrace();
    }
    return yamlContent;
}
```

之前在项目启动时，就已经把配置文件属性封装到了`EnvInitializer`的`envMap`中，并且这个`envMap`是一个单层的 Map，不存在嵌套关系。但是我们这里要使用`jackson`生成标准格式的 yml 文档，这种格式不符合要求，需要将它还原成一个具有层级关系的多层 Map，就需要调用`YamlConverter`的`monoToMultiLayer()`方法。

`monoToMultiLayer()`方法的代码有点长，就不贴在这里了，主要是根据 key 中的`.`进行拆分并不断创建子级的 Map，转换完成后得到的多层 Map 数据如下：

![](https://mmbiz.qpic.cn/mmbiz_png/zpom4BeZSicZnD3Ws4Pt0uia9QEqiblCwZyVVSZRNuhJ1XpibJLwcvYLZlcFcFXibBxu7LZ87GFOKpzYma3cm9MdUjQ/640?wx_fmt=png)

在获得这种格式后的 Map 后，就可以调用`jackson`中的方法将 Map 转换为 yml 格式的字符串传递给前端了，看一下处理完成后返回给前端的字符串：

![](https://mmbiz.qpic.cn/mmbiz_png/zpom4BeZSicZnD3Ws4Pt0uia9QEqiblCwZyoqOlrlld3fjAOvk2US0mQZiaXfABh8Jib5iaIPGWMAe4W3DntvLPHxKiaw/640?wx_fmt=png)

#### save

在前端页面修改了 yml 内容后点击保存时，会调用`save`方法保存并更新配置，方法的实现如下：

```
@PostMapping("save")
public String save(@RequestBody Map<String,Object> newValue) {
    String ymlContent =(String) newValue.get("yml");
    PropertyTrigger.change(ymlContent);
    return "success";
}
```

在拿到前端传过来的 yml 字符串后，调用`PropertyTrigger`的`change`方法，实现后续的更改逻辑。

### 7、PropertyTrigger

在调用`change`方法后，主要做的事情有两件：

*   修改`EnvInitializer`中的环境`envMap`，用于前端页面刷新时返回新的数据，以及下一次属性改变时进行对比使用
    
*   修改 bean 中属性的值，这也是整个配置中心最重要的功能
    

先看一下代码：

```
public class PropertyTrigger {
    public static void change(String ymlContent) {
        Map<String, Object> newMap = YamlConverter.convert(ymlContent);
        Map<String, Object> oldMap = EnvInitializer.getEnvMap();

        oldMap.keySet().stream()
                .filter(key->newMap.containsKey(key))
                .filter(key->!newMap.get(key).equals(oldMap.get(key)))
                .forEach(key->{
                    System.out.println(key);
                    Object newVal = newMap.get(key);
                    oldMap.put(key, newVal);
                    doChange(key,newVal);
                });
        EnvInitializer.setEnvMap(oldMap);
    }

    private static void doChange(String propertyName, Object newValue) {
        System.out.println("newValue:"+newValue);
        Map<String, Map<Class, String>> pool = VariablePool.getPool();
        Map<Class, String> classProMap = pool.get(propertyName);

        classProMap.forEach((clazzName,realPropertyName)->{
            try {
                Object bean = SpringContextUtil.getBean(clazzName);
                Field field = clazzName.getDeclaredField(realPropertyName);
                field.setAccessible(true);
                field.set(bean, newValue);
            } catch (NoSuchFieldException | IllegalAccessException e) {
                e.printStackTrace();
            }
        });
    }
}
```

前面铺垫了那么多，其实就是为了实现这段代码中的功能，具体逻辑如下：

*   调用`YamlConverter`的`convert`方法，将前端传来的 yml 格式字符串解析封装成单层 Map，数据格式和`EnvInitializer`中的`envMap`相同
    
*   遍历旧的`envMap`，查看其中的 key 在新的 Map 中对应的属性值是否发生了改变，如果没有改变则不做之后的任何操作
    
*   如果发生改变，用新的值替换`envMap`中的旧值
    
*   通过属性名称，从`VariablePool`中拿到涉及改变的`Class`，以及类中的字段`Field`。并通过后面的`SpringContextUtil`中的方法获取到这个 bean 的实例对象，再通过反射改变字段的值
    
*   将修改后的 Map 写回`EnvInitializer`中的`envMap`
    

到这里，就实现了全部的功能。

### 8、SpringContextUtil

`SpringContextUtil`通过实现`ApplicationContextAware`接口获得了 spring 容器，而通过容器的`getBean()`方法就可以容易的拿到 spring 中的 bean，方便进行后续的更改操作。

```
@Component
public class SpringContextUtil implements ApplicationContextAware {
    private static ApplicationContext applicationContext;
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
    public static <T> T getBean(Class<T> t) {
        return applicationContext.getBean(t);
    }
}
```

### 9、前端代码

至于前端代码，就是一个非常简单的表单，代码的话可以移步`git`查看。

最后
--

到这里全部的代码介绍完了，最后做一个简要的总结吧，虽然通过这几个类能够实现一个简易版的配置中心功能，但是还有不少的缺陷，例如：

*   没有处理`@ConfigurationProperties`注解
    
*   只处理了 yml 文件，没有处理 properties 文件
    
*   目前处理的 bean 都是基于`singleton`模式，如果作用域为`prototype`，也会存在问题
    
*   反射性能低，如果某个属性涉及的类很多会影响性能
    
*   目前只能代码嵌入到项目中使用，还不支持独立部署及远程注册功能
    
*   ……
    

总的来说，后续需要完善的点还有不少，真是感觉任重道远。

最后再聊聊项目的名称，为什么取名叫`hermit-purple`呢，来源是 jojo 中二乔的替身**隐者之紫**，感觉这个替身的能力和配置中心的感知功能还是蛮搭配的，所以就用了这个哈哈。

那么这次的分享就到这里，我是 Hydra，预祝大家虎年春节快乐，我们下篇再见。

> 项目 git 地址：
> 
> https://github.com/trunks2008/hermit-purple-config