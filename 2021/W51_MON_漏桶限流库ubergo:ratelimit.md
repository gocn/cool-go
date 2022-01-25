# 漏桶限流库 — uber-go/ratelimit

上次有同学分享了 [单机限流器 time/rate 库](https://gocn.vip/topics/12239)，讲了 Golang 标准库中基于令牌桶实现限流组件的 `time/rate` 使用，同时也讲了一些限流算法原理。

这里分享一个 uber 开源的一套基于漏桶实现的用于服务限流的 golang 库 [ratelimit](https://github.com/uber-go/ratelimit/)。

漏洞算法的理解起来，相较于令牌桶，没有那么直观。因为令牌桶是限制 “令牌” 的速率，拿到令牌的请求才能被放行，而漏桶是限制单位时间内允许通过的请求数目，调用方只能严格按照预定的间隔顺序进行消费调用。

## ratelimit 的基本使用

官方示例：

```go
import (
	"fmt"
	"time"

	"go.uber.org/ratelimit"
)

func main() {
    rl := ratelimit.New(100) // per second

    prev := time.Now()
    for i := 0; i < 10; i++ {
        now := rl.Take()
        fmt.Println(i, now.Sub(prev))
        prev = now
    }
}

```

在这个例子中，我们给定限流器每秒可以通过 100 个请求，也就是平均每个请求间隔 10ms。
因此，最终会每 10ms 打印一行数据。输出结果如下：

```go
// Output:
// 0 0
// 1 10ms
// 2 10ms
// 3 10ms
// 4 10ms
// 5 10ms
// 6 10ms
// 7 10ms
// 8 10ms
// 9 10ms
```

## 高级用法

### 最大松弛量

传统的漏桶，每个请求的间隔是固定、匀速的，两次请求 req1 和 req2 之间的延迟至少应该 >=perRequest，然而，在实际上的应用中，流量经常是突发性的。
对于突发这种情况，uber-go 对 Leaky Bucket 做了一些改良，引入了最大松弛量（maxSlack）的概念, 表示允许抵消的最长时间

漏桶算法的限速逻辑：
```go
sleepFor = t.perRequest - now.Sub(t.last)
if sleepFor > 0 {
	t.clock.Sleep(sleepFor)
	t.last = now.Add(sleepFor)
} else {
	t.last = now
}
```

上面计算 sleepFor 的代码，不使用最大松弛量的 sleep 逻辑，严格要求两个请求之间必须间隔 `t.perRequest` 的时间，那么计算 `sleepFor` 的代码就是：

```
t.sleepFor = t.perRequest - now.Sub(t.last)
```

这样就会有以下问题：

我们假设现在有三个请求，`req1`、`req2` 和 `req3`，限速区间为 `10ms`。
`req1` 最先到来，当 `req1` 完成之后 `15ms` ，`req2` 才到来，依据限速策略可以对 `req2` 立即处理，当 `req2` 完成后，`5ms` 后， `req3` 到来，这个时候距离上次请求还不足 `10ms`，因此还需要等待 `5ms` 才能继续执行, 但是，对于这种情况，实际上这三个请求一共消耗了 `25ms` 才完成，并不是预期的 `20ms`。

上面这个 case 模拟了一种请求量突发的状况。但这里我们可以把之前间隔比较长的请求的时间（累加超出 `perRequest` 的时延），匀给后面的请求判断限流时使用。 对于上面的 case，因为 `req2` 相当于多等了 `5ms`，我们可以把这 `5ms` 移给 `req3` 使用。加上 `req3` 本身就是 `5ms` 之后过来的，一共刚好 `10ms`，所以 `req3` 无需等待，直接可以处理。此时三个请求也恰好一共是 `20ms`。如下：

```go
t.sleepFor += t.perRequest - now.Sub(t.last)
if t.sleepFor > 0 {
  t.clock.Sleep(t.sleepFor)
  t.last = now.Add(t.sleepFor)
  t.sleepFor = 0
} else {
  t.last = now
}
```

当 `t.sleepFor > 0`，代表此前的请求多余出来的时间，无法完全抵消此次的所需量，因此需要 sleep 相应时间, 同时将 `t.sleepFor` 置为 0。

当 `t.sleepFor < 0`，说明此次请求间隔大于预期间隔，将多出来的时间累加到 `t.sleepFor` 即可。

但是，对于某种情况，请求 1 完成后，请求 2 过了很久到达 ，那么此时对于请求 2 的请求间隔 `now.Sub(t.last)`，会非常大。以至于即使后面大量请求瞬时到达，也无法抵消完这个时间。那这样就失去了限流的意义。

为了防止这种情况，ratelimit 就引入了最大松弛量 (maxSlack) 的概念, 该值为负值，表示允许抵消的最长时间，防止以上情况的出现。

```
// 当计算出来的 sleepFor 超过一个 maxSlack 时，那么就只 sleep 一个 maxSlack 时间，用于两次请求时间间隔过大的场景
if t.sleepFor < t.maxSlack {
	t.sleepFor = t.maxSlack
}
```

ratelimit 中 maxSlack 的值为 `-10 * time.Second / time.Duration(rate)`, 是十个请求的间隔大小。我们也可以理解为 ratelimit 允许的最大瞬时请求为 10。

### 用法

ratelimit 的 New 函数，除了可以配置每秒请求数 (QPS)， 还提供了一套可选配置项 Option。

```go
func New(rate int, opts ...Option) Limiter
```

### `WithoutSlack`

上面说到 ratelimit 中引入了最大松弛量的概念，而且默认的最大松弛量为 10 个请求的间隔时间。

但是对于需要严格的限制请求的固定间隔的时，我们可以利用 WithoutSlack 来取消松弛量的影响。

```go
limiter := ratelimit.New(100, ratelimit.WithoutSlack)
```

另外，ratelimit 还有其他选项，比如`WithClock(clock Clock)` 可以用来替换默认时钟，来实现精度更高或者特殊需求的计时场景。

## Reference

[uber-go 漏桶限流器使用与原理分析 | 编程沉思录 (cyhone.com)](https://www.cyhone.com/articles/analysis-of-uber-go-ratelimit/)

[开源限流组件分析（一）：Uber 的 Leaky Bucket - 熊喵君的博客 | PANDAYCHEN](https://pandaychen.github.io/2020/07/31/A-UBER-LEAKY-BUCKET-LIMITER-ANALYSIS/)

[限流算法之一 | hzSomthing (hedzr.com)](https://hedzr.com/golang/algorithm/rate-limit-1/#比较)

[uber-go/ratelimit: A Golang blocking leaky-bucket rate limit implementation (github.com)](https://github.com/uber-go/ratelimit)

