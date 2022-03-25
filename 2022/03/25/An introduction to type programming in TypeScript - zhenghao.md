> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.zhenghao.io](https://www.zhenghao.io/posts/type-programming)

> Learn to write types using the type language and leverage your existing javascript knowledge to maste......

#typescript

Published on 31 January, 2022Last updated on 01 March, 2022

Learn to write types using the type language and leverage your existing javascript knowledge to master TypeScript quicker

> See discussions on [Hacker News](https://news.ycombinator.com/item?id=30173375)

Types are a complex language of their own[#](https://www.zhenghao.io/posts/type-programming#types-are-a-complex-language-of-their-own)
--------------------------------------------------------------------------------------------------------------------------------------

I used to think of TypeScript as just JavaScript with type annotations sprinkled on top of it. With that mindset, I often found writing correct types tricky and daunting, to a point they got in the way of building the actual applications I wanted to build, and frequently, it led me to reach for `any`. And with `any`, I lose all type safety.

Indeed, types can get really complicated if you let them. After writing TypeScript for a while, it occurred to me that the TypeScript language actually consists of two sub-languages - one is JavaScript, and the other is the type language:

*   for the JavaScript language, the world is made of JavaScript values
*   for the type language, the world is made of types

When we write TypeScript code, we are constantly dancing between these two worlds: we create types in our type world and "summon" them in our JavaScript world using type annotations (or have them implicitly inferred by the compiler); we can go in the other direction too: use TypeScript's [typeof operator](https://www.typescriptlang.org/docs/handbook/2/typeof-types.html#the-typeof-type-operator) on JavaScript variables/properties to retrieve the corresponding types (not the `typeof` operator JavaScript provides to check runtime values' types).

![](https://www.zhenghao.io/art/blog/type-programming/twoworlds.png)

The JavaScript language is very expressive, so is the type language - in fact, the type language is so expressive that it has been proven to be Turing complete.

Here I don't make any value judgment of whether being Turing complete is good or bad, nor do I know if it is even by design or by accident (in fact, often times, Turing-completeness was achieved [by accident](http://beza1e1.tuxen.de/articles/accidentally_turing_complete.html)). My point is the type language itself, as innocuous as it seems, is certainly powerful, highly capable and can perform arbitrary computation at compile time.

When I started to think of the type language in TypeScript as a full-fledged programming language, I realized it even has a few characteristics of a functional programming language:

1.  use recursion instead of iteration
    1.  in [TypeScript 4.5](https://devblogs.microsoft.com/typescript/announcing-typescript-4-5/#tailrec-conditional) we have tail call optimized recursion (to some extent)
2.  types (data) are immutable

In this post, we will learn the type language in TypeScript by comparing it with JavaScript so that you can leverage your existing JavaScript knowledge to master TypeScript quicker.

> This post assumes that readers have some familiarity with JavaScript and TypeScript. And if you want to learn TypeScript from scratch properly, you should start with [The TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html). I am not here to compete with the docs.

Variable declaration[#](https://www.zhenghao.io/posts/type-programming#variable-declaration)
--------------------------------------------------------------------------------------------

In JavaScript, the world is made of JavaScript values, and we declare variables to refer to values using keywords `var`, `const` and `let`. For example:

```
const obj = {name: 'foo'}
```

In the type language, the world is made of types, and we declare type variables using keywords `type` and `interface`. For example:

> A more accurate name for “type variables” is type synonyms or type alias. I use the word "type variables" to draw an analogy to how a JavaScript variable references a value.
> 
> It is not a perfect analogy though, a type alias doesn’t create or introduce a new type—they are only a new name for existing types. But I hope drawing this analogy makes explaining concepts of the type language much easier.

Types and values are very related. A type, at its core, represents the set of possible values and the valid operations that can be done on the values. Sometimes the set is finite, e.g., `type Name = 'foo' | 'bar'`, a lot of times the set is infinite, e.g., `type Age = number`. In TypeScript we integrate types and values and make them work together to ensure that the runtime values match the compile-time types.

### Local variable declaration[#](https://www.zhenghao.io/posts/type-programming#local-variable-declaration)

We talked about how you can create type variables in the type language. However, the type variables have a global scope by default. To create a local type variable, we can use the `infer` keyword in our type language.

Although this particular way of creating scoped variables might seem strange to JavaScript developers, it actually finds its roots in some pure functional programming languages. For example, in Haskell, we can use the `let` keyword with `in` to perform scoped assignments as in `let {assignments} in {expression}`:

**`infer` is useful for caching some intermediate types**

Here is an example:

Without `infer` to create a local type variable `Bar`, we have to calculate `Bar` twice:

Equality comparisons and conditional branching[#](https://www.zhenghao.io/posts/type-programming#equality-comparisons-and-conditional-branching)
------------------------------------------------------------------------------------------------------------------------------------------------

In JavaScript. we can use `===`/`==` with if statement or the conditional (ternary) operator `?` to perform equality check and conditional branching.

In the type language, on the other hand, we use the `extends` keyword for "equality check", and the conditional (ternary) operator `?` for conditional branching too as in:

If `TypeA` is assignable or substitutable to `TypeB`, then we enter the first branch and get the type from `TrueExpression` and assign that to `TypeC` ; otherwise we get the type from `FalseExpression` as a result to `TypeC`.

> The concept of assignability/substitutability is one of the core concepts in TypeScript that deserves a separate post - I wrote [one covering that in detail](https://www.zhenghao.io/posts/type-hierarchy-tree).

A concrete example in JavaScript:

Translate it into the type language:

The `extends` keyword is versatile. It can also apply constraints to generic type parameters. For example:

By adding the generic constraints, `<T extends {name: string}>` we ensure the argument our function takes always consist of a `name` property of the type `string`.

Retrieve types of properties by indexing into object types[#](https://www.zhenghao.io/posts/type-programming#retrieve-types-of-properties-by-indexing-into-object-types)
------------------------------------------------------------------------------------------------------------------------------------------------------------------------

In JavaScript we can access object properties with square brackets e.g. `obj['prop']` or the dot operator e.g., `obj.prop`.

In the type language, we can extract property types with square brackets as well.

This works not just with object types, we can also index the type with tuples and arrays.

Functions[#](https://www.zhenghao.io/posts/type-programming#functions)
----------------------------------------------------------------------

Functions are the main reusable “building blocks” of any JavaScript program. They take some input (some JavaScript values) and return an output (also some JavaScript values). In the type language, we have generics. Generics **parameterize** types like functions **parameterize** value. Therefore, a generic is conceptually similar to a function in JavaScript.

For example, in JavaScript:

For our type language, we have:

**this is still not a perfect analogy though...**

Generics are by no means exactly the same as JavaScript's functions. For one, unlike functions in JavaScript, Generics are not first-class citizens in the type language. That means we cannot pass a generic to another generic like we pass a function to another function as TypeScript doesn't allow [generics as type parameters](https://github.com/microsoft/TypeScript/issues/1213).

### Map and filter[#](https://www.zhenghao.io/posts/type-programming#map-and-filter)

In our type language, types are immutable. If we want to modify a part of a type, we have to transform the existing ones into **new types**. In the type language, the details of iterating over a data structure (i.e. an object type) and applying transformations evenly are abstracted away by [Mapped Types](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html). We can use it to implement operations that are conceptually similar to the map and filter array methods in JavaScript.

In JavaScript, let's say we want to transform an object's properties from numbers to strings:

In the type langauge, the mapping is done using this syntax `[K in keyof T]` where the `keyof` operator gives us property names as a string union type.

In JavaScript, we can filter out the properties of an object based on some critiria. For example, we can filter out all non-string properties:

In our type language, this can be achieved with the `as` operator and the `never` type:

There are a bunch of builtin [utility “functions”](https://www.typescriptlang.org/docs/handbook/utility-types.html) (generics) for transforming types in TypeScript so often times you don't have to re-invent the wheels.

Pattern matching[#](https://www.zhenghao.io/posts/type-programming#pattern-matching)
------------------------------------------------------------------------------------

We can also use the `infer` keyword to perform pattern matching in the type language.

For example, in a JavaScript program, we can use regex to extract a part of a string:

The equivalence in our type language:

Recursion, instead of iteration[#](https://www.zhenghao.io/posts/type-programming#recursion-instead-of-iteration)
-----------------------------------------------------------------------------------------------------------------

Just like many pure functional programming languages out there, in our type language, there is no syntactical construct for for loop to iterate over a list of data. Recursion take the place of loops.

Let's say in JavaScript, we want to write a function to return an array with same item repeated multiple times. Here is one possible way you can do that:

The recurisve solution would be:

How do we write out the equivalence in our type language? Here are logical steps to arrive at one solution:

1.  create a generic type called `FillArray` (remember we talked about that generics in our type language are just like functions?)
    *   `FillArray<Item, N extends number, Array extends Item[] = []>`
2.  Inside the "function body", we need to check if the `length` property on `Array` is already `N` using the `extends` keyword.
    *   if it has reached to `N` (the base case), then we simply return `Array`
    *   if it hasn't reached to `N`, it recurses and added one more `Item` into `Array`

Putting these together, we have:

### Limits for recursion depth[#](https://www.zhenghao.io/posts/type-programming#limits-for-recursion-depth)

Before TypeScript 4.5, the max recursion depth is [45](https://www.typescriptlang.org/play?ts=4.4.4&ssl=3&ssc=10&pln=3&pc=17#code/C4TwDgpgBAShkENgDkA8BJYEC2AaKyUEAHlgHYAmAzlGQK7YBGEATvgCp1gA20J51KAjIgA2gF0oAXigSAfNKiceEUQHJeZAObAAFmsn8IlGoQD8SrrygAuWPAhI0mHPmT5RAOm-Le+F9jicgDcAFCgkFAAQopwiCioagCMavgALACsCuHg0ACCsQ5OiSnpAGwKAPSVUFTACADGANZQAPYAbqwAZtytAO5AA). In TypeScript 4.5, we have tail call optimization, and the limit increased to [999](https://www.typescriptlang.org/play?ts=4.5.4#code/C4TwDgpgBAShkENgDkA8BJYEC2AaKyUEAHlgHYAmAzlGQK7YBGEATvgCp1gA20J51KAjIgA2gF0oAXigSAfNKiceEUQHJeZAObAAFmsn8IlGoQD8SrrygAuWPAhI0mHPmT5RAOm-Le+F9jicgDcAFChoJBQAIKKcIgoqGoAjGr4AJyZchHg0ABCcQ5OSan4yQAMlQoA9NVQVMAIAMYA1lAA9gBurABm3O0A7qFAA).

Avoid type gymnastics in production code[#](https://www.zhenghao.io/posts/type-programming#avoid-type-gymnastics-in-production-code)
------------------------------------------------------------------------------------------------------------------------------------

Sometimes type programming is jokingly referred to as “type gymnastics” when it gets really complex, fancy and far more sophisticated than it needs to be in a typical application. For example:

1.  [simulating a Chinese chess (象棋)](https://github.com/chinese-chess-everywhere/type-chess)
2.  [simulating a Tic Tac Toe game](https://blog.joshuakgoldberg.com/type-system-game-engines/)
3.  [implementing arithmetic](https://itnext.io/implementing-arithmetic-within-typescripts-type-system-a1ef140a6f6f)

They are more like academic exercises, not suitable for production applications because:

1.  they are hard to comprehend, especially with esoteric TypeScript features.
2.  they are hard to debug due to incredibly long and cryptic compiler error messages.
3.  they are slow to compile.

> Just like we have Leetcode for practicing your core programming skills, we have [type-challenges](https://github.com/type-challenges/type-challenges) for practicing your type programming skills.

Closing thoughts[#](https://www.zhenghao.io/posts/type-programming#closing-thoughts)
------------------------------------------------------------------------------------

We have covered a lot in this blog post. The point of this post is not to really teach you TypeScript, rather than to reintroduce the "hidden" type language you might have overlooked ever since you started learning TypeScript.

Type programming is a niche and underdiscussed topic in the TypeScript community, and I don't think there is anything wrong with that - because ultimately adding types is just a means to an end, the end being writing more dependable web applications in JavaScript. Therefore, to me it is totally understandable that people don't often take the time to "properly" study the type language as they would for JavaScript or other programming languages.

Further Reading[#](https://www.zhenghao.io/posts/type-programming#further-reading)
----------------------------------------------------------------------------------

*   [Proof that TypeScript's Type System is Turing Complete](https://gist.github.com/hediet/63f4844acf5ac330804801084f87a6d4)
*   [TypeScript and Turing Completeness](https://itnext.io/typescript-and-turing-completeness-ba8ded8f3de3)