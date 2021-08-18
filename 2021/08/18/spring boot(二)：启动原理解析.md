> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/xiaoxi/p/7999885.html)

我们开发任何一个 Spring Boot 项目，都会用到如下的启动类

```
1 @SpringBootApplication
2 public class Application {
3     public static void main(String[] args) {
4         SpringApplication.run(Application.class, args);
5     }
6 }
```

从上面代码可以看出，Annotation 定义（@SpringBootApplication）和类定义（SpringApplication.run）最为耀眼，所以要揭开 SpringBoot 的神秘面纱，我们要从这两位开始就可以了。

**一、SpringBootApplication 背后的秘密**

@SpringBootApplication 注解是 Spring Boot 的核心注解，它其实是一个组合注解：

```
1 @Target(ElementType.TYPE)
 2 @Retention(RetentionPolicy.RUNTIME)
 3 @Documented
 4 @Inherited
 5 @SpringBootConfiguration
 6 @EnableAutoConfiguration
 7 @ComponentScan(excludeFilters = {
 8         @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
 9         @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
10 public @interface SpringBootApplication {
11 ...
12 }
```

虽然定义使用了多个 Annotation 进行了原信息标注，但实际上重要的只有三个 Annotation：

*   @Configuration（@SpringBootConfiguration 点开查看发现里面还是应用了 @Configuration）
*   @EnableAutoConfiguration
*   @ComponentScan

即 @SpringBootApplication = (默认属性)@Configuration + @EnableAutoConfiguration + @ComponentScan。

所以，如果我们使用如下的 SpringBoot 启动类，整个 SpringBoot 应用依然可以与之前的启动类功能对等：

```
1 @Configuration
2 @EnableAutoConfiguration
3 @ComponentScan
4 public class Application {
5     public static void main(String[] args) {
6         SpringApplication.run(Application.class, args);
7     }
8 }
```

每次写这 3 个比较累，所以写一个 @SpringBootApplication 方便点。接下来分别介绍这 3 个 Annotation。

**1、@Configuration**

     这里的 @Configuration 对我们来说不陌生，它就是 JavaConfig 形式的 Spring Ioc 容器的配置类使用的那个 @Configuration，SpringBoot 社区推荐使用基于 JavaConfig 的配置形式，所以，这里的启动类标注了 @Configuration 之后，本身其实也是一个 IoC 容器的配置类。

举几个简单例子回顾下，XML 跟 config 配置方式的区别：

**（1）表达形式层面**

基于 XML 配置的方式是这样：

```
1 <?xml version="1.0" encoding="UTF-8"?>
2 <beans xmlns="http://www.springframework.org/schema/beans"
3        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
4        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd"
5        default-lazy-init="true">
6     <!--bean定义-->
7 </beans>
```

而基于 JavaConfig 的配置方式是这样：

```
1 @Configuration
2 public class MockConfiguration{
3     //bean定义
4 }
```

任何一个标注了 @Configuration 的 Java 类定义都是一个 JavaConfig 配置类。

**（2）注册 bean 定义层面**

基于 XML 的配置形式是这样：

```
1 <bean>
2     ...
3 </bean>
```

而基于 JavaConfig 的配置形式是这样的：

```
1 @Configuration
2 public class MockConfiguration{
3     @Bean
4     public MockService mockService(){
5         return new MockServiceImpl();
6     }
7 }
```

任何一个标注了 @Bean 的方法，其返回值将作为一个 bean 定义注册到 Spring 的 IoC 容器，方法名将默认成该 bean 定义的 id。

**（3）表达依赖注入关系层面**

为了表达 bean 与 bean 之间的依赖关系，在 XML 形式中一般是这样：

```
1 <bean>
2     <propery name ="dependencyService" ref="dependencyService" />
3 </bean>
5 <bean></bean>
```

而基于 JavaConfig 的配置形式是这样的：

```
1 @Configuration
 2 public class MockConfiguration{
 3     @Bean
 4     public MockService mockService(){
 5         return new MockServiceImpl(dependencyService());
 6     }
 8     @Bean
 9     public DependencyService dependencyService(){
10         return new DependencyServiceImpl();
11     }
12 }
```

如果一个 bean 的定义依赖其他 bean，则直接调用对应的 JavaConfig 类中依赖 bean 的创建方法就可以了。

@Configuration：提到 @Configuration 就要提到他的搭档 @Bean。使用这两个注解就可以创建一个简单的 spring 配置类，可以用来替代相应的 xml 配置文件。

```
1 <beans> 
2     <bean id = "car"> 
3         <property ></property> 
4     </bean> 
5     <bean id = "wheel"></bean> 
6 </beans>
```

相当于：

```
1 @Configuration 
 2 public class Conf { 
 3     @Bean 
 4     public Car car() { 
 5         Car car = new Car(); 
 6         car.setWheel(wheel()); 
 7         return car; 
 8     }
10     @Bean 
11     public Wheel wheel() { 
12         return new Wheel(); 
13     } 
14 }
```

@Configuration 的注解类标识这个类可以使用 Spring IoC 容器作为 bean 定义的来源。

@Bean 注解告诉 Spring，一个带有 @Bean 的注解方法将返回一个对象，该对象应该被注册为在 Spring 应用程序上下文中的 bean。

**2、@ComponentScan**

     @ComponentScan 这个注解在 Spring 中很重要，它对应 XML 配置中的元素，@ComponentScan 的功能其实就是自动扫描并加载符合条件的组件（比如 @Component 和 @Repository 等）或者 bean 定义，最终将这些 bean 定义加载到 IoC 容器中。

     我们可以通过 basePackages 等属性来细粒度的定制 @ComponentScan 自动扫描的范围，如果不指定，则默认 Spring 框架实现会从声明 @ComponentScan 所在类的 package 进行扫描。

注：所以 SpringBoot 的启动类最好是放在 root package 下，因为默认不指定 basePackages。

**3、@EnableAutoConfiguration**

    个人感觉 @EnableAutoConfiguration 这个 Annotation 最为重要，所以放在最后来解读，大家是否还记得 Spring 框架提供的各种名字为 @Enable 开头的 Annotation 定义？比如 @EnableScheduling、@EnableCaching、@EnableMBeanExport 等，@EnableAutoConfiguration 的理念和做事方式其实一脉相承，简单概括一下就是，**借助 @Import 的支持，收集和注册特定场景相关的 bean 定义。**

*   @EnableScheduling 是通过 @Import 将 Spring 调度框架相关的 bean 定义都加载到 IoC 容器。
*   @EnableMBeanExport 是通过 @Import 将 JMX 相关的 bean 定义加载到 IoC 容器。

而 @EnableAutoConfiguration 也是借助 @Import 的帮助，将所有符合自动配置条件的 bean 定义加载到 IoC 容器，仅此而已！

    @EnableAutoConfiguration 会根据类路径中的 jar 依赖为项目进行自动配置，如：添加了 spring-boot-starter-web 依赖，会自动添加 Tomcat 和 Spring MVC 的依赖，Spring Boot 会对 Tomcat 和 Spring MVC 进行自动配置。

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171207162607144-677920507.png)

 @EnableAutoConfiguration 作为一个复合 Annotation，其自身定义关键信息如下：

```
1 @SuppressWarnings("deprecation")
 2 @Target(ElementType.TYPE)
 3 @Retention(RetentionPolicy.RUNTIME)
 4 @Documented
 5 @Inherited
 6 @AutoConfigurationPackage
 7 @Import(EnableAutoConfigurationImportSelector.class)
 8 public @interface EnableAutoConfiguration {
 9     ...
10 }
```

    其中，最关键的要属 @Import(EnableAutoConfigurationImportSelector.class)，借助 EnableAutoConfigurationImportSelector，@EnableAutoConfiguration 可以帮助 SpringBoot 应用将所有符合条件的 @Configuration 配置都加载到当前 SpringBoot 创建并使用的 IoC 容器。就像一只 “八爪鱼” 一样，借助于 Spring 框架原有的一个工具类：SpringFactoriesLoader 的支持，@EnableAutoConfiguration 可以智能的自动配置功效才得以大功告成！

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171207163759488-1739516792.png)

**自动配置幕后英雄：SpringFactoriesLoader 详解**

SpringFactoriesLoader 属于 Spring 框架私有的一种扩展方案，其主要功能就是从指定的配置文件 META-INF/spring.factories 加载配置。

```
1 public abstract class SpringFactoriesLoader {
 2     //...
 3     public static <T> List<T> loadFactories(Class<T> factoryClass, ClassLoader classLoader) {
 4         ...
 5     }
 8     public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {
 9         ....
10     }
11 }
```

配合 @EnableAutoConfiguration 使用的话，它更多是提供一种配置查找的功能支持，即根据 @EnableAutoConfiguration 的完整类名 org.springframework.boot.autoconfigure.EnableAutoConfiguration 作为查找的 Key，获取对应的一组 @Configuration 类。

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171208161556484-1145877030.jpg)

上图就是从 SpringBoot 的 autoconfigure 依赖包中的 META-INF/spring.factories 配置文件中摘录的一段内容，可以很好地说明问题。

所以，@EnableAutoConfiguration 自动配置的魔法骑士就变成了：**从 classpath 中搜寻所有的 META-INF/spring.factories 配置文件，并将其中 org.springframework.boot.autoconfigure.EnableutoConfiguration 对应的配置项通过反射（Java Refletion）实例化为对应的标注了 @Configuration 的 JavaConfig 形式的 IoC 容器配置类，然后汇总为一个并加载到 IoC 容器。**

**二、深入探索 SpringApplication 执行流程**

SpringApplication 的 run 方法的实现是我们本次旅程的主要线路，该方法的主要流程大体可以归纳如下：

1） 如果我们使用的是 SpringApplication 的静态 run 方法，那么，这个方法里面首先要创建一个 SpringApplication 对象实例，然后调用这个创建好的 SpringApplication 的实例方法。在 SpringApplication 实例初始化的时候，它会提前做几件事情：

*   根据 classpath 里面是否存在某个特征类（org.springframework.web.context.ConfigurableWebApplicationContext）来决定是否应该创建一个为 Web 应用使用的 ApplicationContext 类型。
*   使用 SpringFactoriesLoader 在应用的 classpath 中查找并加载所有可用的 ApplicationContextInitializer。
*   使用 SpringFactoriesLoader 在应用的 classpath 中查找并加载所有可用的 ApplicationListener。
*   推断并设置 main 方法的定义类。

2） SpringApplication 实例初始化完成并且完成设置后，就开始执行 run 方法的逻辑了，方法执行伊始，首先遍历执行所有通过 SpringFactoriesLoader 可以查找到并加载的 SpringApplicationRunListener。调用它们的 started() 方法，告诉这些 SpringApplicationRunListener，“嘿，SpringBoot 应用要开始执行咯！”。

3） 创建并配置当前 Spring Boot 应用将要使用的 Environment（包括配置要使用的 PropertySource 以及 Profile）。

4） 遍历调用所有 SpringApplicationRunListener 的 environmentPrepared() 的方法，告诉他们：“当前 SpringBoot 应用使用的 Environment 准备好了咯！”。

5） 如果 SpringApplication 的 showBanner 属性被设置为 true，则打印 banner。

6） 根据用户是否明确设置了 applicationContextClass 类型以及初始化阶段的推断结果，决定该为当前 SpringBoot 应用创建什么类型的 ApplicationContext 并创建完成，然后根据条件决定是否添加 ShutdownHook，决定是否使用自定义的 BeanNameGenerator，决定是否使用自定义的 ResourceLoader，当然，最重要的，将之前准备好的 Environment 设置给创建好的 ApplicationContext 使用。

7） ApplicationContext 创建好之后，SpringApplication 会再次借助 Spring-FactoriesLoader，查找并加载 classpath 中所有可用的 ApplicationContext-Initializer，然后遍历调用这些 ApplicationContextInitializer 的 initialize（applicationContext）方法来对已经创建好的 ApplicationContext 进行进一步的处理。

8） 遍历调用所有 SpringApplicationRunListener 的 contextPrepared() 方法。

9） 最核心的一步，将之前通过 @EnableAutoConfiguration 获取的所有配置以及其他形式的 IoC 容器配置加载到已经准备完毕的 ApplicationContext。

10） 遍历调用所有 SpringApplicationRunListener 的 contextLoaded() 方法。

11） 调用 ApplicationContext 的 refresh() 方法，完成 IoC 容器可用的最后一道工序。

12） 查找当前 ApplicationContext 中是否注册有 CommandLineRunner，如果有，则遍历执行它们。

13） 正常情况下，遍历执行 SpringApplicationRunListener 的 finished() 方法、（如果整个过程出现异常，则依然调用所有 SpringApplicationRunListener 的 finished() 方法，只不过这种情况下会将异常信息一并传入处理）

去除事件通知点后，整个流程如下：

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171212151321051-993064506.jpg)

本文以调试一个实际的 SpringBoot 启动程序为例，参考流程中主要类类图，来分析其启动逻辑和自动化配置原理。

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171213142128051-351399772.png)

**总览：**    

    上图为 SpringBoot 启动结构图，我们发现启动流程主要分为三个部分，第一部分进行 SpringApplication 的初始化模块，配置一些基本的环境变量、资源、构造器、监听器，第二部分实现了应用具体的启动方案，包括启动流程的监听模块、加载配置环境模块、及核心的创建上下文环境模块，第三部分是自动化配置模块，该模块作为 springboot 自动配置核心，在后面的分析中会详细讨论。在下面的启动程序中我们会串联起结构中的主要功能。

**启动：**

     每个 SpringBoot 程序都有一个主入口，也就是 main 方法，main 里面调用 SpringApplication.run() 启动整个 spring-boot 程序，该方法所在类需要使用 @SpringBootApplication 注解，以及 @ImportResource 注解 (if need)，@SpringBootApplication 包括三个注解，功能如下：

@EnableAutoConfiguration：SpringBoot 根据应用所声明的依赖来对 Spring 框架进行自动配置。

@SpringBootConfiguration(内部为 @Configuration)：被标注的类等于在 spring 的 XML 配置文件中 (applicationContext.xml)，装配所有 bean 事务，提供了一个 spring 的上下文环境。

@ComponentScan：组件扫描，可自动发现和装配 Bean，默认扫描 SpringApplication 的 run 方法里的 Booter.class 所在的包路径下文件，所以最好将该启动类放到根包路径下。

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171213143558363-1466265945.png)

**SpringBoot 启动类**

首先进入 run 方法

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171213144051754-1827098906.png)

run 方法中去创建了一个 SpringApplication 实例，在该构造方法内，我们可以发现其调用了一个初始化的 initialize 方法

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171213144232926-834887500.png)

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171213144259160-129693850.png)

这里主要是为 SpringApplication 对象赋一些初值。构造函数执行完毕后，我们回到 run 方法

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171213144447066-1033381024.png)

该方法中实现了如下几个关键步骤：

1. 创建了应用的监听器 SpringApplicationRunListeners 并开始监听

2. 加载 SpringBoot 配置环境 (ConfigurableEnvironment)，如果是通过 web 容器发布，会加载 StandardEnvironment，其最终也是继承了 ConfigurableEnvironment，类图如下

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171213144808707-1335729370.png)

可以看出，*Environment 最终都实现了 PropertyResolver 接口，我们平时通过 environment 对象获取配置文件中指定 Key 对应的 value 方法时，就是调用了 propertyResolver 接口的 getProperty 方法

3. 配置环境 (Environment) 加入到监听器对象中(SpringApplicationRunListeners)

4. 创建 run 方法的返回对象：ConfigurableApplicationContext(应用配置上下文)，我们可以看一下创建方法：

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171213145353394-1416082242.png)

方法会先获取显式设置的应用上下文 (applicationContextClass)，如果不存在，再加载默认的环境配置（通过是否是 web environment 判断），默认选择 AnnotationConfigApplicationContext 注解上下文（通过扫描所有注解类来加载 bean），最后通过 BeanUtils 实例化上下文对象，并返回。

ConfigurableApplicationContext 类图如下：

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171213154313488-1411301156.png)

主要看其继承的两个方向：

LifeCycle：生命周期类，定义了 start 启动、stop 结束、isRunning 是否运行中等生命周期空值方法

ApplicationContext：应用上下文类，其主要继承了 beanFactory(bean 的工厂类)

5. 回到 run 方法内，prepareContext 方法将 listeners、environment、applicationArguments、banner 等重要组件与上下文对象关联

6. 接下来的 refreshContext(context)方法 (初始化方法如下) 将是实现 spring-boot-starter-*(mybatis、redis 等)自动化配置的关键，包括 spring.factories 的加载，bean 的实例化等核心工作。

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171213154943754-336827902.png)

     配置结束后，Springboot 做了一些基本的收尾工作，返回了应用环境上下文。回顾整体流程，Springboot 的启动，主要创建了配置环境 (environment)、事件监听 (listeners)、应用上下文 (applicationContext)，并基于以上条件，在容器中开始实例化我们需要的 Bean，至此，通过 SpringBoot 启动的程序已经构造完成，接下来我们来探讨自动化配置是如何实现。

**自动化配置：**

之前的启动结构图中，我们注意到无论是应用初始化还是具体的执行过程，都调用了 SpringBoot 自动配置模块。

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171219161544990-1859845219.png)

SpringBoot 自动配置模块

    该配置模块的主要使用到了 SpringFactoriesLoader，即 Spring 工厂加载器，该对象提供了 loadFactoryNames 方法，入参为 factoryClass 和 classLoader，即需要传入上图中的工厂类名称和对应的类加载器，方法会根据指定的 classLoader，加载该类加器搜索路径下的指定文件，即 spring.factories 文件，传入的工厂类为接口，而文件中对应的类则是接口的实现类，或最终作为实现类，所以文件中一般为如下图这种一对多的类名集合，获取到这些实现类的类名后，loadFactoryNames 方法返回类名集合，方法调用方得到这些集合后，再通过反射获取这些类的类对象、构造方法，最终生成实例。

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171219162725615-751500087.png)

工厂接口与其若干实现类接口名称

下图有助于我们形象理解自动配置流程。

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171219162844787-1104034109.png)

SpringBoot 自动化配置关键组件关系图 

    mybatis-spring-boot-starter、spring-boot-starter-web 等组件的 META-INF 文件下均含有 spring.factories 文件，自动配置模块中，SpringFactoriesLoader 收集到文件中的类全名并返回一个类全名的数组，返回的类全名通过反射被实例化，就形成了具体的工厂实例，工厂实例来生成组件具体需要的 bean。

之前我们提到了 EnableAutoConfiguration 注解，其类图如下：

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171219163356100-1697141132.png)

可以发现其最终实现了 ImportSelector(选择器) 和 BeanClassLoaderAware(bean 类加载器中间件)，重点关注一下 AutoConfigurationImportSelector 的 selectImports 方法。

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171219163452318-1362759499.png)

    该方法在 springboot 启动流程——bean 实例化前被执行，返回要实例化的类信息列表。我们知道，如果获取到类信息，spring 自然可以通过类加载器将类加载到 jvm 中，现在我们已经通过 spring-boot 的 starter 依赖方式依赖了我们需要的组件，那么这些组建的类信息在 select 方法中也是可以被获取到的，不要急我们继续向下分析。

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171219164025334-1624354890.png)

该方法中的 getCandidateConfigurations 方法，通过方法注释了解到，其返回一个自动配置类的类名列表，方法调用了 loadFactoryNames 方法，查看该方法

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171219164224912-643901744.png)

    在上面的代码可以看到自动配置器会根据传入的 factoryClass.getName() 到项目系统路径下所有的 spring.factories 文件中找到相应的 key，从而加载里面的类。我们就选取这个 mybatis-spring-boot-autoconfigure 下的 spring.factories 文件

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171219164557240-1466961312.png)

进入 org.mybatis.spring.boot.autoconfigure.MybatisAutoConfiguration 中，主要看一下类头：

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171219164818162-646159475.png)

发现 Spring 的 @Configuration，俨然是一个通过注解标注的 springBean，继续向下看，

@ConditionalOnClass({SqlSessionFactory.class, SqlSessionFactoryBean.class}) 这个注解的意思是：当存在 SqlSessionFactory.class, SqlSessionFactoryBean.class 这两个类时才解析 MybatisAutoConfiguration 配置类，否则不解析这一个配置类，make sence，我们需要 mybatis 为我们返回会话对象，就必须有会话工厂相关类。

@CondtionalOnBean(DataSource.class)：只有处理已经被声明为 bean 的 dataSource。

@ConditionalOnMissingBean(MapperFactoryBean.class) 这个注解的意思是如果容器中不存在 name 指定的 bean 则创建 bean 注入，否则不执行（该类源码较长，篇幅限制不全粘贴）

     以上配置可以保证 sqlSessionFactory、sqlSessionTemplate、dataSource 等 mybatis 所需的组件均可被自动配置，@Configuration 注解已经提供了 Spring 的上下文环境，所以以上组件的配置方式与 Spring 启动时通过 mybatis.xml 文件进行配置起到一个效果。通过分析我们可以发现，只要一个基于 SpringBoot 项目的类路径下存在 SqlSessionFactory.class, SqlSessionFactoryBean.class，并且容器中已经注册了 dataSourceBean，就可以触发自动化配置，意思说我们只要在 maven 的项目中加入了 mybatis 所需要的若干依赖，就可以触发自动配置，但引入 mybatis 原生依赖的话，每集成一个功能都要去修改其自动化配置类，那就得不到开箱即用的效果了。所以 Spring-boot 为我们提供了统一的 starter 可以直接配置好相关的类，触发自动配置所需的依赖 (mybatis) 如下：

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171220160748068-2010633643.png)

这里是截取的 mybatis-spring-boot-starter 的源码中 pom.xml 文件中所有依赖：

![](https://images2017.cnblogs.com/blog/249993/201712/249993-20171220160934240-319846263.png)

 因为 maven 依赖的传递性，我们只要依赖 starter 就可以依赖到所有需要自动配置的类，实现开箱即用的功能。也体现出 Springboot 简化了 Spring 框架带来的大量 XML 配置以及复杂的依赖管理，让开发人员可以更加关注业务逻辑的开发。