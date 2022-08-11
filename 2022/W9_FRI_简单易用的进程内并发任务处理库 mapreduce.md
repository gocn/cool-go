# 简单易用的进程内并发任务处理库 mapreduce

## 为什么需要 mapreduce？

在实际的业务场景中我们常常需要从不同的 rpc 服务中获取相应属性来组装成复杂对象。

比如要查询商品详情：

1. 商品服务-查询商品属性
2. 库存服务-查询库存属性
3. 价格服务-查询价格属性
4. 营销服务-查询营销属性

如果是串行调用的话响应时间会随着 rpc 调用次数呈线性增长，所以我们要优化性能一般会将串行改并行。

简单的场景下使用 waitGroup 也能够满足需求，但是如果我们需要对 rpc 调用返回的数据进行校验、数据加工转换、数据汇总呢？继续使用 waitGroup 就有点力不从心了，go 的官方库中并没有这种工具（java 中提供了 CompleteFuture），go-zero 作者依据 mapReduce 架构思想实现了进程内的数据批处理 mapReduce 并发工具类。

## 核心思想

* 数据生产 generate

* 数据加工 mapper

* 数据聚合 reducer

其中数据生产是不可或缺的阶段，数据加工、数据聚合是可选阶段，数据生产与加工支持并发调用，数据聚合基本属于纯内存操作单协程即可。不同阶段的数据处理由不同的 goroutine 执行，不同 goroutine 之间通过 channel 通信，通过 goroutine 中监听一个全局的结束channel 和调用方提供的 ctx 即能实现整个流程的随时终止 。

## 快速入门

首先安装 mapreduce.

> 目前维护了两个版本 v1, v2,其中 v2 为泛型版本.

```shell
go get github.com/kevwan/mapreduce
# go get github.com/kevwan/mapreduce/v2
```

官方提供的一个求平方和的简单例子如下：

```go
package main

import (
    "fmt"
    "log"

    "github.com/kevwan/mapreduce"
)

func main() {
    val, err := mapreduce.MapReduce(func(source chan<- interface{}) {
        // generator
        for i := 0; i < 10; i++ {
            source <- i
        }
    }, func(item interface{}, writer mapreduce.Writer, cancel func(error)) {
        // mapper
        i := item.(int)
        writer.Write(i * i)
    }, func(pipe <-chan interface{}, writer mapreduce.Writer, cancel func(error)) {
        // reducer
        var sum int
        for i := range pipe {
            sum += i.(int)
        }
        writer.Write(sum)
    })
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println("result:", val)
}
```

使用起来简单方便,  重要的是源码只有几百行, 可以看看源码作者是如何把 goroutine 与 channel 结合的这么精妙, 另外也可以根据自己的业务需要改进代码.

## 参考资料

* https://github.com/kevwan/mapreduce