> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [deepu.tech](https://deepu.tech/first-impression-of-rust/)

> This is what I thought about Rust while learning it. From a pragmatic perspective.

So I started learning Rust a while ago and since my post about what [I thought of Go](https://deepu.tech/reflection-on-golang/) was popular, I decided to write about what my first impressions of Rust were as well.

But unlike Go, I actually didn’t build any real-world application in Rust, so my opinions here are purely personal and some might not be accurate as I might have misunderstood something. So do give me the consideration of a Rust newbie. If you find something I said here is inaccurate please do let me know. Also, my impressions might actually change once I start using the language more. If it does, I’ll make sure to update the post.

As I have said in some of my other posts, I consider myself to be more pragmatic than my younger self now. Some of the opinions are also from that pragmatic perspective(or at least I think so). I have weighed practicality, readability, and simplicity over fancy features, syntax sugars, and complexity. Also, some things which I didn’t like but found not such a big deal are put under nitpicks rather than the dislike section as I thought it was fairer that way.

One thing that sets Rust apart from languages like Go is that Rust is not garbage collected, and I understand that many of the language features/choices where designed with that in mind.

Rust is primarily geared towards procedural/imperative style of programming but it also lets you do a little bit of functional and object-oriented style of programming as well. And that is my favorite kind of mix.

So, without any further ado, let’s get into it.

* * *

Things that I really liked, in no particular order.

No Garbage collection
---------------------

One of the first things you would notice in Rust, especially if you are coming from garbage collected languages like Java or Golang is the lack of garbage collection. Yes, there is no GC in Rust, then how does it ensure my program runs efficiently in the given memory and how does it prevent out of memory errors?

Rust has something called [ownership](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html), so basically any value in Rust must have a variable as its owner(and only one owner at a time) when the owner goes out of scope the value will be dropped freeing the memory regardless of it being in stack or heap memory. For example, in the below example the value of `foo` is dropped as soon as the method execution completes and the value of `bar` is dropped right after the block execution.

```
fn main() {
    let foo = "value"; // owner is foo and is valid within this method

    {
        let bar = "bar value"; // owner is bar and is valid within this block scope
        println!("value of bar is {}", bar); // bar is valid here
    }

    println!("value of foo is {}", foo); // foo is valid here
    println!("value of bar is {}", bar); // bar is not valid here as its out of scope
}
```

So by scoping variables carefully, we can make sure the memory usage is optimized and that is also why Rust lets you use block scopes almost everywhere.

Also, the Rust compiler helps you in dealing with duplicate pointer references and so on. The below is invalid in Rust since `foo` is now using heap memory rather than stack and assigning a reference to a variable is considered a move. If deep copying(expensive) is required it has to be performed using the `clone` function which performs a copy instead of move.

```
fn main() {
    let foo = String::from("hello"); // owner is foo and is valid within this method

    {
        let bar = foo; // owner is bar and is valid within this block scope, foo in invalidated now
        println!("value of bar is {}", bar); // bar is valid here
    }

    println!("value of foo is {}", foo); // foo is invalid here as it has moved
}
```

The ownership concept can be a bit weird to get used to, especially since passing a variable to a method will also move it(if its not a literal or reference), but given that it saves us from GC I think its worth it and the compiler takes care of helping us when we make mistakes.

Immutable by default
--------------------

Variables are immutable by default. If you want to mutate a variable you have to specifically mark it using the `mut` keyword.

```
let foo = "hello" // immutable
let mut bar = "hello" // mutable
```

Variables are by default passed by value, in order to pass a reference we would have to use the `&` symbol. Quite similar to Golang.

```
fn main() {
    let world = String::from("world");

    hello_ref(&world); // pass by reference. Keeps ownership
    // prints: Hello world
    hello_val(world); // pass by value and hence transfer ownership
    // prints: Hello world
}

fn hello_val(msg: String) {
    println!("Hello {}", msg);
}

fn hello_ref(msg: &String) {
    println!("Hello {}", msg);
}
```

When you pass a reference it is still immutable so we would have to explicitly mark that mutable as well as below. This makes accidental mutations very difficult. The compiler also ensures that we can only have one mutable reference in a scope.

```
fn main() {
    let mut world = String::from("world");

    hello_ref(&mut world); // pass by mutable reference. Keeps ownership
    // prints: Hello world!
    hello_val(world); // pass by value and hence transfer ownership
    // prints: Hello world!
}

fn hello_val(msg: String) {
    println!("Hello {}", msg);
}

fn hello_ref(msg: &mut String) {
    msg.push_str("!"); // mutate string
    println!("Hello {}", msg);
}
```

Pattern matching
----------------

Rust has first-class support for pattern matching and this can be used for control flow, error handling, variable assignment and so on. pattern matching can also be used in `if`, `while` statements, `for` loops and function parameters.

```
fn main() {
    let foo = String::from("200");

    let num: u32 = match foo.parse() {
        Ok(num) => num,
        Err(_) => {
            panic!("Cannot parse!");
        }
    };

    match num {
        200 => println!("two hundred"),
        _ => (),
    }
}
```

Generics
--------

One thing I love in Java and TypeScript is the generics. It makes static typing more practical and DRY. Strongly typed languages(Looking at you Golang) without generics are annoying to work with. Fortunately, Rust has great support for generics. It can be used in types, functions, structs, and enums. The icing on the cake is that Rust converts the Generic code using specific code during compile time thus there is no performance penalty in using them.

```
struct Point<T> {
    x: T,
    y: T,
}

fn hello<T>(val: T) -> T {
    return val;
}

fn main() {
    let foo = hello(5);
    let foo = hello("5");

    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```

Static types and advanced type declarations
-------------------------------------------

Rust is a strictly typed language with a static type system. It also has great type inference which means we don’t have to define types manually for everything. Rust also allows for complex type definitions.

```
type Kilometers = i32;

type Thunk = Box<dyn Fn() + Send + 'static>;

type Result<T> = std::result::Result<T, std::io::Error>;
```

Nice and simple error handling
------------------------------

Error handling in Rust is quite nice, there are recoverable and unrecoverable errors. For recoverable errors, you can handle them using pattern matching on the `Result` enum or using the simple expect syntax. There is even a shorthand operator to propagate errors from a function. Take that Go.

```
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");

    // or

    let f = match File::open("hello.txt") {
        Ok(file) => file,
        Err(error) => {
            panic!("Problem opening the file: {:?}", error)
        },
    };
}
```

Tuples
------

Rust has built-in support for tuples, and this is highly helpful when you have to return multiple values from a function or when you want to unwrap a value and so on.

Block expressions
-----------------

In Rust, you can have block expressions with their own scope almost anywhere. It also lets you assign a variable value from block expressions, if statement, loops and so on.

```
fn main() {
    let foo = {
        println!("Assigning foo");
    };

    let bar = if foo > 5 { 6 } else { 10 };
}
```

Beautiful compiler output
-------------------------

Rust simply has the best error output during compilation that I have seen. It can’t get better than this I think. It is so helpful.

![](https://thepracticaldev.s3.amazonaws.com/i/3a6kh5lvp8nqx34xh00c.png)

Like many modern programming languages, Rust also provides a lot of build-in standard tooling and honestly, I think this is one of the best that I have come across. Rust has [Cargo](https://doc.rust-lang.org/book/ch01-03-hello-cargo.html) which is the built-in package manager and build system. It is an excellent tool. It takes care of all common project needs like compiling, building, testing and so on. It can even create new projects with a skeleton and manage packages globally and locally for the project. That means you don’t have to worry about setting up any tooling to get started in Rust. I love this, it saves so much time and effort. Every programming language should have this.

Rust also provides built-in utilities and asserts to write tests which then can be executed using Cargo.

In Rust, related functionality is grouped into modules, modules are grouped together into something called crates and crates are grouped into packages. We can refer to items defined in one module from another module. Packages are managed by cargo. You can specify external packages in the `Cargo.toml` file. Reusable public packages can be published to the [crates.io](https://crates.io/) registry.

There are even offline built-in docs that you can get by running `rustup docs` and `rustup docs --book` which is amazing. Thanks to Mohamed ELIDRISSI for [pointing it](https://dev.to/mohamedelidrissi_98/comment/hf85) out to me.

Concurrency
-----------

Rust has first-class support for memory safe concurrent programming. Rust uses threads for concurrency and has 1:1 threading implementation. i.e, 1 green thread per operating system thread. Rust compiler guarantees memory safety while using the threads. It provides features like waiting for all threads to finish, sharing data with `move` closures or channels(similar to Go). It also lets you use shared state and sync threads.

```
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

While I don’t like all aspects of macros in Rust, there are more things to like here than dislike. The annotation macros, for example, are quite handy. Not a fan of the procedure macros though. For advanced users, you can write your own macro rules and do metaprogramming.

Traits
------

Traits are synonymous to interfaces in Java, it is used to define shared behaviors that can be implemented on structs. Traits can even specify default methods. The only thing I dislike here is the indirect implementation.

```
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}

pub struct NewsArticle {
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize_author(&self) -> String {
        format!("@{}", self.author)
    }
}

fn main() {
    let article = NewsArticle {
        author: String::from("Iceburgh"),
        content: String::from(
            "The Pittsburgh Penguins once again are the best hockey team in the NHL.",
        ),
    };

    println!("New article available! {}", article.summarize());
}
```

Ability to use unsafe features if required
------------------------------------------

*   Useful in advanced use-cases where you know what you are doing. A necessary evil IMO. I like it since its doable only within an `unsafe { }` block making it very explicit. I would have moved this to the dislike section if that was not the case.
*   When you use these, the Rust compiler cannot guarantee memory and runtime safety and you are on your own to get this right. So definitely for advanced and experienced users.

* * *

Things that I didn’t like very much, in no particular order.

Complexity
----------

I don’t like it when a language offers multiple ways to do the same things. This is one think Golang does pretty well, there are no two ways to do the same thing and hence it is easier for people to work on larger codebases and to review code. Also, you don’t have to always think of the best possible way to do something. Unfortunately, Rust does this and I’m not a fan of it. IMO it makes the language more complex.

*   Too many ways for iterations -> loops, while, for, iterators.
*   Too many ways for creating procedures -> Functions, closures, macros

**Edit:** Based on discussions here and on [Reddit](https://www.reddit.com/r/rust/comments/dua0qv/my_first_impressions_of_rust/), I say my perception of complexity only have increased. It seems like once you get past all the niceties there are a lot of things that would take some time to wrap your head around. I’m pretty sure if you are experienced in Rust, it would be a cakewalk for you but the language indeed is quite complex, especially the ways functions and closures behave in different contexts, lifetimes in structs and stuff.

Shadowing of variables in the same context
------------------------------------------

So Rust lets you do this

```
{
    let foo = "hello";
    println!("{}", foo);
    let foo = "world";
    println!("{}", foo);
}
```

*   Kind of beats being immutable by default(I understand the reasoning of being able to perform transformations on an immutable variable, especially when passing references to a function and getting it back)
*   IMO lets people practice bad practices unintentionally, I would have rather marked the variable mutable as I consider the above mutation as well.
*   You can as easily accidentally shadow a variable as you would accidentally mutate one in Languages like JavaScript
*   Gives people a gun to shoot in the foot

**Edit:** I saw a lot of comments here and on [Reddit](https://www.reddit.com/r/rust/comments/dua0qv/my_first_impressions_of_rust/) explaining why this is good. While I agree that it is useful in many scenarios, so is the ability of mutation. I think it would have been perfectly fine not to have this and people would have still loved Rust and all of them would have defended the decision not have this. So my opinion on this hasn’t changed.

Functions are not first-class citizens
--------------------------------------

While it is possible to pass a function to another they are not exactly first-class citizens like in JavaScript or Golang. ~You cannot create closures from functions and you cannot assign functions to variables~. Closures are separate from functions in Rust, they are quite similar to Java lambdas from what I see. While closures would be sufficient to perform some of the functional style programming patterns it would have been much nicer if it was possible using just functions thus making language a bit more simple.

**Edit:** Oh boy! this opinion triggered a lot of discussions here and on [Reddit](https://www.reddit.com/r/rust/comments/dua0qv/my_first_impressions_of_rust/). So seems like Functions and closures are similar and different based on context, It also seems like Functions are almost like first-class citizens, but if you are used to languages like Go or JavaScript where functions are much more straight forward then you are in for a crazy ride. Functions in Rust seems much much more complex. A lot of people who commented seemed to miss the fact that my primary complaint was that having two constructs(closures and functions) that look and act quite similar in most of the scenarios makes things more complex. At least in Java and JS where there are multiple constructs(arrow functions, lambdas) those where due to the fact that they were added much later to the language and those are still something I don’t like in those languages. The best [explanation](https://dev.to/louy2/comment/hjeo) was from [Yufan Lou](https://dev.to/louy2) and [another](https://www.reddit.com/r/rust/comments/dua0qv/my_first_impressions_of_rust/f77w23i/) from [zesterer](https://www.reddit.com/user/zesterer/). I’m not gonna remove this from stuff I don’t like since I still don’t like the complexity here.

Implicit implementation of traits
---------------------------------

I’m not a fan of implicit stuff as its easier to abuse this and harder to read. You could define a struct in one file and you could implement a trait for that struct in another file which makes it less obvious. I prefer when the implementation is done by intent like in Java which makes it more obvious and easier to follow. While the way in Rust is not ideal, it is definitely better than Golang which is even more indirect.

* * *

Finally some stuff I still don’t like but I don’t consider them a big deal.

*   ~I don’t see the point of having the `const` keyword when `let` is immutable by default. It seems more like syntax sugar for the old school constants concept.~ [Diane](https://dev.to/artemis) [pointed out](https://dev.to/artemis/comment/hf2c) the difference `const` provides and that makes sense.
*   The block expression and implicit return style are a bit error-prone and confusing to get used to, I would have preferred explicit return. Also, it’s not that readable IMO.
*   If you have read my other post about Go, you might know that I’m not a fan of structs. Structs in Rust is very similar to structs in Golang. So like in Go, it would be easy to achieve unreadable struct structures. But fortunately, the structs in Rust seem much nicer than Go as you have functions, can use spread operator, shorthand syntax and so on here. Also, you can make structs which are Tuples. The structs in Rust are more like Java POJOs. I would have moved this to the liked section if having optional fields in structs where easier. Currently, you would have to wrap stuff in an `Optional` enum to do this. Also lifetimes :(
*   Given strings are the most used data types, it would have been nice to have a simpler way of working with strings(like in Golang) rather than working with the Vector types for mutable strings or slice types for immutable string literals. This makes code more verbose than it needs to be. This is more likely a compromise due to the fact that Rust is not garbage collected and has a concept of ownership to manage memory. https://doc.rust-lang.org/rust-by-example/std/str.html - **Edit**: _I have moved this point to nitpicks rather than dislikes after a [discussion](https://dev.to/robertorojasr/comment/hf27) on the comments with [robertorojasr](https://dev.to/robertorojasr)_

* * *

I like programming languages that focus more on simplicity rather than fancy syntax sugars and complex features. In my post [“My reflections on Golang”](https://deepu.tech/reflection-on-golang/), I explain why I consider Golang to be too simple for my taste. Rust, on the other hand, is leaning towards the other side of the spectrum. While it is not as complex as Scala it is not as simple as Go as well. So its somewhere in between, not exactly the sweet spot but almost near that quite close to where JavaScript is maybe.

So overall I can say that there are more things in Rust for me to like than to dislike which is what I would expect from a nice programming language. Also, bear in mind that I’m not saying Rust should do anything differently or that there are better ways to do things that I complained about. I’m just saying that I don’t like those things but I can live with it given the overall advantages. Also, I fully understand why some of those concepts are the way they are, those are mostly tradeoffs to focus on memory safety and performance.

But don’t be fooled by what you see over the hood, Rust is definitely not something you should start with as your first programming language IMO, as it has a quite a lot of complex concepts and constructs underneath but if you are already familiar with programming then it shouldn’t be an issue after banging your head on the doors a few times :P

So far I can say that I like Rust more than Golang even without implementing a real project with it and might choose Rust over Go for system programming use cases and for high-performance requirements.

**Note**: Some of my opinions have changed after using Rust more. Checkout my new post about the same

* * *

If you like this article, please leave a like or a comment.

You can follow me on [Twitter](https://twitter.com/deepu105) and [LinkedIn](https://www.linkedin.com/in/deepu05/).

Cover image credit: Image from [Link Clark, The Rust team](https://hacks.mozilla.org/2018/12/rust-2018-is-here/) under [Creative Commons Attribution Share-Alike License v3.0](https://creativecommons.org/licenses/by-sa/3.0/).

Post **2** of **7** in series **"languages"**.