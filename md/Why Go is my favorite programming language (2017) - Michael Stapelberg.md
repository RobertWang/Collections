> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [michael.stapelberg.ch](https://michael.stapelberg.ch/posts/2017-08-19-golang_favorite/)

> I strive to respect everybody’s personal preferences, so I usually steer clear of debates about which......

Table of contents

*   [My background](#my-background)
*   [1. Clarity](#1-clarity)
    *   [Formatting](#formatting)
    *   [High-quality code](#high-quality-code)
    *   [Opinions](#opinions)
    *   [Few keywords and abstraction layers](#few-keywords-and-abstraction-layers)
*   [2. Speed](#2-speed)
    *   [Quick feedback / low latency](#quick-feedback--low-latency)
    *   [Maximum resource usage](#maximum-resource-usage)
*   [3. Rich standard library](#3-rich-standard-library)
*   [4. Tooling](#4-tooling)
*   [Getting started](#getting-started)
*   [Caveats](#caveats)

I strive to respect everybody’s personal preferences, so I usually steer clear of debates about which is the best programming language, text editor or operating system.

However, recently I was asked a couple of times why I like and use a lot of [Go](https://golang.org/), so here is a coherent article to fill in the blanks of my ad-hoc in-person ramblings :-).

My background
-------------

I have used C and Perl for a number of decently sized projects. I have written programs in Python, Ruby, C++, CHICKEN Scheme, Emacs Lisp, Rust and Java (for Android only). I understand a bit of Lua, PHP, Erlang and Haskell. In a previous life, I developed a number of programs using [Delphi](https://en.wikipedia.org/wiki/Delphi_(programming_language)).

I had a brief look at Go in 2009, when it was first released. I seriously started using the language when Go 1.0 was released in 2012, featuring the [Go 1 compatibility guarantee](https://golang.org/doc/go1compat). I still have [code](https://github.com/stapelberg/greetbot) running in production which I authored in 2012, largely untouched.

1. Clarity
----------

### Formatting

Go code, by convention, is formatted using the [`gofmt`](https://golang.org/cmd/gofmt/) tool. Programmatically formatting code is not a new idea, but contrary to its predecessors, `gofmt` supports precisely one canonical style.

Having all code formatted the same way makes reading code easier; the code feels familiar. This helps not only when reading the standard library or Go compiler, but also when working with many code bases — think Open Source, or big companies.

Further, auto-formatting is a huge time-saver during code reviews, as it eliminates an entire dimension in which code could be reviewed before: now, you can just let your continuous integration system verify that `gofmt` produces no diffs.

Interestingly enough, having my editor apply `gofmt` when saving a file has changed the way I write code. I used to attempt to match what the formatter would enforce, then have it correct my mistakes. Nowadays, I express my thought as quickly as possible and trust `gofmt` to make it pretty ([example](https://play.golang.org/p/I6GJwiT77v) of what I would type, click Format).

### High-quality code

I use the standard library ([docs](https://golang.org/pkg/), [source](https://github.com/golang/go/tree/master/src)) quite a bit, see below.

All standard library code which I have read so far was of extremely high quality.

One example is the [`image/jpeg`](https://golang.org/pkg/image/jpeg/) package: I didn’t know how JPEG worked at the time, but it was easy to pick up by switching between the [Wikipedia JPEG article](https://en.wikipedia.org/wiki/JPEG) and the `image/jpeg` code. If the package had a few more comments, I would qualify it as a teaching implementation.

### Opinions

I have come to agree with many opinions the Go community holds, such as:

*   [Variable names](https://github.com/golang/go/wiki/CodeReviewComments#variable-names) should be short by default, and become more descriptive the further from its declaration a name is used.
*   Keep the dependency tree small (to a reasonable degree): [a little copying is better than a little dependency](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=9m28s)
*   There is a cost to introducing an abstraction layer. Go code is usually rather clear, at the cost of being a bit repetitive at times.
*   See [CodeReviewComments](https://github.com/golang/go/wiki/CodeReviewComments) and [Go Proverbs](https://go-proverbs.github.io/) for more.

### Few keywords and abstraction layers

The Go specification lists only [25 keywords](https://golang.org/ref/spec#Keywords), which I can easily keep in my head.

The same is true for [builtin functions](https://golang.org/pkg/builtin/) and [types](https://golang.org/ref/spec#Types).

In my experience, the small number of abstraction layers and concepts makes the language easy to pick up and quickly feel comfortable in.

While we’re talking about it: I was surprised about how readable the [Go specification](https://golang.org/ref/spec) is. It really seems to target programmers (rather than standards committees?).

2. Speed
--------

### Quick feedback / low latency

I love quick feedback: I appreciate websites which load quickly, I prefer fluent User Interfaces which don’t lag, and I will choose a quick tool over a more powerful tool any day. [The findings](https://blog.gigaspaces.com/amazon-found-every-100ms-of-latency-cost-them-1-in-sales/) of large web properties confirm that this behavior is shared by many.

The authors of the Go compiler respect my desire for low latency: compilation speed matters to them, and new optimizations are carefully weighed against whether they will slow down compilation.

A friend of mine had not used Go before. After installing the [RobustIRC](https://robustirc.net/) bridge using `go get`, they concluded that Go must be an interpreted language and I had to correct them: no, the Go compiler just is that fast.

Most Go tools are no exception, e.g. `gofmt` or `goimports` are blazingly fast.

### Maximum resource usage

For batch applications (as opposed to interactive applications), utilizing the available resources to their fullest is usually more important than low latency.

It is delightfully easy to profile and change a Go program to utilize all available IOPS, network bandwidth or compute. As an example, I wrote about [filling a 1 Gbps link](https://people.debian.org/~stapelberg/2014/01/17/debmirror-rackspace.html), and optimized [debiman](https://github.com/Debian/debiman/) to utilize all available resources, reducing its runtime by hours.

3. Rich standard library
------------------------

The [Go standard library](https://golang.org/pkg) provides means to effectively use common communications protocols and data storage formats/mechanisms, such as TCP/IP, HTTP, JPEG, SQL, …

Go’s standard library is the best one I have ever seen. I perceive it as well-organized, clean, small, yet comprehensive: I often find it possible to write reasonably sized programs with just the standard library, plus one or two external packages.

Domain-specific data types and algorithms are (in general) not included and live outside the standard library, e.g. [`golang.org/x/net/html`](https://godoc.org/golang.org/x/net/html). The `golang.org/x` namespace also serves as a staging area for new code before it enters the standard library: the Go 1 compatibility guarantee precludes any breaking changes, even if they are clearly worthwhile. A prominent example is `golang.org/x/crypto/ssh`, which had to break existing code to [establish a more secure default](https://github.com/golang/crypto/commit/e4e2799dd7aab89f583e1d898300d96367750991).

To download, compile, install and update Go packages, I use the `go get` tool.

All Go code bases I have worked with use the built-in [`testing`](https://golang.org/pkg/testing/) facilities. This results not only in easy and fast testing, but also in [coverage reports](https://blog.golang.org/cover) being readily available.

Whenever a program uses more resources than expected, I fire up `pprof`. See this [golang.org blog post about `pprof`](https://blog.golang.org/profiling-go-programs) for an introduction, or [my blog post about optimizing Debian Code Search](https://people.debian.org/~stapelberg/2014/12/23/code-search-taming-the-latency-tail.html). After importing the [`net/http/pprof` package](https://golang.org/pkg/net/http/pprof/), you can profile your server while it’s running, without recompilation or restarting.

Cross-compilation is as easy as setting the `GOARCH` environment variable, e.g. `GOARCH=arm64` for targeting the Raspberry Pi 3. Notably, tools just work cross-platform, too! For example, I can profile [gokrazy](https://gokrazy.org/) from my amd64 computer: `go tool pprof ~/go/bin/linux_arm64/dhcp http://gokrazy:3112/debug/pprof/heap`.

[godoc](https://godoc.org/golang.org/x/tools/cmd/godoc) displays documentation as plain text or serves it via HTTP. [godoc.org](https://godoc.org/) is a public instance, but I run a local one to use while offline or for not yet published packages.

Note that these are standard tools coming with the language. Coming from C, each of the above would be a significant feat to accomplish. In Go, we take them for granted.

Getting started
---------------

Hopefully I was able to convey why I’m happy working with Go.

If you’re interested in getting started with Go, check out [the beginner’s resources](https://github.com/gopheracademy/gopher/blob/1cdbcd9fc3ba58efd628d4a6a552befc8e3912be/bot/bot.go#L516) we point people to when they join the Gophers slack channel. See [https://golang.org/help/](https://golang.org/help/).

Caveats
-------

Of course, no programming tool is entirely free of problems. Given that this article explains why Go is my favorite programming language, it focuses on the positives. I will mention a few issues in passing, though:

*   If you use Go packages which don’t offer a stable API, you might want to use a specific, known-working version. Your best bet is the [dep](https://github.com/golang/dep) tool, which is not part of the language at the time of writing.
*   Idiomatic Go code does not necessarily translate to the highest performance machine code, and the runtime comes at a (small) cost. In the rare cases where I found performance lacking, I successfully resorted to [cgo](https://golang.org/cmd/cgo/) or assembler. If your domain is hard-realtime applications or otherwise extremely performance-critical code, your mileage may vary, though.
*   I wrote that the Go standard library is the best I have ever seen, but that doesn’t mean it doesn’t have any problems. One example is [complicated handling of comments](https://golang.org/issues/20744) when modifying Go code programmatically via one of the standard library’s oldest packages, `go/ast`.