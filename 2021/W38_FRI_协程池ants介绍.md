# 协程池ants介绍

## 推荐理由

协程泄漏引发的血案，想必各位gopher都经历过，通过协程池限制goroutine数是一个有效避免泄漏的手段。今天介绍的ants库是公认且优秀的协程池实现。ants Github主页上的简介：
> ants是一个高性能的 goroutine 池，实现了对大规模 goroutine 的调度管理，允许使用者在开发并发程序的时候限制 goroutine 数量，复用资源，达到更高效执行任务的效果。

## 功能介绍

- 自动调度海量的 goroutines，复用 goroutines
- 定期清理过期的 goroutines，进一步节省资源
- 提供了大量有用的接口：任务提交、获取运行中的 goroutine 数量、动态调整 Pool 大小、释放 Pool、重启 Pool
- 优雅处理 panic，防止程序崩溃
- 资源复用，极大节省内存使用量；在大规模批量并发任务场景下比原生 goroutine 并发具有更高的性能
- 非阻塞机制

## 使用指南

### 目前测试通过的 Golang 版本：

v1.8.x** ～ v1.16.x** 的所有版本。

### 安装

```shell
go get -u github.com/panjf2000/ants
```

使用 v2 版本 (开启 GO111MODULE=on):

```shell
go get -u github.com/panjf2000/ants/v2
```

### 代码示例

ants支持两种协程池，Pool和PoolWithFunc，差别在于，Pool每个任务是一个函数，PoolWithFunc任务函数在初始化时确定，每个任务只是不同的入参调用该函数。

#### Pool

```go
import (
    "fmt"
    "sync"
    "sync/atomic"
    "time"

    "github.com/panjf2000/ants/v2"
)

func demoFunc() {
    time.Sleep(10 * time.Millisecond)
    fmt.Println("Hello World!")
}

func main() {
    // 释放ants的默认协程池
    defer ants.Release()

    var wg sync.WaitGroup
    // 任务函数
    syncCalculateSum := func() {
        demoFunc()
        wg.Done()
    }
    for i := 0; i < 100; i++ {
        wg.Add(1)
        // 提交任务到默认协程池
        _ = ants.Submit(syncCalculateSum)
    }
    wg.Wait()
    fmt.Printf("running goroutines: %d\n", ants.Running())
    fmt.Printf("finish all tasks.\n")
}
```

#### PoolWithFunc

```go
import (
    "fmt"
    "sync"
    "sync/atomic"
    "time"

    "github.com/panjf2000/ants/v2"
)

var sum int32

func myFunc(i interface{}) {
    n := i.(int32)
    atomic.AddInt32(&sum, n)
    fmt.Printf("run with %d\n", n)
}

func main() {
    // 初始化协程池
    p, _ := ants.NewPoolWithFunc(10, func(i interface{}) {
        myFunc(i)
        wg.Done()
    })
    // 释放协程池
    defer p.Release()
    // 提交任务
    for i := 0; i < runTimes; i++ {
        wg.Add(1)
        _ = p.Invoke(int32(i))
    }
    wg.Wait()
    fmt.Printf("running goroutines: %d\n", p.Running())
    fmt.Printf("finish all tasks, result is %d\n", sum)
}
```

#### 协程池配置

ants协程池支持以下配置项：
- 协程池大小设置，可通过
- worker协程过期时间，过期后的协程会被清理
- 内存预分配，是否初始化Pool的时候分配worker资源
- 最大阻塞任务数，默认0，任务阻塞直到分配到worker；当非0时，达到最大任务数，不执行任务并返回overload
- 非阻塞提交，默认否，任务无worker执行时阻塞，当为true时，无worker则立即返回overload
- 自定义panic处理函数
- 自定义日志函数

代码片段：

```go
    // 设置配置项
    options := Options{}
	options.ExpiryDuration = time.Duration(10) * time.Second
	options.Nonblocking = true
	options.PreAlloc = true
    // 初始化
	poolOpts, _ := NewPool(10, WithOptions(options))

```

以上是ants的基本使用，更详细的内容可以参考ants源码。

## 总结

协程池通过复用和限制goroutine数，可以减轻runtime调度压力，避免过多的gorouutine占用系统cpu和内存资源，ants作为协程池实现优秀代表，功能丰富且简单易用，推荐在适合的场景使用。

## 参考资料

1. [https://github.com/panjf2000/ants](https://github.com/panjf2000/ants)
2. [https://strikefreedom.top/high-performance-implementation-of-goroutine-pool](https://strikefreedom.top/high-performance-implementation-of-goroutine-pool)
