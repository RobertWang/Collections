> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/GQ3Nv1r2WyO3fHTH5FVeLg)

众所周知，Python 不是一种执行效率较高的语言。此外在任何语言中，循环都是一种非常消耗时间的操作。假如任意一种简单的单步操作耗费的时间为 1 个单位，将此操作重复执行上万次，最终耗费的时间也将增长上万倍。

`while` 和 `for` 是 Python 中常用的两种实现循环的关键字，它们的运行效率实际上是有差距的。比如下面的测试代码：

这是一个简单的求和操作，计算从 1 到 n 之间所有自然数的总和。可以看到 `for` 循环相比 `while` 要快 1.5 秒。

其中的差距主要在于两者的机制不同。

```
import timeit


def while_loop(n=100_000_000):
    i = 0
    s = 0
    while i < n:
        s += i
        i += 1
    return s


def for_loop(n=100_000_000):
    s = 0
    for i in range(n):
        s += i
    return s


def for_loop_with_inc(n=100_000_000):
    s = 0
    for i in range(n):
        s += i
        i += 1
    return s


def for_loop_with_test(n=100_000_000):
    s = 0
    for i in range(n):
        if i < n:
            pass
        s += i
    return s


def main():
    print('while loop\t\t', timeit.timeit(while_loop, number=1))
    print('for loop\t\t', timeit.timeit(for_loop, number=1))
    print('for loop with increment\t\t',
          timeit.timeit(for_loop_with_inc, number=1))
    print('for loop with test\t\t', timeit.timeit(for_loop_with_test, number=1))


if __name__ == '__main__':
    main()
# => while loop               4.718853999860585
# => for loop                 3.211570399813354
# => for loop with increment          4.602369500091299
# => for loop with test               4.18337869993411


```

```
import timeit


def while_loop(n=100_000_000):
    i = 0
    s = 0
    while i < n:
        s += i
        i += 1
    return s


def for_loop(n=100_000_000):
    s = 0
    for i in range(n):
        s += i
    return s


def sum_range(n=100_000_000):
    return sum(range(n))


def main():
    print('while loop\t\t', timeit.timeit(while_loop, number=1))
    print('for loop\t\t', timeit.timeit(for_loop, number=1))
    print('sum range\t\t', timeit.timeit(sum_range, number=1))


if __name__ == '__main__':
    main()
# => while loop               4.718853999860585
# => for loop                 3.211570399813354
# => sum range                0.8658821999561042


```

```
import timeit


def while_loop(n=100_000_000):
    i = 0
    s = 0
    while i < n:
        s += i
        i += 1
    return s


def for_loop(n=100_000_000):
    s = 0
    for i in range(n):
        s += i
    return s


def sum_range(n=100_000_000):
    return sum(range(n))


def math_sum(n=100_000_000):
    return (n * (n - 1)) // 2


def main():
    print('while loop\t\t', timeit.timeit(while_loop, number=1))
    print('for loop\t\t', timeit.timeit(for_loop, number=1))
    print('sum range\t\t', timeit.timeit(sum_range, number=1))
    print('math sum\t\t', timeit.timeit(math_sum, number=1))


if __name__ == '__main__':
    main()
# => while loop               4.718853999860585
# => for loop                 3.211570399813354
# => sum range                0.8658821999561042
# => math sum                 2.400018274784088e-06


```

索性直接不要循环，通过数学公式，把上亿次的循环操作变成只有一步操作。效率自然得到了空前的加强。

最后的结论（有点谜语人）：

**实现循环的最快方式**—— —— ——**就是不用循环**

对于 Python 而言，则尽可能地使用内置函数，将循环中的纯 Python 代码降到最低。

当然，内置函数在某些情况下还不是最快的。比如在创建列表的时候，是字面量写法的速度更快。  

#### 参考资料

The Fastest Way to Loop in Python - mCoding  (https://youtu.be/Qgevy75co8c)

> 作者：StarryLand
> 
> https://www.starky.ltd/2021/11/23/the-fastest-way-to-loop-in-python

- EOF -