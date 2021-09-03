# 高性能内存缓存 ristretto

## 背景

[**`ristretto`**](https://github.com/dgraph-io/ristretto) 是 [`dgraph`](https://dgraph.io/) 团队开源的一款高性能内存缓存库，旨在解决高并发场景下的缓存性能和吞吐瓶颈。`dgraph` 专攻的方向是高性能图数据库，`ristretto` 就是其图数据库和 KV 数据库产品的核心依赖。

与 golang 社区常见的其他单进程内存缓存类库（[`groupcache`](https://github.com/golang/groupcache)，[`bigcache`](https://github.com/allegro/bigcache)，[`fastcache`](https://github.com/VictoriaMetrics/fastcache) 等）相比，`ristretto` 在缓存命中率和读写吞吐率上的[综合表现](https://github.com/dgraph-io/ristretto#benchmarks)更优。

## ristretto 简介

`ristretto` 主要有以下优点：

* 高命中率 - 特殊设计的录入/驱逐政策
  * 驱逐（SampledLFU）：与精确 LRU 相当，但在搜索和数据跟踪上有更好的性能
  * 录入（TinyLFU）：以极小的内存开销获取额外的性能提升
* 高吞吐率
* 权重感知的驱逐策略 - 价值权重大的条目可以驱逐多个价值权重小的条目
  * 依托权重可以扩展出缓存最大内存占用、缓存最多条目数等场景
* 完全并发支持
* 性能指标 - 吞吐量、命中率及其他统计数据的性能指标
* 用户友好的 API 设计
  * 支持指定缓存失效时间

> `ristretto` 在 v0.1.0(2021-06-03) 版本发布时已正式标注为生产可用！

## ristretto 使用举例

### 构建大小（条目数）受限的缓存

让我们利用 `ristretto` 构建一个缓存条目数最大为 10 的缓存试试看：

```go
package main

import (
	"fmt"

	"github.com/dgraph-io/ristretto"
)

func main() {
	cache, err := ristretto.NewCache(&ristretto.Config{
		// num of keys to track frequency, usually 10*MaxCost
		NumCounters: 100,
		// cache size(max num of items)
		MaxCost: 10,
		// number of keys per Get buffer
		BufferItems: 64,
		// !important: always set true if not limiting memory
		IgnoreInternalCost: true,
	})
	if err != nil {
		panic(err)
	}

	// put 20(>10) items to cache
	for i := 0; i < 20; i++ {
		cache.Set(i, i, 1)
	}

	// wait for value to pass through buffers
	cache.Wait()

	cntCacheMiss := 0
	for i := 0; i < 20; i++ {
		if _, ok := cache.Get(i); !ok {
			cntCacheMiss++
		}
	}
	fmt.Printf("%d of 20 items missed\n", cntCacheMiss)
}
```

运行代码可以发现最后只有 10 个条目还保存在缓存中

```bash
$ go run main.go
10 of 20 item missed
```

> 注：当我们的缓存并非限制最大内存占用时，`IgnoreInternalCost` 一定要设为 `true`，否则创建出的缓存将出现诡异的表现。

### 测试缓存过期时间

还是创建一个简单的缓存，然后存一个过期时间为 1 秒的条目进去，看看接下来的缓存读写表现：

```go
package main

import (
	"log"
	"time"

	"github.com/dgraph-io/ristretto"
)

func main() {
	cache, err := ristretto.NewCache(&ristretto.Config{
		NumCounters:        100,
		MaxCost:            10,
		BufferItems:        64,
		IgnoreInternalCost: true,
	})
	if err != nil {
		panic(err)
	}

	// set item with 1s ttl
	cache.SetWithTTL("foo", "bar", 1, time.Second)

	// wait for value to pass through buffers
	cache.Wait()

	if val, ok := cache.Get("foo"); !ok {
		log.Printf("cache missing")
	} else {
		log.Printf("got foo: %v", val)
	}

	// sleep longer and try again
	time.Sleep(2 * time.Second)
	if val, ok := cache.Get("foo"); !ok {
		log.Printf("cache missing")
	} else {
		log.Printf("got foo: %v", val)
	}
}
```

运行代码可以发现已过期的条目被正常清除出了缓存

```bash
$ go run main.go
2021/09/03 14:19:56 got foo: bar
2021/09/03 14:19:58 cache missing
```

## 总结

`ristretto` 是支持高并发高吞吐的内存缓存库，尤其适用于数据库、搜索引擎、文件系统等 io 密集场景。需要注意的是 `ristretto` 只适用于单机单进程的缓存方案，更像是 golang 中的 [`Caffeine`](https://github.com/ben-manes/caffeine) (java)，并不作为 redis 和 memcache 的替代品。

大家赶快试试吧！

## 参考资料

* [https://github.com/dgraph-io/ristretto](https://github.com/dgraph-io/ristretto)
* [https://dgraph.io/blog/post/introducing-ristretto-high-perf-go-cache/](https://dgraph.io/blog/post/introducing-ristretto-high-perf-go-cache/)
* [https://github.com/dgraph-io/badger](https://github.com/dgraph-io/badger)
* [https://github.com/hashicorp/golang-lru](https://github.com/hashicorp/golang-lru)
* [https://github.com/golang/groupcache](https://github.com/golang/groupcache)

---

欢迎加入 GOLANG 中国社区：[https://gocn.vip](https://gocn.vip)
