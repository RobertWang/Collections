> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/a-4eK32ul5U6a488vfY9sg)

在 Go 语言中，有一些常用的性能压测工具，可以帮助你测试和评估你的应用程序在不同负载下的性能表现。以下是一些常用的性能压测工具，并附带简要的配置和示例：

**1 Apache Benchmark (ab)**：  
`ab` 是一个 Apache 自带的命令行工具，用于对 HTTP 服务器进行基准测试。你可以使用以下命令进行安装（假设你已经安装了 Apache HTTP Server）：

```
sudo apt-get install apache2-utils  # For Debian/Ubuntu


```

使用示例：

```
ab -n 100 -c 10 http://yourserver.com/


```

在上述示例中，`-n` 表示总请求数，`-c` 表示并发数。

**2 Vegeta**：  
`Vegeta` 是一个开源的 HTTP 性能测试工具，它支持丰富的配置选项，并且可以生成结果报告。你可以通过以下方式安装：

```
go get -u github.com/tsenart/vegeta


```

使用示例：

```
echo "GET http://yourserver.com/" | vegeta attack -rate=100/s -duration=10s | vegeta report


```

在上述示例中，`attack` 命令表示发起攻击，`report` 命令用于生成结果报告。

**3 Hey**：  
`Hey` 是一个现代化的 HTTP 性能测试工具，使用简单。你可以通过以下方式安装：

```
go get -u github.com/rakyll/hey


```

使用示例：

```
hey -n 1000 -c 10 http://yourserver.com/


```

在上述示例中，`-n` 表示总请求数，`-c` 表示并发数。

**4 k6**：  
`k6` 是一个可扩展的负载测试工具，支持多种脚本和扩展。你可以通过以下方式安装：

```
brew install k6  # For macOS


```

使用示例：

```
k6 run script.js


```

其中 `script.js` 是你的测试脚本文件。

这些工具在使用时，你需要根据你的应用程序和需求进行适当的配置，例如指定请求的 URL、并发数、持续时间等。

请注意，在进行性能测试时，要确保测试环境和测试方法是真实可靠的，以获得准确的性能指标。同时，不同工具的参数和配置可能有所不同，建议查阅它们的官方文档以获取更详细的配置信息。