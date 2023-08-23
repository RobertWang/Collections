> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [engineering.desmos.com](https://engineering.desmos.com/articles/pratt-parser/)

> When you read an expression, like 1/2+3.4, you can immediately understand some of its meaning. You re......

When you read an expression, like `1/2+3.4`, you can immediately understand some of its meaning. You recognize that there are three numbers, and that the numbers are combined with operators. You may also recall that division has higher precedence than addition, so you would divide `1/2` before adding `+3.4` when evaluating the expression.

Compare this to how you perceive `2H3SGKHJD`. At first glance it appears to be a nonsense sequence of characters. If I told you that letters should be grouped in pairs with `G` being a separator, your mental model might look closer to `2H 3S ; KH JD`, which takes us a step towards understanding that this string represents hands in a card game.

Parsing is the process of taking a string of characters and converting them into an Abstract Syntax Tree (or, AST). This is a representation of the structure of the expression, for example:

```
OperatorNode(
    operator: "+",
    left: OperatorNode(
        operator: "/",
        left: 1,
        right: 2
    ),
    right: 3.4
)


```

or, as a diagram,

```
         "+"
       /     \
    "/"        3.4
  /     \
1         2


```

Such a tree is a first step towards computing the value of the expression, or rendering it beautifully.

![](https://engineering.desmos.com/articles/pratt-parser/mathquill.png)

So, how does one create an AST? At Desmos we use the approach described by [Vaughan Pratt](https://web.archive.org/web/20151223215421/http://hall.org.ua/halls/wizzard/pdf/Vaughan.Pratt.TDOP.pdf). This article will begin with what is hopefully a clear and concise explanation of how Pratt Parsing works. We will then explain our motivations for adopting this technique at Desmos and compare it to the jison parser generator, our previous approach.

Finally, we provide a [sample implementation of the parser (and a lexer) in Typescript](https://github.com/desmosinc/pratt-parser-blog-code), integrated with [CodeMirror](https://codemirror.net/). We hope this will be a useful reference and starting point for anyone interested in doing parsing in the browser.

Our parse function will operate over a `tokens` object. This is a sequence of tokens, like `[1, "/", 2, "+", 3.4]` that is generated from our input through a process called lexing. We will not go into the details of lexing here, other than to point you at [our sample implementation](https://github.com/desmosinc/pratt-parser-blog-code/blob/master/src/lexer.ts).

The `tokens` object is a token stream, which allows us to `consume` a token, returning the next token and advancing the stream. We can also peek a token, which gives us the next token without advancing the stream.

```
[1, "/", 2, "+", 3.4].consume() -> 1, ["/", 2, "+", 3.4]
[1, "/", 2, "+", 3.4].peek() -> 1, [1, "/", 2, "+", 3.4]

[].consume() -> empty token, []
[].peek() -> empty token, []


```

Let’s start with a recursive call and fill things out as we go along. We will present our approach in pseudocode, but you are welcome to reference the Typescript implementation as we go along.

```
function parse(tokens):
    firstToken = tokens.consume()

    if firstToken is a number
        leftNode = NumberNode(firstToken)
    otherwise
        Error("Expected a number")

    nextToken = tokens.consume()

    if nextToken is an operator token
        rightNode = parse(tokens)

        return OperatorNode(
            operator: nextToken,
            leftNode,
            rightNode
        )

    otherwise
        return leftNode


```

We expect a number token followed by an optional operator. We then perform a recursive call to find the sub-expression to the right. So far so good – we start getting an idea of how parsing an expression like `3 * 2 + 1` might work:

```
parse [3, "*", 2, "+", 1]
1 consume 3 as a NumberNode into leftNode
1 consume "*" to be nextToken
1 rightNode = parse [2, "+", 1]
    2 consume 2 as a NumberNode into leftNode
    2 consume "+" to be nextToken
    2 rightNode = parse [1]
        3 consume 1    as a NumberNode into leftNode
        3 no next token
        3 return:
            1
    2 combine 2, "+" and 1 into an OperatorNode
    2 return:
            "+"
          /     \
        2         1

1 combine 3, "*" and rightNode into an OperatorNode
1 return:
    "*"
  /     \
3        "+"
       /     \
     2         1


```

Precedence
----------

If we were to evaluate this expression, we would add `2 + 1` first, and then multiply the result of that sub-tree by `3`, to get `9`. This is not desirable, since conventionally multiplication has higher precedence than addition, and we would like the tree to look like this instead:

```
         "+"
       /     \
    "*"        1
  /     \
3         2


```

Pratt represents this idea with the term _binding power_. Multiplication has a higher binding power than addition, and so the `3 * 2` in the expression above takes precedence. When we perform the recursive call to parse `2 + 1`, we are looking for the node that represents the right side of our product. So, when we see “+”, we want to stop since it binds less strongly than “*”. Let’s add this to our code, noting that this is still incomplete and we will improve things as we go along:

```
function parse(tokens, lastOperator):
    firstToken = tokens.consume()

    if firstToken is a number
        leftNode = NumberNode(firstToken)
    otherwise
        Error("Expected a number")

    nextToken = tokens.peek()

    if nextToken is an operator and greaterBindingPower(nextToken, lastOperator)
        tokens.consume()
        rightNode = parse(tokens, nextToken)
        return OperatorNode(
            operator: nextToken,
            leftNode,
            rightNode
        )

    otherwise
        return leftNode


```

Let’s consider how this changes the execution of parsing `3 * 2 + 1`:

```
parse [3, "*", 2, "+", 1], empty token
1 consume 3 as a NumberNode into leftNode
1 peek "*" to be nextToken
1 greaterBindingPower("*", empty token) is true by default, so continue
1 consume "*"
1 parse [2, "+", 1], "*"
    2 consume 2 as a NumberNode into leftNode
    2 peek "+" to be nextToken
    2 greaterBindingPower("+", "*") is false
    2 return:
        2
1 combine 3, "*" and 2 into an OperatorNode
1 return:
    "*"
  /     \
3         2


```

Accumulating on the left
------------------------

As desired, our recursive call stopped before `+` when parsing the sub-expression `2 + 1`. This allowed us to correctly combine `3 * 2` into a product node in the outer call. However, the computation halted prematurely, and we left `+ 1` unprocessed. Let’s remedy this now:

```
function parse(tokens, lastOperator):
    firstToken = tokens.consume()

    if firstToken is a number
        leftNode = NumberNode(firstToken)
    otherwise
        Error("Expected a number")

    repeat
        nextToken = tokens.peek()

        if nextToken is an operator and greaterBindingPower(nextToken, lastOperator)
            tokens.consume()
            rightNode = parse(tokens, nextToken)
            leftNode = OperatorNode(
                operator: nextToken,
                leftNode,
                rightNode
            )
        otherwise
            return leftNode


```

This yields the following computation:

```
parse [3, "*", 2, "+", 1], empty token
1 consume 3 as a NumberNode into leftNode
1 peek "*" to be nextToken
1 greaterBindingPower(*, empty token) is true by default, so continue
1 consume "*"
1 rightNode = parse [2, "+", 1], "*"
    ... returns 2, as above
1 combine 3, "*" and 2 into an OperatorNode, and store it in leftNode

leftNode:
    "*"
  /     \
3         2

1 repeat
1 peek "+" to be nextToken
1 greaterBindingPower(+, empty token) is true by default
1 consume +
1 parse [1], '+'
    ... returns 1
1 combine leftNode, "+" and 1 into an OperatorNode, and store it in leftNode

leftNode:
         "+"
       /     \
    "*"        1
  /     \
3         2

1 repeat
1 nextToken is empty, so return leftNode


```

We now correctly group the `3 * 2` sub-expression as an OperatorNode within our AST!

Synthesis
---------

We can now see how the binding power guides us to make the right groupings while building our tree. As long as the operators we encounter have higher binding power, we continue to make recursive calls, which builds up our expression on the right hand side of the tree. When we encounter an operator with a lower binding power, we propagate the result up the call chain until we reach the level where the binding power is sufficient to continue grouping. There, we transfer our accumulated term into `leftNode`, and resume building up the right hand side of the expression.

In other words, while the binding power is higher than our context, we associate to the right using the recursive call. When it is lower, we associate to the left using the repeat loop. This is really the crux of understanding how Pratt parsers work, so it’s worth taking a minute to walk yourself through the execution of something like `3 + 4 * 2 ^ 2 * 3 - 1` to get a feel for it.

Associativity and binding power
-------------------------------

One thing that we haven’t explicitly mentioned yet is operator associativity. Some operators, like addition and subtraction are left-associative, meaning that when we apply them repeatedly, `3 - 2 - 1`, we associate to the left `(3 - 2) - 1`. Others, like exponentiation associate to the right, so `2 ^ 3 ^ 4` is the same as `2 ^ (3 ^ 4)`.

Hopefully the exposition so far makes it clear how we can implement this using our `greaterBindingPower` function. We want left-associative operators to stop recursion when they encounter the same operator. So, `greaterBindingPower(-, -)` should be false. On the other hand, we want to continue recursing when the operator is right-associative, so `greaterBindingPower(^, ^)` should be true.

In practice, this behavior is implemented by assigning to each operator class a binding power number. So for instance,

```
+, -: 10
*, /: 20
^: 30


```

We pass this number into the parse function, and lookup the binding power of the next token to make our decisions. Right-associative operators are implemented by subtracting `1` from their binding power when making the recursive call.

```
parse(tokens, currentBindingPower):
    ...
    repeat
        ...
        nextToken = tokens.peek()
        if bindingPower[nextToken] <= currentBindingPower
            stop repeating

        nextBindingPower = if nextToken is left-associative then bindingPower otherwise bindingPower - 1
        rightNode = parse(tokens, nextBindingPower)
    ...


```

Extending the grammar
---------------------

So far, we can parse numbers and binary operators of the form `<expression> <operator> <expression>`, but we may have to deal with other forms, like `( <expression> )`, `log <expression>`, or even `if <expression> then <expression> otherwise <expression>`.

We have at our disposal the parse call which can give us a sub-expression that binds stronger than a given context. With this, we can parse these different forms in an elegant, readable way. For example, to parse an expression contained in a pair of braces,

```
...
currentToken = tokens.consume()
if currentToken is "("
    expression = parse(tokens, 0) // Note, reset the binding power to 0 so we parse a whole subexpression
    next = tokens.consume()
    if next is not ")"
        Error("Expected a closing brace")
...


```

We can combine these concepts - the parsing of a sub-expression, the adjustment of the binding power passed to the recursive call, the left/right associativity, and error handling into a unit called a _Parselet_.

We have two places in our code where parselets may be called. The first is the one between expressions that we have spent some time looking at (in Pratt parlance, this is referred to as “led”). The other is at the beginning of a new expression (in Pratt’s paper, “nud”). Currently we handle number tokens there, converting them to number nodes. This nicely abstracts into a parselet - one that converts a single token into a node and doesn’t perform any recursive calls to parse sub-expressions. This is also where the above code for parsing braces would go.

In the sample code, we identify these as `initialParselet` and `consequentParselet`. Each set of parselets are stored in a map, keyed by the token type that identifies the parselet.

```
initialPareselets = {
    "number": NumberParselet,
    "(": ParenthesisParselet,
    ...
}

NumberParselet:
    parse(tokens, currentToken):
        return NumberNode(currentToken)


consequentParselets = {
    "+": OperatorParselet("+"),
    "-": OperatorParselet("-"),
    ...
}

OperatorParselet(operator):
    parse(tokens, currentToken, leftNode):
        myBindingPower = bindingPower[operator]
        rightNode = parse(tokens, if operator is left associative then myBindingPower otherwise myBindingPower - 1)

        return OperatorNode(
            operator,
            leftNode,
            rightNode
        )


```

Putting it all together
-----------------------

With the above changes, we get the following pseudocode for our completed parse function:

```
parse(tokens, currentBindingPower):
    initialToken = tokens.consume()
    initialParselet = initialMap[initialToken.type]
    if we didn't find a initialParselet
        Error("Unexpected token")

    leftNode = initialParselet.parse(tokens, initialToken)
    repeat
        nextToken = tokens.peek()
        if nextToken is empty
            stop repeating

        consequentParselet = consequentMap[nextToken]
        if we didn't find an consequentParselet
            stop repeating

        if bindingPower[nextToken] <= currentBindingPower
            stop repeating

        tokens.consume()
        leftNode = consequentParselet.parse(tokens, leftNode, nextToken)

    return leftNode


```

Or, see the [reference implementation](https://github.com/desmosinc/pratt-parser-blog-code/blob/master/src/parser.ts#L66) in Typescript.

Before moving to Pratt parsers, we were using [jison](https://github.com/zaach/jison). For those unfamiliar, jison is a javascript implementation of the bison parsor generator. In jison, you specify a grammar, like:

```
an expression is a number or an operation
an operation is a sum, difference, product, or quotient
a sum is <expression> + <expression>

etc...


```

jison takes such a description and spits out a javascript program that is able to parse that grammar. The great thing about this is that you only need to worry about declaring the grammar, and all of the implementation is handled for you! This, combined with the fact that some of our engineers were familiar with similar approaches, made jison an easy choice for our initial implementation.

However, over time we found several issues that convinced us to look for alternatives:

Error messages
--------------

If the user typed in an expression that didn’t satisfy our grammar, say by forgetting to close a parenthesis or populate an exponent, our jison implementation was only able to inform us that the whole expression was malformed. As you can imagine, this is a frustrating experience for students and teachers.

![](https://engineering.desmos.com/articles/pratt-parser/sorry_i_dont_understand.jpg)

In jison it is possible to customize errors by anticipating incorrect patterns in your grammar. This approach has two significant drawbacks, however. First, it is opt-in, meaning that you can never quite be sure that you’ve covered all possible syntax errors of your grammar. Second, it complicates your grammar, making it much harder to reason about completeness and correctness, thus cancelling one of the main advantages of using parser generators in the first place.

The Pratt parser approach, on the other hand, naturally encourages you to think about edge cases as you write each parselet. It also made it very straightforward to capture the context of the error for consumption in external code. This allowed us to highlight the location of the error in the editor easily. We also took advantage of this to create a very robust autocomplete system (a topic for a future post).

![](https://engineering.desmos.com/articles/pratt-parser/cl_error.png)

Developer Friendliness
----------------------

Previously, working on parser internals required one to get familiar with the jison specification language, as well as the surrounding tooling for generating and testing parsers. Now, our implementation is written in Typescript – the same language our team uses every day. This makes the parser code accessible to everyone on the team, especially since the implementation is readable and concise. Furthermore, changes can be made with confidence since all members of the team are comfortable reviewing the code.

Code reuse and type safety
--------------------------

Previously, we had to maintain two lexers - one that was compatible with jison, and another to perform syntax highlighting in CodeMirror. By adapting Pratt parsing, we were able to build our parsing pipeline on top of the same interface that CodeMirror uses, thus getting rid of that duplication. Furthermore, our code is now Typescript throughout, which means we get thorough type checking both inside the implementation and at the boundaries with other code.

Speed + Bundle Size
-------------------

The parser implementation required many more lines of code than specifying the grammar in jison. However, when jison generates the parsing program, it expands the grammar into very large transition tables. The result is that we actually sent ~20KB to the client, which was cut down to ~10KB with the new implementation. Furthermore, tested over 100k calculator expressions, the Pratt parser ended up being about 4 times faster than the jison implementation. We think (although we haven’t verified) that this is because the transition table generated by jison is too big to keep in the cache, while browsers are quite good at optimizing recursive function calls.

There are several disadvantages to using a Pratt parser that we have discovered that may be useful to you.

Recursion and Memory
--------------------

Because we rely on recursive function calls, it is possible that your parser may run out of space on the call stack for deeply nested expressions, like `1^1^1^1...`. You could mitigate this by keeping track of the depth of the expression while parsing and throwing a custom “This expression is nested too deeply” error. Another strategy is to move the parsing stack into the heap, either by managing the parser state yourself or using something like trampolining.

Correctness and Performance Complexity
--------------------------------------

There is a lot of tooling for parser generators and grammars. For example, you could analyze your grammar and make guarantees about the correctness or performance characteristics of the parser. Because the Pratt parser is just code, there is always the danger of introducing inefficiencies. Developers may be tempted to solve tricky parsing situations by trying several parsing paths, which can easily cause exponential complexity. Unfortunately, the solution here is to be careful. Even with code review and thorough testing, you can never have a guarantee that your parser won’t crash on some inputs.

Our primary motivation for moving to Pratt parsers was flexibility. It allowed us to show helpful and localized error messages, which significantly improved the experience of users on our site. We hope this article will help you choose the right approach, and is a good starting point if you choose to use Pratt parsers in your project.

Citations
---------

In the process of getting up to speed on Pratt parsers, we found the following articles incredibly helpful, and you may too:

*   This [series of blog posts](http://www.oilshell.org/blog/2017/03/31.html) by Andy Chu.
*   this [tutorial on pratt parsing](https://dev.to/jrop/pratt-parsing) by Jonathan Apodaca.
*   this [tutorial on pratt parsing](http://journal.stuffwithstuff.com/2011/03/19/pratt-parsers-expression-parsing-made-easy/) by Bob Nystrom.
*   this [tutorial on Top-Down operator precedence parsing](https://eli.thegreenplace.net/2010/01/02/top-down-operator-precedence-parsing) by Eli Bendersky
*   this [tutorial on creating a JavaScript parser](https://crockford.com/javascript/tdop/tdop.html) by Douglas Crockford