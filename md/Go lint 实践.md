> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5OTA1MDUyMA==&mid=2655465109&idx=4&sn=ef952abc23a1d6f75a7e603d19c9d61f&chksm=bd72e2e28a056bf41ef23730551bab0d53a8c40fa9ffff980ee0e089d9966b82173dc72d4a4c&scene=21#wechat_redirect)

【导读】go 语言静态检查工具 go lint 是什么？如何使用？本文详细介绍了 go lint。  

什么是 lint？
---------

静态代码分析。通俗地讲，扫描源代码，在不运行代码的情况下，找出一些不规范的书写方式以修正，  
例如：`if foo != false {...}` 这样的写法显然不如 `if foo {}` 好理解。  
lint 程序扫描到这样的 case 就可以给出提示，敦促开发者修正。下面是一些 lint 能检测到的问题样例：

```
fmt.Sprintf("%d", "123")  // 错误
fmt.Sprintf("%s", "123")  // 正确

for i, _ := range []int{1,2,3}{} // 错误
for i := range []int{1,2,3}{}    // 正确

a := make([]int, 0, 0)  // 错误
a := make([]int, 0)     // 正确

for _, v := range vs {          // 错误
    vectors = append(vectors, v)
}
vectors = append(vectors, vs...) // 正确
```

### 怎样使用 lint？

1.  执行扫描工作的程序是一个 bin 文件（称为 linter）。执行它就可以扫描项目源代码给出提示。  
    在 Go 生态内的 linter 有很多种，大部分是开源项目提供的，它们能扫描各式各样的问题如 “废弃代码”，“冗余的写法”，“调用 deprecated api” 等。
    
2.  经调研，在 Go 开发中最好的选择不是使用某一种 linter，而是使用 Golangci-lint，他是一个开源的 linter 框架，作用类似手机上的应用市场，你可以通过配置选择开启五花八门的 linter，兼顾了使用灵活、功能强大。
    
3.  golangci-lint 的安装使用见后文。
    

### 怎样将 lint 集成到团队工作流？

1.  团队工作中代码仓库是共享的，一个人跑 lint 没用，需要大家遵循同一套 lint 规则。
    
2.  具体办法是，将 linter 的运行加入到 ci 流程中。  
    任意开发 push 新代码，都会触发 lint，结果展示在 merge request 界面。如果存在 bad case，就不允许 merge 到 master 分支。这样协作项目的代码就能始终符合 lint 规范。
    

### 实践介绍

**背景：** 团队代码库，因为历史遗留小问题很多，肉眼排查不实际，想到引入 lint 解决该问题。  
**ci 工具选择：**linter 如上介绍使用 Golangci-lint。我们公司使用 Gitlab 作为源码管理平台，ci 工具使用它提供的 Gitlabci。

下面是我自己做的准备工作，

1.  **安装 Golangci-lint**，命令如下，详细说明可参考其 README
    

```
GOPATH_FIRST=$(echo $(go env GOPATH) | awk -F':' '{ print $1}')
curl -sfL https://install.goreleaser.com/github.com/golangci/golangci-lint.sh | sh -s -- -b ${GOPATH_FIRST}/bin v1.21.0
```

2. **配置 Golangci-lint**  
在项目根目录下创建文件`.golangci.yml`，这个文件是 golangci-lint 的配置文件，可自定义 `开启/关闭哪些linter`、`跳过某些文件`、`跳过某些lint规则`等。  
如无配置文件，Golangci-lint 会开启一些预设 linter，这些在其文档内有说明。

下面是我们组一个项目现在的`.golangci.yml`内容，

```
linters:
  disable-all: true  # 关闭其他linter
  enable:            # 下面是开启的linter列表，之后的英文注释介绍了相应linter的功能
    - deadcode      # Finds unused code
    # - errcheck      # Errcheck is a program for checking for unchecked errors in go programs. These unchecked errors can be critical bugs in some cases
    - gosimple      # Linter for Go source code that specializes in simplifying a code
    - govet         # Vet examines Go source code and reports suspicious constructs, such as Printf calls whose arguments do not align with the format string
    - ineffassign   # Detects when assignments to existing variables are not used
    - staticcheck   # Staticcheck is a go vet on steroids, applying a ton of static analysis checks
    - structcheck   # Finds unused struct fields
    - typecheck     # Like the front-end of a Go compiler, parses and type-checks Go code
    - unused        # Checks Go code for unused constants, variables, functions and types
    - varcheck      # Finds unused global variables and constants
    - scopelint     # Scopelint checks for unpinned variables in go programs
    # - golint        # Carry out the stylistic conventions put forth in Effective Go and CodeReviewComments

linters-settings:
  govet:            # 对于linter govet，我们手动开启了它的某些扫描规则
    check-shadowing: true
    check-unreachable: true
    check-rangeloops: true
    check-copylocks: true
```

3. **运行 linter / 修正错误**  
在项目根目录下运行`golangci-lint run`，  
该命令会加载`.golangci.yml`（如有），执行 lint。如代码有问题，命令行会输出错误提示，包括犯错地点、修正方法，有时 linter 提示会比较含糊，拿关键字 google 一下能找到详细说明

![](https://mmbiz.qpic.cn/mmbiz_jpg/IgylNib7ZE2K1673lf1xWWjQxAVKTM8uvoibtjmrsT9lxPDoH2OLiaweR9D5pCeiaq0ucUdHTxr6ictUOfkYYxic6nsA/640?wx_fmt=jpeg)

4. **推广**  
为做到平滑的推广 lint，在操作时有下面的实践  

*   分批放开 linter，先启用规则简单的 linter 例如`deadcode`，`unused`，之后开启规则更严格的 linter 例如`gosimple`。以使同事逐渐适应。
    
*   在开启一个 linter 前，我会手动修复该 linter 对应的的所有错误。确保之后发起 mr 的同事只需为新改动负责。
    

5. ** 与 gitlabci 集成，** 走到这里 lint 工具的准备就 ok 了，下面要将它和 ci 工具衔接在一起，

1.  参考 Gitlabci 文档，安装 gitlabci runner
    
2.  将它和 gitlab project 绑定在一起。自行安装 gitlab runner 有点复杂，我们公司的工程效率团队提供了公用的 docker runner，我这里只要做好绑定就行。
    
3.  在项目根目录下创建一个名为`.gitlab-ci.yml`的配置文件，这样每当 push 代码时，gitlab 检测到有该文件，就会触发 runner 执行 ci。我的`.gitlab-ci.yml`文件内容如下
    

```
image: registry.company.cn/ee/go:1.12.9-4

stages:
  - qa

lint:
  stage: qa
  script:
    - golangci-lint runGitLab CI/CD | GitLabimage: registry.company.cn/ee/go:1.12.9-4

stages:
  - qa

lint:
  stage: qa
  script:
    - golangci-lint run
```

> 转自：夏路
> 
> zhuanlan.zhihu.com/p/143567949

- EOF -