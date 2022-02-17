# 简单易用的性能分析工具nitro

## 推荐理由

Golang官方在程序性能分析方面提供了用来分析cpu/内存等采样信息的pprof，以及用来追踪和分析运行时事件的trace，这两个工具对于分析程序的性能瓶颈可以说是得心应手。但是，对于想分析一个项目中的某个代码片段或者函数的耗时/内存占用，pprof和trace就有点不是很方便了。本文介绍的nitro库可以简单方便的应用于类似的性能分析场景。

## 功能介绍

源码不到200行的nitro，提供执行时间和内存占用的打点和统计功能。

## 使用指南

> 注意：nitro非协程安全，也不支持调用链上的性能分析。

### 安装

```shell
go get github.com/spf13/nitro
```

### 代码示例

nitro使用起来非常简单，下面是个简单示例，分析someFunc和otherFunc函数执行耗时和内存分配。

```go
package main

import (
	"flag"

	"github.com/spf13/nitro"
)

func analysis() {
	timer := nitro.Initialize()
	// 通过命令行参数开启分析
	flag.Parse()
	// 默认开启
	// nitro.AnalysisOn = true

	someFunc()
	timer.Step("step 1, write index")

	otherFunc()
	timer.Step("step 2, batch insert")
}
```

nitro将打点结果直接输出到标准输出，对于想输出到日志文件或者监控系统的，源码本身也很简单，改造一下也比较方便，可结合场景自行改造。

## 总结

nitro适用于分析代码片段的耗时/内存占用，对于类似场景可以考虑使用。

## 参考资料
- [https://github.com/spf13/nitro](https://github.com/spf13/nitro)