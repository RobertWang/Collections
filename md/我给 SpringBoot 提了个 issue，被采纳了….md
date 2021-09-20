> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7008568299361402911) @Configuration(proxyBeanMethods = false) @ConditionalOnClass(RedisOperations.class) @EnableConfigurationProperties(RedisProperties.class) @Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class }) public class RedisAutoConfiguration { @Bean @ConditionalOnMissingBean(name = "redisTemplate") @ConditionalOnSingleCandidate(RedisConnectionFactory.class) public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) { RedisTemplate<Object, Object> template = new RedisTemplate<>(); template.setConnectionFactory(redisConnectionFactory); return template; } } 复制代码

`LettuceConnectionConfiguration` 继承自`RedisConnectionConfiguration`, 核心代码如下

```
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(RedisClient.class)  // -->①
@ConditionalOnProperty(name = "spring.redis.client-type", havingValue = "lettuce", matchIfMissing = true)  // -->②
class LettuceConnectionConfiguration extends RedisConnectionConfiguration {

	LettuceConnectionConfiguration(RedisProperties properties,
			ObjectProvider<RedisSentinelConfiguration> sentinelConfigurationProvider,
			ObjectProvider<RedisClusterConfiguration> clusterConfigurationProvider) {
		super(properties, sentinelConfigurationProvider, clusterConfigurationProvider);
	}

	@Bean
	@ConditionalOnMissingBean(RedisConnectionFactory.class)  // -->③
	LettuceConnectionFactory redisConnectionFactory(
			ObjectProvider<LettuceClientConfigurationBuilderCustomizer> builderCustomizers,
			ClientResources clientResources) {
		LettuceClientConfiguration clientConfig = getLettuceClientConfiguration(builderCustomizers, clientResources,
				getProperties().getLettuce().getPool());
		return createLettuceConnectionFactory(clientConfig);
	}
}
复制代码
```

从中可以看出，Spring boot 自动装配 Lettuce 连接工厂的条件如下

① 存在 `RedisClient` ， `lettuce.io` 中自带的 redis 客户端类

② 项目中使用配置`spring.redis.client-type` 为`lettuce`

③ 项目代码中只要不定义`RedisConnectionFactory` , 便会自动按照配置文件创建 `LettuceConnectionFactory`

其中，包含两处关键，

*   构造函数`LettuceConnectionConfiguration` 出现的`RedisProperties` 和两个`ObjectProvider`，并且调用了父类构造函数
*   `redisConnectionFactory` 中包含两个重要方法`getLettuceClientConfiguration` 和 `createLettuceConnectionFactory`, 其中`getLettuceClientConfiguration` 主要处理 Pool 连接池的相关配置，不做赘述，从下面的分析也可以知道，`properties`其实就是`RedisProperties`，重点看`createLettuceConnectionFactory`

下面，逐个解析这些关键点。

父类构造函数 `RedisConnectionConfiguration`
-------------------------------------

```
protected RedisConnectionConfiguration(RedisProperties properties,
      ObjectProvider<RedisSentinelConfiguration> sentinelConfigurationProvider,
      ObjectProvider<RedisClusterConfiguration> clusterConfigurationProvider) {
   this.properties = properties;
   this.sentinelConfiguration = sentinelConfigurationProvider.getIfAvailable();
   this.clusterConfiguration = clusterConfigurationProvider.getIfAvailable();
}
复制代码
```

理解这段代码的关键是`ObjectProvider`, 其实你如果细心留意，你会发现，Springboot 的代码，特别是构造函数，大量的用到`ObjectProvider`

### `ObjectProvider`

关于`ObjectProvider` , 可以简单聊两句 Spring 4.3 的一些改进

> 当构造方法的参数为单个构造参数时，可以不使用 @Autowired 进行注解

```
@Service
public class FooService {
    private final FooRepository repository;
    public FooService(FooRepository repository) {
        this.repository = repository
    }
}
复制代码
```

比如，上面这段代码是 spring 4.3 之后的版本，不需要`@Autowired` 也可以正常运行。

> 同样是在 Spring 4.3 版本中，不仅隐式的注入了单构造参数的属性，还引入了`ObjectProvider`接口。

```
//A variant of ObjectFactory designed specifically for injection points, allowing for programmatic optionality and lenient not-unique handling.
public interface ObjectProvider<T> extends ObjectFactory<T>, Iterable<T> {
    // ...省略了部分代码
    @Nullable
	T getIfAvailable() throws BeansException;
}
复制代码
```

从源码注释中可以得知，`ObjectProvider`接口是`ObjectFactory`接口的扩展，专门为注入点设计的，可以让注入变得更加宽松和更具有可选项。

其中，由`getIfAvailable()`可见，当待注入参数的 Bean 为空或有多个时，便是`ObjectProvider`发挥作用的时候。

*   如果注入实例为空时，使用`ObjectProvider`则避免了强依赖导致的依赖对象不存在异常
*   如果有多个实例，`ObjectProvider`的方法会根据 Bean 实现的`Ordered`接口或`@Order`注解指定的先后顺序获取一个`Bean`, 从而了提供了一个更加宽松的依赖注入方式

回到，`RedisConnectionConfiguration`这个父类构造函数本身，其实就是实现这样的功能：如果用户提供了`RedisSentinelConfiguration`和 `RedisSentinelConfiguration` , 会在构造函数中加载进来，而`RedisProperties`则比较简单，就是 redis 的相关配置。

### `RedisProperties`

从配置中读取 redis 的相关配置，最简单的单机 redis 配置的是简单的属性，sentinel 是哨兵相关配置，cluster 是集群相关配置，Pool 是连接池的相关配置

```
@ConfigurationProperties(prefix = "spring.redis")
public class RedisProperties {
	private int database = 0;
	private String url;
	private String host = "localhost";
	private int port = 6379;

	private String username;
	private String password;

	private Sentinel sentinel;
	private Cluster cluster;
	public static class Pool {}
	public static class Cluster {}
	public static class Sentinel {}
	// ... 省略非必要代码
}
复制代码
```

小结一下，目前，我们可以看到`RedisAutoConfiguration`依赖于配置类`LettuceConnectionConfiguration`, 其构造函数读取了用户定义的 redis 配置，其中包含 单机配置 + 集群配置 + 哨兵配置 + 连接池配置，其中集群配置和哨兵配置是两个允许用户自定义的 Bean。

`createLettuceConnectionFactory`
--------------------------------

`LettuceConnectionConfiguration`中实现连接池的方法中调用了`createLettuceConnectionFactory`, 其实现如下

```
private LettuceConnectionFactory createLettuceConnectionFactory(LettuceClientConfiguration clientConfiguration) {
		if (getSentinelConfig() != null) {
			return new LettuceConnectionFactory(getSentinelConfig(), clientConfiguration);
		}
		if (getClusterConfiguration() != null) {
			return new LettuceConnectionFactory(getClusterConfiguration(), clientConfiguration);
		}
		return new LettuceConnectionFactory(getStandaloneConfig(), clientConfiguration);
	}
复制代码
```

其实就是依次读取哨兵的配置，集群的配置 以及 单机的配置，如果有就创建连接池返回。

其中`getSentinelConfig()` 和 `getClusterConfiguration()` 是父类的方法，其实现如下，

```
protected final RedisSentinelConfiguration getSentinelConfig() {
   if (this.sentinelConfiguration != null) {
      return this.sentinelConfiguration;
   }
   RedisProperties.Sentinel sentinelProperties = this.properties.getSentinel();
   if (sentinelProperties != null) {
      RedisSentinelConfiguration config = new RedisSentinelConfiguration();
      // 省略装载代码
      config.setDatabase(this.properties.getDatabase());
      return config;
   }
   return null;
}

protected final RedisClusterConfiguration getClusterConfiguration() {
   if (this.clusterConfiguration != null) {
      return this.clusterConfiguration;
   }
   if (this.properties.getCluster() == null) {
      return null;
   }
   RedisProperties.Cluster clusterProperties = this.properties.getCluster();
   RedisClusterConfiguration config = new RedisClusterConfiguration(clusterProperties.getNodes());
   // 省略装载代码
   return config;
}
复制代码
```

从中，我们可以知道，其优先读取在构造函数中由`ObjectProvider`引入的可能存在的用户自定义配置 Bean，如果没有，再通过读取`RedisProperties`完成装配。

但是，细心的读者要问了，How about 单机配置？

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3daa7fcd687843a2bb74e55d3b797d2b~tplv-k3u1fbpfcp-watermark.awebp?)

```
protected final RedisStandaloneConfiguration getStandaloneConfig() {
   RedisStandaloneConfiguration config = new RedisStandaloneConfiguration();
   if (StringUtils.hasText(this.properties.getUrl())) {
      // 省略装载代码
   }
   else {
      // 省略装载代码
   }
   config.setDatabase(this.properties.getDatabase());
   return config;
}
复制代码
```

是的，你没有看错，单身狗不配……

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/68f108fd1fe34db1af64103c0e01b885~tplv-k3u1fbpfcp-watermark.awebp?)

总结起来就是，在构造函数中获取合适的配置 bean，然后在创建连接池的方法里面查找，如果没有就用配置文件构造一个，但是不支持单实例的 redis。

提一个 issue 吧
-----------

保护单身狗，人人有责，于是，我~以 “单身狗保护协会” 的名义~给 SpringBoot 社区提了一个 issue

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/181cb526fda242e494e720ad5a17e512~tplv-k3u1fbpfcp-watermark.awebp?) 然后，大佬回复，~可以保护~可以支持，很开心。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7cc9baf535224e8c90c55aab04bf510e~tplv-k3u1fbpfcp-watermark.awebp?) 其中，有提到使用`BeanPostProcessor`的方法去改写`RedisProperties`的配置，中途我有想到，所以把 issue 关了，沉吟一阵，觉得不优雅，不开心，又把 issue 给打开了，很感谢开源团队的支持和理解，备受鼓舞。