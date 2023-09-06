> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/vzpsmHG0ah_PNo0KvQBYsQ)

> Python 的 functools 模块是标准库中的一个强大工具，提供了一系列函数，用于优化和增强函数式编程的能

Python 的 functools 模块是标准库中的一个强大工具，提供了一系列函数，用于优化和增强函数式编程的能力。这些函数可以帮助我们处理函数、操作装饰器、缓存结果等。本文将介绍 functools 模块中的五个常用函数，包括`partial`、`wraps`、`lru_cache`、`reduce`和`compose`，并提供相关的代码示例，以帮助读者更好地理解和应用这些函数。

一、partial 函数  
`partial`函数用于部分应用一个函数的参数，返回一个新的可调用对象。它允许我们固定一个或多个函数的参数，从而创建一个接受较少参数的新函数。

以下是一个示例代码：

```
from functools import partial

def multiply(a, b):
    return a * b

# 创建一个新函数double，固定第一个参数为2
double = partial(multiply, 2)

result = double(4)
print(result)  # 输出：8


```

在上述示例中，我们使用`partial`函数创建了一个新的函数`double`，该函数固定了`multiply`函数的第一个参数为 2。然后我们调用`double`函数，传入剩余的参数 4，得到结果 8。

二、wraps 函数  
`wraps`函数是一个装饰器，用于更新装饰函数的元数据，如函数名、参数列表等。它可以帮助我们保留原始函数的信息，并避免在使用装饰器后丢失有关函数的重要信息。

以下是一个示例代码：

```
from functools import wraps

def my_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("Before function execution")
        result = func(*args, **kwargs)
        print("After function execution")
        return result
    return wrapper

@my_decorator
def my_function():
    print("Inside my_function")

my_function()


```

在上述示例中，我们定义了一个装饰器`my_decorator`，并在内部使用`wraps`函数来更新`wrapper`函数的元数据。然后我们使用`my_decorator`装饰`my_function`函数，确保在执行装饰后的函数时，原始函数的信息不会丢失。

三、lru_cache 函数  
`lru_cache`函数是一个装饰器，用于缓存函数的结果。它可以帮助我们避免重复计算相同参数的函数结果，从而提高函数的执行效率。

以下是一个示例代码：

```
from functools import lru_cache

@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2:
        return n
    else:
        return fibonacci(n-1) + fibonacci(n-2)

result = fibonacci(10)
print(result)  # 输出：55


```

在上述示例中，我们定义了一个递归函数`fibonacci`来计算斐波那契数列。通过使用`lru_cache`装饰器并指定最大缓存大小，我们可以避免重复计算相同的斐波那契数，提高计算效率。

四、reduce 函数  
`reduce`函数是一个高阶函数，它对一个序列的元素进行累积操作，返回一个单一的值。它需要一个二元函数作为参数，并将该函数应用于序列中的相邻元素，直到将序列减少为单个值。

以下是一个示例代码：

```
from functools import reduce

numbers = [1, 2, 3, 4, 5]

# 使用reduce函数计算列表中所有元素的乘积
product = reduce(lambda x, y: x * y, numbers)
print(product)  # 输出：120


```

在上述示例中，我们使用`reduce`函数和 Lambda 函数计算了列表中所有元素的乘积。

五、compose 函数  
`compose`函数用于组合多个函数，创建一个新函数，该函数按照从右到左的顺序依次调用传入的函数。

```
from functools import reduce

def compose(*functions):
    return reduce(lambda f, g: lambda x: f(g(x)), functions)

def add_one(x):
    return x + 1

def square(x):
    return x * x

# 创建一个新函数，先将输入值加1，然后对结果进行平方
add_one_and_square = compose(square, add_one)

result = add_one_and_square(3)
print(result)  # 输出：16


```

在上述示例中，我们定义了一个`compose`函数，它接受多个函数作为参数，并返回一个新函数。这个新函数按照从右到左的顺序依次调用传入的函数。我们使用`compose`函数创建了一个新函数`add_one_and_square`，它先将输入值加 1，然后对结果进行平方。然后我们调用`add_one_and_square`函数，传入参数 3，得到结果 16。

**总结：  
**functools 模块是 Python 中强大的函数式编程工具，提供了一些有用的函数，如`partial`、`wraps`、`lru_cache`、`reduce`和`compose`。这些函数可以帮助我们更好地处理函数参数、装饰器、结果缓存和函数组合等问题，提高代码的可读性和性能。通过熟练掌握 functools 模块的使用，我们可以更加高效地编写函数式风格的 Python 代码。