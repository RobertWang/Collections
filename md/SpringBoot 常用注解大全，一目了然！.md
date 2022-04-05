> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA4NzQ0Njc4Ng==&mid=2247503931&idx=1&sn=0d7ec4d48e4979990ff52095c74a9239&chksm=903bd456a74c5d404c6af5365059579b2ca6e3a99a624549ce1d7d5c52c72c9635887dd6984f&mpshare=1&scene=1&srcid=0405TMPAjUMdhTYCMKFc0gC2&sharer_sharetime=1649144894149&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

大家好，我说程序汪  

常见注解工作中使用频繁，建议大家学习下

### 一、简介

基于 SpringBoot 平台开发的项目数不胜数，与常规的基于`Spring`开发的项目最大的不同之处，SpringBoot 里面提供了大量的注解用于快速开发，而且非常简单，基本可以做到开箱即用！

那 SpringBoot 为开发者提供了多少注解呢？我们该如何使用？

针对此问题，小编特意对其进行了一番整理，内容如下，个人感觉还是比较清晰的，今天我们就一起来整一整每个注解的含义和用法，以免踩坑！

![](https://mmbiz.qpic.cn/mmbiz_png/tWOhQMr1wdDZrjj79bXaA1l9ZN9lYBeC5xUGnQIAic2dibrNOOlaNicbG6vvKvd7FXOZlFLobsG2LWlT3MTpz6LMA/640?wx_fmt=png)

### 二、注解总结

#### 2.1、SpringMVC 相关注解

*   `@Controller`
    

通常用于修饰`controller`层的组件，由控制器负责将用户发来的`URL`请求转发到对应的服务接口，通常还需要配合注解`@RequestMapping`使用。

*   `@RequestMapping`
    

提供路由信息，负责`URL`到`Controller`中具体函数的映射，当用于方法上时，可以指定请求协议，比如`GET`、`POST`、`PUT`、`DELETE`等等。

*   `@RequestBody`
    

表示请求体的`Content-Type`必须为`application/json`格式的数据，接收到数据之后会自动将数据绑定到`Java`对象上去

*   `@ResponseBody`
    

表示该方法的返回结果直接写入`HTTP response body`中，返回数据的格式为`application/json`

比如，请求参数为`json`格式，返回参数也为`json`格式，示例代码如下：

```
/**
 * 登录服务
 */
@Controller
@RequestMapping("api")
public class LoginController {
 
    /**
     * 登录请求，post请求协议，请求参数数据格式为json
     * @param request
     */
    @RequestMapping(value = "login", method = RequestMethod.POST)
    @ResponseBody
    public ResponseEntity login(@RequestBody UserLoginDTO request){
     //...业务处理
        return new ResponseEntity(HttpStatus.OK);
    }
}
```

*   `@RestController`
    

和`@Controller`一样，用于标注控制层组件，不同的地方在于：它是`@ResponseBody`和`@Controller`的合集，也就是说，在当`@RestController`用在类上时，表示当前类里面所有对外暴露的接口方法，返回数据的格式都为`application/json`，示范代码如下：

```
@RestController
@RequestMapping("api")
public class LoginController {
 
    /**
     * 登录请求，post请求协议，请求参数数据格式为json
     * @param request
     */
    @RequestMapping(value = "login", method = RequestMethod.POST)
    public ResponseEntity login(@RequestBody UserLoginDTO request){
        //...业务处理
        return new ResponseEntity(HttpStatus.OK);
    }
}
```

*   `@RequestParam`
    

用于接收请求参数为表单类型的数据，通常用在方法的参数前面，示范代码如下：

```
/**
 * 登录请求，post请求协议，请求参数数据格式为表单
 */
@RequestMapping(value = "login", method = RequestMethod.POST)
@ResponseBody
public ResponseEntity login(@RequestParam(value = "userName",required = true) String userName,
                            @RequestParam(value = "userPwd",required = true) String userPwd){
    //...业务处理
    return new ResponseEntity(HttpStatus.OK);
}
```

*   `@PathVariable`
    

用于获取请求路径中的参数，通常用于`restful`风格的`api`上，示范代码如下：

```
/**
 * restful风格的参数请求
 * @param id
 */
@RequestMapping(value = "queryProduct/{id}", method = RequestMethod.POST)
@ResponseBody
public ResponseEntity queryProduct(@PathVariable("id") String id){
    //...业务处理
    return new ResponseEntity(HttpStatus.OK);
}
```

*   `@GetMapping`
    

除了`@RequestMapping`可以指定请求方式之外，还有一些其他的注解，可以用于标注接口路径请求，比如`GetMapping`用在方法上时，表示只支持`get`请求方法，等价于`@RequestMapping(value="/get",method=RequestMethod.GET)`

```
@GetMapping("get")
public ResponseEntity get(){
    return new ResponseEntity(HttpStatus.OK);
}
```

*   `@PostMapping`
    

用在方法上，表示只支持`post`方式的请求。

```
@PostMapping("post")
public ResponseEntity post(){
    return new ResponseEntity(HttpStatus.OK);
}
```

*   `@PutMapping`
    

用在方法上，表示只支持`put`方式的请求，通常表示更新某些资源的意思

```
@PutMapping("put")
public ResponseEntity put(){
    return new ResponseEntity(HttpStatus.OK);
}
```

*   `@DeleteMapping`
    

用在方法上，表示只支持`delete`方式的请求，通常表示删除某些资源的意思

```
@DeleteMapping("delete")
public ResponseEntity delete(){
    return new ResponseEntity(HttpStatus.OK);
}
```

#### 2.2、bean 相关注解

*   `@Service`
    

通常用于修饰`service`层的组件，声明一个对象，会将类对象实例化并注入到`bean`容器里面

```
@Service
public class DeptService {
 
 //具体的方法
}
```

*   `@Component`
    

泛指组件，当组件不好归类的时候，可以使用这个注解进行标注，功能类似于于`@Service`

```
@Component
public class DeptService {
 
 //具体的方法
}
```

*   `@Repository`
    

通常用于修饰`dao`层的组件，

`@Repository`注解属于`Spring`里面最先引入的一批注解，它用于将数据访问层 (`DAO`层 ) 的类标识为`Spring Bean`，具体只需将该注解标注在 DAO 类上即可，示例代码如下：

```
@Repository
public interface RoleRepository extends JpaRepository<Role,Long> {

 //具体的方法
}
```

为什么现在使用的很少呢？

主要是因为当我们配置服务启动自动扫描`dao`层包时，`Spring`会自动帮我们创建一个实现类，然后注入到`bean`容器里面。当某些类无法被扫描到时，我们可以显式的在数据持久类上标注`@Repository`注解，`Spring`会自动帮我们声明对象。

*   `@Bean`
    

相当于 xml 中配置 Bean，意思是产生一个 bean 对象，并交给 spring 管理，示例代码如下：

```
@Configuration
public class AppConfig {
 
   //相当于 xml 中配置 Bean
    @Bean
    public Uploader initFileUploader() {
        return new FileUploader();
    }

}
```

*   `@Autowired`
    

自动导入依赖的`bean`对象，默认时按照`byType`方式导入对象，而且导入的对象必须存在，当需要导入的对象并不存在时，我们可以通过配置`required = false`来关闭强制验证。

```
@Autowired
private DeptService deptService;
```

*   `@Resource`
    

也是自动导入依赖的`bean`对象，**由`JDK`提供**，默认是按照`byName`方式导入依赖的对象；而`@Autowired`默认时按照`byType`方式导入对象，当然`@Resource`还可以配置成通过`byType`方式导入对象。

```
/**
 * 通过名称导入（默认通过名称导入依赖对象）
 */
@Resource(name = "deptService")
private DeptService deptService;

/**
 * 通过类型导入
 */
@Resource(type = RoleRepository.class)
private DeptService deptService;
```

*   `@Qualifier`
    

当有多个同一类型的`bean`时，使用`@Autowired`导入会报错，提示当前对象并不是唯一，`Spring`不知道导入哪个依赖，这个时候，我们可以使用`@Qualifier`进行更细粒度的控制，选择其中一个候选者，一般于`@Autowired`搭配使用，示例如下：

```
@Autowired
@Qualifier("deptService")
private DeptService deptService;
```

*   `@Scope`
    

用于生命一个`spring bean`的作用域，作用的范围一共有以下几种：

*   singleton：唯一 bean 实例，Spring 中的 bean 默认都是单例的。
    
*   prototype：每次请求都会创建一个新的 bean 实例，对象多例。
    
*   request：每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP request 内有效。
    
*   session：每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP session 内有效。
    

```
/**
 * 单例对象
 */
@RestController
@Scope("singleton")
public class HelloController {

}
```

#### 2.3、JPA 相关注解

*   `@Entity`和`@Table`
    

表明这是一个实体类，这两个注解一般一块使用，但是如果表名和实体类名相同的话，`@Table`可以省略。

*   `@Id`
    

表示该属性字段对应数据库表中的主键字段。

*   `@Column`
    

表示该属性字段对应的数据库表中的列名，如果字段名与列名相同，则可以省略。

*   `@GeneratedValue`
    

表示主键的生成策略，有四个选项，分别如下：

*   AUTO：表示由程序控制，是默认选项 ，不设置就是这个
    
*   IDENTITY：表示由数据库生成，采用数据库自增长，Oracle 不支持这种方式
    
*   SEQUENCE：表示通过数据库的序列生成主键 ID，MYSQL 不支持
    
*   Table：表示由特定的数据库产生主键，该方式有利于数据库的移植
    
*   `@SequenceGeneretor`
    

用来定义一个生成主键的序列，它需要与`@GeneratedValue`联合使用才有效，以`TB_ROLE`表为例，对应的注解配置如下：

```
@Entity
@Table(name = "TB_ROLE")
@SequenceGenerator(name = "id_seq", sequenceName = "seq_repair",allocationSize = 1)
public class Role implements Serializable {

    private static final long serialVersionUID = 1L;

    /**
     * 主键ID，采用【id_seq】序列函数自增长
     */
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE,generator = "id_seq")
    private Long id;


    /* 角色名称
     */
    @Column(nullable = false)
    private String roleName;

    /**
     * 角色类型
     */
    @Column(nullable = false)
    private String roleType;
}
```

*   `@Transient`
    

表示该属性并非与数据库表的字段进行映射，ORM 框架会将忽略该属性。

```
/**
 * 忽略该属性
 */
@Column(nullable = false)
@Transient
private String lastTime;
```

*   `@Basic(fetch=FetchType.LAZY)`
    

用在某些属性上，可以实现懒加载的效果，也就是当用到这个字段的时候，才会装载这个属性，如果配置成`fetch=FetchType.EAGER`，表示即时加载，也是默认的加载方式！

```
/**
 * 延迟加载该属性
 */
@Column(nullable = false)
@Basic(fetch = FetchType.LAZY)
private String roleType;
```

*   `@JoinColumn`
    

用于标注表与表之间关系的字段，通常与`@OneToOne`、`@OneToMany`搭配使用，例如如下

```
@Entity
@Table(name = "tb_login_log")
public class LoginLog implements Serializable {
 
 /**
  * 查询登录的用户信息
  */
 @OneToOne
    @JoinColumn(name = "user_id")
    private User user;
 
 //...get、set
}
```

*   `@OneToOne`、`@OneToMany`和`@ManyToOne`
    

这三个注解，相当于`hibernate`配置文件中的`一对一`，`一对多`，`多对一`配置，比如下面的客户地址表，通过客户 ID，实现客户信息的查询。

```
@Entity
@Table()
public class AddressEO implements java.io.Serializable {
 
  @ManyToOne(cascade = { CascadeType.ALL })
  @JoinColumn()
  private CustomerEO customer;
 
  //...get、set
}
```

#### 2.4、配置相关注解

*   `@Configuration`
    

表示声明一个 Java 形式的配置类，Spring Boot 提倡基于 Java 的配置，相当于你之前在 xml 中配置 bean，比如声明一个配置类`AppConfig`，然后初始化一个`Uploader`对象。

```
@Configuration
public class AppConfig {

    @Bean
    public Uploader initOSSUploader() {
        return new OSSUploader();
    }

}
```

*   `@EnableAutoConfiguration`
    

`@EnableAutoConfiguration`可以帮助`SpringBoot`应用将所有符合条件的`@Configuration`配置类，全部都加载到当前`SpringBoot`里，并创建对应配置类的`Bean`，并把该`Bean`实体交给`IoC`容器进行管理。

某些场景下，如果我们想要避开某些配置类的扫描（包括避开一些第三方`jar`包下面的配置，可以这样处理。

```
@Configuration
@EnableAutoConfiguration(exclude = { org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration.class})
public class AppConfig {

 //具有业务方法
}
```

*   `@ComponentScan`
    

标注哪些路径下的类需要被`Spring`扫描，用于自动发现和装配一些`Bean`对象，默认配置是扫描当前文件夹下和子目录下的所有类，如果我们想指定扫描某些包路径，可以这样处理。

```
@ComponentScan(basePackages = {"com.xxx.a", "com.xxx.b", "com.xxx.c"})
```

*   `@SpringBootApplication`
    

等价于使用`@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan`这三个注解，通常用于全局启动类上，示例如下：

```
@SpringBootApplication
public class PropertyApplication {

    public static void main(String[] args) {
        SpringApplication.run(PropertyApplication.class, args);
    }
}
```

把`@SpringBootApplication`换成`@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan`这三个注解，一样可以启动成功，`@SpringBootApplication`只是将这三个注解进行了简化！

*   `@EnableTransactionManagement`
    

表示开启事务支持，等同于 xml 配置方式的`<tx:annotation-driven />`

```
@SpringBootApplication
@EnableTransactionManagement`
public class PropertyApplication {

    public static void main(String[] args) {
        SpringApplication.run(PropertyApplication.class, args);
    }
}
```

*   `@Conditional`
    

从 Spring4 开始，可以通过`@Conditional`注解实现按条件装载`bean`对象，目前 Spring Boot 源码中大量扩展了`@Condition`注解，用于实现智能的自动化配置，满足各种使用场景。下面我给大家列举几个常用的注解：

*   `@ConditionalOnBean`：当某个特定的`Bean`存在时，配置生效
    
*   `@ConditionalOnMissingBean`：当某个特定的`Bean`不存在时，配置生效
    
*   `@ConditionalOnClass`：当`Classpath`里存在指定的类，配置生效
    
*   `@ConditionalOnMissingClass`：当`Classpath`里不存在指定的类，配置生效
    
*   `@ConditionalOnExpression`：当给定的`SpEL`表达式计算结果为`true`，配置生效
    
*   `@ConditionalOnProperty`：当指定的配置属性有一个明确的值并匹配，配置生效
    

具体的应用案例如下：

```
@Configuration
public class ConditionalConfig {


    /**
     * 当AppConfig对象存在时，创建一个A对象
     * @return
     */
    @ConditionalOnBean(AppConfig.class)
    @Bean
    public A createA(){
        return new A();
    }

    /**
     * 当AppConfig对象不存在时，创建一个B对象
     * @return
     */
    @ConditionalOnMissingBean(AppConfig.class)
    @Bean
    public B createB(){
        return new B();
    }


    /**
     * 当KafkaTemplate类存在时，创建一个C对象
     * @return
     */
    @ConditionalOnClass(KafkaTemplate.class)
    @Bean
    public C createC(){
        return new C();
    }

    /**
     * 当KafkaTemplate类不存在时，创建一个D对象
     * @return
     */
    @ConditionalOnMissingClass(KafkaTemplate.class)
    @Bean
    public D createD(){
        return new D();
    }


    /**
     * 当enableConfig的配置为true，创建一个E对象
     * @return
     */
    @ConditionalOnExpression("${enableConfig:false}")
    @Bean
    public E createE(){
        return new E();
    }


    /**
     * 当filter.loginFilter的配置为true，创建一个F对象
     * @return
     */
    @ConditionalOnProperty(prefix = "filter",name = "loginFilter",havingValue = "true")
    @Bean
    public F createF(){
        return new F();
    }
}
```

*   `@value`
    

可以在任意 Spring 管理的 Bean 中通过这个注解获取任何来源配置的属性值，比如你在`application.properties`文件里，定义了一个参数变量！

```
config.name=zhangsan
```

在任意的`bean`容器里面，可以通过`@Value`注解注入参数，获取参数变量值。

```
@RestController
public class HelloController {

    @Value("${config.name}")
    private String config;

    @GetMapping("config")
    public String config(){
        return JSON.toJSONString(config);
    }
}
```

*   `@ConfigurationProperties`
    

上面`@Value`在每个类中获取属性配置值的做法，其实是不推荐的。

一般在企业项目开发中，不会使用那么杂乱无章的写法而且维护也麻烦，通常会一次性读取一个 Java 配置类，然后在需要使用的地方直接引用这个类就可以多次访问了，方便维护，示例如下：

首先，在`application.properties`文件里定义好参数变量。

```
config.name=demo_1
config.value=demo_value_1
```

然后，创建一个 Java 配置类，将参数变量注入即可！

```
@Component
@ConfigurationProperties(prefix = "config")
public class Config {

    public String name;

    public String value;

    //...get、set
}
```

最后，在需要使用的地方，通过`ioc`注入`Config`对象即可！

*   `@PropertySource`
    

这个注解是用来读取我们自定义的配置文件的，比如导入`test.properties`和`bussiness.properties`两个配置文件，用法如下：

```
@SpringBootApplication
@PropertySource(value = {"test.properties","bussiness.properties"})
public class PropertyApplication {

    public static void main(String[] args) {
        SpringApplication.run(PropertyApplication.class, args);
    }
}
```

*   `@ImportResource`
    

用来加载 xml 配置文件，比如导入自定义的`aaa.xml`文件，用法如下：

```
@ImportResource(locations = "classpath:aaa.xml")
@SpringBootApplication
public class PropertyApplication {

    public static void main(String[] args) {
        SpringApplication.run(PropertyApplication.class, args);
    }
}
```

#### 2.5、异常处理相关注解

*   `@ControllerAdvice`和`@ExceptionHandler`
    

通常组合使用，用于处理全局异常，示例代码如下：

```
@ControllerAdvice
@Configuration
@Slf4j
public class GlobalExceptionConfig {
 
 private static final Integer GLOBAL_ERROR_CODE = 500;
 
 @ExceptionHandler(value = Exception.class)
    @ResponseBody
    public void exceptionHandler(HttpServletRequest request, HttpServletResponse response, Exception e) throws Exception {
        log.error("【统一异常处理器】", e);
        ResultMsg<Object> resultMsg = new ResultMsg<>();
        resultMsg.setCode(GLOBAL_ERROR_CODE);
        if (e instanceof CommonException) {
           CommonException ex = (CommonException) e;
           if(ex.getErrCode() != 0) {
                resultMsg.setCode(ex.getErrCode());
            }
            resultMsg.setMsg(ex.getErrMsg());
        }else {
            resultMsg.setMsg(CommonErrorMsg.SYSTEM_ERROR.getMessage());
        }
        WebUtil.buildPrintWriter(response, resultMsg);
    }
 
 
}
```

#### 2.6、测试相关注解

*   `@ActiveProfiles`
    

一般作用于测试类上， 用于声明生效的 Spring 配置文件，比如指定`application-dev.properties`配置文件。

*   `@RunWith`和`@SpringBootTest`
    

一般作用于测试类上， 用于单元测试用，示例如下：

```
@ActiveProfiles("dev")
@RunWith(SpringRunner.class)
@SpringBootTest
public class TestJunit {

    @Test
    public void executeTask() {
        //测试...
    }
}
```

### 三、小结

整个篇幅内容比较多，比较干，大家在看的过程中，也没有必要去记住，可以先收藏起来，等到需要用到的时候，再把它拿出来看看！