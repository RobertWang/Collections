> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/uJLKyCjCRi1sQ5YV8w8FhQ)

背景
--

前一段时间线上出现了一个问题：在压测后偶尔会出现一台机器查询数据无结果但是没有返回 `err` 的情况，导致后续处理都出错。由于当时我们仅在最外层打印了 `err` ，没有打印入参和出参，所以导致很难排查问题到底出现在哪一环节。

经过艰难地排查出问题后，感觉需要在代码里添加打印关键函数的入参和出参数，但这个逻辑都是重复的，也不想将这一逻辑侵入开发流程，所以就想到了代码生成方式的注释注解。即我们提前定义好一个注释注解（例如：`// @Log()`），并且在 Docker 中编译前运行代码生成的逻辑，将所有拥有该注释注解的函数进行修改，在函数体前面添加打印入参和出参的逻辑。这样就不需要让日志打印侵入到业务代码中，并且后续可以很方便替换成其他的打印逻辑（例如根据`@Log` 内的参数或者返回值等自定义日志级别）。

编写代码
----

我们可以使用 AST 的方式去解析、识别并修改代码， `go/ast` 已经提供了相应的功能，我们查看我们关心的节点部分及其相关的信息结构，可以使用 goast-viewer[1] 直接查看 AST ，当然也可以本地进行调试。

### 遍历 `.go` 文件

首先需要使用 `filepath.Walk` 函数遍历指定文件夹下的所有文件，对每一个文件都会执行传入的 `walkFn` 函数。 `walkFn` 函数会将 `.go` 文件解析成 AST ，将其交由注释注解的处理器处理，然后根据是否修改了 AST 决定是否生成新的代码。

```
// walkFn 函数会对每个 .go 文件处理，并调用注解处理器
func walkFn(path string, info os.FileInfo, err error) error {
    // 如果是文件夹，或者不是 .go 文件，则直接返回不处理
    if info.IsDir() || !strings.HasSuffix(info.Name(), ".go") {
        return nil
    }
    // 将 .go 文件解析成 AST
    fileSet, file, err := parseFile(path)
    // 如果注解修改了内容，则需要生成新的代码
    if logannotation.Overwrite(path, fileSet, file) {
        buf := &bytes.Buffer{}
        if err := format.Node(buf, fileSet, file); err != nil {
            panic(err)
        }
        // 如果不需要替换，则生成到另一个文件
        if !replace {
            lastSlashIndex := strings.LastIndex(path, "/")
            genDirPath :=path[:lastSlashIndex] + "/_gen/"
            if err := os.Mkdir(genDirPath, 0755); err != nil && os.IsNotExist(err){
                panic(err)
            }
            path = genDirPath + path[lastSlashIndex+1:]
        }
        if err := ioutil.WriteFile(path, buf.Bytes(), info.Mode()); err != nil {
            panic(err)
        }
    }
    return nil
}

```

### 遍历 AST

当注释注解处理器拿到 AST 后，就需要使用 `astutil.Apply` 函数遍历整颗 AST ，并对每个节点进行处理，同时为了方便修改时添加 `import` ，我们包一层函数供内部调用，并把一些关键信息打包在一起。

### 识别注释注解

接下来我们就需要识别注释注解，跳过不相关的节点，示例中不做额外处理，仅当注释为 `@Log()` 才认为需要处理，可以根据需要添加相应的逻辑。

### 获取函数入参和出参

首先我们需要获取函数的入参和出参，这里我们以出参举例。出参定义在 `funcDecl.Type.Results` ，并且可能没有指定名称，所以需要先为以 `_0`, `_1`, `...` 这样的形式为没有名称的变量设置默认名称，然后按照顺序获取所有变量的名称列表。

```
// SetDefaultNames 给没有名字的 Field 设置默认的名称
// 默认名称格式：_0, _1, ...
// true: 表示至少设置了一个名称
// false: 表示未设置过名称
func SetDefaultNames(fields ...*ast.Field) bool {
    index := 0
    for _, field := range fields {
        if field.Names == nil {
            field.Names = NewIdents(fmt.Sprintf("_%v", index))
            index++
        }
    }
    return index > 0
}

```

### 获取打印语句

```
// NewCallExpr 产生一个调用表达式
// 待产生表达式：log.Logger.WithContext(ctx).Infof(arg0, arg1)
// 其中：
//    funcSelector = "log.Logger.WithContext(ctx).Infof"
//     args = ("arg0", "arg1")
// 调用语句：NewCallExpr("log.Logger.WithContext(ctx).Infof", "arg0", "arg1")
func NewCallExpr(funcSelector string, args ...string) (*ast.CallExpr, error) {
    // 获取函数对应的表达式
    funcExpr, err := parser.ParseExpr(funcSelector)
    if err != nil {
        return nil, err
    }
    // 组装参数列表
    argsExpr := make([]ast.Expr, len(args))
    for i, arg := range args {
        argsExpr[i] = ast.NewIdent(arg)
    }
    return &ast.CallExpr{
        Fun:  funcExpr,
        Args: argsExpr,
    }, nil
}

```

```
// NewFuncLitDefer 产生一个 defer 语句，运行一个匿名函数，函数体是入参语句列表
func NewFuncLitDefer(funcStmts ...ast.Stmt) *ast.DeferStmt {
    return &ast.DeferStmt{
        Call: &ast.CallExpr{
            Fun: NewFuncLit(&ast.FuncType{}, funcStmts...),
        },
    }
}

```

### 修改函数体

```
toBeAddedStmts := []ast.Stmt{
    &ast.ExprStmt{X: beforeExpr},
    // 离开函数时的语句使用 defer 调用
    NewFuncLitDefer(&ast.ExprStmt{X: afterExpr}),
}
// 我们将添加的语句放在函数体最前面
funcDecl.Body.List = append(toBeAddedStmts, funcDecl.Body.List...)

```

运行
--

```
package main
import (
    "context"
    "logannotation/testdata/log"
)
func main() {
    fn(context.Background(), 1, "2", "3", true)
}
// @Log()
func fn(ctx context.Context, a int, b, c string, d bool) (int, string, string) {
    log.Logger.WithContext(ctx).Infof("#fn executing...")
    return a, b, c
}

```

```
package main
import (
    "context"
    "logannotation/testdata/log"
)
func main() {
    fn(context.Background(), 1, "2", "3", true)
}
func fn(ctx context.Context, a int, b, c string, d bool) (_0 int, _1 string, _2 string) {
    log.Logger.WithContext(
        ctx).WithField("filepath", "/Users/idealism/Workspaces/Go/golang-log-annotation/testdata/main.go").Infof("#fn start, params: %+v, %+v, %+v, %+v", a, b, c, d)
    defer func() {
        log.Logger.WithContext(
            ctx).WithField("filepath", "/Users/idealism/Workspaces/Go/golang-log-annotation/testdata/main.go").Infof("#fn end, results: %+v, %+v, %+v", _0, _1, _2)
    }()
    log.Logger.WithContext(ctx).Infof("#fn executing...")
    return a, b, c
}

```

```
INFO[0000] #fn start, params: 1, 2, 3, true              filepath=/Users/idealism/Workspaces/Go/golang-log-annotation/testdata/main.go
INFO[0000] #fn executing...                             
INFO[0000] #fn end, results: 1, 2, 3                     filepath=/Users/idealism/Workspaces/Go/golang-log-annotation/testdata/main.go

```

扩展
--

### References