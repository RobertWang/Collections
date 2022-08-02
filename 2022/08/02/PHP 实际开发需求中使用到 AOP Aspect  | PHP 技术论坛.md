> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [learnku.com](https://learnku.com/articles/50524)

> 前言 什么是 AOP，这里不深入，这里主要是阐述 AOP 中的 Aspect(切面)，主要是以实际开发中使用到 AOP 的一个例子来说明比较有深刻的印象。

前言[#](#df3688)
--------------

什么是 AOP，这里不深入，这里主要是阐述 AOP 中的 Aspect (切面)，主要是以实际开发中使用到 AOP 的一个例子来说明比较有深刻的印象。

例如 ThinkPHP 中的控制器前置操作 `beforeActionList`，又或者中间件，还有自己创建一个基类来实现等等，以上方法在某种意义上也有一些 AOP 的思想（一般来说 AOP 是指将几个类共有的代码，抽取到一个切片中，等到需要时再切入对象中去，从而改变其原有的行为），AOP 像 OOP 一样，只是一种编程范式，AOP 并没有规定说，实现 AOP 协议的代码，要用什么方式去实现。我的理解是 将代码切入到类的指定方法、指定位置上的编程思想就是面向切面的编程，把和主业务无关的事情，放到代码外面去做，对原有代码毫无入侵性。

AOP 可能很多人只是了解过并没有真正使用到实际的项目中，我认为只有在实际项目中去实现才算是有意义的（最好先在个人项目中实践一下）

[ar414](https://www.ar414.com/)

实践[#](#507829)
--------------

### 环境[#](#fa405f)

网关中某个服务使用了 `hyperf 2.0`

### 需求[#](#e6cefb)

有分页的数据中将 `Laravel` 分页器的结果元数据字段转为网关分页规范数据字段

> 这个的需求有很多方法实现，这里我使用了将 提供分页数据的控制器中的方法 统一进行切面处理，这样的做法，对原有代码毫无入侵性，这就是 AOP 的好处了，把和主业务无关的事情，放到代码外面去做

#### Laravel 分页器默认字段[#](#76f8cb)

```
{
   "total": 50,
   "per_page": 15,
   "current_page": 1,
   "last_page": 4,
   "first_page_url": "http://laravel.app?page=1",
   "last_page_url": "http://laravel.app?page=4",
   "next_page_url": "http://laravel.app?page=2",
   "prev_page_url": null,
   "path": "http://laravel.app",
   "from": 1,
   "to": 15,
   "data":[
        {
            // Result Object
        },
        {
            // Result Object
        }
   ]
}

```

#### 网关分页规范字段[#](#83009f)

```
{
   "total": 50,
   "current_page": 1,
   "last_page": 4,
   "data":[
        {
            // Result Object
        },
        {
            // Result Object
        }
   ]
}

```

### 具体实现方法[#](#acfc37)

在 `hyperf 2.0` 中使用 [Aspect 切面](https://hyperf.wiki/2.0/#/zh-cn/aop?id=%E5%AE%9A%E4%B9%89%E5%88%87%E9%9D%A2aspect)  
通过配置要切入的类，这里切入的是面向有分页数据的控制器中的具体方法  
例如 `TestController::getList` 和 `Test1Controller::getList` 这两个方法提供了分页数据

****定义切面 (Aspect)****  
一般而言，我们管切入到指定类指定方法的代码片段称为切面，而切入到哪些类、哪些方法则叫切入点

```
class PageResultAspect extends AbstractAspect
{

  public $classes = [
    TestController::class . '::' . 'getList',
    Test1Controller::class . '::' . 'getList',
  ];

  public $annotations = [];

  public function process(ProceedingJoinPoint $proceedingJoinPoint)
  {
        // 切面切入后，执行对应的方法会由此来负责 
      // $proceedingJoinPoint 为连接点，通过该类的 process() 方法调用原方法并获得结果
      $result = $proceedingJoinPoint->process();

      //将结果进行统一处理
      if(isset($result['first_page_url'])) {
          unset($result['first_page_url']);
          unset($result['last_page_url']);
          unset($result['next_page_url']);
          unset($result['path']);
          unset($result['prev_page_url']);
          unset($result['per_page']);
          unset($result['from']);
          unset($result['to']);
      }

      return $result;
  }

}

```