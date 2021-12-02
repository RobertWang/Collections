> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6976891381914533918)

本文主要解决以下问题：

1.  如何在代码提交时进行代码规范检查（lint-staged）？
2.  如何使用工具生成符合 Angular commit 规范的 message？
3.  如何通过 gitHooks 验证 message 是否符合规范？
4.  如何使用工具自动生成 changelog？

此文对应的 github 代码仓库：[github.com/eyunhua/cod…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Feyunhua%2Fcode-commit-check "https://github.com/eyunhua/code-commit-check")

**如果你想了解更详细的过程，请认真往下阅读；如果你想快速查看如何使用，请看『总结』目录。**

前言
==

在一个多人协作的项目中，简洁清晰易懂的代码提交注释是能够快速定位问题的有效方式。commit message 应该说明本次提交的目的。

**对比下面这两张代码提交 commit log 图：**

第一张：message 没有实际的意义，看不出来本次提交改动的范围。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/00e4c3275bcf444aa9219c1a854b595b~tplv-k3u1fbpfcp-watermark.awebp)

第二张：message 简洁清晰易懂，可以看出当前是新增功能、修复问题等，并可看出此次提交影响的范围。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e28bfd933844db8b7336f9d1309f598~tplv-k3u1fbpfcp-watermark.awebp)

通过上面两张图可以看出，message 写的符合规范是多么的重要！

那我们在日常工作中，仅仅通过人工去输入 message，无法完全保证 message 提交的规范性。

**接下来介绍一种社区比较流行的规范 [Angular 规范](https://link.juejin.cn?target=https%3A%2F%2Fdocs.google.com%2Fdocument%2Fd%2F1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y%2Fedit%23heading%3Dh.greljkmo14y0 "https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#heading=h.greljkmo14y0")，并且通过工具去生成符合这种规范的 message 信息。**

Angular commit 规范
=================

[介绍](https://link.juejin.cn?target=https%3A%2F%2Fdocs.google.com%2Fdocument%2Fd%2F1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y%2Fedit%23heading%3Dh.greljkmo14y0 "https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#heading=h.greljkmo14y0")

规范格式概览
------

```
<type>(<scope>): <subject> #header部分
// 空一行
<body>
// 空一行
<footer> 
复制代码

```

**Header 是必需的，Body 和 Footer 可以省略。**

规范格式详解
------

### Header

Header 部分只有一行，包括三个字段：type（必需）、scope（可选）和 subject（必需）。

#### type

> 用于说明本次 commit 的类型

只允许使用下面 7 个标识

*   feat：新功能（feature）
*   fix：修补 bug
*   docs：文档（documentation）
*   style： 格式（不影响代码运行的变动）
*   refactor：重构（即不是新增功能，也不是修改 bug 的代码变动）
*   perf: 性能提升（提高性能的代码改动）
*   test：测试
*   build：构建过程或辅助工具的变动（webpack 等）
*   ci：更改 CI 配置文件和脚本
*   chore：不修改 src 或测试文件的其他更改
*   revert：撤退之前的 commit

**如果 type 为 feat、fix、perf、revert，则该 commit 将肯定出现在 Change log 之中。其他情况（docs、chore、style、test）由你决定，要不要放入 Change log，建议是不要。**

#### scope

> 用于说明本次 commit 影响的范围，比如首页、详情页等。

#### subject

> 用于说明本次 commit 的简短描述，不超过 50 个字符。

### Body

> 用于本次 commit 的详细描述，可以分成多行

举例（实际应用应说明具体改动）：

```
more info... 

- first changes...
- second changes...
复制代码

```

### Footer

只用于下面两种情况

#### 不兼容变动

> 如果当前代码与上一个版本不兼容，则 Footer 部分以 BREAKING CHANGE 开头，后面是对变动的描述、以及变动理由和迁移方法。

举例：

vue-next changelog.md：[github.com/vuejs/vue-n…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue-next%2Fcommit%2Fe58beecc97635ea61e39b84ea406fcc42166095b "https://github.com/vuejs/vue-next/commit/e58beecc97635ea61e39b84ea406fcc42166095b")

```
BREAKING CHANGE: `getTextMode` compiler option signature has changed from

  ``ts
  (tag: string, ns: string, parent: ElementNode | undefined) => TextModes
  ``

  to

  ``ts
  (node: ElementNode, parent: ElementNode | undefined) => TextModes
  ``
复制代码

```

#### 关闭 Issue

> 如果当前 commit 针对某个 issue，那么可以在 Footer 部分关闭这个 issue 。

```
Closes #123, #234
复制代码

```

### Revert（可忽视）

> 还有一种特殊情况，如果当前 commit 用于撤销以前的 commit，则必须以 revert: 开头，后面跟着被撤销 Commit 的 Header。

```
revert: feat(pencil): add 'graphiteWidth' option

This reverts commit 667ecc1654a317a13331b17617d973392f415f02.
复制代码

```

Body 部分的格式是固定的，必须写成 This reverts commit .，其中的 hash 是被撤销 commit 的 SHA 标识符。

如果当前 commit 与被撤销的 commit，在同一个发布（release）里面，那么它们都不会出现在 Change log 里面。如果两者在不同的发布，那么当前 commit，会出现在 Change log 的 Reverts 小标题下面。

[代码规范检查：lint-staged](https://link.juejin.cn?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Flint-staged "https://www.npmjs.com/package/lint-staged")
===================================================================================================================================================

> 检测暂存区的代码是否符合规范

介绍
--

在代码提交之前，进行代码规则检查能够确保进入`git`库的代码都是符合代码规则的。但是整个项目上运行`lint`速度会很慢，`lint-staged`能够让`lint`只检测暂存区的文件，所以速度很快。

Install
-------

```
npx mrm@2 lint-staged
复制代码

```

此命令将根据项目的`package.json`依赖项中的代码质量工具安装和配置`husky`和`lint staged`，因此，请确保在此之前安装（npm install—save dev）并配置所有代码质量工具，如`Prettier`和`ESLint`。

运行完上述命令后，将会在`package.json`中出现：

```
"lint-staged": {
    "*.{js,vue}": [ // 对不同的后缀文件，运行不同的检查命令
      "vue-cli-service lint",
      "git add"
    ],
    "*.{ts}": [ // 对不同的后缀文件，运行不同的检查命令
      "xxx"
    ]
}
复制代码

```

husky 中使用
---------

在`.husky`目录下的`pre-commit`钩子脚本中，添加如下脚本：

```
npx lint-staged
复制代码

```

yorkie 中使用
----------

在`package.json`中，添加如下脚本：

```
{
    "gitHooks": {
        "pre-commit": "lint-staged"
    }
}
复制代码

```

**配置完脚本后，在我们执行`git commit -m 'xxx'`提交代码时，就会自动触发`pre-commit`钩子进行代码规范检查。**

commit-msg 检查方式一：脚本
===================

`package.json`文件：

```
"gitHooks": {
    "commit-msg": "node build/verify-commit-msg.js"
}
复制代码

```

`build/verify-commit-msg.js`文件：

> 返回非 0 值则退出

```
/* eslint-disable */
const chalk = require("chalk");
const msgPath = process.env.GIT_PARAMS;
const msg = require("fs")
    .readFileSync(msgPath, "utf-8")
    .trim();
// ([A-Za-z0-9]+-[A-Za-z0-9]+)+ 
const commitRE = /^(build|chore|ci|docs|feat|fix|wip|perf|refactor|revert|style|test|temp|)(\(.+\))?: .{1,50}/;
if (!(commitRE.test(msg) || msg.indexOf("Merge") === 0)) {
    console.error(
        `  ${chalk.bgRed.white(" ERROR ")}
  [${chalk.red(msg)}] 是 ${chalk.red("无效的提交消息格式")}
  ${chalk.red("自动生成更新日志需要正确的提交消息格式 例如:")}

  ${chalk.green("issue-1 feat(模块): 预发布环境增加 A 模块")}
  ${chalk.green("issue-2 fix(文案): 修复错误文案")}`
    );
    process.exit(1);  // 返回非0值得退出
}
/* eslint-enable */

复制代码

```

[commit-msg 检查方式二：commitlint](https://link.juejin.cn?target=https%3A%2F%2Fcommitlint.js.org%2F%23%2F "https://commitlint.js.org/#/")
====================================================================================================================================

> 检查`git commit -m 'xxx'`的 message 是否符合规范

Install
-------

项目内安装`@commitlint/cli`、`@commitlint/config-conventional`：

```
npm install --save-dev @commitlint/cli @commitlint/config-conventional
复制代码

```

Configure
---------

可以在`commitlint.config.js`、`.commitlintrc.js`、`.commitlintrc.json`或`.commitlintrc.yml`文件或`package.json`中的`commitlint`字段中定义配置。

当前以`commitlint.config.js`为例：

```
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
复制代码

```

在 husky 中使用
-----------

添加`.husky/commit-msg`钩子，内容为`npx --no-install commitlint --edit $1`：

```
npx husky add .husky/commit-msg 'npx --no-install commitlint --edit $1'
复制代码

```

在 yorkie 中使用
------------

`package.json`：

```
"gitHooks": {
    "commit-msg": "commitlint --edit"
 }
复制代码

```

Test
----

经过安装`commitlint`、`commitlint.config.js`配置项，并且在 hooks 钩子中配置之后，就可以在提交时校验`message`是否符合规范了。

输入不符合规范的 message，将会出现如下的错误提示： ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89e29829bb4e4ba0ba3e15c24e035741~tplv-k3u1fbpfcp-watermark.awebp)

**仅仅通过人工输入 message，显然是不可靠的，`commitlint`提供了一个可以生成 message 的工具`@commitlint/prompt-cli`**。

生成 message 方式一：commitlint
=========================

**`commitlint`为`comitizen`提供了两种方式：**

### 方式一：[@commitlint/prompt-cli](https://link.juejin.cn?target=https%3A%2F%2Fcommitlint.js.org%2F%23%2Fguides-use-prompt%3Fid%3Dguide-use-prompt "https://commitlint.js.org/#/guides-use-prompt?id=guide-use-prompt")

#### Install

```
npm install --save-dev @commitlint/prompt-cli
复制代码

```

#### Usage

在`package.json`中加入一个`npm run script`

```
"scripts": {
    "commit": "commit"
}
复制代码

```

#### 执行

```
git add .
npm run commit
复制代码

```

运行`npm run commit`将会出现如下图：

根据提示一步一步输入 commit message ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c57e3518f7f141bbb2982561c5d81b91~tplv-k3u1fbpfcp-watermark.awebp)

### 方式二：[@commitlint/cz-commitlint](https://link.juejin.cn?target=https%3A%2F%2Fcommitlint.js.org%2F%23%2Freference-prompt "https://commitlint.js.org/#/reference-prompt")

[npm 介绍](https://link.juejin.cn?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2F%40commitlint%2Fcz-commitlint "https://www.npmjs.com/package/@commitlint/cz-commitlint")

#### Install

```
npm install --save-dev @commitlint/cz-commitlint commitizen
复制代码

```

#### Usage

`package.json`

```
{
  "scripts": {
    "commit": "git-cz"
  },
  "config": {
    "commitizen": {
      "path": "@commitlint/cz-commitlint"
    }
  }
}
复制代码

```

#### 执行

```
git add .
npm run commit
复制代码

```

运行`npm run commit`将会出现如下图：

根据提示一步一步输入 commit message

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/29b5abbd436f4605acd229b82ad9f9f8~tplv-k3u1fbpfcp-watermark.awebp)

#### [prompt](https://link.juejin.cn?target=https%3A%2F%2Fcommitlint.js.org%2F%23%2Freference-prompt "https://commitlint.js.org/#/reference-prompt")

可以在`commitlint.config.js`配置文件中进行`rules`和`prompt`相关的规则，详细请查看官方文档[戳这里](https://link.juejin.cn?target=https%3A%2F%2Fcommitlint.js.org%2F%23%2Freference-prompt "https://commitlint.js.org/#/reference-prompt")。

检查历史提交 message
--------------

```
npx commitlint -- --from HEAD~1 --to HEAD --verbose
复制代码

```

生成 message 方式二：Commitizen
=========================

> [Commitizen](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fcommitizen%2Fcz-cli "https://github.com/commitizen/cz-cli") 是一个撰写合格 Commit message 的工具。

全局安装 commitizen
---------------

```
npm install -g commitizen
复制代码

```

项目初始化 commitizen
----------------

在项目目录里，运行下面的命令，使其支持 Angular 的 Commit message 格式

```
// 使用npm包cz-conventional-changelog进行初始化
commitizen init cz-conventional-changelog --save --save-exact
复制代码

```

[cz-conventional-changelog 介绍](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fcommitizen%2Fcz-conventional-changelog "https://github.com/commitizen/cz-conventional-changelog")

执行完上述命令后，会往`package.json`文件中的`devDependencies`中加入`cz-conventional-changelog`包，并且自动增加`config`配置项，如下：

```
"devDependencies": {
    "cz-conventional-changelog": "^3.3.0"
},
"config": {
   "commitizen": {
       "path": "./node_modules/cz-conventional-changelog"
    }
}
复制代码

```

之后，凡是用到`git commit`命令，一律改为`git cz`。这时就会出现选项，用来生成符合格式的 commit message。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e2db98bf9d8f4a1ea8d0566ae456bbe1~tplv-k3u1fbpfcp-watermark.awebp)

我们也可以在`package.json`文件中的`script`中配置命令：

```
"scripts": {
    ...
    "commit": "git cz"
},
复制代码

```

之后在执行`git commit`时，直接运行`npm run commit`即可！

完整版生成 message 示例
----------------

如果想生成`Breaking changes`，请在`Are there any breaking changes?`的时候选`y`

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d2bdf3b90215471fb5b2533931a1779d~tplv-k3u1fbpfcp-watermark.awebp)

生成 changelog
============

介绍
--

如果你的所有 commit 都符合`Angular commit规范`，那么发布新版本时，就可以通过脚本自动生成 changelog。

示例：[vue-next v3.1.0 changelog](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue-next%2Freleases%2Ftag%2Fv3.1.0 "https://github.com/vuejs/vue-next/releases/tag/v3.1.0")

**生成的 changelog 文档包括以下几个部分：**

*   Features（对应 type: feat）
*   Bug Fixes（对应 type: fix）
*   Code Refactoring (对应 type: refactor 且 breaking changes 为 y)
*   Performance Improvements（对应 type: perf）
*   Reverts (对应 type: revert)
*   BREAKING CHANGES (显示 body 中为 BREAKING CHANGES 的内容)

_每个部分都会罗列相关的 commit ，并且有指向这些 commit 的链接。当然，生成的文档允许手动修改，所以发布前，你还可以添加其他内容。_

conventional-changelog-cli
--------------------------

> 用来生成 changelog 的工具

[conventional-changelog-cli 介绍](https://link.juejin.cn?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Fconventional-changelog-cli "https://www.npmjs.com/package/conventional-changelog-cli")

### 安装

```
npm install -g conventional-changelog-cli
复制代码

```

### 生成 changelog

使用下述命令可生成 changelog：

```
conventional-changelog -p angular -i CHANGELOG.md -s
复制代码

```

上面命令不会覆盖以前的 Change log，只会在 CHANGELOG.md 的头部加上自从上次发布以来的变动。

如果你想生成所有发布的 Change log，需要运行下面的命令：

```
conventional-changelog -p angular -i CHANGELOG.md -s -r 0
复制代码

```

为了方便使用，可以将其写入`package.json`的`scripts`字段。

```
{
  "scripts": {
    "changelog": "conventional-changelog -p angular -i CHANGELOG.md -s -r 0"
  }
}
复制代码

```

以后，直接运行下面的命令即可。

```
npm run changelog
复制代码

```

### changelog 生成的依据

可能会有这样的疑问，如何生成某一个版本号所对应的 changelog 呢？

那就涉及到了`package.json`中的`version`字段，并且要在此版本开发完成的`commit`上创建一个`tag`。

这样，运行`npm run changelog`的时候，就会根据`version`和`tag`生成对应的版本 log。

示例
--

**1. commit log 如图：**

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9c689c393e5945b7a881706ddef3e888~tplv-k3u1fbpfcp-watermark.awebp) **2. 生成的 changelog 如图：**

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9b1434aaca7742b2b6c252fbaef2e3f9~tplv-k3u1fbpfcp-watermark.awebp)

总结
==

代码检查
----

```
// 安装
npx mrm@2 lint-staged
// 在package.json中配置如下脚本检查
"lint-staged": {
    "*.{js,vue}": [ // 对不同的后缀文件，运行不同的检查命令
      "vue-cli-service lint",
      "git add"
    ],
    "*.{ts}": [ // 对不同的后缀文件，运行不同的检查命令
      "xxx"
    ]
}
// husky中使用，`.husky/pre-commit`中，添加如下脚本：
npx lint-staged
// yorkie中使用，`package.json`中，添加如下脚本：
"gitHooks": {
    "pre-commit": "lint-staged"
}
复制代码

```

验证 message 规范性
--------------

### 脚本

`package.json`

```
"gitHooks": {
    "commit-msg": "node build/verify-commit-msg.js"
}
复制代码

```

`build/verify-commit-msg.js`

```
const chalk = require("chalk");
const msgPath = process.env.GIT_PARAMS;
const msg = require("fs")
    .readFileSync(msgPath, "utf-8")
    .trim();
const commitRE = /^(build|chore|ci|docs|feat|fix|wip|perf|refactor|revert|style|test|temp|)(\(.+\))?: .{1,50}/;
// 根据正则表达式校验提交的message是否符合团队规范
if (!(commitRE.test(msg) || msg.indexOf("Merge") === 0)) {
    console.error(
        `  ${chalk.bgRed.white(" ERROR ")}
  [${chalk.red(msg)}] 是 ${chalk.red("无效的提交消息格式")}
  ${chalk.red("自动生成更新日志需要正确的提交消息格式 例如:")}

  ${chalk.green("issue-1 feat(模块): 预发布环境增加 A 模块")}
  ${chalk.green("issue-2 fix(文案): 修复错误文案")}`
    );
    // 以非0值退出，放弃提交
    process.exit(1);
}
复制代码

```

### commitlint

```
// 安装
npm install --save-dev @commitlint/cli @commitlint/config-conventional
// 配置
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
// 在husky中使用
npx husky add .husky/commit-msg 'npx --no-install commitlint --edit $1'
// 在yorkie中使用
"gitHooks": {
    "commit-msg": "commitlint --edit"
}
复制代码

```

生成 message
----------

### @commitlint/prompt-cli

```
// 安装
npm install --save-dev @commitlint/prompt-cli
复制代码

```

`package.json`配置 script 脚本：

```
"scripts": {
    "commit": "commit"
}
// 提交时，直接运行npm run commit 即可
复制代码

```

### @commitlint/cz-commitlint

```
// 安装
npm install --save-dev @commitlint/cz-commitlint commitizen
复制代码

```

`package.json`：

```
{
    "scripts": {
        "commit":"git-cz"
    },
    "config": {
        "commitizen": {
            "path": "@commitlint/cz-commitlint"
         }
    }
}
// 提交时，直接运行npm run commit 即可
复制代码

```

### commitizen

```
npm install -g commitizen
// 切换至项目目录
commitizen init cz-conventional-changelog --save --save-exact
// 命令行执行git cz生成commit message
git cz   // 或者package.json配置`scripts`
复制代码

```

生成 changelog
------------

```
// 安装
npm install -g conventional-changelog
// 生成全部changelog
conventional-changelog -p angular -i CHANGELOG.md -s -r 0
// 生成自上次版本号变更以来changelog
conventional-changelog -p angular -i CHANGELOG.md -s
复制代码

```

常见问题
====

**1. gitHooks 钩子验证时，仅可保留下述方式的其中一种：**

*   `.git/hoos/`目录下，去掉`.sample`后缀后生效的钩子，如

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dd878ea503a1486792ef9f6c2d844c0c~tplv-k3u1fbpfcp-watermark.awebp)

*   `package.json`中通过`gitHoos`定义的钩子，可自定义通过 npm 包或者 js 文件去校验

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8fed73ca49704c34b9235c1cbdebe959~tplv-k3u1fbpfcp-watermark.awebp) **2. 每一次版本发布记得修改 package.json 中的 version 版本，生成 changelog 需根据版本号去生成**

参考代码库
=====

*   webpack：[github.com/webpack/web…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fwebpack%2Fwebpack%2Fblob%2Fmain%2Fpackage.json "https://github.com/webpack/webpack/blob/main/package.json")
*   vue：[github.com/vuejs/vue-n…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue-next%2Fblob%2Fmaster%2Fpackage.json "https://github.com/vuejs/vue-next/blob/master/package.json")

参考文档
====

1.  [www.jianshu.com/p/c7e40dab5…](https://link.juejin.cn?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fc7e40dab5b05 "https://www.jianshu.com/p/c7e40dab5b05")
2.  [www.ruanyifeng.com/blog/2016/0…](https://link.juejin.cn?target=http%3A%2F%2Fwww.ruanyifeng.com%2Fblog%2F2016%2F01%2Fcommit_message_change_log.html "http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html")
3.  [git 钩子官方介绍](https://link.juejin.cn?target=https%3A%2F%2Fwww.git-scm.com%2Fbook%2Fzh%2Fv2%2F%25E8%2587%25AA%25E5%25AE%259A%25E4%25B9%2589-Git-Git-%25E9%2592%25A9%25E5%25AD%2590 "https://www.git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90")
4.  [www.jianshu.com/p/62b5c6471…](https://link.juejin.cn?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F62b5c6471b1f "https://www.jianshu.com/p/62b5c6471b1f")
5.  [vue 代码库](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue-next%2Fblob%2Fmaster%2Fpackage.json "https://github.com/vuejs/vue-next/blob/master/package.json")
6.  [commitlint](https://link.juejin.cn?target=https%3A%2F%2Fcommitlint.js.org%2F%23%2F "https://commitlint.js.org/#/")