> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [davidamos.dev](https://davidamos.dev/three-things-you-might-not-know-about-numbers-in-python/)

> If you’ve written anything in Python, you’ve probably used a number in one of your programs. But ther......

If you've done any coding in Python, there's a good chance that you've used a number in one of your programs. For instance, you may have used an integer to specify the index of a value in a list.

But there's much more to numbers in Python than just their raw values. Let's look at three things about numbers in Python that you might not be aware of.

#1 - Numbers Have Methods
-------------------------

Pretty much everything in Python is an **object**. One of the first objects you learn about in Python is the `str` object for representing strings. Maybe you've seen how strings have **methods**, such as the `.lower()` method, which returns a new string with all lower case characters:

```
>>> "HELLO".lower()
'hello'

```

Numbers in Python are also objects and, just like strings, have their own methods. For instance,  you can convert an integer to a [byte string](https://docs.python.org/3/library/stdtypes.html#bytes) using the [`.to_bytes()` method](https://docs.python.org/3/library/stdtypes.html#int.to_bytes):

```
>>> n = 255
>>> n.to_bytes(length=2, byteorder="big")
b'\x00\xff'

```

The `length` parameter specifies the number of bytes to use in the byte string, and the `byteorder` parameter determines the order of the bytes. For example, setting `byteorder` to `" big"` returns a byte string with the most significant byte first, and setting `byteorder` to `" little"` puts the least significant byte first.

255 is the largest integer that can be represented as an 8-bit integer, so you can set `length=1` in `.to_bytes()` with no problem:

```
>>> n.to_bytes(length=1, byteorder="big")
b'\xff'

```

If you set `length=1` in `.to_bytes()` for 256, however, you get an `OverflowError` :

```
>>> n = 256
>>> n.to_bytes(length=1, byteorder="big")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
OverflowError: int too big to convert


```

You can convert a byte string to an integer using the [`.from_bytes()` class method](https://docs.python.org/3/library/stdtypes.html#int.from_bytes):

```
>>> int.from_bytes(b'\x06\xc1', byteorder="big")
1729


```

**Class methods** are called from a class name instead of a class instance, which is why the `.from_bytes()` method is called on `int` above.

🤓

**Nerd nugget:** 1729 is the smallest positive integer that can be written as a sum of two positive cubes in two different ways — a fact [anectodetly attributed](https://en.wikipedia.org/wiki/1729_(number)) to the Indian mathematician [Srinivasa Ramanujan](https://en.wikipedia.org/wiki/Srinivasa_Ramanujan) who revealed this property to his mentor [G. H. Hardy](https://en.wikipedia.org/wiki/G._H._Hardy):

_"I [Hardy] remember once going to see him [Ramanujan] when he was ill at Putney. I had ridden in taxi cab number 1729 and remarked that the number seemed to me rather a dull one, and that I hoped it was not an unfavourable omen.'No,'he replied,'it is a very interesting number; it is the smallest number expressible as the sum of two cubes in two different ways.'"_

One of the ways you can express 1729 as the sum of two cubes is 1³ + 12³. Can you find the other way?

Floating point numbers also have methods. Perhaps the most useful method for floats is `.is_integer()` which is used to check whether or not a float has no fractional part:

```
>>> n = 2.0
>>> n.is_integer()
True

>>> n = 3.14
>>> n.is_integer()
False


```

One fun float method is the `.as_integer_ratio()` method, which returns a tuple containing the numerator and denominator of the fraction representing the floating point value:

```
>>> n.as_integer_ratio()
(1, 2)


```

Thanks to [floating point representation error](https://docs.python.org/3/tutorial/floatingpoint.html), though, this method can return some unexpected values:

```
>>> n = 0.1
>>> n.as_integer_ratio()
(3602879701896397, 36028797018963968)


```

If you need to, you can call number methods on number literals by surrounding the literals with parentheses:

```
>>> (255).to_bytes(length=1, byteorder="big")
b'\xff'

>>> (3.14).is_integer()
False


```

If you don't surround an integer literal with parentheses, you'll see a `SyntaxError` when you call a method — although, strangely, you don't need the parentheses with floating-point literals:

```
>>> 255.to_bytes(length=1, byteorder="big")
  File "<stdin>", line 1
    255.to_bytes(length=1, byteorder="big")
        ^
SyntaxError: invalid syntax

>>> 3.14.is_integer()
False


```

You can find a complete list of methods available on Python's number types [in the docs](https://docs.python.org/3/library/stdtypes.html#numeric-types-int-float-complex).

#2 - Numbers Have a Hierarchy
-----------------------------

In mathematics, numbers have a natural hierarchy. For example, all natural numbers are integers, all integers are rational numbers, all rational numbers are real numbers, and all real numbers are complex numbers.

The same is true for numbers in Python. This “numeric tower” is expressed through [abstract types](https://docs.python.org/3/glossary.html#term-abstract-base-class) contained in the [`numbers` module](https://docs.python.org/3/library/numbers.html).

### The Numeric Tower

Every number in Python is an instance of the `Number` class:

```
>>> from numbers import Number

>>> # Integers inherit from Number
>>> isinstance(1729, Number)
True

>>> # Floats inherit from Number
>>> isinstance(3.14, Number)
True

>>> # Complex numbers inherit from Number
>>> isinstance(1j, Number)
True


```

If you need to check if a value in Python is numeric, but you don't care what type of number the value is, use `isinstance(value, Number)`.

Python comes with four additional abstract types whose hierarchy, beginning with the most general numeric type, is as follows:

1.  **The `Complex`  class** is used to represent complex numbers. There is one built-in concrete `Complex` type: `complex`.
2.  **The** `**Real**` **class** is used to represent real numbers. There is one built-in concrete `Real` type: `float`.
3.  **The `Rational`  class** is used to represent rational numbers. There is one built-in concrete `Rational` type: `Fraction`.
4.  **The `Integral`  class** is used to represent integers. There are two built-in concrete `Integral` types: `int` and `bool`.

Wait. `bool` values are numbers?! Yes! You can verify all of this in your REPL:

```
>>> import numbers

>>> # Complex numbers inherit from Complex
>>> isinstance(1j, numbers.Complex)
True

>>> # Complex numbers are not Real
>>> isinstance(1j, numbers.Real)
False

>>> # Floats are Real
>>> isinstance(3.14, numbers.Real)
True

>>> # Floats are not Rational
>>> isinstance(3.14, numbers.Rational)
False

>>> # Fractions are Rational
>>> from fractions import Fraction
>>> isinstance(Fraction(1, 2), numbers.Rational)
True

>>> # Fractions are not Integral
>>> isinstance(Fraction(1, 2), numbers.Integral)
False


>>> # Ints are Integral
>>> isinstance(1729, numbers.Integral)
True

>>> # Bools are Integral
>>> isinstance(True, numbers.Integral)
True

>>> True == 1
True

>>> False == 0
True

```

At first glance, everything seems right here — other than `bool` values being numbers, perhaps.

🐍

**Python Peculiarity:** Since the `bool` type is `Integral` — in fact, `bool` inherits directly from `int` — you can do some pretty weird stuff with `True` and `False`.

For instance, you can use `True` as an index to the get the second value of an iterable. If you divide a number by `False` you get a `ZeroDivisionError` error.

Try running `"False"[True]` and `1 / False` in your REPL!

Take a closer look, though, there are a couple of things that are a bit weird about Python's numeric hierarchy.

### Decimals Don't Fit In

There are four concrete numeric types corresponding to the four abstract types in Pythons number tower: `complex`, `float`, `Fraction`, and `int`. But Python has a fifth number type, [the `Decimal` class](https://docs.python.org/3/library/decimal.html), that is used to exactly represent decimal numbers and overcome limitations of floating-point arithmetic.

You might guess that `Decimal` numbers are `Real`, but you'd be wrong:

```
>>> from decimal import Decimal
>>> import numbers

>>> isinstance(Decimal("3.14159"), numbers.Real)
False


```

In fact, the only type that `Decimal` numbers inherit from are Python’s `Number` class:

```
>>> isinstance(Decimal("3.14159"), numbers.Complex)
False

>>> isinstance(Decimal("3.14159"), numbers.Rational)
False

>>> isinstance(Decimal("3.14159"), numbers.Integral)
False

>>> isinstance(Decimal("3.14159"), numbers.Number)
True


```

It makes sense that `Decimal` doesn't inherit from `Integral`. To some extent, it also makes sense that `Decimal` doesn't inherit from `Rational`. But why doesn't `Decimal` inherit from `Real` or `Complex`?

The answer lies in the [CPython source code](https://github.com/python/cpython/blob/9e78dc25179a492550dc602e47e7f4d24e3c89a3/Lib/numbers.py#L24-L30):

> Decimal has all of the methods specified by the `Real` abc, but it should not be registered as a `Real` because decimals do not interoperate with binary floats (i.e.  `Decimal('3.14') + 2.71828` is undefined).  But, abstract reals are expected to interoperate (i.e. R1 + R2 should be expected to work if R1 and R2 are both Reals).

It all boils down to implementation.

### Floats Are Weird

On the one hand, floats implement the `Real` abstract base class and are used to represent real numbers. But thanks to finite memory constraints, floating-point numbers are merely finite approximations of real numbers. This leads to confusing examples, like the following:

```
>>> 0.1 + 0.1 + 0.1 == 0.3
False


```

Floating-point numbers get stored in memory as binary fractions, but this causes some problems. Just like the fraction \(\frac{1}{3}\) has no finite decimal representation — there are infinitely many threes after the decimal point — the fraction \(\frac{1}{10}\) has no finite binary fraction representation.

In other words, you can't store 0.1 on a computer with exact precision — unless that computer has infinite memory.

From a strictly mathematical standpoint, all floating-point numbers are rational — except for `float("inf")` and `float("nan")`. But programmers use them to approximate real numbers and treat them, for the most part, as real numbers.

🐍

**Python Peculiarity:** `float("nan")` is a special floating point value representing "not a number" values — often abbreviated as `NaN` values. But since `float` is a numeric type, `isinstance(float("nan"), Number)` returns `True`.

That's right:"not a number" values are numbers.

Floats are weird.

#3 - Numbers Are Extensible
---------------------------

Python's abstract numeric base types allow you to create your own custom abstract and concrete numeric types.

As an example, consider the following class `ExtendedInteger` which implements numbers of the form \(a + b\sqrt{p}\) where \(a\) and \(b\) are integers and \(p\) is prime (note that primality is not enforced by the class):

```
import math
import numbers

class ExtendedInteger(numbers.Real):
    
    def __init__(self, a, b, p = 2) -> None:
        self.a = a
        self.b = b
        self.p = p
        self._val = a + (b * math.sqrt(p))
    
    def __repr__(self):
        return f"{self.__class__.__name__}({self.a}, {self.b}, {self.p})"
    
    def __str__(self):
        return f"{self.a} + {self.b}√{self.p}"
    
    def __trunc__(self):
        return int(self._val)
    
    def __float__(self):
        return float(self._val)
    
    def __hash__(self):
        return hash(float(self._val))
    
    def __floor__(self):
        return math.floot(self._val)
    
    def __ceil__(self):
        return math.ceil(self._val)
    
    def __round__(self, ndigits=None):
        return round(self._val, ndigits=ndigits)
    
    def __abs__(self):
        return abs(self._val)
    
    def __floordiv__(self, other):
        return self._val // other
    
    def __rfloordiv__(self, other):
        return other // self._val
    
    def __truediv__(self, other):
        return self._val / other
    
    def __rtruediv__(self, other):
        return other / self._val
    
    def __mod__(self, other):
        return self._val % other
        
    def __rmod__(self, other):
        return other % self._val
    
    def __lt__(self, other):
        return self._val < other
    
    def __le__(self, other):
        return self._val <= other
    
    def __eq__(self, other):
        return float(self) == float(other)
    
    def __neg__(self):
        return ExtendedInteger(-self.a, -self.b, self.p)
    
    def __pos__(self):
        return ExtendedInteger(+self.a, +self.b, self.p)
    
    def __add__(self, other):
        if isinstance(other, ExtendedInteger):
            # If both instances have the same p value,
            # return a new ExtendedInteger instance
            if self.p == other.p:
                new_a = self.a + other.a
                new_b = self.b + other.b
                return ExtendedInteger(new_a, new_b, self.p)
            # Otherwise return a float
            else:
                return self._val + other._val
        # If other is integral, add other to self's a value
        elif isinstance(other, numbers.Integral):
            new_a = self.a + other
            return ExtendedInteger(new_a, self.b, self.p)
        # If other is real, return a float
        elif isinstance(other, numbers.Real):
            return self._val + other._val
        # If other is of unknown type, let other determine
        # what to do
        else:
            return NotImplemented
    
    def __radd__(self, other):
        # Addition is commutative so defer to __add__
        return self.__add__(other)
    
    def __mul__(self, other):
        if isinstance(other, ExtendedInteger):
            # If both instances have the same p value,
            # return a new ExtendedInteger instance
            if self.p == other.p:
                new_a = (self.a * other.a) + (self.b * other.b * self.p)
                new_b = (self.a * other.b) + (self.b * other.a)
                return ExtendedInteger(new_a, new_b, self.p)
            # Otherwise, return a float
            else:
                return self._val * other._val
        # If other is integral, multiply self's a and b by other
        elif isinstance(other, numbers.Integral):
            new_a = self.a * other
            new_b = self.b * other
            return ExtendedInteger(new_a, new_b, self.p)
        # If other is real, return a float
        elif isinstance(other, numbers.Real):
            return self._val * other
        # If other is of unknown type, let other determine
        # what to do
        else:
            return NotImplemented
    
    def __rmul__(self, other):
        # Multiplication is commutative so defer to __mul__
        return self.__mul__(other)
    
    def __pow__(self, exponent):
        return self._val ** exponent
    
    def __rpow__(self, base):
        return base ** self._val


```

You need to implement lots of [dunder methods](https://docs.python.org/3/reference/datamodel.html#special-method-names) to ensure the concrete type implements the `Real` interface. You also have to consider how methods like `.__add__()` and `.__mul__()` interact with other `Real` types.

❗

**Note:** By no means is the above example intended to be complete — nor entirely correct, for that matter. Its only purpose in life is to give you a taste of what's possible.

With `ExtendedInteger` implemented, you can now do things like this:

```
>>> a = ExtendedInteger(1, 2)
>>> b = ExtendedInteger(2, 3)

>>> a
ExtendedInteger(1, 2, 2)

>>> # Check that a is a Number
>>> isinstance(a, numbers.Number)
True

>>> # Check that a is Real
>>> isinstance(a, numbers.Real)
True

>>> print(a)
1 + 2√2

>>> a * b
ExtendedInteger(14, 7, 2)

>>> print(a * b)
14 + 7√2

>>> float(a)
3.8284271247461903


```

Python's number hierarchy is quite flexible. But, of course, you should always take great care when implementing types derived from built-in abstract base types. You need to make sure they play well with others.

There are [several tips in the docs](https://docs.python.org/3/library/numbers.html#notes-for-type-implementors) for type implementors that you should read before implementing custom number types. It's also helpful to peruse the [implementation of `Fraction`](https://github.com/python/cpython/blob/main/Lib/fractions.py).

Conclusion
----------

So there you have it. Three things (plus a whole lot more, maybe) that you might not have known about numbers in Python:

1.  Numbers have methods just like pretty much every other object in Python.
2.  Numbers have a hierarchy, even if that hierarchy is abused a bit by `Decimal` and `float`.
3.  You can create your own numbers that fit into the Python number hierarchy.

I hope you learned something new!

_Are you looking to take your Python to the next level? I offer private one-on-one coaching. [Click here](https://davidamos.dev/coaching) for more information._