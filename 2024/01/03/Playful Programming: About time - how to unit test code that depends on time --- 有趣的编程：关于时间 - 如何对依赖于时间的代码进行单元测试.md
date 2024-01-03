> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [playfulprogramming.blogspot.com](https://playfulprogramming.blogspot.com/2023/12/about-time.html)

Tuesday, December 19, 2023  
2023 年 12 月 19 日星期二
------------------------------------------------

  

### About time - how to unit test code that depends on time  
关于时间 - 如何对依赖于时间的代码进行单元测试

Suppose that the logic of your program depends on time. That is, you need to keep track of when something in the past happened, and what time it is now, and the logic of what to do depends on how much time passed between that previous event and now.  
假设程序的逻辑取决于时间。也就是说，你需要跟踪过去的事情是什么时候发生的，现在是什么时候，而做什么的逻辑取决于前一个事件和现在之间经过了多少时间。

There are many programs with this kind of behaviour. My experience is primarily from networking, where we need to figure out if a response is timely or late. Such systems often uses timers, i.e. on some action, request a notification at a specific point in time in the future.  
有许多程序具有这种行为。我的经验主要来自网络，我们需要弄清楚响应是及时还是迟到。此类系统通常使用计时器，即在某些操作上，请求将来特定时间点的通知。

How do you design such a system so that it is testable?  
你如何设计这样一个系统，使其可测试？

The naïve approach, to just call std::chrono::<some_clock>::now(), whenever you need a time stamp, makes unit-tests more or less impossible, so avoid that.  
幼稚的方法，在需要时间戳时只调用 std：：chrono：：：：now（），使单元测试或多或少不可能，所以请避免这种情况。  

Approach 1, the alias clock  
方法 1，别名时钟
---------------------------------------

Instead of directly referring to  std::chrono::<some_clock>::now() in your code, you refer to app_clock::now(), and in system builds app_clock is defined to be std::chrono::<some_clock>, but in unit-tests, they're some test clock.  
在代码中，您不是直接引用 std：：chrono：：：：now（），而是引用 app_clock：：now（），在系统构建中，app_clock 定义为 std：：chrono：：，但在单元测试中，它们是某个测试时钟。

This is an improvement. In the tests, you now have a means to control what time it is and how time advances. However, a major drawback is that you need to have different builds of your unit under test depending on situation.  
这是一个改进。在测试中，您现在有一种方法可以控制现在是什么时间以及时间如何前进。但是，一个主要缺点是您需要根据情况对测试单元进行不同的构建。

Approach 2, template specialization access  
方法 2，模板专用化访问
---------------------------------------------------------

This is a neat trick, using (abusing?) how the template machinery works in C++.  
这是一个巧妙的技巧，使用（滥用？）模板机制在 C++ 中的工作方式。

Create an encapsulation like this:  
创建如下所示的封装：

```
template <typename ...>
constexpr auto clock_impl = std::chrono::some_clock{};

template <typename ... Ts>
struct app_clock
{
    static
    std::chrono::some_clock::time_point now()
    {
        return clock_impl<Ts...>.now();
    }
};
```

Now whenever you call app_clock::now(), it will call clock_impl<>.now() which is std::chrono::<some_clock>::now()  
现在，每当您调用 app_clock：：now（） 时，它都会调用 clock_impl<>.now（），即 std：：chrono：：：：now（）.

For our tests, we can define a test_clock.  
对于我们的测试，我们可以定义一个 test_clock。

```
struct test_clock
{
    using time_point = std::chrono::some_clock::time_point;
    static time_point now() { return {};}
};

template <>
constexpr auto clock_impl<> = test_clock{};
```

  

Now we have a specialization for clock_impl that is our test_clock. It is imperative that the signatures of test_clock::now() and the default std::chrono::some_clock::now() are identical.  
现在我们有一个 clock_impl 专业化，这是我们 test_clock。test_clock：：now（） 和默认 std：：chrono：：some_clock：：now（） 的签名必须相同。

See an example at [https://godbolt.org/z/GbWYaGc7q](https://godbolt.org/z/GbWYaGc7q)  
请参阅 https://godbolt.org/z/GbWYaGc7q 中的示例  

This overcomes the need for having separate compilations for tests and production.  
这克服了对测试和生产进行单独编译的需要。  

Approach 3, the clock factory  
方法 3、钟表厂
----------------------------------------

 In this case, whenever the program needs to know what time it is, it calls clock_factory::get_clock(), which in production code returns some encapsulation of std::chrono::<some_clock>.  
在这种情况下，每当程序需要知道现在是几点时，它都会调用 clock_factory：：get_clock（），在生产代码中，它会返回 std：：chrono：：的一些封装。

This is better. Now the only code that differs between a unit test build and a production build is what clock_factory::get_clock() returns.  
这样更好。现在，单元测试版本和生产版本之间唯一不同的代码是 clock_factory：：get_clock（） 返回的内容。

Unfortunately these factories tends to be singletons, with all the problems that they bring.  
不幸的是，这些工厂往往是单例的，它们带来了所有问题。

If you want to model your test clock as a mock, you have the additional problem of how to ensure that the test code and the unit under test sees the same clock. It's also really difficult to correctly provide all the right expectations for the mock without over constraining the tests (exactly how many times should the time be asked for, and what time should be reported on each call?)  
如果要将测试时钟建模为模拟时钟，则还要解决另一个问题，即如何确保测试代码和被测单元看到相同的时钟。在不过度限制测试的情况下，正确地为模拟提供所有正确的期望也非常困难（确切地说应该要求多少次时间，以及每次通话应该报告什么时间？  

Approach 4, clocks from above  
方法 4，从上方的时钟
-------------------------------------------

Instead of having code ask for clocks from factories, you can model your program so that every class that needs to know the time has a constructor that accepts a clock and stores it as a member variable.  
您可以对程序进行建模，以便每个需要知道时间的类都有一个接受时钟并将其存储为成员变量的构造函数，而不是让代码向工厂请求时钟。

This gets rid of the singleton (yay!!!), but it adds a lot of extra storage and all the other problems remain.  
这摆脱了单例（是的!!），但它增加了很多额外的存储空间，所有其他问题仍然存在。  

Approach 5, pass time stamps  
方法 5，传递时间戳
-----------------------------------------

Here the problem is turned on its head. What if the code doesn't need to ask for the time, but can be told what time it is?  
在这里，问题被颠倒过来了。如果代码不需要询问时间，但可以被告知现在是几点怎么办？

An observation is that many systems like this only need to know the time at a few places in the code, typically at the source of events. Get the time when a message is received. Get the time on user input. Get the time when receiving a signal. Get the time when a timer fires.  
一个观察结果是，许多这样的系统只需要知道代码中几个位置的时间，通常是在事件的源头。获取收到消息的时间。获取用户输入的时间。获取接收信号时的时间。获取计时器触发的时间。

Then, all actions that come as a result of these events, are passed the time stamp.  
然后，由于这些事件而发生的所有操作都将传递时间戳。

Passing a chrono time_point is cheap (it's typically a 64-bit value passed in a register).  
传递计时 time_point 很便宜（它通常是在寄存器中传递的 64 位值）。

Now tests become easy. All tests of code that needs to know the time are given time points as input, controlled in full by the test code.  
现在测试变得容易了。所有需要知道时间的代码测试都以时间点作为输入，完全由测试代码控制。

An additional advantage is that you can (should!) instrument your timer code so that for every timer that fires, you keep track of how late (or early!) it fired. This is typically much more interesting than to measure CPU-load.  
另一个优点是，您可以（应该！）检测计时器代码，以便对于每个触发的计时器，您都可以跟踪它触发的时间有多晚（或早！这通常比测量 CPU 负载更有趣。

The disadvantage is a loss in precision of time. Some cycles will pass between getting the time stamp at the event source and the logic decision that depends on the time. The programs that I have experience in writing are not bothered with that loss of precision. Your program may be different, in which case one of the other approaches may be a better choice.  
缺点是时间精度下降。在获取事件源的时间戳和取决于时间的逻辑决策之间会经过一些周期。我有编写经验的程序不会为这种精度损失而烦恼。您的程序可能有所不同，在这种情况下，其他方法之一可能是更好的选择。

Which to choose? 选择哪个？
----------------------

If you can live with the loss of precision from Approach 5, pass time stamps, I think that is the preferred way. It makes everything so much easier. If the loss of precision is unacceptable, Alternative 2, template specialization access is probably the best option.  
如果你能忍受方法 5 的精度损失，传递时间戳，我认为这是首选的方式。它使一切变得如此简单。如果精度损失是不可接受的，则备选方案 2（模板专用化访问）可能是最佳选择。