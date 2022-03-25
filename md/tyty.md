> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zackoverflow.dev](https://zackoverflow.dev/writing/tyty)

> TLDR; I'm making a Typescript type-checker in Rust. Right now it supports a smaller subset of the typ......

In the past I've [written](https://zackoverflow.dev/writing/the-problem-with-typescript) about Typescript's biggest underlying problem: it's slow compilation speeds. Amazing projects like [esbuild](https://github.com/evanw/esbuild), [SWC](https://github.com/swc-project/swc), and [bun](https://bun.sh/) solve the first half of the problem, emitting JS from TS, but they skip the critical type-checking phase completely due to complexity.

In that post I briefly mentioned that most of the major players (also including Deno, Rome, etc.) in the JS compilation ecosystem don't have any plans to type-check TS code. SWC is the only project actively solving this problem, but it's unclear if it will be [open-source or commercially licensed](https://github.com/swc-project/swc/issues/571#issuecomment-803553183).

This basically means that the majority of us will be at the mercy of `tsc`'s abysmally slow speed, and that prompted me to consider building an open-source type-checker of my own. It certainly would be a fun and challenging undertaking, and would provide utility to a vast number of people.

And thus, I started working on a TS type-checker arbitrarily named `tyty` about a week ago in my spare time. It currently supports a small subset of Typescript, I plan to open-source it soon.

The obvious starting point was to build upon an existing JS/TS parser/bundler by type-checking its output AST. The [difficult problem](https://github.com/evanw/esbuild/issues/101#issuecomment-626239597) of parsing JS/TS has been solved many times over now, why reinvent the wheel? Of the existing projects out there, SWC is the only one which makes its AST nodes accessible to its public facing API, so it became the clear choice.

Challenges
----------

Typescript's type-system is simple in some ways and complex in others. In the traditional type systems view, it is quite simple. Typescript uses bidirectional type-checking which has a very simple local type inference algorithm. There are type systems with more sophicasted inference, for example the [Hindley-Milner](https://en.wikipedia.org/wiki/Hindley%E2%80%93Milner_type_system) type system. As a result, the majority of type-checking in Typescript is more straightforward than you'd think.

The remaining difficulty comes from some of Typescript's unique features. One of the coolest features is its conditional and recursive types. When paired with generic parameters, they allow you to add complex logic to your type definitions, effectively giving you a simple functional programming language (that operates on types instead of data), all embedded within Typescript's type-system. For example, see this [JSON parser](https://github.com/jamiebuilds/json-parser-in-typescript-very-bad-idea-please-dont-use) created entirely from Typescript types.

The consequence of this is the problem of type checking complex types becomes more of a problem of building an interpreter for this meta-language embedded within the type-system. Having created a [programming language](https://github.com/zackradisic/aussieplusplus) before, I am really excited to tackle this aspect of Typescript as it's more familiar ground for me.

Goals
-----

The end goal is to reach feature parity with Typescript and provide a way to compile TS code either through SWC's upcoming plugin system or a wrapper around it.

I also want to leverage the speed of a Rust-based type-checker to create a very fast LSP server implementation that works well in all editors (not just VSCode!).

Additionally, I want to `tyty` to have better and more descriptive error messages than `tsc`'s. Isn't this nice?: ![](https://zackoverflow.dev/tyty-errors.jpeg) This is made using the amazing [ariadne](https://github.com/zesterer/ariadne) Rust crate.

Ideally I would also like to expose some of the type-checking functionality in a library, because Typescript's compiler API is quite horrendous. This would give developers more robust tooling to manipulate Typescript types. When I was working on [tsgql](https://zackoverflow.dev/writing/tsgql) I had such a terrible time working with the TS compiler API.

This is more of a moonshot idea, but it would be amazing if `tyty` could interoperate with other JS/TS compilers. It could be compiled into a dynamic library and called through FFI.