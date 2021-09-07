> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [shopify.engineering](https://shopify.engineering/shopify-Webassembly)

> At Shopify, we’re keeping the flexibility of untrusted Partner code, but executing it on our own infr......

_On February 24, 2021, Shipit!, our monthly event series, presented Making Commerce Extensible with WebAssembly. The [video is now available](#Register "Shipit! Presents: Shipit! Presents: Making Commerce Extensible with WebAssembly")._

At Shopify we aim to make what most merchants need easy, and the rest possible. We make the rest possible by exposing interfaces to query, extend and alter our Platform. These interfaces empower a rich ecosystem of Partners to solve a variety of problems. The primary mechanism of this ecosystem is an “App”, an independently hosted web service which communicates with Shopify over the network. This model is powerful, but comes with a host of technical issues. Partners are stretched beyond their available resources as they have to build a web service that can operate at Shopify’s scale. Even if Partners’ resources were unlimited, the network latency incurred when communicating with Shopify precludes the use of Apps for time sensitive use cases.

We want Partners to focus on using their domain knowledge to solve problems, and not on managing scalable web services. To make this a reality we’re keeping the flexibility of untrusted Partner code, but executing it on our own infrastructure. We choose a universal format for that code that ensures it’s performant, secure, and flexible: WebAssembly.

What is WebAssembly? According to [WebAssembly.org](https://webassembly.org/ "WebAssembly"): 

“WebAssembly (abbreviated Wasm) is a binary instruction format for a stack-based virtual machine. Wasm is designed as a portable compilation target for programming languages, enabling deployment on the web for client and server applications.”

To learn more, see this series of [illustrated articles written by Lin Clark of Mozilla](https://hacks.mozilla.org/2017/02/a-cartoon-intro-to-webassembly/ "A cartoon intro to WebAssembly") with information on WebAssembly and its history.

Wasm is often presented as a performant language that runs alongside JavaScript from within the Browser. We, however, execute Wasm outside of the browser and with no Javascript involved. Wasm, far from being solely a Javascript replacement, is designed for [Web and Non-Web Embeddings alike](https://webassembly.org/docs/non-web/ "Non-Web Embeddings - WebAssembly"). It solves the more general problem of performant execution in untrusted environments, which exists in browsers and code execution engines alike. Wasm satisfies our three main technical requirements: security, performance, and flexibility.

Security
--------

Executing untrusted code is a dangerous thing—it's exceptionally difficult to predict by nature, and it has potential to cause harm to Shopify’s platform at large. While no application is entirely secure, we need to both prevent security flaws and mitigate their impacts when they occur.

Wasm executes within a sandboxed stack-based environment, relying upon explicit imports to allow communication with the host. Because of this, you cannot **express** anything malicious in Wasm. You can only express manipulations of the virtual environment and use provided imports. This differs from bytecodes which have references to the computers or operating systems they expect to run on built right into the syntax.

Wasm also hosts a number of features which protect the user from buggy code, including protected call stacks and runtime type checking. [More details on the security model of Wasm can be found on WebAssembly.org](https://webassembly.org/docs/security/ "Security - WebAssembly").

Performance
-----------

In ecommerce, speed is a competitive advantage that merchants need to drive sales. If a feature we deliver to merchants doesn’t come with the right tradeoff of load times to customization value, then we may as well not deliver it at all.

Wasm is designed to leverage common hardware capabilities that provide it near native performance on a wide variety of platforms. It’s used by a community of performance driven developers looking to optimize browser execution. As a result, Wasm and surrounding tooling was built, and continues to be built, with a performance focus.

Flexible
--------

A code execution service is only as useful as the developers using it are productive. This means providing first class development experiences in multiple languages they’re familiar with. As a bytecode format, Wasm is targeted by a number of different compilers. This allows us to support multiple languages for developer use without altering the underlying execution model.

Community Driven
----------------

We have a fundamental alignment in goals and design, which provides our “engineering reason” for using Wasm. But there’s more to it than that—it’s about the people as well as the technology. If nobody was working on the Wasm ecosystem, or even if it was just on life support in its current state, **we wouldn’t use it**. WebAssembly is an energized community that’s constantly building new things and has a lot of potential left to reach. By becoming a part of that community, Shopify stands to gain significantly from that enthusiasm.

We’re also contributing to that enthusiasm ourselves. We’re collecting user feedback, discussing feature gaps, and most importantly contributing to the open source tools we depend on. We think this is the start of a healthy reciprocal relationship between ourselves and the WebAssembly community, and we expect to expand these efforts in the future.

Now that we’ve covered WebAssembly and why we’re using it, let’s move onto how we’re executing it.

We use an open source tool called [Lucet](https://www.fastly.com/blog/announcing-lucet-fastly-native-webassembly-compiler-runtime "Announcing Lucet: Fastly’s native WebAssembly compiler and runtime") (originally written by Fastly). As a company, Fastly provides a programmable edge cloud platform. They’re trying to bring execution of high-volume, short-lived, and untrusted modules closer to where they’re being requested. This is the same as the problem we’re trying to solve with our Partner code, so it’s a natural fit to be using the same tools.

### Lucet

Lucet is both a runtime and a compiler for Wasm. Modules are represented in Wasm for the safety that representation provides. Recall that you can’t _express_ anything malicious in Wasm. Lucet takes advantage of this and uses a validation of the Wasm module as a security check. After the validation, the module is compiled to an executable artifact with near bare metal performance. It also supports ahead of time compilation, allowing us to have these artifacts ready to execute at runtime. Lucet containers boast an impressive startup time of _35_ _μs_. That’s because it’s a container that doesn’t need to do anything at all to start up.  If you want the full picture, Tyler McMullan, the CTO of Fastly, did a great talk which [gives an overview of Lucet and how it works](https://www.youtube.com/watch?v=QdWaQOgvd-g "Lucet: Safe WebAssembly Outside the Browser by Tyler McMullen").

![](https://cdn.shopify.com/s/files/1/0779/4361/files/flow-diagram-shopify-wasm-engine.jpg?v=1608224390) A flow diagram showing Shopify's Wasm engine

We wrap Lucet within a Rust web service which manages the I/O and storage of modules, which we call the Wasm Engine. This engine is called by Shopify during a runtime process, usually a web request, in order to satisfy some function. It then applies the output in context of the callsite. This application could involve the creation of a discount, the enforcement of a constraint, or any form of synchronous behaviour Merchants want to customize within the Platform.

### Execution Performance

Here’s some metrics pulled from a recent performance test. During this test, 100k modules were executed per minute for approximately 5 min. These modules contained a trivial implementation of enforcing a limit on the number of items purchased in a cart. 

![](https://cdn.shopify.com/s/files/1/0779/4361/files/Lucet-module-execution.jpg?v=1608225321) Time taken to execute a module

This chart demonstrates a breakdown of the time taken to execute a module, including I/O with the container and the execution of the module. The y-axis is time in ms, the x-axis is the time over which the test was running.

The light purple bar shows the time taken to execute the module in Lucet, the width of which hovers around 100 μs. The remaining bars deal with I/O and engine specifics, and the total time of execution is around 4 ms. All times are 99th percentiles (p99).To put these times in perspective, let’s compare these times to the request times of [Storefront Renderer, our performant Online Store rendering service](https://shopify.engineering/how-shopify-reduced-storefront-response-times-rewrite "How Shopify Reduced Storefront Response Times with a Rewrite"):

![](https://cdn.shopify.com/s/files/1/0779/4361/files/Storefront-Renderer-Response-Times.jpg?v=1608300459) Storefront Renderer response time

This chart demonstrates the request time to Storefront Renderer over time. The y-axis is request time in seconds. The x-axis is the time over which the values were retrieved. The light blue line representing the 99th percentile hovers around 700 ms.

Then if we consider the time taken by our module execution process to be generally under 5 ms, we can say that the performance impact of Lucet execution is negligible.

Generating WebAssembly
----------------------

To get value out of our high performance execution engine, we’ll need to empower developers to create compatible Wasm modules. Wasm is primarily intended as a compilation target, rather than something you write by hand ([though you can write Wasm by hand](https://blog.scottlogic.com/2018/04/26/webassembly-by-hand.html "Writing WebAssembly By Hand")). This leaves us with the question of what languages we’ll support and to what extent.

Theoretically any language with a Wasm target can be supported, but the effort developers spend to conform to our API is better focused on solving problems for merchants. That’s why we’ve chosen to provide first class support to a single language that includes tools that get developers up and running quickly.At Shopify, our language of choice is Ruby. However, because Ruby is a dynamic language, we can’t compile it down to Wasm directly. We explored solutions involving compiling interpreters, but found that there was a steep performance penalty. Because of this, we decided to go with a statically compiled language and revisit the possibility of dynamic languages in the future.

Through our research we found that developers in our ecosystem were most familiar with Javascript. Unfortunately, Javascript was precluded as it’s a dynamic language like Ruby. Instead, we chose a language with familiar TypeScript-like syntax called [AssemblyScript](https://www.assemblyscript.org/ "AssemblyScript").

Using AssemblyScript
--------------------

At first glance, there are a huge number of languages that support a WebAssembly target. Unfortunately, there are two broad categories of WebAssembly compilers which we can’t use:

*   Compilers that generate environment or language specific artifacts, namely node or the browser. (Examples: [Asterius](https://github.com/tweag/asterius "Asterius: A Haskell to WebAssembly compiler"), [Blazor](https://dotnet.microsoft.com/apps/aspnet/web-apps/blazor "Blazor Build client web apps with C#"))
*   Compilers that are designed to work only with a particular Runtime. The modules generated by these compilers rely upon special language specific imports. This is often done to support a language’s standard library, which expects certain system calls or runtime features to be available. Since we don’t want to be locked down to a certain language or tool, we don’t use these compilers. (Examples: [Lumen](https://github.com/lumen/lumen "Lumen - A new compiler and runtime for BEAM languages"))

These are powerful tools in the right conditions, but aren’t built for our use case. We need tools that produce WebAssembly, rather than tools which are _powered_ by WebAssembly. AssemblyScript is one such tool.

AssemblyScript, like many tools in the WebAssembly space, is still under development. It’s missing a few key features, such as closure support, and it still has a number of edge case bugs. This is where the importance of the community comes in.

The language and the tooling around AssemblyScript has an active community of enthusiasts and maintainers who have supported Shopify since we first started using the language in 2019. We’ve supported the community through [an OpenCollective donation](https://opencollective.com/shopify "Open Collective: Shopify") and continuing code contributions. We’ve written [a language server](https://github.com/saulecabrera/asls), made some progress towards implementing closures, and have written bug fixes for the compiler and surrounding tooling.

We’ve also integrated AssemblyScript into our own early stage tooling. We’ve built integrations into [the Shopify CLI](https://github.com/Shopify/shopify-app-cli "Shopify App CLI") which will allow developers to create, test, and deploy modules from their command line. To improve developer ergonomics, we provide SDKs which handle the low level implementation concerns of Shopify defined objects like “Money”. In addition to these tools, we’re building out systems which allow Partners to monitor their modules and receive alerts when their modules fail. The end goal is to give Partners the ability to move their code onto our service without losing any of the flexibility or observability they had on their own platform.

As we tear down the boundaries between Partners and Merchants, we connect merchants with the entrepreneurs ready to solve their problems. If you have ideas on how our code execution could help you and the Apps you own or use, please tweet us at @ShopifyEng. To learn more about Apps at Shopify and how to get started, visit [our developer page](https://developers.shopify.com/app-development "Shopify Developers - You build a great app. We’ll make the rest easy.").

Duncan is a Senior Developer at Shopify. He is currently working on the Scripts team, a team dedicated to enabling and managing untrusted code execution within Shopify for Merchants and Developers alike.

If you love working with open source tools, are passionate about API design and extensibility, and want to [work remotely](https://www.shopify.com/partners/blog/remote-work), we’re always hiring! Reach out to us or [apply on our careers page](https://www.shopify.com/careers/teams/data?itcat=EngBlog&itterm=Post).