# 时间轮算法定时器antlabs/timer介绍

## 推荐理由

当业务要处理大量的定时任务时，如果每个任务都创建一个Golang原生的timer的话，会占用较多的cpu资源，这类场景，可以用时间轮算法优化timer的资源消耗。本次介绍一款多级时间轮库antlabs/timer（以下timer特指antlabs/timer库），处理类似场景的优化。

## 功能介绍

timer最小的时间粒度是10ms，
antlabs/timer支持以下功能：
1. 一次性定时，类似time.AfterFunc；
2. 周期性执行，类似time.Ticker；
3. 取消单个任务；
4. 停止所有任务。

## 使用指南

### 安装

```shell
go get github.com/antlabs/timer
```

### 代码示例

下面是两个简单的例子：

```golang
package main

import (
	"log"
	"time"

	"github.com/antlabs/timer"
)

func main() {
	tm := timer.NewTimer()

	// 一次性执行，2s后执行
	tm.AfterFunc(2*time.Second, func() {
		log.Printf("2 time.Second")
	})

	// tk3 会被 tk3.Stop()函数调用取消掉
	tk3 := tm.AfterFunc(3*time.Second, func() {
		log.Printf("3 time.Second")
	})

	tk3.Stop() //取消tk3

	// 周期执行，每1s执行一次
	tm.ScheduleFunc(1*time.Second, func() {
		log.Printf("schedule\n")
	})

	tm.Run()

}
```
timer使用起来是比较简单的，根据业务场景选择合适的接口就好。

### benchmark

Golang 1.14对定时器做了优化，每个P维护一个定时器，减少了添加删除任务时的锁竞争，但是和本文介绍的timer库对比起来还是有些耗资源，下面是benchmark的结果：

Golang 版本：1.16.6

```shell
cpu: Intel(R) Core(TM) i5-8257U CPU @ 1.40GHz
Benchmark_antlabs_Timer_AddTimer
Benchmark_antlabs_Timer_AddTimer/N-1m
Benchmark_antlabs_Timer_AddTimer/N-1m-8       	 9226951	       116.6 ns/op	      80 B/op	       1 allocs/op
Benchmark_antlabs_Timer_AddTimer/N-5m
Benchmark_antlabs_Timer_AddTimer/N-5m-8       	 9589074	       153.9 ns/op	      80 B/op	       1 allocs/op
Benchmark_antlabs_Timer_AddTimer/N-10m
Benchmark_antlabs_Timer_AddTimer/N-10m-8      	 9621186	       167.0 ns/op	      80 B/op	       1 allocs/op
Benchmark_Stdlib_AddTimer
Benchmark_Stdlib_AddTimer/N-1m
Benchmark_Stdlib_AddTimer/N-1m-8              	 4967716	       220.0 ns/op	      81 B/op	       1 allocs/op
Benchmark_Stdlib_AddTimer/N-5m
Benchmark_Stdlib_AddTimer/N-5m-8              	 5265825	       237.9 ns/op	      80 B/op	       1 allocs/op
Benchmark_Stdlib_AddTimer/N-10m
Benchmark_Stdlib_AddTimer/N-10m-8             	 7703940	       230.8 ns/op	      80 B/op	       1 allocs/op
```

## 总结

timer利用时间轮算法，通过降低定时器精度的方式，将同一个时间单位内的任务集中存储到一个双向链表，可以一次锁操作处理，减少锁竞争，进而提高性能，对于业务中有大量定时任务，同时对精度要求大于10ms的场景，可以尝试timer库来优化。

## 参考资料

1. [https://github.com/antlabs/timer](https://github.com/antlabs/timer)
2. [https://spongecaptain.cool/post/widget/timingwheel/](https://spongecaptain.cool/post/widget/timingwheel/)