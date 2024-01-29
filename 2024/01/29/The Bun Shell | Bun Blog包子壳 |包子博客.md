> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [bun.sh](https://bun.sh/blog/the-bun-shell)

> The Bun Shell is a cross-platform shell that allows you to run shell scripts in JavaScript & TypeScri......Bun Shell是一个跨平台的shell，允许你在JavaScript和TypeScri中运行shell脚本......

JavaScript is the world's most popular scripting language.  
JavaScript 是世界上最流行的脚本语言。

So why is it hard to run shell scripts in JavaScript?  
那么，为什么在 JavaScript 中运行 shell 脚本很困难呢？

```
import { spawnSync } from "child_process";

// this is a lot more work than it could be
const { status, stdout, stderr } = spawnSync("ls", ["-l", "*.js"], {
  encoding: "utf8",
});


```

You could use APIs to do something similar:  
您可以使用 API 执行类似操作：

```
import { readdir } from "fs/promises";

(await readdir(".", { withFileTypes: true })).filter((a) =>
  a.name.endsWith(".js"),
);


```

But, that's still not quite as simple as a shell script:  
但是，这仍然不像 shell 脚本那么简单：

[Why existing shells don't work in JavaScript  
为什么现有的 shell 在 JavaScript 中不起作用](#why-existing-shells-don-t-work-in-javascript)
-------------------------------------------------------------------------------------------------------------------------------

Shells like `bash` or `sh` have been around for decades.  
贝壳喜欢 `bash` 或 `sh` 已经存在了几十年。

Shells are a solved problem!!  
贝壳是一个已解决的问题！！

*   Hacker News Commenter, probably.  
    黑客新闻评论员，可能。

But, they don't work well in JavaScript. Why?  
但是，它们在 JavaScript 中效果不佳。为什么？

macOS (zsh), Linux (bash), and Windows (cmd) all have slightly different shells with different syntaxes and different commands. The commands available on each platform are different, and even the same command can have different flags and behaviors.  
macOS （zsh）、Linux （bash） 和 Windows （cmd） 的 shell 都略有不同，具有不同的语法和不同的命令。每个平台上可用的命令都不同，即使相同的命令也可能具有不同的标志和行为。

To date, npm's solution is to rely on the community to polyfill missing commands with JavaScript implementations.  
到目前为止，npm 的解决方案是依靠社区用 JavaScript 实现来填充缺失的命令。

### [`rm -rf` doesn't work on Windows  
`rm -rf` 在 Windows 上不起作用](#rm-rf-doesn-t-work-on-windows)

`rimraf`, the cross-platform JavaScript implementation of `rm -rf`, is downloaded 60 million times per week:  
`rimraf` ，的跨平台 JavaScript 实现 `rm -rf` ，每周下载量达 6000 万次：

[![](https://github.com/oven-sh/bun/assets/709451/a201e61c-2131-4982-9be6-ffdf2a37db7e)](https://github.com/oven-sh/bun/assets/709451/a201e61c-2131-4982-9be6-ffdf2a37db7e)

### [Environment variables like `FOO=bar <script>` doesn't work on Windows  
环境变量（如 `FOO=bar <script>` ）在 Windows 上不起作用](#environment-variables-like-foo-bar-script-doesn-t-work-on-windows)

Setting an environment variable is different on each platform. Instead of running `FOO=bar`, you probably use & install `cross-env`:  
在每个平台上设置环境变量是不同的。而不是运行 `FOO=bar` ，你可能使用 & install `cross-env` ：

[![](https://github.com/oven-sh/bun/assets/709451/ab42d9b3-0fa7-4abe-beae-ca26a48e9d75)](https://github.com/oven-sh/bun/assets/709451/ab42d9b3-0fa7-4abe-beae-ca26a48e9d75)

### [`which` is `where` on Windows  
`which` 在 `where` Windows 上](#which-is-where-on-windows)

Thus, another package with 60 million weekly downloads was born:  
于是，另一个每周下载量达到 6000 万次的软件包诞生了：

[![](https://github.com/oven-sh/bun/assets/709451/e6659440-f322-47a8-99fe-ec9e3bec104e)](https://github.com/oven-sh/bun/assets/709451/e6659440-f322-47a8-99fe-ec9e3bec104e)

### [Shells also take too long to start  
Shell 启动时间也太长](#shells-also-take-too-long-to-start)

How long does it take to spawn a shell?  
生成一个贝壳需要多长时间？

On a Linux x64 Hetzner Arch Linux machine, it takes about 7ms:  
在 Linux x64 Hetzner Arch Linux 机器上，大约需要 7 毫秒：

```
$ hyperfine --warmup 3 'bash -c "echo hello"' 'sh -c "echo hello"' -N

Benchmark 1: bash -c 'echo hello'
  Time (mean ± σ):       7.3 ms ±   1.5 ms    [User: 5.1 ms, System: 1.9 ms]
  Range (min … max):     1.7 ms …   9.4 ms    529 runs

Benchmark 2: sh -c 'echo hello'
  Time (mean ± σ):       7.2 ms ±   1.6 ms    [User: 4.8 ms, System: 2.1 ms]
  Range (min … max):     1.5 ms …   9.6 ms    327 runs


```

If your intent is to run a single command, starting the shell can take longer than running the command itself. If you're running many commands in a loop, that gets expensive quickly.  
如果您的意图是运行单个命令，则启动 shell 所需的时间可能比运行命令本身的时间更长。如果你在一个循环中运行许多命令，那很快就会变得昂贵。

You could try embedding a shell, but that's really complicated and their license may not be compatible with your project.  
您可以尝试嵌入 shell，但这真的很复杂，而且它们的许可证可能与您的项目不兼容。

[Are all these polyfills really necessary?  
所有这些填充物真的有必要吗？](#are-all-these-polyfills-really-necessary)
-------------------------------------------------------------------------------------------------------

In the world of 2009 - 2016, when JavaScript was still relatively new and experimental, relying on the community to polyfill missing functionality made a lot of sense. But it's 2024 now. JavaScript on the server is mature and widely adopted. The JavaScript ecosystem understands the requirements today in a way nobody did in 2009.  
在 2009 年至 2016 年的世界里，当 JavaScript 还处于相对较新的实验阶段时，依靠社区来填充缺失的功能是很有意义的。但现在是 2024 年。服务器上的 JavaScript 已经成熟并被广泛采用。JavaScript 生态系统以 2009 年没有人能做到的方式理解当今的需求。

We can do better. 我们可以做得更好。

[Introducing the Bun Shell  
介绍包子壳](#introducing-the-bun-shell)
---------------------------------------------------------------

The Bun Shell is a new experimental embedded language and interpreter in Bun that allows you to run cross-platform shell scripts in JavaScript & TypeScript.  
Bun Shell 是 Bun 中一种新的实验性嵌入式语言和解释器，它允许您在 JavaScript 和 TypeScript 中运行跨平台的 shell 脚本。

```
import { $ } from "bun";

// to stdout:
await $`ls *.js`;

// to string:
const text = await $`ls *.js`.text();


```

You can use JavaScript variables in your shell scripts:  
您可以在 shell 脚本中使用 JavaScript 变量：

```
import { $ } from "bun";

const resp = await fetch("https://example.com");

const stdout = await $`gzip -c < ${resp}`.arrayBuffer();


```

For security, all **template variables are escaped**:  
为了安全起见，所有模板变量都已转义：

```
const filename = "foo.js; rm -rf /";

// This will run `ls 'foo.js; rm -rf /'`
const results = await $`ls ${filename}`;

console.log(results.exitCode); // 1
console.log(results.stderr.toString()); // ls: cannot access 'foo.js; rm -rf /': No such file or directory


```

Using Bun Shell feels like regular JavaScript. You can redirect stdout to buffers:  
使用 Bun Shell 感觉就像普通的 JavaScript。您可以将 stdout 重定向到缓冲区：

```
import { $ } from "bun";

const buffer = Buffer.alloc(1024);

await $`ls *.js > ${buffer}`;

console.log(buffer.toString("utf8"));


```

You can redirect stdout to a file:  
您可以将 stdout 重定向到文件：

```
import { $, file } from "bun";

// as a file()
await $`ls *.js > ${file("output.txt")}`;

// or as a file path string, if you prefer:
await $`ls *.js > output.txt`;
await $`ls *.js > ${"output.txt"}`;


```

You can pipe stdout to another command:  
您可以将 stdout 通过管道传递给另一个命令：

```
import { $ } from "bun";

await $`ls *.js | grep foo`;


```

You can even use `Response` as stdin:  
您甚至可以用作 `Response` stdin：

```
import { $ } from "bun";

const buffer = new Response("bar\n foo\n bar\n foo\n");

await $`grep foo < ${buffer}`;


```

Builtin commands like `cd`, `echo`, and `rm` are available:  
内置命令（如 `cd` 、 `echo` 和 ） `rm` 可用：

```
import { $ } from "bun";

await $`cd .. && rm -rf node_modules/rimraf`;


```

It works on Windows, macOS, and Linux. We've implemented many common commands and features like globbing, environment variables, redirection, piping, and more.  
它适用于 Windows、macOS 和 Linux。我们实现了许多常见的命令和功能，如通配、环境变量、重定向、管道等。

It's designed as a drop-in replacement for simple shell scripts. In Bun for Windows, it will power `package.json` "scripts" in `bun run`.  
它被设计为简单 shell 脚本的直接替代品。在 Bun for Windows 中 `bun run` ，它将为 `package.json` .

For fun, you can also use it as a standalone shell script interpreter:  
为了好玩，您还可以将其用作独立的 shell 脚本解释器：

```
echo "cat package.json" > script.bun.sh

```

### [How do I install it?  
如何安装？](#how-do-i-install-it)

**Bun Shell is built into Bun**. If you already have Bun v1.0.24 or later installed, you can use it today:  
Bun Shell 内置于 Bun 中。如果您已经安装了 Bun v1.0.24 或更高版本，您可以立即使用它：

If you don't have Bun installed, you can install it with curl:  
如果你没有安装 Bun，你可以用 curl 安装它：

```
curl -fsSL https://bun.sh/install | bash

```

Or with npm: 或者使用 npm：