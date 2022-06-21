> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI4NTM1NDgwNw==&mid=2247519665&idx=1&sn=ac0406c078b9374e1711ac5f4abe5a3a&chksm=ebefb3e9dc983affc6468c0889a64a5b7c5ab9fb877f39abfdff12c0854871866cff571bc37e&mpshare=1&scene=1&srcid=0619GYwYKaP0T60J1pFoLnEq&sharer_sharetime=1655643321220&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

**大家好，我是磊哥。**

一、异步执行
------

实现方式二种：

**1、** 使用异步注解`@aysnc`、`启动类：添加@EnableAsync注解`；

**2、** JDK8 本身有一个非常好用的 Future 类——`CompletableFuture`；

```
@AllArgsConstructorpublic class AskThread implements Runnable{    private CompletableFuture<Integer> re = null;    public void run() {        int myRe = 0;        try {            myRe = re.get() * re.get();        } catch (Exception e) {            e.printStackTrace();        }        System.out.println(myRe);    }    public static void main(String[] args) throws InterruptedException {        final CompletableFuture<Integer> future = new CompletableFuture<>();        new Thread(new AskThread(future)).start();        //模拟长时间的计算过程        Thread.sleep(1000);        //告知完成结果        future.complete(60);    }}
```

在该示例中，启动一个线程，此时 AskThread 对象还没有拿到它需要的数据，执行到 myRe = re.get() * re.get() 会阻塞。我们用休眠 1 秒来模拟一个长时间的计算过程，并将计算结果告诉 future 执行结果，AskThread 线程将会继续执行。

```
public class Calc {    public static Integer calc(Integer para) {        try {            //模拟一个长时间的执行            Thread.sleep(1000);        } catch (InterruptedException e) {            e.printStackTrace();        }        return para * para;    }    public static void main(String[] args) throws ExecutionException, InterruptedException {        final CompletableFuture<Void> future = CompletableFuture.supplyAsync(() -> calc(50))                .thenApply((i) -> Integer.toString(i))                .thenApply((str) -> "\"" + str + "\"")                .thenAccept(System.out::println);        future.get();    }}
```

CompletableFuture.supplyAsync 方法构造一个 CompletableFuture 实例，在 supplyAsync() 方法中，它会在一个新线程中，执行传入的参数。

在这里它会执行 calc() 方法，这个方法可能是比较慢的，但这并不影响 CompletableFuture 实例的构造速度，supplyAsync() 会立即返回。

而返回的 CompletableFuture 实例就可以作为这次调用的契约，在将来任何场合，用于获得最终的计算结果。

supplyAsync 用于提供返回值的情况，CompletableFuture 还有一个不需要返回值的异步调用方法 runAsync(Runnable runnable), 一般我们在优化 Controller 时，使用这个方法比较多。

这两个方法如果在不指定线程池的情况下，都是在 ForkJoinPool.common 线程池中执行，而这个线程池中的所有线程都是 Daemon（守护）线程，所以，当主线程结束时，这些线程无论执行完毕都会退出系统。

核心代码：

```
CompletableFuture.runAsync(() ->   this.afterBetProcessor(betRequest,betDetailResult,appUser,id));
```

**异步调用使用 Callable 来实现**

```
@RestController  public class HelloController {        private static final Logger logger = LoggerFactory.getLogger(HelloController.class);            @Autowired      private HelloService hello;        @GetMapping("/helloworld")      public String helloWorldController() {          return hello.sayHello();      }        /**      * 异步调用restful      * 当controller返回值是Callable的时候，springmvc就会启动一个线程将Callable交给TaskExecutor去处理      * 然后DispatcherServlet还有所有的spring拦截器都退出主线程，然后把response保持打开的状态      * 当Callable执行结束之后，springmvc就会重新启动分配一个request请求，然后DispatcherServlet就重新      * 调用和处理Callable异步执行的返回结果， 然后返回视图      *       * @return      */      @GetMapping("/hello")      public Callable<String> helloController() {          logger.info(Thread.currentThread().getName() + " 进入helloController方法");          Callable<String> callable = new Callable<String>() {                @Override              public String call() throws Exception {                  logger.info(Thread.currentThread().getName() + " 进入call方法");                  String say = hello.sayHello();                  logger.info(Thread.currentThread().getName() + " 从helloService方法返回");                  return say;              }          };          logger.info(Thread.currentThread().getName() + " 从helloController方法返回");          return callable;      }  }
```

**异步调用的方式 WebAsyncTask**

```
@RestController  public class HelloController {        private static final Logger logger = LoggerFactory.getLogger(HelloController.class);            @Autowired      private HelloService hello;            /**      * 带超时时间的异步请求 通过WebAsyncTask自定义客户端超时间      *       * @return      */      @GetMapping("/world")      public WebAsyncTask<String> worldController() {          logger.info(Thread.currentThread().getName() + " 进入helloController方法");            // 3s钟没返回，则认为超时          WebAsyncTask<String> webAsyncTask = new WebAsyncTask<>(3000, new Callable<String>() {                @Override              public String call() throws Exception {                  logger.info(Thread.currentThread().getName() + " 进入call方法");                  String say = hello.sayHello();                  logger.info(Thread.currentThread().getName() + " 从helloService方法返回");                  return say;              }          });          logger.info(Thread.currentThread().getName() + " 从helloController方法返回");            webAsyncTask.onCompletion(new Runnable() {                @Override              public void run() {                  logger.info(Thread.currentThread().getName() + " 执行完毕");              }          });            webAsyncTask.onTimeout(new Callable<String>() {                @Override              public String call() throws Exception {                  logger.info(Thread.currentThread().getName() + " onTimeout");                  // 超时的时候，直接抛异常，让外层统一处理超时异常                  throw new TimeoutException("调用超时");              }          });          return webAsyncTask;      }        /**      * 异步调用，异常处理，详细的处理流程见MyExceptionHandler类      *       * @return      */      @GetMapping("/exception")      public WebAsyncTask<String> exceptionController() {          logger.info(Thread.currentThread().getName() + " 进入helloController方法");          Callable<String> callable = new Callable<String>() {                @Override              public String call() throws Exception {                  logger.info(Thread.currentThread().getName() + " 进入call方法");                  throw new TimeoutException("调用超时!");              }          };          logger.info(Thread.currentThread().getName() + " 从helloController方法返回");          return new WebAsyncTask<>(20000, callable);      }    }
```

二、增加内嵌 Tomcat 的最大连接数
--------------------

```
@Configurationpublic class TomcatConfig {    @Bean    public ConfigurableServletWebServerFactory webServerFactory() {        TomcatServletWebServerFactory tomcatFactory = new TomcatServletWebServerFactory();        tomcatFactory.addConnectorCustomizers(new MyTomcatConnectorCustomizer());        tomcatFactory.setPort(8005);        tomcatFactory.setContextPath("/api-g");        return tomcatFactory;    }    class MyTomcatConnectorCustomizer implements TomcatConnectorCustomizer {        public void customize(Connector connector) {            Http11NioProtocol protocol = (Http11NioProtocol) connector.getProtocolHandler();            //设置最大连接数                           protocol.setMaxConnections(20000);            //设置最大线程数                           protocol.setMaxThreads(2000);            protocol.setConnectionTimeout(30000);        }    }}
```

三、使用 @ComponentScan()
---------------------

定位扫包比 @SpringBootApplication 扫包更快
---------------------------------

四、默认 tomcat 容器改为 Undertow
-------------------------

Jboss 下的服务器，Tomcat 吞吐量 5000，Undertow 吞吐量 8000
---------------------------------------------

```
<exclusions><br data-darkmode-bgcolor-16557772936001="rgb(56, 56, 56)" data-darkmode-original-bgcolor-16557772936001="#fff|rgb(30, 30, 30)" data-darkmode-color-16557772936001="rgb(220, 220, 220)" data-darkmode-original-color-16557772936001="#fff|rgb(220, 220, 220)">        <exclusion><br data-darkmode-bgcolor-16557772936001="rgb(56, 56, 56)" data-darkmode-original-bgcolor-16557772936001="#fff|rgb(30, 30, 30)" data-darkmode-color-16557772936001="rgb(220, 220, 220)" data-darkmode-original-color-16557772936001="#fff|rgb(220, 220, 220)">                <groupId>org.springframework.boot</groupId><br data-darkmode-bgcolor-16557772936001="rgb(56, 56, 56)" data-darkmode-original-bgcolor-16557772936001="#fff|rgb(30, 30, 30)" data-darkmode-color-16557772936001="rgb(220, 220, 220)" data-darkmode-original-color-16557772936001="#fff|rgb(220, 220, 220)">                <artifactId>spring-boot-starter-tomcat</artifactId><br data-darkmode-bgcolor-16557772936001="rgb(56, 56, 56)" data-darkmode-original-bgcolor-16557772936001="#fff|rgb(30, 30, 30)" data-darkmode-color-16557772936001="rgb(220, 220, 220)" data-darkmode-original-color-16557772936001="#fff|rgb(220, 220, 220)">        </exclusion><br data-darkmode-bgcolor-16557772936001="rgb(56, 56, 56)" data-darkmode-original-bgcolor-16557772936001="#fff|rgb(30, 30, 30)" data-darkmode-color-16557772936001="rgb(220, 220, 220)" data-darkmode-original-color-16557772936001="#fff|rgb(220, 220, 220)"></exclusions><br data-darkmode-bgcolor-16557772936001="rgb(56, 56, 56)" data-darkmode-original-bgcolor-16557772936001="#fff|rgb(30, 30, 30)" data-darkmode-color-16557772936001="rgb(220, 220, 220)" data-darkmode-original-color-16557772936001="#fff|rgb(220, 220, 220)"><br data-darkmode-bgcolor-16557772936001="rgb(56, 56, 56)" data-darkmode-original-bgcolor-16557772936001="#fff|rgb(30, 30, 30)" data-darkmode-color-16557772936001="rgb(220, 220, 220)" data-darkmode-original-color-16557772936001="#fff|rgb(220, 220, 220)">
```

改为:

```
<dependency><br data-darkmode-bgcolor-16557772936001="rgb(56, 56, 56)" data-darkmode-original-bgcolor-16557772936001="#fff|rgb(30, 30, 30)" data-darkmode-color-16557772936001="rgb(220, 220, 220)" data-darkmode-original-color-16557772936001="#fff|rgb(220, 220, 220)">        <groupId>org.springframework.boot</groupId><br data-darkmode-bgcolor-16557772936001="rgb(56, 56, 56)" data-darkmode-original-bgcolor-16557772936001="#fff|rgb(30, 30, 30)" data-darkmode-color-16557772936001="rgb(220, 220, 220)" data-darkmode-original-color-16557772936001="#fff|rgb(220, 220, 220)">        <artifactId>spring-boot-starter-undertow</artifactId><br data-darkmode-bgcolor-16557772936001="rgb(56, 56, 56)" data-darkmode-original-bgcolor-16557772936001="#fff|rgb(30, 30, 30)" data-darkmode-color-16557772936001="rgb(220, 220, 220)" data-darkmode-original-color-16557772936001="#fff|rgb(220, 220, 220)"></dependency><br data-darkmode-bgcolor-16557772936001="rgb(56, 56, 56)" data-darkmode-original-bgcolor-16557772936001="#fff|rgb(30, 30, 30)" data-darkmode-color-16557772936001="rgb(220, 220, 220)" data-darkmode-original-color-16557772936001="#fff|rgb(220, 220, 220)"><br data-darkmode-bgcolor-16557772936001="rgb(56, 56, 56)" data-darkmode-original-bgcolor-16557772936001="#fff|rgb(30, 30, 30)" data-darkmode-color-16557772936001="rgb(220, 220, 220)" data-darkmode-original-color-16557772936001="#fff|rgb(220, 220, 220)">
```

五、使用 BufferedWriter 进行缓冲
------------------------

六、Deferred 方式实现异步调用
-------------------

```
@RestControllerpublic class AsyncDeferredController {    private final Logger logger = LoggerFactory.getLogger(this.getClass());    private final LongTimeTask taskService;        @Autowired    public AsyncDeferredController(LongTimeTask taskService) {        this.taskService = taskService;    }        @GetMapping("/deferred")    public DeferredResult<String> executeSlowTask() {        logger.info(Thread.currentThread().getName() + "进入executeSlowTask方法");        DeferredResult<String> deferredResult = new DeferredResult<>();        // 调用长时间执行任务        taskService.execute(deferredResult);        // 当长时间任务中使用deferred.setResult("world");这个方法时，会从长时间任务中返回，继续controller里面的流程        logger.info(Thread.currentThread().getName() + "从executeSlowTask方法返回");        // 超时的回调方法        deferredResult.onTimeout(new Runnable(){     @Override   public void run() {    logger.info(Thread.currentThread().getName() + " onTimeout");    // 返回超时信息    deferredResult.setErrorResult("time out!");   }  });                // 处理完成的回调方法，无论是超时还是处理成功，都会进入这个回调方法        deferredResult.onCompletion(new Runnable(){     @Override   public void run() {    logger.info(Thread.currentThread().getName() + " onCompletion");   }  });                return deferredResult;    }}
```

七、异步调用可以使用 AsyncHandlerInterceptor 进行拦截
---------------------------------------

```
@Componentpublic class MyAsyncHandlerInterceptor implements AsyncHandlerInterceptor {  private static final Logger logger = LoggerFactory.getLogger(MyAsyncHandlerInterceptor.class);  @Override public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)   throws Exception {  return true; }  @Override public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,   ModelAndView modelAndView) throws Exception {//  HandlerMethod handlerMethod = (HandlerMethod) handler;  logger.info(Thread.currentThread().getName()+ "服务调用完成，返回结果给客户端"); }  @Override public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)   throws Exception {  if(null != ex){   System.out.println("发生异常:"+ex.getMessage());  } }  @Override public void afterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response, Object handler)   throws Exception {    // 拦截之后，重新写回数据，将原来的hello world换成如下字符串  String resp = "my name is chhliu!";  response.setContentLength(resp.length());  response.getOutputStream().write(resp.getBytes());    logger.info(Thread.currentThread().getName() + " 进入afterConcurrentHandlingStarted方法"); } }
```