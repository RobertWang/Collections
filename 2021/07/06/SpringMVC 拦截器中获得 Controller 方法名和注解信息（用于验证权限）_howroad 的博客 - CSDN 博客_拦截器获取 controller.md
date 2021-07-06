> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/howroad/article/details/80220320)

在使用 SpringMVC 进行项目的时候用到了权限验证。  
表分为:  
用户表；  
角色表；  
资源表。  
用户 - 角色 - 资源都是多对多的关系，验证无非就是收到请求后，在拦截器循环判断用户是否有权限执行操作。

### 方法一：通过 request 获得用户的 URI，再逐一循环判断是否可以操作。只是这种方法很让人难受。

### 方法二：通过用户要访问的方法来判断是否有权限：

preHandle 方法中 handler 实际为 HandlerMethod，（看网上说的有时候不是 HandlerMethod），加个 instanceof 验证吧  
可以得到方法名：h.getMethod().getName()  
可以得到 RequestMapping 注解中的值：h.getMethodAnnotation(RequestMapping.class)  
这种方法还是不太方便

### 方法三：自定义注解

##### 自定义注解代码：

```
@Retention(RUNTIME)
@Target(METHOD)
public @interface MyOperation {
    String value() default "";//默认为空，因为名字是value，实际操作中可以不写"value="
}

```

##### Controller 代码：

```
@Controller("testController")
public class TestController {
    @MyOperation("用户修改")//主要看这里
    @RequestMapping("test")
    @ResponseBody
    public String test(String id) {
        return "Hello,2018!"+id;
    }
}

```

##### 拦截器的代码：

```
@Override
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
        throws Exception {
    System.out.println("进入拦截器");
    if(handler instanceof HandlerMethod) {
        HandlerMethod h = (HandlerMethod)handler;
        System.out.println("用户想执行的操作是:"+h.getMethodAnnotation(MyOperation.class).value());
        //判断后执行操作...
    }
    return HandlerInterceptor.super.preHandle(request, response, handler);
}

```

//2018-05-16 注:

##### 在每个方法上面加注解太麻烦啦, 可以在类上加注解

```
@Retention(RUNTIME)
@Target(TYPE)
public @interface MyOperation {
    String value() default "";
}
//拦截器中这样获得
h.getMethod().getDeclaringClass().getAnnotation(MyOperation.class);

```

#### 我可以获取 requestMapping, 不用创建自定义注解啊, 值得注意的是, 不要使用 GetMapping 等, 要使用 requestMapping