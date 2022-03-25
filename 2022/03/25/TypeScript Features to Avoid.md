> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.executeprogram.com](https://www.executeprogram.com/blog/typescript-features-to-avoid)

> Learn programming tools like JavaScript, TypeScript, SQL, and regular expressions fast. Interactive l......

[TypeScript Features to Avoid](/blog/typescript-features-to-avoid)
==================================================================

Posted on 2022-01-18
--------------------

This post lists four TypeScript features that we recommend you avoid. Depending on your circumstances, there may be good reasons to use them, but we think that avoiding them is a good default.

TypeScript is a complex language that has evolved significantly over time. Early in its development, the team added features that were incompatible with JavaScript. Recent development is more conservative, maintaining stricter compatibility with JavaScript features.

As with any mature language, we have to make difficult decisions about which TypeScript features to use and which to avoid. We experienced these trade-offs firsthand while building [Execute Program](/)'s backend and frontend in TypeScript, and while creating our comprehensive [TypeScript courses](/courses/typescript). Based on our experience, here are four recommendations about which features to avoid.

**I. Avoid enums ([See our lesson](/courses/typescript/lessons/enum))**

Enums give names to a set of constants. In the example below, `HttpMethod.Get` is a name for the string `'GET'`. The `HttpMethod` type is conceptually similar to a union type between literal types, like `'GET' | 'POST'`.

```
enum HttpMethod {
  Get = 'GET',
  Post = 'POST',
}
const method: HttpMethod = HttpMethod.Post;
method; // Evaluates to 'POST'
```

Here's the argument in favor of enums:

Suppose that we eventually need to replace the string `'POST'` above with `'post'`. We change the enum's value to `'post'` and we're done! Other code in the system only references the enum member via `HttpMethod.Post`, and that enum member still exists.

Now imagine the same change with a union type instead of an enum. We define the union `'GET' | 'POST'`, then later we decide to change it to `'get' | 'post'`. Any code that tries to use `'GET'` or `'POST'` as an `HttpMethod` is now a type error. We have to update all of that code manually, which is an extra step when compared to enums.

This code maintenance argument for enums isn't very strong. When we add a new member to an enum or union, it rarely changes after creation. If we use unions, it's true that we may have to spend some time updating them in multiple places, but it's not a big problem because it happens rarely. Even when it does happen, the type errors can show us which updates to make.

The downside to enums comes from how they fit into the TypeScript language. TypeScript is supposed to be JavaScript, but with static type features added. If we remove all of the types from TypeScript code, what's left should be valid JavaScript code. The formal word used in the TypeScript documentation is "type-level extension": most TypeScript features are type-level extensions to JavaScript, and they don't affect the code's runtime behavior.

Here's a concrete example of type-level extension. We write this TypeScript code:

```
function add(x: number, y: number): number {
  return x + y;
}
add(1, 2); // Evaluates to 3
```

The compiler checks the code's types. Then it needs to generate JavaScript code. Fortunately, that step is easy: the compiler simply removes all of the type annotations. In this case, that means removing the `: number`s. What's left is perfectly legal JavaScript code.

```
function add(x, y) {
  return x + y;
}
add(1, 2); // Evaluates to 3
```

Most TypeScript features work in this way, following the type-level extension rule. To get JavaScript code, the compiler simply removes the type annotations.

Unfortunately, enums break this rule. `HttpMethod` and `HttpMethod.Post` were parts of a type, so they should be removed when TypeScript generates JavaScript code. However, if the compiler simply removes the enum types from our code examples above, we're still left with JavaScript code that references `HttpMethod.Post`. That will error during execution: we can't reference `HttpMethod.Post` if the compiler deleted it!

```
/* This is compiled JavaScript code referencing a TypeScript enum. But if the
 * TypeScript compiler simply removes the enum, then there's nothing to
 * reference!
 *
 * This code fails at runtime:
 *   Uncaught ReferenceError: HttpMethod is not defined */
const method = HttpMethod.Post;
```

TypeScript's solution in this case is to break its own rule. When compiling an enum, the compiler emits extra JavaScript code that never existed in the original TypeScript code. Few TypeScript features work like this, and each adds a confusing complication to the otherwise simple TypeScript compiler model. For these reasons, we recommend avoiding enums and using unions instead.

Why does the type-level extension rule matter?

Let's consider how the rule interacts with the ecosystem of JavaScript and TypeScript tools. TypeScript projects are inherently JavaScript projects, so they often use JavaScript build tools like [Babel](https://babeljs.io/) and [webpack](https://webpack.js.org/). These tools were designed for JavaScript, and it's still their primary focus today. Each tool is also an ecosystem of its own. There's a seemingly-endless universe of Babel and webpack plugins to process code.

How can Babel, webpack, their many plugins, and all of the other tools and plugins in the ecosystem fully support TypeScript? For most of the TypeScript language, the type-level extension rule makes these tools' jobs relatively easy. The tools strip out the type annotations, leaving valid JavaScript.

When it comes to enums (and namespaces, which we'll see in a moment) things are more difficult. It's not good enough to simply remove enums. The tools have to turn `enum HttpMethod { ... }` into working JavaScript code, even though JavaScript doesn't have enums at all.

This brings us to the practical problem with TypeScript's violations of its own type-level extension rule. Tools like Babel, webpack, and their plugins are all designed for JavaScript first, so TypeScript support is just one of their many features. Sometimes, TypeScript support doesn't receive as much attention as JavaScript support, which can lead to bugs.

The vast majority of tools will do a good job with variable declarations, function definitions, etc.; all of those are relatively easy to work with. But sometimes mistakes creep in with enums and namespaces, because they require more than just stripping off the type annotations. You can trust the TypeScript compiler itself to compile those features correctly, but some rarely-used tools in the ecosystem may make mistakes.

When your compiler, bundler, minifier, linter, code formatter, etc. silently miscompiles or misinterprets an out-of-the-way piece of your system, it can be very difficult to debug. Compiler bugs are [notoriously difficult](http://r6.ca/blog/20200929T023701Z.html) to track down. Note these words: "over **the week**, with the **help of my colleagues**, we managed to get a better understanding of the scope of the bug." (Emphasis added.)

**II. Avoid namespaces ([See our lesson](/courses/typescript/lesson/namespaces))**

Namespaces are like modules, except that more than one namespace can live in a single file. For example, we might have a file that defines separate namespaces for its exported code and for its tests. (We don't recommend doing this, but it's a simple way to show off namespaces.)

```
namespace Util {
  export function wordCount(s: string) {
    return s.split(/\b\w+\b/g).length - 1;
  }
}

namespace Tests {
  export function testWordCount() {
    if (Util.wordCount('hello there') !== 2) {
      throw new Error("Expected word count for 'hello there' to be 2");
    }
  }
}

Tests.testWordCount();
```

Namespaces can cause problems in practice. In the section on enums above, we saw TypeScript's"type-level extension" rule. Normally, the compiler removes all of the type annotations, and what's left is valid JavaScript code.

Namespaces break the type-level extension rule in the same way as enums. In `namespace Util { export function wordCount ... }`, we can't remove the type definitions. The entire namespace is a TypeScript-specific type definition! What would happen to the other code outside of the namespace calling `Util.wordCount(...)`? If we delete the `Util` namespace before generating JavaScript code, then `Util` doesn't exist any more, so the `Util.wordCount(...)` function call can't possibly work.

As with enums, the TypeScript compiler can't simply delete the namespace definitions. Instead, it has to generate new JavaScript code that doesn't exist in the original TypeScript code.

For enums, our suggestion was to use unions instead. For namespaces, we recommend using regular modules. It may be a bit annoying to create many small files, but modules have the same fundamental functionality as namespaces without the potential downsides.

**III. Avoid decorators (for now)**

Decorators are functions that modify or replace other functions (or classes). Here's a decorator example taken from the [official docs](https://www.typescriptlang.org/docs/handbook/decorators.html).

```
// This is the decorator.
@sealed
class BugReport {
  type = "report";
  title: string;

  constructor(t: string) {
    this.title = t;
  }
}
```

The `@sealed` decorator above alludes to the [C# sealed modifier](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/sealed), which prevents other classes from inheriting from the sealed class. We'd implement it by writing a `sealed` function that takes a class and modifies it to prevent inheritance.

Decorators were added to TypeScript first, before beginning their standardization process in JavaScript (ECMAScript). As of January 2022, decorators are still a [stage 2 ECMAScript proposal](https://github.com/tc39/proposal-decorators). Stage 2 is "draft". The decorator proposal also seems to be stuck in committee purgatory: it's been at stage 2 [since February of 2019](https://tc39.es/proposal-decorators/).

We recommend avoiding decorators until they're at least a stage 3 ("candidate") proposal, or stage 4 ("finished") for more conservative teams.

There's always a chance that ECMAScript decorators will never be finalized. If that happens, they'll end up in a similar situation to TypeScript enums and namespaces. They'll continue to break TypeScript's type-level extension rule forever, and they'll be more likely to break when using build tools other than the official TypeScript compiler. We don't know whether that will happen or not, but the benefits of decorators are minor enough that we'd rather wait and see.

Some open source libraries, most notably [TypeORM](https://typeorm.io/), use decorators heavily. We recognize that following our recommendation here precludes using TypeORM. Using TypeORM and its decorators is a fine choice, but it should be done intentionally, recognizing that decorators are currently in standardization purgatory and may never be finalized.

**IV. Avoid the private keyword ([See our lesson](/courses/typescript/lessons/classes))**

TypeScript has two ways to make class fields private. There's the old `private` keyword, which is specific to TypeScript. Then there's the new `#somePrivateField` syntax, which is taken from JavaScript. Here's an example showing each of them:

```
class MyClass {
  private field1: string;
  #field2: string;
  ...
}
```

We recommend the new `#somePrivateField` syntax for a straightforward reason: these two features are roughly equivalent. We'd like to maintain feature parity with JavaScript unless there's a compelling reason not to.

To recap our four recommendations:

1.  Avoid enums.
2.  Avoid namespaces.
3.  Favor `#somePrivateField` over `private somePrivateField`.
4.  Hold off on using decorators until they're standardized. If you really need a library that requires them, consider their standardization status when making that decision.

Even when avoiding these features, it's good to have a working knowledge of them. They show up often in legacy code, and even in some new code. Not everyone agrees that they should be avoided. Execute Program's TypeScript courses teach these features for the reasons that we explained in our post on [Teaching the Unfortunate Parts](/blog/teaching-the-unfortunate-parts).

This post was written by the Execute Program team. Execute Program teaches TypeScript, JavaScript, Regular Expressions, SQL, and more using thousands of interactive code examples. It has an integrated spaced repetition system to ensure that you don't forget what you've learned!

[Try Execute ProgramRight Arrow Icon](/)