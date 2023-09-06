> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/PUtAsSDfOaPb_rR0SAS4BQ)

        在前面文章中讲了装饰器, 但是都是装饰器作用在同步函数上, 如果是异步函数就会有问题, 因为异步函数需要 async 关键字声明, 同时需要使用 await 在调用, 所以需要让装饰器支持作用在异步函数上  

代码  

  

关于装饰器可以看前面的文章, 链接放文尾, 直接上代码

```
# -*- coding: utf-8 -*-
# @Author: Mehaei
# @Date: 2023-08-27 10:36:16
# @Last Modified by: Mehaei
# @Last Modified time: 2023-08-27 12:33:45
import asyncio
import time
import functools


def exec_time(func):
    """
    统计函数耗费时间函数
    """
    @functools.wraps(func)
    async def async_warp(*args, **kwargs):
        st = time.time()
        result = await func(*args, **kwargs)
        print(f"{func.__name__} spend {time.time() - st}")
        return result

    @functools.wraps(func)
    def sync_warp(*args, **kwargs):
        st = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} spend {time.time() - st}")
        return result
    # 判断函数是否为异步函数
    if asyncio.iscoroutinefunction(func):
        return async_warp
    else:
        return sync_warp


@exec_time
async def async_func():
    """
    异步函数
    """
    await asyncio.sleep(1)
    print("sleep done")


@exec_time
def sync_func():
    """
    同步函数
    """
    time.sleep(1)
    print("sleep done")


if __name__ == "__main__":
    # 调用异步函数
    asyncio.run(async_func())
    # 调用同步函数
    sync_func()

```

结果输出

```
sleep done
async_func spend 1.0007221698760986
sleep done
sync_func spend 1.0049903392791748
[Finished in 2.1s]

```

总结  

  

        其中 async_func 是异步函数, 使用 syncio.run 来执行, sync_func 是一个同步函数, 直接执行即可,  asyncio.iscoroutinefunction 是判断是不是异步函数, 异步函数则使用异步装饰器, 同步函数就使用同步装饰器, 这样就实现了一个既支持同步函数, 又支持异步的一个装饰器